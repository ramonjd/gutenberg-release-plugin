# Phase: `setup` (Days 1–2)

> Follow the interaction protocol in SKILL.md for every numbered step.

Goal: open the checklist issue and label every milestone PR so the changelog generator produces a clean draft.

1. **Confirm Gutenberg write permission.** Read-only `gh` check. If missing, draft a Slack ask for `#core-editor` (w.org) — write access requires ≥3 merged Gutenberg contributions:
   ```
   > Hi! I'm coordinating the Gutenberg X.Y release and would like write access
   > on WordPress/gutenberg to manage labels/milestones. GitHub: <your GitHub handle>. Thanks!
   ```
2. **Open the release checklist issue.** Read `.github/ISSUE_TEMPLATE/New_release.md`, fill in the version, create via `gh issue create` (or hand off). Post back the issue URL, title, assignee, and milestone for verification.
3. **Run the changelog generator** to find PRs missing labels (read-only — run inline, show output):
   ```bash
   npm run other:changelog -- --milestone="Gutenberg X.Y"
   ```
4. **Label uncategorized PRs.** From the generator output, find PRs in the "Various" bucket or missing a subcategory. Propose a label for each per the labels-mapping doc, ask once for the whole batch, apply via `gh pr edit`, then **show a table of every PR touched (before → after)** and wait for confirmation:
   ```
   PR     Title                      Added                     Removed
   12345  Fix block selection …      [Package] Block editor    —
   12351  Refactor inserter search   [Type] Enhancement        [Type] Various
   ```
