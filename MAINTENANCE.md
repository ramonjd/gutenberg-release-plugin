# Maintenance: verifying the skill against reality

The skill references repo files and official docs wherever possible, but a few facts are
hard-coded. Each row pairs a claim with the command that re-verifies it — run from a
Gutenberg checkout. If a check fails, fix the corresponding phase file and bump the
plugin version in `.claude-plugin/plugin.json`.

| Claim (where) | Verify with | Expect |
|---|---|---|
| `npm run other:changelog -- --milestone="Gutenberg X.Y"` exists (`setup`, `rc`, `stable`) | `grep '"other:changelog"' package.json` | script present |
| Changelog "Various" bucket = PRs without a mapped `[Type]` label (`setup`) | `grep -n "Various" tools/release/commands/changelog.js` | fallback to `'Various'` |
| `npm run other:cherry-pick` silently defaults to `Backport to WP Beta/RC` when no label is passed (`backport`, `patch`) | `sed -n '1,12p' tools/release/cherry-pick.mjs` | `process.argv[2] \|\| 'Backport to WP Beta/RC'` |
| Workflow names `Build Gutenberg Plugin Zip`, `Publish npm packages` (`rc`, `backport`, `stable`, `patch`) | `grep -h "^name:" .github/workflows/build-plugin-zip.yml .github/workflows/publish-npm-packages.yml` | exact names |
| npm publishes only on `rc.1` — RC2+ and patch builds skip it (`backport`) | `grep -n "rc.1" .github/workflows/build-plugin-zip.yml` | npm-publish job gated on `endsWith(..., '-rc.1')` |
| The "Auto Cherry-Pick" action only matches `Backport to WP X.Y Beta/RC`, never Gutenberg labels (`patch` gotchas) | `grep -n "Backport to" .github/workflows/cherry-pick-wp-release.yml` | WP-only regex |
| PR #78879 (auto cherry-pick to Gutenberg release branches) is still open (`backport` step 2) | `gh pr view 78879 --repo WordPress/gutenberg --json state` | `OPEN` — **when merged, rewrite `backport` step 2: cherry-picks become automatic on label** |
| Labels `Backport to Gutenberg RC` and `Backport to Gutenberg Minor Release` exist (`backport`, `patch`) | `gh label list --repo WordPress/gutenberg --search "Backport to Gutenberg"` | both listed |
| Checklist template path (`setup`) | `ls .github/ISSUE_TEMPLATE/New_release.md` | exists |
| Minor-release doc section (`patch`) | `ls docs/contributors/code/release/plugin-release.md` | exists |

## Design rules that keep drift small

- **Reference, don't duplicate.** The skill reads repo files (issue template, docs) and links
  official documentation instead of embedding copies. If you find yourself pasting process
  prose into a phase file, link it instead.
- **State time-bound facts conditionally** ("once #78879 is merged, skip this step") so each
  session re-checks reality rather than trusting a snapshot.
- **All versions are placeholders** (`X.Y`, `X.Y.Z`) — never bake a specific release into the
  skill. Format conventions live in `SKILL.md` → Conventions.
