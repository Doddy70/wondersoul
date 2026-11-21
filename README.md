# WONDERSOUL JAKARTA

**Storefront project.** A Jakarta clothing label — Mods, Ivy League, Madchester and soul — moving from Instagram + WhatsApp to a real web store built on [InnoShop](https://github.com/innocommerce/innoshop) (Laravel 13).

---

## The brand in one paragraph

WONDERSOUL JAKARTA is a local clothing label inspired by British Mods, American Ivy League, 90s Madchester, and the soul scene. The name traces back to *WonderSoul Weekender*, a Jakarta soul/funk and creative collective with roots in 2017 — the label is that community's clothing arm. Today the brand runs on Instagram (`@wondersoul_jkt`, 8,347 followers) and Facebook (`WONDERSOUL`, 2,076 followers), takes Indonesia orders on WhatsApp (**+62 812-9865-108**) and worldwide orders by email (**wondersoul.jakarta@gmail.com**), and has a physical address in Kemang, Jakarta.

**This repository is the plan and the platform for turning that into a proper storefront.**

---

## Repository layout

```
.
├── docs/
│   ├── 01-product-research.md      Brand, market, competitors, gaps
│   ├── 02-evaluation-innoshop.md   Source-code audit against the four goals
│   └── 03-development-goals.md     Goals, architecture, roadmap, open decisions
└── design-reference/               Original static template (MIT) — design language reference
```

The Laravel application lives in `laravel_innoshop/` (working copy of InnoShop CE v1.0.1) and is tracked separately; the docs here are the source of truth for **what** gets built and **why**.

---

## The four development goals

| # | Goal | Status |
|---|---|---|
| 1 | Ecommerce on the InnoShop baseline, **staged on localhost → deployed to a production server** | ✅ Target decided — Nginx + PHP-FPM + MySQL + Redis + Supervisor |
| 2 | **Meta Pixel (CAPI)** tracking, browser↔server dedup | ❌ Not present in InnoShop — build as a plugin |
| 3 | **Midtrans** payment gateway (+ MCP for intelligent automation) | ❌ Not present in InnoShop — build as a plugin |
| 4 | Plugin mechanisms and theme template development | ✅ Core already ships these |

**Goal 1 was re-scoped on 2026-11-21.** The original brief was "build to Vercel"; the evaluation showed Vercel is a poor fit for InnoShop (no official PHP runtime, and the documented Horizon/Reverb/Supervisor topology cannot be hosted there). The target is now a conventional production server, which lets InnoShop run its own documented topology unmodified, keeps media on local disk, and drops the monthly cost from ≈ USD 60–150 to ≈ USD 12–24. See [`docs/03-development-goals.md §5`](docs/03-development-goals.md).

---

## Current status

📋 **Planning.** No production code written yet. The baseline application has been evaluated, the brand researched, and a six-phase roadmap written.

| Document | What's in it |
|---|---|
| [`docs/01-product-research.md`](docs/01-product-research.md) | Verified brand metrics, positioning, market context, the research gaps |
| [`docs/02-evaluation-innoshop.md`](docs/02-evaluation-innoshop.md) | File-path-level audit of the InnoShop baseline against all four goals |
| [`docs/03-development-goals.md`](docs/03-development-goals.md) | Goals, non-goals, anti-goals, three architecture approaches, phased roadmap, pixel event map, risk register, 9 open decisions |

### Blocking questions for the owner

1. **The 88 Instagram posts** — an export (or the bio + recent captions) gives the real product taxonomy, price band, colourway names and size matrix. Everything about the catalogue is currently inferred.
2. **Do you sell COD today?** InnoShop core has **no** cash-on-delivery payment method; if the answer is yes it becomes a plugin in v1.
3. **Is the WonderSoul Weekender music collective part of the business?** Decides whether releases and events are first-class on the store.

See [`docs/03-development-goals.md §11`](docs/03-development-goals.md) for all nine, each with a defensible default.

---

## Stack

| Layer | Choice |
|---|---|
| Framework | InnoShop CE v1.0.1 — Laravel 13, PHP 8.3 |
| Frontend | Vite 8 + Bootstrap 5, theme layer `themes/wondersoul` |
| Payments | Midtrans Snap (ID) · Stripe / PayPal (global) · COD (optional) |
| Tracking | Meta Pixel (browser) + Conversions API (server, deduped) |
| Automation | InnoShop MCP server (`/mcp`) + `laravel/ai` tools |
| Deployment | Local staging (PHP 8.3 + MySQL 8) → **production VPS**: Ubuntu 24.04 · Nginx · PHP-FPM 8.3 · MySQL 8 · Redis · Supervisor (Horizon + Reverb + scheduler) · Let's Encrypt TLS |

---

## Licence

Project code and documentation: see [`LICENSE`](LICENSE).
[`design-reference/`](design-reference/) contains a third-party MIT-licensed static template, unmodified, with its own licence and credits retained — see [`design-reference/LICENSE`](design-reference/LICENSE) and [`design-reference/CREDITS.md`](design-reference/CREDITS.md).
