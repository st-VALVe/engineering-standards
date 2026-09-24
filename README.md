# Engineering Standards

Practical rules for people and coding agents, balancing code quality, reliability and delivery speed.

- [`CLAUDE.md`](./CLAUDE.md) is the authoritative English standard, version 16. It combines engineering practices with a focused agent supplement and replaces the former separate Session Protocol.
- [`ENGINEERING_STANDARD.ru.md`](./ENGINEERING_STANDARD.ru.md) is the Russian reading translation of the same version. Update and review both texts together; English governs if they differ.

The standard covers simplicity, architecture, existing code style, tests, dependencies, Git, review, task completion and authority. Agents finish authorized local work and report after committing by default. Pushes, PRs, merges and deployments require further owner instructions; instructions already covering the work do not need repeated approval.

## Use

Install the approved English text in the instruction file your tool loads. Existing global locations include `~/.codex/AGENTS.md`, `~/.claude/CLAUDE.md` and `~/.gemini/GEMINI.md`; verify the active configuration of your tool.

Keep any private/local appendix outside this public repository. When updating an installed copy, replace its public standard while preserving local additions and checking for conflicting older instructions. Record the installed source commit so it can be verified or rolled back. Existing sessions may retain previously loaded instructions; start a fresh session to use the updated global file.

Project instructions provide actual build/test commands, architectural boundaries, delivery procedures and evidence locations. They may replace explicitly marked defaults and strengthen requirements. Other exceptions require the owner's recorded decision; outgoing secret screening cannot be waived. Project files do not themselves grant an agent authority for external actions.

References explain the selected practices, not a claim that one organization mandates this entire policy. They need not be reread for every task. Adoption and document review do not prove productivity or defect improvements: evaluate those on real project work.

## Changes

Changes to governing rules are high-risk. Review the design and result independently, check representative scenarios, retain review limitations, and verify any translations. Publish and install only within the owner's authorization. Keep task-specific review evidence outside the installed instruction text.

## License

[CC0 1.0 Universal](./LICENSE). Linked publications retain their own licenses.
