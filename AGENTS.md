# CSV2Doc — agent guide

CSV2Doc is a single-file Streamlit app (`app.py`) that uploads a CSV, groups, sorts and filters it with pandas, and exports CSV, HTML or DOCX (python-docx). Live site: https://csv2doc.streamlit.app/.

## Hard rules

- A push to `main` deploys to production. Streamlit Cloud auto-deploys from GitHub. Do not push untested changes to `main`.

## Gotchas

- All state is file-based and gitignored: `data/csv_cache/` (CSV + `{hash}_options.json`), `data/users/{user_id}/history.json`, and `uploaded_files_history.json` at the repo root. Streamlit Cloud storage is ephemeral, so this state can vanish on redeploy.
- User identity is the `user_id` query parameter (`st.query_params`). A URL without it creates a new user and loses the history.
- A file upload triggers a full Streamlit rerun. Keep expensive work behind `st.session_state` or the file cache.
- Grouping creates one table per group value. It does not aggregate. Sort and filter run after grouping.
- Whole files load into memory. There is no streaming, so large CSVs can exhaust memory.

## Conventions

- Sidebar holds all controls. The main area only displays data.
- No test suite exists. When you add tests, use pytest with sample-CSV fixtures.

## Build / run

- `streamlit run app.py` (add `--server.port 8501` to pin the port).
- Production logs: https://share.streamlit.io/ dashboard.

## Scanning & review

Policy lives in `~/.config/agents/AGENTS.md` -> "Dependency, code, and secret
scanning" and "Model delegation & cross-provider review". It is not restated
here; read it there. What is specific to this repo:

- CI runs `gitleaks`, `osv-scanner` and `zizmor` on every PR, every push to `main`, and
  weekly (`.github/workflows/security.yml`). There is no committed
  `package-lock.json`, so the npm jobs do not apply.
- Dependabot config is committed, with grouped security PRs (`.github/dependabot.yml`)
  for `requirements.txt` and the pinned GitHub Actions.
- SHA-pinned actions get **no Dependabot alerts**: GitHub generates them only
  for actions on semantic versions (docs: "About Dependabot alerts"). So no
  security-update PR can ever fire for a pinned action, an
  `applies-to: security-updates` group over github-actions is dead config, and
  the weekly version update is their only automatic remediation -- which is why
  the github-actions block carries no semver-major ignore. DETECTION comes
  from the `zizmor` job in `security.yml`: it maps each SHA pin to its
  release tag and fails on a known-vulnerable, impostor or unpinned action.
  osv-scanner's `github/actions` plugin only lists the pinned actions: with
  osv-scanner 2.7.0 it did not flag `tj-actions/changed-files`
  (GHSA-mrrh-fwg8-r2c3) at a vulnerable SHA or tag. zizmor looks up a SHA
  pin by one tag: for a root action (owner/repo@sha) the longest tag
  without a `/`, for a sub-action the longest tag under its path or its
  last directory (else the longest of all). The job fails unless that tag, and any tag tied
  with it, is a plain release version (v1, v1.2.3): an untagged commit,
  a non-version tag or a path-prefixed tag matches no advisory. Pin
  released versions.
- Dependabot **alerts** are a per-repo GitHub setting; a config file does not
  turn them on. Without them the config above governs version updates only.
  Check it by HTTP STATUS, not by exit code -- `gh api` exits 0 on success
  and 1 on any failure, and never exits 404 or 403:
  `gh api -i -X GET repos/lelandg/CSV2Doc/vulnerability-alerts | head -1`
  204 = on, 404 = off, 403 = the token has no admin on this repo, which is
  NOT the same as off. Enable: same path with `-X PUT`.
- Alerts and automatic security **updates** are two switches, not one. Alerts
  only report; the half that opens PRs is `automated-security-fixes` (same API
  shape). It is deliberately OFF here: this repo took its first scan with a
  pre-existing advisory backlog, and enabling it would open that backlog as
  PRs at once. Turn it on per repo once the backlog is cleared.
- PR reviewers configured for this repo: **Codex** (run locally against the
  committed branch before opening the PR) and **CodeRabbit**
  (`coderabbitai[bot]`). How to read their checks -- which one re-reviews a
  push, why a green check does not mean "no findings", and which label skips
  which bot -- is in the global policy cited above and is deliberately not
  repeated here.
