# Phase: `stable` (Day 8)

> Follow the interaction protocol in SKILL.md for every numbered step.

Goal: ship stable, publish the post, hand off.

1. **Check for outstanding backports** (read-only): PRs still labeled `Backport to Gutenberg RC` and not yet on `release/X.Y`. If any, stop and ask — wait and cherry-pick first (recommended), or cut stable anyway (they become X.Y.1 material).
2. **Announce the stable release is starting** in `#core-editor` (w.org). Slack messages are user actions: draft this for the user to paste before triggering the stable build:
   ```
   > Gutenberg X.Y stable release is starting now.
   >
   > Please avoid merging or backporting release-targeted changes unless coordinated in this thread. If there is a must-have issue for X.Y.0, please raise it here immediately; otherwise it will likely be X.Y.1 material.
   >
   > I'll share the GitHub release, WordPress.org plugin link, and "What's new" post once the release is published.
   ```
3. **Trigger the stable `Build Gutenberg Plugin Zip`.** Always ask first — this is the public release, never auto-run. Post back the run URL.
4. **Curate the draft release notes, then publish the release.** The draft combines the published RC changelogs plus anything cherry-picked since — delete the RC version headers (informational only) and move post-RC1 entries into the right sections. If RC1's changelog was fully curated before *it* was published, this is quick. The user presses "Publish release" ("Set as the latest release" is pre-selected for stable — leave the checkboxes alone).
5. **Approve the wp.org upload.** Publishing the release triggers the `Update Changelog and upload Gutenberg plugin to WordPress.org plugin repo` workflow, which needs approval from a Gutenberg Release, Gutenberg Core, or WordPress Core team member — the user can approve it themselves if they're on one of those teams, otherwise draft the `#core-editor` ask. Once it completes, confirm the new version appears on https://wordpress.org/plugins/gutenberg/ (may lag a little).
6. **Publish the "What's new" post — user's action.** Agents cannot publish to Make Core. Surface the draft URL, tell the user to hit Publish, and wait for the live URL before continuing.
7. **Announce** in `#core-editor` (w.org) and `#core` (a8c) — fenced Slack drafts with the live post URL embedded:
   ```
   > Gutenberg X.Y has been released! 🎉
   > - Plugin: https://wordpress.org/plugins/gutenberg/
   > - GitHub: https://github.com/WordPress/gutenberg/releases/tag/vX.Y.0
   > - What's new: <published post URL>
   > Thanks to everyone who contributed!
   ```
8. **Close the `Gutenberg X.Y` milestone — only after the post is live.** Closing early breaks `npm run other:changelog`. If it was closed early and the generator failed, run reopen → regen → reclose:
   ```bash
   gh api -X PATCH repos/WordPress/gutenberg/milestones/<id> -f state=open
   npm run other:changelog -- --milestone="Gutenberg X.Y"
   gh api -X PATCH repos/WordPress/gutenberg/milestones/<id> -f state=closed
   ```
9. **Ask for a volunteer for the next rotation.** Slack draft for `#core-editor` (w.org) with the X.Y+1 RC and stable dates.
