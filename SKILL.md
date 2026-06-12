---
name: soul-trail
description: How to preserve and anchor your soul — your identity and memory files — as a versioned, Bitcoin-anchored trail. Read and follow this whenever asked to back up, version, snapshot, save, or anchor your soul or workspace.
version: 0.0.2
license: AGPL-3.0-or-later
uses: [gitmark, blocktrails]
---
# soul-trail

Your **soul** is the durable content of your workspace: `SOUL.md`, `MEMORY.md`,
the identity files, `memory/`, and `skills/`. This skill is how you look after
it — turning it into a git history and anchoring that history to Bitcoin, so an
agent's identity becomes portable, versioned, and cryptographically timestamped.

It is the soul-layer counterpart of [gitmark](https://github.com/melvincarvalho):
gitmark anchors commits; **soul-trail anchors souls** (orchestrating gitmark and
blocktrails over an agent workspace).

This skill grows over time — follow only the numbered steps that exist here, in
order, and stop after the last one.

## Steps

When asked to look after, back up, or anchor your soul, run these steps in
order and stop after the last one. **Verify with tools — never report a state
you did not check.**

### 1. Ground
Run `pwd` to find your workspace directory.

### 2. Detect repo
Run `git rev-parse --is-inside-work-tree 2>/dev/null` in the workspace. It
prints `true` if this is already a git repository; nothing (or an error) if not.

### Report
After the last step above, reply with exactly one line and nothing else:

🪢 soul-trail v2 — workspace: <path from pwd> — git repo: <yes if true, else no>

Then stop and await further instruction. (Future steps add: git init, stage the
soul explicitly, commit, anchor to Bitcoin via gitmark.)
