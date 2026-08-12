---
name: financial-intel
description: "Multi-signal financial intelligence on a crypto asset, token, or equity ticker — fuses live spot price, token market data, macro/equities context, on-chain smart-money positioning, fundamentals and funding history, and current market news into one brief with citations. Use when asked \"give me a full read on ETH\", \"what's the market intel on <token>\", \"smart-money + fundamentals brief for <coin>\", \"macro + equities + crypto snapshot on <ticker>\", or \"is <asset> a buy right now\". Read-only, and paid per call in USDC from the user's own self-custody Circle Agent Wallet across three settlement rails — no API keys, no signups. Dry-run first to see live prices."
version: 1.0.0
metadata:
  openclaw:
    emoji: "📊"
    homepage: https://github.com/SELAT-AI/selat-skills/tree/main/skills/financial-intel
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

# financial-intel

A cross-source financial read on a single asset, keylessly and pay-per-run. This
skill gathers paid signal from six providers — spot price, token market data,
macro/equities, on-chain smart-money positioning, fundamentals and funding, and
market news — and hands you (the agent) the raw results to fuse into one brief:
what the asset costs, how its market is behaving, the macro backdrop, where
labeled smart money sits, the funding picture, and the news that confirms or
contradicts it.

It wraps a **SELAT skill**: a declarative, vetted recipe of paid API calls (no
API keys, no signups) settled in USDC. Signal spans **three settlement modes** —
a Circle Gateway-batched nanopayment, MPP on Tempo, and x402 on Base — which the
`selat` CLI auto-detects per step. Read-only: it never trades, posts, or mutates
anything, and it is **not** financial advice.

## Cost — read this first

- Every run makes **real paid API calls** in USDC from the **user's own Circle
  Agent Wallet** (MPC self-custody — SELAT never holds keys or funds).
- **Prices and spend limits live in the underlying SELAT skill**, not here — the
  live 402 quote from `selat skill verify`/`run` is the price source of truth, so
  this wrapper doesn't restate dollar figures (they'd only drift).
- **This is one flow, not a menu.** `selat skill run` executes every step and the
  value is in fusing them; the params retarget which asset each step reads, they
  don't let you skip steps. Budget for a full run, not a partial one.
- **Always dry-run first** (Step 1 — free, no wallet), show the user the real
  quoted prices, and get their OK before any wallet setup or paid run.
- Never ask for, paste, or handle a private key. Wallet auth is the CLI's Circle
  integration.

## Step 0 — get the CLI (free, no account)

If `selat` isn't on PATH yet, install it — one npm package, no signup:

```bash
selat --version || npm install -g @selat-ai/selat-cli
```

Installing the CLI creates nothing money-related — no wallet, no account, no keys.

## Step 1 — dry run first, before any wallet setup

**Do this before creating a wallet or asking the user to fund anything.** The dry
run probes every step's live price and reachability for free — no wallet, no
funds, no account, no `selat init`:

```bash
selat skill install financial-intel
SELAT_ROUTER_URL=https://router.selat.ai \
  selat skill verify ~/.config/selat/skills/financial-intel
```

(`verify` takes the installed skill's directory — `$XDG_CONFIG_HOME/selat/skills/<name>`,
which defaults to the path above. The `SELAT_ROUTER_URL` prefix is only needed
before `selat init` has written config.)

This prints each step's real quoted price from the live 402 challenges, plus
which rail settles it. **Show the user these prices and get their OK before
wallet setup.** If they don't want to proceed, stop here — nothing has been spent
or created.

## Step 2 — wallet setup (only after the user opts in)

```bash
selat init     # creates the self-custody Circle Agent Wallet + config
selat fund     # deposit USDC into Circle Gateway (user action)
selat doctor   # verify wallet, router, and balance are ready
```

`selat init` is safe to re-run — it detects an existing wallet and asks before
changing anything.

## Step 3 — run

Pass the params for the asset in question; unused params take safe defaults.

```bash
selat skill run financial-intel \
  --symbol ETH \
  --coin ethereum \
  --ticker AAPL \
  --chain ethereum \
  --query "ethereum ETF flows"
```

| Param | Steers | Feeds this step |
|---|---|---|
| `symbol` | Token symbol, no `$` (required). | spot price |
| `coin` | Coin id, lowercase slug. | token market data |
| `ticker` | Equity ticker for macro/risk context. | macro/equities quote |
| `chain` | Chain name for the smart-money query. | on-chain smart-money holdings |
| `query` | Free-text search for market context. | market news |

**Set the params together so the steps describe one asset** — a run with
`--symbol ETH` but a default equity ticker and a mismatched news query produces
six answers about five different things. Each step returns raw JSON; your job is
to distill it into a short brief in plain language — price and market snapshot,
macro backdrop, smart-money posture, fundamentals and funding, and the news that
supports or cuts against the read — with the dollar cost the CLI reported and a
clearly hedged conclusion. Keep endpoint URLs and raw JSON out of what you relay.
If a step returns nothing usable, say so and reason from the steps that did
return.

## Why this is safe to install

- The skill is a **declarative JSON manifest — no executable code**. Installing
  this wrapper only ever writes text.
- Endpoints are **https-only**, first-party, and pre-vetted; each publish is gated
  on a machine-checked live verification receipt.
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
