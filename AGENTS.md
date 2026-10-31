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

- CI runs `gitleaks` and `osv-scanner` on every PR, every push to `main`, and
  weekly (`.github/workflows/security.yml`). There is no committed
  `package-lock.json`, so the npm jobs do not apply.
- Dependabot config is committed, with grouped security PRs (`.github/dependabot.yml`)
  for `requirements.txt` and the pinned GitHub Actions.
- Dependabot **alerts** are a per-repo GitHub setting; a config file does not
  turn them on. Without them the config above governs version updates only.
  Check: `gh api -X GET repos/lelandg/CSV2Doc/vulnerability-alerts` (exit 0 = on,
  404 = off). Enable: same path with `-X PUT`.
- PR reviewers: **Codex** (GPT-6 Astra, run locally against the committed branch
  before opening the PR) and **CodeRabbit** (`coderabbitai[bot]`, re-reviews
  every push — the one to wait on for the head being merged). A green CodeRabbit
  check does not mean a review ran: read the check run's output title, where only
  `Finished` means reviewed. Silence means clean; findings arrive as inline
  comments. `no-ai-review` at PR creation skips the bots.
- The global pre-commit hook runs `gitleaks` and fails closed.
