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
     If the workflow failed while checking whether the SVN tag exists, inspect `.github/workflows/upload-release-to-plugin-repo.yml` before falling back to manual SVN. The expected behavior is that a missing tag is treated as `exists=false`; WP.org SVN may report that as `E160013`, `W160013`, `E200009`, or `non-existent`.
   - **If the SVN tag and ZIP are missing**, ask a person with Gutenberg plugin SVN commit access to manually create trunk + tag from the GitHub release artifact. Do not commit `trunk` alone; `Stable tag: X.Y.0` must not point at a missing tag.
   - **Manual SVN publish for an authorized committer**:
     ```bash
     # Set the release version and keep all temporary files in one disposable workspace.
     VERSION=X.Y.0
     WORK=/tmp/gutenberg-svn-release-$VERSION
     TRUNK=$WORK/trunk
     TAGS=$WORK/tags
     STABLE_TAG_PLACEHOLDER='Stable tag: V\.V\.V'

     # Start clean so stale SVN state or old artifacts cannot leak into the release.
     rm -rf "$WORK"
     mkdir -p "$WORK"

     # Check out SVN trunk and the tags directory separately.
     svn checkout https://plugins.svn.wordpress.org/gutenberg/trunk "$TRUNK"
     svn checkout --depth immediates https://plugins.svn.wordpress.org/gutenberg/tags "$TAGS"

     # Download the exact GitHub release ZIP and the generated trunk changelog artifact.
     gh release download "v$VERSION" --repo WordPress/gutenberg --pattern gutenberg.zip --dir "$WORK"
     gh run download <failed-upload-run-id> --repo WordPress/gutenberg --name "changelog trunk" --dir "$WORK"

     # Replace SVN trunk contents with the ZIP contents, preserving only SVN metadata.
     find "$TRUNK" -mindepth 1 -maxdepth 1 ! -name .svn -exec rm -rf {} +
     unzip -q "$WORK/gutenberg.zip" -d "$TRUNK"

     # Use the workflow-generated changelog and set readme.txt to the new stable tag.
     cp "$WORK/changelog.txt" "$TRUNK/changelog.txt"
     sed -i.bak "s/$STABLE_TAG_PLACEHOLDER/Stable tag: $VERSION/g" "$TRUNK/readme.txt" && rm "$TRUNK/readme.txt.bak"

     # Mirror the workflow: add new files and remove tracked files the release no longer contains.
     svn status "$TRUNK" | awk '$1 == "?" { print $2 }' > "$WORK/new-svn-paths.txt"
     if [ -s "$WORK/new-svn-paths.txt" ]; then
       xargs svn add < "$WORK/new-svn-paths.txt"
     fi
     svn status "$TRUNK" | awk '$1 == "!" { print $2 }' > "$WORK/missing-svn-paths.txt"
     if [ -s "$WORK/missing-svn-paths.txt" ]; then
       xargs svn rm < "$WORK/missing-svn-paths.txt"
     fi

     # Create the version tag from trunk using SVN copy-with-history, matching the workflow.
     # This is much cheaper than adding the full tag contents as new files.
     svn cp "$TRUNK" "$TAGS/$VERSION"
     ```
   - **Guardrails before commit**:
     ```bash
     # Confirm readme.txt and changelog.txt point at the version being published.
     grep -i '^Stable tag:' "$TRUNK/readme.txt"
     head -20 "$TRUNK/changelog.txt"

     # This should print nothing: no unversioned, missing, or conflicted paths.
     svn status "$TRUNK" "$TAGS/$VERSION" | awk '$1 ~ /^[?!C]/ { print }'

     # This should show "A +"; the plus means SVN copied tag history instead of uploading a full plain-add tag.
     svn status "$TAGS/$VERSION" | sed -n '1,5p'
     ```
   - **Commit both paths together** with the authorized WordPress.org SVN username:
     ```bash
     # Commit trunk and the version tag together so Stable tag never points at a missing tag.
     svn commit "$TRUNK" "$TAGS/$VERSION" \
       -m "Releasing version $VERSION" \
       --username <wporg-svn-committer-username> \
       --no-auth-cache \
       --config-option=servers:global:http-timeout=600
     ```
     If SVN errors after `Committing transaction...`, do not immediately retry; run the verification commands again first. If `svn cp` or `svn commit` says creating `gutenberg/tags/X.Y.0` is forbidden, the account does not have the needed plugin SVN permission.
     If the commit times out, is interrupted, or stalls long enough that the user stops it, run the verification commands first. If the tag/ZIP did not land, clean up the partial local tag and recreate it with the workflow-style SVN copy before rerunning the guardrails and commit:
     ```bash
     svn cleanup "$TRUNK" "$TAGS"
     svn revert -R "$TAGS/$VERSION"
     rm -rf "$TAGS/$VERSION"
     svn cp "$TRUNK" "$TAGS/$VERSION"
     ```
   - **Verify the manual publish succeeded** before continuing with announcements:
     ```bash
     # The versioned ZIP and SVN tag should both exist.
     curl -sI "https://downloads.wordpress.org/plugin/gutenberg.$VERSION.zip" | head -1
     curl -sI "https://plugins.svn.wordpress.org/gutenberg/tags/$VERSION/" | head -1

     # SVN trunk should now advertise the new stable version.
     curl -s https://plugins.svn.wordpress.org/gutenberg/trunk/readme.txt | grep -i "Stable tag"
     ```
     Success means the ZIP and SVN tag return HTTP 200, and trunk shows `Stable tag: X.Y.0`. A browser check is also useful: visit `https://plugins.trac.wordpress.org/browser/gutenberg/tags/$VERSION` or `https://plugins.trac.wordpress.org/browser/gutenberg/#tags` and confirm the tag appears. The WordPress.org plugin page and Plugin API can lag behind the SVN commit.
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
