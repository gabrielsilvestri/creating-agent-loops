# Loop spec template

Fill every slot. Delete the bracketed hints. Keep the labels and the order. Write it in the user's language. No em dashes or en dashes.

```
GOAL: [one checkable sentence. "Every test in /tests passes when I run npm test", not "improve the tests"]

SUCCESS CRITERIA (strict, no soft passes):
  - [criterion a machine or rubric can check]
  - [criterion]
  - [criterion]

LOOP PROTOCOL (repeat each pass):
  1. [gather context / state the single next step]
  2. [do ONE thing, the smallest change that moves toward the goal]
  3. VERIFY: [the concrete check that matches the check type]
            [if it fails, name exactly what is still weak]
  4. DECIDE: [all criteria pass? print done and stop.
             else iterate, fixing the weakest point first]

STATE: [write one line to PROGRESS.md each pass and commit to git, so the next
        pass resumes instead of restarting. Skip ONLY if the loop is at most
        3 passes AND fits one context window; otherwise this is mandatory.]

GUARDRAILS:
  - must NOT touch: [files/dirs/systems out of scope]
  - one change per pass (do not batch unrelated fixes)
  - pause and ask before: [irreversible action: delete, send, pay, deploy, merge]

STOP WHEN: [success condition] OR [hard ceiling: e.g. after 8 passes / token
           budget / time limit], whichever comes first

ON STOP: [summarize what changed and what still fails. If the ceiling was hit
         without success, partial work stays committed and the loop is
         restartable from the last PROGRESS.md entry: say what is left and how
         to resume, do not discard progress.]
```

After the spec, always add:

- **Launch:** `[the exact command, e.g. /goal "..." or /loop 30m "..." or /schedule ...]` plus one line on why this mechanism honors the stop condition
- **Check type / shape:** `[Functional|Visual|Judgment|Human gate] / [Solo|Maker->Checker|Manager->Helpers]`
- **Cost cap:** `[the hard ceiling restated, e.g. "max 8 passes; watch the first run"]`
- **Assumptions:** a small table for everything you inferred or could not confirm, so the user can correct a wrong guess in one read:

  | Assumption | Default chosen | How to correct |
  |---|---|---|
  | [what you assumed] | [the value used] | [what to tell you to change it] |

- **Checker brief** (only when shape is Maker -> Checker): the exact rubric the separate scorer applies, written ready to paste, with at least one concrete example of a passing result and one of a failing result, so the gate can actually fail bad work.

---

## Self-checking variant (run by hand in any chat, no coding agent)

When the user has no agent tooling and just wants to paste a loop into a chat.

Caveat to give the user: this variant has NO maker/checker separation, the same model produces and grades. For Functional criteria (word count, a present keyword) that is fine. For Judgment criteria it will systematically overgrade, so treat its output as a strong draft, not a verified result; for real Judgment stakes use a separate scoring agent instead.

```
You will work in a loop until the task meets the bar. Do not ask me questions;
make a sensible assumption, note it, and keep going.

TASK:
[describe exactly what you want produced]

SUCCESS CRITERIA (be strict, no soft passes):
- [criterion 1]
- [criterion 2]
- [criterion 3]

LOOP PROTOCOL, repeat every turn:
1. PLAN   - state the single next step.
2. DO     - produce or improve the work.
3. VERIFY - score the result 1-10 on each criterion against an EXTERNAL
            standard (a concrete example of a 10, or what a named reader would
            demand), NOT against your own previous draft. Be brutally honest.
            List exactly what is still weak. NOTE: for any Judgment criterion
            this self-score is unreliable (you are grading your own work);
            treat the result as a strong draft, not a verified pass.
4. DECIDE - if every criterion is 8+, print "FINAL" and stop.
            Otherwise print "ITERATING" and go again, fixing the weakest first.

Begin. Run the loop until FINAL.
```
