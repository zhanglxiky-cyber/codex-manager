# AGENTS Guide for codex-manager

Working contract for agentic coding tools in this repository.
Use it for planning, editing, testing, and preparing reviewable diffs.

## 1) Project Snapshot

- Runtime: Python 3.10+ backend and vanilla browser JavaScript.
- Web stack: FastAPI, Uvicorn, Jinja2, SQLAlchemy.
- Tests: `pytest` for Python and Node built-in `node:test` for `.cjs` frontend tests.
- Packaging: PyInstaller via `build.sh`, `build.bat`, and GitHub Actions.
- Dependencies: `uv` preferred, `pip` supported.

## 2) Cursor / Copilot Rule Sources

- `.cursor/rules/`: not present.
- `.cursorrules`: not present.
- `.github/copilot-instructions.md`: not present.
- Result: this `AGENTS.md` is the primary instruction source for coding agents.

## 3) Setup / Run / Build Commands

### Install dependencies

```bash
# preferred
uv sync

# fallback
pip install -r requirements.txt
```

### Run web UI locally

```bash
python webui.py
python webui.py --debug
python webui.py --host 0.0.0.0 --port 15555
```

### Build executable

```bash
# macOS/Linux
bash build.sh

# Windows
build.bat
```

### Docker (optional)

```bash
docker-compose up -d
docker-compose logs -f
```

## 4) Test Commands (single-test first)

### Python (`pytest`)

```bash
# full suite
python -m pytest tests

# one file
python -m pytest tests/test_newapi_service_routes.py

# one test function
python -m pytest tests/test_newapi_service_routes.py::test_create_newapi_service_rejects_non_ascii_api_key

# keyword filter
python -m pytest tests -k "newapi_service_routes"
```

### Frontend Node tests (`node:test`)

```bash
# one file
node --test tests/test_single_task_websocket_status.cjs

# multiple .cjs files
node --test tests/*.cjs

# one named test case
node --test --test-name-pattern "single task websocket completion" tests/test_single_task_websocket_status.cjs
```

### Focused workflow rule

- Start with the narrowest relevant test target.
- Expand to broader suites only after focused tests pass.

## 5) Lint / Formatting Expectations

- No strict formatter/linter is enforced by project config.
- `pyproject.toml` currently does not define `ruff`, `black`, `isort`, `mypy`, or pytest config blocks.
- Keep edits style-consistent with nearby code; avoid repository-wide formatting churn.
- Optional local lint check (if installed):

```bash
python -m ruff check src tests
```

## 6) Python Code Style Guidelines

### Imports

- Order imports as: stdlib, third-party, local (`src...`).
- Separate groups with one blank line.
- Prefer explicit imports; avoid wildcard imports.

### Formatting

- 4-space indentation, no tabs.
- Keep functions focused; extract helper functions for repeated logic.
- Prefer readable multi-line calls over compact one-liners.
- Preserve existing quote style in touched files.

### Types and models

- Add type hints to public functions and complex internals.
- Prefer existing local typing style (`Optional`, `Dict[str, Any]`, `List[...]`, etc.).
- Use Pydantic `BaseModel` for API request/response schemas.
- Use dataclasses (including `@dataclass(frozen=True)`) for structured immutable state where appropriate.

### Naming

- `snake_case`: variables/functions.
- `PascalCase`: classes.
- `UPPER_SNAKE_CASE`: constants.
- Keep route/helper names domain-oriented and explicit.

### Error handling and logs

- API-layer validation failures should raise `HTTPException` with clear `detail`.
- Domain/service failures should use typed exceptions (for example `EmailServiceError` family).
- Write actionable error messages; avoid vague failures.
- Catch narrowly, log context, then re-raise or translate at layer boundaries.
- Use module loggers: `logger = logging.getLogger(__name__)`.

### DB sessions and async

- Use DB context managers (`with get_db() as db:` / `session_scope()`).
- Keep transaction scope tight and explicit.
- Do not hold long-lived sessions across unrelated operations.
- Respect existing async patterns in FastAPI/task manager code.
- Avoid blocking work in async routes unless deliberately delegated.

## 7) Frontend JavaScript Guidelines

- Follow existing style in `static/js/*.js`.
- Use `const`/`let` in new code (avoid `var`).
- Keep semicolons and single quotes consistent with existing files.
- Centralize DOM access through shared `elements` maps when practical.
- Reuse shared helpers from `static/js/utils.js` (`api`, `toast`, etc.).
- Keep UI state transitions explicit (disable/enable controls, status text, fallback polling).

## 8) Agent Change Discipline Checklist

- Run focused tests for the changed area before broader runs.
- Backend edits: run at least one targeted `pytest` path.
- Websocket/status/frontend behavior edits: run targeted `.cjs` Node tests when applicable.
- Keep diffs minimal and scoped; avoid speculative refactors.
- Do not change unrelated files for style-only reasons.
- Ensure commands in notes/reviews are reproducible from repository root.
