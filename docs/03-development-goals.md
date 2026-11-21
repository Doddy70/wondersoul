# Development Goals — WONDERSOUL JAKARTA Storefront

**Status:** Draft v0.1 — *pending owner decisions in §11* · **Date:** 2026-11-21
**Companion docs:** [`01-product-research.md`](01-product-research.md) · [`02-evaluation-innoshop.md`](02-evaluation-innoshop.md)

---

## 1. Mission

Turn an existing Jakarta clothing label with a live community — 8,347 Instagram followers, 2,076 Facebook followers, a physical store in Kemang, and a soul-music-collective origin — into a storefront that:

1. **Looks like the brand it already is**, not a startup template.
2. **Converts Instagram attention into owned audience** (email + Meta custom audiences).
3. **Serves two different buyers with two different checkouts** — Jakarta local (COD, WhatsApp) and worldwide (card, email).
4. **Runs on infrastructure that matches the team's actual size** — likely one or two people, not a platform team.

### 1.1 Why this is a migration, not a launch

This is the framing that changes every decision downstream. The brand already has customers, a physical presence and seven years of creative history. **The web store's job is to give an existing business a better shopfront — not to manufacture a new one.** That means:

- The site must earn trust on first paint (a visitor arriving from an Instagram bio link is judging authenticity, not features).
- Nothing in the build may *replace* the WhatsApp relationship; it must complement it.
- Existing Instagram content (88 posts) is the primary launch asset, not a placeholder.

---

## 2. Goals

| # | Goal | Measure of success |
|---|---|---|
| **G1** | A production-quality InnoShop storefront running the Wondersoul brand | `wondersoul.jkt` serving real traffic with no console/backend errors |
| **G2** | Deployable to **Vercel** via the official Container runtime | `vercel deploy` → live URL; one-command rollback; no SSH, no server admin |
| **G3** | **Midtrans** (Snap) as the primary Indonesian payment rail | ≥ 95% of online Indonesian orders paid without manual reconciliation |
| **G4** | **Meta Pixel + CAPI** with correct browser↔server event dedup | Events Manager shows matched events; Purchase ROAS attributable; custom audiences (ViewContent→AddToCart, 30d) populating |
| **G5** | **MCP** enabled, with brand-specific tools for AI-assisted merchandising | `claude mcp add` works; ≥ 3 Wondersoul-specific tools registered |
| **G6** | **Theme template** `themes/wondersoul` built from scratch and owned by us | Theme builds with `THEME=wondersoul npm run build`; no upstream theme fork required |
| **G7** | Bilingual (ID primary, EN secondary) with multi-currency | A Jakarta buyer sees IDR + Bahasa; a worldwide buyer sees their currency + English |
| **G8** | COD available if the business sells it today | COD appears at checkout and order state transitions correctly on delivery confirmation |

### 2.1 Non-goals (explicitly out of scope for v1)

- ❌ Multi-vendor / marketplace mode (one brand, one operator)
- ❌ Native mobile apps
- ❌ Live chat / websockets (Reverb) — Vercel cannot host it, and the value is low for this traffic profile
- ❌ Owning payments (Midtrans/Stripe remain third-party rails)
- ❌ ERP/WMS integration
- ❌ Membership/subscription program — a plausible v2, not v1
- ❌ Event ticketing — pending §11 Q5

### 2.2 Anti-goals (what would make this a failure even if it "works")

- 🔴 **A generic e-commerce skin.** If a visitor cannot tell within 3 seconds that this is a Jakarta soul/Mods label, the project failed regardless of uptime.
- 🔴 **A store that is harder to run than WhatsApp.** Every operational task (price change, stock update, order status) must be doable by one person in the panel without a developer.
- 🔴 **Data loss or unreachable orders.** Payment webhooks must be idempotent and orders must never be lost to a failed background job.
- 🔴 **Silently losing attribution.** Purchase events that fail to reach Meta's Events Manager without anyone noticing.

---

## 3. Audiences & journeys

| Audience | How they arrive | Needs | Primary payment |
|---|---|---|---|
| **Jakarta local** | Instagram bio, IG story, bazaar QR, Google | Bahasa Indonesia, IDR, COD, WhatsApp confirmation, local delivery | COD / Midtrans VA / QRIS |
| **Worldwide** | Instagram (followers travel), link-in-bio, Google | English, local currency, card, shipping calculator, tracked delivery | Stripe / PayPal / Midtrans card |

**Journey A — local, impulse, from IG.**
`IG post → site landing (single garment, not a catalogue) → size + colour → add to cart → checkout → COD or Midtrans QRIS → WhatsApp order confirmation → delivery → order marked complete → Pixel Purchase fires server-side`

**Journey B — worldwide, considered.**
`Google / IG → home → collection (Mods / Ivy / Madchester / Soul) → garment detail (size chart, fabric, fit note) → cart → address + shipping method → card → Midtrans/Stripe → shipment → order complete`

**Journey C — the AI operator.**
`Owner: "which colourway of the Harrington is running out?" → Claude Code → MCP tool `stock_by_colorway` → JSON answer → owner acts in panel`

---

## 4. Product model

Derived from the brand positioning; **to be validated against the 88 Instagram posts** (§11 Q1).

### 4.1 Collection taxonomy

Four collections, one per brand reference — this is the site's information architecture:

| Collection | Reference | Product families (draft) |
|---|---|---|
| **Mods** | British 60s | Harrington jacket, tailored blazer, Chelsea boots, Oxford shirt, anorak |
| **Ivy** | American East Coast prep | Cable knit, tweed blazer, oxford shirt, chinos, duffle coat |
| **Madchester** | 88–92 | Oversized graphic tee, track jacket, mesh layer, baggy trouser, 90s-print shirt |
| **Soul** | 60s Motown → 90s neo-soul | Shearling-collar jacket, velour, satin shirt, earth-tone knitwear |

### 4.2 Product attributes required

- **Colourways** — named, not numeric. The design reference already establishes a colour-field motif; colour names are brand asset, not metadata.
- **Sizes** — S–XXL, unisex, with a **size chart per garment** (Mods/90s cuts fit differently; a generic chart will generate returns).
- **Fit note** — one line per garment ("boxy, take your usual size" / "slim through the body, size up").
- **Release** — a *drop* concept: a named release (e.g. "Release 03 — Madchester") with a publish date. This maps the catalogue onto the community's event-driven rhythm and makes the MCP/AI tools meaningful.

### 4.3 Catalogue seeding

`themes/wondersoul/demo/Seeder.php` follows the InnoShop closure pattern (`return function (string $dir): void { … }`), populating demo collections, products, colourways and settings for local dev — **not** the live catalogue. The live catalogue is loaded by the owner via the panel or a CSV importer.

---

## 5. Architecture — approaches

> The single biggest open decision is **what Vercel is allowed to be**. Three coherent options, each with a real trade-off. **G2's wording ("build ke vercel") rules out a hybrid in spirit — but the cost difference is worth stating plainly.**

### Approach A — "Vercel-native" ✅ **recommended**

Everything runs on Vercel: app as a **Container** (FrankenPHP + Caddy), database on **managed MySQL**, media on **S3/R2**, queue on the **database driver** drained by **Vercel Cron**, no websockets.

| Pros | Cons |
|---|---|
| One platform; zero server admin; scales to zero | Managed MySQL + S3 + Vercel Pro = **real recurring cost** |
| Matches the stated goal literally | No websockets (accepted in §2.1) |
| Rollback / preview deployments for free | Cold starts; container build is slower than a static build |
| | Cron-based queue drain has **minute-level latency** — order emails may lag |

**Cost shape:** Vercel Pro + PlanetScale/Aiven MySQL + S3 + domain ≈ **USD 60–150/month**.

### Approach B — "Vercel front, classic server back"

Vercel hosts only the CDN/edge/static assets and a thin rewrite; the Laravel app runs on a normal VPS (Hetzner/DigitalOcean) with Horizon + Reverb + Supervisor exactly as InnoShop documents.

| Pros | Cons |
|---|---|
| Runs InnoShop's **documented** production topology unmodified | **Two systems**; Vercel is no longer the app host |
| Cheapest overall (≈ USD 25–40/month all-in) | Vercel adds latency + a rewrite layer |
| Websockets, Horizon, real cron all available | The stated goal "build ke Vercel" is only half met |

### Approach C — "Vercel with a worker sidecar"

Approach A, plus a tiny always-on worker (Fly.io / Railway, ~USD 5–10/mo) running `queue:work` and the scheduler, so jobs are not cron-latent.

| Pros | Cons |
|---|---|
| Best of both: Vercel-hosted app + real queue | Two platforms again (but the sidecar is trivial) |
| Order emails go out in seconds | Slightly more to operate |

### Decision

**Adopt Approach A now; keep Approach C as a one-line upgrade path.** Rationale: the team is one or two people, and Vercel Pro + managed services removes all server administration — the single largest source of operational risk for a small team. Queue latency is acceptable because the only queued work is order-confirmation mail and image processing, and the **authoritative order state is written synchronously during checkout** (never dependent on a worker completing). If mail latency becomes visible to customers, adding a USD 5–10 sidecar converts Approach A into C with no code change.

**Database:** **managed MySQL** (Approach A assumes this). Postgres is cheaper on Vercel, but InnoShop's schema is MySQL-specific — a port is a multi-week project that dwarfs the entire rest of the roadmap. Recommend MySQL now; revisit only if MySQL hosting becomes the dominant cost.

---

## 6. Phased roadmap

Each phase has an explicit exit criterion. Estimates assume **one full-time developer** plus owner availability for content.

### Phase 0 — Foundation *(~1 week)*

| Task | Detail |
|---|---|
| Toolchain | Install **PHP 8.3+** (required extensions: bcmath, curl, dom, fileinfo, libxml, openssl, pdo, simplexml) + Composer 2. ⚠️ Neither is currently on PATH. |
| Baseline repo | Initialise `laravel_innoshop` as a git repo; tag `upstream/v1.0.1`; commit the InnoShop baseline unmodified |
| Brand assets | Logo, wordmark, colour palette, type stack, favicon, OG image, product photography |
| Copy | Indonesian + English: home, collection, about, size chart, shipping, returns, contact, privacy, terms |
| Legal | Business entity, NPWP/registration, consumer-protection terms — **required for Midtrans production activation** |
| Config | `APP_NAME=WONDERSOUL JAKARTA`, `APP_URL`, `APP_LOCALE=id`, `APP_FALLBACK_LOCALE=en`, IDR + IDR/USD currency, `system_setting('theme') = 'wondersoul'` |

**Exit:** local `php artisan serve` renders the InnoShop front end in Indonesian with Wondersoul branding; `git log` shows the baseline commit.

---

### Phase 1 — Commerce core *(~2 weeks)*

| Task | Detail |
|---|---|
| **Theme `wondersoul`** | Author from scratch: `config.json`, `views/`, `assets/scss/`, `assets/js/`, `boot.php`, `demo/Seeder.php`. Use the SCSS override strategy (import `@front/css/app`, then override) to inherit catalogue/cart/checkout behaviour, and own the home/collection/product pages |
| Design language | Carry over the working device from the design reference: the **notch** motif, the **full-bleed colour fields**, the **stacked-scroll rhythm**, single typeface, one accent colour — re-interpreted for garments, not vehicles |
| Products | Collections (Mods / Ivy / Madchester / Soul), products, colourways, size charts, fit notes |
| Releases | A `Release` concept (name, date, collection) surfaced on the home page and in the MCP tools |
| Admin training | Owner walks the panel: product, stock, order, settings, plugin enable/disable |

**Exit:** a complete catalogue renders correctly at 360 px / 768 px / 1440 px in Indonesian; a test order can be placed and the owner can fulfil it from the panel.

---

### Phase 2 — Payments *(~1 week)*

| Task | Detail |
|---|---|
| **`plugins/Midtrans`** | `config.json` (`type: payment`, `code: midtrans`), `fields.php` (server_key, client_key, production flag, merchant id), `Boot.php` (`service.payment.pay.midtrans.data`), `Services/MidtransService.php` (Snap token creation), `Views/payment.blade.php` (Snap embed), `Controllers/MidtransController.php` (webhook + status), `Routes/front.php`, `Lang/{en,id}/common.php`, `Tests/Feature/` |
| Webhook correctness | Verify Midtrans notification **signature** before trusting it; make the handler **idempotent** (Midtrans retries); mark order paid only on `capture`/`settlement`; reject `expire`/`deny`/`cancel` appropriately |
| Status reconciliation | A scheduled `GET /<order>/status` sweep for orders stuck in `pending` (Snap redirect lost) — prevents the classic "paid but unpaid in the store" bug |
| Sandbox test | Full round-trip in Midtrans sandbox across VA, QRIS, and e-wallet, with the webhook hitting the real production URL shape |
| **`plugins/Cod`** *(if §11 Q2 = yes)* | A first-party COD method with a configurable fee and a "mark as delivered → paid" flow |
| Card rail | Enable the bundled `Stripe` plugin for the worldwide audience (already in core) |

**Exit:** a sandbox Midtrans order transitions `unpaid → paid` **via webhook only**; killing the browser redirect still results in a paid order; the same order is never marked paid twice.

---

### Phase 3 — Tracking & attribution *(~1 week)*

| Task | Detail |
|---|---|
| **`plugins/MetaPixel`** | `config.json` (`type: marketing`), `fields.php` (pixel_id, access_token, test_event_code, enabled, advanced_matching toggle), `Boot.php` (blade insert + event hooks), `Services/MetaPixelService.php`, `Services/CapiClient.php` |
| Base code | Inject via `listen_blade_insert('front.layout.app.head.bottom', …)` → `innopacks/front/resources/views/layouts/app.blade.php:51` |
| Event map | See §7 |
| **Dedup** | Browser `fbq('track', …)` must pass `eventID` equal to the server event's `event_id`. For Purchase, `event_id = order number` |
| Advanced Matching | SHA-256 `em`, `ph`, `fn`, `ct`, `st`, `zp` — normalised (lowercase, trimmed) exactly as Meta specifies |
| Reliability | Server events must not break checkout: failures are caught, logged, and retried by the queue; a visible "Pixel health" panel widget shows last success / error count |
| Events Manager validation | Test events during development; confirm browser+server pairs show as deduplicated |

**Exit:** Events Manager shows matched ViewContent / AddToCart / InitiateCheckout / Purchase events with correct value and currency; a test order's Purchase appears **once**, not twice.

---

### Phase 4 — Deployment *(~1 week)*

| Task | Detail |
|---|---|
| **Dockerfile** | Multi-stage: `composer:2` deps → `php:8.3-alpine` runtime with all required extensions + `pdo_mysql`, `gd`/`imagick`, `exif`, `intl`, `zip`; Caddy + FrankenPHP per Vercel's official guide |
| **Caddyfile** | `root * /app/public`, `php_server`, `PORT` from env, zstd/gzip, security headers |
| **`vercel.json`** | Container preset; `functions` → the container; `crons` → queue drain + scheduler + reconciliation; `regions` → `sin1` (Singapore — closest to Jakarta) |
| **Install bypass** | Browser wizard cannot run on Vercel (it writes `.env` and `storage/installed`). Ship a **CLI install command** (`php artisan wondersoul:install`) that runs migrations + seeds against the managed DB, and create `storage/installed` **in the image** |
| **`.env` gate** | `public/index.php` requires a real `.env` file containing `APP_KEY=`. Ship a placeholder `.env` in the image and let Vercel env vars override it — or patch the gate to read `getenv()` |
| **Media** | Define the `media` disk → S3/R2; verify where it is currently configured; add `AWS_*` / custom endpoint env |
| **Logging** | `LOG_CHANNEL=stderr` (FrankenPHP) so logs reach Vercel's log drain |
| **Queue** | `QUEUE_CONNECTION=database`; `vercel.json` cron → `/api/internal/queue-work` (signed) |
| **Migrations** | Run as a **release step**, not at boot — never `migrate` from multiple concurrent cold starts |
| Domain + DNS | `wondersoul.jkt` (or `.co.id`) → Vercel; A record / nameservers |

**Exit:** `vercel deploy` → production URL serves the storefront; a full order completes in production; the DB is migrated; media uploads land in S3; the app does not redirect to `/install`.

---

### Phase 5 — MCP & automation *(~3 days)*

| Task | Detail |
|---|---|
| Enable MCP | `system_setting('mcp_enabled', true)`; create a Sanctum admin token; verify `GET /mcp` welcome page |
| Client wiring | `claude mcp add --transport http wondersoul https://<host>/mcp --header "Authorization: Bearer <token>"` |
| **Brand tools** | Register via `add_hook_filter('ai.tools', …)` — e.g. `drop_release_list`, `stock_by_colorway`, `low_stock_garments`, `best_sellers_by_release`, `out_of_stock_by_size` |
| Panel AI chat | Confirm the same tools work in the admin panel's built-in AI assistant |
| Guardrails | Every tool returns a `requiredPermission()`; tools are read-only by default; write tools prefixed `⚠️ WRITE:` per InnoShop convention |

**Exit:** an AI assistant can answer "which sizes are running out for the Madchester graphic tee?" against live data, and a write tool requires an admin permission to execute.

---

### Phase 6 — Launch hardening *(~1 week)*

| Task | Detail |
|---|---|
| SEO | Product/category schema, `sitemap.xml` (spatie/laravel-sitemap already in core), OG/Twitter cards, `hreflang` for `id`/`en` |
| Performance | Image CDN + WebP/AVIF, lazy loading, Core Web Vitals budget |
| Legal | ToS, privacy, return/exchange policy, NPWP/business data in the footer |
| Accessibility | Contrast, focus states, keyboard nav on the custom theme |
| Observability | Uptime check, Pixel-health widget, webhook-failure alert (email), daily order-count sanity check |
| Backup | Managed MySQL automated backups + a verified restore run |
| Rollback | Documented rollback procedure (previous Vercel deployment + previous DB snapshot) |

**Exit:** a written launch checklist passes; a colleague can take an order end-to-end using only the panel + the phone.

---

## 7. Meta Pixel event map

| Event | Trigger (hook) | Browser | Server (CAPI) | Params |
|---|---|---|---|---|
| `PageView` | Every front page | ✅ | ❌ (page-level, browser only) | — |
| `ViewContent` | Product detail view | ✅ | ✅ | `content_ids`, `content_type=product`, `content_name`, `value`, `currency` |
| `AddToCart` | `service.cart.add.after` | ✅ | ✅ | `content_ids`, `value`, `currency`, `num_items` |
| `InitiateCheckout` | `service.checkout.checkout.result` | ✅ | ✅ | `content_ids`, `value`, `currency`, `num_items` |
| `AddPaymentInfo` | `service.checkout.billing.methods` | ✅ | ✅ | `value`, `currency` |
| `Purchase` | `service.checkout.confirm.after` | ✅ | ✅ | `order_id`, `value`, `currency`, `content_ids`, `num_items` |
| `Lead` | Newsletter signup / waitlist | ✅ | ✅ | — |
| `Search` | Catalogue search | ✅ | ❌ | `search_string` |
| `Contact` | WhatsApp / contact click | ✅ | ❌ | — |

**Rules:**
- `event_id` must be identical between browser and server for every paired event (Purchase: `order number`).
- Server events are sent **after** the order is durably created, never before.
- `value` must be the **order subtotal actually charged**, in the order's currency — no rounding drift from float maths.
- Server events are fire-and-forget from the request's perspective: **checkout must never fail because Meta is down.**

---

## 8. Component inventory

### 8.1 New code we own

| Component | Type | Contents |
|---|---|---|
| `themes/wondersoul/` | Theme | `config.json`, `views/` (home, collection, product, plus overrides), `assets/scss/` (base→layout→components→modules→pages), `assets/js/`, `boot.php`, `demo/Seeder.php` |
| `plugins/Midtrans/` | Plugin | `config.json`, `fields.php`, `Boot.php`, `Services/MidtransService.php`, `Controllers/MidtransController.php`, `Routes/front.php`, `Views/payment.blade.php`, `Lang/{en,id}/common.php`, `Tests/Feature/` |
| `plugins/MetaPixel/` | Plugin | `config.json`, `fields.php`, `Boot.php`, `Services/{MetaPixelService,CapiClient}.php`, `Controllers/PixelHealthController.php`, `Lang/{en,id}/common.php`, `Tests/Feature/` |
| `plugins/Cod/` *(conditional)* | Plugin | `config.json`, `fields.php`, `Boot.php`, `Views/payment.blade.php`, `Lang/{en,id}/common.php` |
| `plugins/WondersoulTools/` | Plugin | MCP/AI tools: `Tools/{DropsTool,StockTool,BestSellersTool}.php` registered via `ai.tools` |
| `Dockerfile` / `Caddyfile` | Infra | FrankenPHP + Caddy, multi-stage |
| `vercel.json` | Infra | Container preset, crons, region |
| `app/Console/Commands/WondersoulInstall.php` | CLI | Migration + seed + `storage/installed` marker (replaces the browser wizard on Vercel) |

### 8.2 Upstream we consume unchanged

InnoShop core (`innopacks/*`), the `PageBuilder` plugin (for lookbook/editorial pages), `Stripe` + `PayPal` plugins (worldwide card rail), `FixedShipping` (shipping rules), `laravel/ai` + `laravel/mcp`.

---

## 9. Dependencies & environment

| Requirement | Status | Action |
|---|---|---|
| PHP 8.3 + extensions | 🔴 **Missing** | Install locally (XAMPP/Laragon/standalone) |
| Composer 2 | 🔴 **Missing** | Install |
| Node 26 / npm 11 | ✅ Present | — |
| MySQL (managed) | 🔴 Needed | Provision for staging + production |
| S3-compatible bucket | 🔴 Needed | For the `media` disk |
| Vercel account (Pro) | 🔴 Needed | Container runtime requires it |
| Midtrans account | 🔴 Needed | Sandbox first; production activation needs business docs + website URL live |
| Meta Business Manager + Pixel | 🔴 Needed | Create Pixel + CAPI token; site must be live and matching |
| Domain | 🔴 Needed | For Meta domain verification and Midtrans production |
| Brand assets (logo, type, photography) | 🔴 Needed from owner | |

---

## 10. Risk register

| Risk | Likelihood | Impact | Mitigation |
|---|---|---|---|
| Vercel container cold starts hurt conversion | Medium | Medium | Warm the function; keep the storefront cacheable at the edge; Vercel Fluid/Compute if needed |
| Managed MySQL cost exceeds budget | Medium | Medium | Compare PlanetScale vs Aiven vs DigitalOcean managed MySQL; Phase 0 decision |
| Midtrans webhook missed → customer paid, store says unpaid | **High** if unmitigated | **High** | Idempotent handler + **status reconciliation sweep** on a schedule + manual "check status" button in the order panel |
| CAPI/browser event mismatch → double-counted purchases | Medium | High | Single shared `event_id` helper used by *both* paths; covered by a feature test |
| Checkout blocked by a slow Meta API call | Medium | High | CAPI dispatch is queued, never synchronous inside the checkout request |
| Checkout broken by the container build | Medium | High | Stage + preview deployments; the theme/plugin set is small and testable |
| Payment marked paid twice (webhook + reconciliation) | Medium | Medium | Idempotency key on `order_id`; state machine guards |
| Owner cannot operate the panel | Medium | High | Phase 1 includes a written admin runbook; keep the panel surface small |
| Catalog content never arrives | **High** | **High** | Phase 0 gate: owner supplies the 88-post export before Phase 1 starts |
| Meta Pixel/CAPI event schema rejected (API version drift) | Medium | Medium | Re-verify against live Meta docs at implementation time; pin the Graph API version |
| Site goes down and nobody notices | Low | High | Uptime check + webhook-failure alert + daily order sanity check |

---

## 11. Open decisions — owner input required

These change what gets built. **Q1 and Q2 are blocking.**

| # | Question | Why it changes the build | Default if unanswered |
|---|---|---|---|
| **Q1** | **The 88 Instagram posts** — can you export them (or paste the bio + last ~20 captions)? | Gives the real product taxonomy, price band, colourway names and size matrix. Without it, the theme and catalogue are designed against guesses and the brand's actual visual language is lost. | Build against the 4-collection taxonomy in §4.1 and standard S–XXL |
| **Q2** | **Do you sell COD today?** | If yes, `plugins/Cod` is a v1 deliverable (~1–2 days). If no, we skip it. | Build it — COD dominates Indonesian fashion e-commerce |
| **Q3** | **Which countries do your "worldwide" orders come from, and what do they pay with?** | Decides whether Stripe is enough or whether Midtrans international is also needed | Enable Stripe + PayPal + Midtrans |
| **Q4** | **Made-to-order, pre-order, or stocked?** | Determines whether we need a waitlist/stock-notify flow and whether stock is real-time | Stocked, with stock shown as remaining |
| **Q5** | **Is the WonderSoul Weekender music collective part of the business?** | Decides whether the site also hosts releases, event pages or tickets — and whether the launch story is "a label" or "a community" | Assume yes: model a "Release" as a first-class concept, but ship no ticketing |
| **Q6** | **Bilingual ID/EN from day one, or ID only for launch?** | Affects content volume, SEO and the `hreflang` setup | Bilingual, EN for the homepage/collection/product pages only |
| **Q7** | **Vercel Pro + managed MySQL + S3 (~USD 60–150/mo) — acceptable?** | If not, Approach B (VPS) is cheaper and simpler | Assume yes; flag costs before Phase 4 |
| **Q8** | **Domain?** (`wondersoul.jkt` / `wondersoul.co.id` / other) | Required for Meta verification and Midtrans production activation | Reserve `wondersoul.co.id` and point it at Vercel |
| **Q9** | **Return/exchange policy, and who pays return shipping?** | Needed for policy pages, and for the COD flow's risk model | 14-day returns, buyer pays return shipping (common for local labels) |

---

## 12. Definition of Done (v1)

- [ ] `vercel deploy` → production URL live on the final domain, HTTPS, no `/install` redirect
- [ ] Indonesian is the default locale; English available; IDR and one foreign currency selectable
- [ ] Catalogue: 4 collections, garments with colourways, size charts, fit notes, releases
- [ ] A full order completes in production: add to cart → checkout → Midtrans sandbox → webhook → order paid → owner sees it in the panel
- [ ] COD (if Q2 = yes) completes the same flow
- [ ] Events Manager shows deduplicated Purchase events with correct value and currency
- [ ] MCP endpoint responds; `claude mcp add` succeeds; ≥ 3 brand tools listed on `/mcp`
- [ ] `themes/wondersoul` builds from a clean checkout with one command
- [ ] Owner completes a full fulfilment cycle unaided, using the runbook
- [ ] Backup + restore rehearsed; rollback documented; alerts firing

---

## 13. What I need from you next

1. **Answer Q1 and Q2** — they unblock Phase 1 and Phase 2 respectively.
2. **Confirm or reject the architecture in §5** (Approach A, managed MySQL).
3. **Give the go-ahead on Phase 0** — it is mostly environment setup and content collection, and nothing downstream can start without it.

Everything in §11 marked "default if unanswered" has a defensible default, so unanswered questions will not stall the build — but Q1 and Q2 will produce a *wrong* catalogue if guessed.
