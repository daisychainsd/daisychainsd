# Daisy Chain Recordings — System Map

Independent electronic music label, San Diego. This is the map of everything
that runs the operation. Each repo has an `OPERATIONS.md` with the full detail:
what it does, env vars, failure modes, and how to verify it's healthy.

**New here? Read [ONBOARDING.md](ONBOARDING.md) first** —
the whole operation explained in plain English, no meeting required.

## Systems

| Repo | What it runs | Deployed at |
|---|---|---|
| [daisychain-site](https://github.com/daisychainsd/daisychain-site) | Public site + store: releases, streaming, digital downloads, merch (Shopify), unlimited pass, Sanity CMS | [daisychainsd.com](https://daisychainsd.com) (Vercel) |
| [dc-email-api](https://github.com/daisychainsd/dc-email-api) | Subscriber sync: Bandcamp + Shotgun + Laylo → Beehiiv newsletter list | dc-email-api.vercel.app (Vercel) |
| [chain-mail](https://github.com/daisychainsd/chain-mail) | Newsletter HTML design system — sections built here, sent from Beehiiv | local / Beehiiv |
| [daily-checkin](https://github.com/daisychainsd/daily-checkin) | PD's morning digest + evening reply parser | — |
| [archangel-site](https://github.com/daisychainsd/archangel-site) | Archangel creative agency site (separate brand) | Vercel |

## How money and fans flow

```
Fan buys music/merch ──→ daisychain-site (Stripe) ──→ Supabase + Shopify draft order
                                   │                        │
                                   └── auto-subscribe ──→ Beehiiv ←── dc-email-api
                                                            ▲    (Bandcamp/Shotgun/Laylo syncs)
                                        chain-mail designs ─┘    (newsletter sends)
```

## Health

- Failures alert playerdave@daisychainsd.com by email (Stripe fulfillment
  alerts from the site; cron/token/silence alerts from dc-email-api via Resend).
- Ops dashboard: daisychainsd.com/ops (password-protected) — system status,
  recent orders, upcoming events.

## Contributing

Work happens on branches / `dev`, merged to `main` by PR (`main` is protected).
See each repo's `OPERATIONS.md` for local setup. Secrets live in Vercel env —
ask PD for access.
