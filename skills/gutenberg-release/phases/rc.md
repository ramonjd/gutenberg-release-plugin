# Phase: `rc` (Day 3 — RC1)

> Follow the interaction protocol in SKILL.md for every numbered step.

Goal: cut RC1 cleanly.

1. **Final changelog curation — fix labels, not prose.** The build workflow generates the release notes itself (`other:changelog --unreleased` → draft GitHub release), so durable fixes belong in PR labels and titles *before* dispatch. Run the generator, propose label fixes for anything misfiled or still in "Various", and ask before applying. Prose polish (merging near-duplicates, tightening wording) happens by editing the draft release *after* the build — ask before touching it:
   ```diff
   - - Add new color picker to global styles. (#12345)
   - - Improve color picker focus state. (#12351)
   + - Color picker: new picker UI in global styles, improved focus state. (#12345, #12351)
   ```
2. **Trigger `Build Gutenberg Plugin Zip`** (`version: rc`, from `trunk`) via `gh workflow run`. Ask first. It cuts `release/X.Y`, commits the `X.Y.0-rc.1` version bump there, cherry-picks that bump back to trunk, creates the draft release with generated notes, then pauses for an npm-publish environment approval from a release-team member (`WordPress/gutenberg-release`); post back the run URL.
3. **Announce the RC cut in `#core-editor` (w.org).** Time-critical: from the moment RC1 is dispatched, unlabeled trunk merges silently miss the release. Draft for the user to paste:
   ```
   Gutenberg X.Y RC1 is being cut now 🚀

   A couple of notes:
   • From this point, anything merged to trunk that should ship in X.Y needs
     the `Backport to Gutenberg RC` label.
   • Release team: the build pauses for an npm-publish approval — could someone
     approve when it pings? Run: <run URL>

   RC1 will appear at https://github.com/WordPress/gutenberg/releases shortly.
   ```
4. **Watch for known failures.** `npm whoami` 401 at publish = rotated `NPM_TOKEN`; someone on the team must refresh it. Draft the Slack ask for `#core-editor` (w.org):
   ```
   > Heads-up — the NPM_TOKEN secret for WordPress/gutenberg looks rotated
   > (npm publish hit 401). Could someone refresh it? Workflow run: <run URL>
   ```
5. **Verify milestone `Gutenberg X.Y+1` appears — do not create it manually.** The PR-merge automation (`add-milestone` task in `packages/project-management-automation`) creates it on the first merge after the RC1 version bump and assigns merged PRs to it. Check read-only via `gh api` once post-RC1 PRs have merged. If it's still missing: confirm the `Gutenberg X.Y` milestone has a due date (the automation derives the next due date from it), then create it by hand.
6. **Do not close the `Gutenberg X.Y` milestone.** Closing it breaks `npm run other:changelog`; it closes in the `stable` phase, after the post is live.
