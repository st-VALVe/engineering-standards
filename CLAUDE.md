# Global Engineering Standards

These rules apply to **ALL projects** and **ALL code changes**. Every AI assistant must follow these guidelines without exception.

---

## Prime Directive: Minimize Code Through Quality

- Measure success by code removed or held constant, not by code added.
- A modification or rewrite must leave the artifact equal or smaller. Growth requires per-line justification.
- No scope expansion mid-change. Coverage gaps, missing exports, edge cases — separate tickets.
- Diff line count and artifact count after every change. Unjustified growth — revert.

---

## Core Working Principles

Four principles frame how every task is approached. They override default tendencies and apply before any other rule in this document.

### 1. Think Before Coding

*Don't assume. Don't hide confusion. Surface tradeoffs.*

- State assumptions explicitly. If uncertain, ask rather than guess.
- Present multiple interpretations when ambiguity exists. Do not pick silently and run.
- Push back when a simpler approach exists. Say so.
- Stop when confused. Name what is unclear and ask for clarification.

### 2. Simplicity First

*Minimum code that solves the problem. Nothing speculative.*

- No features beyond what was asked.
- No abstractions for single-use code.
- No flexibility, configurability, or extensibility that was not requested.
- No error handling for scenarios that cannot happen.
- If 200 lines could be 50, rewrite it.
- Self-check: would a senior engineer call this overcomplicated? If yes, simplify.

### 3. Surgical Changes

*Touch only what you must. Clean up only your own mess.*

- Do not improve adjacent code, comments, or formatting that was not in scope.
- Do not refactor things that are not broken.
- Match existing style, even if you would write it differently.
- If you notice unrelated dead code, mention it. Do not delete it.
- Remove imports, variables, and functions that *your* changes orphaned.
- Do not remove pre-existing dead code unless explicitly asked.
- Self-check: every changed line traces directly to the user's request.

### 4. Goal-Driven Execution

*Define success criteria. Loop until verified.*

- Translate imperative tasks into verifiable goals before starting:
  - "Add validation" → "write tests for invalid inputs, then make them pass".
  - "Fix the bug" → "write a regression test that reproduces it, then make it pass".
  - "Refactor X" → "ensure existing tests pass before and after".
- For multi-step tasks, state a brief plan with a verification check per step:
  1. [Step] → verify: [check]
  2. [Step] → verify: [check]
- Strong, testable success criteria let work proceed independently. Weak criteria ("make it work") cause drift and require constant re-clarification.

---

## Prompt Writing

Applies to every file that feeds an LLM as instructions: bot system prompts, agent SOUL/AGENTS/persona files, skill SKILL.md files, LLM-backed feature prompts, meta-prompts, tool descriptions the model reads, any `<system>`/`<instructions>` block.

**Rules:**

1. State directives. Do not illustrate them.
2. Do not list anti-examples ("replies like X, Y, Z are failure modes"). State the directive; the enumeration is noise.
3. Do not list user-phrasing variants the model should react to. Detection lives in code, not in the prompt.
4. Do not include Q→A illustrative dialogue. Rules are directives, not FAQ.
5. No softening hedges ("try to", "ideally", "if possible"). Say what is required.
6. One rule per line or short paragraph. No multi-sentence elaboration of a single rule.
7. Keep operational data: path constants, command templates, exact file names, JSON/session-key shapes. That is data the model needs, not illustration.
8. **English only, always.** Every prompt file, every rule, every directive, every comment inside a prompt file is written in English. No exceptions. User-facing output (what the bot replies to the user in chat) is a separate concern — it follows the user's language — but the instructions that drive the model are English.

**Style:** short, hard, dry, engineering-grade, unambiguous. If a rule could be cut in half and still be clear, cut it.

**Check:** after writing or editing a prompt file, re-read it and delete every sentence that is an example, every clause that softens a rule, every list of phrasings or variants. Scan for non-English content; translate or remove.

---

## Project Approach Pipeline

When starting work on any project or task, follow this sequence:

1. **Understand context**: Read project-level CLAUDE.md, README, existing architecture before writing code.
2. **Define scope**: Clarify what is and is NOT included in the change. Ask if unclear.
3. **Design first**: For non-trivial changes, outline the approach before coding. Consider SOLID principles and how the change fits existing architecture. **Identify edge cases before implementation** — see "Planning Quality" below.
4. **Write tests first**: Define expected behavior through tests before implementing. This forces clear thinking about interfaces and edge cases.
5. **Implement**: Write the minimal code that makes tests pass.
6. **Refactor**: Clean up while tests are green. Improve naming, extract duplications, simplify.
7. **Review**: Verify all standards in this document are met before considering the work done.

### Planning Quality

Every plan or design MUST go beyond the happy path. Before finalizing a plan, explicitly answer:

1. **Edge & boundary cases**: empty/null/zero/negative/oversized/malformed inputs; 0- or 1-element collections; first/last element; max capacity; timeout thresholds; integer overflow; unicode, special characters, duplicates.
2. **Failure modes**: network call fails, file missing, dependency unavailable, permission denied, operation interrupted midway.
3. **Concurrency & ordering**: simultaneous calls, out-of-order events, race conditions.
4. **State transitions**: unexpected state, re-entry (same operation called twice), idempotency.

If the plan does not address these, it is incomplete. Do NOT defer edge case thinking to implementation — surface it during design so it can influence architecture decisions.

---

## Architectural Principles

### SOLID

- **Single Responsibility**: Each class/module has one reason to change. If it does more than one thing, split it.
- **Open/Closed**: Extend behavior through composition and abstractions, not by modifying existing working code.
- **Liskov Substitution**: Subtypes must be substitutable for their base types without altering correctness.
- **Interface Segregation**: Prefer small, focused interfaces over large ones. Clients should not depend on methods they don't use.
- **Dependency Inversion**: Depend on abstractions, not concretions. High-level modules must not import from low-level modules directly.

### Additional Principles

- **Reuse over Create**: Before proposing new abstractions, skills, tools, or modules — check if an existing mechanism can solve the problem with configuration alone. New code is a last resort, not a first instinct.
- **DRY**: Extract only when there are 3+ duplicates with identical intent. Premature abstraction is worse than duplication.
- **Composition over Inheritance**: Prefer composing behaviors over deep inheritance hierarchies.

---

## Testing Requirements

**Every code change must include tests.** No exceptions for "simple" changes.

### Rules

1. **New features**: Must have unit tests covering core logic and edge cases before the feature is considered complete.
2. **Bug fixes**: Must include a regression test that fails without the fix and passes with it.
3. **Refactoring**: Existing tests must continue to pass. Add tests if coverage is insufficient for the refactored area.
4. **Legacy code**: see Edge Cases & Special Scenarios → Legacy Code.
5. **Tests derive from the spec, not from observed behavior.** Assertions come from the task statement, never from running the code and asserting the output. If you cannot write the test without seeing the implementation, the spec is too vague — clarify it first.
6. **Tests must be authored outside the implementation context** — by a different person, agent, or session, working from the spec only. Self-authored tests are forbidden except in throwaway prototypes.
7. **Never edit tests after observing their output.** After writing (or delegating writing of) test files, STOP — do not run them, read results, or adjust assertions, mocks, or setup based on observed errors. Hand verification to the user or a fresh session. Delegating to sub-agents does not grant the orchestrator a "different agent" exemption — the orchestrator is the author. If the user reports a failure, the fix must cite a spec clause, not the error message.

### Test Quality

- Tests must be deterministic — no flaky tests, no reliance on timing or external services.
- Test names must describe the behavior being tested, not the implementation.
- Each test should test one behavior/scenario.
- Prefer integration tests for behavior verification, unit tests for logic-heavy code.
- Mock external dependencies (network, filesystem, databases), not internal collaborators — unless there's a specific reason.

### When Tests Can Be Deferred

- **Prototypes/spikes explicitly marked as throwaway** — but they must never be merged to main/develop without tests.
- **Configuration-only changes** (CI configs, build settings) — but verify they work.

---

## Branch Naming

Format: `<type>/<TICKET>-<short-slug>` — e.g. `feature/OCP-650-animated-UI-coins`, `feature/gh#242-identity-resolve-client`.

- `<type>`: `feature`, `fix`, `refactor`, `chore`, `docs`, `test`, `perf`.
- `<TICKET>`: original-case ticket id (omit if none).
- `<short-slug>`: 2–5 words, kebab-case; acronyms keep case.

### Auto-generated names — FORBIDDEN. Rename is mandatory and is the FIRST action.

Names of the shape `claude/<adjective>-<scientist>-<hex>` (e.g. `claude/ecstatic-brahmagupta-f9620b`, `claude/quizzical-engelbart-e675cc`), `gifted-hertz-529996`, or any random two-word/three-word slug **must never appear in commit history or remote refs**. Claude Code creates these by default inside auto-named worktrees, and every one left behind becomes permanent cleanup debt: after squash-merge git cannot prove the work is reflected in main, so future sessions waste 5–10 minutes per branch on `git cherry` / diff-checks / "is this work in main or not" investigation.

**Mandatory first action when entering ANY worktree — before reading files, before any task-related tool use:**

```bash
git branch --show-current
```

If the result matches the forbidden pattern (starts with `claude/`, random two/three-word slug, hex suffix, no `<type>/` prefix), rename **immediately**:

```bash
git branch -m <type>/<ticket-or-context>-<short-slug>
```

No commits, no edits, no other actions on an auto-named branch — ever. If you already committed on one (e.g., picked up a mid-session task), rename before pushing. If you already pushed, push the renamed branch and delete the old remote ref in the same operation.

This rule has teeth because the cost compounds: one ignored auto-name today is a 10-minute cleanup investigation in three weeks, multiplied by every other ignored auto-name. Don't add to the debt.

---

## Commit Conventions

### Mandatory Behavior

- **Never commit automatically.** Only commit when the user explicitly requests it ("commit this", "make a commit", etc.). Accumulate changes silently until asked.
- **Never use `--no-verify`.** This flag bypasses git hooks and is reserved for human-only emergency operations. The agent must never use it.
- **Check the current branch before every commit.** If the branch is protected (main, master, develop, release/*) — stop, inform the user, and ask which feature branch to use.

### Default Format: Conventional Commits

Unless the project specifies otherwise, follow [Conventional Commits](https://www.conventionalcommits.org/):

```
<type>(<scope>): <description>

[optional body]

[optional footer]
```

#### Types

| Type       | When to use                                          |
|------------|------------------------------------------------------|
| `feat`     | New feature or user-facing capability                |
| `fix`      | Bug fix                                              |
| `refactor` | Code restructuring without behavior change           |
| `test`     | Adding or updating tests only                        |
| `docs`     | Documentation changes only                           |
| `chore`    | Build, CI, dependencies, tooling                     |
| `perf`     | Performance improvement                              |
| `style`    | Formatting, whitespace, naming (no logic change)     |

### Project-Specific Formats

Projects may define their own commit format (e.g., ticket-prefixed: `OCP-123: Description`). When a project defines its own format, use it instead of Conventional Commits. Check the project's CLAUDE.md, commit workflow, or contribution guide.

### Quality Rules (apply to ANY format)

- Description under 72 characters.
- Imperative mood: "add feature" not "added feature" or "adds feature".
- Body explains "why", not "what" — the diff shows the what.
- **Never include AI attribution** — no `Co-Authored-By`, no "generated by AI", no mentions of AI assistants in commits.
- One logical change per commit. Do not mix feature + refactor + fix.

---

## Task Management

### Task Structure

1. **Title**: Clear, actionable, starts with a verb — "Implement X", "Fix Y", "Refactor Z".
2. **Acceptance criteria**: Concrete, testable conditions for "done".
3. **Scope**: Explicitly state what is NOT included if boundaries are unclear.

### Definition of Done

A task is complete when:
- Tests pass; new/modified code has coverage per Testing Requirements
- Code follows SOLID principles and project conventions
- Commit messages follow conventions above
- No TODO/FIXME/HACK comments left without a linked task
- No debug code (console.log, Debug.Log, print) left in production code

### Closing Tasks (Linear / Jira / GitHub Issues / any tracker)

Whenever a task is fully delivered — shipped, deployed, and verified — the agent MUST before closing it:

1. **Post a completion note on the ticket** summarising the outcome. Minimum content:
   - **What shipped** — one or two sentences describing the delivered behaviour (not the implementation).
   - **Commit / PR links** — direct URLs to the merged commits or PR. If multiple commits, list each with its scope.
   - **Files changed** — brief list of the primary files/modules touched (pointers, not full diff).
   - **Verification** — explicit evidence it works: CI run link, test counts (`N/N passing`), live smoke results (sample input → sample output), deploy confirmation. Never claim "tested" without artefacts.
   - **Out of scope / follow-up** — anything explicitly deferred, with a pointer to the follow-up ticket if one exists.
2. **Attach external links** where the tracker supports it (Linear `links` field, GitHub linked-PRs, Jira remote links) so the commit/PR is discoverable from the ticket page, not buried in a comment.
3. **Transition the status** to the tracker's terminal state (`Done`, `Closed`, `Completed`, etc.) only after the note is posted. Never close silently.
4. **Ask before closing parent epics / initiatives.** A sub-ticket being done does not mean the parent is done — confirm scope with the user.

**Why:** tickets are the permanent record of *why* a change happened. The commit message explains the code; the ticket explains the business context, the evidence it shipped correctly, and who to ask. Closing without a note forces future contributors to guess.

**Rule of thumb:** if someone 6 months from now opens the ticket, they should be able to answer "what did we actually deliver and how do I know it worked?" from the ticket alone, without digging through git log.

This applies to ANY work-tracking system the project uses — Linear, Jira, GitHub Issues, Asana, Trello, internal trackers. If the user works with tickets, close them properly.

---

## Code Quality

### Naming

- Names must reveal intent. If a name needs a comment to explain it, rename it.
- Booleans: `isActive`, `hasPermission`, `canEdit` — positive predicates.
- Functions: verb + noun — `calculateScore`, `fetchUser`, `validateInput`.
- Avoid abbreviations unless universally understood (`id`, `url`, `http`).

### Error Handling

- Handle errors at the appropriate level — not too early, not too late.
- Fail fast: validate inputs at system boundaries, trust internal data.
- Never swallow exceptions silently. Log or rethrow with context.
- Use typed errors/exceptions where the language supports it.

### Dependencies

- Minimize external dependencies. Every dependency is a liability.
- Pin versions. Use lock files.
- Evaluate before adding: is it maintained? Does it solve a real problem we can't solve in <50 lines?

---

## Cross-Project Application

These rules apply regardless of:
- **Language**: C#, TypeScript, Python, Go, etc.
- **Framework**: Unity, React, Node.js, etc.
- **IDE**: VS Code, JetBrains, Cursor, etc.
- **AI assistant**: Claude Code, Cursor AI, GitHub Copilot, etc.

### Project-Specific Overrides

Each project MAY have its own `CLAUDE.md` (or equivalent config for other AI tools) that extends these rules. Project-level rules can:
- Add stricter requirements (e.g., "100% branch coverage for payment code")
- Specify tech-stack-specific conventions (naming, file structure, patterns)
- **Never** relax global rules below the minimums defined here

---

## Edge Cases & Special Scenarios

### Hotfixes

- Hotfixes still require tests, but pragmatically: cover the fix, not the whole module.
- Use `fix(HOTFIX):` prefix: `fix(HOTFIX): prevent crash on null user`.

### Legacy Code

- When touching legacy code without tests, add tests for the area you're modifying before making changes.
- Do not refactor legacy code without test coverage as a safety net.
- Boy Scout Rule applies: leave the code better than you found it, but only within the scope of your change.

### Third-Party / Vendored Code

- Never modify vendored/third-party code directly. Wrap it.
- If a library has a bug, patch via wrapper or fork — document why.

### Generated Code

- Do not manually edit generated files (protobuf, codegen, shader variants, etc.).
- Generated code is exempt from style/coverage rules but must be clearly marked.

### Performance-Critical Code

- When performance conflicts with clean architecture, document the tradeoff explicitly.
- Benchmark before and after. Do not optimize without measurement.
- Premature optimization is the root of all evil — but known hot paths deserve attention.

### Secrets & Sensitive Data

- Never commit secrets, tokens, API keys, or credentials.
- Use environment variables or secret management tools.
- If a secret is accidentally committed, rotate it immediately — git history retains it even after removal.

### Merge Conflicts

- Always resolve conflicts by understanding both sides. Never blindly accept "ours" or "theirs".
- After resolving, run all tests to verify correctness.
