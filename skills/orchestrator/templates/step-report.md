<!--
The report the orchestrator presents at every gate. Keep it readable in about a minute.
Omit a section only when it is genuinely empty, and say "none" rather than leaving the reader
to wonder. Replace everything in <angle brackets>.
-->

## Step <N> of <total>: <step title>

**Status:** ✅ ready for your approval <!-- or: ❌ blocked, needs your decision -->

**What changed**
- `<file>`: <one line on what and why>
- `<file>`: <...>

**Verification:** <PASS | FAIL | BLOCKED, as the verifier reported it>
- <command or check>: <result with evidence, e.g. "build OK", "18/18 tests passed">
- <...>

**Needs your manual check** <!-- omit if everything could be checked automatically -->
- <What to open, what to do, what to expect.>

**Review**
- 🔴 <blocker, and whether it was fixed> <!-- or: "Review: not run, no reviewer available" -->
- 🟡 <important finding> — recommendation: <...>
- No other findings. <!-- or: "⚪ <n> minor items parked under Later" -->

**Deviations from the plan** <!-- omit if none -->
- <What differs from the step as written, and why.>

**Plan:** step <N> marked waiting for approval in `<plan file>`.

**Next:** Step <N+1>: <title>

**Your call:** approve (commits this step and starts the next) · approve and stop here · fix
something · redo · change the plan
