# Engineering Standards

A vendor-neutral, dropped-in standards file for AI coding assistants (Claude Code, Cursor, GitHub Copilot, etc.) and humans alike.

## What's here

[`CLAUDE.md`](./CLAUDE.md) — the standards file itself. Covers:

- Core working principles (think before coding, simplicity, surgical changes, goal-driven execution)
- Prompt writing rules (for files that instruct LLMs)
- Project approach pipeline + planning quality bar
- SOLID + supporting architectural principles
- Testing requirements
- Branch naming + commit conventions
- Task management and closure discipline
- Code quality, error handling, dependencies
- Edge cases (hotfixes, legacy code, vendored code, secrets, merge conflicts)

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
