# Phase: `whats-new` (Day 7)

> Follow the interaction protocol in SKILL.md for every numbered step.

Goal: draft and get review on the "What's new in Gutenberg X.Y" Make Core post.

1. **Check Make Core editor permission.** Make Core uses a wordpress.org account, **not GitHub** — never reference a GitHub handle here. If the user is unsure they have access, draft the Slack ask for `#core` (w.org):
   ```
   > Hi! I'm coordinating the Gutenberg X.Y release this week and would like editor
   > access on make.wordpress.org/core to publish the "What's new" post.
   > Username: <your wordpress.org username>. Thanks!
   ```
2. **Draft the post as block markup, not Markdown** — the user pastes it into the Make Core editor:
   ```html
   <!-- wp:heading {"level":2} -->
   <h2 class="wp-block-heading">Feature name</h2>
   <!-- /wp:heading -->

   <!-- wp:paragraph -->
   <p>One-paragraph description with PR links
   (<a href="https://github.com/WordPress/gutenberg/pull/12345">#12345</a>).</p>
   <!-- /wp:paragraph -->
   ```
   Pull the milestone PRs grouped by category. Structure: 2–4 highlights → other notable changes → full changelog. Show each section one at a time for review before drafting the next.
3. **Pin every claim to a real source.** Each highlight's title and description must come verbatim (or faithfully paraphrased) from the originating issue/PR — **never invent a feature description, author name, or behavior.** Cross-check each PR's actual title and body before showing a section, and surface any drift:
   ```
   ✓ #12345  "…actual PR title…"                            — matches draft
   ⚠ #12351  draft credits @userX → actual author is @userY — needs fix
   ⚠ "Refreshed every 30s" — no source PR found             — remove unless sourced
   ```
4. **Post for review** in `#core` (w.org), optionally `#design` (w.org) and `#core` (a8c). Draft each Slack message in a fenced block for the user to paste.
