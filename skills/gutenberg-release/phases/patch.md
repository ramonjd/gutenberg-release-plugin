# Phase: `patch` (Post-release X.Y.Z hotfix)

> Follow the interaction protocol in SKILL.md for every numbered step.

Goal: ship a point release (`X.Y.Z`, tagged `vX.Y.Z`) carrying critical fixes that landed on `trunk` after `X.Y.0` shipped. Reference: §"Creating minor releases" in `docs/contributors/code/release/plugin-release.md`.

1. **List candidate PRs** (read-only) and confirm the scope before doing anything else:
   ```bash
   gh pr list --label "Backport to Gutenberg Minor Release" --state merged --limit 30
   ```
2. **Decide the cherry-pick path.** `npm run other:cherry-pick "Backport to Gutenberg Minor Release"` pushes straight to `release/X.Y` — Gutenberg Core team only. **Always pass the label** — omitted, the script silently defaults to `Backport to WP Beta/RC` and picks the wrong PRs. Non-Core default: open a coordination PR. Ask which path before branching.
3. **(PR route) Branch off `release/X.Y` and cherry-pick** each merge commit with `-x` (preserves the source-SHA trailer). Resolve CHANGELOG conflicts conservatively — `trunk` package CHANGELOGs often carry `Unreleased` entries from *unrelated* PRs; drop those, keep only entries for the PRs being backported.
   ```bash
   git fetch origin trunk release/X.Y
   git checkout -b backport/X.Y.Z origin/release/X.Y
   git cherry-pick -x <merge-sha-1> <merge-sha-2> ...
   git push -u origin backport/X.Y.Z
   gh pr create --base release/X.Y --title "Backport fixes for X.Y.Z" --body ...
   ```
4. **Label the backport PR**: touched-package labels (`[Package] foo`), `[Type] Bug`, milestone `Gutenberg X.Y`. If the milestone was closed at stable time, reopen it first (see gotchas). Show the PR URL.
5. **Wait for review + merge.** Merging into `release/X.Y` needs release-team permission — if the user lacks it, draft a `#core-editor` ask for a Core member to hit merge, even after approval.
6. **After merge: reassign source-PR milestones and strip the backport label.** The changelog generator queries by **milestone, not branch content** — skip this and the X.Y.Z draft release will be empty.
   ```bash
   for pr in <pr1> <pr2> <pr3>; do
     gh pr edit $pr --milestone "Gutenberg X.Y" --remove-label "Backport to Gutenberg Minor Release"
   done
   ```
7. **Trigger `Build Gutenberg Plugin Zip` with input `stable`.** Branch selection (check read-only first: `gh release list --limit 5`):
   - **No `X.(Y+1)` RC published yet** → leave `Use workflow from: trunk`; the workflow sees the previous stable was `X.Y.0` and auto-bumps to the next patch.
   - **An `X.(Y+1).0-rc.N` prerelease exists** → MUST select `release/X.Y` from the dropdown, or the workflow ships the next major stable instead of the patch.
8. **Publish the draft release**:
   - `X.Y.Z` is the **newest line** (no `X.(Y+1)` released) → leave **"Set as latest release" checked**; the Upload workflow republishes SVN `trunk` + creates the SVN tag.
   - A **newer major is already out** → **uncheck it**; the workflow publishes only the SVN tag and leaves `trunk` alone.
9. **Approve `Publish npm packages`.** Same as RC1/stable — a release-team member must approve.
10. **Verify WP.org propagation** after a few minutes:
    ```bash
    curl -sI https://downloads.wordpress.org/plugin/gutenberg.X.Y.Z.zip | head -1
    curl -sI https://plugins.svn.wordpress.org/gutenberg/tags/X.Y.Z/ | head -1
    curl -s https://plugins.svn.wordpress.org/gutenberg/trunk/readme.txt | grep -i "stable tag"
    curl -s "https://api.wordpress.org/plugins/info/1.0/gutenberg.json" | python3 -c "import sys,json; print(json.load(sys.stdin)['version'])"
    ```
    The Plugin API and plugin page can lag 15–60 minutes behind the SVN commit — don't panic.
11. **Announce** in `#core-editor` (w.org) — fenced Slack draft with the `vX.Y.Z` release URL.
12. **Close the `Gutenberg X.Y` milestone** again once everything is verified.

## Known gotchas

- **Upload workflow fails with `svn: E160013: '/gutenberg' path not found`** in the "Publish as trunk (and tag)" step. Cosmetic WP.org SVN backend quirk — the commit usually went through. **Diagnose, don't re-run:**
  ```bash
  curl -sI https://plugins.svn.wordpress.org/gutenberg/tags/X.Y.Z/        # 200 → tag landed
  curl -s https://plugins.svn.wordpress.org/gutenberg/trunk/readme.txt | grep "Stable tag"  # X.Y.Z → trunk updated
  ```
  If both succeed, the release is out; the Plugin API cache just lags 15–60 min. If the public page still shows the old version after ~1 hour, ask in `#core-editor` for someone with WP.org plugin-directory access.
- **Backport PR unmergeable even after approval** — `release/*` branches are protection-restricted; the merge button stays greyed out for non-release-team users. Ask a Core member in `#core-editor` to merge.
- **The `Gutenberg X.Y` milestone is usually closed by patch time** (closed manually at stable; nothing auto-closes milestones). Reopen it before assigning the backport PR or moving source PRs, or `gh pr edit --milestone` fails with `'Gutenberg X.Y' not found`:
  ```bash
  gh api -X PATCH repos/WordPress/gutenberg/milestones/<id> -f state=open
  ```
- **The "Auto Cherry-Pick" GitHub Action does NOT handle Gutenberg minor releases.** Its label regex is `^Backport to WP X.Y Beta/RC$` — for WordPress-Core beta backports, a different workflow. It runs on every merge, sees the label mismatch, and exits with `cherry_pick=false`. Don't debug it for patches; use the cherry-pick script (Core team) or a coordination PR (everyone else).
