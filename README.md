# _py_template

Copier template source for the Python project standard used in this workspace.

This template has also been a learning process for me, and it keeps improving as I apply it to real projects.

## What It Standardizes

- Python 3.13 baseline
- `src/` layout
- Hatch + uv workflow
- Ruff + basedpyright strict + pytest baseline
- Copier modes for `qt_app` and `shared_lib`
- Qt composition playbook for decomposing large UI orchestrators into focused collaborators
- Naming conventions: kebab-case project names, snake_case package/module names, and Qt override allow-list in Ruff
- Windows-first CI
- Policy enforcement via `scripts/policy/check_standard.py`

## Template Usage

Apply template to a project:

```powershell
uvx copier copy --trust --force --vcs-ref <TEMPLATE_TAG> <PATH_TO_PY_TEMPLATE> <PATH_TO_PROJECT> --data-file <PROJECT_DATA_YAML>
```

Update an existing templated project:

```powershell
uvx copier update --trust --defaults --vcs-ref <TEMPLATE_TAG>
```

For sibling repos inside this workspace, keep `.copier-answers.yml` on the
relative template path `../_py_template` and run `copier update` from the target
repo root. Do not commit absolute local template paths.

## Notes

- `_py_template` is the template source repository.
- Sibling repositories under this directory are template targets.
- Rollout procedure: [`COPIER_ROLLOUT_RUNBOOK.md`](COPIER_ROLLOUT_RUNBOOK.md).
