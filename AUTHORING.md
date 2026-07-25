# Authoring a ClawHub wrapper `SKILL.md` — SOP

How to write a **wrapper skill** for this repo: a thin, text-only OpenClaw /
ClawHub `SKILL.md` that teaches an agent *when* to use a capability and how to
run it **safely and keylessly** over a vetted [SELAT](https://github.com/SELAT-AI/selat-skills)
skill. Keep this open while you write; every rule here is load-bearing.

## What a wrapper is (and is not)

- A wrapper is **one `SKILL.md`, nothing else** — no scripts, no code, no
  endpoint URLs. Installing it only ever writes text.
- It wraps **exactly one vetted SELAT skill** from `SELAT-AI/selat-skills`. The
  machine-readable payment recipe (endpoints, rails, price caps) lives *there*
  and is fetched by the `selat` CLI at install/run time — never duplicated here.
- Its job is **intent + safety**: tell the agent when the capability applies,
  make it dry-run before touching wallets, and cap spend. The heavy lifting
  (the actual paid calls) is the `selat` CLI's.

**Do NOT wrap a SELAT skill that isn't merged and passing `verify` in
`selat-skills`.** The wrapper is only as trustworthy as the recipe underneath.

## Workflow

1. **Pick the SELAT skill.** It must already be merged in `selat-skills/skills/<name>/`
   with a passing verify receipt. Read its `manifest.json` (params, per-step and
   full-run `maxAmount`, rails) and `SKILL.md` (intent, workflow, gotchas).
2. **Scaffold.** `mkdir -p skills/<name>` — the folder name **==** the SELAT
   skill name **==** the wrapper's frontmatter `name`.
3. **Write the frontmatter** (schema below).
4. **Write the body** (required sections below).
5. **Add a row** to the README "Skills" table.
6. **Self-check** against the checklist, then open a PR.

Copy an existing wrapper (`skills/vc-ai-infra-scout/SKILL.md`,
`skills/x-ray/SKILL.md`) as the starting shape.

## Frontmatter schema

```yaml
---
name: <kebab-name>                 # == folder == the SELAT skill name
description: "<one paragraph>"      # see rules below; quote it, escape inner quotes
version: 1.0.0                      # semver; bump on any content change
metadata:
  openclaw:
    emoji: "🔎"                     # one glyph for the ClawHub card
    homepage: https://github.com/SELAT-AI/selat-skills/tree/main/skills/<name>
    requires:
      anyBins:                      # the agent needs ONE of these on PATH
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
```

**`description` rules** (this is what ClawHub search + the agent's router see):
- Lead with the capability in plain words, then **when-to-use triggers** as
  quoted example phrases (`"who is @X on Twitter"`, `"scout AI-infra startups"`).
- State the money model in one clause: *pays per call in USDC from the user's
  own self-custody Circle wallet, no API keys*.
- End with the **hard cost cap** (`Full run hard-capped at $X`), taken from the
  SELAT skill's full-run `maxAmount`.
- Keep it to one paragraph; it's a JSON string — escape inner `"`.

**`homepage`** always points at the wrapped SELAT skill's directory — that's the
source of truth reviewers follow.

## Required body sections (in this order)

1. **`# <name>` + one-paragraph intro.** What it does; that it wraps a *SELAT
   skill* (declarative, vetted, keyless, USDC); read-only vs. side-effecting.
2. **`## Cost — read this first`.** Per-step and full-run caps (from the
   manifest); "every call is real USDC from the user's own Circle Agent Wallet
   (MPC self-custody — SELAT never holds keys or funds)"; "always dry-run first";
   "never ask for/paste/handle a private key". If the underlying skill is a
   **menu** (independent steps, agent picks a subset), say so here and note that
   `selat skill run` executes *all* steps.
3. **`## Step 0 — get the CLI (free, no account)`.**
   `selat --version || npm install -g @selat-ai/selat-cli`, plus "installing the
   CLI creates nothing money-related."
4. **`## Step 1 — dry run first, before any wallet setup`.** The free
   `selat skill install <name>` + `SELAT_ROUTER_URL=… selat skill verify
   ~/.config/selat/skills/<name>`; it prints real 402-quoted prices with no
   wallet. "Show the user these prices and get their OK before wallet setup."
5. **`## Step 2 — wallet setup (only after the user opts in)`.**
   `selat init` / `selat fund` / `selat doctor`; note `selat init` is safe to
   re-run.
6. **`## Step 3 — run`.** `selat skill run <name> --param …` with a **params
   table** (name · what it steers · which reads/steps it feeds). Then: "each step
   returns raw JSON — distill it into a plain-language answer with the dollar
   cost; keep endpoint URLs and raw JSON out of what you relay." For a menu
   skill, add routing guidance (which params for which intent).
7. **`## Why this is safe to install`.** Declarative manifest / no code; endpoints
   https-only, first-party, pre-vetted, gated on a live verify receipt; spend
   capped per step and per run; funds stay in the user's own wallet.
8. **`## Beyond this skill`.** `selat search "<intent>"` (free discovery) and
   `selat skill list --available`, plus the `selat-skills` docs link.

## Rules & gotchas (the invariants)

- **No endpoint URLs, no code, no secrets.** The only URLs allowed are the
  `github.com` homepage/docs links and `router.selat.ai` in the dry-run env var.
  Don't name provider hosts or paths (`catalog.selat.ai/twitter/...`) — say
  "SELAT's own first-party API." Grep your file:
  `grep -nE "https?://(api|catalog)|def |import |/[a-z_]+/[a-z_]+\?" SKILL.md`
  should return nothing.
- **`name` == folder == the SELAT skill name.** Consistent everywhere; ClawHub
  and `selat skill install` both key on it.
- **Caps come from the SELAT manifest, verbatim.** Don't invent prices. If the
  full-run cap is `$0.10`, the description and Cost section say `$0.10`.
- **Dry-run-first is non-negotiable.** Step 1 (`verify`, free, no wallet) always
  precedes any wallet/fund/run step. This is the trust spine — never reorder it.
- **Menu vs. pipeline.** If the wrapped skill's steps are independent reads the
  agent selects among, the wrapper must say "menu, not pipeline" and route by
  intent. If it's a true pipeline (every step always runs), present it as one
  flow. Match the SELAT skill's `SKILL.md`.
- **Read-only vs. side-effecting.** State it. If the skill can post/spend/mutate,
  the Cost/Workflow sections must add a confirm-before-the-irreversible-step line.
- **Never handle keys.** Wallet auth is the CLI's Circle integration; the wrapper
  never asks for, prints, or accepts a private key.
- **Version discipline.** Bump `version` on any content change; ClawHub treats a
  new version as a new publish.

## Pre-PR checklist

- [ ] Wraps a **merged, verify-passing** SELAT skill; `homepage` points at it.
- [ ] `name` == folder == SELAT skill name; frontmatter is valid YAML.
- [ ] `description` has triggers + the money model + the hard cost cap.
- [ ] All 8 body sections present, in order; dry-run precedes wallet setup.
- [ ] Caps match the SELAT manifest; params table matches the manifest params.
- [ ] **No endpoint URLs, no code, no secrets** (run the grep above).
- [ ] README "Skills" table row added.
- [ ] PR describes which SELAT skill it wraps and the cap.

## Why this structure

The wrapper is the *only* thing a ClawHub user reads before installing, and it's
the only thing that runs at install time. Keeping it text-only means installing
a wrapper can never execute code or leak an endpoint; putting the priced recipe
in `selat-skills` (behind a live-verify gate) means the wrapper can't drift from
what actually gets paid. Dry-run-first means a user always sees real prices,
with no wallet and no commitment, before anything money-shaped exists.
