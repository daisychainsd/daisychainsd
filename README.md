# Daisy Chain Recordings — System Map

Independent electronic music label, San Diego. This is the map of everything
that runs the operation. The site/service `OPERATIONS.md` files and the Ops repository's SOPs document
what each system does, failure modes, and how to verify it is healthy.

**New here? Read [ONBOARDING.md](ONBOARDING.md) first** —
the whole operation explained in plain English, no meeting required.

## Systems

| Repo | What it runs | Deployed at |
|---|---|---|
| [daisychain-ops](https://github.com/daisychainsd/daisychain-ops) | How the label runs: Operations Map, SOP drafts, consultant docs, and the label's Claude Code skills (`chain-mail`, `offer`, `gdrive`) | private repo; live SOPs at [daisychainsd.com/ops](https://daisychainsd.com/ops) |
| [daisychain-site](https://github.com/daisychainsd/daisychain-site) | Public site + store: releases, streaming, digital downloads, physical orders/shipping (Supabase Ops + Pirate Ship), Shopify catalog, unlimited pass, Sanity CMS | [daisychainsd.com](https://daisychainsd.com) (Vercel) |
| [dc-email-api](https://github.com/daisychainsd/dc-email-api) | Subscriber sync: Bandcamp + Shotgun + Laylo → Beehiiv; authenticated physical Bandcamp order feed for Ops | dc-email-api.vercel.app (Vercel) |
| [chain-mail](https://github.com/daisychainsd/chain-mail) | Newsletter HTML design system — sections built here, sent from Beehiiv | local / Beehiiv |
| [daily-checkin](https://github.com/daisychainsd/daily-checkin) | PD's morning digest + evening reply parser | — |
| [archangel-site](https://github.com/daisychainsd/archangel-site) | Archangel creative agency site (separate brand) | Vercel |

## How money and fans flow

```text
Website music purchase → Stripe → Supabase digital entitlement → download
Website merch purchase → Stripe → Supabase Merch Ops → Pirate Ship labels
                                      │                    │
                                      └── manual shipped/unshipped status
Bandcamp physical purchase → dc-email-api merch feed → Supabase Merch Ops
Website buyers → Beehiiv ← dc-email-api (Bandcamp / Shotgun / Laylo)
                    ↑
           chain-mail designs
```

## Physical website and Bandcamp orders

Merch Ops opens on Unshipped; Shipped and All orders provide history. Use [Merch Ops](https://www.daisychainsd.com/ops/merch), not the short recent-payments panel or Shopify drafts, to find orders and record shipping. Four paid website orders were restored on September 22, with shipping history left unverified for manual Pirate Ship reconciliation. Tracking is optional; downloading a shipping CSV is not proof of shipment.

The [site recovery record](https://github.com/daisychainsd/daisychain-site/blob/main/ORDER-RECOVERY-2026-09-22.md) tracks the exact deployment status of the direct order webhook and hourly Stripe reconciliation. The [fulfillment SOP](https://github.com/daisychainsd/daisychain-ops/blob/main/SOP-merch-fulfillment.md) is the team workflow.

Shopify still supplies products. Its replacement, product/image import and opening inventory count remain unfinished; `MERCH_BACKEND` is unset. The order-recovery deployment does not activate that migration. Eight physical Bandcamp orders were imported and replay-verified September 22, retaining their recorded shipped dates. Physical Bandcamp merchandise enters Ops through a separate hourly feed from dc-email-api; digital music sales are excluded. Initial imports retain Bandcamp shipping status; later syncs preserve manual Ops status/notes/tracking. See [Bandcamp activation](https://github.com/daisychainsd/daisychain-site/blob/main/BANDCAMP-ORDERS-2026-09-22.md). Independent Shopify-native and booth orders remain separate.

## Health

- Failures alert playerdave@daisychainsd.com by email (Stripe fulfillment
  alerts from the site; cron/token/silence alerts from dc-email-api via Resend).
- Ops dashboard: daisychainsd.com/ops (password-protected) — system status,
  recent orders, upcoming events.

## Contributing

Work happens on branches / `dev`, merged to `main` by PR (`main` is protected).
See each repo's `OPERATIONS.md` for local setup. Secrets live in Vercel env —
ask PD for access.
