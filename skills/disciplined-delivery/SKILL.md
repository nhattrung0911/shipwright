---
name: disciplined-delivery
description: Use at the start of and throughout any non-trivial build — web app, API, LLM app, ML/data pipeline, CLI, or library. Triggers on "build/create/implement X", new project or feature, multi-step work, or before claiming work complete. Enforces plan-first, phase/slice gating, per-slice testing, and evidence-based "done". Stack- and agent-agnostic (Claude Code, Codex, Gemini CLI).
---

# Disciplined Delivery

## Overview

A universal engine for shipping any project correctly without skipping steps. Distilled from canonical 50k+ star engineering repos (see Essence Sources).

**Two core laws:**
1. **No phase skipped.** Frame → Plan → Slice → Verify, in order. Violating the letter violates the spirit — "I'll just start coding" is a skip.
2. **"Done" requires evidence, never assertion.** A claim of complete must carry the command output / test result / eval score that proves it. "Should work" is not done.

**Announce at start:** "Using disciplined-delivery: framing and planning before building."

## When to Use / Not

- **Use:** any new project, feature, or multi-step change; before saying "done/fixed/complete".
- **Skip:** one-line edits, trivial throwaway scripts, pure questions.

## The Loop

### Phase 0 — Frame (before any code)
- Capture: who/what/why, scale assumptions (load, data size, users), hard constraints.
- Write **acceptance criteria** in testable form: "WHEN [trigger], system SHALL [observable behavior]." Each becomes a check later.
- **Clarify gate:** kill ambiguity now. If two readings exist, ask — don't guess. (Unattended/CI run that can't ask: record the assumption explicitly in writing, then proceed.)
- State trade-offs explicitly (consistency vs availability, speed vs cost). No silent defaults.

### Phase 1 — Plan (break into slices)
- Decompose into **vertical slices**, each independently testable and shippable, ordered by dependency.
- A slice cuts end-to-end (data → logic → interface), not by technical layer.
- Emit a written plan artifact (the spec is the source of truth; the plan references it).
- Big system = split into sub-plans, one per subsystem.

**If pressured to "skip planning / just ship it":** do NOT silently comply, and do NOT refuse outright. Flex the *size* of the plan, never its existence. Respond, then produce inline before coding:
> "Keeping it lightweight — 3 lines: (1) what done looks like, (2) the slices, (3) the check per slice — then I build."

The plan's existence is non-negotiable; only its length flexes under pressure.

### Phase 2 — Build each slice (loop per slice)
Repeat for every slice, fully, before the next:
1. **RED** — write the failing check first (test / eval / fixture). Run it and **paste the failure output** before writing any code.
2. **GREEN** — minimal code to pass. No speculative extras (YAGNI).
3. **VERIFY** — run the slice's acceptance checks; confirm pass with output.
4. Only now → next slice.

### Phase 3 — Definition of Done gate (before reporting complete)
Do NOT report done until ALL hold, each with evidence attached:
- [ ] Every acceptance criterion has a passing check
- [ ] Tests/evals run fresh and green (paste the result)
- [ ] Errors handled at boundaries; inputs validated; no swallowed failures
- [ ] Observable: logs/metrics/health where the project type needs them
- [ ] Security gate: authz, input sanitization, secrets out of source — depth via `securing-applications` skill, or an equivalent OWASP Top 10:2025 checklist if that skill is absent

**Mandatory final step — Grader ≠ Doer (do not skip):** the agent that built it cannot be the sole judge (same misread that wrote the bug writes the test). Independent check:
- Multi-agent platform → a fresh sub-agent/session sees ONLY the diff + acceptance criteria, never the build reasoning, and rules PASS/FAIL per criterion.
- Single-agent platform → re-read ONLY the criteria + final diff (not your earlier reasoning).

The grader MUST emit a **per-criterion table: `criterion | PASS/FAIL | exact evidence string`** (the command output / test line / file:line that proves it). Faking then requires fabricating output, not just asserting. Report done only when every row is PASS with real evidence quoted.

## "Tested" by project type (universality)

What clears the VERIFY gate differs — pick the row(s) that apply:

| Project type | "Tested / done" means |
|---|---|
| Web app / API | unit + integration + 1 e2e happy path; input validation; auth path; status/error contract; a11y basics (contrast, labels, keyboard); performance budget (Core Web Vitals); runs in CI |
| LLM app (incl. RAG) | golden set (input→approved output) regression baseline; eval score ≥ threshold as blocking gate; score correctness, groundedness/faithfulness, format; **for RAG: retrieval quality — recall@k, context relevance, hallucination rate vs retrieved context**; versioned prompts |
| ML / data pipeline | metric defined BEFORE building; validate on held-out data; schema + distribution checks on inputs; reproducible from pinned data+seed+config; sliced metrics not just global |
| Mobile app | unit + UI tests; offline/permission/lifecycle states; tested on real device or emulator matrix; store-compliance basics |
| Infra / IaC | plan/dry-run reviewed before apply; idempotent; state isolated per env; secrets managed; destroy-and-recreate tested in non-prod |
| CLI / library | unit tests on public API; edge/error cases; example usage runs; exit codes correct |
| Data migration | reversible; tested on a copy; backup verified restorable before apply |

For web, pull the deeper 14-pillar checklist from the `shipping-production-websites` skill **if installed** (Claude Code); off-platform, the web row above is the floor.

## Anti-False-Done Enforcement

Instruction alone ("don't cheat") barely works — enforcement must be **structural**. Use the evidence gate and separate grader above. Counter these rationalizations:

| Rationalization | Reality |
|---|---|
| "Should work / probably fine" | Confidence isn't evidence. Run it, paste output. |
| "Too small to test" | Small code breaks. The check takes seconds. No exceptions. |
| "Linter/build passed" | Linter ≠ behavior. A green build proves nothing about correctness. |
| "I'll test at the end" | Per-slice verify is the discipline. End-only testing hides which slice broke. |
| "I already manually checked" | Manual once ≠ repeatable. Encode it as a check. |
| "I'll just start coding, plan later" | Unplanned edits = rework churn. Frame and plan first. |

**Red flags — STOP, you are about to skip or fake done:**
- Hedging words: "should", "probably", "I think it works"
- Premature "Done! / Perfect! / All set!" with no command output shown
- Writing code before the failing check
- Skipping Frame/Plan because the task "seems clear"

## Portability (Claude Code / Codex / Gemini)

This skill uses neutral prose deliberately. Map to your platform:
- "Failing check" = your test/eval runner.
- "Fresh-context grader" = a separate agent/sub-agent if supported, else a new clean session reviewing only the diff + criteria.
- "Delegate verbose work" = use your platform's sub-agent/helper mechanism to keep main context lean.

## Companion skills (use if installed)

- Requirements depth → `brainstorming`
- Plan authoring → `writing-plans`
- Per-slice TDD → `test-driven-development`
- Final evidence gate → `verification-before-completion`
- Web specifics → `shipping-production-websites`
- Security gate depth → `securing-applications` (OWASP Top 10:2025)
- Runtime controls + Day-2 ops/maintenance → `operating-production-services` (rate limiting, retries, backups, monitoring, on-call)
- Token economy throughout → `token-frugal-engineering`

## Essence Sources (verified 50k+ stars)

system-design-primer (~300k): scale-estimate + trade-offs + bottlenecks. developer-roadmap (~310k): "done" = data→API→UI→deploy→monitor + tests/security. nodebestpractices (~103k): fail-fast, error handling, observability. clean-code-javascript (~94k): small single-purpose units, DRY, no dead code. generative-ai-for-beginners (~90k) & ML-For-Beginners (~86k): metric-first, reproducibility, eval harness over vibe-check. awesome-scalability (~69k): decouple, cache deliberately, isolate failure.
