# PadelPro

> This repository is a showcase: screenshots and an overview. The source code is private. I'm happy to walk through it on request.

A booking marketplace for padel courts, built for the Iranian market: persian-language, right-to-left, Jalali calendar, prices in rial. Players book courts online, join open matches and split the bill, and build a verified ranking. Clubs and coaches get their own panels.

I built it alone over about ten weeks (338 commits, August to October 2026). It is a full product rather than a demo: real double-entry accounting, a rating system, four role-based dashboards and a deploy pipeline.

![Home page](docs/screenshots/home.jpg)

## What it does

- **Court booking.** A club's schedule grid shows every slot as free, held, booked or past. You pick slots, pay, and get a ticket. Two people can never hold the same court at the same time (more on how below).
- **Padel Passport.** A player identity card: level, official games, city, and which coach verified the level. A passport is the key to online booking and is the main thing the platform sells.
- **Open matches.** Book a court, leave the empty spots open. Players at the right level join and each pays only their own share.
- **Ranking.** Elo-based. Ratings come only from confirmed match results, never from self-reports.
- **Coach ID.** Coaches are validated by a club before they can publish sessions. Players book a seat in a private or group session.
- **Four panels.** Player, coach, club and admin, each with its own colour and navigation, built from one shared shell.

| Booking a court | Passport |
|---|---|
| ![Club page with schedule](docs/screenshots/club-booking.jpg) | ![Player passport, opened](docs/screenshots/passport-open.jpg) |

| Ranking | Coaches |
|---|---|
| ![Ranking](docs/screenshots/leaderboard.jpg) | ![Coach directory](docs/screenshots/coaches.jpg) |

| Club panel | Admin panel |
|---|---|
| ![Club panel](docs/screenshots/panel-club.jpg) | ![Admin panel](docs/screenshots/panel-admin.jpg) |

It works on phones too (bottom navigation, one court at a time instead of the wide grid):

<p>
  <img src="docs/screenshots/home-mobile.jpg" width="200" alt="Home on a phone">
  <img src="docs/screenshots/club-booking-mobile.jpg" width="200" alt="Booking on a phone">
  <img src="docs/screenshots/passport-mobile.jpg" width="200" alt="Passport on a phone">
  <img src="docs/screenshots/panel-player-mobile.jpg" width="200" alt="Player panel on a phone">
</p>

## The parts I'm most happy with

**Double booking is prevented by the database.** A Redis lock makes the UI feel instant, but Redis can restart or evict keys, and then two payments land on one court. So the real guarantee is a Postgres `EXCLUDE USING gist` constraint on the bookings table. The readiness endpoint refuses to report "ready" if that constraint is missing. I tested it against eleven SQL scenarios.

**There is no balance column.** Every rial movement is a row in a double-entry ledger, and a balance is always a sum. Each transaction must add up to zero, and a nightly job checks it. It is more work than a column, but it means a wallet number can never silently disagree with what happened.

**Money and rating logic are pure packages.** `packages/pricing` and `packages/rating` do no I/O at all, so they are easy to test hard. Both have an 85% coverage threshold, because a quiet regression in either one is the kind that costs real money or trust.

**Two rating scales.** Elo runs on 800 to 2200 internally, and players see a 0 to 7 level. Running Elo directly on 0 to 7 does not work: a two-level gap would give a 50.3% win chance. A letter grade (D to A) is derived for display only and never feeds back into any formula.

**Persian details done properly.** Everything is stored as UTC `timestamptz` and money as integer rial (`BIGINT`). Jalali dates are a display concern only, and opening hours are resolved in `Asia/Tehran`. Layout uses logical CSS properties, so RTL is one attribute on `<html>`.

**Smaller things:** per-identity rate limiting, a nonce-based CSP and the usual security headers, idempotency keys on payment endpoints, refresh-token rotation with compare-and-set, and a JS budget on the home page that is checked by a script (about 192 KB gzipped of a 196.6 KB limit).

Every non-obvious choice has a written reason in a decision log (20 records).

## Stack

| | |
|---|---|
| Frontend | Next.js 16 (App Router), React 19, Tailwind v4, PrimeReact (unstyled, themed through PassThrough) |
| Backend | NestJS 10, REST and Socket.io, Zod validation shared with the frontend |
| Data | PostgreSQL 16 with Prisma, Redis 7 |
| Payments | Mock gateways for demos plus a real ZarinPal v4 driver |
| Tooling | npm workspaces, Vitest, type-aware ESLint, Docker, nginx |

## Numbers

| | |
|---|---|
| Source | about 80,000 lines of TypeScript, 53 pages |
| API | 17 controllers, roughly 170 route handlers |
| Database | 33 migrations |
| Tests | 478 across 51 files, all passing; lint has no errors |

## Status and what's next

The core product is finished and runs end to end. It is deployed as a demo with mock payments. What is left is outside the code: a real payment-gateway contract, SMS provider and a domain, plus testing on real iOS devices. The English version of the site is partly done (home and role pages), and tournaments and leagues are planned for later. There is also a written business plan covering the market, competitors and unit economics.

## خلاصه به فارسی

پدل‌پرو یک پلتفرم رزرو زمین پدل برای بازار ایران است: رزرو آنلاین، بازی‌های باز با پرداخت تقسیمی، رتبه‌بندی بر پایه‌ی نتیجه‌ی تأییدشده، و پنل جدا برای بازیکن، مربی، باشگاه و ادمین. رابط فارسی و راست‌به‌چپ است، تقویم شمسی نمایش داده می‌شود و مبلغ‌ها به ریال صحیح ذخیره می‌شوند.

## Author

Kasra Jamei · kasrajamei@yahoo.com
