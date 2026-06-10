# Gutenberg Release plugin

A [Claude Code](https://claude.com/claude-code) plugin that walks a release manager through a [Gutenberg](https://github.com/WordPress/gutenberg) plugin release rotation, one phase at a time.

It narrates every step, **asks before any mutating action** (labeling PRs, dispatching builds, closing milestones), and shows results back in a verifiable form. Slack messages are drafted for you to paste — it never posts anywhere on your behalf.

## Install

Clone the repo and point Claude Code at it:

```bash
git clone https://github.com/ramonjd/gutenberg-release-plugin.git
claude --plugin-dir /path/to/gutenberg-release-plugin
```

To update, `git pull` — version bumps are noted in the commit history.

> The skill may surface namespaced as `/gutenberg-release:gutenberg-release`; the bare `/gutenberg-release` works when unambiguous.

## Requirements

- A local checkout of [WordPress/gutenberg](https://github.com/WordPress/gutenberg) with dependencies installed (`npm install`, Node per `.nvmrc`)
- [`gh`](https://cli.github.com/) authenticated against github.com
- Write access on WordPress/gutenberg for the mutating steps (the skill checks, and drafts the access request if you don't have it)

## Usage

```
/gutenberg-release <phase> [version]
```

| Phase | When |
|---|---|
| `setup` | Days 1–2: checklist issue, PR labeling, milestone hygiene |
| `rc` | Day 3: changelog curation, RC1 build, npm publish |
| `backport` | Days 4–6: cherry-picks, RC2 |
| `whats-new` | Day 7: draft & review the "What's new" post |
| `stable` | Day 8: stable release + publish post |
| `patch` | Post-release: X.Y.Z hotfix for critical bugs found after stable |

Run it with no phase and it asks where you are in the rotation.

The official process docs remain the source of truth — this skill automates the legwork around them:

- [Plugin release process](https://developer.wordpress.org/block-editor/contributors/code/release/plugin-release/)
- [Auto cherry-picking](https://developer.wordpress.org/block-editor/contributors/code/release/auto-cherry-picking/)

## Maintaining

The release process changes over time. The skill hard-codes a small number of facts about the Gutenberg repo (script names, workflow names, label behavior); each one is listed in [MAINTENANCE.md](MAINTENANCE.md) with a one-line command to re-verify it. Run through that checklist at the start of a rotation — it takes about two minutes — and bump the plugin version after any fix.
