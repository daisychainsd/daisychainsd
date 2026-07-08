# Start Here — How Daisy Chain's Tech Works

You're reading this because you're going to help work on Daisy Chain's systems.
This page explains what exists and why, in plain English, so nobody has to
explain it to you in a meeting. Read this, then the `OPERATIONS.md` in whichever
repo you'll be working in.

## The business, in one paragraph

**Daisy Chain Recordings** is an independent electronic music label in San
Diego, run by Player Dave (PD). It releases music, throws club shows at Spin
Nightclub (tickets via Shotgun), and sells music + merch from its own website
instead of relying on Bandcamp. The tech's job: sell things, deliver them
automatically, and grow the fan list — with as little manual work as possible.

## The three systems

### 1. The website — [`daisychain-site`](https://github.com/daisychainsd/daisychain-site) → [daisychainsd.com](https://daisychainsd.com)
The storefront and the brand's home. Fans stream previews, buy digital
downloads (or a $99 all-catalog pass), and buy vinyl/CDs/shirts. Under the
hood it's a Next.js app where:
- **Sanity** is the CMS — PD edits releases, artists, and events at `/studio`, no code involved.
- **Stripe** takes every payment.
- **Supabase** stores user accounts and who-bought-what.
- **Shopify** holds the merch catalog and receives fulfillment orders (shipped via Pirate Ship).
- A **webhook from Stripe** is the heart of it: when a payment lands, it records the purchase, emails the download link, creates the merch order, and adds the buyer to the newsletter — all automatically. If that webhook breaks, someone paid and got nothing, so it alerts PD by email immediately.
- An hourly **release-day cron** flips scheduled releases live at the right moment — nobody has to be awake at midnight on release day.

### 2. The subscriber sync — [`dc-email-api`](https://github.com/daisychainsd/dc-email-api) → dc-email-api.vercel.app
Fans give their email in three places that have nothing to do with each other:
buying on **Bandcamp**, buying show tickets on **Shotgun**, and signing up for
drop alerts on **Laylo**. This small service pulls all of those into one place —
the **Beehiiv** newsletter list — so every fan can be reached from a single
list. Bandcamp and Shotgun are checked once a day; Laylo pushes in real time.
It keeps its place with cursors in Redis, so it never re-processes old data,
and duplicates are harmless. If a sync fails or a token expires, it emails PD.

### 3. The newsletter — [`chain-mail`](https://github.com/daisychainsd/chain-mail)
"Chain Mail" is the label's email newsletter, sent from **Beehiiv**. This repo
is the design workshop: custom HTML sections are built here and pasted into
Beehiiv as snippets. No servers, nothing deployed — just design files.

## Glossary — the services you'll see named

| Service | Plain-English role |
|---|---|
| **Vercel** | Hosts the website and the sync service; also runs the scheduled jobs (crons) |
| **Stripe** | Takes all the money |
| **Sanity** | The CMS — where release/artist/event content lives |
| **Supabase** | Database + user accounts (who bought what) |
| **Shopify** | Merch catalog + fulfillment orders (physical goods only) |
| **Beehiiv** | The newsletter list and sender |
| **Laylo** | SMS/drop-notification list (separate audience from the newsletter, on purpose) |
| **Shotgun** | Ticketing for the club shows |
| **Bandcamp** | Legacy sales channel — still a source of fan emails |
| **Resend** | Sends transactional email (download links, order confirmations, failure alerts) |
| **Upstash Redis** | Tiny database the sync service uses to remember where it left off |
| **Pirate Ship** | Where PD buys shipping labels for merch orders |

## How we know everything is working

- **Ops dashboard** — daisychainsd.com/ops (password from PD). Live green/red
  health for every system, recent orders + 30-day revenue, newsletter stats,
  and what's coming up. Check it before assuming anything is broken.
- **Alert emails** — the systems email PD when something actually fails
  (a purchase that didn't record, a sync that crashed, an expired token, a
  daily all-systems health check). Silence means healthy — nobody polls logs.
- **Per-repo `OPERATIONS.md`** — each repo documents its own failure modes and
  exactly how to recover. That's the runbook; keep it updated when you change
  behavior.

## Working on this stuff

1. **Access you need from PD**: GitHub org invite, the ops dashboard password,
   and env secrets (they live in Vercel, never in git — `vercel env pull` once
   you're added to the Vercel team, or PD hands you a `.env.local`).
2. **Workflow**: work on `dev` (or a feature branch → PR into `dev`). Pushing
   `dev` on the website auto-deploys a preview at **dev.daisychainsd.com** —
   review there. `main` = production and is protected; changes land by PR.
3. **House rules**:
   - Never merge site changes to `main` without walking the purchase flow
     end-to-end on the preview. Real buyers, real money.
   - The website's look is governed by the `design-system/` folder in the site
     repo — it's law, don't freestyle the brand.
   - `npm run build` locally before pushing; keep the preview green.
4. **Read next**: the `OPERATIONS.md` of your repo, then the site's `CLAUDE.md`
   (deep technical reference) if you're on the website.

Questions after reading all that → PD (playerdave@daisychainsd.com).
