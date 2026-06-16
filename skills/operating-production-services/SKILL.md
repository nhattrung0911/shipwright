---
name: operating-production-services
description: Use when a system will run live and must stay up — designing traffic controls (rate limiting, throttling, timeouts, retries, circuit breakers, idempotency, caching, graceful shutdown, autoscaling) and Day-2 operations/maintenance (monitoring, alerting, SLO, on-call, incident response, deploys/rollback, backups, DB upkeep, dependency patching, log/data retention, cost). Triggers before launch and for any "how do we keep it running" concern. Stack-agnostic.
---

# Operating Production Services

## Overview

Building it is half the job; **keeping it alive under real traffic and over time** is the other half. This skill covers two areas the build checklist only name-drops: **runtime reliability controls** and **Day-2 operations/maintenance**.

**Core principle:** Assume everything fails — slow networks, traffic spikes, bad deploys, abusive clients, aging dependencies. Design controls so failure degrades gracefully and recovery is routine, not heroic.

**Announce when relevant:** "Using operating-production-services for runtime controls and maintenance."

## A. Runtime Reliability Controls

| Control | What / how | Bar to clear |
|---|---|---|
| **Rate limiting** | Cap requests per client key (user/IP/API-key). Algorithm: token bucket or sliding window. Return **429 + `Retry-After`** header. Apply at edge/gateway AND sensitive endpoints (login, signup, write APIs) | Abusive client throttled, normal user unaffected; limits documented |
| **Throttling / quotas** | Per-tenant quotas; backpressure when overloaded; **load shedding** (drop low-priority work) before total collapse | System degrades, doesn't crash, under overload |
| **Timeouts** | Every outbound call (DB, API, cache) has a timeout. No unbounded waits | No request hangs forever |
| **Retries** | Retry only idempotent ops; **exponential backoff + jitter**; cap attempts; never retry 4xx **except 408/429** (honor `Retry-After`) | No retry storms; transient errors recover |
| **Circuit breaker** | Trip open after repeated downstream failures; fail fast; half-open probe to recover | One sick dependency doesn't cascade |
| **Idempotency** | Idempotency keys on writes/payments so retries don't double-charge/double-create | Duplicate request = single effect |
| **Caching** | Client/CDN/server/DB layers; **explicit invalidation + TTL**; stampede protection | Cache never serves stale silently; no thundering herd |
| **Graceful shutdown** | On deploy/restart: stop taking new work, drain in-flight, close connections cleanly | Zero dropped requests on redeploy |
| **Health/readiness** | Liveness (am I up) + readiness (can I serve) + **startup probe** for slow-init apps; load balancer respects them | Unready instance gets no traffic; slow boot not killed |
| **Autoscaling** | Scale on real signal (CPU/queue depth/latency); min/max bounds; **scale-in cooldown/stabilization** to prevent flapping | Spikes absorbed; no thrash; no runaway cost |
| **Edge protection** | WAF, DDoS mitigation, bot filtering at the perimeter | Common attacks blocked before app |

## B. Day-2 Operations & Maintenance

| Area | What / cadence | Bar to clear |
|---|---|---|
| **Monitoring** | Metrics (latency, error rate, throughput, saturation — the "golden signals") + dashboards | Can answer "is it healthy right now" in 10s |
| **Alerting** | Alert on symptoms (SLO burn-rate breach), not every blip; **every alert maps to a runbook**; route to a person; no alert fatigue | Real problems page someone; noise suppressed |
| **Tracing** | Distributed tracing with propagated request IDs across services, not just metrics/logs | Can localize cross-service latency/errors |
| **SLO / error budget** | Define target (e.g. 99.9% / p95 latency); track budget; freeze risky changes when budget spent | Reliability is measured, not vibes |
| **On-call & runbook** | Who responds; written runbook per common failure (symptom → diagnosis → fix) | New engineer can resolve a page from the runbook |
| **Incident response** | Detect → mitigate → communicate (status page) → blameless postmortem with action items | Every incident produces a prevention task |
| **Deploys** | Progressive: canary or blue-green; automated rollback on error-rate spike; never big-bang prod | Bad deploy auto-reverts; no manual ftp |
| **Schema migrations** | Backward-compatible **expand→migrate→contract**; no locking DDL on hot tables; decoupled from app deploy; reversible | Migration can't break running old code; no long table lock |
| **Disaster recovery** | Define **RTO/RPO**; test region/AZ **failover**; backups ≠ DR; documented recovery procedure | Failover drill meets RTO/RPO |
| **Backups & restore** | Automated backups, **encrypted at rest + access-controlled**; **periodically test restore** (a backup never restored = no backup); offsite copy | Restore drill passes on a schedule |
| **Cert & secret rotation** | Automate TLS cert renewal; **alert on cert/secret expiry N days out**; rotate secrets on schedule | No expired-cert outage; rotation tested |
| **Queue / async health** | **Dead-letter queue** for poison messages past retry cap; alert on DLQ depth; monitor lag | Stuck messages isolated, not silently lost |
| **Database upkeep** | Index health, vacuum/analyze, slow-query review, connection-pool limits, archival of old rows | Queries stay fast as data grows |
| **Dependency patching** | Regular CVE scan + update cadence; pin + lockfile; test before bumping; track EOL/deprecated | No unpatched critical CVE lingering |
| **Log & data retention** | Log rotation + retention policy; **scheduled cleanup jobs** for expired/temp data; PII retention limits | Disks don't fill; data isn't hoarded forever |
| **Feature flags** | Decouple deploy from release; kill-switch for risky features; clean up stale flags | Can disable a feature without redeploy |
| **Cost monitoring** | Ongoing spend dashboards + budget alarms; catch runaway resources/queries | No surprise bill; cost tracked like latency |
| **Capacity & deprecation** | Periodic capacity review vs growth; planned sunset path for old APIs/versions with notice | Scale ahead of the wall; clean retirement |

## Pre-Launch Ops Gate

Do NOT call a live service production-ready until:
- [ ] Rate limiting + timeouts + retries (with backoff) on critical paths
- [ ] Graceful shutdown + health/readiness probes wired to the load balancer
- [ ] Monitoring dashboard + alerting to a real person live BEFORE launch
- [ ] Runbook for top failures; on-call owner named
- [ ] Progressive deploy + automated rollback tested
- [ ] Backups automated, encrypted, AND a restore drill passed
- [ ] DR: RTO/RPO defined; failover tested
- [ ] TLS cert auto-renew + expiry alerting; secret rotation set
- [ ] Distributed tracing wired; DLQ for async work
- [ ] Dependency CVE scan clean; patch cadence set
- [ ] Log retention + scheduled data-cleanup jobs configured
- [ ] Cost budget alarms set

## Common Mistakes

| Mistake | Fix |
|---|---|
| Rate limit only on login, not write APIs | Limit every abusable endpoint; return 429 + Retry-After |
| Retry without backoff/jitter | Retry storm DOSes your own backend — add backoff + jitter, cap attempts |
| Retrying non-idempotent writes | Add idempotency keys first, or don't retry |
| "We have backups" (never restored) | A backup never test-restored is not a backup — drill it |
| Alert on everything | Alert fatigue hides real fires — alert on SLO symptoms |
| Big-bang prod deploy | Canary/blue-green + auto-rollback |
| Caching with no invalidation plan | Define TTL + invalidation before adding cache |

## Companion skills

- Overall gate → `disciplined-delivery` (its "observable" + DoD reference this)
- Web build → `shipping-production-websites` (pillars 9/10/13 = this skill's depth)
- Security perimeter overlaps → `securing-applications` (rate-limit also a security control; WAF, A09 logging)
