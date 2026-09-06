---
---

# Apartment Hunt Monitor — a Personal Automation Agent

*Fun project (2026): a scheduled agent that watches a local listings platform and pushes only-new results to WhatsApp every morning.*

> **Personal project, built for my own apartment search.** All search criteria below are **configurable** — you would set your own location, budget, unit type, and swap preference. Nothing here reflects any single person's actual criteria; the point is the pattern, not the preferences.

## What it does

Every morning at 08:00 the agent:
1. Scans a classifieds platform (Kleinanzeigen) across **configurable searches** — e.g. a city within a budget, the same city with a **swap/trade filter** (some landlords only accept exchanges), and a nearby commuter town
2. Parses each listing: title, price, location, URL
3. Compares against a stored baseline and **reports only NEW listings** — never repeats what you already saw
4. Filters **noise** (e.g. short-term rentals, festival-season offers, tradesman accommodations) by keyword
5. Delivers a short digest to **WhatsApp** on your phone

First run "prewarms" the baseline (swallows the whole pool silently, so day one isn't a wall of old ads); every run after that is strictly incremental.

## How it works

| Layer | What it does |
|-------|--------------|
| **Scheduler** | Cron job, daily at 08:00 (adjustable) |
| **Scraper** | HTTP fetch of the platform's search pages with pagination; user-agent header; captcha/anti-bot detection with exponential backoff retries |
| **Parser** | Regex extraction of ad ID, title, description, price, location from the listing HTML |
| **State** | JSON baseline file (`seen_ids`) — the memory that makes alerts incremental and idempotent |
| **Filter** | Noise keyword list + swap-detection tagging (lists with a swap flag get a warning to check the direction of the trade) |
| **Delivery** | stdout → personal WhatsApp gateway; on total failure it stays silent and raises an error alert instead of sending a misleading "no listings" message |

## Configurable criteria (set your own)

| Parameter | Options |
|-----------|---------|
| Location(s) | Any city/region the platform covers; multiple searches supported |
| Max budget | Your own ceiling |
| Unit type | Whole-unit rentals / shared flats / both |
| Swap (Tausch) accepted | Yes / No — adds a dedicated swap-filtered search |
| Noise keywords | Short-term lets, seasonal offers, etc. (editable list) |
| Delivery time & channel | Any cron schedule; WhatsApp/Telegram/email |

## Why this pattern matters (transferable skills)

The agent is the same shape as any **monitor-and-alert automation**: *scheduled fetch → normalize → diff against state → filter → deliver, silently when nothing qualifies.* That pattern applies to price watching, compliance monitoring, and content alerts — the housing case is just the most relatable instance. Building it required:

- **Robustness thinking**: anti-bot handling, retry/backoff, never overwriting last-known-good state with an error page, silent-when-nothing-new discipline
- **State design**: the baseline file makes runs idempotent — replaying the same listings never double-alerts
- **Noise engineering**: real-world data is messy; keyword filtering cut the signal-to-noise ratio sharply
- **End-to-end delivery**: from a cron trigger to a phone notification, including failure semantics

## Limitations (honest)

- Scraper depends on the platform's HTML structure — a redesign breaks the parser (acceptable for a personal tool, not a product)
- Single platform, no cross-listing dedup
- No image/floorplan analysis — text fields only
- Filtering is keyword-based, so genuinely relevant listings occasionally get filtered (tuned to err on the side of false negatives over spam)

*Stack: Python, cron, JSON state, HTTP + regex parsing, WhatsApp gateway. Built on Hermes (agent platform).*
