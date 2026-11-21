# PingWhip Integration & Documentation

> **Zero-agent heartbeat monitoring and synthetic cron telemetry.**  
> Monitor periodic tasks, batch workers, and scheduled pipelines with sub-10ms edge ingestion across 300+ PoPs.

---

## Quickstart

Add a PingWhip token ping to the end of any scheduled script or crontab command:

```bash
# Basic Crontab Append
0 3 * * * /opt/scripts/backup-db.sh && curl -fsS -m 10 --retry 3 [https://pingwhip.com/ping/YOUR_MONITOR_TOKEN](https://pingwhip.com/ping/YOUR_MONITOR_TOKEN)
