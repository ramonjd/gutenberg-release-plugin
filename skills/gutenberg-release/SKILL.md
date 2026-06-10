---
name: gutenberg-release
description: Use when coordinating a Gutenberg plugin release rotation (RC, stable, or X.Y.Z point release) — changelog curation, PR labeling, backports/cherry-picks, plugin-zip builds, "What's new" post, milestone hygiene. Pass a phase argument: setup, rc, backport, whats-new, stable, or patch.
---

# Gutenberg Release

Helper for a Gutenberg plugin release rotation. Each phase has its own reference
file — **read only the file for the phase being run**, from this skill's `phases/`
directory. Always narrate the next step, ask before acting, and present results
back for review.

## Usage

```
/gutenberg-release <phase> [version]
```

| Phase | When | File |
|---|---|---|
| `setup` | Days 1–2: checklist issue, PR labeling, milestone hygiene | `phases/setup.md` |
| `rc` | Day 3: changelog curation, RC1 build, npm publish | `phases/rc.md` |
| `backport` | Days 4–6: cherry-picks, RC2 | `phases/backport.md` |
| `whats-new` | Day 7: draft & review the "What's new" post | `phases/whats-new.md` |
| `stable` | Day 8: stable release + publish post | `phases/stable.md` |
| `patch` | Post-release: X.Y.Z hotfix for critical bugs found after stable | `phases/patch.md` |

If no phase is passed, ask the user which phase they're on, then read that file.

## Conventions

- Milestones are titled `Gutenberg X.Y` (e.g. `Gutenberg 23.4`).
- Release tags are `vX.Y.Z` (e.g. `v23.3.0`); point releases are `X.Y.Z`.
- In the phase files, `X.Y` is the version being released, `X.Y+1` the next one.

## Interaction protocol (every step in every phase)

1. **Narrate** the step in one short sentence — what's about to happen and why.
2. **Ask** via `AskUserQuestion`: short question (e.g. "Run this now?"), header ≤12 chars, two options: "Yes, do it" / "I'll do it manually".
3. If "Yes": execute, then **show the result in a verifiable form** — PRs changed, issue URL, summary table, diff. Wait for "looks good" or fixes before moving on.
4. If "I'll do it manually": wait for the user to say it's done, then continue.

**Hard rules:**
- Never chain two mutating actions without an approval gate between them.
- Read-only queries (listing PRs, milestone state, CI status) run inline without asking.
- **Slack messages: never ask y/n.** Claude can't post to Slack — draft the message in a fenced block and tell the user where to paste it.

**Example gate:**
> **❓ Ask** — *Label PRs*: "Apply these 4 label changes?"
> Options: `Yes, do it` · `I'll do it manually`
>
> After applying: show the before → after table and wait for confirmation.

## Canonical references

- Plugin-release doc: https://developer.wordpress.org/block-editor/contributors/code/release/plugin-release/
- Auto cherry-picking: https://developer.wordpress.org/block-editor/contributors/code/release/auto-cherry-picking/
- Labels mapping: https://developer.wordpress.org/block-editor/contributors/code/release/plugin-release/#labels-mapping
- "What's new" template: https://docs.google.com/document/d/1D-MTOCmL9eMlP9TDTXqlzuKVOg_ghCPm9_whHFViqMk/edit
