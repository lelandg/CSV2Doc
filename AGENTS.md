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
