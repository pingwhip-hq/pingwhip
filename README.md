# PingWhip Integration & Documentation

> **Zero-agent heartbeat monitoring and synthetic cron telemetry.**  
> Monitor scheduled tasks, background workers, and batch pipelines with sub-10ms edge ingestion across 300+ PoPs.

---

## Quickstart

Append a PingWhip heartbeat ping to the end of any scheduled script or crontab command:

### 1. Basic Crontab Append

```bash
0 3 * * * /opt/scripts/backup-db.sh && curl -fsS -m 10 --retry 3 [https://pingwhip.com/ping/YOUR_MONITOR_TOKEN](https://pingwhip.com/ping/YOUR_MONITOR_TOKEN)

#!/usr/bin/env bash
set -Eeuo pipefail

PING_TOKEN="YOUR_MONITOR_TOKEN"
START_TIME=$(date +%s)

# Trap execution failures and alert immediately
trap 'curl -fsS -m 5 -X POST "[https://pingwhip.com/ping/$](https://pingwhip.com/ping/$){PING_TOKEN}/fail" \
  -H "Content-Type: application/json" \
  -d "{\"error\": \"Failed at line $LINENO with exit code$?\", \"duration\": $(($(date +%s) - START_TIME))}"' ERR

# Run your critical workload
python3 /srv/workers/reconcile_ledgers.py

# Send successful heartbeat with measured execution duration
DURATION=$(($(date +%s) - START_TIME))
curl -fsS -m 5 "[https://pingwhip.com/ping/$](https://pingwhip.com/ping/$){PING_TOKEN}?duration=${DURATION}"

Endpoints
Method        Endpoint                       Description
GET / POST    /ping/:token                   Acknowledges successful heartbeat.
                                             Optional parameter: ?duration=<seconds>
POST          /ping/:token/fail              Trips monitor to DOWN state immediately
                                             with optional diagnostic error payload
POST          /api/v1/monitors/:slug/ping    Bearer-authenticated ping via scoped API key

Resources
Web Console: app.pingwhip.com
System Status: app.pingwhip.com/status
Issues & Feedback: GitHub Issues

License
Examples and documentation are licensed under the MIT License.
