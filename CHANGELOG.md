# Changelog

All notable changes to Shipwright are documented here.
Format follows [Keep a Changelog](https://keepachangelog.com/); versioning is [SemVer](https://semver.org/).

## [1.0.0] — 2026-06-16

### Added
- **`disciplined-delivery`** — universal delivery engine: Frame → Plan → Slice → Verify, evidence-based "done", grader-≠-doer check, anti-false-done counters. Covers web, LLM, ML, CLI, library, mobile, infra.
- **`shipping-production-websites`** — web specialization: 12-layer full-stack map, 14 production pillars, pre-launch gate.
- **`securing-applications`** — OWASP Top 10:2025 mapped to defense + check per risk; CSRF/SSRF/BOLA/JWT/file-upload/CORS/headers; LLM prompt-injection; data privacy.
- **`operating-production-services`** — runtime reliability controls + Day-2 operations: monitoring/SLO, on-call, incident response, deploys/rollback, DR (RTO/RPO), backups + restore drills, cert rotation.
- **`token-frugal-engineering`** — context-economy discipline for long agent sessions.
- Plugin packaging (`.claude-plugin/plugin.json` + `marketplace.json`) for one-command install on Claude Code; manual-copy instructions for Codex and Gemini CLI.
