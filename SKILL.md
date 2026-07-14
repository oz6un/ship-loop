---
name: ship-loop
description: The end-to-end rigor loop for delivering a substantial change you can stand behind — research the approach, converge on the simplest root-cause solution, pin it with a test, implement, adversarially self-audit with independent refute-reviewers, verify with the project's own gates, then branch → PR → wait-for-green → merge → confirm it landed. Use when the user invokes /ship-loop, asks you to audit / harden / verify a change, or asks for a non-trivial change to be done carefully AND shipped (opened as a PR or merged). Not for quick edits, questions, or read-only exploration.
---

# Ship-loop

The loop for delivering a substantial change with rigor. **Scale to the task** — a one-liner
skips most of this. Most of what follows is the part a capable model does *unreliably by
default* — the teeth, not the etiquette. Two things never bend: a substantial change isn't done
until it passes the project's **own gates** and survives an **adversarial pass at breaking it**;
and push/PR/merge happen on the **user's cadence**, never by assumption.

## The loop — loop back freely; the audit reopens implement

1. **Map the blast radius; research the approach.** Before touching anything *shared*, find
   every dependent — `rg`/`grep` the symbol for all call sites, enumerate every schema
   column/table that references it, trace the data end to end; the worst bugs hide here.
   Separately, for any non-trivial design or architecture call, **research the SOTA / prior art
   first** — how do mature tools actually solve this? — instead of shipping your first instinct.

2. **Converge on the optimal solution *before* writing code.** Decide what *should* happen,
   then take the **simplest** fix at the **root cause** — the shared source every consumer
   reads, not a patch per view. Don't gold-plate a rare edge. Name the one real trade-off.
   **Default to a reasoned decision, not a menu of questions** — that judgment is the value; ask
   only on a genuine either/or that's the user's alone, and once they've said "use your
   judgment," stop asking and decide.

3. **Reproduce before you theorize.** For a reported bug, *measure* — compute the actual
   values, run the real function, print the real state (a throwaway `node -e`/script) — before
   hypothesizing. The instrument usually names the cause; a guess usually doesn't.

4. **Pin the change with a test, then implement to the grain.** Turn the repro into a test that
   **fails now** — *watch it go red*: a test you never saw fail may pin nothing, and an expected
   value pasted from the run (rather than one you reproduced independently) passes by
   construction. A test that still passes with the fix reverted is theater; one whose two
   branches don't force *different* outcomes doesn't pin the claim. Make it pass by implementing
   the fix — mirror the nearest existing pattern (the analogous helper/cascade), and keep the
   diff to **one reviewable concern**. When the fix is a bug *pattern*, **`grep` for its
   siblings** (the twin function, the other caller, the same call shape) — a fix applied to one
   of N identical sites is a half-fix. A change with no test that goes red→green across it is
   asserted, not proven.

5. **Verify with the project's OWN gates.** Run the FULL set locally, **cheap-first** —
   typecheck + lint before the slow e2e — over the whole workspace, not just the file you
   touched. How to discover a repo's gates: **[references/ship.md](references/ship.md)**. A
   green local run is required before any push.

6. **Audit adversarially — the gate, not a formality.** Spawn independent subagents told to
   **refute** your diff; keep what survives; decide fix-vs-accept by proportion; loop back to
   step 4 when it's real. **If the change is a *control* — its worth is a guarantee against an
   adversary you can name — refuting the diff isn't enough; audit its *design* (a clean pass from
   same-framed refuters validates the frame, not the design):
   [references/control-review.md](references/control-review.md).** The refute angles, the
   claim-fidelity rule (a doc/PR line that overstates the code is a defect), when to stop, how to
   reconcile conflicts: **[references/audit.md](references/audit.md)**.

7. **Ship — on the user's cadence.** branch → commit (let the hooks run) → PR → **wait for CI
   to *actually* go green** → merge (squash + delete branch) → fast-forward local `main`. A red
   check is often **generated-artifact drift** (snapshot / lockfile / OpenAPI / generated
   types) → regenerate with the project's generator, **never hand-edit** it. Merge only when
   asked or clearly authorized. PR template, failing-test triage, verify-after-merge:
   **[references/ship.md](references/ship.md)**.

8. **Confirm it landed; record the footgun.** "Shipped" ≠ "works in prod" — verify the deploy
   and the behavior. Then log the non-obvious decision and the one footgun a future session
   could re-introduce, in the project's memory/log — skip what the code, tests, and git history
   already say.
