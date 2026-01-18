---
name: cli
description: CLI development patterns with Click, Typer, and argparse. Use when creating command-line tools, argument parsing, or interactive scripts. Triggers on CLI, command-line, Click, argparse, terminal keywords.
---

# CLI Development Patterns

## Click (Recommended)

### Basic Command

```python
import click

@click.command()
@click.argument("name")
@click.option("--count", "-c", default=1, help="Number of greetings")
@click.option("--shout", is_flag=True, help="Shout the greeting")
def hello(name: str, count: int, shout: bool):
    """Greet NAME."""
    greeting = f"Hello, {name}!"
    if shout:
        greeting = greeting.upper()
    for _ in range(count):
        click.echo(greeting)

if __name__ == "__main__":
    hello()
```

### Command Groups

```python
@click.group()
@click.option("--debug/--no-debug", default=False)
@click.pass_context
def cli(ctx, debug):
    ctx.ensure_object(dict)
    ctx.obj["debug"] = debug

@cli.command()
@click.argument("src", type=click.Path(exists=True))
@click.argument("dst", type=click.Path())
@click.pass_context
def copy(ctx, src, dst):
    """Copy SRC to DST."""
    if ctx.obj["debug"]:
        click.echo(f"Debug: copying {src} to {dst}")
    # ...

@cli.command()
@click.option("--force", "-f", is_flag=True)
def clean(force):
    """Clean temporary files."""
    if not force:
        click.confirm("Delete all temp files?", abort=True)
    # ...
```

### Option Types

```python
# File paths
@click.option("--input", type=click.Path(exists=True, path_type=Path))
@click.option("--output", type=click.Path(path_type=Path))

# Choices
@click.option("--format", type=click.Choice(["json", "csv", "xml"]))

# Multiple values
@click.option("--tag", "-t", multiple=True)

# Key-value pairs
@click.option("--env", "-e", multiple=True, type=(str, str))

# File handles
@click.option("--input", type=click.File("r"), default="-")
@click.option("--output", type=click.File("w"), default="-")

# Integer ranges
@click.option("--port", type=click.IntRange(1, 65535))

# Required options
@click.option("--api-key", required=True, envvar="API_KEY")
```

### Progress & Output

```python
import click

# Simple progress
with click.progressbar(items, label="Processing") as bar:
    for item in bar:
        process(item)

# Styled echo
click.secho("Success!", fg="green", bold=True)
click.secho("Warning!", fg="yellow")
click.secho("Error!", fg="red", err=True)

# Pager for long output
click.echo_via_pager(long_text)

# Clear screen
click.clear()
```

### Prompts

```python
# Simple input
name = click.prompt("Your name")

# With default
port = click.prompt("Port", default=8080, type=int)

# Password (hidden)
password = click.prompt("Password", hide_input=True)

# Confirmation
if click.confirm("Continue?"):
    ...

# Confirmation with abort
click.confirm("Delete all files?", abort=True)

# Choice
choice = click.prompt(
    "Select option",
    type=click.Choice(["a", "b", "c"])
)
```

## Typer (Type-Hint Based)

```python
import typer
from pathlib import Path
from typing import Annotated, Optional

app = typer.Typer(help="My CLI tool")

@app.command()
def process(
    input_file: Annotated[Path, typer.Argument(help="Input file")],
    output: Annotated[Path, typer.Option("--output", "-o")] = Path("out.csv"),
    verbose: Annotated[bool, typer.Option("--verbose", "-v")] = False,
):
    """Process the INPUT_FILE."""
    if verbose:
        typer.echo(f"Processing {input_file}")
    ...
    typer.secho(f"✓ Done: {output}", fg=typer.colors.GREEN)

@app.command()
def serve(
    port: Annotated[int, typer.Option(envvar="PORT")] = 8000,
):
    """Start the server."""
    typer.echo(f"Starting on port {port}")

if __name__ == "__main__":
    app()
```

## argparse (Standard Library)

```python
import argparse
from pathlib import Path

def main():
    parser = argparse.ArgumentParser(
        description="Process files",
        formatter_class=argparse.RawDescriptionHelpFormatter
    )
    
    parser.add_argument("input", type=Path, help="Input file")
    parser.add_argument("-o", "--output", type=Path, default=Path("output.csv"))
    parser.add_argument("-v", "--verbose", action="store_true")
    parser.add_argument("--format", choices=["csv", "json"], default="csv")
    
    args = parser.parse_args()
    
    if args.verbose:
        print(f"Processing {args.input}")
    
    # Use args.input, args.output, args.format

if __name__ == "__main__":
    main()
```

## Interactive Mode

```python
import click

def interactive_mode():
    """Run in interactive mode."""
    click.echo("Interactive mode. Type 'help' for commands, 'quit' to exit.")
    
    while True:
        try:
            cmd = click.prompt(">>>", prompt_suffix=" ")
        except (EOFError, KeyboardInterrupt):
            break
        
        cmd = cmd.strip().lower()
        
        match cmd:
            case "help":
                click.echo("Commands: help, status, quit")
            case "status":
                show_status()
            case "quit" | "exit":
                break
            case "":
                continue
            case _:
                click.echo(f"Unknown command: {cmd}")
    
    click.echo("Goodbye!")
```

## Configuration Loading

```python
import click
import tomllib
from pathlib import Path

def load_config(config_path: Path | None) -> dict:
    """Load config from file or defaults."""
    defaults = {"verbose": False, "output_dir": "./output"}
    
    if config_path and config_path.exists():
        with config_path.open("rb") as f:
            user_config = tomllib.load(f)
        return {**defaults, **user_config}
    
    return defaults

@click.command()
@click.option(
    "--config", "-c",
    type=click.Path(exists=True, path_type=Path),
    help="Config file path"
)
@click.pass_context
def cli(ctx, config):
    ctx.ensure_object(dict)
    ctx.obj["config"] = load_config(config)
```
