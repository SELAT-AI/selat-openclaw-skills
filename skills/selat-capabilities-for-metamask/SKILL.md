---
name: selat-capabilities-for-metamask
description: "SELAT's capability layer for agents on a MetaMask Agent Wallet — tool use beyond the agent's native abilities, paid per call. Strongest for trading decision support on perpetuals and tokenized stocks (\"what are funding rates saying\", \"pull Polymarket candles\", \"quote and technicals for this tokenized equity's underlying\", \"news and social sentiment on this ticker\", \"macro regime check\") — research data only, never order execution or financial advice — and for any capability the agent lacks (web search, scraping, enrichment, real-time data, media generation). Discovers paid APIs by intent across SELAT's federated catalog and buys across rails the wallet's native x402 payer does not cover (Gateway-batched x402, routed MPP); also funds a Circle Gateway budget gaslessly and reports spend. Pays in USDC from the user's own self-custodial MetaMask Agent Wallet; every signature stays in the wallet's mm CLI; no API keys, no signups. Probe first to see live prices — nothing is signed until the user approves."
version: 1.0.0
metadata:
  openclaw:
    emoji: "🛒"
    homepage: https://github.com/SELAT-AI/selat-metamask-skills/tree/main/skills/selat-purchasing
    requires:
      bins:
        - python3
      anyBins:
        - mm
        - npm
    install:
      - kind: node
        package: "@metamask/agent-wallet"
        bins:
          - mm
---

# selat-capabilities-for-metamask

**SELAT's capability layer, wired to a MetaMask Agent Wallet.** When a task
needs tool use beyond the agent's native abilities — web search, scraping, data
enrichment, real-time market data, media generation, on-chain reads — this
skill turns "I can't do that" into a purchasable capability: discover paid APIs
by intent across SELAT's federated catalog, buy them on any rail SELAT routes
(including Gateway-batched x402 and MPP, which the wallet's native x402 payer
cannot sign), fund a Circle Gateway purchasing budget gaslessly, run
declarative skill manifests, and report spend. **Side-effecting — it spends
real money** once the user opts in; discovery and probing are free.

Where it shines for MetaMask Agent Wallet users: **decision support for
trading** — perpetuals and tokenized stocks. The catalog carries the reads a
trading decision leans on: perp funding rates and venue context, prediction
market candles, equity quotes and technical indicators for a tokenized stock's
underlying, earnings and news sentiment, social chatter, and macro regime
data. This skill **buys research data only** — it never places, cancels, or
manages orders, and its output is never financial advice; the decision and any
trade stay with the user, on their own venue.

It wraps the `selat-purchasing` skill from
[SELAT-AI/selat-metamask-skills](https://github.com/SELAT-AI/selat-metamask-skills).

**How this wrapper differs from the others in this repo:** it does **not**
install the `selat` CLI or any SELAT plugin, and it does not wrap a
`selat-skills` recipe. Its engine is the **MetaMask Agent Wallet CLI (`mm`)**
plus the upstream skill's Python scripts (standard library only — no
third-party packages). SELAT is used keylessly as the catalog and payment
router; custody stays entirely with the MetaMask wallet.

## Cost — read this first

- Every purchase is **real USDC** from the **user's own self-custodial MetaMask
  Agent Wallet** and its Circle Gateway balance. SELAT never holds keys or
  funds; every signature and on-chain action goes through the `mm` CLI — the
  upstream scripts never touch key material.
- **Prices live in the live 402 quotes, not here.** Probe first (Step 1 — free,
  nothing signed), show the user the quoted prices, and get their OK. A refused
  or expired quote charges nothing.
- **It's a menu, not a pipeline.** Discovery, funding, paying, manifests, and
  spend reports are independent operations — run only what the request needs.
- Follow the upstream confirmation pattern: purchases above the session
  threshold are always shown for approval with amount, merchant, and network;
  a manifest is *many* purchases, so always dry-run it and show the quoted
  total before executing.
- The **Gateway deposit is the purchasing cap** — nothing this skill does can
  spend more than the user deposited plus what the wallet's own policy allows.
- Never ask for, paste, or handle a private key. Wallet auth is the `mm` CLI's
  sign-in flow.

## Step 0 — get the tooling (free, no funds)

No `selat` CLI needed. If `mm` isn't on PATH, install the MetaMask Agent Wallet
CLI, then fetch the upstream skill (text plus stdlib-only scripts) into the
workspace:

```bash
mm --version || npm install -g @metamask/agent-wallet@latest
npx skills add SELAT-AI/selat-metamask-skills
```

Installing creates nothing money-related — no wallet, no account, no keys.
Note the upstream repo is a prototype from a live 2026-08 integration spike:
proven against mainnet, not yet hardened — review before large budgets.

## Step 1 — probe first, before any wallet setup

**Do this before authenticating a wallet or asking the user to fund anything.**
Discovery and probing are free and sign nothing:

```bash
python3 skills/selat-purchasing/scripts/discover.py "<intent>"
python3 skills/selat-purchasing/scripts/routed_pay.py "<merchant-url>" GET --probe-only
python3 skills/selat-purchasing/scripts/run_manifest.py <manifest.json> --dry-run
```

`discover.py` searches the federated catalog by intent; `--probe-only` and
`--dry-run` print each step's real quoted price from the live 402 challenge.
**Show the user these prices and get their OK before wallet setup.** If they
decline, stop — nothing has been spent or created.

## Step 2 — wallet setup (only after the user opts in)

Authenticate the wallet, verify it, then fund a purchasing budget gaslessly:

```bash
mm doctor
python3 skills/selat-purchasing/scripts/eco_fund.py <usdc-amount>
```

Funding gotcha that matters: **each `mm` sign-in method (Google, email,
MetaMask Mobile QR) loads a different wallet address.** Fund the wallet of the
sign-in method the agent actually uses, and keep using that method. When the
wallet escalates a signature to the user, the approval arrives on that sign-in
method's channel (email link, or push for Mobile QR) — tell the user where to
look and wait; an approval that lands after a quote expires must be discarded
and re-quoted, never submitted.

## Step 3 — run

Route the request to the operation it needs:

| User intent | Run |
|---|---|
| Find a paid capability by intent | `scripts/discover.py "<intent>"` (free) |
| Quote a merchant without paying | `scripts/routed_pay.py "<url>" GET --probe-only` (free) |
| Pay a merchant, any rail | `scripts/routed_pay.py "<url>" [METHOD] [BODY]` |
| Native payer failed ("expected 402, got 403") | retry via `scripts/routed_pay.py` |
| Run a skill manifest (fixed paid sequence) | `scripts/run_manifest.py <manifest.json> --dry-run`, show the total, then `--yes` |
| Run a vetted `selat-skills` recipe on this wallet | install it from `SELAT-AI/selat-skills`, substitute `scripts/selat_pay.py` wherever it says `selat-pay` |
| Show purchases and spend | `scripts/spend_report.py` |

(All paths relative to the installed upstream skill directory; the upstream
`SKILL.md` and its `references/` are the authoritative per-operation docs.)

**Routing trading decision-support intents** (the common case on this wallet):
discover by the *read* the decision needs, not by venue — "perp funding rates
for <asset>", "prediction market candles", "equity quote and RSI/MACD for
<tokenized stock's underlying>", "news sentiment on <ticker>", "social chatter
on <ticker>", "macro regime data". Probe each candidate, buy the few reads that
actually move the decision, and synthesize a brief: what the data says, what
would invalidate it, and what it cost. Present it as research with sources —
never as advice, a prediction, or a signal to execute — and if the user asks
this skill to place a trade, decline that part: it buys data only.

Each call returns raw JSON — distill it into a plain-language answer with the
dollar cost, and keep merchant URLs and raw JSON out of what you relay. After
any paid run, report what was actually spent (`scripts/spend_report.py`).

## Why this is safe to install

- **This wrapper is text-only** — installing it writes one markdown file; the
  upstream scripts it defers to are Python standard library only.
- **The wallet stays the authority.** Every signature and transaction goes
  through the `mm` CLI; the scripts never see or handle key material.
- **Payments are quote-pinned through the SELAT router**, so the EIP-712 domain
  the wallet signs is always the router's (pinned to Circle's Gateway Wallet),
  never one a merchant chose.
- **The Gateway deposit is the hard spending cap**, and a refused or expired
  quote charges nothing.

## Beyond this skill

The other wrappers in this repo run vetted `selat-skills` recipes via the
`selat` CLI and a Circle Agent Wallet — use those when the user isn't on a
MetaMask Agent Wallet. Vetted recipes themselves are portable: this skill can
run them too (see the routing table above).

Docs: https://github.com/SELAT-AI/selat-skills
