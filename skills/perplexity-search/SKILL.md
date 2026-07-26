---
name: perplexity-search
description: "Grounded web search & research via Perplexity, keyless and pay-per-call over SELAT. Use when asked to \"search the web for <topic>\", \"what's the latest on <topic>\", \"research <topic> with sources\", \"do a deep-research report on <X>\", or \"give me a grounded answer with citations\". Runs Perplexity's cheap web Search by default, and can escalate to a one-shot Agent answer or an async deep-research report when a plain search isn't enough. Paid per call in USDC (on Base) from the user's own self-custody Circle Agent Wallet — no Perplexity API key, no signup. Dry-run first to see live prices; every price the CLI shows already includes SELAT's ~5% routing markup."
version: 1.1.2
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

A keyless **Perplexity research toolkit** over SELAT. The **default** action is a
cheap web **Search** — ranked results with page content and **source URLs** that
you (the agent) synthesize into a cited answer. When a search-and-synthesize pass
isn't enough, the skill also documents two **escalations** — a one-shot **Agent
answer** and an async **deep-research** report — that you run per the installed
skill's own guidance, always confirming the cost first.

It wraps a **SELAT skill**: a declarative, vetted recipe of paid API calls (no API
keys, no signups) settled in USDC — **routed x402 on Base** through the SELAT
Router. The `selat` CLI resolves the vetted endpoints and prints a receipt.

## Cost — read this first

- Every call is **real USDC** from the **user's own Circle Agent Wallet** (MPC
  self-custody — SELAT never holds keys or funds).
- **Prices and spend limits live in the underlying SELAT skill**, not here — the
  live 402 quote from `selat skill verify`/`run` is the price source of truth, so
  this wrapper doesn't restate dollar figures (they'd only drift).
- **The quote the CLI shows already includes SELAT's ~5% routing markup** — it's
  the final settled charge; nothing is added on top.
- The default **Web Search** is the wired `selat skill run` step. **Agent answer**
  and **deep research** are **separate, agent-run** paid calls — each its own
  spend, so **tell the user the cost (from its live quote) and get a yes before
  every escalation**.
- **Always dry-run first** (Step 1 — free, no wallet) for the default step, show
  the user the real quoted price, and get their OK before any wallet setup or paid
  run.
- Never ask for, paste, or handle a private key. Wallet auth is the CLI's Circle
  integration.

## Step 0 — get the CLI (free, no account)

```bash
selat --version || npm install -g @selat-ai/selat-cli
```

Installing the CLI creates nothing money-related — no wallet, no account, no keys.

## Step 1 — dry run first, before any wallet setup

**Do this before creating a wallet or asking the user to fund anything.** The dry
run probes the default step's live price and reachability for free — no wallet, no
funds, no account, no `selat init`:

```bash
selat skill install perplexity-search
SELAT_ROUTER_URL=https://router.selat.ai \
  selat skill verify ~/.config/selat/skills/perplexity-search
```

(`verify` takes the installed skill's directory — `$XDG_CONFIG_HOME/selat/skills/<name>`,
which defaults to the path above. The `SELAT_ROUTER_URL` prefix is only needed
before `selat init` has written config.)

This prints the default Search's real quoted price (markup included). **Show the
user the price and get their OK before wallet setup.** If they don't want to
proceed, stop here — nothing has been spent or created.

## Step 2 — wallet setup (only after the user opts in)

```bash
selat init     # creates the self-custody Circle Agent Wallet + config
selat fund     # deposit USDC into Circle Gateway (user action)
selat doctor   # verify wallet, router, and balance are ready
```

`selat init` is safe to re-run — it detects an existing wallet and asks before
changing anything.

## Step 3 — run the default web search

```bash
selat skill run perplexity-search \
  --query "latest x402 / agentic payments adoption" \
  --recency month
```

| Param | Required | Default | What it steers |
|---|---|---|---|
| `query` | yes | `latest x402 / agentic payments adoption` | The web search query. |
| `recency` | no | `month` | Publication recency filter — one of `hour` / `day` / `week` / `month` / `year`. |

The step returns ranked web results with page content and source URLs. **Your job:**
synthesize a concise answer **with inline citations to the source URLs**, note the
recency window, and flag if results are thin or stale. Keep raw JSON out of what you
relay — lead with the answer, the sources, and the dollar cost the CLI reported.

## Escalating beyond a plain search

When one search pass isn't enough, the installed skill documents two higher-tier
moves — **Agent answer** (a one-shot synthesized answer) and **deep research** (an
async `sonar-deep-research` report, kicked off once then polled for free until it
completes). These are **agent-run**, not part of the default `selat skill run`. To
use them:

1. **Get the live price for the call, tell the user, and get a yes** before
   spending.
2. Follow the exact request shapes and steps in the installed skill's own docs —
   its `SKILL.md` ("Escalations") and `references/endpoints.md` — which carry the
   pinned schemas and the poll loop. Don't guess the request body.
3. Synthesize the result into a cited brief, same as the default step.

## Why this is safe to install

- The skill is a **declarative JSON manifest — no executable code**. Installing this
  wrapper only ever writes text.
- Endpoints are **https-only** and pre-vetted; each publish is gated on a
  machine-checked live verification receipt.
- Each paid call's **spend limit is defined by the underlying SELAT skill and
  enforced by the runner**; escalations are each their own cost-confirmed call, and
  the runner surfaces the wallet's spending policy at every money moment.
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
