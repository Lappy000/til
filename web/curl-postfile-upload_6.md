# curl File Upload Techniques

```bash
# Simple file upload
curl -X POST https://api.example.com/upload \
  -F 'file=@/path/to/file.txt'

# With additional fields
curl -X POST https://api.example.com/upload \
  -F 'file=@photo.jpg' \
  -F 'title=My Photo' \
  -F 'category=nature'

# Multipart with JSON
curl -X POST https://api.example.com/upload \
  -H 'Content-Type: multipart/form-data' \
  -F 'file=@data.json' \
  -F 'metadata={"type":"json"};type=application/json'

# Download with progress
curl -O https://example.com/largefile.zip --progress-bar
```
