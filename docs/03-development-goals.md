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
| **G2** | Deployable to a **production server** (Nginx + PHP-FPM + MySQL + Supervisor), with **localhost staging** as the development environment | `git pull` on the server → live site; one-command rollback; no platform-specific rewrites of InnoShop |
| **G3** | **Midtrans** (Snap) as the primary Indonesian payment rail | ≥ 95% of online Indonesian orders paid without manual reconciliation |
| **G4** | **Meta Pixel + CAPI** with correct browser↔server event dedup | Events Manager shows matched events; Purchase ROAS attributable; custom audiences (ViewContent→AddToCart, 30d) populating |
| **G5** | **MCP** enabled, with brand-specific tools for AI-assisted merchandising | `claude mcp add` works; ≥ 3 Wondersoul-specific tools registered |
| **G6** | **Theme template** `themes/wondersoul` built from scratch and owned by us | Theme builds with `THEME=wondersoul npm run build`; no upstream theme fork required |
| **G7** | Bilingual (ID primary, EN secondary) with multi-currency | A Jakarta buyer sees IDR + Bahasa; a worldwide buyer sees their currency + English |
| **G8** | COD available if the business sells it today | COD appears at checkout and order state transitions correctly on delivery confirmation |

### 2.1 Non-goals (explicitly out of scope for v1)

- ❌ Multi-vendor / marketplace mode (one brand, one operator)
- ❌ Native mobile apps
- ❌ Live chat on the storefront — Reverb is available on the production server, but the value is low for this traffic profile and the owner's WhatsApp already handles it
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

## 5. Architecture — deployment target

> **Decision (owner, 2026-11-21):** Vercel is dropped as the deployment target — *"Karena Vercel tidak kompatible kita lakukan deployment di staging localhost dahulu, lalu goals akan menjadi upload ke server production"* — and **G2 becomes "upload to a production server"**.
>
> This resolves the single largest technical risk in the plan. It also **removes** the highest-cost line item. The consequences are recorded below.

### 5.1 Why dropping Vercel is the right call

The Vercel path required *rewriting InnoShop's deployment story*: no official PHP runtime (→ Container + FrankenPHP), no persistent workers (→ Horizon and Reverb both dropped, Reverb not replaceable), a browser installer that writes `.env` and `storage/installed` (→ patched and bypassed), an ephemeral filesystem (→ media forced onto S3), and per-request cold starts. Every one of those was a permanent tax on the codebase.

A production server removes all of them — and lets InnoShop run **the topology its own documentation prescribes**, unmodified.

| | Vercel (dropped) | Production server ✅ |
|---|---|---|
| PHP runtime | Container + FrankenPHP (rewritten) | Native PHP-FPM 8.3 |
| Queue | DB driver + Cron (minute latency) | **Horizon** — real daemon, sub-second |
| Websockets | Not possible | **Reverb** — official InnoShop topology |
| Scheduler | Cron-over-HTTP workaround | `schedule:work` / crontab |
| Media storage | Forced to S3/R2 (extra service) | **Local disk** — `storage/app/public`, one less dependency |
| Install wizard | Broken; needed a CLI bypass + image patch | **Works as shipped** (writable FS) |
| Backups / TLS / monitoring | Provided by platform | **Our responsibility** (see §10) |
| Cold starts | Yes | No |
| Cost | USD 60–150/month | **USD 12–24/month** + domain |
| Rollback | Platform deployments | `git checkout <previous-tag>` + `migrate:rollback` |

### 5.2 Target topology

```
┌─ LOCAL STAGING (this machine) ─────────────────────────────┐
│  PHP 8.3 + Composer 2 + MySQL 8 + Node 26                   │
│  php artisan serve  (fast loop)                             │
│  nginx + php-fpm on :8080  (staging-accurate)               │
└───────────────┬─────────────────────────────────────────────┘
                │  git push  →  git pull on server
                ▼
┌─ PRODUCTION SERVER (Ubuntu 24.04 LTS) ─────────────────────┐
│  Nginx  →  PHP-FPM 8.3   (public/nginx.conf is InnoShop's  │
│                            own production vhost — reuse it) │
│  MySQL 8   (local, daily mysqldump → offsite)               │
│  Supervisor                                                     │
│    ├─ horizon   (queue daemon)                                 │
│    ├─ reverb    (websockets, :8080)                           │
│    └─ scheduler (schedule:work)                                │
│  storage/app/public  (media, local disk — served by Nginx)    │
│  Let's Encrypt TLS, auto-renew                                │
└───────────────────────────────────────────────────────────────┘
```

**Reference:** InnoShop's own [Supervisor Process Management](https://docs.innoshop.cn/en/development/supervisor.html) doc prescribes exactly this — Horizon + Reverb + scheduler under Supervisor, with `QUEUE_CONNECTION=redis` and `BROADCAST_CONNECTION=reverb`.

### 5.3 Local staging — the immediate environment

Because staging is localhost, the **toolchain is the critical path**, and it is currently absent:

| Need | Status | Note |
|---|---|---|
| **PHP 8.3+** | 🔴 **Not installed** | Required extensions: bcmath, curl, dom, fileinfo, libxml, openssl, **pdo_mysql**, simplexml, gd/imagick, exif, intl, zip |
| **Composer 2** | 🔴 **Not installed** | — |
| **MySQL 8** | 🔴 Not installed | Or SQLite for a quick first-run try-out (InnoShop supports it) |
| Node 26 / npm 11 | ✅ Present | — |

Once installed, `composer install && npm install && npm run build && php artisan migrate && php artisan serve` gives a full local storefront. `php artisan serve` is fine for the dev loop; **nginx + php-fpm locally** is worth adding in Phase 4 so staging matches production's request handling (permissions, `public/` serving, upload limits, TLS-less HTTPS).

### 5.4 Deployment mechanism

Plain git + a release script on the server. No CI provider, no container registry, no platform deploy hooks — a one-file script the owner can run and read:

```
deploy/
├── deploy.sh              # pull → composer install → npm ci → npm run build → migrate → restart workers
├── nginx/wondersoul.conf  # production vhost (derived from public/nginx.conf)
└── supervisor/
    ├── horizon.ini
    ├── reverb.ini
    └── scheduler.ini
```

**Rules:** `git pull` on the server is the only deploy action; migrations run in the script with `--force`; Supervisor processes restart at the end; the previous release is a git tag, so rollback is `git checkout <tag> && php artisan migrate:rollback --step=1`.

### 5.5 Database

**MySQL 8 on the production server, local instance for staging.** InnoShop's schema is MySQL-specific (`DB_PREFIX=inno_`, MySQL DDL and collations), so Postgres was never a low-cost swap — on a self-hosted VPS the MySQL-vs-Postgres cost difference disappears entirely. One engine, one set of migrations, one backup procedure.

### 5.6 What we take on instead of the platform

Dropping Vercel means these become **our** responsibilities, and they belong on the Phase 4 / Phase 6 checklists:

1. **Backups** — daily `mysqldump` + `storage/` tar, retained offsite, with a **rehearsed restore**.
2. **TLS** — Let's Encrypt with auto-renewal; a failed renewal is a silent outage.
3. **Monitoring** — uptime check, disk-space alert, MySQL-slow-query watch.
4. **Security patching** — `unattended-upgrades` for OS, and a deliberate composer-update cadence.
5. **A second server** — a single VPS is a single point of failure; a snapshot-based standby (Hetzner Snapshots, DigitalOcean Snapshots) is the cheap mitigation.

---

## 6. Phased roadmap

Each phase has an explicit exit criterion. Estimates assume **one full-time developer** plus owner availability for content.

### Phase 0 — Foundation *(~1 week)*

| Task | Detail |
|---|---|
| Toolchain | Install **PHP 8.3+** (required extensions: bcmath, curl, dom, fileinfo, libxml, openssl, **pdo_mysql**, simplexml, gd, exif, intl, zip) + **Composer 2** + **MySQL 8** + **Redis**. ⚠️ None of the three are currently on PATH on this machine — this is the critical path for everything else |
| Local staging | `composer install` → `npm install && npm run build` → `php artisan key:generate` → `php artisan migrate` → `php artisan serve` (dev loop), with Nginx + PHP-FPM added in Phase 4 for fidelity |
| Baseline repo | Initialise `laravel_innoshop` as a git repo; tag `upstream/v1.0.1`; commit the InnoShop baseline unmodified |
| Brand assets | Logo, wordmark, colour palette, type stack, favicon, OG image, product photography |
| Copy | Indonesian + English: home, collection, about, size chart, shipping, returns, contact, privacy, terms |
| Legal | Business entity, NPWP/registration, consumer-protection terms — **required for Midtrans production activation** |
| Config | `APP_NAME=WONDERSOUL JAKARTA`, `APP_URL`, `APP_LOCALE=id`, `APP_FALLBACK_LOCALE=en`, IDR + IDR/USD currency, `system_setting('theme') = 'wondersoul'` |

**Exit:** `php artisan serve` renders the InnoShop front end in Indonesian with Wondersoul branding; `php artisan migrate` succeeds against local MySQL 8; `git log` shows the baseline commit.

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

### Phase 4 — Staging → production server *(~1 week)*

**Split into two halves: first make localhost staging accurate, then provision and cut over the production server.**

#### 4a. Local staging fidelity *(~2 days)*

| Task | Detail |
|---|---|
| Local Nginx + PHP-FPM | Mirror production request handling (permissions, `public/` serving, upload limits, `client_max_body_size 100M`). `php artisan serve` stays the fast dev loop; nginx is the fidelity check. |
| Local Supervisor | Horizon + Reverb + scheduler running locally, using the same `deploy/supervisor/*.ini` files as production — so a config bug surfaces on localhost, not in production |
| Local MySQL 8 | Same engine and version as production. **Never develop against SQLite** — DDL and collation differences hide real bugs |
| Media on local disk | `FILESYSTEM_DISK=public`, `storage/app/public` served by Nginx; confirm the `media` disk configuration resolves (§8 risk 9) |
| `storage/` permissions | `chown -R www-data:www-data storage bootstrap/cache` — the single most common Laravel-on-Nginx failure |
| `.env` for staging | `APP_ENV=staging`, `APP_DEBUG=false`, `APP_URL=http://localhost:8080`, real `APP_KEY`, `LOG_CHANNEL=daily` |

#### 4b. Production server *(~3 days)*

| Task | Detail |
|---|---|
| Provision | Ubuntu 24.04 LTS VPS (Hetzner CX22 / DigitalOcean 2 vCPU). **≈ USD 12–24/month.** A snapshot-capable provider is preferred, for cheap standby |
| Base stack | Nginx, PHP-FPM 8.3 (all required extensions), MySQL 8, Redis (for `QUEUE_CONNECTION=redis` + Reverb), Composer 2, Node 22 LTS (build-time only) |
| Nginx vhost | **Reuse InnoShop's own `public/nginx.conf`** — it already contains the rewrite rules, static caching, security headers, upload limit and the `/vendor` + `/storage` PHP-execution denies. Adjust the server name, root and `fastcgi_pass` socket path only |
| TLS | Let's Encrypt via `certbot --nginx`, auto-renewal verified (`certbot renew --dry-run`) |
| Supervisor | `deploy/supervisor/{horizon,reverb,scheduler}.ini` — mirroring the layout in InnoShop's Supervisor doc. `BROADCAST_CONNECTION=reverb`, `QUEUE_CONNECTION=redis` |
| Release script | `deploy/deploy.sh`: `git pull` → `composer install --no-dev --optimize-autoloader` → `npm ci && npm run build` → `THEME=wondersoul npm run build` → `php artisan migrate --force` → `php artisan config:clear && cache:clear` → `php artisan storage:link` → `supervisorctl restart all` |
| Installer | The **browser wizard works on a real server** (writable FS) — use it for the first boot. Keep `php artisan wondersoul:install` as a documented CLI path for re-provisioning |
| Media | Local disk via `storage/app/public` + `storage:link`; Nginx serves `/storage` directly. **No S3 needed** |
| Backups | Daily `mysqldump` + `storage/` tar → offsite (S3 bucket or restic). **A restore must be rehearsed**, not assumed |
| Monitoring | Uptime check (external, e.g. UptimeRobot) + disk-space and MySQL-connection alerts. A dead server that nobody notices is worse than a slow one |
| DNS | `wondersoul.jkt` / `wondersoul.co.id` → A record to the VPS; `www` → apex |

**Exit:** a full order completes on the production server (Midtrans sandbox → webhook → order paid → visible in the panel); media uploads land in `storage/app/public` and are served by Nginx; `supervisorctl status` shows Horizon, Reverb and the scheduler running; a TLS renewal dry-run succeeds; and a database restore has been performed once, successfully, from a backup.

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
| Performance | Image optimisation (WebP/AVIF) + lazy loading, Core Web Vitals budget, Nginx gzip/brotli + long-cache static assets |
| Legal | ToS, privacy, return/exchange policy, NPWP/business data in the footer |
| Accessibility | Contrast, focus states, keyboard nav on the custom theme |
| Observability | Uptime check (external), Pixel-health widget, webhook-failure alert (email), daily order-count sanity check, disk-space alert |
| Backup | Daily `mysqldump` + `storage/` tar to offsite, with a **rehearsed restore** |
| Rollback | Documented rollback procedure (previous git tag + `migrate:rollback` + server snapshot) |

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
| `deploy/` | Infra | `deploy.sh` (git-pull release script), `nginx/wondersoul.conf` (derived from InnoShop's own `public/nginx.conf`), `supervisor/{horizon,reverb,scheduler}.ini` |
| `app/Console/Commands/WondersoulInstall.php` | CLI | Optional: a scripted install for re-provisioning the server without the browser wizard |

### 8.2 Upstream we consume unchanged

InnoShop core (`innopacks/*`), the `PageBuilder` plugin (for lookbook/editorial pages), `Stripe` + `PayPal` plugins (worldwide card rail), `FixedShipping` (shipping rules), `laravel/ai` + `laravel/mcp`.

---

## 9. Dependencies & environment

| Requirement | Status | Action |
|---|---|---|
| PHP 8.3 + extensions | 🔴 **Missing** | Install locally (XAMPP/Laragon/standalone) |
| Composer 2 | 🔴 **Missing** | Install |
| Node 26 / npm 11 | ✅ Present | — |
| MySQL 8 | 🔴 Needed | Local instance for staging + on the production VPS |
| Redis | 🔴 Needed | `QUEUE_CONNECTION=redis` + `BROADCAST_CONNECTION=reverb`, per InnoShop's Supervisor doc |
| Production VPS | 🔴 Needed | Ubuntu 24.04, Nginx + PHP-FPM 8.3 + MySQL 8 + Redis + Supervisor. ≈ USD 12–24/month |
| S3-compatible bucket | 🟢 **Dropped** | Media lives on local disk (`storage/app/public`); only a small offsite **backup** bucket is needed |
| Vercel account | 🟢 **Dropped** | Not required |
| Midtrans account | 🔴 Needed | Sandbox first; production activation needs business docs + website URL live |
| Meta Business Manager + Pixel | 🔴 Needed | Create Pixel + CAPI token; site must be live and matching |
| Domain | 🔴 Needed | For Meta domain verification and Midtrans production |
| Brand assets (logo, type, photography) | 🔴 Needed from owner | |

---

## 10. Risk register

| Risk | Likelihood | Impact | Mitigation |
|---|---|---|---|
| Vercel container cold starts hurt conversion | — | — | ❌ **Dropped with the Vercel target** |
| Managed MySQL cost exceeds budget | — | — | ❌ **Dropped** — MySQL runs on the same VPS |
| Midtrans webhook missed → customer paid, store says unpaid | **High** if unmitigated | **High** | Idempotent handler + **status reconciliation sweep** on a schedule + manual "check status" button in the order panel |
| CAPI/browser event mismatch → double-counted purchases | Medium | High | Single shared `event_id` helper used by *both* paths; covered by a feature test |
| Checkout blocked by a slow Meta API call | Medium | High | CAPI dispatch is queued, never synchronous inside the checkout request |
| Checkout broken by the container build | — | — | ❌ **Dropped** — `deploy/deploy.sh` runs migrations and restarts workers in one atomic step; the previous git tag is the rollback target |
| Payment marked paid twice (webhook + reconciliation) | Medium | Medium | Idempotency key on `order_id`; state machine guards |
| Owner cannot operate the panel | Medium | High | Phase 1 includes a written admin runbook; keep the panel surface small |
| Catalog content never arrives | **High** | **High** | Phase 0 gate: owner supplies the 88-post export before Phase 1 starts |
| Meta Pixel/CAPI event schema rejected (API version drift) | Medium | Medium | Re-verify against live Meta docs at implementation time; pin the Graph API version |
| **Server goes down and nobody notices** | Medium | **High** | External uptime check + disk-space alert + webhook-failure email. *This risk was transferred from the platform to us.* |
| **No backups, or backups that were never restored** | Medium | **High** | Daily `mysqldump` + `storage/` tar to offsite; **rehearse a restore before launch**, and again quarterly |
| **TLS certificate expires silently** | Low | **High** | `certbot --nginx` auto-renew + a `certbot renew --dry-run` check on the release checklist + expiry monitoring |
| **Single VPS = single point of failure** | Medium | High | Provider snapshots (cheap standby) + a documented re-provision path (`deploy/deploy.sh` must work from a clean server) |
| **Server security rot (no OS patching, SSH exposed)** | Medium | High | `unattended-upgrades`, SSH key-only auth, fail2ban, UFW, quarterly composer-update cadence |
| Disk fills with logs/uploads → MySQL or app dies | Low | High | Log rotation, `logrotate` for `storage/logs`, disk-usage alert at 80% |
| Midtrans webhook endpoint not reachable over HTTPS | Low | **High** | Let's Encrypt cert valid before going live; verify the Midtrans dashboard shows a successful notification delivery |
| Nginx/PHP permission mistake → 500s in production | Low | High | Local nginx staging mirrors production; `storage/` + `bootstrap/cache` ownership set in the release script |

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
| **Q7** | ~~Vercel Pro + managed MySQL + S3 (~USD 60–150/mo)~~ | ✅ **Resolved 2026-11-21** — dropped. Replaced by a single VPS (≈ USD 12–24/month) | Resolved |
| **Q7b** | **Which VPS provider?** (Hetzner / DigitalOcean / Linode / Contabo) | Determines price, data-centre proximity to Jakarta (Singapore region preferred), snapshot support, and who holds the account | Hetzner (cheapest, snapshots, Singapore region) unless the owner already has an account elsewhere |
| **Q8** | **Domain?** (`wondersoul.jkt` / `wondersoul.co.id` / other) | Required for Meta verification and Midtrans production activation | Reserve `wondersoul.co.id`, A record to the VPS |
| **Q9** | **Return/exchange policy, and who pays return shipping?** | Needed for policy pages, and for the COD flow's risk model | 14-day returns, buyer pays return shipping (common for local labels) |
| **Q10** | **Who holds the VPS + domain + Midtrans + Meta accounts?** | If accounts sit with one person, a departure takes the business with it. Affects the runbook and the handover documents | Assume the owner holds them; the runbook documents every console URL and credential location |

---

## 12. Definition of Done (v1)

- [ ] Production server serves the storefront on the final domain over HTTPS, with no `/install` redirect
- [ ] Indonesian is the default locale; English available; IDR and one foreign currency selectable
- [ ] Catalogue: 4 collections, garments with colourways, size charts, fit notes, releases
- [ ] A full order completes in production: add to cart → checkout → Midtrans sandbox → webhook → order paid → owner sees it in the panel
- [ ] COD (if Q2 = yes) completes the same flow
- [ ] Events Manager shows deduplicated Purchase events with correct value and currency
- [ ] MCP endpoint responds; `claude mcp add` succeeds; ≥ 3 brand tools listed on `/mcp`
- [ ] `themes/wondersoul` builds from a clean checkout with one command
- [ ] `deploy/deploy.sh` takes the server from the previous tag to the new one in a single run
- [ ] Supervisor reports Horizon, Reverb and the scheduler running; order confirmation email arrives in seconds, not minutes
- [ ] Owner completes a full fulfilment cycle unaided, using the runbook
- [ ] Backup + **restore rehearsed**; rollback documented; uptime, disk and webhook alerts firing

---

## 13. What I need from you next

1. **Answer Q1 and Q2** — they unblock Phase 1 and Phase 2 respectively.
2. **Confirm the revised architecture in §5** — local staging → a single production VPS running InnoShop's documented topology (Nginx + PHP-FPM + MySQL + Redis + Supervisor). Vercel is out; cost drops from ≈ USD 60–150 to ≈ USD 12–24/month.
3. **Give the go-ahead on Phase 0** — it is mostly environment setup (PHP 8.3 + Composer + MySQL are **not installed** on this machine) and content collection. Nothing downstream can start without it.

Everything in §11 marked "default if unanswered" has a defensible default, so unanswered questions will not stall the build — but Q1 and Q2 will produce a *wrong* catalogue if guessed.
