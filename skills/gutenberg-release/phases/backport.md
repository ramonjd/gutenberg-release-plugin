# Phase: `backport` (Days 4–6)

> Follow the interaction protocol in SKILL.md for every numbered step.

Goal: collect backport PRs, cut RC2 if needed.

1. **List merged PRs labeled `Backport to Gutenberg RC`** since RC1 (read-only `gh pr list`).
2. **Cherry-pick.** Once PR #78879 (auto cherry-pick CI) is merged, this happens automatically on label — skip this step. Until then, ask to run:
   ```bash
   npm run other:cherry-pick "Backport to Gutenberg RC"
   ```
   **Always pass the label** — omitted, the script silently defaults to `Backport to WP Beta/RC` and picks WordPress-Core-backport PRs instead. If the user isn't on the Gutenberg Core team, the push to `release/X.Y` fails with `GH006` — fall back to asking a Core member to push, or open a coordination PR. The script comments on source PRs, strips the label, and realigns milestones automatically; don't second-guess it.
3. **Review the results.** List each PR picked (PR → SHA) and any failures. For any PR that cannot be cherry-picked cleanly because of conflicts, draft a source-PR comment asking the author for a manual backport PR. Fill in the PR number, release branch, merge SHA, conflict paths, and target version.

   **Manual backport guardrails:**
   - Always isolate the backport before touching code: create a new no-upstream `backport/<pr-number>-release-X.Y` branch from `origin/release/X.Y`.
   - Never cherry-pick directly onto `release/X.Y`.
   - Never push to `release/X.Y`; the release branch changes only when the backport PR merges.
   - Do not run `git push` unless the user explicitly says `push` and the target ref is explicit.
   - Refuse to push when the current upstream is a protected/release branch, or when the push target is `release/*`.
   - Do not run `gh pr create` unless the user explicitly says `open PR`.
   - After cherry-picking, report: branch tip, upstream tracking, commits ahead of `origin/release/X.Y`, and whether a PR would be non-empty.

   ```markdown
   This PR did not cherry-pick cleanly into `release/X.Y` during the Gutenberg X.Y RC backport run. Could you please open a manual backport PR?

   Suggested steps:
   1. `git fetch origin release/X.Y:refs/remotes/origin/release/X.Y`
   2. `git switch --no-track -c backport/<pr-number>-release-X.Y origin/release/X.Y`
   3. `git cherry-pick -x <merge-sha>`
   4. Resolve the conflicts in `<conflict-paths>`, then run `git add -A` and `git cherry-pick --continue`.
   5. Before pushing, confirm you are still on `backport/<pr-number>-release-X.Y` and not on `release/X.Y`:
      - `git branch --show-current`
      - `git rev-parse --short HEAD`
      - `git rev-parse --abbrev-ref --symbolic-full-name @{u} 2>/dev/null || echo "(no upstream)"`
      - `git rev-list --count origin/release/X.Y..HEAD`
      - `git diff --quiet origin/release/X.Y...HEAD && echo "PR would be empty" || echo "PR would contain changes"`
   6. Push only the isolated backport branch, never the release branch:
      - `git push -u origin HEAD:backport/<pr-number>-release-X.Y`
   7. Open a PR targeting `release/X.Y` with a title like `[Release/X.Y] Backport #<pr-number>`.
   8. Link the backport PR in a comment here so we can include it in the RC cut.

   If you hit permission issues or need help resolving the conflicts, mention it here and we can coordinate.
   ```
   Do not post the comment automatically; show the filled-in draft and ask whether to post it or let the user paste it manually. Wait for confirmation before cutting RC2.
4. **Trigger `Build Gutenberg Plugin Zip` for RC2.** No npm-publish environment approval needed (npm publishes only on rc.1), but always ask before dispatching — timing depends on backport readiness. Post back the run URL.
