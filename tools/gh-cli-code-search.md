# gh CLI - Search Code Across All Repos

```bash
# Find leaked API keys in your own repos
gh search code 'AKIA' --owner Lappy000

# Search all public repos for a pattern
gh search code 'password' --filename .env --language python
```

The `--filename` and `--language` filters cut noise dramatically.
