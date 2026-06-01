# GitHub CLI GraphQL Batch Operations

Using `gh api graphql` for bulk repository and user operations.

## Get All Repos for a User

```bash
gh api graphql -f query='
query {
  user(login: "Lappy000") {
    repositories(first: 100, orderBy: {field: UPDATED_AT, direction: DESC}) {
      nodes { name description updatedAt stargazerCount }
      pageInfo { hasNextPage endCursor }
    }
  }
}'
```

## Paginated Query Helper

```python
import subprocess, json

def graphql_paginate(query, path, page_size=100):
    results = []
    cursor = None
    while True:
        q = query.replace("$cursor", f'"{cursor}"' if cursor else "null")
        r = subprocess.run(
            ["gh", "api", "graphql", "-f", f"query={q}"],
            capture_output=True, text=True
        )
        data = json.loads(r.stdout)
        node = data["data"]
        for p in path.split("."):
            node = node[p]
        results.extend(node["nodes"])
        if not node["pageInfo"]["hasNextPage"]:
            break
        cursor = node["pageInfo"]["endCursor"]
    return results

repos = graphql_paginate('''
query {
  user(login: "Lappy000") {
    repositories(first: 100, after: $cursor) {
      nodes { name }
      pageInfo { hasNextPage endCursor }
    }
  }
}''', "user.repositories")
```

## Useful Queries

```bash
# List all PRs across repos
gh search prs --author=@me --state=all --limit=100

# Bulk update topics
gh repo edit Lappy000/repo --add-topic security,pentest
```
