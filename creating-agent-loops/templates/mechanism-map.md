# Mechanism map (Claude Code)

How to turn a loop spec into something that actually runs. Pick by the stop condition and the cadence. The governing rule: never pick a mechanism that cannot satisfy the stop condition.

## The five building blocks of any loop

1. **Automation (the heartbeat)** the trigger that makes it a loop, not a one-off. `/goal`, `/loop`, hooks, `/schedule`, cron.
2. **Skill (reusable instructions)** save the loop's rules once as a skill the loop reads each run, instead of pasting a wall of text every time.
3. **Sub-agents (maker away from checker)** the single most useful structural trick: split who does the work from who checks it.
4. **Connectors (so it acts, not suggests)** tools/MCP that let the loop open the PR, file the ticket, send the message, not just describe it.
5. **The verifier (the gate)** the check that rejects bad work. The one block that decides whether the loop helps or just burns money.

## Choosing the heartbeat

| You want | Use | Notes |
|---|---|---|
| Keep working in THIS session until a condition is true | `/goal "<condition in plain words>"` | Sets a Stop hook that blocks stopping until the condition holds, then auto-clears. This is the right tool for "run until X is done". Clear early with `/goal clear`. |
| Re-run a prompt or command on an interval | `/loop <interval> <prompt>` e.g. `/loop 30m "<prompt>"`; omit the interval to let the model self-pace | Re-runs every N. Does NOT stop itself when the goal is met, so it is for polling/maintaining, not for until-true goals. Good for "check the deploy every 5 minutes". |
| Run unattended on a daily/weekly cron, results come to you | `/schedule` | Creates a scheduled cloud agent (routine). Also handles one-time future runs ("run this once at 3pm"). |
| Fire commands around the agent's lifecycle (on stop, after an edit, on session start) | hooks in settings.json | Use the update-config skill to add them. Good for: sanitize output on Stop, run tests after edits, wake a persona on SessionStart. |
| Split a big job across workers | sub-agents (the Task/Agent tool), or the Workflow tool | Sub-agents for maker/checker and manager/helpers. The Workflow tool is for deterministic fan-out (loops, conditionals) but is heavier and needs explicit opt-in; reach for it only when one agent genuinely cannot keep up. |

## How they compose

- **`/goal` running a saved skill.** Best pattern for a proven loop: `/goal "<done condition>"` and let the session run the skill you saved in build-order step 2. The goal supplies the stop; the skill supplies the instructions.
- **`/loop` calling a slash command.** `/loop 1h /my-maintenance-skill` re-runs a saved skill on a schedule.
- **Hooks as the gate or the sanitizer.** A Stop hook can run the test suite and refuse to let the session end until it passes (a verify gate enforced by the harness, not trusted to the model). A PostToolUse hook can sanitize written files. Note: a Stop hook blocks the session from ending, but it does not stop `/loop` from re-firing on its interval; for an until-true recurrence, `/goal` is still required.
- **Maker/checker via sub-agents.** Inside any loop (including a `/goal` or `/loop` session), the running agent uses the Task/Agent tool to dispatch the work to one sub-agent and the grading to a SEPARATE sub-agent with different instructions, sometimes a stronger model on higher effort. The writer never grades itself. No special syntax is needed: the parent session spawns both.
- **Sub-agents vs the Workflow tool.** Use plain sub-agents for a handful of pieces or a maker/checker pair. Reach for the Workflow tool only when the fan-out is large (roughly 8 or more parallel pieces) or needs deterministic control flow (loops/conditionals over many items), and the user has opted into that scale.
- **Schedule LAST.** Per the build order, only move to `/schedule` or cron after one manual run is reliable and the loop is wrapped with a gate and a stop.

## Stop-condition sanity check

Before handing over a mechanism, confirm:

- Is there a hard ceiling (max passes / budget / time), not just a success condition? Required, always.
- Does the chosen heartbeat actually honor the success stop? `/goal` does; `/loop` does not (it keeps re-running). If the user wants "until X then stop", that is `/goal`.
- For unattended runs: is there a kill path (operator can stop, schedule has an end, budget caps tokens)? An unattended loop with no model-side stop runs until something external kills it.
