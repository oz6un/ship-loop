# ship-loop

A [Claude Code](https://code.claude.com) skill: a disciplined loop for taking a **substantial
code change from request to merged-and-verified** — research → converge → reproduce → pin with a
test → implement → adversarially self-audit → verify with the project's own gates → PR →
wait-for-green → merge → confirm it landed.

It is deliberately opinionated and tuned for capable (Opus-class) models: it contains only the
disciplines a strong model does *unreliably by default* — the "teeth" — not generic engineering
etiquette. It's **project-agnostic**: it teaches discovering and running *your* repo's own gates
rather than hardcoding any toolchain.

> Honest caveats: the name under-sells the audit (which is the sharpest part), and this is an
> opinionated distillation of one engineer's implement→audit→ship workflow, not a neutral
> standard. Scale it to the task — a one-line fix skips most of it.

## The loop

1. **Map the blast radius; research the approach** — grep every caller, enumerate every schema
   reference, and research the SOTA / prior art before committing to a first instinct.
2. **Converge on the optimal solution before writing code** — the simplest fix at the root
   cause; default to a reasoned decision, not a menu of questions.
3. **Reproduce before you theorize** — measure the real values; don't guess.
4. **Pin the change with a test** (red→green), then implement to the grain.
5. **Verify with the project's own gates**, cheap-first (typecheck/lint before the slow e2e).
6. **Audit adversarially** with independent refute-reviewers — the gate.
7. **Ship on the user's cadence** — branch → PR → wait-for-*actually*-green → merge → sync.
8. **Confirm it landed; record the one footgun** a future session could re-introduce.

Two references load on demand:
- [`references/audit.md`](references/audit.md) — the adversarial self-audit playbook (refute
  angles, CONFIRMED/PLAUSIBLE/REFUTED, a stop condition, fix-vs-accept by proportion).
- [`references/ship.md`](references/ship.md) — project-agnostic ship mechanics (gate discovery,
  a PR template, failing-test triage, generated-artifact-drift, verify-after-merge).

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
