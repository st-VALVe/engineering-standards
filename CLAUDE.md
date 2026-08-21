# Global Engineering Standards

These rules apply to **ALL projects** and **ALL code changes**. Every AI assistant must follow them.

## Precedence

When rules conflict, resolve in this order. Higher wins.

1. **Authority limits.** Never overridden by any instruction, deadline, or reviewer approval.
2. **Mandatory stops.** An escalation required by Termination, Degraded review, Testing, Detached work, Concurrency, or Supervision is terminal for whatever it names — a gate, a change, or a worktree. No default, placeholder, or schedule pressure resumes it. A stop outranks the owner's *task* instruction; it does not outrank the owner's explicit resolution of that stop.

   Only an owner answer clears a stop, and it must state which resolution applies:
   - **New budget** — clears numeric exhaustion only. It replaces the old budget and counts from zero at the recorded resolution. A budget never cures a missing capability.
   - **Supply the missing capacity** — the lease tool, the independent test author, the reviewer. Clears capability stops once the capacity exists.
   - **Reduce scope to read-only**, or **abandon the change**.
   - **Accept the risk**, recorded with its exact scope and what is exposed. Never available for anything in Authority.

   Record the stop and its resolution in the durable record before resuming. A run that cannot find a recorded resolution treats the stop as still in force.
3. **The owner's explicit instruction for the current task.**
4. **These standards.**
5. **Project-level standards.** They may add constraints, never relax these. Widening the mandatory change footprint is adding a constraint: a project rule requiring adjacent migration on touch overrides Surgical Changes for exactly that work.

Within these standards, when two rules still collide:

- Surgical Changes beats Boy Scout Rule.
- Simplicity First beats SOLID for single-use code.
- Blast-radius category beats artifact-type category when both apply.
- "Never block" (Owner decisions) beats "ask rather than guess" (Think Before Coding) once a task is under way, and never overrides a mandatory stop.

Everything here is a directive for a literal reader. If a rule admits two readings, the narrower one that limits external effects is correct.

## Definitions

- **Run id** — unique identifier of one execution turn or process. Not a session id; a resumed session produces a new run id each turn.
- **Task lineage id** — stable identifier of the task across all its runs. Successive runs of the same task share it; a different task never does.
- **Change** — the implementation work for one ticket on one branch, from first edit to merge or abandonment. Producing a new candidate does not start a new change.
- **Candidate** — a source-tree commit SHA submitted for review, excluding ledger commits. A candidate exists only once submitted; edits before submission do not create one, and resubmitting after a fix creates the next candidate.
- **Gate** — the review of one pull request, from first submission to merge or escalation.
- **Reservation** — a ledger entry written before a reviewer call. It ends as `verdict` or as `failed` citing the logged error. **The cap counts reservations not marked `failed`** — this includes reservations abandoned by a crash, so a crash never buys a free call.
- **Durable record** — the task's ticket. Where no tracker exists or it is unreachable, use a **ledger ref**: git notes under `refs/notes/agent-ledger`, or an orphan `agent-ledger` branch. The ledger never lands on the task branch and never alters a candidate SHA. Migrate it to the ticket when one exists; on merge it stays on its own ref and is not squashed into the mainline.

---

## Authority

Some actions are never authorized by task assignment, reviewer approval, or schedule pressure. They require explicit owner authorization for the specific action.

**Requires explicit owner authorization:**

- Any production mutation: deployment, migration execution, data change, configuration change. A passed review is a quality gate, not execution authority.
- Making content publicly readable that was not already: a new public repository, a published site or package, social or marketing output, or any assertion of fact about a real person or business.
- Deleting or force-updating a remote ref that any other author has pushed to.
- Rotating, revoking, or replacing a credential the agent did not itself issue.
- Closing a parent epic or initiative.
- Irrecoverable deletion: stashes, remote refs, data stores, other agents' worktrees, and untracked files the agent did not itself create.

**Permitted without asking:** local edits; commits to the task branch; pushing a branch this task created; pushing the ledger ref; opening a pull request **from** that branch into the repository's normal base; creating a ticket for this task; commenting on it; transitioning a child ticket this task completed; deleting tracked files as part of a reviewed diff; deleting build output, dependency directories, and scratch files this task created.

Contributing to a repository or tracker that is **already public** is not "making publicly readable that was not already". Branch pushes, pull requests, ticket comments, and completion notes about the work itself are permitted there. The restriction on factual claims about a real person or business governs published *product* content — a site, a listing, a profile, marketing — not engineering notes describing what a change does.

**Exposed secret:** contain and notify. Revoke only what the agent itself issued. Rotating a shared credential is an owner decision — an agent cannot know the blast radius.

An irreversible action that is not on the permitted list is an owner decision, whatever else this document says.

---

## Prime Directive: Minimize Code Through Quality

- Measure success by code removed or held constant, not by code added.
- A modification must leave the production artifact equal or smaller. Growth requires justification in the PR body.
- Tests, fixtures, durable records, and other artifacts this document mandates are outside the size budget. Never drop a required test to satisfy it.
- Never leave a known correctness, security, or data-loss gap to satisfy a size or scope directive. Fix it or file it as a follow-up ticket and say so in the PR.
- No scope expansion mid-change. Coverage gaps, missing exports, edge cases — separate tickets.

---

## Core Working Principles

### 1. Think Before Coding

*Don't assume. Don't hide confusion. Surface tradeoffs.*

- State assumptions explicitly.
- At task start, ask when the goal or scope is ambiguous. Mid-task, take a reversible default and record the question.
- Present multiple interpretations when ambiguity exists. Do not pick silently and run.
- Push back when a simpler approach exists.

### 2. Simplicity First

*Minimum code that solves the problem. Nothing speculative.*

- No features beyond what was asked.
- No abstractions for single-use code.
- No flexibility, configurability, or extensibility that was not requested.
- No error handling for a scenario the type system or a boundary check already excludes. Anything reachable from outside the process is possible.
- If 200 lines could be 50, rewrite it.

### 3. Surgical Changes

*Touch only what you must. Clean up only your own mess.*

- Do not improve adjacent code, comments, or formatting that was not in scope.
- Do not refactor what is not broken.
- Match existing style within the file. Where the file is inconsistent, match the nearest enclosing scope.
- If you notice unrelated dead code, mention it. Do not delete it.
- Remove imports, variables, and functions that *your* changes orphaned.
- Every changed line traces directly to the request.

### 4. Goal-Driven Execution

*Define success criteria. Loop until verified.*

- Translate imperative tasks into verifiable goals before starting.
- For multi-step tasks, state a plan with a verification check per step.
- Weak criteria ("make it work") cause drift. Define what proves the task done before starting it.

---

## Reviews and Subagents

### Gates

- Gate merges, not plans. Plans get one sanity pass.
- Reviewer count by blast radius. **Two** independent reviewers, strongest available model from each of two different vendors: schema or RLS migrations, auth, payments, data deletion, deploy, public data exposure. **One:** ordinary features. **None:** test-only, docs, config.
- Blast radius is decided by what the change can affect, not by file type. A config or test file that alters deploy, auth, or data access is a two-reviewer change.
- A runtime prompt file changes production behavior. It is never in the no-reviewer tier: one reviewer minimum, two when it touches auth, payments, or data access.
- Findings are BLOCKING (correctness, security, data loss, contract violation) or ADVISORY. Only BLOCKING gates. ADVISORY becomes a follow-up issue.
- One verdict per invocation. No reviewer dialogue.
- Maximum reasoning effort is for two-reviewer tiers only.
- Reviewer approval authorizes technical direction only. It grants nothing in Authority.

### CI

- CI is authoritative about the checks it actually ran and their results. Failing required CI is BLOCKING and no reviewer overrides it.
- Passing CI does not refute a finding outside what CI asserted. Do not dismiss a BLOCKING finding because unrelated tests are green.
- Never wait for CI inside a turn. A CI run is an externally managed job: its provider run id is the registry entry, its status API is the heartbeat, and its terminal conclusion is the status record. The Detached work file protocol does not apply to it.

### Termination

Review exists to converge, not to continue. These limits are hard.

- **Per gate: at most 6 counted reservations**, across every candidate, fix cycle, reviewer, and session. Producing a new candidate does not reset it. Authoring the doctrine that defines this cap is not ordinary gate work; a documentation gate whose reviewers are the acceptance criteria gets its budget from the owner at the outset.
- **Per candidate: at most one reservation per required reviewer slot**, plus one retry if the first returned no verdict. Swapping reviewer identity does not create a new slot.
- Reaching the per-gate cap is a mandatory stop, regardless of what the findings say.
- BLOCKING findings are fixed autonomously; the fix produces a new candidate, and re-review covers only the diff between the previous candidate and it.
- Escalate also when the same finding appears on three candidates, when any two distinct findings have each appeared on two or more candidates within the gate, or for any product or business decision. A finding's identity is the id assigned on its first appearance; a reviewer restating it under a new name is the same finding, and the implementer records the mapping in the ledger.

**Ledger.** A reviewer call takes minutes and therefore runs as a detached job. The reservation records that job's run id, and the job's status record is authoritative for how the reservation terminates.

All ledger writes — reservation and terminal status alike — are serialized by the **gate lease** (see Concurrency). Without it two runs can both see five used and both reserve a sixth.

Each reservation records: gate id, candidate SHA, reviewer identity, run id, timestamp, status. It ends `verdict` with the outcome and finding ids, or `failed` citing the logged error. Anything else counts against the cap.

Session memory is not a counter; after a compaction or crash the durable record is the only truth. If it cannot be read or reconstructed, escalate — never resume from an assumed count of zero.

### Degraded review

- Reviewer times out or errors: retry once. Failures are logged in the ledger and do not consume the cap. A second failure means that reviewer is unavailable.
- **Any tier with a required reviewer unavailable: do not merge.** Record the gap and escalate. Never silently downgrade a gate.
- **Provider quota exhausted:** durable for the whole task. Stop automatic retries, record the outage and its reset time, escalate. Only an owner answer resumes the gate — a recorded reset time is information, not an automatic unlock.
- If a reviewer session is lost, a successor may replace it. Give it the diff, plus the prior verdict when one exists and the failure record when it does not. A successor counts against the same per-gate cap and against the same slot: it is not a third attempt on a candidate whose slot is already spent.
- A reviewer that emits findings but no verdict has not returned a verdict. Treat the partial output as evidence for the next invocation, not as a result.

### Owner decisions

- Never block on a pending owner decision that is a product, content, or business question. Take a reversible default or an explicitly marked placeholder, record the open question, keep working. This does not apply to mandatory stops.
- Keep one running list of pending owner decisions in the durable record. Each entry: the question, the placeholder used, what changes when it is answered. Deliver the batch at the end of the task.
- Placeholders stay marked and source-traceable. A placeholder never reaches public or production-facing output — see Authority.
- Never delegate product direction, business priorities, scope intent, or owner vision to a reviewer.

### Subagents

- Large tasks: the main session orchestrates; subagents do the work and return results, not process detail.
- One execution session per child task: fresh start, narrow brief, ends at the completion note. That child's follow-up fixes stay in its session. If the session is unrecoverable, a fresh session inherits the task with the completion note and ledger as its brief.
- Pick each subagent's model by task difficulty: the cheapest tier that solves it. Top tier only for judgment-critical review and architecture.
- Every subagent gets a deadline at launch, and a token budget when the harness can enforce or report one. Exceeding either is terminal for that subagent. Where no token budget is observable, the deadline and the check budget are the bound — do not state a budget that cannot be measured.
- Log every model choice and its reason in the durable record.
- Launch prompts reference this section. Never restate review policy inline.

### Detached work

Long work does not run inside a turn. The environment kills long calls and sessions crash without notice.

- **Detect capability first.** If durable detachment is unavailable, split the work into sub-deadline steps or report the unsupported state. Do not assume a child process survives the turn.
- **Harness-tracked subagents are externally managed**, like CI: the harness identifier is the registry entry and its completion notification is the status record. The heartbeat file, check budget, kill rules, registry schema, and atomic status publication below apply to **self-managed jobs only**. An externally managed job is complete when its provider reports terminal status; normalize that report into the registry entry, and if the provider reports no usable terminal status, treat the job as failed rather than inventing one.
- **Every self-managed detached job gets:** a unique run id, a purpose id, its own output paths, an input hash, a hard deadline, a check interval, and an entry in a job registry the durable record points to. Externally managed jobs (CI, provider APIs) use the provider's identifiers instead.
- **Completion is a status record**: run id, input hash, exit code, output path, output hash. For a reviewer job it also carries the verdict and the finding ids, and that is what terminates the reservation. A job that exits zero but produces no parseable verdict terminates the reservation as `failed`, not as a verdict. Publish output and status the same way — write to a unique temporary path, flush, atomically rename. A status record is valid when its run id and input hash match **the registry entry for that job**; the collector is normally a later run than the job, and that is expected. A record matching no registry entry is stale; ignore it. Never read a result file that has no valid status record.
- **At most 2 launches per unit of work.** A unit of work is named by a stable purpose id recorded in the registry — invariant across input revisions and run ids, so a relaunch with tweaked input is the same unit. For a reviewer call the purpose id is (gate, reviewer slot, candidate), so reviewing a later candidate is a different unit and draws on Termination's budget, not this one. A killed or failed job may be relaunched once; a second failure is a mandatory stop, not a third launch.
- **Liveness of a job is heartbeat recency**, not process inspection: a job whose heartbeat has not advanced within its check interval has no live owner.
- **Progress is readable.** A job appends to a heartbeat file. Reading a heartbeat is the only sanctioned partial read. Reading the registry or a status record is free and never counts as a check.
- **Liveness checks are bounded:** at most 5 heartbeat checks per job, each at a pre-planned deadline, never immediately consecutive. A check that finds progress may re-plan the remaining schedule; it does not add checks.
- **Deadline is terminal.** A job past its deadline is killed and reported, whatever the heartbeat shows. One extension is allowed, recorded in the registry, only when the heartbeat is advancing; after it, the deadline is final.
- **Stuck** is no heartbeat progress across two consecutive checks, or token budget exceeded. A stuck job is killed and reported.
- **Exhausted checks are terminal.** When the check budget is spent and no status record exists, kill the job, record it failed, and escalate. Never leave a job neither collected nor killed.
- **On resume after a crash:** read the registry before launching anything. Reap or adopt orphans. Never launch a duplicate for a run id that already has a live owner.

### Cost discipline

- No zero-work probes. Empty-stdin reads of a running process, repeated agent listings between steps, and in-turn waits re-send the entire context for nothing. Listing agents once while reconstructing state after a crash or compaction is permitted.
- Capture full command output, including stderr, to a durable file. Bring into context the exit code, the tail, and matches for error markers — never the whole body, and never a field selection that could drop the failure.
- Redact credentials, tokens, and personal data before writing output anywhere durable or pasting it into a tracker. When unsure, link the run instead of pasting it.

### Concurrency

Two writers in one worktree destroy work. This has happened. The rules below are obligations, not an algorithm — a correct lock is a tested tool, not prose.

**Invariants.** These hold however they are implemented.

- One writer per key. The key is a worktree path or a gate. The writer is a run id: two resumed turns of the same session are two writers. Where one storage ref backs several gates, keeping their writes from colliding is the lease tool's problem, not the gate's.
- Acquisition, takeover, and release are atomic against every other writer of that key.
- A holder identifies itself by its run id. Run ids are never reused, so a superseded holder can always detect that it was superseded.
- Ownership is guarded **through** the mutation, not merely checked before it. A check-then-write lease permits a paused writer to be superseded and then complete its write; that is the failure this exists to prevent.
- A holder renews within the staleness window. Graceful completion or abandonment releases; abnormal termination expires through staleness after 30 minutes, and only then may another writer take over.
- All writers of one key see the same lease state. A writer that cannot reach it is not a writer for that key.

**Implementation.** Use a lease tool that satisfies the invariants and has tests. This document does not specify the algorithm; a lock protocol written in prose cannot be tested and will regress.

**Single-writer mode** is the alternative, and it is valid **only where the harness itself serializes runs of a task and names the authoritative writer.** Then: that agent alone mutates the worktree, subagents return diffs and reports instead of writing, and gate ledger writes happen only in it.

An instruction to "be the only writer" enforces nothing. Two resumed turns of one task can each believe they are the designated agent, which is exactly the failure this environment has produced. **Where neither a conforming lease tool nor harness-enforced serialization exists, mutating a shared worktree or writing a gate ledger is a mandatory stop.** Read-only work continues.

A worktree is shared if any run id other than the current one may mutate it, including a resumed turn of this same task. In this environment that is the normal case.

Record which mode the task ran in, and when relying on harness serialization, record the mechanism relied on. An unevidenced claim that the harness serializes is not serialization.

- Parallel writers get separate worktrees. Never two writers in one.
- On entering a dirty worktree, diff and reconcile before writing. Never overwrite uncommitted work you did not create.
- Detached jobs do not hold the worktree lease. They write only to their own output paths.
- In single-writer mode the writer may place an externally authored test into the worktree by applying its patch verbatim and recording its hash. Applying a patch unchanged is not authoring and does not violate Testing.

### Supervision

Reviewers judge the artifact. Nobody judges the session. A review loop that ran eighteen rounds produced eighteen individually reasonable verdicts; the defect was in the process, and no reviewer's remit reached it.

**Assignment.** The owner assigns a supervisor before work starts and writes into the task's durable record: which task lineage is supervised, the supervisor's own record, its check schedule, its check budget, and whether stopping is real or advisory. No such record means no supervisor — an agent never infers its own need for one, because how attended it is is not observable to it.

**The supervisor's record** is a durable append-only log at a key of its own, never the gate ledger and never the task branch. **That key is a Concurrency key** — a resumed supervisor is a new run id and therefore a second writer, so writes to it follow the same lease-or-single-writer rules as any other key.

**A check is one supervisor invocation that examines the evidence and appends one entry** — finding, no-finding, stop, or report — carrying the time and the evidence it rests on. Reserve the entry before the check and terminalize it after, exactly as a reviewer reservation works, so a check lost to a crash still counts. The budget counts entries; the log, not session memory, is the count.

**Powers.** The supervisor reads, appends to its own record, reports to the owner, and — where stopping is real — terminates the supervised task. It never edits the work, never reviews artifacts, never spawns subagents, and never supervises another supervisor. It needs no supervisor itself: it cannot change the work, so it cannot run away with it.

**Stopping is only as strong as the harness.** Stopping is **real** only where the harness can terminate the supervised task *and* prevent or govern the resumption of that task lineage. Otherwise it is **advisory**: the supervisor records the stop and reports, and the owner stops the work. Doctrine cannot make cross-process cancellation reliable, and a flag a running task is not obliged to re-read is not a stop. The assignment states which applies, so the owner knows what they have.

**The task reads the supervisor's record at named points**, never continuously: on start, on resume, before each reviewer reservation, and immediately before each merge, deployment, or other consequential act — reading it last, after every other precondition, so the window between the read and the act is as small as this document can make it. A recorded stop with no recorded owner resolution is a mandatory stop under Precedence.

That window is not zero, and an advisory stop closes nothing on its own. Between read points the task keeps working, and what bounds the damage is the rest of this document: merges gated by review, production by Authority, spend by budgets. A supervisor is not a substitute for any of them.

**Resolution belongs to the record that raised it.** A stop entry carries an id; the owner's resolution is recorded against that id in the task's durable record, and the task treats the stop as cleared only when it can match the two. An unmatched resolution clears nothing.

**Liveness is the schedule, not a probe.** A supervisor whose latest entry is older than its assigned schedule allows is not live, and neither is one whose budget is spent — both are the loss of an assigned capability, which is a mandatory stop; a budget does not replace it. The task observes this at the same named read points. Replacement is an owner decision, and no agent launches a chain of replacements.

**Evidence.** The ledger, escalations, and completion notes are written by the supervised party and evidence only what that party recorded; its narration is never evidence. Independent state is pull request status, CI conclusions, merge history, and harness-reported spend. A finding resting solely on the supervised party's own account is not a finding.

**What it judges.** The process, not the code: the same class of problem recurring across gates, spend rising without gates closing, mandatory stops resumed with no recorded owner answer, and work outside the assigned brief — which is judgeable only because the assignment records that brief and the supervisor compares it against provider-side state, never against the supervised party's account of what it was doing.
---

## Prompt Writing

Applies to **runtime prompt files**: bot system prompts, agent persona files, skill files, LLM-backed feature prompts, tool descriptions the model reads, `<system>`/`<instructions>` blocks. It does not apply to this document or other human-facing standards.

1. State directives. Do not illustrate them.
2. No anti-examples. State the directive; the enumeration is noise.
3. No lists of user-phrasing variants. Detection lives in code, not in the prompt.
4. No Q→A dialogue. Rules are directives, not FAQ.
5. No softening hedges ("try to", "ideally", "if possible").
6. One rule per line or short paragraph.
7. Keep operational data: path constants, command templates, exact file names, JSON and session-key shapes.
8. **English only.** Every rule and comment inside a prompt file is English. User-facing output follows the user's language; the instructions driving the model do not.

**Style:** short, hard, dry, unambiguous. If a rule could be cut in half and stay clear, cut it.

---

## Project Approach Pipeline

1. **Understand context**: read project standards, README, and the code the change touches. Bounded by the change footprint — do not read an entire codebase before a scoped fix.
2. **Define scope**: state what is and is not included.
3. **Design first**: for non-trivial changes, outline the approach and identify edge cases before coding.
4. **Obtain tests**: tests come from a separate authoring context. The implementer requests them; it does not write them.
5. **Implement**: minimal code that makes those tests pass. Running them is required.
6. **Refactor**: clean up while green, within the change footprint.
7. **Review**: per Reviews and Subagents.

### Planning Quality

Every plan addresses, for the surface it touches:

1. **Edge and boundary cases**: empty, null, zero, negative, oversized, malformed; 0- and 1-element collections; first and last; capacity; timeouts; overflow; unicode and duplicates.
2. **Failure modes**: dependency unavailable, permission denied, interruption midway.
3. **Concurrency and ordering**: simultaneous calls, out-of-order events, races.
4. **State transitions**: unexpected state, re-entry, idempotency.

Address the categories that apply. State briefly which do not and why. Do not defer edge cases to implementation.

---

## Architectural Principles

### SOLID

- **Single Responsibility**: one reason to change per module.
- **Open/Closed**: prefer extending through composition over reworking a stable abstraction. Modifying working code is normal for bug fixes and in-scope changes.
- **Liskov Substitution**: subtypes substitutable without altering correctness.
- **Interface Segregation**: small, focused interfaces.
- **Dependency Inversion**: depend on abstractions; high-level modules do not import low-level ones directly.

These guide design at module boundaries. For single-use code, Simplicity First wins — do not introduce an interface where one concrete implementation exists.

### Additional Principles

- **Reuse over Create**: before proposing new abstractions, skills, tools, or modules, check whether an existing mechanism solves it with configuration alone.
- **DRY**: extract at 3+ duplicates with identical intent. Premature abstraction is worse than duplication.
- **Composition over Inheritance.**

---

## Testing Requirements

**Every code change ships with tests.** No exceptions for "simple" changes.

### Authorship and execution

Authoring a test and running it are different acts with different rules.

1. **Tests derive from the spec, not from observed behavior.** Assertions come from the task statement, never from running the code and asserting what it printed. If you cannot write the test without seeing the implementation, the spec is too vague — clarify it first.
2. **Tests are authored outside the implementation context**, by a different person, agent, or session, working from the spec only. Self-authored tests are forbidden outside throwaway prototypes. If no independent authoring context is available, that is a mandatory stop: escalate. Never self-author to keep moving.
3. **The implementer runs the tests and reads the results.** That is required — a regression test must be observed failing before the fix and passing after.
4. **The implementer never edits a test.** Not assertions, not mocks, not setup. A failing test means the code is wrong until proven otherwise.
5. **Defective test procedure.** When the implementer believes the test itself is wrong, it stops and refers the test, with evidence, to a third context — neither author nor implementer. That context may amend the test **once per test per change**, citing the spec clause it violates, and logs the amendment in the durable record. If it finds the test correct, the implementation is wrong. If no third context is available, escalate; the implementer still may not edit the test.
6. **At most 3 verify-fix cycles per attempt.** Running tests is free. One cycle is one edit-then-rerun iteration against failing tests. An attempt is the work up to the first submission for review, and thereafter the work answering one round of BLOCKING findings — each attempt carries its own three cycles, and Termination bounds how many attempts a gate may have. Exceeding three within one attempt is a mandatory stop: the spec or the design is wrong. An accepted test amendment resets the count to zero once per change. Record the attempt number and its cycle count in the durable record so a resumed session can read them.

### Coverage

- **New features**: unit tests covering core logic and edge cases.
- **Bug fixes**: a regression test that fails without the fix and passes with it.
- **Refactoring**: existing tests keep passing; add tests where coverage is insufficient for the refactored area.
- **Legacy code**: see Edge Cases → Legacy Code.

### Test Quality

- Deterministic. No reliance on wall-clock timing or live external services.
- Names describe behavior, not implementation.
- One behavior per test.
- Prefer integration tests for behavior, unit tests for logic-heavy code.
- Mock external dependencies, not internal collaborators.

### When tests can be deferred

- Prototypes explicitly marked throwaway — never merged to a mainline without tests.
- Configuration-only changes — but verify they work.

---

## Branch Naming

Format: `<type>/<TICKET>-<short-slug>` — `feature/OCP-650-animated-UI-coins`, `fix/gh#242-identity-resolve`.

- `<type>`: `feature`, `fix`, `refactor`, `chore`, `docs`, `test`, `perf`.
- `<TICKET>`: original-case ticket id, omitted if none.
- `<short-slug>`: 2–5 words, kebab-case; acronyms keep case.

### Harness-generated names

Check the branch before the first mutation of any worktree:

```bash
git branch --show-current
```

Rename **only** a branch matching `^(claude/)?[a-z]+-[a-z]+-[0-9a-f]{6,}$`, and only when it is local-only or carries no commits by another author:

```bash
git branch -m <type>/<ticket-or-context>-<short-slug>
```

Any other non-conforming name: report it, do not rename. A human's `quick-login-fix` is not a harness name, and renaming it breaks their tracking branch and open PR.

Never delete a remote ref to complete a rename. Push the new name and report the old one — remote deletion is in Authority.

Left behind, harness names become cleanup debt: after a squash merge git cannot prove the work reached the mainline, and every future session re-investigates.

---

## Commit Conventions

### Mandatory behavior

- **Select or create the task branch before the first edit**, not before the first commit. Editing on a protected branch and discovering it at commit time strands the work.
- **Commit to the task branch when a coherent increment is complete.** Never leave hours of work uncommitted — a crash loses it.
- **Never use `--no-verify`.** Bypassing hooks is a human-only emergency operation.
- **Check the branch before every commit.** If it is protected (main, master, develop, release/*), stop and ask which branch to use.
- Merging is not committing. A merge into a mainline follows Reviews and Subagents.

### Default format: Conventional Commits

```
<type>(<scope>): <description>

[optional body]

[optional footer]
```

| Type | When |
|---|---|
| `feat` | New feature or user-facing capability |
| `fix` | Bug fix |
| `refactor` | Restructuring without behavior change |
| `test` | Tests only |
| `docs` | Documentation only |
| `chore` | Build, CI, dependencies, tooling |
| `perf` | Performance improvement |
| `style` | Formatting, naming, no logic change |

Projects may define their own format; theirs wins.

### Quality rules

- Description under 72 characters, imperative mood.
- Body explains why, not what.
- One logical change per commit.
- **Do not write AI attribution into the message.** A trailer the harness appends and the agent cannot suppress is tolerated and is not an agent failure; removing it is the repository owner's concern.

---

## Task Management

### Task structure

1. **Title**: actionable, starts with a verb.
2. **Acceptance criteria**: concrete, testable.
3. **Scope**: state what is excluded when boundaries are unclear.

### Definition of Done

Scoped to what this change introduced or modified:

- Tests pass; new and modified code has coverage per Testing Requirements.
- Code follows project conventions.
- Commit messages follow conventions.
- No TODO/FIXME/HACK added without a linked ticket.
- No debug output added to production code.

Pre-existing violations elsewhere in the file are not this task's obligation. Report them.

### Closing tasks

A task may be closed when it is delivered and verified. Before closing, post a completion note:

- **What shipped** — the delivered behavior, not the implementation.
- **Commit / PR links** — direct URLs.
- **Files changed** — primary modules touched.
- **Verification** — CI run link, test counts, deploy confirmation, smoke results. Never claim "tested" without artifacts. Redact credentials and personal data; link the run rather than pasting it when unsure.
- **Review ledger** — invocations used against the cap, verdicts, unresolved ADVISORY items.
- **Out of scope / follow-up** — what was deferred, with ticket pointers.
- **Session cost** — only when the harness reports it, quoted verbatim. If it does not, omit the line. Never estimate a token count.

Transition to the terminal state only after the note is posted. Do not close a task whose rollout is incomplete. Closing a parent epic is in Authority.

A ticket is the permanent record of *why* a change happened. Someone opening it in six months should learn what shipped and how we know it worked, without reading git log.

---

## Code Quality

### Naming

- Names reveal intent. If a name needs a comment, rename it.
- Booleans are positive predicates: `isActive`, `hasPermission`, `canEdit`.
- Functions are verb + noun: `calculateScore`, `fetchUser`.
- No abbreviations beyond the universal (`id`, `url`, `http`).

### Error handling

- Handle errors where there is enough context to act.
- Validate inputs at system boundaries; trust internal data.
- Never swallow exceptions silently. Log or rethrow with context.
- Use typed errors where the language supports them.

### Dependencies

- Every dependency is a liability. Pin versions, use lock files.
- Before adding one: is it maintained, and does it solve a real problem?
- Before hand-rolling an alternative: does it touch cryptography, authentication, parsing untrusted input, or time zones? Then take a dependency rather than writing your own. Adding a dependency the project does not already use is an owner decision — propose it with maintenance status, licence, and source, and do not hand-roll these categories while waiting.

---

## Cross-Project Application

These rules hold across language, framework, IDE, and AI assistant.

Projects may extend them with stricter requirements and stack-specific conventions. Projects may never relax them.

---

## Edge Cases

### Hotfixes

- Still require tests, scoped to the fix rather than the module.
- Prefix `fix(HOTFIX):`.

### Legacy code

- Add tests for the area you are modifying before changing it.
- Do not refactor untested legacy code without a safety net.
- Boy Scout Rule applies within the change footprint only, and yields to Surgical Changes.

### Third-party and vendored code

- Do not modify it. Wrap it. Patch a library bug via wrapper or fork and document why.

### Generated code

- Do not hand-edit generated files. They are exempt from style and coverage rules and must be marked.

### Performance-critical code

- Benchmark before and after; do not optimize without measurement.
- Document the tradeoff when performance conflicts with clean architecture.

### Secrets

- Never commit secrets, tokens, keys, or credentials. Use environment variables or a secret manager.
- On accidental commit: contain, notify, and follow Authority on rotation.

### Merge conflicts

- Resolve by understanding both sides. Never blindly take "ours" or "theirs".
- Run the tests afterward.
