# Ship mechanics

Project-agnostic. A change lands on the default branch **only** after it passes the project's
own gates locally and in CI. Discover how *this* repo works; the moves below are the shape.

## Discover the gates (early)

Learn what "verified" means here, then run the whole set — cheap gates first (typecheck, lint)
so a fast failure saves the slow suite:

```bash
rg -n '"(scripts)"' package.json            # or: cat Makefile justfile pyproject.toml Cargo.toml
ls .husky .githooks 2>/dev/null; cat .pre-commit-config.yaml 2>/dev/null   # the mandatory hooks
ls .github/workflows 2>/dev/null            # the real bar: the jobs CI runs
```

Look for: typecheck (every package/workspace), formatter/linter, **custom guard scripts**
(design tokens, test-id coverage, API-contract drift, import boundaries — repos hide these),
unit tests, and the type-check for e2e/integration suites. Run them all green before pushing.

## Branch → commit → PR

- Never commit substantial work on the default branch: `git checkout -b <type>/<slug>`
  (`fix/…`, `feat/…`, `refactor/…`, `docs/…`).
- **Branch from the *target*, and verify the diff is exactly what you intend.** Cut from the
  base you'll merge into (`git checkout -b … <target>`), not from whatever happens to be checked
  out — a branch cut from another feature branch carries *its* commits (often *pre-fix* code)
  into your PR. Before opening it: `git diff <base>..HEAD --stat` shows only the files you meant
  and `git log <base>..HEAD` only the commits you meant; if a second concern rode along, it
  belongs on its own branch.
- Let the pre-commit hooks run; if they reformat or fail, fix and re-stage — do **not**
  `--no-verify` past a real failure. Warnings in files you didn't touch are pre-existing; a
  warning in *your* file is yours.
- Commit message: the repo's convention (often Conventional Commits) + a body that says the
  *why* and the trade-off + any co-author/sign-off trailer the repo uses.
- Push (if the normal remote/auth is broken, use whatever works here — e.g. an explicit HTTPS
  remote URL when the SSH agent is down). Open the PR:

```
## Why      the problem, in 1–2 lines
## What     the approach + the one real trade-off (and what you deliberately did NOT do)
## Verified the gates you ran + tests, local and in CI
```

## Wait for CI to *actually* go green

Poll until nothing is pending, then read the result — never merge on partial/assumed-green.
A red check is one of three things; know which before you touch anything:

- **Real regression** → fix the code, re-verify locally, push again.
- **A test that legitimately must change** because you changed the contract → update it and
  say why in the commit. (Never edit a test *just* to make it pass.)
- **Generated-artifact drift** — a committed snapshot / lockfile / OpenAPI spec / generated
  types file is stale because your change touched its source. **Regenerate with the project's
  own generator and amend; never hand-edit** the generated file. Note: an API-spec snapshot
  drifts on a route's *summary/description*, not only its schema.

After an amend, re-push (`--force-with-lease`; fall back to `--force` if the environment can't
track the remote ref).

## Merge → sync → verify

- Merging the default branch is outward-facing and hard to reverse: do it only when asked or
  clearly authorized; otherwise stop at a green PR and confirm.
- Squash-merge + delete the branch. Confirm state = MERGED.
- Fast-forward local default (`git fetch` then `git merge --ff-only`) — never force it to
  diverge. Delete the merged local branch; leave the tree clean.
- **"Merged" ≠ "works in prod."** If the merge deploys, confirm the deploy succeeded and the
  behavior is live. If it broke prod, **revert first** (fast, reversible) and diagnose after —
  don't hot-fix forward under pressure.

## Discipline

- **One concern per change.** If implementing or auditing tempts an adjacent fix, fold it in
  only when it's the same bug-class (and say so in the PR); otherwise split it out.
- Environment quirks (auth fallbacks, offline-install lockfile hashes, a leaked port failing a
  test) are session facts, not the process — apply the workaround, confirm it's the cause, say
  so, and never relax a gate to make it pass.
