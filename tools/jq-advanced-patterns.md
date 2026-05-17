# Advanced jq Patterns

## Group and count
```bash
cat access.log.json | jq -s 'group_by(.status) | map({status: .[0].status, count: length})'
```

## Recursive descent
```bash
echo "$json" | jq '.. | .id? // empty'
```

## Update nested fields
```bash
jq '.products[].price |= . * 1.1' catalog.json
```

## Multi-file merge
```bash
jq -s '.[0] * .[1]' defaults.json overrides.json
```

## GitHub API with jq
```bash
gh api repos/org/repo/issues --paginate --jq '
  .[] | select(.labels | map(.name) | contains(["bug"])) |
  {number, title, assignee: .assignee.login}
'
```
