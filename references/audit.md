# The adversarial self-audit

The gate. Its job is to *break* the change before a user or production does. Run it on
substantial changes before calling them done, and again whenever the user says "audit,"
"make sure," "harden," or "check it carefully."

## Stance

- **Refute, don't confirm.** Hunt the input, state, timing, or platform that makes it wrong.
- **Your own claims are in scope.** A comment, doc, or PR line that says more than the code
  does is a real defect — verify every "matches / always / can't happen" you wrote.
- **Independent eyes for large or subtle changes** — spawn subagents that don't share your
  rationalizations. For a small, self-contained change, a careful read against the spec is
  proportionate; match the audit's weight to the change.

## Run it

1. **Get the exact diff** (`git show <sha>`, `git diff main...HEAD`, or the working tree).
2. **Fan out reviewers, each with one angle, each told to REFUTE before reporting:**
   - **Line-by-line** on every changed hunk *and its enclosing function* (a bug on an
     unchanged line the change re-exposes still counts).
   - **Removed-behavior**: for each deleted/changed line, name the invariant it enforced and
     find where it's re-established; if you can't, that's a finding.
   - **Caller impact**: does a changed signature/return-shape/precondition break a call site?
   - **Concurrency**: construct the exact interleaving of two concurrent actors. A
     read-then-write without a transaction is a TOCTOU — walk it explicitly.
   - **Cascade/cleanup completeness**: for a delete or migration, enumerate *every* row/table
     keyed to the thing and check each. A fix that trades one dangling row for another is not
     a fix.
   - **Claim-fidelity**: does every comment/doc/PR sentence match the code?
   - **Control-design** (whenever the change *claims* to prevent / guarantee / enforce /
     validate — a check, floor, guard, validator, rate-limit, invariant): does it enforce a
     value the adversary supplies (**trust boundary**), can the adversary *induce* the dependency
     failure it skips on (**fail-open bypass**), does it check a spoofable proxy instead of the
     property (**balance-delta ≠ provenance; 200 ≠ success; sim ≠ execution**), and what does it
     **cost** on the hot path (added RPC / false-rejects)? Line-by-line refute cannot see these —
     they are *design* flaws, not diff flaws: **[control-review.md](control-review.md)**.
3. **Verify each candidate, recall-biased**: CONFIRMED / PLAUSIBLE / REFUTED against the real
   code. Refute only when constructible — impossible (a constant/type/guard, shown) or
   factually wrong (quote the line). A realistic-but-rare path (error handler, cold cache,
   falsy-zero, boundary off-by-one, retry storm) is PLAUSIBLE, not refuted.
4. **Severity by realistic likelihood × blast radius, and factor the platform.** Where agents
   act concurrently, a "two actors race" is realistic, not exotic; a two-human race on a niche
   surface may be negligible. Say which and why.

## Stop condition + conflicts

- **Stop** when a round surfaces nothing new, or when refutation is just re-quoting the same
  guards. An adversarial loop with no bound spins forever — two dry rounds is done. But for a
  **control**, two dry refute-the-diff rounds is *not* the end: the design review
  ([control-review.md](control-review.md)) must also pass, and a same-framed "nothing found" is
  not validation of the design.
- **Reconcile conflicts by constructibility, not vote.** A REFUTE that quotes an actual
  line/constant beats a vague CONFIRM; a CONFIRM with a concrete failing input beats a
  hand-wave. When two reviewers disagree, build the case from the code yourself and decide.

## Fix vs. accept — by proportion

- **Fix now** when it's correctness-affecting on a reachable path, or the fix is cheap and
  clearly right (there's often an elegant one you missed — e.g. reordering a read-then-write
  to *write-then-recheck* closes a race with no transaction).
- **Accept + document** when the residual is rare, cosmetic, self-contained, and the only fix
  is disproportionate (a cross-dialect transaction, a rewrite). A limitation named on purpose
  is care; a silent one is a latent bug.
- **Never gold-plate.** Code that runs on every request to guard a near-impossible edge is its
  own defect. Matching effort to risk *is* the engineering.
- **Re-audit the fix itself.** A fix that relaxes a check, widens a bound, or flips a fail-mode
  (fail-closed → fail-open) can be a *worse* defect than the one it closed — the reviewer who
  pushed for it saw only the symptom. Run the fix back through the angles above, not just its
  now-green tests; a fix to a control especially.

## Close

Fix real findings and re-verify (implement + gates). Report honestly: what you verified as
correct (with specifics), the findings + severity, what you fixed, what you accepted on
purpose. Don't say "all clear" before an in-flight independent pass returns — wait, then
reconcile.
