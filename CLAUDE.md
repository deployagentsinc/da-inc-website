# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Stack

Static HTML site for Deploy Agents Inc. (deployagentsinc.com / deployagentsinc.agency). No build step, no package manager, no framework. Each page is a single self-contained `.html` file with CSS embedded in `<head>` and any JS inline at the bottom. Hosted on Vercel — `vercel.json` only sets `cleanUrls: true`, which is why internal links use `/privacy` and `/terms` rather than `.html` extensions.

To preview locally, open the file directly in a browser or run any static server from the repo root (e.g. `python3 -m http.server 8000`). There is nothing to build, lint, or test.

## Pages and what each one is for

- `index.html` (~2.2k lines) — main marketing site. Contains nav, hero, BusinessOS section, HumanOS section, pricing, agent channels, footer. All styles live in one `<style>` block at the top.
- `privacy.html`, `terms.html` — legal pages. Share a stripped-down color palette (`--concrete`, `--cream`, `--gold`, `--gold-light`, `--steel-light`).
- `restaurant.html` — print-format pitch deck (A4, `210mm × 297mm` pages with `page-break-after`). Different palette (steel/wood/fire) and Playfair Display font. Not part of the public nav — it's a standalone artifact, intentionally has no Meta Pixel.

## Things that are easy to break

- **Stripe checkout URLs are hardcoded in two places** for each product: the `<a href>` itself, and the Meta Pixel click handler in `index.html` around lines 1334–1352, which matches on the URL's path segment (e.g. `bJedR8cJv5JMdFR2NGe7m03`). If you swap a Stripe link, update the matcher too or the conversion event stops firing. Current mapping:
  - `bJedR8cJv5JMdFR2NGe7m03` → BusinessOS $20/mo (`InitiateCheckout` value 20.00)
  - `3cI6oG5h31twgS373We7m01` → HumanOS Single Agent $3 (`InitiateCheckout` value 3.00)
  - `4gMfZgfVHegi59l4VOe7m02` → HumanOS Full Stack $25 (`InitiateCheckout` value 25.00)
- **Meta Pixel ID `1622984352364723`** is initialized in `index.html` only. If you add a new public page (not a print deck), copy the Pixel `<script>` blocks over.
- **Internal links assume `cleanUrls`**: `<a href="/privacy">` resolves to `privacy.html` only because of `vercel.json`. Don't rewrite these to include `.html` — and if the site ever moves off Vercel, the routing has to be replicated.
- **Contact channels are duplicated** across the nav, agent-channel grids, and footer in `index.html`. Update everywhere when a number/handle changes:
  - WhatsApp `https://wa.me/66839799707`
  - Telegram bots: `t.me/DeployAgentsIncBot` (sales), `t.me/HumanOS_DA_Inc_bot` (HumanOS signup)
  - Line `https://line.me/R/ti/p/@963ieadj`
  - Email `hello@deployagentsinc.com`, `founder@deployagentsinc.com`

## Brand conventions

- Color tokens are defined per-file in `:root`. The main site uses a concrete/cream/gold palette; restaurant.html uses a steel/wood/fire palette. Don't try to share them across files — pages are intentionally self-contained.
- Fonts loaded from Google Fonts: Bebas Neue (display), Cormorant Garamond (body, main site), DM Mono (labels), Playfair Display (restaurant.html only).
- Product names are styled as proper nouns: **BusinessOS**, **HumanOS**, **MA** (Managing Agent). Keep capitalization exact.

## Commit style

Recent history is terse imperative one-liners, often `Add X`, `Replace X with Y`, `Rebrand to …`. No prefixes, no scopes, no body unless the change needs it.

## Related systems

The site advertises two products served by sibling repos in `~/`:

- `~/humanos/` — the HumanOS Express service (WhatsApp/Telegram/Stripe webhooks, 14 specialist agents, ChoreGenie family flow). Deployed to Railway.
- `~/` (root) — BusinessOS specialists / Hermes gateway (separate runtime).

If you change a Stripe link, contact number, or Telegram bot handle on this site, mirror the change in the corresponding service repo.

**Telegram bot setup gotcha:** the `@HumanOS_DA_Inc_bot` linked from `index.html` and the HumanOS invite flow requires **BotFather privacy mode set to DISABLED** to receive non-command messages in group chats (chore photos, chore keywords, role changes). Without that, ChoreGenie silently never fires. Set via `@BotFather` → `/setprivacy` → pick the bot → **Disable**. Mirrored in `~/humanos/README.md`.
