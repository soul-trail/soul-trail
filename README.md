# soul-trail

A skill that gives an AI agent's **soul** — its identity and memory — a
versioned, Bitcoin-anchored trail. It is the soul-layer counterpart of
[gitmark](https://git-mark.com): gitmark anchors commits; **soul-trail anchors
souls**.

The short version: an agent's soul is a set of files, and standards are
converging on that (persona, identity, memory, instructions, heartbeat).
soul-trail adds the one thing files can't give themselves — an **immutable,
witnessed past** — by anchoring the soul's git history to Bitcoin. The files are
the body and mind; the anchor is the **spine** that proves they're continuous.

---

## For an LLM picking this up

This section is a map of the whole system, so an agent (or an LLM) can join the
dots and **build it up, tear it down, and progress it**. The numbered, runnable
steps live in [`SKILL.md`](./SKILL.md); this is the *why* and the *shape*.

### The one key — the spine of everything

The entire stack converges on a **single secp256k1 key**, stored at rung-0 as
`git config nostr.privkey`. gitmark states it plainly: *the same key is used for
Nostr, Bitcoin, and Solid pod authentication.* So one key is, at once:

- **Identity** — `did:nostr:<pubkey>`, published in the WebID (`card.jsonld`) as
  a [CID](https://www.w3.org/TR/cid-1.0/) Multikey `verificationMethod`.
- **Pod auth** — a [NIP-98](https://nips.nostr.com/98) Nostr signature
  authenticates the agent's reads/writes to its Solid pod.
- **Money** — the same key (BIP-341 taproot) holds funds and signs Bitcoin
  anchors.

Custody is a **ladder**. Rung 0 is `git config nostr.privkey` (local, plaintext)
— fine for testnet play. Higher rungs (encrypted file → OS keychain → KMS/HSM →
MPC/threshold) keep the same seam: `sign(hash) → signature`, never expose the
key. Don't promote to mainnet value without climbing the ladder.

### The pieces

| Layer | What | Tool |
|---|---|---|
| Self | the soul files (`SOUL.md`, `IDENTITY.md`, `MEMORY.md`, `AGENTS.md`, `HEARTBEAT.md`, `skills/`, `card.jsonld` WebID, `ANCHOR.md` provenance) | this skill |
| Identity | `did:nostr` + WebID; the key published as a `verificationMethod` | `card.jsonld` |
| Pod | a Solid pod hosts the WebID + the agent's data | jspod / JavaScriptSolidServer |
| Pod I/O | read/write pod resources, NIP-98-signed (get → edit → put) | [podwire](https://www.npmjs.com/package/podwire) |
| Funding | fund the soul's key from a testnet voucher | [fund-agent](https://www.npmjs.com/package/fund-agent) |
| Anchor | timestamp the soul's commits on Bitcoin | [gitmark](https://www.npmjs.com/package/gitmark) |
| Payments | the funded key holds a sat balance, pays for gated resources | pod `/pay` + webledger |

### Build it up

The soul's lifecycle, in order (the runnable form is `SKILL.md`):

1. **Repo** — the soul's workspace becomes its own git repo (`git init`).
2. **Identity** — write `card.jsonld`, a minimal WebID.
3. **Key + funds** (bootstrap, one-time, human) — `fund-agent` generates the one
   key into `git config nostr.privkey` and funds it from a testnet voucher
   (`~/.gitmark/faucet.txt`). Testnet = play money.
4. **Publish the key** — add the pubkey to `card.jsonld` as a CID Multikey
   `verificationMethod` (encoding: `f` + `e701` + `02` + x-only-pubkey),
   referenced from `authentication`. Now a pod can resolve the key to the WebID.
5. **Pod** — serve the WebID from a pod; the key reads/writes pod resources via
   `podwire` (public read, NIP-98-signed write). Writing to *another* pod needs a
   WAC `.acl` granting `did:nostr:<pubkey>`.
6. **Anchor** — `git mark` timestamps each soul snapshot on Bitcoin. The
   canonical trail is `blocktrails.json` (committed, verifiable); `ANCHOR.md` is
   the human-readable face.

### Tear it down

- Stop the agent / gateway.
- Delete the pod's data dir (the WebID + resources).
- The soul *is* its repo: remove the working tree, or just `.git` to drop the
  history.
- The key lives at `git config --local nostr.privkey` — destroy it (and clear
  `gitmark.txo` / `gitmark.network`) to retire the soul.
- On **testnet**, funds are worthless — abandon freely.
- **Never** touch a *global* key or another repo's key.

### Progress it

- **Approvals** — autonomous *spending* (as opposed to receiving / anchoring)
  must be gated by human or policy approval before it touches real value.
- **Payments** — deposit the soul's funded TXO into the pod's webledger
  (`/pay/.deposit` takes a TXO URI) → the soul has a sat balance → pay-to-access;
  Lightning; then mainnet.
- **Mainnet anchoring** — once the testnet trail is proven.
- **Distribution** — publish the skill and tools to skill registries.

### Safety rails (non-negotiable)

- **Never `--global`** with fund-agent / gitmark — it would overwrite a global
  key. Local only.
- **Testnet (`tbtc4`) first.** Mainnet is a deliberate, separate choice.
- **Explicit staging only** — the soul commits *named* files; never `git add -A`.
  Keep the tree clean while anchoring with `git config gitmark.dirty false`.
- **The key is bearer value.** `git config nostr.privkey` is a secret; a voucher
  carrying `&key=` is spendable cash — reference TXOs by `&pubkey=` instead.

### The mental model

- The files are the soul's **body and mind** — mutable, present-tense,
  custodian-dependent.
- The one key is its **hand** — it acts: signs its identity, writes its pod,
  holds its money.
- gitmark + `ANCHOR.md` are its **spine / keel** — provenance: the witnessed past
  that proves it is the same soul over time.
- Bitcoin is the **witness** — continuity without a custodian.

---

## Status

Early — grown one verified micro-step at a time (see `SKILL.md`). A SKILL.md
skill: it works with [jsclaw](https://jsclaw.dev) agents and any
openclaw/Anthropic SKILL.md runtime.

## License

AGPL-3.0-or-later.
