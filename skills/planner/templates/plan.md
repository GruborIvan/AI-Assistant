# Plan: <Feature name>

**Status:** Draft <!-- Draft | Approved | In progress | Done | Abandoned -->
**Created:** YYYY-MM-DD
**Current step:** — <!-- set by the orchestrator, e.g. "3 - Add validation" -->

<!--
This file is the contract between the roles. The planner writes it, the user approves it, the
orchestrator runs it one step at a time, and the verifier and reviewer check each step against
it. Keep it current: if reality differs from the plan, change the plan before changing the code.
-->

## Goal

<Two or three sentences: what problem this solves, for whom, and why now.>

## Users and context

<Who uses the result and in what situation. This decides trade-offs like speed vs completeness.>

## Acceptance criteria

<How we will know the feature works, as a whole. Each item checkable by a person.>

- [ ] <...>
- [ ] <...>

## Scope

**In:**

- <...>

**Out (explicitly not in this plan):**

- <...>

## Constraints

<Existing code to fit, technology, things that must not change, deadlines. Link to CLAUDE.md
sections instead of repeating them.>

## Assumptions

<Everything decided without an explicit answer from the user. Each one can be overturned.>

| # | Assumption | Why this default |
|---|---|---|
| A1 | <...> | <...> |

## Risks and concerns

| Level | Concern | Decision |
|---|---|---|
| 🔴 / 🟡 | <...> | <what we decided, or "open"> |

## Steps

<!--
Status markers, updated by the orchestrator:
  [ ] not started   [~] in progress   [?] waiting for human approval   [x] approved
Every step leaves the application building and its existing tests passing.
-->

### [ ] 1. <Step title>

- **Goal:** <What this step achieves, in one sentence.>
- **Changes:** <Files or areas likely touched. A guide, not a promise.>
- **Verify:** <Exact commands, tests, or manual check with expected result.>
- **Done when:** <The condition a reviewer can check.>

### [ ] 2. <Step title>

- **Goal:**
- **Changes:**
- **Verify:**
- **Done when:**

## Open questions

<Unanswered questions that do not block starting. Note which step needs each answer.>

- [ ] <Question> — needed before step <N>

## Later

<Parked items: real ideas, not for this plan. Raised during planning or while building.>

- <...>

## Approvals

<Every human gate, in order. Written by the planner (plan approval) and the orchestrator (step
approvals).>

| Date | What | Decision | Notes |
|---|---|---|---|
| YYYY-MM-DD | Plan | Approved | <...> |
