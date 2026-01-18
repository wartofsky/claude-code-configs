# Python Scripts Project

Automation scripts and CLI tools using modern Python.

## Tech Stack

- **Python**: 3.12+
- **CLI**: Click / Typer
- **Output**: Rich
- **HTTP**: httpx
- **Config**: pydantic-settings
- **Data**: pandas (if needed)

## Project Structure

```
scripts/
├── script_name/
│   ├── __init__.py
│   ├── main.py           # Entry point / CLI
│   ├── core.py           # Business logic
│   └── utils.py          # Helpers
├── pyproject.toml
├── requirements.txt
└── README.md
```

Or simple:
```
scripts/
├── process_data.py
├── sync_files.py
└── requirements.txt
```

## Conventions

### Code Style
- Type hints on all functions
- Docstrings for public functions
- f-strings for formatting
- Pathlib for file paths

### CLI
- Use Click or Typer
- Always provide `--help`
- Support `--verbose` flag
- Color output (green=success, red=error, yellow=warning)

### Error Handling
- Exit code 0 for success
- Exit code 1 for errors
- Catch `KeyboardInterrupt` gracefully

## Commands

```bash
# Run script
python script_name/main.py --help
python script_name/main.py process input.csv -o output.csv

# Install as CLI (with pyproject.toml)
pip install -e .
my-cli --help

# Development
pip install -r requirements.txt
python -m pytest
ruff check .
```

## pyproject.toml Template

```toml
[project]
name = "my-scripts"
version = "0.1.0"
requires-python = ">=3.11"
dependencies = [
    "click>=8.0",
    "rich>=13.0",
    "httpx>=0.25",
    "pydantic-settings>=2.0",
]

[project.scripts]
my-cli = "my_scripts.main:cli"

[project.optional-dependencies]
dev = ["pytest", "ruff"]
```

## Environment

```bash
# .env
API_KEY=secret
DEBUG=true
OUTPUT_DIR=./output
```

## Patterns

### Script Entry Point
```python
def main():
    try:
        run()
    except KeyboardInterrupt:
        print("\nInterrupted")
        sys.exit(130)
    except Exception as e:
        print(f"Error: {e}", file=sys.stderr)
        sys.exit(1)

if __name__ == "__main__":
    main()
```

### Progress Bar
```python
from rich.progress import track
for item in track(items, description="Processing"):
    process(item)
```

## Notes

<!-- Project-specific notes -->
