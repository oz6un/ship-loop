# ship-loop

A [Claude Code](https://code.claude.com) skill: a disciplined loop for taking a **substantial code
change from request to merged-and-verified** — research → converge → reproduce → pin with a test →
implement → try to break it → verify with the project's own gates → PR → wait-for-green → merge →
confirm it landed.

Its one organizing idea is **make a claim, then try to break it** — Karl Popper's *conjecture and
refutation*, pointed at shipping code. Reproduce the bug instead of theorizing about it; watch a
test fail before you trust it passing; hand the change to reviewers whose only job is to break it.
What survives a real attempt to break it is what you ship.

It's deliberately opinionated and tuned for capable (Opus-class) models: it contains only the
disciplines a strong model does *unreliably by default*, not general engineering etiquette. And
it's **project-agnostic** — it teaches you to discover and run *your* repo's own gates rather than
hardcoding any toolchain.

> Honest caveats: the name under-sells the audit (which is the sharpest part), and this is an
> opinionated distillation of one engineer's implement→audit→ship workflow, not a neutral
> standard. Scale it to the task — a one-line fix skips most of it.

## The loop

1. **Find everything the change touches; research the approach** — grep every caller, enumerate
   every schema reference, and look at how mature tools solve this before trusting your first
   instinct.
2. **Decide the best solution before writing code** — the simplest fix at the root cause; make the
   call yourself instead of handing over a menu of questions.
3. **Reproduce before you theorize** — measure the real values; don't guess.
4. **Pin the change with a test** (watch it fail, then pass), then implement to match the code
   around it.
5. **Verify with the project's own gates**, cheapest first (typecheck and lint before the slow
   end-to-end suite).
6. **Try to break your own change** with independent refute-reviewers — the gate. If the change is
   a *control* (its worth is a guarantee against someone you can name), also break its **design**,
   not just its code: a clean pass from same-framed refuters validates the framing, not the design.
7. **Ship on the user's cadence** — branch → PR → wait for CI to *actually* go green → merge → sync.
8. **Confirm it landed; record the one trap** a future session could fall back into.

Three references load on demand:
- [`references/audit.md`](references/audit.md) — the playbook for breaking your own change (refute
  angles, CONFIRMED/PLAUSIBLE/REFUTED, when to stop, fix-vs-accept by proportion).
- [`references/control-review.md`](references/control-review.md) — the design-level review for a
  *control*: does it trust the right input, can they turn it off, is every path covered, does it
  check the real thing or a
  stand-in, what does it cost, is a defeat visible. Grounded in Saltzer & Schroeder / the reference
  monitor / STRIDE and framed as falsification — the design flaws that breaking the diff
  line-by-line misses.
- [`references/ship.md`](references/ship.md) — project-agnostic ship mechanics (finding the gates,
  a PR template, failing-test triage, generated-artifact drift, verifying after merge).

## Install

```bash
git clone https://github.com/oz6un/ship-loop ~/.claude/skills/ship-loop
```

Or copy `SKILL.md` and `references/` into `~/.claude/skills/ship-loop/` (personal, all projects)
or `.claude/skills/ship-loop/` (one project). Claude Code discovers it automatically; the extra
`README.md`/`LICENSE` are ignored by the skill loader.

## Use

It auto-triggers when you ask for a non-trivial change to be built carefully **and** shipped, or
ask to audit / harden / verify a change — or invoke it explicitly with `/ship-loop`. It is not
for quick edits, questions, or read-only exploration.

## License

[MIT](LICENSE).
