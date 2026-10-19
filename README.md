# AI Assistant

A role-based development workflow for [Claude Code](https://claude.com/claude-code), packaged as
a plugin. Instead of one long conversation that plans, codes and checks its own work, each job
has a dedicated role, and **a human approves every step** before the next one starts.

```
Idea ──▶ PLAN ──▶ [you approve] ──▶ BUILD one step ──▶ VERIFY ──▶ REVIEW ──▶ [you approve] ──▶ next step
```

The roles hand work to each other through a **plan file** in the project
(`docs/plans/YYYY-MM-DD-<slug>.md`), not through chat. A plan can be resumed in a new session,
reviewed by a person, and checked against what was actually built.

## Roles

| Role | Type | Job | Status |
|---|---|---|---|
| **Planner** | skill | Leads a focused requirements interview and writes a plan of small, verifiable steps | ✅ available |
| **Orchestrator** | skill | Runs the plan one step at a time and stops for approval after each | ✅ available |
| **Developer** | skill | Implements exactly one step, in the project's existing style | planned |
| **Verifier** | subagent | Proves a step works: build, tests, running the app | ✅ available |
| **Reviewer** | subagent | Reviews each step's diff against the plan with fresh eyes | planned |

The orchestrator and developer run in your main session because they need its context. The
verifier and reviewer run as subagents: the verifier keeps noisy build and test output out of the
conversation, and the reviewer judges the code without the author's reasoning in its head.

## The planner

The planner leads the conversation instead of waiting for instructions:

- **Reads before asking.** It never asks what the code can answer.
- **Covers what matters.** It works through a checklist (problem, users, acceptance criteria,
  scope, constraints, data, failure modes, quality bar) until each item is known or explicitly
  assumed.
- **Stays focused.** It asks only questions whose answer would change what gets built, 1-3 per
  round, each with a recommended answer.
- **Parks distractions.** Details that don't matter yet go into a *Later* section instead of the
  conversation.
- **Triages concerns.** Blockers are raised now, important issues come with a recommendation, and
  nice-to-haves are parked.

Its output follows [the plan template](skills/planner/templates/plan.md).

## The orchestrator

The orchestrator runs an approved plan and never gets ahead of you:

- **One step at a time.** It implements a step, verifies it with evidence, has it reviewed, then
  stops and presents a short [step report](skills/orchestrator/templates/step-report.md).
- **You approve every step.** Approving commits that step (one commit per step, never pushed)
  and starts the next. You can also ask for fixes, redo the step, or change the plan.
- **The plan file is its memory.** Step markers (`[ ]` `[~]` `[?]` `[x]`) and an approvals log
  are kept current, so an interrupted session resumes exactly where it stopped.
- **It stays honest about what it checked.** Checks only a person can do (a desktop UI, a
  printout) appear as exact manual instructions. Until the reviewer role exists, the report
  says the review was not run, instead of passing off a self-check as a review.
- **It stops instead of improvising.** If the code doesn't match what the plan assumed, or a fix
  fails twice, it reports back rather than building something you didn't approve.

Start it with `/ai-assistant:orchestrator`, or say "start the plan" once a plan is approved.

## The verifier

The verifier is a subagent the orchestrator calls after each step, and once before a plan starts
to check the project is healthy. Its build and test output stays out of your conversation, and
it can't edit code: it only runs things and reports.

- **Evidence, not opinions.** It reports build results, test counts, and the actual values the
  step's checks returned, not just "200 OK".
- **Three verdicts.** `PASS`, `FAIL` (the code is wrong, so it goes back for a fix), or
  `BLOCKED` (something outside the code, like a database that's down, which goes to you).
- **No quiet substitutions.** A check it can't run is reported as not run, never swapped for an
  easier one without saying so.
- **Knows how builds lie.** A locked executable can make tests run against the old binary, and a
  filter in a pipe can hide a failing exit code. It checks for both.
- **Leaves human checks to humans.** Desktop UI, printouts and devices come back as exact
  instructions for you.

It reads the project's build, test and run commands from `CLAUDE.md`, so it works for any stack.

## Installation

In Claude Code, add this repository as a plugin marketplace and install the plugin:

```
/plugin marketplace add GruborIvan/AI-Assistant
/plugin install ai-assistant@ai-assistant
```

Pull later changes with `/plugin marketplace update ai-assistant`.

To try a local copy for a single session without installing (useful while changing the plugin):

```bash
claude --plugin-dir path/to/AI-Assistant
```

Then describe a feature you want to build, or invoke the planner directly with
`/ai-assistant:planner`.

## Contributing

The project is at an early stage and the roles are being built one at a time. Issues with
feedback from real use are especially welcome: a question the planner should not have asked, one
it missed, or a step that was too large to review.

## License

[MIT](LICENSE)
