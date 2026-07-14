# Auditing a control — refute its design, not just its diff

**When this applies.** A **control** is a change whose *worth is a guarantee* — "X can't
happen," "only Y may," "at most Z" — with a **threat model**: you can name the adversary (or the
untrusted input) *and* what's lost when it's defeated. Can't name both? It's ordinary
validation, not a control — refute the diff ([audit.md](audit.md)) and move on. Don't run this on
internal invariants, format checks, or defensive assertions that have no adversary.

The refute angles in [audit.md](audit.md) hunt bugs on the changed lines; a control can pass all
of them and still be **broken by design** — the whole apparatus can return "clean" on a
guarantee that a single *"where does this value come from?"* breaks. So each check below is a
**falsification**: state the guarantee, then actively try to construct the case where it fails —
a control you *tried and failed* to break is earned; one you only eyeballed is assumed. This is
the canon (Saltzer & Schroeder; the reference monitor; STRIDE) distilled to what a strong model
skips by default.

**1 — Trust the input.** *Claim:* every input the decision reads originates on the trusted side.
*Refute:* trace each to its producer — refuted the instant one comes from the party the control
is meant to constrain. A rate limit keyed off a client-set header, a role read from an unverified
token, an order total echoed back by the client, a min-output quoted by the swap provider you're
guarding against — each is the adversary's number wearing a check's clothes. *(never-trust-the-client;
confused deputy.)*

**2 — Can they turn it off?** *Claim:* the adversary can't reach the failure branch. *Refute:*
name what the check does when its dependency is missing / errors / empty, then try to make the
adversary trigger it — stall the datastore the limiter reads (→ `allow`), error the JWKS the auth
catch-block falls through on (→ `next()`), exhaust a count limit, race a timeout, poison the
cache, force the revert. **Fail-open is a bypass the moment the failure is inducible; fail-closed
is a DoS if benign failures are common** — pick one, justify it against the *real* failure
distribution, and prove the adversary can't force the losing side. (A fix that turns a
false-reject into fail-open has traded a nuisance for a bypass.) *(fail-safe defaults.)*

**3 — Is every path mediated?** *Claim:* this control sits on the *only* route to the asset.
*Refute:* find a second, unmediated route — another endpoint, a cache or replica, a retry queue,
a batch job, an admin or internal API — that reaches the same asset without passing this check. A
guarded front door is worth nothing while a side door is open. *(complete mediation / reference
monitor.)*

**4 — Property, not proxy — measured when it counts.** *Claim:* the code checks property P, on
the state P will act on. *Refute:* find one state where the proxy reads OK while P is false —
row-exists ≠ settled (a `pending` row counted as done), request-count ≠ resource-cost, key-match ≠
same-variant, unique-bytes ≠ same-identity (`Alice@x` vs `alice@x`), 200 ≠ durable commit,
account-present ≠ authorized, balance-delta ≠ provenance (a refund / self-transfer moves the
number without delivering value). Then mind the gap between check-time and use-time: a dry-run or
simulation is not the committed execution. *(STRIDE-spoofing; TOCTOU / CWE-367.)*

**5 — What does it cost — latency and complexity?** *Claim:* the cost is affordable on the path
it runs. *Refute:* put the number on it (added round-trips × latency, false-reject rate) and find
the load or input that blows the budget or makes users route around it — a freshness re-check
that hits origin on every cache hit has deleted the cache's reason to exist. And ask: is it
**simple enough to be verifiably correct?** A control too complex to audit hides the very design
bug you're hunting. *(psychological acceptability; economy of mechanism.)*

**6 — If it fails or is bypassed, do you know — and how much does it grant?** *Claim:* a defeat is
visible and contained. *Refute:* check the deny / fail / bypass path is logged, attributable, and
alertable (prevention with no detection is blind exactly when it's attacked), and bound what the
holder can do *after* it fails — least privilege caps the blast radius. *(compromise recording;
STRIDE-repudiation; least privilege.)*

**7 — Does it deliver the claim?** If 1–6 surfaced a bypass, an adversary-controlled input, an
unmediated path, a spoofable proxy, a disqualifying cost, or a silent failure, the honest verdict
is *"the control does not do what it claims,"* not "code runs, tests pass." A control **sold as a
guarantee** but bypassable by its target is worse than none sold that way — the claim invites
reliance the design can't bear. The usual fix is to **downgrade the claim to fit the design**
("best-effort layer," not "guarantee"), not delete the code: an honestly-labeled defense-in-depth
layer that catches *other* cases is a net positive. What you must never ship is the claim the
design can't back.

## Meta — a clean diff-refute pass does not validate a control

Same-framed refuters ("find a drain") returning nothing means *nothing was found in that frame*,
not that the design is sound — the exact false confidence that ships broken controls. Three
refuters and an audit can all come back "clean" on a control that §1 breaks in one line. For
anything security- or safety-critical, §1–7 must pass **and** a differently-framed reviewer —
ideally a human — must sign off before you report "clean."
