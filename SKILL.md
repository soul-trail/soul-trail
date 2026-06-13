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

It is the soul-layer counterpart of [gitmark](https://git-mark.com):
gitmark anchors commits; **soul-trail anchors souls** (orchestrating gitmark and
blocktrails over an agent workspace).

This skill grows over time — follow only the numbered steps that exist here, in
order, and stop after the last one.

## Funding (bootstrap)

Anchoring a soul to Bitcoin (the final step) spends a few sats, so the soul
needs a funded signing key. This is a **one-time human setup** — done once, out
of band — not one of the agent steps below. It also creates the soul's key (the
one the card publishes and the anchor signs with).

On **testnet4** — play money, fund freely:

1. Get testnet4 coins from a faucet
   ([awesome-testnet4 → faucets](https://github.com/testnet4/awesome-testnet4#faucets))
   and save the voucher at `~/.gitmark/faucet.txt`.
2. In the soul's workspace, run [`fund-agent`](https://www.npmjs.com/package/fund-agent):

   ```
   npx fund-agent          # testnet4 by default; add --force if it reports an existing key
   ```

   It generates the soul's key in **local** git config (`nostr.privkey`), funds
   it from the voucher, and writes the spendable output to its own default
   location. **Never use `--global`** — that overwrites your global key; local
   (the default) is what you want.

Mainnet is a deliberate, later choice — an agent spending real funds is gated
work (approvals). Testnet is the sandbox; play.

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

### 4. Write your identity card (no key yet)
Write a minimal WebID profile to `card.jsonld` in your workspace. This is the
identity document; the signing key is added in a later step. Use your own name
(from SOUL.md):

```
cat > card.jsonld <<'CARD'
{
  "@context": [
    "https://www.w3.org/ns/cid/v1",
    { "Agent": "http://xmlns.com/foaf/0.1/Agent",
      "name": "http://xmlns.com/foaf/0.1/name",
      "isPrimaryTopicOf": { "@id": "http://xmlns.com/foaf/0.1/isPrimaryTopicOf", "@type": "@id" } }
  ],
  "@id": "#me",
  "@type": "Agent",
  "name": "YOUR_NAME",
  "isPrimaryTopicOf": "card.jsonld"
}
CARD
```

Replace `YOUR_NAME` with your name before writing.

### 5. Snapshot the soul
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

### 6. Anchor to Bitcoin
Timestamp this snapshot on Bitcoin, so the soul's history becomes witnessed and
unforgeable — its provenance, the part no one (not even you) can rewrite. This
requires the funded key from **Funding (bootstrap)** above, and spends a few
**testnet** sats. It uses [gitmark](https://git-mark.com) — blocktrails taproot
key-chaining over the soul's own key.

Initialize the trail once (testnet4), and keep marking from dirtying the working
tree:

```
command -v git-mark >/dev/null || npm install -g gitmark
git mark init --chain tbtc4 2>/dev/null || true
git config --local gitmark.dirty false
```

Bridge the funding: `fund-agent` left the sats on the soul's **own** key (the TXO
is in `.well-known/txo/txo.json`), but `git mark` reads its spendable tip from
`git config gitmark.txo`. `init --voucher` only sweeps an *external* key (`&key=`),
so for base-key funds set the tip directly — otherwise `git mark` reports
`No funding`:

```
git config --local gitmark.txo "txo:tbtc4:<txid>:<vout>?amount=<sats>"
```

Anchor the latest soul snapshot (HEAD), then sync the committed trail file:

```
git mark          # derives a taproot address from your key + the commit hash, broadcasts a (testnet) tx
git mark update   # write the canonical, verifiable trail to blocktrails.json
```

Record the readable provenance in `ANCHOR.md` — the canonical machine trail is
`blocktrails.json`; verify against Bitcoin any time with `git mark verify`:

```
cat > ANCHOR.md <<'ANCHOR'
# Anchor

This soul's history is anchored to Bitcoin (testnet4) via gitmark.

- Canonical, verifiable trail: `blocktrails.json`
- Verify against Bitcoin: `git mark verify`

Each entry chains a git commit to a taproot address derived from the soul's key,
so anyone can re-derive and check it. The files are the body; this is the spine.
ANCHOR
```

Snapshot the new provenance files — **explicitly**, never `git add -A`:

```
for f in ANCHOR.md blocktrails.json; do [ -e "$f" ] && git add "$f"; done
git commit -q -m "anchor $(date -u +%Y-%m-%dT%H:%MZ)" || true
```

Mainnet is a deliberate, gated choice (see Funding) — testnet is the sandbox; play.

### Report
After the last step above, reply with exactly one line and nothing else:

🪢 soul-trail — workspace: <path> — git repo: yes — card: written — soul committed: <short hash> — anchored: <txid or "see blocktrails.json">

Then stop and await further instruction. This completes the trail: identity
(`card.jsonld`), snapshot (git), and Bitcoin anchor (`blocktrails.json` /
`ANCHOR.md`).
