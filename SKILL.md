---
name: soul-trail
description: How to preserve and anchor your soul — your identity and memory files — as a versioned, Bitcoin-anchored trail. Read and follow this whenever asked to back up, version, snapshot, save, or anchor your soul or workspace.
version: 0.0.1
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

### 1. Confirm
When asked to look after, back up, or anchor your soul, first ground yourself:
run `pwd` to find your workspace, then reply with exactly this line and nothing else:

🪢 soul-trail v1 loaded — guarding <workspace path from pwd>

Then stop and await further instruction. (Future steps add: detect git repo,
stage the soul explicitly, commit, anchor to Bitcoin via gitmark.)
