# Worked examples

Three loops built with this skill, end to end. They show the gate, the check type, the shape, the mechanism, and the filled spec. Use them as models, not as fill-in templates.

These specs are written in English to document the skill. When you deliver to a user, write the whole spec in the user's language; keep only command tokens (`/goal`, `/loop`, file paths) verbatim.

For brevity these examples show the core spec; a real delivery also carries the post-spec elements from the loop-spec template: the launch line with why the mechanism honors the stop, the stated-assumptions table (assumption, default, how to correct), and, for Maker -> Checker, the ready-to-paste checker brief with one pass example and one fail example.

---

## Example 1 - Functional, Maker -> Checker, run-until-true (`/goal`)

**User:** "Quero um loop que mantenha a cobertura de testes acima de 90%."

**Gate:** repeats (yes, ongoing), auto-rejectable (yes, coverage % is a hard number), end-to-end (yes), objective done (yes, 90%). All four pass.

**Check type:** Functional (a number). **Shape:** Maker -> Checker, because the same agent writing tests and judging coverage can game it (trivial asserts that lift the number without exercising code). A separate checker catches that.

**Mechanism:** `/goal`, because the stop is "until coverage >= 90%". (Not `/loop`, which would keep re-running after the goal is met.)

```
GOAL: line coverage of the test suite is at or above 90%, with every new test
      genuinely exercising the code it covers.

SUCCESS CRITERIA (strict, no soft passes):
  - the coverage tool reports total line coverage >= 90%
  - the full suite passes (no failing or skipped tests)
  - no test was deleted or weakened to lift the number

LOOP PROTOCOL (each pass):
  1. run the coverage tool, read the total and the least-covered file
  2. write tests for the real uncovered branches in ONE file
  3. VERIFY: re-run coverage and the suite. Then a SEPARATE checker agent reads
            the new tests and confirms each asserts real behavior (not a trivial
            pass). If the checker rejects any test, that test does not count.
  4. DECIDE: coverage >= 90% AND suite green AND checker approves all new tests?
            stop. else iterate on the next least-covered file.

STATE: append one line to PROGRESS.md each pass (file treated, coverage delta)
       and commit.

GUARDRAILS:
  - must NOT touch: anything outside /src and /tests
  - one file per pass
  - never delete or skip a test to make coverage pass

STOP WHEN: criteria met OR after 12 passes, whichever comes first
ON STOP: report final coverage, files treated, and any file left uncovered with why
```

- **Launch:** `/goal "test line coverage is >= 90% and the suite is green and no tests were weakened"` then let the session run this spec.
- **Check type / shape:** Functional / Maker -> Checker
- **Cost cap:** max 12 passes (estimate ~8 passes x 1.5; raised above the maker/checker default of 8 because each pass is a cheap test run); checker doubles per-pass cost; watch the first run

---

## Example 2 - Judgment, Maker -> Checker, weekly (`/loop` or `/schedule`)

**User:** "Quero um loop que escreva um post por semana e melhore o texto ate ficar bom."

**Gate:** repeats (weekly, yes), auto-rejectable (only via a rubric, so borderline), end-to-end (yes), objective done (NO, "good writing" is taste). Because "done" is taste, this is the Judgment path: it needs a written rubric and a separate scoring agent, otherwise it has no real gate.

**Check type:** Judgment. **Shape:** Maker -> Checker (the scorer must be a different agent).

**Mechanism (two distinct loops, do not conflate):** the OUTER heartbeat is cadence only, `/loop 1w` (or `/schedule` if unattended); it re-fires weekly and is NOT a stop condition. The INNER quality loop runs inside each weekly fire and DOES stop itself when every rubric score clears the bar. `/loop` is correct here precisely because the outer trigger is a recurring heartbeat, not an until-true goal.

```
GOAL: a publish-ready post that scores at least 8/10 on every rubric line.

SUCCESS CRITERIA (strict, no soft passes):
  - hook in the first sentence (no throat-clearing)
  - one clear idea, not three half-ideas
  - concrete example or number, not just claims
  - ends with a single specific takeaway
  - reads in the user's voice, not generic AI cadence

LOOP PROTOCOL (each pass):
  1. draft or revise the post
  2. (no self-grading) hand the draft to a SEPARATE checker agent
  3. VERIFY: the checker scores each criterion 1-10 against the rubric and names
            the single weakest line with a concrete reason
  4. DECIDE: every criterion >= 8? stop and present. else revise, fixing the
            weakest criterion first
            (this stops the INNER revision loop only; the weekly /loop
             heartbeat keeps firing on its cadence regardless)

STATE: keep each draft + its scores in a notes file so revisions do not regress

GUARDRAILS:
  - must NOT publish or post anywhere; output the draft only
  - pause and ask before sending to any channel

STOP WHEN: all criteria >= 8 OR after 6 revision passes (then present the best draft)
ON STOP: present the post and its final rubric scores
```

- **Launch:** weekly cadence, not interchangeable: `/loop 1w "<this spec>"` if you will be around to stop it, or `/schedule` if it must run unattended; the inner revise-until-8 loop runs inside each weekly fire.
- **Check type / shape:** Judgment / Maker -> Checker
- **Cost cap:** max 6 revision passes per week; separate scorer doubles per-pass cost

---

## Example 3 - Scheduled, with a quality gate, daily (`/schedule`)

**User:** "Quero um resumo de prioridades todo dia de manha, das minhas tarefas e do email."

**Gate:** repeats (daily, yes), end-to-end (yes), auto-rejectable + objective done are the weak points: a "good briefing" is judgment. So this is NOT a pure automation; it needs a self-check rubric before it sends, or it becomes a scheduled prompt with no gate (the most common failure for life loops).

**Check type:** Judgment (light rubric) before delivery. **Shape:** Solo with a self-grade pass.

**Mechanism:** `/schedule` for the daily cron. BUT per the build order, prove ONE manual run first; only schedule after it reliably produces a briefing you would actually act on.

```
GOAL: a daily briefing under 120 words that surfaces the 3 things that actually
      matter today, pulled from tasks and email.

SUCCESS CRITERIA (strict, no soft passes):
  - names the single most important focus for the day
  - lists only items that are due today or genuinely urgent (no filler)
  - flags anything with an external deadline or dependency
  - under 120 words, no greeting, straight into the briefing

LOOP PROTOCOL (each run):
  1. pull today's tasks and unread important email via the connectors
  2. draft the briefing in the format above
  3. VERIFY: score the draft against the criteria; if any item is filler or the
            word count is over, cut and re-tighten
  4. DECIDE: all criteria met? deliver. else tighten once more (max 3 tightenings)

STATE: not needed (each day is independent), but keep yesterday's briefing to
       avoid repeating stale items

GUARDRAILS:
  - read-only on tasks and email; must NOT reply, archive, or complete anything
  - deliver as a notification only

STOP WHEN: a briefing meeting all criteria is delivered, OR 3 tightenings reached
ON STOP: deliver the best briefing and note any source that failed to load
```

- **Launch:** prove it once by hand, then `/schedule` it for weekday mornings.
- **Check type / shape:** Judgment (light, self-graded) / Solo. Honest caveat: a self-grade by the same agent overgrades; this is a pragmatic compromise for a low-stakes daily note, not a true Judgment check. For higher stakes, add a separate scorer (Maker -> Checker).
- **Cost cap:** one run/day, max 3 tightenings; cheap. Watch the first week.

**The point of step 3:** without it this is just a scheduled prompt that sends whatever it produced; the self-grade is what makes it a loop, even though (per the caveat above) it is not a true Judgment check.
