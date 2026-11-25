# Security posture -- pinned actions and Dependabot switches

Reference for the "Scanning & review" section of `AGENTS.md`. Read it before
editing `.github/**`. The bullets below were moved here from `AGENTS.md`; the
only change since is the corrected check for `automated-security-fixes`.

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
  only report; the half that opens PRs is `automated-security-fixes`. Its GET
  does NOT answer like the alerts one: it returns 200 with a JSON body whether
  the switch is on or off, so the 204/404 status mapping above does not apply.
  Read the body, which also shows whether updates are paused:
  `gh api repos/lelandg/CSV2Doc/automated-security-fixes --jq '{enabled, paused}'`
  Enable or disable: same path with `-X PUT` or `-X DELETE`.
  It is deliberately OFF here: this repo took its first scan with a
  pre-existing advisory backlog, and enabling it would open that backlog as
  PRs at once. Turn it on per repo once the backlog is cleared.
