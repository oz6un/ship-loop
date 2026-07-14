# The adversarial self-audit

This is the gate, and it's the sharpest part of the loop — the refutation half of "make a claim,
then try to break it." Its job is to **break the change before a user or production does.** Run it
on any substantial change before you call it done, and again whenever the user says "audit," "make
sure," "harden," or "check it carefully."

## Stance

- **Refute, don't confirm.** Hunt the input, state, timing, or platform that makes it wrong.
- **Your own claims are in scope.** A comment, doc, or PR line that says more than the code
  does is a real defect — verify every "matches / always / can't happen" you wrote.
- **A subagent's self-checked number is not verified.** A value a subagent both produced *and*
  validated with its own harness is self-confirming — re-measure any load-bearing fact (a
  threshold, a rate, a formula, a live config value) by a second, independent method before you
  build on it.
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
   - **Control-design** (when the change is a *control* — its worth is a guarantee against a
     named adversary): trust-the-input, fail-open, an unmediated second path, proxy-not-property,
     cost, silent-failure — the *design* flaws line-by-line refute can't see:
     **[control-review.md](control-review.md)**.
3. **Verify each candidate against the real code, and when in doubt keep it**: mark each
   CONFIRMED / PLAUSIBLE / REFUTED. Only mark something REFUTED when you can actually show it's
   dead — either the path is impossible (point to the constant, type, or guard that rules it out)
   or the claim is factually wrong (quote the line). A path that's realistic but rare — an error
   handler, a cold cache, a falsy zero, a boundary off-by-one, a retry storm — is PLAUSIBLE, not
   refuted.
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
reconcile. And before you pass along any clean bill of health, name the *class* of flaw the review
was actually able to catch — a clean pass from reviewers who could only see one class of flaw tells
you that class is clear, not that the whole change is.
