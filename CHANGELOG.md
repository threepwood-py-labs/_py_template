# Changelog

## v1.7.5 - 2026-03-21
- Bumped generated `threep-commons` qt-app dependency pins from `v0.3.0` to `v0.3.1`.
- Fixed generated Windows packaging helper formatting so fresh renders stay Ruff-clean.

## v1.7.4 - 2026-03-13
- Fixed generated Windows helper scripts to satisfy Ruff line-length checks in fresh renders.
- Restored generated `.copier-answers.yml` output so policy checks can infer `project_kind` correctly.
- Added a Copier post-render/update `uv lock` task so fresh generated projects include `uv.lock` and pass policy checks without a manual lock step.

## v1.7.3 - 2026-03-11
- Bumped generated `threep-commons` app dependency pins from `v0.1.0` to `v0.2.0` for the Windows storage commons rollout.

## v1.7.1 - 2026-03-07
- Enabled `tool.hatch.metadata.allow-direct-references = true` in generated projects so app-mode repos can install pinned Git URL dependencies.

## v1.7.0 - 2026-03-07
- Added Copier `project_kind` mode support with `qt_app` and `shared_lib`.
- Removed generated `runtime_paths.py` and `utils/logging_config.py` wrappers from app outputs.
- Added direct `threep-commons` dependency wiring for `qt_app` outputs.
- Updated generated app constants to expose `APP_IDENTITY`, `SETTINGS_ORG_NAME`, and `SETTINGS_APP_NAME`.
- Made policy checks mode-aware for shared-library outputs and removed wrapper-file requirements.
- Removed hardcoded `qbittorrent-api` from standardized Hatch test/default dependency lists.

## v1.6.4 - 2026-03-07
- Expanded template policy checks:
  - reject UTF-8 BOM in tracked text files
  - enforce `src/<package>/__main__.py` contract:
    - `from __future__ import annotations`
    - `if __name__ == "__main__":`
    - `raise SystemExit(main())`
  - reject module/package name collisions such as `utils.py` with `utils/__init__.py`.

## v1.6.3 - 2026-03-07
- Fixed `runtime_paths.py` scaffold regression in v1.6.1/v1.6.2:
  - restored `configure_qsettings`
  - restored `resolve_config_root`, `resolve_data_root`, and `resolve_app_data_dir`
  - kept `get_config_dir`, `get_data_dir`, `get_log_dir`, and `get_settings_ini_path`
    as compatibility wrappers.

## v1.6.1 - 2026-03-06
- Switched template to module-first entrypoint policy:
  - removed required `src/<package>/main.py` scaffold
  - `src/<package>/__main__.py` now contains a self-contained `main()` shim
  - restored `src/<package>/__init__.py` exports to version-oriented module defaults.
- Added baseline module scaffolding for generated projects:
  - `src/<package>/constants.py`
  - `src/<package>/runtime_paths.py`
  - `src/<package>/utils/__init__.py`
  - `src/<package>/utils/logging_config.py`.
- Added required pytest structure for generated projects:
  - `tests/conftest.py` with shared `window` fixture
  - package dirs: `tests/unit/`, `tests/integration/`, `tests/gui/` with `__init__.py`.
- Expanded policy checker to enforce new baseline scaffold files and provide non-blocking guidance for:
  - missing public docstrings
  - silent `except Exception: pass` patterns.
- Removed hardcoded `deptry` `package_module_name_map` from template `pyproject.toml`.
- Replaced personal Copier defaults with neutral `author_name` / `author_email` defaults.
- Added rollout documentation: `COPIER_ROLLOUT_RUNBOOK.md`.
- Added `aatemplate` agent note clarifying module-first entrypoint policy to prevent `main.py` confusion.

## v1.5.7 - 2026-03-06
- Added `*.log` to standardized `.gitignore` rules for template and project repos.
- Ensures runtime log files remain untracked by default across all managed projects.

## v1.5.6 - 2026-03-06
- Added Windows companion bootstrap script: `scripts/windows/setup_env.py`.
- `run_app_gui.pyw` now attempts env bootstrap through `setup_env.py` when `.venv` is missing.
- Removed direct `windll.user32` usage from GUI launcher error reporting.
- Updated template policy/docs to require and document `setup_env.py`.

## v1.5.5 - 2026-03-06
- Updated `run_app_gui.pyw` launcher behavior to avoid invoking `hatch` at runtime.
- GUI launcher now starts the app directly via `.venv\\Scripts\\pythonw.exe -m <package>`.
- Keeps Windows GUI launch console-free while preserving Python-only launcher policy.

## v1.5.4 - 2026-03-06
- Switched Windows GUI launcher to `scripts/windows/run_app_gui.pyw`.
- Initial detached GUI launcher behavior update.
- Updated template policy and docs to require/reference `run_app_gui.pyw`.

## v1.5.3 - 2026-03-06
- Standardized `README.md` legal disclaimer content using canonical marker block:
  - `<!-- legal-disclaimer:start -->`
  - `<!-- legal-disclaimer:end -->`
- Added canonical source text at `aatemplate/legal_disclaimer.md`.
- Updated template README to include the standardized legal disclaimer block.
- Expanded policy checker to require:
  - tracked `README.md`
  - legal disclaimer markers in `README.md`
  - exact canonical disclaimer content between markers.

## v1.5.2 - 2026-03-06
- Updated policy checker to disallow tracked `.ps1` scripts in addition to `.cmd`.
- Updated project standard prompt to require Python-only script runners (no `.cmd` or `.ps1` wrappers).
- Updated agent context to persist the same no-`.cmd`/no-`.ps1` rule for template outputs.

## v1.5.1 - 2026-03-06
- Removed `w_ignore_prompt*.txt` from template and synced project `.gitignore` files.
- Removed `.everything_sdk/` from template and synced project `.gitignore` files.

## v1.5.0 - 2026-03-05
- Standardized TOML app config contract for Copier-managed projects:
  - tracked `config/app.defaults.toml`
  - tracked `config/app.example.toml`
  - untracked `config/app.local.toml`
- Updated template `.gitignore` to ignore local config overrides and secrets under `config/`.
- Expanded template README with config precedence, env variables, and OS secret-path conventions.
- Expanded policy checker to:
  - reject legacy root runtime config filenames (`config.toml`, `*_config.toml`, etc.)
  - reject tracked `config/app.local.toml`
  - conditionally enforce canonical `config/app.defaults.toml` and `config/app.example.toml`.

## v1.4.0 - 2026-03-05
- Removed standardized `.cmd` launch/test wrappers.
- Added Python runner scripts:
  - `scripts/windows/run_app.py`
  - `scripts/windows/run_app_gui.py`
  - `scripts/windows/run_tests.py`
- Updated policy checks to disallow legacy `.cmd` scripts entirely.
- Updated template documentation to use Python runner invocations.

## v1.3.0 - 2026-03-05
- Added strict project policy checker at `scripts/policy/check_standard.py` and wired it into:
  - `hatch run lint:policy`
  - `hatch run lint:all`
  - CI lint job
  - pre-commit local hook
- Added Ruff naming enforcement (`N` rules) for more uniform identifier style.
- Added pre-commit hooks:
  - `check-case-conflict`
  - `check-illegal-windows-names`
- Updated template README quick commands with `lint:policy` and standardized Windows helper scripts.

## v1.2.0 - 2026-03-05
- Added standardized Windows script launchers for Copier-managed projects:
  - `scripts/windows/run-app.cmd`
  - `scripts/windows/run-app-gui.cmd`
  - `scripts/windows/run-tests.cmd`
- Added template `src/<package>/__main__.py` and standardized app launch contract as `python -m <package>`.
- Added Copier `_skip_if_exists` rule for `src/*/__main__.py` to preserve customized entrypoints.

## v1.1.1 - 2026-03-05
- Added `PySide6>=6.10.2` and `qbittorrent-api>=2025.11.1` to standardized Hatch `default` and `test` environment dependencies.
- Enables consistent GUI/API test tooling across Copier-managed projects.

## v1.1.0 - 2026-03-05
- Added Copier `_skip_if_exists` rules for `README.md` and `src/*/__init__.py`.
- Prevents future `copier update` runs from clobbering project-specific docs and package init behavior.

## v1.0.2 - 2026-03-05
- Expanded baseline `.gitignore` to ignore local workspace-only artifacts.

## v1.0.1 - 2026-03-05
- Updated starter generated test to a package import smoke test.
- Keeps compatibility when existing package `__init__.py` is preserved.

## v1.0.0 - 2026-03-05
- Initial Copier template release.
- Standardized Python 3.13 Hatch+uv project scaffold.
- Ruff + basedpyright strict + pytest-cov + pre-commit baseline.
- Windows-first CI with `main`/`master` branch filters.
