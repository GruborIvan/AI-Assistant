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
| **Orchestrator** | skill | Runs the plan one step at a time and stops for approval after each | planned |
| **Developer** | skill | Implements exactly one step, in the project's existing style | planned |
| **Verifier** | subagent | Proves a step works: build, tests, running the app | planned |
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
