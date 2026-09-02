# Engineering Standards

A vendor-neutral, drop-in standards file for AI coding assistants.

## What's here

[`CLAUDE.md`](./CLAUDE.md) — the standards file itself, in two parts.

**Engineering Standard** — what must be true of the work:

- Precedence, and what the owner authorizes
- Which acts require that authorization
- How much review a change needs, and when a review stops counting
- What counts as evidence
- Scope, tests, secrets, concurrent work

**Session Protocol** — how a session conducts itself so the standard holds:

- Which path applies to the change at hand
- What is recorded before work starts, and what the plan must address
- How a review is requested so it is not steered
- The gates before delivery, and what makes a test run admissible
- The durable record, and how conclusions are reached

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
