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

Policy lives in `~/.config/agents/AGENTS.md` -> "Dependencies, advisories, and
scanning" (full text: `~/.claude/instructions/security-scanning.md`),
"Shipping autonomy" and "Model delegation & cross-provider review". It is not
restated here; read it there. What is specific to this repo:

- CI runs `gitleaks`, `osv-scanner` and `zizmor` on every PR, every push to `main`, and
  weekly (`.github/workflows/security.yml`). There is no committed
  `package-lock.json`, so the npm jobs do not apply.
- Dependabot config is committed (`.github/dependabot.yml`): grouped security
  PRs for `requirements.txt`, and grouped minor and patch version updates for
  the pinned GitHub Actions, which get no security PRs (next bullet).
- Pinned actions get no Dependabot alerts; the `zizmor` job is their
  detection, so pin released version tags only. `automated-security-fixes` is
  deliberately off. Pinned-action detection, the Dependabot alert and auto-fix
  switches and how to check them: `Docs/security-posture.md` (read before
  editing `.github/**`).
- **PR reviewers.** The review council in the global rules applies here; its
  members are whatever `~/.config/agents/council.toml` seats, and the runner
  and commands are in `~/.claude/instructions/model-delegation.md`. This repo
  adds **CodeRabbit** (`coderabbitai[bot]`, config `.coderabbit.yaml`):
  automatic on every non-draft PR, re-reviews every push, and is skipped by
  the `no-ai-review` label (on the PR when it opens) and for Dependabot and
  Renovate PRs. A draft marked ready with no new push is not reliably
  reviewed: comment `@coderabbitai full review` (not `review`, which reads
  only commits pushed since its last review). The list is additive, not
  closed: nothing named here replaces a council member. Why a green check does not mean "no findings" is in
  `~/.claude/instructions/pr-review-automation.md`; how to tell a finished
  review from a skipped or rate-limited one (the check run's output title) is
  in the comments of `.coderabbit.yaml`. Neither is repeated here.
