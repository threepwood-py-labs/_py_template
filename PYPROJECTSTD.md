# Python Project Standardization Prompt

Copy-paste this prompt when setting up or auditing any Python project.
Replace `{PROJECT_NAME}`, `{DESCRIPTION}`, `{PACKAGE_NAME}` as needed.

---

## THE PROMPT

````text
You are setting up a Python project called {PROJECT_NAME} to a strict, consistent standard.
Apply all of the following requirements exactly.
The goal is that every project follows the same Copier-managed template so updates can be pushed everywhere.

REQUIREMENTS OVERVIEW
- Python minimum: 3.13
- Project layout: src/
- Build/backend: hatchling
- Orchestration: Hatch
- Installer/lock backend: uv
- Lockfile: uv.lock committed to git
- Lint/format: Ruff
- Naming style enforcement: Ruff pep8-naming (`N` rules)
- Naming conventions: kebab-case `project.name`, snake_case package directory under `src/`
- Type checking: basedpyright strict
- Testing: pytest + pytest-cov (+ pytest-qt for `qt_app` outputs)
- Test helper deps in Hatch envs: PySide6>=6.10.2 for `qt_app`; repo-specific extras only when needed
- Git hooks: pre-commit (including local policy)
- CI: GitHub Actions on windows-latest
- CI trigger branches: main and master
- Template sync: Copier
- Qt architecture decomposition guidance for large UI classes (playbook-driven, warning-level policy)
- Python pinning: .python-version
- LF line endings for tracked text files
- No setup.py/setup.cfg/requirements*.txt
- No [project.scripts]
- Windows script policy: Python-only runners under scripts/windows/ (`.py` + `.pyw`, no .cmd or .ps1)
- Launch contract: python -m {PACKAGE_NAME} via src/{PACKAGE_NAME}/__main__.py
- If package has src/{PACKAGE_NAME}/main.py, migrate it into __main__.py and remove main.py
- README legal policy: enforce standardized marker-wrapped `## Legal Disclaimer` block
- TOML app config contract (when applicable):
  - tracked config/app.defaults.toml
  - tracked config/app.example.toml
  - untracked config/app.local.toml
  - reject legacy root config names (config.toml, *_config.toml, *_config_example.toml, *_config_totemp.toml)
- QSettings runtime contract for Qt/PySide apps (when applicable):
  - use `QSettings(IniFormat, UserScope, "ThreepSoftwz", "<package_name>")`
  - default INI path (Windows): `%APPDATA%\ThreepSoftwz\<package_name>.ini`
  - default non-INI runtime data root (Windows): `%LOCALAPPDATA%\ThreepSoftwz\<package_name>\`
  - OV01 overrides:
    - `CONFIG_DIR` => `<CONFIG_DIR>\<package_name>.ini`
    - `DATA_DIR` => `<DATA_DIR>\<package_name>\...`
  - if CLI parser exists, expose `--config-dir` and `--data-dir`
  - override precedence: CLI > env > default

DIRECTORY STRUCTURE
{PROJECT_NAME}/
|- .github/workflows/ci.yml
|- config/                                  # only if app uses TOML runtime config
|  |- app.defaults.toml                     # tracked
|  |- app.example.toml                      # tracked
|  |- app.local.toml                        # local-only, untracked
|- src/{PACKAGE_NAME}/
|  |- __init__.py
|  |- __main__.py
|  |- py.typed
|- tests/
|  |- __init__.py
|  |- test_{PACKAGE_NAME}.py
|- scripts/windows/
|  |- setup_env.py
|  |- run_app.py                         # qt_app only
|  |- run_app_gui.pyw                    # qt_app only
|  |- run_tests.py
|- scripts/policy/
|  |- check_standard.py
|- .editorconfig
|- .gitattributes
|- .gitignore
|- .pre-commit-config.yaml
|- .python-version
|- pyproject.toml
|- README.md
|- uv.lock
|- .copier-answers.yml

FILE REQUIREMENTS
1) .python-version must be exactly:
   3.13

2) pyproject.toml must include:
   - [tool.hatch.env] installer = "uv"
   - [project] requires-python = ">=3.13"
   - no [project.scripts]
   - [tool.hatch.envs.default.dependencies] includes:
     pytest>=8.0
     pytest-cov>=5.0
     pytest-qt>=4.4                      # qt_app only
     PySide6>=6.10.2                     # qt_app only
   - [tool.hatch.envs.default.scripts]:
     test = "pytest {args:tests}"
     test-cov = "pytest --cov=src/{PACKAGE_NAME} --cov-report=term-missing --cov-report=xml {args:tests}"
   - [tool.hatch.envs.test.dependencies] same dependency set as default
   - [tool.hatch.envs.lint.dependencies]:
     ruff>=0.6
     basedpyright>=1.18
   - [tool.hatch.envs.lint.scripts]:
     check = "ruff check ."
     fmt = "ruff format --check ."
     fix = ["ruff check --fix .", "ruff format ."]
     types = "basedpyright src/{PACKAGE_NAME}"
     policy = "python scripts/policy/check_standard.py"
     all = ["ruff check --fix .", "ruff format .", "basedpyright src/{PACKAGE_NAME}", "python scripts/policy/check_standard.py"]
   - [tool.ruff] target-version = "py313", src = ["src"]
   - [tool.ruff.lint.select] includes `N`
   - [tool.ruff.lint.pep8-naming.ignore-names] includes the canonical Qt override method allow-list
   - [tool.basedpyright] typeCheckingMode = "strict"
   - [tool.coverage.report] fail_under = 80

3) .pre-commit-config.yaml must include:
   - pre-commit-hooks: check-case-conflict, check-illegal-windows-names, mixed-line-ending (--fix=lf)
   - ruff + ruff-format
   - basedpyright
   - local hook running: python scripts/policy/check_standard.py

4) .gitattributes must include exactly this LF rule:
   * text=auto eol=lf

5) .gitignore must include untracked local config/secrets:
   - *.log
   - config/app.local.toml
   - config/secrets.toml
   - config/*.secrets.toml
   and local workspace artifacts:
   - .claude/
   - .tmp_localappdata/
   - .tmp_scan_root/
   - .tmp_wheel_venv/
   and must not include deprecated ignore entries:
   - w_ignore_prompt*.txt
   - .everything_sdk/

6) CI workflow must be Windows-first:
   - runs-on: windows-latest
   - branches: [main, master] for push and pull_request
   - blocking lint job running lint/fmt/types/policy
   - blocking test job running test-cov

7) scripts/windows/run_app.py (qt_app only):
   - resolves repo root from script location
   - checks hatch is on PATH
   - runs: hatch run python -m {PACKAGE_NAME} <args>

8) scripts/windows/setup_env.py:
   - ensures local `.venv` exists via:
     uv sync --locked
   - may retry with `uv sync` if lock sync fails
   - must verify `.venv\Scripts\pythonw.exe` exists at the end

9) scripts/windows/run_app_gui.pyw (qt_app only):
   - launches directly via local venv pythonw (no hatch process):
     .venv\Scripts\pythonw.exe -m {PACKAGE_NAME} <args>
   - if pythonw is missing, must attempt:
     python scripts\windows\setup_env.py
   - if still missing, fail with a clear user-visible error message

10) scripts/windows/run_tests.py:
   - same pattern, runs:
     hatch run test <args>

11) src/{PACKAGE_NAME}/__main__.py:
   - must be the canonical app entrypoint for python -m {PACKAGE_NAME}
   - eliminate references to python -m {PACKAGE_NAME}.main

12) TOML app config policy (conditional):
   - if any TOML app config files are present, require both tracked:
     config/app.defaults.toml
     config/app.example.toml
   - never track config/app.local.toml
   - disallow legacy root config names:
     config.toml
     config_example.toml
     *_config.toml
     *_config_example.toml
     *_config_totemp.toml

12.1) QSettings policy for Qt/PySide projects (conditional):
   - use `QSettings(IniFormat, UserScope, "ThreepSoftwz", "<package_name>")`
   - default INI path on Windows: `%APPDATA%\ThreepSoftwz\<package_name>.ini`
   - default non-INI runtime data root on Windows: `%LOCALAPPDATA%\ThreepSoftwz\<package_name>\`
   - OV01 override contract:
     - `CONFIG_DIR` => `<CONFIG_DIR>\<package_name>.ini`
     - `DATA_DIR` => `<DATA_DIR>\<package_name>\...`
   - if CLI argument parsing already exists, expose:
     - `--config-dir`
     - `--data-dir`
   - override precedence: CLI > env > default

13) README legal disclaimer policy:
   - README.md must be tracked
   - include markers exactly:
     <!-- legal-disclaimer:start -->
     <!-- legal-disclaimer:end -->
   - content between markers must match canonical text in:
     _py_template/legal_disclaimer.md

COPIER TEMPLATE CONTRACT
- Template source repo directory in this workspace: _py_template
- In this workspace, sibling repos are Copier-managed targets and must be updated from _py_template (not treated as template source).
- copier.yml must include:
  - _answers_file: .copier-answers.yml
  - _subdirectory: template
  - _templates_suffix: .jinja
  - _skip_if_exists:
    - README.md
    - src/*/__init__.py
    - src/*/__main__.py
  - _exclude:
    - python-project-standard-prompt.md
- Copier defaults:
  - author_name: MT0z
  - author_email: mt0z1@pir0z.yxz
  - github_username: threepwood-py-labs
  - project_version: 0.1.0
  - runtime_dependencies: []

POST-SCAFFOLD COMMANDS (PowerShell)
uv tool install hatch
uv tool install copier
uv tool install pre-commit
uv lock
hatch env create
hatch run test
pre-commit install
pre-commit run --all-files
hatch run lint:all

COPIER ROLLOUT COMMANDS (PowerShell)
# Apply template to a repo (initial migration)
uvx copier copy --trust --force --vcs-ref <TEMPLATE_TAG> <PATH_TO_PY_TEMPLATE> <PATH_TO_PROJECT> --data-file <PROJECT_DATA_YAML>

# Update an already templated repo (run from target repo root)
uvx copier update --trust --defaults --vcs-ref <TEMPLATE_TAG>

# For sibling repos in this workspace, keep _src_path relative:
# ../_py_template

RULES TO ENFORCE
1. Commit uv.lock and .copier-answers.yml.
2. Keep exactly one package directory under src/.
3. Keep required tracked root files: .python-version, .editorconfig, .gitattributes, .pre-commit-config.yaml, pyproject.toml, README.md.
4. Keep required tracked Windows runners:
   - always: scripts/windows/setup_env.py, scripts/windows/run_tests.py
   - qt_app only: scripts/windows/run_app.py, scripts/windows/run_app_gui.pyw
5. Keep package root files: src/{PACKAGE_NAME}/__init__.py, src/{PACKAGE_NAME}/__main__.py, src/{PACKAGE_NAME}/py.typed.
6. Keep tests/__init__.py and use test filenames test_*.py under tests/.
7. Keep src module filenames lowercase snake_case (*.py).
8. Do not add mypy/pyright; basedpyright strict is the only type checker.
9. Do not use .cmd or .ps1 launch/test wrappers; use Python runners under scripts/windows/.
10. Remove setup.py, setup.cfg, requirements.txt, requirements-dev.txt, requirements.lock, requirements-dev.lock if present.
11. Keep existing README content during migrations unless explicitly requested otherwise.
12. Do not reintroduce [project.scripts].
13. Ensure python -m {PACKAGE_NAME} works in every repo and remove all references to python -m {PACKAGE_NAME}.main.
14. Enforce LF text endings for tracked text files and keep .gitattributes rule `* text=auto eol=lf`.
15. If TOML app config is used, enforce canonical config/app.defaults.toml + config/app.example.toml and keep config/app.local.toml untracked.
16. Do not track legacy root config filenames (config.toml, *_config.toml, *_config_example.toml, *_config_totemp.toml).
17. For Qt/PySide projects using QSettings, enforce OV01:
    - org/app naming: `ThreepSoftwz` + `<package_name>`
    - defaults: `%APPDATA%\ThreepSoftwz\<package_name>.ini` and `%LOCALAPPDATA%\ThreepSoftwz\<package_name>\...`
    - overrides: `CONFIG_DIR`, `DATA_DIR`; CLI flags where parser already exists
    - precedence: CLI > env > default
18. Enforce policy checks via scripts/policy/check_standard.py in CI and pre-commit.
19. Enforce README legal disclaimer markers and exact canonical content from _py_template/legal_disclaimer.md.
20. Do not reintroduce deprecated template ignore entries: w_ignore_prompt*.txt and .everything_sdk/.
21. Keep oversized class/function checks as warning-level architecture guidance (non-blocking).

VALIDATION CHECKLIST
- hatch run lint:check
- hatch run lint:fmt
- hatch run lint:types
- hatch run lint:policy
- hatch run test
- hatch run test-cov
- pre-commit run --all-files
- uv lock
- hatch build
- python scripts\windows\run_tests.py (Windows smoke check)
- python scripts\windows\setup_env.py (Windows env bootstrap)
- python scripts\windows\run_app.py (Windows smoke check, qt_app only)
- pyw scripts\windows\run_app_gui.pyw (Windows smoke check, qt_app only)
````

---

## Quick Reference

| Task | Command |
|---|---|
| Run tests | `hatch run test` |
| Coverage | `hatch run test-cov` |
| Lint check | `hatch run lint:check` |
| Format check | `hatch run lint:fmt` |
| Fix lint/format | `hatch run lint:fix` |
| Type check | `hatch run lint:types` |
| Policy check | `hatch run lint:policy` |
| Run all lint+types | `hatch run lint:all` |
| Build | `hatch build` |
| Refresh lock | `uv lock --upgrade` |
| Update from template | `uvx copier update --trust --defaults --vcs-ref <tag>` |

For sibling repos in this workspace, run the update from the target repo root and keep `_src_path: "../_py_template"` in `.copier-answers.yml` so no absolute local path is committed.
