## Reference Structure

For a mono-repo with frontend and backend folder, reference structure for backend agent:
```
backend/
├── src/my_agent/
│   ├── core/
│   │   ├── graph.py        # agent/graph construction (build_agent(llm, tools, checkpointer))
│   │   ├── state.py        # state schema (TypedDict / pydantic)
│   │   ├── ports.py        # Protocols: LLMClient, MemoryStore, Retriever
│   │   └── prompts/        # versioned prompt templates (as data, not f-strings in code)
│   ├── tools/              # pure, deterministic capabilities (mortgage math) — no LLM imports
│   ├── adapters/
│   │   ├── llm.py          # model client(s)
│   │   ├── memory.py       # checkpointer / Lakebase
│   │   └── tracing.py      # MLflow / observability hooks
│   ├── utilities/          # formatting, parsing
│   └── settings.py         # pydantic-settings, reads env
├── app/
│   ├── main.py             # app factory = composition root (wires adapters → core)
│   ├── routes.py           # /invoke, /stream (SSE), /health
│   └── schemas.py          # request/response models = frontend contract
├── tests/
│   ├── unit/               # tools, utilities, graph with fake LLM
│   └── integration/        # real adapters, gated by env/markers
├── evals/                  # LLM-quality evaluation datasets + scorers
├── scripts/                # notebooks, backfills, one-offs
├── config/
├── .env.example
└── pyproject.toml
```

## Installing your own packages (editable mode)
- add the build system and its config inside `pyproject.toml`. A build system is the tool that turns your project folder into an installable package.
- run `uv sync` (during development)
- Example:
    - The repo structure
        ```
        .
        ├── backend/
        │   ├── src
        │   ├── app
        │   ├── tests
        │   └── pyproject.toml
        ├── frontend
        ├── README.md
        └── LICENSE
        ```
    - The build system and config inside `pyproject.toml`
        ```
        [build-system]
        requires = ["setuptools>=61"]
        build-backend = "setuptools.build_meta"

        [tool.setuptools.packages.find]
        where = ["src"]
        ```
- Namespace packages are on by default. In `pyproject.toml`, `packages.find` defaults to `namespaces = true`. That's why folders inside `src/` get picked up as packages without `__init__.py` files. With the old `setup.py` approach this would not have worked.

## Tests
- Recommended `pyproject.toml` (NO `pythonpath = ["src"]`):
    ```
    [tool.pytest.ini_options]
    testpaths = ["tests"]
    ```
- `[tool.pytest.ini_options]` is where pytest reads its settings when `pyproject.toml` is used.
    - Other options for pytest settings: `pytest.ini`, `tox.ini` or `setup.cfg`.
- The directory containing that file becomes the rootdir. In the structure above, the `backend/` folder is the rootdir.
- How discovery of test files works:
    - Starting from `testpaths` (or the current directory if unset), pytest walks the directories recursively.
    - It collects files matching the naming convention of `test_*.py` or `*_test.py`.
    - In each file, it collects functions named `test_*`, and methods named `test_*` in classes named `Test*` (those classes must have no __init__).
    - Thus, specifying `testpaths = ["tests"]` restricts the search space, and prevent unexpected behaviors when other non-test files having the names matching the pytest naming convention above get treated as test files.
- How imports inside the `test_*.py` files are resolved:
    - Preferred: by relying on the project dependencies installed inside the `.venv`
    - Works but not recommended: by using `pythonpath = ["src"]` inside `pyproject.toml`, which adds backend/src to sys.path for the run.
- By not using `pythonpath = ["src"]` inside `pyproject.toml`, there is only 1 single way to resolve imports everywhere - inside your packages, inside tests... If the tests fail importing, your app will fail, and there is no silent failure.
        