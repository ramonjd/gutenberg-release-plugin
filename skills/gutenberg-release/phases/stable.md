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
   - **If the wp.org upload workflow fails after the GitHub release is published**, diagnose before rerunning. Check whether the SVN commit already landed:
     ```bash
     curl -sI https://downloads.wordpress.org/plugin/gutenberg.X.Y.0.zip | head -1
     curl -sI https://plugins.svn.wordpress.org/gutenberg/tags/X.Y.0/ | head -1
     curl -s https://plugins.svn.wordpress.org/gutenberg/trunk/readme.txt | grep -i "Stable tag"
     ```
   - **If the SVN tag and ZIP are missing**, ask a person with Gutenberg plugin SVN commit access to manually create trunk + tag from the GitHub release artifact. Do not commit `trunk` alone; `Stable tag: X.Y.0` must not point at a missing tag.
   - **Manual SVN publish for an authorized committer**:
     ```bash
     VERSION=X.Y.0
     WORK=/tmp/gutenberg-svn-release-$VERSION
     TRUNK=$WORK/trunk
     TAGS=$WORK/tags

     rm -rf "$WORK"
     mkdir -p "$WORK"

     svn checkout https://plugins.svn.wordpress.org/gutenberg/trunk "$TRUNK"
     svn checkout --depth immediates https://plugins.svn.wordpress.org/gutenberg/tags "$TAGS"

     gh release download "v$VERSION" --repo WordPress/gutenberg --pattern gutenberg.zip --dir "$WORK"
     gh run download <failed-upload-run-id> --repo WordPress/gutenberg --name "changelog trunk" --dir "$WORK"

     find "$TRUNK" -mindepth 1 -maxdepth 1 ! -name .svn -exec rm -rf {} +
     unzip -q "$WORK/gutenberg.zip" -d "$WORK/unzip"
     rsync -a "$WORK/unzip/" "$TRUNK/"
     cp "$WORK/changelog.txt" "$TRUNK/changelog.txt"
     sed -i.bak "s/^Stable tag:.*/Stable tag: $VERSION/" "$TRUNK/readme.txt" && rm "$TRUNK/readme.txt.bak"

     svn status "$TRUNK" | awk '$1 == "!" { print $2 }' | while IFS= read -r path; do svn rm "$path"; done
     svn status "$TRUNK" | awk '$1 == "?" { print $2 }' | while IFS= read -r path; do svn add "$path"; done

     mkdir "$TAGS/$VERSION"
     rsync -a --exclude='.svn' "$TRUNK/" "$TAGS/$VERSION/"
     svn add "$TAGS/$VERSION"
     ```
   - **Guardrails before commit**:
     ```bash
     grep -i '^Stable tag:' "$TRUNK/readme.txt"
     head -20 "$TRUNK/changelog.txt"
     svn status "$TRUNK" "$TAGS/$VERSION" | awk '$1 ~ /^[?!C]/ { print }'
     svn status "$TAGS/$VERSION" | sed -n '1,5p' # should show plain "A", not "A +"
     ```
   - **Commit both paths together** with the authorized WordPress.org SVN username:
     ```bash
     svn commit "$TRUNK" "$TAGS/$VERSION" \
       -m "Releasing version $VERSION" \
       --username <wporg-svn-committer-username> \
       --no-auth-cache \
       --config-option=servers:global:http-timeout=600
     ```
     If SVN errors after `Committing transaction...`, do not immediately retry; run the verification commands again first. If SVN says creating `gutenberg/tags/X.Y.0` is forbidden, the account does not have the needed plugin SVN permission.
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
