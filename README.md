# PingWhip Integration & Documentation

> **Zero-agent heartbeat monitoring and synthetic cron telemetry.**  
> Monitor scheduled tasks, background workers, and batch pipelines with sub-10ms edge ingestion across 300+ PoPs.

---

## Quickstart

Append a PingWhip heartbeat ping to the end of any scheduled script or crontab command:

### 1. Basic Crontab Append

```bash
# Ping on successful completion
0 3 * * * /opt/scripts/backup-db.sh && curl -fsS -m 10 --retry 3 [https://pingwhip.com/ping/YOUR_MONITOR_TOKEN](https://pingwhip.com/ping/YOUR_MONITOR_TOKEN)

# Or capture both success and failure inline:
0 * * * * /opt/scripts/sync-stripe.sh \
  && curl -fsS -m 5 [https://pingwhip.com/ping/YOUR_MONITOR_TOKEN](https://pingwhip.com/ping/YOUR_MONITOR_TOKEN) \
  || curl -fsS -m 5 [https://pingwhip.com/ping/YOUR_MONITOR_TOKEN/fail](https://pingwhip.com/ping/YOUR_MONITOR_TOKEN/fail)
```

### 2. Advanced Bash Wrapper (Duration + Exit Code Trapping)

```bash
#!/usr/bin/env bash
set -Eeuo pipefail

PING_TOKEN="YOUR_MONITOR_TOKEN"
START_TIME=$(date +%s%3N) # Milliseconds

# Signal task start (optional)
curl -fsS -m 5 "[https://pingwhip.com/ping/$](https://pingwhip.com/ping/$){PING_TOKEN}/start" >/dev/null 2>&1 || true

# Trap execution failures and alert immediately with line number and exit code
trap 'curl -fsS -m 5 -X POST "[https://pingwhip.com/ping/$](https://pingwhip.com/ping/$){PING_TOKEN}/fail" \
  -H "Content-Type: application/json" \
  -d "{\"error\": \"Failed at line $LINENO with exit code $?\", \"duration\": $(($(date +%s%3N) - START_TIME))}"' ERR

# Run your critical workload
python3 /srv/workers/reconcile_ledgers.py

# Send successful heartbeat with measured duration (ms)
DURATION=$(($(date +%s%3N) - START_TIME))
curl -fsS -m 5 "[https://pingwhip.com/ping/$](https://pingwhip.com/ping/$){PING_TOKEN}?duration=${DURATION}"
```

### 3. Piping Raw `stderr` Diagnostics

When debugging legacy jobs, pipe error logs directly into the failure ping (up to 4KB stored automatically):

```bash
/srv/scripts/nightly-billing.sh 2>&1 | tee /tmp/billing.log || \
  curl -fsS -m 5 -X POST --data-binary "@/tmp/billing.log" "[https://pingwhip.com/ping/YOUR_MONITOR_TOKEN/fail](https://pingwhip.com/ping/YOUR_MONITOR_TOKEN/fail)"
```

---

## Ingestion Endpoints

| Method | Endpoint | State | Description |
| :--- | :--- | :--- | :--- |
| `GET / POST` | `/ping/:token` | `success` | Acknowledges successful heartbeat. Optional query: `?duration=<ms>` |
| `GET / POST` | `/ping/:token/start` | `start` | Registers task initialization |
| `GET / POST` | `/ping/:token/fail` | `fail` | Trips monitor to DOWN immediately. Accepts JSON or raw text error payloads |
| `POST` | `/api/v1/monitors/:slug/ping` | Dynamic | Programmatic ping via scoped API key (`Bearer pw_live_...`) |

---

## Programmatic API (`/api/v1`)

For containerized microservices and automated deployment pipelines:

```bash
curl -X POST "[https://pingwhip.com/api/v1/monitors/nightly-backup/ping](https://pingwhip.com/api/v1/monitors/nightly-backup/ping)" \
  -H "Authorization: Bearer pw_live_your_api_key_here" \
  -H "Content-Type: application/json" \
  -d '{
    "state": "success",
    "duration": 1420,
    "payload": "Backup size: 1.4GB uploaded to S3"
  }'
```

---

## Alert Dispatch Channels

PingWhip dispatches incident alerts directly through:
* **Email:** Powered by Resend (sent from `notice@alerts.pingwhip.com`)
* **Slack:** Rich attachments with status banners, run duration, and error mutation diffs
* **PagerDuty:** Events API v2 with automatic deduplication (`pingwhip-${monitor_id}`)
* **Custom Webhooks:** Authenticated JSON POST payloads with customizable HTTP headers

---

## Resources

* **Web Console:** [app.pingwhip.com](https://app.pingwhip.com)
* **System Status:** [app.pingwhip.com/status](https://app.pingwhip.com/status)
* **Issues & Feedback:** [GitHub Issues](https://github.com/pingwhip-hq/pingwhip/issues)

---

## License

Examples and documentation are licensed under the [MIT License](LICENSE).
