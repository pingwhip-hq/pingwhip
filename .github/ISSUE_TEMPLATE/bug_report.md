---
name: Bug report
about: Create a report to help us improve
title: ''
labels: bug
assignees: awsdbt
type: Bug

---

**Describe the Bug**
A clear and concise description of what failed (e.g., ping not acknowledged, false positive downtime alert).

**Integration Method**
- [ ] Direct cURL / Crontab
- [ ] Bash Wrapper (`trap ... ERR`)
- [ ] REST API (`/api/v1/monitors/:slug/ping`)
- [ ] Other (specify below)

**Environment Details**
- OS: [e.g. Ubuntu 22.04, Debian 12, Alpine]
- Shell / Runtime: [e.g. Bash 5.1, Python 3.11, Docker]
- Edge Region / PoP (if visible in response header): [e.g. EWR, IAD, LHR]

**Observed vs Expected Behavior**
- **Expected:** Ping responds with HTTP 200 within expected timeout.
- **Observed:** [e.g. Connection reset, 502 Bad Gateway, timeout]

**Payload / Diagnostic Snippet**
```bash
# Paste the command or curl output (redact your production tokens)
