# selat-openclaw-skills

OpenClaw / ClawHub skills powered by [SELAT](https://github.com/SELAT-AI) —
domain skills (deal sourcing, lead gen, social intelligence, …) that pay for
the data they use per call, in USDC, from the **user's own self-custody Circle
Agent Wallet**. No API keys, no signups, no platform balance.

Each skill here is a **thin intent wrapper** over a vetted SELAT skill. The
wrapper teaches an OpenClaw agent *when* to use the capability and how to run
it safely; the machine-readable payment recipe (endpoints, rails, price caps)
lives in [`SELAT-AI/selat-skills`](https://github.com/SELAT-AI/selat-skills)
and is fetched by the `selat` CLI at install time. Wrappers contain **no
executable code and no endpoint URLs** — installing one only ever writes text.

## Skills

| Skill | Intent | Full-run cap |
|---|---|---|
| [`vc-ai-infra-scout`](skills/vc-ai-infra-scout/SKILL.md) | VC deal sourcing across an AI-infra / crypto-AI / robotics / agentic-payments thesis — discover founders, read raise chatter, distill lead-fund theses, enrich the top lead | $0.40 |

More coming: lead enrichment, influencer discovery, social intelligence.

## How every skill works — dry run before wallets

All skills in this repo follow the same three-step spine, in this order:

1. **Dry run first (free).** `selat skill verify` probes every endpoint's live
   HTTP 402 challenge and prints the real quoted price per step — no wallet,
   no funds, no account. The agent shows the user actual prices before
   anything money-shaped exists.
2. **Wallet setup — only after the user opts in.** `selat init` creates a
   self-custody Circle Agent Wallet (MPC — SELAT never holds keys or funds);
   `selat fund` deposits USDC into Circle Gateway.
3. **Capped paid run.** `selat skill run <name>` executes the vetted recipe.
   Spend is capped per step and per run; the runner prints a per-rail receipt
   summary. The live 402 quote is always the price source of truth.

Payments settle across multiple rails — direct Circle x402, routed MPP, and
routed x402 on Base — auto-detected per step by the CLI.

## Using a skill

Install from [ClawHub](https://clawhub.ai) with the OpenClaw skills commands,
or point your agent at a `SKILL.md` in this repo. The only dependency is the
`selat` CLI, one npm package:

```bash
npm install -g @selat-ai/selat-cli
```

## License

MIT — see [LICENSE](LICENSE). Skills published to ClawHub are additionally
available under MIT-0 per ClawHub's terms.
