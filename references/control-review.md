# Auditing a control — when the change CLAIMS to prevent / guarantee / enforce

A safety / security / correctness **control** — a check, validator, floor, guard, rate-limit,
allow-list, invariant — can pass every *"is there a bug in this diff"* review and still be
**broken by design**. The refute angles in [audit.md](audit.md) hunt bugs on the changed lines;
they systematically miss whether the control is **sound against its own threat**. When a change
claims to prevent, guarantee, enforce, or validate something, run these five before calling it
done — and treat a failure here as *"it doesn't do what it says,"* not a nit.

Refute-the-diff asks *"can I exploit the code here."* These ask *"does the control trust the
right thing, can it be turned off, does it measure the real property, what does it cost, and
does it actually deliver."* That reframing is the whole point — a fleet of same-framed
exploit-hunters will not surface any of it.

**1 — Trust the right input.** Name the threat it defends against. Then trace **every input the
decision depends on to its source**. If the party you're defending against controls any of
them, the control is void — it is enforcing the adversary's number. *Tell:* a
threshold / limit / expected-value / signature / floor read from the same response you are
validating. (A "minimum output" supplied by the swap provider you're guarding against is not a
check; the trusted value has to come from your own side of the boundary.)

**2 — Can the adversary turn it off?** Every control leans on something external — a simulation,
an RPC, an oracle, a cache, a config, a clock. Ask what it does when that dependency is
missing / errors / returns empty. **Fail-open (skip / allow on failure) is a bypass the instant
the adversary can *induce* the failure** — so construct that trigger (oversize the input,
exhaust a count limit, race a timeout, poison the cache, force the revert). **Fail-closed
(reject on failure) is a DoS / false-reject** if benign failures are common. You cannot have
both — pick one, justify it against the *real* failure distribution, and prove the adversary
can't force the losing side. Two traps: a "skip when unavailable" added for robustness is a
security hole if unavailability is attacker-triggerable; and **a fix that turns a false-reject
into fail-open has traded a nuisance for a bypass** (see §fix-re-audit in audit.md).

**3 — Property, not proxy.** State the property claimed; verify the code measures *that*, not a
stand-in that can be made to look right while the property is violated. Balance-delta ≠
provenance — a refund, a rent reclaim, a self-transfer moves the number without delivering
value. A 200 ≠ success. A passing simulation ≠ on-chain execution. Presence of an account ≠
authorization over it. *Tell:* the check compares magnitudes but never asks *where the value
came from* or *whether it's the same actor / asset* it claims to be about.

**4 — What does it cost?** A control runs somewhere; state the bill on that path.
**Latency / resource:** an added RPC / fetch / scan / lock on a hot path — with what timeout,
what payload size, deduped against existing reads or not, on the critical path or overlapping
work already in flight? **False-reject / availability:** how often does it reject *legitimate*
traffic, and does that pre-empt an existing recovery path (retry / reroute / fallback)? A
control that taxes the very metric the system exists to optimize (latency, throughput, success
rate) can be **net-negative even when correct**. Name the cost; if it's on the hot path,
justify it.

**5 — Does it deliver the claim — and if not, is it worse than nothing?** If §1–4 surfaced a
bypass, a wrong / adversary-controlled input, a spoofable proxy, or a disqualifying cost, the
honest verdict is **"the control does not do what it claims,"** not "the code runs and tests
pass." A control bypassable by its *target adversary* is a **false sense of security — worse
than absent, because someone will trust it.** Say that plainly; do not ship a claim the design
can't back, and do not let green tests or a clean refute pass stand in for it.

## Meta — a clean diff-refute pass does not validate a control

Same-framed refuters ("find a drain") returning nothing means *nothing was found within that
frame* — not that the design is sound. That is the exact false-confidence that ships broken
controls: three refuters + an audit can all come back "clean / merge-ready" on a control that a
single reviewer asking §1 ("where does the floor come from?") breaks in one line. For anything
security- or safety-critical, §1–5 must pass **and** a differently-framed reviewer — ideally a
human — must sign off before you report "clean." When you catch yourself relaying a clean bill,
check that it came from a review that could have found a *design* flaw, not only a diff flaw.
