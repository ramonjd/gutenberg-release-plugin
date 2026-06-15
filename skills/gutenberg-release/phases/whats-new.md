# Phase: `whats-new` (Day 7)

> Follow the interaction protocol in SKILL.md for every numbered step.

Goal: draft and get review on the "What's new in Gutenberg X.Y" Make Core post.

1. **Check Make Core editor permission.** Make Core uses a wordpress.org account, **not GitHub** — never reference a GitHub handle here. If the user is unsure they have access, draft the Slack ask for `#core` (w.org):
   ```
   > Hi! I'm coordinating the Gutenberg X.Y release this week and would like editor
   > access on make.wordpress.org/core to publish the "What's new" post.
   > Username: <your wordpress.org username>. Thanks!
   ```
2. **Confirm the release context.** If the user does not pass `X.Y`, infer it from the latest RC tag (`git tag --list 'v*-rc.*' --sort=-v:refname | head`) and verify with `gh release view vX.Y.0-rc.1 --repo WordPress/gutenberg`. Then check the `Gutenberg X.Y` milestone state and due date. Treat these as read-only checks.
3. **Gather source material.** Pull the RC release notes and milestone PRs. Use the generated changelog to identify candidate highlights, but source every highlight from the originating PR or issue body before drafting prose.
4. **Draft the post as block markup, not Markdown** — the user pastes it into the Make Core editor. Prefer writing the draft to `.context/gutenberg-X.Y-whats-new-draft.html` and a companion source audit to `.context/gutenberg-X.Y-whats-new-sources.md` so the user can review and other agents can continue the work:
   ```html
   <!-- wp:heading {"level":2} -->
   <h2 class="wp-block-heading">Feature name</h2>
   <!-- /wp:heading -->

   <!-- wp:paragraph -->
   <p>One-paragraph description with PR links
   (<a href="https://github.com/WordPress/gutenberg/pull/12345">#12345</a>).</p>
   <!-- /wp:paragraph -->
   ```
   Structure: intro → 2–4 highlights → other notable changes → accessibility → full changelog/RC notes link → props. If producing a full first draft in one pass, keep claims conservative and mark anything not deeply checked in the source audit.
5. **Pin every claim to a real source.** Each highlight's title and description must come verbatim (or faithfully paraphrased) from the originating issue/PR — **never invent a feature description, author name, or behavior.** Cross-check each PR's actual title and body before showing a section, and surface any drift:
   ```
   ✓ #12345  "…actual PR title…"                            — matches draft
   ⚠ #12351  draft credits @userX → actual author is @userY — needs fix
   ⚠ "Refreshed every 30s" — no source PR found             — remove unless sourced
   ```
   The source audit should list checked PRs separately from changelog-only bullets that need deeper review before publication.
6. **Refresh before publishing.** Before the final stable post goes live, refresh from the stable release notes and any post-RC backports. Do not present the RC changelog as final if stable has not shipped yet.
7. **Post for review** in `#core` (w.org), optionally `#design` (w.org) and `#core` (a8c). Draft each Slack message in a fenced block for the user to paste.
