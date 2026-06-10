# Phase: `rc` (Day 3 — RC1)

> Follow the interaction protocol in SKILL.md for every numbered step.

Goal: cut RC1 cleanly.

1. **Re-curate the changelog.** Run the generator again; propose edits (merge near-duplicates, move misfiled entries, tighten wording) as a diff and ask before applying. Ask again before anything touches a GitHub release.
   ```diff
   - - Add new color picker to global styles. (#12345)
   - - Improve color picker focus state. (#12351)
   + - Color picker: new picker UI in global styles, improved focus state. (#12345, #12351)
   ```
2. **Verify milestone `Gutenberg X.Y+1` exists** (read-only `gh api`). If missing, ask, then create it — otherwise PRs merged after RC1 fall into X.Y and pollute the release.
3. **Trigger `Build Gutenberg Plugin Zip`** (`version: rc`, from `trunk`) via `gh workflow run`. Ask first. It pauses for an npm-publish environment approval from a release-team member (`WordPress/gutenberg-release`); post back the run URL.
4. **Watch for known failures.** `npm whoami` 401 at publish = rotated `NPM_TOKEN`; someone on the team must refresh it. Draft the Slack ask for `#core-editor` (w.org):
   ```
   > Heads-up — the NPM_TOKEN secret for WordPress/gutenberg looks rotated
   > (npm publish hit 401). Could someone refresh it? Workflow run: <run URL>
   ```
5. **Do not close the `Gutenberg X.Y` milestone.** Closing it breaks `npm run other:changelog`; it closes in the `stable` phase, after the post is live.
