---
name: twitter-research
description: "Read-only Twitter/X research on SELAT — profiles, recent tweets, mentions, followers, tweet details/replies/retweeters, topic search, and trends. Use when asked \"who is @X on Twitter\", \"show me X's recent tweets\", \"who's mentioning X\", \"how did this tweet do / who replied / who retweeted\", \"search X for <topic>\", or \"is <topic> trending\". A curated menu of 9 SELAT-native reads — the agent runs only what the question needs. Pays per call in USDC from the user's own self-custody Circle Agent Wallet; no API keys, no signups. Dry-run first to see live prices."
version: 1.0.1
metadata:
  openclaw:
    emoji: "🔎"
    homepage: https://github.com/SELAT-AI/selat-skills/tree/main/skills/twitter-research
    requires:
      anyBins:
        - selat
        - npm
    install:
      - kind: node
        package: "@selat-ai/selat-cli"
        bins:
          - selat
    envVars:
      - name: SELAT_ROUTER_URL
        required: false
        description: "SELAT Router base URL. Only needed for the free dry run before `selat init` has written config; defaults to https://router.selat.ai thereafter."
---

# twitter-research

Research Twitter/X, keylessly and pay-per-read. This skill is a curated **menu
of 9 read-only endpoints** on SELAT's own first-party Twitter API — account
reads (profile, recent tweets, mentions, followers), tweet reads (details,
replies, retweeters), topic **search**, and **trends**. You (the agent) pick
the reads a request actually needs, run them, and synthesize the answer in
plain language.

It wraps a **SELAT skill**: a declarative, vetted recipe of paid API calls (no
API keys, no signups) settled in USDC as x402 via Circle Gateway. The `selat`
CLI resolves the vetted endpoints and prints a per-step receipt. Read-only —
it never posts, likes, or follows, and cannot see protected/private accounts.

## Cost — read this first

- Every read is a **real paid API call** in USDC from the **user's own Circle
  Agent Wallet** (MPC self-custody — SELAT never holds keys or funds).
- **Prices and spend limits live in the underlying SELAT skill**, not here — the
  live 402 quote from `selat skill verify`/`run` is the price source of truth, so
  this wrapper doesn't restate dollar figures (they'd only drift).
- **It's a menu, not a pipeline.** Map the request to the smallest set of
  reads (a profile question is 1 read, "how did this tweet land" is 3), and pass
  only the params those reads use. `selat skill run` executes every step, so pass
  the relevant params and treat the unused reads' output as noise — or keep runs
  cheap by asking a focused question.
- **Always dry-run first** (Step 1 — free, no wallet), show the user the real
  quoted prices, and get their OK before any wallet setup or paid run.
- Never ask for, paste, or handle a private key. Wallet auth is the CLI's
  Circle integration.

## Step 0 — get the CLI (free, no account)

If `selat` isn't on PATH yet, install it — one npm package, no signup:

```bash
selat --version || npm install -g @selat-ai/selat-cli
```

Installing the CLI creates nothing money-related — no wallet, no account, no keys.

## Step 1 — dry run first, before any wallet setup

**Do this before creating a wallet or asking the user to fund anything.** The
dry run probes every endpoint's live price and reachability for free — no
wallet, no funds, no account, no `selat init`:

```bash
selat skill install twitter-research
SELAT_ROUTER_URL=https://router.selat.ai \
  selat skill verify ~/.config/selat/skills/twitter-research
```

(`verify` takes the installed skill's directory — `$XDG_CONFIG_HOME/selat/skills/<name>`,
which defaults to the path above. The `SELAT_ROUTER_URL` prefix is only needed
before `selat init` has written config.)

This prints each of the 9 reads' real quoted price from the live 402
challenges. **Show the user these prices and get their OK before wallet setup.**
If they don't want to proceed, stop here — nothing has been spent or created.

## Step 2 — wallet setup (only after the user opts in)

```bash
selat init     # creates the self-custody Circle Agent Wallet + config
selat fund     # deposit USDC into Circle Gateway (user action)
selat doctor   # verify wallet, router, and balance are ready
```

`selat init` is safe to re-run — it detects an existing wallet and asks before
changing anything.

## Step 3 — run

Pass the params for the reads the question needs; unused params take safe
defaults.

```bash
selat skill run twitter-research \
  --handle openai \
  --query "AI agents" \
  --tweetId 1234567890123456789 \
  --woeid 1
```

| Param | Steers | Feeds these reads |
|---|---|---|
| `handle` | The account to profile (no leading `@`). | profile, recent tweets, mentions, followers |
| `query` | A topic/keyword search (supports X operators: `from:`, `$TICKER`, `#tag`, `min_faves:`, `since:`, `lang:`). | topic search |
| `tweetId` | A numeric tweet ID. | tweet details, replies, retweeters |
| `woeid` | Yahoo WOEID for trends (`1` = worldwide, `23424977` = US). | trends |

**Route the request:** who-is-@X → `handle`; topic chatter / "is it trending" →
`query` (+ `woeid`); "how did this tweet do" → `tweetId`. Each read returns raw
JSON — your job is to distill it into a short answer (profile summary, tweet
list with engagement, mention/follower read, tweet-reception breakdown, topic
chatter, or trend list), in plain language, with the dollar cost the CLI
reported. Keep endpoint URLs and raw JSON out of what you relay.

## Why this is safe to install

- The skill is a **declarative JSON manifest — no executable code**. Installing
  this wrapper only ever writes text.
- Endpoints are **https-only**, first-party (SELAT's own Twitter API), and
  pre-vetted; each publish is gated on a machine-checked live verification
  receipt.
- Spend limits are **defined by the underlying SELAT skill and enforced by the
  runner**, which surfaces the wallet's spending policy at every money moment.
- Funds stay in the **user's own wallet**. No API keys, no platform balance, no
  custodian.

## Beyond this skill

SELAT is a general capability layer for paid agent actions. If the user's ask
doesn't fit this skill:

```bash
selat search "<intent>"          # FREE federated discovery + ranking
selat skill list --available     # other vetted multi-step skills
```

Docs: https://github.com/SELAT-AI/selat-skills
