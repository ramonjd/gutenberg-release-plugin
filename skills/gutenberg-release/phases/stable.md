# Phase: `stable` (Day 8)

> Follow the interaction protocol in SKILL.md for every numbered step.

Goal: ship stable, publish the post, hand off.

1. **Check for outstanding backports** (read-only): PRs still labeled `Backport to Gutenberg RC` and not yet on `release/X.Y`. If any, stop and ask — wait and cherry-pick first (recommended), or cut stable anyway (they become X.Y.1 material).
2. **Trigger the stable `Build Gutenberg Plugin Zip`.** Always ask first — this is the public release, never auto-run. Post back the run URL (it still needs a release-team environment approval).
3. **Publish the "What's new" post — user's action.** Claude can't publish to Make Core. Surface the draft URL, tell the user to hit Publish, and wait for the live URL before continuing.
4. **Announce** in `#core-editor` (w.org) and `#core` (a8c) — fenced Slack drafts with the live post URL embedded:
   ```
   > Gutenberg X.Y has been released! 🎉
   > - Plugin: https://wordpress.org/plugins/gutenberg/
   > - GitHub: https://github.com/WordPress/gutenberg/releases/tag/vX.Y.0
   > - What's new: <published post URL>
   > Thanks to everyone who contributed!
   ```
5. **Close the `Gutenberg X.Y` milestone — only after the post is live.** Closing early breaks `npm run other:changelog`. If it was closed early and the generator failed, run reopen → regen → reclose:
   ```bash
   gh api -X PATCH repos/WordPress/gutenberg/milestones/<id> -f state=open
   npm run other:changelog -- --milestone="Gutenberg X.Y"
   gh api -X PATCH repos/WordPress/gutenberg/milestones/<id> -f state=closed
   ```
6. **Ask for a volunteer for the next rotation.** Slack draft for `#core-editor` (w.org) with the X.Y+1 RC and stable dates.
