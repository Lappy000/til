# AWS Bedrock cross-region inference bypasses geo-restrictions

When AWS Bedrock blocks Claude / Anthropic models for callers in unsupported
countries, the error looks like this:

```
ValidationException: Access to Anthropic models is not allowed from
unsupported countries, regions, or territories.
```

This check is enforced **per-request** based on the caller's source IP, not
the AWS account or IAM principal. A few practical implications:

- An IAM user with `AdministratorAccess` still gets blocked if the request
  originates from a denied region.
- The check fires on `Converse` / `ConverseStream`, not on `ListFoundationModels`.
  You can enumerate available models, then fail at invocation time.
- Cross-region inference profiles (`us.*`, `eu.*`, `global.*` model IDs) route
  the actual inference through a pool of regions, but the **caller-side**
  geo check still applies before routing.

## Workaround: front the API with a relay

Run a thin OpenAI-compatible relay on a host inside an allowed region, point
it at Bedrock with boto3, and tunnel the relay back to the original caller
(ngrok / bore / Cloudflare Tunnel). The Bedrock API call now originates from
the allowed region; the local caller talks plain HTTPS to the relay.

Minimum viable relay (FastAPI):

```python
from fastapi import FastAPI, Request
import boto3, json

app = FastAPI()
client = boto3.client("bedrock-runtime", region_name="us-east-1")

@app.post("/v1/chat/completions")
async def completions(req: Request):
    body = await req.json()
    resp = client.converse(
        modelId=body["model"],
        messages=[{"role": m["role"],
                   "content": [{"text": m["content"]}]}
                  for m in body["messages"]],
        inferenceConfig={"maxTokens": body.get("max_tokens", 1024)},
    )
    text = resp["output"]["message"]["content"][0]["text"]
    return {
        "id": "chatcmpl-relay",
        "object": "chat.completion",
        "model": body["model"],
        "choices": [{"index": 0,
                     "message": {"role": "assistant", "content": text},
                     "finish_reason": "stop"}],
        "usage": resp.get("usage", {}),
    }
```

Run it on `0.0.0.0:8069`, expose via tunnel, point clients at the public URL
with an OpenAI client SDK. No code changes needed on the caller side beyond
`base_url` and an API key.

## Detection notes

If you're on the defender side, this pattern is visible:

- All Bedrock invocations originate from one EC2 / Lambda / on-prem IP.
- The same IAM principal services many distinct downstream user IDs.
- Request frequency exceeds normal interactive usage for a single workstation.

CloudTrail `bedrock.amazonaws.com` events with the same `sourceIPAddress` and
`userIdentity.arn` but bursty `Converse` patterns are the giveaway.

## Cost gotcha

Cross-region inference profiles bill at the destination region's rate. If the
relay forces all traffic through `us-east-1` while the inference profile would
normally route to `eu-west-1`, you may pay slightly more per token. Compare
the per-region pricing in the Bedrock console before committing to a single
relay region.
