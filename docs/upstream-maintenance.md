# Keeping up with upstream

This repository is a light fork of BetterGov's transparency portal. The aim is to stay close
to upstream so that its updates remain easy to review and integrate.

| Remote | URL | Use |
| --- | --- | --- |
| `origin` | `https://github.com/bettersanfernando/budget-portal` | BetterSanFernando's fork. Branches and PRs go here. |
| `upstream` | `https://github.com/bettergovph/transparency-portal` | BetterGov's source. **Read-only: fetch from it, never push to it.** |

Production: <https://budget.bettersanfernando.org>

## Remotes

```bash
git remote -v
```

Expect `origin` to be the fork. If `upstream` is missing, add it and block pushes to it:

```bash
git remote add upstream https://github.com/bettergovph/transparency-portal
git remote set-url --push upstream DISABLED
git fetch upstream
```

A fresh clone has only `origin`. Open PRs against `origin`'s `main`, never against upstream.

## What is local to this fork

`portal.config.ts` is intentionally local. It holds the San Fernando configuration, and an
upstream copy of it must never replace it:

| Setting | Current value |
| --- | --- |
| Identity | `id: san-fernando-pampanga`, `kind: city`, `name: City of San Fernando, Pampanga`, `shortName: San Fernando` |
| `siteUrl` | `https://budget.bettersanfernando.org` |
| `operator` | BetterSanFernando, `https://bettersanfernando.org` |
| `scope.offices` | none (the Pampanga 1st DEO is deliberately not added) |
| `scope.places` | `San Fernando`, `q: "San Fernando Pampanga"`, region `III` |
| `scope.districts` | `leg:pampanga:3rd` |
| `scope.exclude` | `dpwh:house-reading:4112` (Minalin, outside the city); `dpwh:house-reading:12536` (held: coordinates conflict with the stated location) |
| `reporting` | `enabled: false` |

Beyond that file, the fork differs from upstream in three places. Look at them first when a
sync conflicts:

- `vite.config.ts`: the meta description wording (IAN-11).
- `src/pages/Home.tsx`: the home route's runtime page title (IAN-11).
- `docs/validation-san-fernando-fy2027.md` and this file: fork-only documentation.

Compare with upstream at any time: `git diff upstream/main --stat`.

## Syncing from upstream

Never update `main` directly. Do the work on a branch and open a PR.

```bash
git checkout main
git pull origin main
git fetch upstream

# Review before merging
git log --oneline main..upstream/main
git diff main...upstream/main --stat
git diff main...upstream/main -- src/config.ts src/api portal.config.ts package.json vite.config.ts

git checkout -b sync/upstream-YYYY-MM-DD
git merge upstream/main
# resolve conflicts deliberately; keep the San Fernando values in portal.config.ts

npm install        # only if package.json or package-lock.json changed
npm run check -- --verbose
npm run lint
npm run build

git push -u origin sync/upstream-YYYY-MM-DD
# open a PR to main on bettersanfernando/budget-portal; do not merge until reviewed
```

Use a merge. Rebasing and force-pushing are not needed and make the history harder to review.

When reviewing the upstream diff, read the changed files; do not merge blind. Look hardest at:

- **Config and schema:** `src/config.ts` and anything that changes the shape or checks of
  `portal.config.ts`. A new required field, or a renamed one, means updating the local config by
  hand.
- **API client and types:** `src/api/`, `src/lib/scope.ts`, `src/lib/gaa.ts`. A change in how lines
  are matched or counted can change what the six-line scope shows.
- **Files this fork edits:** the ones listed above. Keep the fork's wording unless upstream's
  change is a fix you want.
- **Dependencies and deployment:** `package.json`, the lockfile, `wrangler.jsonc`, `vercel.json`,
  `netlify.toml`.

If upstream rewrites `portal.config.ts` (for example a new generated header or field), take the
new structure and re-apply the values in the table above. Do not take upstream's file as is.

## Checking the result

Run all three after any upstream sync or config change:

```bash
npm run check -- --verbose
npm run lint
npm run build
```

`npm run check` matters most. It resolves the configured scope against the live API and fails if an
exclusion matches nothing, a district id is unknown or a query is too broad. At the time of writing
it reports the **six-line San Fernando scope**: 6 lines (2 excluded by hand), NEP ₱67.0M on 2 lines,
House bill ₱316.2M on 6 lines, 4 house-only and 2 unchanged. The NEP and House bill totals are
compared, never added. A warning that three lines sit under a DPWH office outside the scope is
expected (see [validation-san-fernando-fy2027.md](validation-san-fernando-fy2027.md)).

If the numbers change, find out why before merging. The cause may be a new API build or an upstream
matching change, not a mistake in the sync. The API figures themselves can move when BetterGov
publishes a new build.

## Principles

- Keep fork-specific source changes to a minimum.
- Prefer supported configuration in `portal.config.ts` over editing components.
- Inspect upstream changes before integrating them.
- Never blindly overwrite `portal.config.ts`.
- Verify the six-line scope after any relevant upstream, API or config change.
- Land every upstream sync through a PR.
- Keep `upstream` read-only.
