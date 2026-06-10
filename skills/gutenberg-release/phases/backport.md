# Phase: `backport` (Days 4–6)

> Follow the interaction protocol in SKILL.md for every numbered step.

Goal: collect backport PRs, cut RC2 if needed.

1. **List merged PRs labeled `Backport to Gutenberg RC`** since RC1 (read-only `gh pr list`).
2. **Cherry-pick.** Once PR #78879 (auto cherry-pick CI) is merged, this happens automatically on label — skip this step. Until then, ask to run:
   ```bash
   npm run other:cherry-pick "Backport to Gutenberg RC"
   ```
   **Always pass the label** — omitted, the script silently defaults to `Backport to WP Beta/RC` and picks WordPress-Core-backport PRs instead. If the user isn't on the Gutenberg Core team, the push to `release/X.Y` fails with `GH006` — fall back to asking a Core member to push, or open a coordination PR. The script comments on source PRs, strips the label, and realigns milestones automatically; don't second-guess it.
3. **Review the results.** List each PR picked (PR → SHA) and any failures; wait for confirmation before cutting RC2.
4. **Trigger `Build Gutenberg Plugin Zip` for RC2.** No npm-publish environment approval needed (npm publishes only on rc.1), but always ask before dispatching — timing depends on backport readiness. Post back the run URL.
