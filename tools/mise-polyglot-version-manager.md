# mise - Unified Runtime Version Manager

Replaces nvm, pyenv, rbenv, goenv with one tool.

## Usage

```bash
mise install node@20 python@3.12 go@1.22
mise use node@20          # per-project
mise use --global node@20 # global

# .mise.toml
[tools]
node = "20"
python = "3.12"

[tasks.test]
run = "pytest tests/"
```

## vs Alternatives

| Feature | mise | asdf | nvm/pyenv |
|---------|------|------|-----------|
| Speed | Fast (Rust) | Slow (bash) | Varies |
| Languages | All | All (plugins) | Single |
| Task runner | Built-in | No | No |
