# Engineering Standards

A vendor-neutral, drop-in standards file for AI coding assistants.

## What's here

[`CLAUDE.md`](./CLAUDE.md) — the standards file itself. Covers:

- Authority and safe autonomy
- Task scope and minimal changes
- Architecture and code quality
- Verification and risk-based review
- Concurrent work, delegation, and bounded execution
- Version control and evidence-based delivery
- Prompt and policy writing

## Use

Pick whichever path your assistant reads global instructions from:

| Tool | Path |
|---|---|
| Claude Code | `~/.claude/CLAUDE.md` |
| Cursor | `.cursor/rules/` or project root |
| GitHub Copilot | `.github/copilot-instructions.md` |
| Other | wherever your tool reads global rules |

Copy `CLAUDE.md` there. Add project-specific rules on top — they extend, never relax, the global baseline.

## Modifying

Fork it. Adapt it. No attribution required (see License). If you find a rule that's wrong or stale, open a PR — happy to discuss.

## License

[CC0 1.0 Universal](./LICENSE) — public domain dedication. Copy, modify, distribute freely, no strings.
