## Using `uv` and `pyproject.toml`
- Initialize a uv project inside an existing folder:
    - without git: `uv init --vcs none`
    - with git: `uv init`
    - writes: pyproject.toml, .python-version, README, main.py
    - A default python version is installed (e.g., 3.13), and specified in the pyproject.toml
        - `requires-python = ">=3.13"`
        - to change to another python version (e.g., to 3.12): 
            - manually change the constraint above `requires-python = ">=3.12"`, 
            - then: `uv python pin 3.12`
- Activate the environment
    - classic activation: `source .venv/bin/activate`
        - Explicit activation is mainly for interactive shells and IDEs.
    - preferred when running scripts: `uv run python train.py` -> syncs .venv to uv.lockn then runs
        - covers: `uv sync && source .venv/bin/activate && python yourscript.py`
        - is the idiomatic choice because it guarantees the env matches the lock before every run. 
- Modify the environment by adding/removing packages.
    - with every add/remove action, this chain of actions are performed by uv
        - `pyproject.toml` is updated to reflect the intent.
        - `uv.lock` is relocked. The new package resolves to the highest compatible version.
        - `.venv` is synced. This is an exact sync of the whole environment against the lock. Anything orphaned or extra gets removed.
        - This is the preferred way to change dependencies

    - Adding and removing packages:
        - add the latest package version: `uv add fastapi`
            - this will specify `fastapi>=<latest version>`
        - add a specific package version: `uv add uvicorn[standard]==0.50`
        - add a package version in a range: `uv add "langchain>=1.4, <1.5"`
        - remove a package: `uv remove pandas`

    Flags that stop the chain early
        Flag	    1. pyproject	2. lock	    3. sync
        (default)	✓	            ✓	        ✓
        --no-sync	✓	            ✓	        ✗
        --frozen	✓	            ✗	        ✗

- Modify the environment by hand-editing the `pyproject.toml`
    - After a hand-edit, run `uv sync` to validate the resolution and re-sync.
    - If `uv sync` is not performed, the net `uv run` will relock and re-sync.
    - Common scenarios for hand-editting:
        - Tightening or loosening bounds on existing deps, or bulk changes across many lines.
        - `requires-python`
        - Build and entry-point config: [build-system], [project.scripts], package metadata such as authors, license and classifiers.
        - Other tools' config: [tool.ruff], [tool.pytest.ini_options], [tool.mypy] and so on, which uv never touches.

- Inspect a package
    - `uv pip show fastapi`

## Summary for Development
- Step 1: init project `uv init`
- Step 2: hand-edit the pyproject.toml if needed, then `uv sync`
- Step 3: manual activation for development `source .venv/bin/activate`
- Step 4: add/remove packages `uv add ...`, `uv remove ...`
- Step 5: run program `uv run ...`

## Self-contained Backend Folder
- For a mono-repo where the backend is treated as an independent project from the rest of the repo, use the following commands:
    ```
    # from repo root
    uv init backend --no-workspace --vcs none   # add --app --package for a src/ layout
    cd backend
    uv python pin 3.12
    uv sync
    ```
    - Workspace isolation. --no-workspace stops uv init from joining a parent workspace.
- Reference project structure
    ```
    repo/
    ├── .git/
    ├── frontend/ ...
    └── backend/
        ├── .gitignore        ← its own: .venv/, __pycache__/, .pytest_cache/
        ├── .python-version
        ├── pyproject.toml    ← nearest config wins, so this shadows anything above
        ├── uv.lock
        ├── README.md
        └── src/backend/...
    ```