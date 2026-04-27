# AGENTS.md

High-signal operating notes for agents working in `ssl-website`.

## What this repo is

- Sphinx site source lives in `src/`; rendered output goes to `build/`.
- Main wiring is `src/conf.py` (extensions: `ablog`, `sphinx_design`, `autosectionlabel`, `intersphinx`, `todo`).
- Dependency/runtime management is via `uv` (`pyproject.toml`, `uv.lock`, `.python-version` = 3.14).

## Commands that are easy to guess wrong

- Setup: `uv sync`
- Build (clean-ish/full): `uv run sphinx-build -vvv --write-all --fresh-env src build`
- Strict validation before PR: `uv run sphinx-build -n -W --keep-going --write-all src build`
- Link check when URLs changed: `uv run sphinx-build -b linkcheck src build/linkcheck`
- Local preview: `uv run sphinx-autobuild src build`
- Make target is `make serve-site` (not `make serve`).

## Make/pre-commit gotchas

- `make build-proj` and `make serve-site` call `sphinx-build`/`sphinx-autobuild` directly (no `uv run`), so prefer `uv run ...` in agent commands unless env is already activated.
- `make create-dev` runs `pre-commit autoupdate`; this can rewrite `.pre-commit-config.yaml`. Do not run it unless you intentionally want hook version churn.
- Pre-commit is configured and includes `markdownlint-cli2 --fix` and `mdformat`; Markdown files may be auto-rewritten by hooks.

## Content wiring rules that break easily

- Member cards in `src/members.rst` are include-sliced from `src/_static/*.rst` using `:start-after:` and `:end-before:`.
- In those `_static/*.rst` files, keep marker comments exact (e.g. `.. Full Name`) and preserve the trailing `..` sentinel; changing markers breaks includes.
- Publication pages under `src/publications/` feed ABlog `.. postlist::`; include post metadata fields at top (e.g. `:blogpost: true`, `:date:`, `:category:`, `:tags:`).
- Top-level nav is controlled by the hidden toctree in `src/index.rst`; new top-level pages must be added there.

## CI constraints to keep in mind

- CI in `.github/workflows/build.yml` builds with Python 3.13 and runs `sphinx-build --write-all src build` from `./.venv/bin`.
- Local target is Python 3.14, so avoid 3.14-only syntax/features in `conf.py` or tooling scripts unless CI is updated too.

## Workflow conventions from repo docs

- Follow `.github/CONTRIBUTING.md`: branch from `main` as `issue-<number>`, open PR to `main`, and include a closing keyword (`Closes #<n>`, etc.).

## Maintenance rule

- If build/test/lint/tooling workflow changes, update this file in the same PR.
