---
name: perplexity-search
description: "Grounded web search & research via Perplexity, keyless and pay-per-call over SELAT. Use when asked to \"search the web for <topic>\", \"what's the latest on <topic>\", \"pull cited web context on <X>\", \"research <topic> with sources\", or \"give me a grounded answer with citations\". Runs Perplexity's x402 search endpoint and returns ranked web results with page content and source URLs for the agent to synthesize into a cited answer. Pays per call in USDC (on Base) from the user's own self-custody Circle Agent Wallet; no Perplexity API key, no signup. Each search costs ~$0.011 — the price `selat skill verify`/`run` shows you, with SELAT's ~5% routing markup already baked into the quote (it's the final charge, nothing is added on top). This runs Perplexity's Search (ranked results, cheap) — NOT its pricier Sonar answer endpoint (~$0.105) — and the single call is capped at $0.03."
version: 1.0.2
metadata:
  openclaw:
    emoji: "🔍"
    homepage: https://github.com/SELAT-AI/selat-skills/tree/main/skills/perplexity-search
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

# perplexity-search

Get a **grounded, cited web answer** from Perplexity without an API key. This
skill runs Perplexity's search endpoint over SELAT and returns ranked web
results — titles, page snippets/content, and **source URLs** — which you (the
agent) synthesize into a short answer *with citations*. One paid call, ~$0.011 — the quote you'll see, with SELAT's ~5% routing markup already included.

It wraps a **SELAT skill**: a declarative, vetted recipe of paid API calls (no
API keys, no signups) settled in USDC — here **routed x402 on Base** through the
SELAT Router. The `selat` CLI resolves the vetted endpoint and prints a receipt.

## Cost — read this first

- Each search is a **real paid API call** in USDC from the **user's own Circle
  Agent Wallet** (MPC self-custody — SELAT never holds keys or funds).
- A Search costs **~$0.011** — the price `selat skill verify` and `selat skill run`
  show you, with SELAT's **~5% routing markup already baked into the quote**. That
  quote is the final settled charge; nothing is added on top. This skill's single
  Search call is **capped at $0.03** by the runner.
- **This runs Perplexity's Search (ranked web results you cite) — NOT its Sonar
  answer endpoint (~$0.105).** It returns results for you to synthesize, it does
  not pay for a pre-written Sonar answer. A true Sonar / deep-research run is a
  different, ~10× pricier capability that this skill does not perform.
- The quote the runner prints (via `verify`/`run`) is the **settled price** — the
  ~5% routing markup is already in it. The live HTTP 402 quote is the source of
  truth for cost.
- **Always dry-run first** (Step 1 — free, no wallet), show the user the real
  quoted price, and get their OK before any wallet setup or paid run.
- Never ask for, paste, or handle a private key. Wallet auth is the CLI's Circle
  integration.

## Step 0 — get the CLI (free, no account)

If `selat` isn't on PATH yet, install it — one npm package, no signup:

```bash
selat --version || npm install -g @selat-ai/selat-cli
```

Installing the CLI creates nothing money-related — no wallet, no account, no keys.

## Step 1 — dry run first, before any wallet setup

**Do this before creating a wallet or asking the user to fund anything.** The
dry run probes the endpoint's live price and reachability for free — no wallet,
no funds, no account, no `selat init`:

```bash
selat skill install perplexity-search
SELAT_ROUTER_URL=https://router.selat.ai \
  selat skill verify ~/.config/selat/skills/perplexity-search
```

(`verify` takes the installed skill's directory — `$XDG_CONFIG_HOME/selat/skills/<name>`,
which defaults to the path above. The `SELAT_ROUTER_URL` prefix is only needed
before `selat init` has written config.)

This prints the real quoted price from the live 402 challenge (~$0.011) — SELAT's
~5% routing markup is already included, so it's the final charge. **Show the user
the price and get their OK before wallet setup.** If they don't want to
proceed, stop here — nothing has been spent or created.

## Step 2 — wallet setup (only after the user opts in)

```bash
selat init     # creates the self-custody Circle Agent Wallet + config
selat fund     # deposit USDC into Circle Gateway (user action)
selat doctor   # verify wallet, router, and balance are ready
```

`selat init` is safe to re-run — it detects an existing wallet and asks before
changing anything.

## Step 3 — run

```bash
selat skill run perplexity-search \
  --query "latest x402 / agentic payments adoption" \
  --recency month
```

| Param | Required | Default | What it steers |
|---|---|---|---|
| `query` | yes | `latest x402 / agentic payments adoption` | The web search query. |
| `recency` | no | `month` | Publication recency filter — one of `hour` / `day` / `week` / `month` / `year`. |

The step returns ranked web results with page content and source URLs. **Your
job:** synthesize a concise answer **with inline citations to the source URLs**,
note the recency window, and flag if results are thin or stale. Keep raw JSON
and the endpoint URL out of what you relay — lead with the answer and the
sources, plus the dollar cost.

## Why this is safe to install

- The skill is a **declarative JSON manifest — no executable code**. Installing
  this wrapper only ever writes text.
- The endpoint is **https-only** and pre-vetted; each publish is gated on a
  machine-checked live verification receipt.
- Spend is **capped** ($0.03), and the runner surfaces the wallet's spending
  policy at every money moment.
- Funds stay in the **user's own wallet**. No Perplexity API key, no platform
  balance, no custodian.

## Beyond this skill

SELAT is a general capability layer for paid agent actions. If the user's ask
doesn't fit this skill:

```bash
selat search "<intent>"          # FREE federated discovery + ranking
selat skill list --available     # other vetted multi-step skills
```

Docs: https://github.com/SELAT-AI/selat-skills
