# Copier Rollout Runbook

This runbook defines how to propagate `_py_template` updates to sibling projects in
`c:\prj\p2p\ongithub` after template changes are validated.

## Preconditions

1. `_py_template` branch is clean and all template checks pass.
2. A fresh project render from `_py_template` passes:
   - `python -m <package>`
   - `hatch run lint:policy`
   - `hatch run test`
3. A version tag or commit SHA is selected for rollout.

## Per-Repository Procedure

1. Open target repo and capture baseline:
   - `git status --short`
   - `hatch run lint:policy` (if available)
   - `hatch run test` (if available)
2. Create a rollout branch:
   - `git checkout -b chore/template-sync-<date>`
3. Apply template update:
   - `uvx copier update --trust --defaults --vcs-ref <TEMPLATE_TAG_OR_SHA>`
   - keep `.copier-answers.yml` `_src_path` on `../_py_template` so updates work from the repo root without committing absolute paths
4. Resolve conflicts with template-first rules:
   - keep repo-specific logic,
   - accept template updates for shared scaffolding/policy files.
5. Run post-update validation:
   - `python -m <package>`
   - `hatch run lint:policy`
   - `hatch run test`
6. Commit with a single template-sync commit (or split functional vs formatting when needed).

## Conflict Triage Rules

1. Keep template updates for:
   - `pyproject.toml`
   - `scripts/policy/check_standard.py`
   - `scripts/windows/*`
   - baseline package/test scaffolding
2. Keep repo-specific updates for:
   - domain modules,
   - custom UI/business logic,
   - app-specific integrations.
3. If both sides change the same shared file, prefer template structure and re-apply repo logic explicitly.

## Recommended Rollout Order

1. `many-panelz-explorer` (clean architecture, lower conflict risk)
2. `git-statuz` (moderate size, straightforward service/UI split)
3. `video-duperz` (larger modules but contained domain surface)
4. `prowlarr-ui` (very large app module; validate carefully)
5. `arr-helper-ui` (known weakest baseline, more remediation likely)
6. `qbiremo-enhanced` (reference-quality repo; sync last to preserve stable baseline)
