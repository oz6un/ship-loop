# Auditing a control — try to break its design, not just its code

**When this applies.** A **control** is a change whose worth is a *guarantee* — "X can't happen,"
"only Y may," "at most Z" — against someone you can name. To qualify, you have to be able to say
two things: who the adversary is (or which input is untrusted), *and* what is lost when the
guarantee is defeated. If you can't say both, it isn't a control, just ordinary validation —
refute the diff ([audit.md](audit.md)) and move on. Don't run this on internal invariants, format
checks, or defensive assertions that have no adversary.

The refute angles in [audit.md](audit.md) hunt for bugs on the lines you changed. A control can
pass every one of them and still be **broken by design** — the whole review comes back "clean" on
a guarantee that a single question, *"where does this value actually come from?"*, would have
broken. So treat each check below as an attempt to *falsify* the guarantee: state it out loud,
then actively try to build the case where it fails. A control you *tried and failed* to break is
earned; one you only looked over is merely assumed. These seven checks are the long-established
security canon (Saltzer & Schroeder's design principles; the reference monitor; STRIDE) reduced to
the parts a strong model skips by default.

**1 — Does it trust the right input?** *The guarantee:* every value the decision reads comes from
the trusted side. *Try to break it:* trace each input back to whoever produced it. The guarantee
fails the moment one of them comes from the very party the control is meant to restrain — a rate
limit keyed on a client-set header, a role read from a token nobody verified, an order total the
client sent back to you. Each is the adversary's own number dressed up as a check. *(Never trust
the client; the confused-deputy problem.)*

**2 — Can the adversary switch it off?** *The guarantee:* the adversary can't reach the failure
branch. *Try to break it:* say plainly what the check does when the thing it depends on is
missing, errors, or comes back empty — then try to make the adversary cause exactly that. Stall
the datastore the limiter reads (does it then `allow`?); make the key lookup the auth check relies
on error so its catch-block falls through (does it call `next()`?); exhaust a counter; race a
timeout; poison a cache. **Failing open is a bypass the moment the failure can be induced; failing
closed is a denial of service if harmless failures are common.** Pick one on purpose, justify it
against how the dependency *actually* fails, and show the adversary can't force the losing side. (A
fix that turns an annoying false-reject into fail-open has traded a nuisance for a bypass.)
*(Fail-safe defaults.)*

**3 — Is every path covered?** *The guarantee:* this control sits on the *only* route to the thing
it protects. *Try to break it:* find a second route that skips it — another endpoint, a cache or
read-replica, a retry queue, a batch job, an internal or admin API — that reaches the same thing
without passing the check. A locked front door is worth nothing while a side door stands open.
*(Complete mediation; the reference monitor.)*

**4 — Does it check the real thing, or a stand-in?** *The guarantee:* the code checks the actual
property it cares about, on the actual state it will act on. *Try to break it:* find one state
where the stand-in looks fine but the real property is false. A row exists, but it's still
`pending`, not settled. The request count is low, but each request is enormously expensive. Two
keys match, but they point at different variants. Two strings look equal, but `Alice@x` and
`alice@x` are different identities. A `200` came back, but nothing was durably committed. An
account is present, but it was never authorized. Then mind the gap between when you check and when
you act: a dry run or simulation is not the real, committed execution. *(STRIDE spoofing;
time-of-check to time-of-use, CWE-367.)*

**5 — What does it cost, in latency and complexity?** *The guarantee:* the cost is affordable on
the path where it runs. *Try to break it:* put a real number on it — added round-trips times their
latency, plus the false-reject rate — and find the load or the input that blows the budget or
makes people route around the control. A freshness re-check that calls the origin on every cache
hit has quietly deleted the reason the cache exists. Then ask the other half: **is it simple
enough to be checked by eye?** A control too complex to audit is hiding the exact design bug you're
looking for. *(Psychological acceptability; economy of mechanism.)*

**6 — If it fails or is bypassed, will you know — and how much does a defeat grant?** *The
guarantee:* a defeat is both visible and contained. *Try to break it:* check that the deny, fail,
and bypass paths are actually logged, attributable to someone, and able to raise an alert —
prevention with no detection is blind exactly when it's under attack. Then bound what the holder
can do *after* the control fails: least privilege is what keeps one defeat from turning total.
*(Recording compromises; STRIDE repudiation; least privilege.)*

**7 — Does it actually deliver what it claims?** If checks 1–6 turned up a bypass, an
adversary-controlled input, an uncovered path, a spoofable stand-in, an unaffordable cost, or a
silent failure, then the honest verdict is *"this control does not do what it claims"* — not "the
code runs and the tests pass." A control **sold as a guarantee** but defeatable by its own target
is worse than no control sold that way, because the claim invites people to rely on something the
design can't support. The usual fix is not to delete the code but to **shrink the claim to fit the
design** — call it a "best-effort layer," not a "guarantee." An honestly labeled extra layer that
catches *other* cases is still a net gain. The one thing you must never ship is a guarantee the
design can't keep.

## Meta — a clean refute pass does not, by itself, validate a control

Reviewers who are all framed the same way ("find a way to break in") coming back empty means only
that *nothing was found from that one angle* — not that the design is sound. This is the exact
false confidence that ships broken controls: three refuters and a full audit can all report
"clean" on a control that check 1 breaks in a single line. For anything security- or
safety-critical, checks 1–7 must pass **and** a reviewer framed differently — ideally a human —
must sign off before you report "clean."
