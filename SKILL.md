---
name: soul-trail
description: How to preserve and anchor your soul — your identity and memory files — as a versioned, Bitcoin-anchored trail. Read and follow this whenever asked to back up, version, snapshot, save, or anchor your soul or workspace.
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
Check whether the workspace is its **own** git repository — not merely inside
some ancestor repo (a home directory or parent can be a repo, which would
otherwise mislead you):

```
[ "$(git rev-parse --show-toplevel 2>/dev/null)" = "$(pwd)" ] && echo own-repo || echo not-own-repo
```

`own-repo` means this workspace is its own git root; `not-own-repo` means it is
not yet a repo of its own (even if an ancestor directory happens to be one).

### 3. Initialize if needed
Only if step 2 reported `not-own-repo`, make this workspace its own repository:

```
git init -q
```

This creates an independent repo for the soul, even if an ancestor directory is
already a repo (the new inner repo governs this subtree — intended). If step 2
reported `own-repo`, do nothing here.

### 4. Snapshot the soul
Stage **only** the durable soul files — never `git add -A` or `git add .`, since
the workspace may hold scratch files or downloads that must not be committed or
anchored. Then commit:

```
for f in SOUL.md IDENTITY.md AGENTS.md TOOLS.md USER.md HEARTBEAT.md MEMORY.md card.jsonld memory skills; do
  [ -e "$f" ] && git add "$f"
done
git commit -q -m "soul snapshot $(date -u +%Y-%m-%dT%H:%MZ)" || true
```

The commit hash is your soul's fingerprint. Capture it: `git rev-parse --short HEAD`.

### Report
After the last step above, reply with exactly one line and nothing else:

🪢 soul-trail — workspace: <path> — git repo: yes — soul committed: <short hash>

Then stop and await further instruction. (Future steps add: git init, stage the
soul explicitly, commit, anchor to Bitcoin via gitmark.)
