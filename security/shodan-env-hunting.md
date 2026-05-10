# Shodan CLI Filters for Exposed .env Files

```bash
shodan search --fields ip_str,port,org 'http.html:DB_PASSWORD ext:env'
shodan search 'http.title:"Index of" http.html:".env"'
```

The second query catches directory listings that expose .env files. Combine with `http.html:AWS_SECRET` for cloud cred hunting.

## Pro tip
Add `-C 200` to target only responsive hosts. Filter by `country:` to narrow scope.
