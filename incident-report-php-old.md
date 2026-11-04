# Incident report: php-old (PHP2Shell)

**Simulated incident (tabletop exercise).** The exposure window is hypothetical. The system, evidence and checks are real.

## Summary
php-old, a Docker container running Apache with PHP 8.4.26 on my lab laptop, was vulnerable to PHP2Shell (CVE-2026-109618 and CVE-2026-109619), which allows unauthenticated remote code execution.

## Exposure window
29 October to 2 November 2026 (4 days), ending when session.upload_progress was disabled.

## Timeline (UTC, from Apache logs)
- 2 Nov 09:34: php-old started
- 2 Nov 09:45: restarted after the mitigation was applied
- 3 Nov 08:57: started again after the host was switched off
- 4 Nov: contained, investigated, snapshotted and removed

## What I checked
1. docker diff: only my mitigation file and Apache's PID file changed. Nothing new in the web root.
2. docker top: only Apache processes (one root manager, five www-data workers). No shell, curl or miner.
3. Logs: no POST requests and no worker crashes. Only my own start and restart events. I confirmed the search worked by matching a word I knew was in the logs.

**Conclusion:** no evidence of compromise. This does not prove nothing happened. An attacker who only read files would not show up in these checks.

## Actions taken
1. 2 Nov: mitigated by disabling session.upload_progress.
2. 4 Nov: contained by disconnecting php-old from all networks, keeping it running to preserve evidence.
3. 4 Nov: saved an evidence snapshot (php-old-evidence:2026-11-04), removed php-old, and replaced it with php-new on PHP 8.4.27 built from a clean official image.

## Secrets
php-old held no secrets. A real server would need every password and key the www-data user could read rotated, because an attacker with a PHP2Shell shell runs as that user.

## Lessons learned
- Patching does not remove an attacker who is already inside. Replace, do not clean.
- Preserve evidence before deleting anything.
- An empty search result only counts once you have proved the search can find something.
