---
name: ship-loop
description: The end-to-end rigor loop for delivering a substantial change you can stand behind — research the approach, converge on the simplest root-cause solution, pin it with a test, implement, adversarially self-audit with independent refute-reviewers, verify with the project's own gates, then branch → PR → wait-for-green → merge → confirm it landed. Use when the user invokes /ship-loop, asks you to audit / harden / verify a change, or asks for a non-trivial change to be done carefully AND shipped (opened as a PR or merged). Not for quick edits, questions, or read-only exploration.
---

# Ship-loop

One idea runs through the whole loop, from request to merged: **make a claim, then try to break
it.**

At every step you'll be tempted to just assert something — "this is the cause," "this fixes it,"
"this can't be bypassed," "it's clean." An assertion you haven't tried to disprove is only a
confident guess. So each step below turns a claim into something you could actually watch fail:
you reproduce the bug instead of theorizing about it, you watch a test fail before you trust it
passing, you hand the change to reviewers whose only job is to break it. What survives a real
attempt to break it, you can ship. What you merely asserted, you can't — however sure you feel.
(This is Karl Popper's *conjecture and refutation*, pointed at shipping code: a claim earns trust
only by surviving a genuine attempt to prove it wrong.)

**Scale it to the task** — a one-line fix skips most of this. What's here is the part a capable
model does *unreliably by default*: the disciplines it tends to skip, not general good manners.
Two things never bend. A substantial change isn't done until it passes the project's **own gates**
*and* survives a real attempt to break it. And push, PR, and merge happen on the **user's
cadence**, never by assumption.

## The loop — go back a step whenever you need to; the audit sends you back to implement

1. **Find everything the change touches, and research the approach.** Before you touch anything
   *shared*, list everything that depends on it — `rg`/`grep` the symbol for every call site,
   enumerate every schema column or table that references it, and follow the data end to end. The
   worst bugs hide in a caller you didn't know was there. Separately, for any real design or
   architecture decision, **look at how mature tools already solve this** before committing to
   your first instinct.

2. **Decide the best solution before you write code.** Work out what *should* happen, then take
   the **simplest fix at the root cause** — the one shared source every consumer reads, not a
   separate patch in each place it surfaces. Don't over-build for a rare edge. Name the one real
   trade-off your choice makes. **Make the call yourself instead of handing the user a menu of
   questions** — that judgment is the value you add. Ask only on a genuine either/or that's truly
   theirs to decide, and once they've said "use your judgment," stop asking and decide.

3. **Reproduce the problem before you explain it.** For a reported bug, *measure* first: compute
   the actual values, run the real function, print the real state with a throwaway `node -e` or
   script. Don't theorize from reading the code. This is the first place "make a claim, then break
   it" bites — the instrument usually names the cause, and a guess usually doesn't.

4. **Pin the change with a test, then write the fix.** Turn the repro into a test and **watch it
   fail first.** A test you never saw fail may be pinning nothing. An expected value you pasted
   from the buggy run — instead of one you worked out independently — will pass no matter what the
   code does. A test that still passes after you undo the fix is theater; and if its two branches
   don't force *different* results, it isn't pinning your claim at all. Now make it pass: implement
   the fix by following the nearest existing pattern in the codebase, and keep the change to **one
   reviewable idea**. If the bug is really a *pattern*, `grep` for its siblings — the twin
   function, the other caller, the same shape elsewhere — because fixing one of five identical
   sites is a half-fix. A change with no test that goes from failing to passing across it is
   asserted, not proven.

5. **Verify with the project's OWN gates.** Run the full set locally, **cheapest first** —
   typecheck and lint before the slow end-to-end suite — across the whole workspace, not just the
   file you edited. How to find out what a repo's gates are: **[references/ship.md](references/ship.md)**.
   A green local run is required before any push.

6. **Try to break your own change — this is the gate, not a formality.** Spawn independent
   reviewers whose job is to **refute** the diff, not admire it; keep the findings that survive;
   decide fix-or-accept by proportion; go back to step 4 when a finding is real. **One kind of
   change needs more than this: a *control*** — a change whose worth is a *guarantee* against
   someone you can name ("X can't happen," and you can say who would want it to and what is lost
   when it does). For a control, refuting the diff isn't enough: reviewers all hunting line-by-line
   bugs can come back clean while the *design* is broken. Audit the design directly:
   **[references/control-review.md](references/control-review.md)**. For the refute angles, the
   rule that a doc or PR line overstating the code is itself a defect, when to stop, and how to
   settle disagreements between reviewers: **[references/audit.md](references/audit.md)**.

7. **Ship on the user's cadence.** branch → commit (let the hooks run) → open the PR → **wait for
   CI to *actually* go green** → merge (squash and delete the branch) → fast-forward local `main`.
   A red check is often just **generated-artifact drift** — a stale snapshot, lockfile, OpenAPI
   spec, or generated types file — which you fix by re-running the project's generator, **never by
   hand-editing** the output. Merge only when you're asked or clearly authorized. PR template, what
   to do about a failing test, verifying after merge: **[references/ship.md](references/ship.md)**.

8. **Confirm it landed, and record the one footgun.** "Shipped" is not "works in production" —
   check the deploy and the live behavior. Then write down the non-obvious decision and the single
   trap a future session could fall back into, in the project's memory or log — skipping anything
   the code, tests, and git history already make obvious.
