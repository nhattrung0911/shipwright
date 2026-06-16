---
name: shipping-production-websites
description: Use when starting, planning, scoping, or auditing any website or web app intended to reach real users — greenfield builds, "build me a site/dashboard/SaaS/landing page", adding a major feature, or pre-launch production-readiness checks. Stack-agnostic.
---

# Shipping Production Websites

## Overview

Single entry-point gate + checklist for building websites/web apps to a production standard. This skill does **not** replace superpowers — it **sequences** them and adds the production-readiness checklist that generic planning misses.

**Core principle:** A site is "done" only when every one of the 14 production pillars below has an explicit decision (even if the decision is "deliberately skipped, here's why"). Silence on a pillar = a gap, not a default.

**Announce at start:** "Using shipping-production-websites to gate and plan this build."

## The Flow (coordinate with superpowers)

Run in order. Each step hands to a superpowers skill; this skill supplies the web-specific content.

1. **Requirements** → **REQUIRED SUB-SKILL:** `superpowers:brainstorming`. Lock: who uses it, scale, must-have vs later, hard constraints (budget, compliance, deadline). Don't pick stack silently — surface options.
2. **Architecture decisions** → fill the Architecture table below. One line per choice + why.
3. **Production checklist** → walk the 14 pillars. Mark each: ✅ in-scope / ⏭️ deferred (+reason) / ❌ N/A (+reason).
4. **UI/UX** → **REQUIRED SUB-SKILL:** `ui-ux-pro-max` for layout, design system, components, a11y-aware states. The interface is the product for a website — do not defer this to a footnote.
5. **Plan** → **REQUIRED SUB-SKILL:** `superpowers:writing-plans`. Embed pillar decisions as acceptance criteria, not afterthoughts. Big build = split per subsystem.
6. **Isolate** (if non-trivial) → **REQUIRED SUB-SKILL:** `superpowers:using-git-worktrees`.
7. **Build** → **REQUIRED SUB-SKILL:** `superpowers:test-driven-development`. Red-green-refactor per task. Keep context lean throughout: **REQUIRED SUB-SKILL:** `token-frugal-engineering` (delegate tests/logs/search to subagents).
8. **Verify before "done"** → **REQUIRED SUB-SKILL:** `superpowers:verification-before-completion`, then run the `/code-review` and `/security-review` slash commands. Evidence, not assertion.

Skip a step only when its superpowers skill itself says it doesn't apply.

**If `disciplined-delivery` is already active**, this skill fills its web row — don't re-run Frame/Plan; supply the web-specific pillars below into its loop.

**Portability:** the `superpowers:*`, `ui-ux-pro-max`, and `/code-review`-style references are Claude Code sub-skills/commands. On Codex/Gemini they may not exist — treat each as "(if installed)"; the pillars and gates below stand alone and are the portable core.

## Architecture Decisions (fill before planning)

| Decision | Pick + why |
|---|---|
| Rendering | SSR / SSG / SPA / hybrid — why |
| Framework / lang | — |
| Data store | SQL / NoSQL / serverless — why |
| Auth model | sessions / JWT / OAuth / managed (Clerk/Auth0) |
| Hosting / deploy target | VPS / PaaS / serverless / edge |
| State of multi-tenancy | single / multi-tenant |
| Sync vs async work | jobs/queues needed? |

## Full-Stack Layer Map (what "full-stack" actually includes)

A site is not just frontend + backend. Decide each layer or mark N/A — these are the buildable parts:

| Layer | Covers | Common miss |
|---|---|---|
| **Config & env** | 12-factor: per-env config (dev/stage/prod) separate from secrets; no config baked into build; feature flags | config-vs-secret confusion; prod config in repo |
| **i18n / l10n** (decide or N/A) | locale routing, translations, RTL, timezone/currency formatting | hardcoded strings; breaks for non-US users |
| **Payments** (if any) | webhook signature verify + idempotency; never store raw card (PCI scope); refund/dispute path | double-charge on retry; storing PAN |
| **Frontend** | UI components, routing, client state, forms, loading/error/empty states | error & empty states forgotten |
| **Backend** | business logic, services, domain rules, validation | logic leaking into controllers |
| **API** | endpoints, contract, versioning, auth, rate-limit (pillar 3) | no input schema |
| **Database** | schema, indexes, migrations, backups (pillar 2) | no indexes, irreversible migrations |
| **Auth** | login + per-resource authorization (pillar 4) | authn without authz |
| **File/media** | upload, validation, storage (S3-class), CDN delivery | unvalidated uploads |
| **Background work** | queues, scheduled jobs, retries, idempotency | sync work that should be async |
| **Realtime** (if needed) | websockets/SSE, presence, reconnection | N/A for many sites |
| **Email/notifications** | transactional email, deliverability (pillar 12) | SPF/DKIM unset → spam folder |
| **Caching** | client/CDN/server/DB cache + invalidation (pillar 10) | cache with no invalidation plan |
| **Infra/DevOps** | hosting, CI/CD, env config, secrets (pillars 8, 14) | manual deploy, secrets in code |
| **Observability** | logs, error tracking, uptime (pillar 9) | bolted on after an incident |

Each layer maps to the pillars below for its "done" bar. Skip a layer only with a written reason.

## The 14 Production Pillars

Every shipped site needs a decision on each. Don't confuse "I didn't think about it" with "not needed."

| # | Pillar | Minimum bar to clear |
|---|---|---|
| 1 | **Architecture** | Rendering + data flow + boundaries decided and written (ADR-style) |
| 2 | **Data & migrations** | Schema versioned; migrations reversible; no manual prod edits; backups |
| 3 | **API contract** | Versioned, validated input (schema), consistent errors, pagination, rate-limit |
| 4 | **AuthN / AuthZ** | Login + per-resource authorization (not just authentication); least privilege |
| 5 | **Security** | OWASP Top 10:2025 — broken access control, injection, misconfig, supply chain, secrets in env not code, HTTPS/headers. Depth: **REQUIRED SUB-SKILL** `securing-applications` |
| 6 | **Validation & errors** | Server-side validation always; graceful error states; no stack traces to users |
| 7 | **Testing** | Unit + integration + e2e covering EACH critical journey (incl. auth + payment), not just one happy path; CI runs them on every PR |
| 8 | **CI/CD** | Automated build→test→deploy; rollback path; no manual ftp |
| 9 | **Observability & ops** | Structured logs + error tracking + uptime/health check + alerting to a person. Runtime controls (rate limit, timeouts, retries, graceful shutdown) + Day-2 maintenance depth: **REQUIRED SUB-SKILL** `operating-production-services` |
| 10 | **Performance** | Core Web Vitals budget (default LCP<2.5s, INP<200ms, CLS<0.1); caching strategy; image/asset optimization; DB indexes |
| 11 | **Accessibility, SEO & analytics** | WCAG AA basics (contrast, labels, keyboard); semantic HTML; meta/OG, canonical URLs, robots, sitemap, structured data, redirect map; consent-gated analytics |
| 12 | **Domain & delivery** | DNS configured; TLS cert issuance + auto-renewal; transactional email deliverability (SPF/DKIM/DMARC) if app sends mail |
| 13 | **Cost & quotas** | Cloud budget alarms; third-party API quota/rate limits understood; no surprise-bill failure mode |
| 14 | **Compliance & ops** | Data-privacy: PII inventory, DSAR (deletion/export), cookie/consent gate, retention (depth in `securing-applications`); legal pages; secrets mgmt; runbook |

## Pre-Launch Gate

Do NOT call a site production-ready until:
- [ ] Every pillar marked ✅ / ⏭️(+reason) / ❌(+reason) — no blanks
- [ ] Tests green in CI; `/security-review` clean or risks accepted in writing
- [ ] Rollback tested; backups verified restorable
- [ ] Error tracking + uptime check live
- [ ] Secrets out of source; prod env separate from dev

## Escalation: installable vendor skills

When a pillar needs depth beyond this checklist, pull the specialist skill instead of improvising:

- **DevOps/IaC/observability** (Docker, K8s, Terraform, CI, Prometheus): `/plugin marketplace add akin-ozer/cc-devops-skills`
- **Full-stack practical** (tests, db migrations, API scaffold, a11y, git): `/plugin marketplace add jezweb/claude-skills`
- **Official vendor skills** (Stripe payments, Neon Postgres, Cloudflare perf, Trail of Bits SAST, Playwright): see officialskills.sh / VoltAgent/awesome-agent-skills
- **UI/UX**: `ui-ux-pro-max` (already installed)

## Common Mistakes

| Mistake | Fix |
|---|---|
| Pick framework before requirements | Brainstorm first (step 1) |
| AuthN without AuthZ | Pillar 4 — authorize every resource access |
| Tests added "after it works" | TDD (step 6); tests-after prove nothing |
| "Works on my machine" → ship | CI gate (pillar 7/8) before deploy |
| Observability bolted on post-incident | Pillar 9 in plan, not after fire |
| Treating a skipped pillar as a default | Every pillar gets an explicit written decision |
