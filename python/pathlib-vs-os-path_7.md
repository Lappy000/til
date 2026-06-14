# pathlib vs os.path

```python
from pathlib import Path

# Old way
import os
os.path.join('/tmp', 'file.txt')
os.path.exists('/tmp/file.txt')
os.path.basename('/tmp/file.txt')

# New way (cleaner)
p = Path('/tmp') / 'file.txt'
p.exists()
p.name
p.suffix     # '.txt'
p.stem       # 'file'
p.parent     # Path('/tmp')

# Globbing
list(Path('.').glob('**/*.py'))

# Read/write text
Path('file.txt').read_text()
Path('file.txt').write_text('content')

# Create dirs
Path('a/b/c').mkdir(parents=True, exist_ok=True)
```
