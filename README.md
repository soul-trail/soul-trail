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

### The stack — every moving part

| Component | Role | npm | source |
|---|---|---|---|
| **jsclaw** | agent orchestration / gateway (channels: webchat, Telegram, Nostr) | [`jsclaw`](https://www.npmjs.com/package/jsclaw) | [jsclaw/jsclaw](https://github.com/jsclaw/jsclaw) |
| **agent-micro** | zero-dep agent runner — drives the model loop; jsclaw's `localRunner` | [`jsclaw-agent-micro`](https://www.npmjs.com/package/jsclaw-agent-micro) | [jsclaw/agent-micro](https://github.com/jsclaw/agent-micro) |
| **JavaScriptSolidServer** (jss) | the Solid server | [`javascript-solid-server`](https://www.npmjs.com/package/javascript-solid-server) | [JavaScriptSolidServer](https://github.com/JavaScriptSolidServer/JavaScriptSolidServer) |
| **jspod** | batteries-included Solid pod (bundles jss) | [`jspod`](https://www.npmjs.com/package/jspod) | [JavaScriptSolidServer/jspod](https://github.com/JavaScriptSolidServer/jspod) |
| **podwire** | agent ↔ pod read/write, NIP-98 signed (get → edit → put) | [`podwire`](https://www.npmjs.com/package/podwire) | [jsclaw/podwire](https://github.com/jsclaw/podwire) |
| **fund-agent** | fund the soul's key from a testnet voucher | [`fund-agent`](https://www.npmjs.com/package/fund-agent) | [blocktrails/fund-agent](https://github.com/blocktrails/fund-agent) |
| **gitmark** | anchor git commits to Bitcoin (taproot key-chaining) | [`gitmark`](https://www.npmjs.com/package/gitmark) | [solidpayorg/gitmark](https://github.com/solidpayorg/gitmark) · [git-mark.com](https://git-mark.com) |
| **soul-trail** | anchor _souls_ (this skill) | _(skill)_ | [soul-trail/soul-trail](https://github.com/soul-trail/soul-trail) |
| **blocktrails** | Bitcoin taproot key-chaining substrate | — | [blocktrails.org](https://blocktrails.org) |
| pod payments | sat balance + pay-to-access on the pod | _(in jss)_ | `/pay` + webledger |

The soul itself is just files: `SOUL.md`, `IDENTITY.md`, `MEMORY.md` + `memory/`,
`AGENTS.md`, `HEARTBEAT.md`, `skills/`, `card.jsonld` (WebID), `ANCHOR.md`
(provenance).

### Run the whole stack

```bash
# 1. A Solid pod — hosts the WebID + the soul's data (batteries-included; bundles jss)
npx jspod                                  # → http://localhost:5444

# 2. The agent runtime
npm i -g jsclaw
jsclaw onboard                             # pick model/provider, scaffold an agent
jsclaw gateway --port 18790                # run it; agent-micro is the runner it drives
#   (agent-micro: github:jsclaw/agent-micro — set as jsclaw's localRunner)

# 3. Agent ↔ pod  (NIP-98 signed with the soul's key)
npm i -g podwire
POD_NOSTR_KEY=<hex> podwire get https://your.pod/public/tracker/todo-data.jsonld
POD_NOSTR_KEY=<hex> podwire put https://your.pod/...     # full body on stdin

# 4. Fund the soul's key  (testnet — voucher in ~/.gitmark/faucet.txt; see SKILL.md)
npx fund-agent                             # tbtc4; writes git config nostr.privkey + funds it

# 5. Anchor the soul to Bitcoin
npm i -g gitmark
git mark init --chain tbtc4
git commit -m "soul snapshot" && git mark  # timestamp HEAD on Bitcoin
git mark verify                            # re-derive + check the trail
```

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

## Setup & configuration

Enough for an agent to stand the stack up from scratch.

**Prerequisites:** Node 18+, git, a model-provider key (Anthropic / Z.ai-GLM /
Kimi / any Anthropic-compatible proxy). For funding + anchoring, a testnet4
voucher (see [`SKILL.md`](./SKILL.md) → Funding).

### 1. Configure the agent (`jsclaw`)

Easiest path: `jsclaw onboard` — interactive; picks a provider/model, writes
`jsclaw.json`, scaffolds an agent workspace. Or write `jsclaw.json` by hand:

```jsonc
{
  "model": "kimi-for-coding",
  "providerBaseUrl": "https://api.kimi.com/coding",  // omit for Anthropic direct
  "providerAuthToken": "${KIMI_API_KEY}",            // ${VAR} is read from the env
  "gatewayToken": "<shared-secret>",                 // auth for the gateway WS/MCP + chat
  "localRunner": "<path>/agent-micro/runner.js",     // the runner jsclaw drives
  "sandboxMode": "off",                              // off = run the runner natively
  "channels": {
    "telegram": { "botToken": "${TELEGRAM_BOT_TOKEN}", "allowFrom": ["<telegram-user-id>"] },
    "nostr":    { "privateKey": "${AGENT_NOSTR_KEY}", "relays": ["wss://relay.example"], "allowFrom": ["<owner-pubkey-hex>"] }
  },
  "plugins": { "load": { "paths": [] } }
}
```

Provider presets — set the matching env var (`${VAR}` is expanded at load):

| Provider | `model` | `providerBaseUrl` | key env (sent as) |
|---|---|---|---|
| Anthropic (direct) | `claude-sonnet-4-6` | _(default)_ | `ANTHROPIC_API_KEY` |
| Z.ai / GLM | `glm-4.6` | `https://api.z.ai/api/anthropic` | `ZAI_API_KEY` (authToken) |
| Kimi Code | `kimi-for-coding` | `https://api.kimi.com/coding` | `KIMI_API_KEY` (authToken) |
| Custom proxy (LiteLLM, …) | _your model_ | _your proxy_ | `ANTHROPIC_AUTH_TOKEN` |

### 2. Run the gateway

```bash
export KIMI_API_KEY=...  TELEGRAM_BOT_TOKEN=...  AGENT_NOSTR_KEY=...
jsclaw gateway --port 18790
# webchat: http://127.0.0.1:18790/chat?token=<gatewayToken>
```

`agent-micro` is the runner jsclaw spawns per session (set as `localRunner`); it
receives the provider env + model over stdin and drives the model loop. Channels
(`webchat`, `telegram`, `nostr`) only accept messages from the `allowFrom` lists.

### 3. Stand up the pod

```bash
npx jspod                       # single-user Solid pod → http://localhost:5444
```

The WebID is served at `http://localhost:5444/profile/card.jsonld#me`. Publish
the soul's **public** key into it as a CID Multikey `verificationMethod`
(referenced from `authentication`) so NIP-98 writes authenticate to the WebID.
`podwire` then reads/writes pod resources with the soul's key; cross-pod writes
need a WAC `.acl` granting `did:nostr:<pubkey>`.

### 4. The soul's key

One secp256k1 key at `git config --local nostr.privkey` (rung 0). `fund-agent`
generates and funds it; `gitmark` and `podwire` sign with it; the WebID
publishes its pubkey. Keep it **local**, never `--global`, **testnet first**.

---

## Status

Early — grown one verified micro-step at a time (see `SKILL.md`). A SKILL.md
skill: it works with [jsclaw](https://jsclaw.dev) agents and any
openclaw/Anthropic SKILL.md runtime.

## License

AGPL-3.0-or-later.
