---
name: securing-applications
description: Use when building or reviewing anything that handles untrusted input, authentication, authorization, secrets, user data, PII/privacy, payments, file uploads, CORS/headers, external dependencies, LLM/AI features, or deployment config — and as a mandatory gate before shipping or claiming a feature done. Covers OWASP Top 10:2025 + LLM risks + data privacy. Stack- and agent-agnostic.
---

# Securing Applications

## Overview

Security is a **gate, not a feature** — it blocks "done", it isn't an optional add-on. Built on **OWASP Top 10:2025** (current edition). Most breaches are boring: missing access checks, misconfig, leaked secrets, unpatched dependencies. Cover the basics relentlessly.

**Core principle:** Never trust input, never trust the caller, never trust defaults. Verify at every boundary.

**Announce when securing/reviewing:** "Using securing-applications: checking against OWASP Top 10:2025."

## When to Use

- Building: auth, input handling, file upload, payments, admin features, anything touching user data.
- Configuring: deploy, server, cloud, CORS, headers, env.
- Reviewing: before shipping any feature; pre-launch; dependency updates.

## OWASP Top 10:2025 — risk → defense → how to check

| # | Risk | Defense | Check |
|---|---|---|---|
| A01 | **Broken Access Control** (#1) | Authorize EVERY request server-side per resource & action; deny by default; never trust client role/ID; check ownership per object (**BOLA**); reject unexpected fields (**mass assignment**); validate outbound URLs (**SSRF** — allowlist, block link-local & cloud metadata `169.254.169.254`) | Access another user's object by ID across ≥2 users → 403; unauthenticated request → deny; POST an extra `isAdmin` field → ignored; SSRF to metadata IP → blocked |
| A02 | **Security Misconfiguration** | Harden defaults; **full security headers** (CSP, HSTS, X-Content-Type-Options, X-Frame-Options/frame-ancestors, Referrer-Policy, Permissions-Policy, COOP/COEP); disable debug/dir-listing; least-privilege cloud IAM; **CORS: never reflect arbitrary Origin, never `*` + credentials** | Scan headers (securityheaders.com-class); review prod config diff; no `0.0.0.0` open ports, no default creds; CORS not wildcard-with-creds |
| A03 | **Software Supply Chain Failures** (rising) | Pin deps (lockfile, frozen install in CI); scan for known CVEs; verify package integrity; pin CI actions by SHA; generate SBOM; beware typosquats/compromised maintainers | Run dependency audit (e.g. `npm audit --audit-level=high`, `pip-audit`); CI fails on critical CVE; review new deps before adding |
| A04 | **Cryptographic Failures** | TLS everywhere; hash passwords with **argon2id (tuned params)** or bcrypt; encrypt sensitive data at rest; no homemade crypto; rotate keys; monitor cert expiry | Grep for weak algos (md5/sha1 for passwords); confirm HTTPS-only + HSTS; secrets encrypted at rest |
| A05 | **Injection** (SQL/NoSQL/cmd/XSS) | Parameterized queries/ORM; validate+sanitize input; **escape/encode output (XSS)**; never string-concat queries/commands; **CSRF: SameSite=Lax/Strict cookies + per-request token on state-changing routes**; **file uploads: validate magic bytes, AV-scan, store non-executable off-origin** | Injection payloads blocked; no string-built SQL; output encoded; CSRF token required on writes; malicious upload rejected |
| A06 | **Insecure Design** | Threat-model before building; security requirements as acceptance criteria; rate-limit as anti-abuse (credential stuffing, scraping); abuse-case thinking | Documented abuse cases for auth + payments exist; limits/quotas designed in |
| A07 | **Authentication Failures** | Strong auth (MFA option); secure session mgmt; lockout/throttle on login; no creds in URL; rotate/expire tokens; **JWT: validate alg/aud/iss/exp server-side, reject `alg=none`, plan revocation** | Brute-force login → blocked; sessions expire; tokens not in logs; tampered/`alg=none` JWT → rejected |
| A08 | **Software/Data Integrity Failures** | Verify update/deploy signatures; protect CI/CD pipeline; no unsigned/auto-updates from untrusted source; validate deserialized data | Pipeline access controlled; artifacts signed; no untrusted deserialization |
| A09 | **Security Logging & Alerting Failures** | Log auth events, access-control failures, server errors (structured); alert on anomalies; protect & retain logs; NEVER log secrets/PII | Are failed logins logged + alerted? Secrets absent from logs? |
| A10 | **Mishandling of Exceptional Conditions** (new) | Fail closed (deny on error, never grant); handle all error paths; no stack traces/internal detail to users; consistent error responses | Force errors → app denies access, shows generic message, doesn't crash open |

## Secrets — non-negotiable

- Secrets in environment/secret-manager, **never in source or commits**. Scan history before pushing.
- Separate dev/staging/prod credentials. Rotate on exposure and on a schedule.
- No secrets in client-side code, logs, error messages, or URLs.

## LLM / AI-app risks (if the app calls an LLM)

OWASP LLM Top 10 territory — the build checklist's "tested" is not "safe" here:
- **Prompt injection:** treat ALL model input (user text, retrieved docs, tool output) as untrusted; never let it override system instructions.
- **Treat model OUTPUT as untrusted:** never exec/eval it, never render unsanitized, never trust it for authz decisions.
- **Tool/function calls:** least-privilege, allowlist actions, confirm destructive ops; a compromised prompt must not reach dangerous tools.
- **Data exfiltration:** guard against the model leaking secrets/PII via tool calls or output.

## Data privacy & consent (owner: this skill)

- **PII inventory:** know what personal data you store and why.
- **Data-subject rights (DSAR):** deletion + export paths exist (GDPR/CCPA).
- **Consent:** cookie/tracking consent gate before non-essential tracking; honor opt-out.
- **Retention:** delete data past its retention window (cleanup jobs — see `operating-production-services`).

## Pre-Ship Security Gate

Do NOT mark a feature/site done until (tick = tested with evidence, not assumed):
- [ ] Every user-reachable action has a server-side authorization check; tested across ≥2 users + unauthenticated (A01)
- [ ] **BOLA / mass-assignment / SSRF** checked (A01)
- [ ] All input validated server-side; all output encoded (A05)
- [ ] **CSRF** defense on state-changing routes (A05)
- [ ] **File uploads** validated + scanned + stored non-executable (A05)
- [ ] **CORS** not wildcard-with-credentials; full security headers set (A02)
- [ ] Secrets out of source; prod config hardened (A02, A04)
- [ ] Dependencies scanned, no unpatched critical CVEs (A03)
- [ ] Auth: throttling, session expiry, password hashing (argon2id/bcrypt), JWT validated (A07, A04)
- [ ] Errors fail closed; no internal detail leaked (A10)
- [ ] Security events logged + alertable; no secrets in logs (A09)
- [ ] **If LLM app:** prompt-injection + untrusted-output + tool-scope handled
- [ ] **Privacy:** PII inventory + DSAR + consent + retention addressed

## Verify (neutral — map to your stack)

- "Dependency audit" = your ecosystem's CVE scanner.
- "Header/config scan" = a security-header checker + manual prod-config review.
- "Injection/access test" = attempt the attack yourself (or via a helper agent) and confirm it's blocked — evidence, not assumption.
- For a structured review on Claude Code, the `/security-review` command if available.

## Companion skills

- Overall delivery gate → `disciplined-delivery` (security gate references this skill)
- Web build → `shipping-production-websites` (pillar 5 = this skill's depth)

## Source

OWASP Top 10:2025 — owasp.org/Top10/2025 (current edition; A03 Supply Chain rising, A10 Mishandling Exceptional Conditions new).
