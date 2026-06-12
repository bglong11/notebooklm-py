# Repository Guidelines

**Status:** Active
**Last Updated:** 2026-03-13

## Project Structure & Module Organization

`src/notebooklm/` contains the async client and typed APIs. Internal feature modules use `_` prefixes such as `_sources.py` and `_artifacts.py`; `src/notebooklm/cli/` holds Click commands, and `src/notebooklm/rpc/` handles protocol encoding and decoding. Tests are split by scope: `tests/unit/`, `tests/integration/`, and `tests/e2e/`. Recorded HTTP fixtures live in `tests/cassettes/`. Examples are in `docs/examples/`, and diagnostics live in `scripts/`.

## Build, Test, and Development Commands

Use system `/usr/bin/python3.12` for local work:

```bash
pip install --break-system-packages -e ".[dev,browser]"
/usr/bin/python3.12 -m pytest
/usr/bin/python3.12 -m ruff check src/ tests/
/usr/bin/python3.12 -m ruff format src/ tests/
/usr/bin/python3.12 -m mypy src/notebooklm
/usr/bin/python3.12 -m pre_commit run --all-files
```

Run `/usr/bin/python3.12 -m pytest tests/e2e -m readonly` only after `notebooklm login` and setting test notebook env vars.

## Coding Style & Naming Conventions

Target Python 3.10+, 4-space indentation, and double quotes. Ruff enforces formatting and import order with a 100-character line length. Keep module and test file names in `snake_case`; prefer descriptive Click command names that match existing groups such as `source`, `artifact`, and `research`. Preserve the internal/public split: `_*.py` for implementation, exported types in `src/notebooklm/__init__.py`.

## Testing Guidelines

Put pure logic in `tests/unit/`, VCR-backed flows in `tests/integration/`, and authenticated NotebookLM coverage in `tests/e2e/`. Name tests `test_<behavior>.py` and record cassettes with `NOTEBOOKLM_VCR_RECORD=1 /usr/bin/python3.12 -m pytest tests/integration/test_vcr_*.py -v`. Coverage is expected to stay at or above the configured 90% threshold.

## Commit, PR, and Agent Notes

Follow the existing commit style: `feat(cli): ...`, `fix(cli): ...`, `refactor(test): ...`, `style: ...`. PRs should include a short summary, linked issue when relevant, and the commands run locally. For Codex or other parallel agents, prefer `--json`, pass explicit notebook IDs instead of relying on `notebooklm use`, and isolate runs with `NOTEBOOKLM_HOME=/tmp/<agent-id>` when multiple agents share one machine.
