# creating-agent-loops

A Claude Code skill that turns a rough goal into a ready-to-run **agent loop**. It interviews you for the few answers that matter, then hands back a labelled loop spec plus the exact command to launch it.

An agent loop is an LLM that calls tools, checks the result, and repeats toward a stated goal until it is done or you stop it. This skill is a generator, not a catalog: you describe what you want, it builds one to fit.

> **Core principle: a loop is only as good as its "done" check.** Without a real verify gate, the agent grades its own homework and is far too generous, so it produces confident, polished, wrong work and calls it finished. The check is what turns repetition into progress. Everything else is plumbing.

## What it does

When you ask for a loop, the skill walks a fixed process:

1. **Gate** decides whether a loop is even the right tool. Most tasks are not loops; if yours is a predictable one-off, it tells you to just prompt once.
2. **Interview** explores the project first to answer what it can, then asks one thing at a time, each with a recommended answer you can just accept. It pins what "done" means, how it verifies, guardrails, and cadence; everything else gets a stated default.
3. **Classify** the check type (Functional / Visual / Judgment / Human gate) and **derive** the loop shape (Solo / Maker to Checker / Manager to Helpers). A Judgment check always gets a separate scorer.
4. **Pick the mechanism** matched to the stop condition: `/goal` to run until a condition is true, `/loop` for an interval, `/schedule` for an unattended cron, hooks for lifecycle events. It never picks a mechanism that cannot honor the stop.
5. **Emit** a ready loop: GOAL, SUCCESS CRITERIA, LOOP PROTOCOL with a real VERIFY step, STATE, GUARDRAILS, STOP WHEN (success and a hard ceiling), ON STOP, plus the launch command, a cost cap, a stated-assumptions table, and (when a separate checker is used) a ready-to-paste checker brief with a pass and a fail example.
6. **Build order** every time: get one manual run reliable, save it as a skill, wrap it in a loop, then schedule it. Never schedule an unproven loop.

It always includes a hard stop, treats irreversible actions (delete, send, pay, deploy, merge) as a mandatory human gate, and is honest about cost (the metric that matters is cost per accepted change).

## Install

### Manual (works everywhere)

Copy the `creating-agent-loops/` folder from this repo into your skills directory:

- Personal (all projects): `~/.claude/skills/creating-agent-loops/`
- Project-scoped: `<your-project>/.claude/skills/creating-agent-loops/`

```bash
git clone https://github.com/gabrielsilvestri/creating-agent-loops.git
cp -r creating-agent-loops/creating-agent-loops ~/.claude/skills/
```

### Via the skills CLI

```bash
npx skills add gabrielsilvestri/creating-agent-loops --skill creating-agent-loops -g
```

## Use

In Claude Code, just describe a goal:

> "Create a loop that keeps my test coverage above 90%."
>
> "I want a loop that drafts one post a week and improves it until it is good."
>
> "Run a loop every morning that sends me my top priorities."

The skill activates, interviews you, and hands back the loop spec plus the command to launch it. See `creating-agent-loops/templates/worked-examples.md` for three fully worked loops (a Functional coding loop, a Judgment content loop, and a scheduled daily loop).

## What is inside

```
creating-agent-loops/
  SKILL.md                      the process: gate, interview, classify, mechanism, emit, build order
  templates/
    loop-spec.md                the output contract to fill, plus a self-checking variant for any chat
    mechanism-map.md            how to map a loop to /goal, /loop, /schedule, hooks, and sub-agents
    worked-examples.md          three loops built end to end
```

## Credits

The skill distills four sources on agent loops:

- "Agent loops, made simple" (beginner guide): the reason to act to observe skeleton, the four check types, and that a loop is only as good as its done check.
- An audit of 45 real sources on what people mean by "agent loop": the invariant core (model, tools, state, goal, verify, stop) and the beginner build sequence.
- Anatoli Kopadze, "Loops explained": the discover to plan to execute to verify to iterate cycle, the four-condition test for whether to loop, the five building blocks, cost per accepted change, and the build order.
- The Forward-Future [loop-library](https://signals.forwardfuture.ai/loop-library/): the catalog of pre-made loops this generator complements.

## License

MIT. See [LICENSE](LICENSE).
