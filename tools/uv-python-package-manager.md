# uv - Fast Python Package Manager

Written in Rust. 10-100x faster than pip. Drop-in compatible.

## Commands

```bash
uv pip install requests     # replaces pip install
uv venv .venv              # replaces python -m venv
uv pip sync requirements.txt  # replaces pip install -r
uv pip compile requirements.in -o requirements.txt
uvx ruff check .           # replaces pipx run
uv init my-project         # replaces poetry new
uv add requests            # replaces poetry add
uv python install 3.12     # version management built in
```

## CI
```yaml
- uses: astral-sh/setup-uv@v3
- run: uv sync --frozen
- run: uv run pytest
```

Single binary, no Python needed to bootstrap. Deterministic resolution.
