# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Commands

```bash
# Run tests
uvx --with tox-uv tox -e py

# Run a single test by name
uv run pytest -k test_dryrun

# Run linter (check only)
uvx --with tox-uv tox -e style

# Auto-fix linting/formatting
uvx --with tox-uv tox -e reformat

# Run type checking
uvx --with tox-uv tox -e typecheck
```

## Architecture

The entire implementation lives in a single file: `snip_stitch.py`. No external runtime dependencies.

**CLI** uses `argparse` with subcommands: `text`, `file`, `remove`. Content resolution happens in `main()` before anything reaches the class. Module-level globals `DRYRUN` and `VERBOSE` control output behavior.

**`SnipStitch`** is the core class. It takes a `tag`, `target_path`, and pre-resolved `snippet_lines` (list of strings), then:
1. Reads the existing target file
2. Parses it into three segments: `before_insertion`, `marked_contents`, `after_insertion`
3. Detects whether the snippet has changed (idempotency check)
4. Rewrites the file with the updated managed section wrapped in marker lines

**Marker format:**
```
# ---8<- {tag} -- {start_comment}
...snippet lines...
# -8<--- {tag} -- {end_comment}
```

## Testing

Tests use `runez`'s `cli` fixture (configured in `tests/conftest.py`) which wraps `argparse`-based CLIs for convenient invocation. `ClickRunner.default_main = main` sets `main()` as the entry point.

Sample files in `tests/samples/` are copied to a temp dir per test run via `runez.copy`.
