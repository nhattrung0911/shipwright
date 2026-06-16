# Contributing to Shipwright

Thanks for helping make AI agents ship better. Contributions of all sizes are welcome — new skills, sharper checks, corrections, examples.

## House rules

These keep the suite coherent and portable. PRs that follow them get merged fast.

1. **Portable prose.** Skill bodies must avoid agent-specific syntax (no slash-command syntax, no Claude-only tool names) in the *procedure*. Describe actions in neutral language ("run the test", "delegate to a helper agent if supported"). Mark platform-specific references as "(if installed)".
2. **Evidence-based gates.** Any "done"/"verified" step must require concrete evidence (command output, test result, eval score) — never self-assertion.
3. **Lean bodies.** Prefer tables and tight bullets over prose. Push heavy reference into separate files only when genuinely large.
4. **Accurate triggers.** The `description` frontmatter states *when to use* (triggering conditions), not a workflow summary. Keep it under 1024 characters.
5. **Stack-agnostic by default.** Make a skill technology-specific only when the topic demands it, and say so in the trigger.
6. **Cite real sources.** Security/ops claims should map to recognized standards (e.g. OWASP Top 10:2025) or widely-trusted references.

## Adding a new skill

```
skills/
  your-skill-name/
    SKILL.md        # required: YAML frontmatter (name, description) + body
```

- `name`: lowercase, hyphens only, matches the folder name.
- Cross-link related skills by name so they chain.
- Test it: have a fresh agent read it and try to apply it to a realistic scenario before submitting. Adversarial review (try to find what's missing or wrong) is encouraged.

## Submitting

1. Fork → branch → make your change.
2. Keep commits focused; describe *why* in the message.
3. Open a PR explaining the gap it fills or the bug it fixes.

By contributing you agree your work is licensed under the [MIT License](LICENSE).
