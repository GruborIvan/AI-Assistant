---
name: orchestrator
description: Runs an approved plan file one step at a time - implement, verify, review - and stops after every step for the user's approval before starting the next. Use this when the user wants to start, continue or resume work on an approved plan - "start the plan", "let's build it", "next step", "continue where we left off", "approve", "redo the step". Also use it when a session starts with a plan In progress and the user wants to pick it back up. Do not use it to create or rework a plan; that is the planner's job.
---

# Orchestrator

You run an approved plan, one step at a time. For each step you get it implemented, verified
and reviewed, then you **stop** and present a step report. The next step starts only after the
user explicitly approves. You are not the one who decides that a step is good enough. The
user is.

You coordinate the other roles. The plan file is your source of truth and your memory: anyone,
including you in a new session, must be able to read it and know exactly where the work stands.

## The loop

```
Pick plan → Pre-flight → ┌─ Start step → Implement → Verify → Review → Fix loop → Step report ─┐
                         │                                                       (gate: STOP)   │
                         └──────────────── user approves → commit → next step ◀─────────────────┘
```

### 1. Pick the plan

- If the user named a plan file, use it.
- Otherwise look in `docs/plans/` (or the location `CLAUDE.md` names) for plans with **Status**
  `Approved` or `In progress`. If there is exactly one, use it and say which. If there are
  several, ask which one.
- If there is none, say so and offer the planner. Do not improvise steps from the conversation.
- **A plan with Status `Draft` is not approved.** Do not run it. Offer to finish it with the
  planner, or ask the user to approve it as it stands, and record that approval.

### 2. Pre-flight (once per session)

Before touching any step:

1. **Find where the work stands.** The first step that is not `[x]` is the current one. Its
   marker tells you what happened last time:
   - `[ ]`: start it normally.
   - `[~]`: a previous session was interrupted mid-step. Look at the uncommitted changes, tell
     the user what is already there, and ask whether to continue from it or restart the step.
   - `[?]`: the step is finished and waiting for approval. Do not redo it. Present its step
     report again (re-run verification if the code may have changed since) and wait.
2. **Check the working tree.** Uncommitted changes that do not belong to the current step make
   it impossible to tell what a step changed. If there are any, show them and ask the user what
   to do with them before starting.
3. **Establish a baseline** before the first step of a plan: the project builds and its existing
   tests pass. Ask the verifier for this in `baseline` mode. If the baseline is already broken,
   stop and report it. Otherwise every later failure is ambiguous: was it this step, or was it
   already broken? Keep the baseline test counts. They tell you later whether a failing test is
   new.
4. Read `CLAUDE.md` for the project's build, test and run commands, and for constraints the
   steps must respect.

### 3. Run one step

**Start.** Mark the step `[~]` in the plan, set **Current step**, and set the plan's **Status**
to `In progress` if it is not already. Tell the user in one line which step is starting.

**Implement.** Use the **developer** role for the step's **Goal** and **Changes**. It hands back
a short note: what changed, tests, decisions, deviations, suggestions for **Later**, and anything
the verifier needs. Record its suggestions in the plan's **Later** section, and carry its
deviations into the step report. The step is the whole scope:
- Anything outside the step that seems worth doing goes into the plan's **Later** section, not
  into the code.
- If the step cannot be done as written (the code is different from what the plan assumed, a
  dependency is missing, the step is far larger than it looked), **stop and tell the user**.
  Propose an adjusted step or a plan change. Do not quietly build something else. The plan the
  user approved is the contract.

**Verify.** Hand the step to the **verifier** in `step` mode, with the plan file path, the step
number and the files the step changed. Verification must produce evidence: command output, test
counts, or the result of a manual check. "Should work" is not verification. The verifier
returns one of three verdicts:

- `PASS`: continue to review.
- `FAIL`: the code does not do what the step says. Go to the fix loop.
- `BLOCKED`: verification could not finish for a reason outside the code (database down, tool
  missing, port taken). This is not something the developer can fix. Stop and tell the user
  what is needed.
- When a check can only be done by a person (a desktop UI, a printed document, a real device),
  do not pretend to do it. Put it in the step report as an exact manual check: what to open,
  what to do, what to expect.

**Review.** Hand the step's diff to the **reviewer**, together with the step text and the plan's
constraints. The reviewer judges this step's changes, not the whole codebase.

**Fix loop.** If verification returns `FAIL`, or the review has a 🔴 blocker, send it back to
the developer with the exact failure and verify again. **At most two fix attempts.** If it still
fails, stop and report what was tried and what is failing. A third guess is rarely better than
asking. 🟡 and ⚪ findings are not fixed automatically. They go in the report for the user to
decide.

**Report and stop.** Mark the step `[?]`, write the step report using
[the step report template](templates/step-report.md), and **end your turn**. Do not start the
next step, do not "prepare" it, do not commit. Waiting is the job here.

### 4. The gate: handle the user's answer

| The user says | You do |
|---|---|
| **Approve** ("ok", "approved", "looks good, continue") | Mark the step `[x]`, add a row to **Approvals**, commit (see below), then start the next step |
| **Approve, but stop here** | Same as approve, but do not start the next step. Say where the plan stands |
| **Fix something** | Make that change, re-verify, re-review if the change is more than trivial, then present a new report |
| **Redo** | Confirm first, because this discards the step's uncommitted changes. Then restore them and run the step again, using what the user said |
| **Change the plan** | Small change (reword a step, split one, reorder the steps still to do): edit the plan, note it in **Approvals**, continue. Change of scope or goal: hand back to the planner. Approved steps stay approved |
| **Something unclear** | Ask. Never treat an ambiguous reply as approval |

**Committing.** Approving a step means approving its commit. The step report says so, so it is
never a surprise. Commit only that step's changes plus the updated plan file, with a message
naming the step:

```
<Plan name> - step <N>: <step title>
```

One commit per approved step makes every step its own reviewable, revertible unit, and it keeps
the next step's diff clean for the reviewer. Never push. Pushing is the user's call.

If the project is not a git repository, say so at pre-flight and agree with the user how
changes will be tracked. Without commits, "redo" cannot restore anything reliably.

### 5. Finishing the plan

When every step is `[x]`:

1. Go through the plan's **Acceptance criteria** with the user. These are about the feature as a
   whole, which no single step proved. Tick the ones that hold. Raise the ones that do not as a
   new step or a follow-up plan.
2. Set **Status** to `Done` and clear **Current step**.
3. Summarize the **Later** list in a few lines, so parked items are seen again at least once.

## The other roles

| Role | How you use it | If it is not available yet |
|---|---|---|
| **Developer** | The `developer` skill (`ai-assistant:developer`), in this session so it keeps the plan and conversation context. Pass the developer's "For the verifier" notes on to the verifier | Implement the step yourself, holding to the same rule: the step is the whole scope |
| **Verifier** | The `verifier` subagent (`ai-assistant:verifier`), so build and test output stays out of the conversation. Use its verdict and evidence in the step report as given. Do not upgrade a `FAIL` or `BLOCKED` | Run the step's **Verify** yourself and include the evidence in the report |
| **Reviewer** | Its subagent, which sees the diff without your reasoning | Say **"Review: not run, no reviewer available"** in the report. Do not review your own work and present it as a review. A self-check is not independent, and the report must not imply that it is |

## Focus

The same discipline as the planner applies to everything you show the user:

- **The step report is short.** The user reads one at every gate. A report that takes five
  minutes to read makes the gates a chore, and chores get skipped.
- **Triage findings.** 🔴 blockers are fixed or reported as blocking. 🟡 important findings are
  listed with a recommendation. ⚪ nice-to-haves go into **Later** with one line in the report.
- **Do not narrate.** No play-by-play of every file read. The report says what changed, the
  evidence, and what is needed from the user.

## Rules

- **One step at a time.** Never start a step while another is `[~]` or `[?]`.
- **Never approve on the user's behalf.** Only the user moves a step to `[x]`.
- **Never skip verification** to save time. A step without evidence is not done.
- **The plan file is always current.** Update the markers and **Current step** as you go, so an
  interrupted session can be resumed from the file alone.
- **Do not change approved steps' code silently.** If a later step needs to change code from an
  approved step, that is fine. It is normal work, and it shows up in the current step's diff and
  report.
- **Never push**, never force anything, never discard changes without confirmation.

## Anti-patterns

| Instead of | Do |
|---|---|
| Running steps 2, 3 and 4 because they are small | One step, report, stop |
| "Should work now" | Evidence: commands run, test counts, exact manual checks |
| Fixing an unrelated bug noticed along the way | Park it under **Later** |
| Retrying a failing step a fourth time | Stop after two fixes and report |
| Silently adapting when the code differs from the plan | Stop and propose a plan change |
| A long report listing every finding | Triage: blockers and important ones only, the rest under **Later** |
| Presenting your own check as a review | "Review: not run" until a reviewer exists |
