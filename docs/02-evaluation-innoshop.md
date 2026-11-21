# Source Code Evaluation — InnoShop Baseline

**Subject:** `laravel_innoshop` (working copy of InnoShop CE, upstream `innocommerce/innoshop`)
**Upstream baseline:** v1.0.1 (released 2026-11-18) · **Source of truth:** <https://github.com/innocommerce/innoshop/blob/main/README.md>
**Evaluated:** 2026-11-21 · against the four development goals for WONDERSOUL JAKARTA.

---

## 0. Executive summary

| # | Goal | Verdict | Effort to close |
|---|---|---|---|
| 1 | Ecommerce on InnoShop, buildable to **Vercel** | ⚠️ **Blocked by architecture, not by code.** Vercel has no official PHP runtime, and InnoShop's documented production topology (Horizon + Reverb + Supervisor) requires persistent processes Vercel cannot host. **→ Resolved 2026-11-21: Vercel dropped; production = Nginx + PHP-FPM + MySQL + Redis + Supervisor.** | **Closed** |
| 2 | **Meta Pixel (CAPI)** tracking | ❌ **Entirely absent.** Zero pixel/analytics/GTM code anywhere in the codebase. | **Low** — clean plugin, one known hook point |
| 3 | **Midtrans** payment gateway | ❌ **Absent.** But the payment-plugin contract is clean and well-documented; Stripe is a full reference implementation. | **Medium** — one plugin, ~1 week |
| 4 | **MCP** integrated, plugin mechanisms, theme templates | ✅ **Substantially built in.** MCP server, 45+ tools, AI core, hook system, plugin system, theme system all ship with core. | **Low** — mostly configuration + custom tools |

**The headline:** InnoShop is an excellent fit for goals 2, 3 and 4, and a *poor* fit for goal 1 as originally stated. Goal 1 required an explicit decision about what Vercel is allowed to be — **taken on 2026-11-21: Vercel dropped, production moves to a conventional server.** See §2 and `03-development-goals.md §5`.

---

## 1. Baseline inventory

### 1.1 Stack

| Component | Value | Evidence |
|---|---|---|
| Framework | **Laravel 13** | `composer.json:15` → `"laravel/framework": "^13.0"` |
| PHP | **^8.3** | `composer.json:8`; platform pinned to `8.3.0` (`composer.json:157`) |
| Frontend build | **Vite 8** (not Laravel Mix) | `package.json`, `build.js` |
| Auth | Laravel Sanctum 4 | `composer.json:26` |
| Queue admin | **Laravel Horizon 5** | `composer.json:23` |
| Websockets | **Laravel Reverb 1** | `composer.json:24` |
| Server | Laravel Octane 2 | `composer.json:25` |
| AI | **`laravel/ai` ^0.10.1** | `composer.json:19` |
| MCP | **`laravel/mcp` ^1.1** | `composer.json:20` |
| Permissions | spatie/laravel-permission 8 | `composer.json:42` |
| Storage | Flysystem AWS S3 v3 | `composer.json:34` |
| Payments (core) | `srmklive/paypal`, `stripe/stripe-php` | `composer.json:40-41` |

> ⚠️ **The repo's own `CLAUDE.md` is stale.** It says "Laravel Mix", "`webpack.mix.js`", "`themes/` (e.g. `petnow`, `thangka`)" and "PHP 8.2+". The actual code is Laravel 13 / Vite 8 / PHP 8.3 / theme layout `assets/scss` + `assets/js`. Treat `CLAUDE.md` as documentation debt, not a spec.

### 1.2 Module layout (`innopacks/`)

`aicore` · `common` · `devtools` · `front` · `install` · `mcp` · `panel` · `plugin` · `restapi`

The README's headline claims — *"integrated with multiple AI models and native MCP for intelligent automation, and featuring plugin mechanisms and theme template development"* — are **backed by real code**, not aspirational. `innopacks/mcp/` and `innopacks/aicore/` are substantial, with their own service providers, lang trees (20 locales), views, and tests.

### 1.3 Bundled plugins (`plugins/`)

`BankTransfer` · `FixedShipping` · `PageBuilder` · `PayPal` · `Stripe`

All five declare `"type": "payment"` or a non-payment type in `config.json`. A dedicated migration `innopacks/plugin/database/migrations/2026_03_09_000000_update_plugin_type_billing_to_payment.php` confirms `billing` → `payment` was the recent canonicalisation.

### 1.4 Themes — ⚠️ **empty**

`themes/` contains **only a `.gitignore`**. Confirmed upstream too (`api.github.com/repos/innocommerce/innoshop/contents/themes` → only `.gitignore`). Themes are **not distributed in the source repo**; they ship in the release ZIP (`innoshop-v1.0.1.zip`) or via the InnoShop marketplace.

**Consequence:** there is no upstream theme to fork locally, and the `themes/` directory must be populated from the release ZIP or built from scratch. Since goal 4 explicitly calls for *theme template development*, building `themes/wondersoul/` from scratch is in scope regardless — but it means **starting from zero, not forking**, which materially raises the theme estimate.

Theme mechanics verified:
- Theme selection is a DB system setting: `system_setting('theme')` — `innopacks/front/src/FrontServiceProvider.php:238,278,301,337`
- Theme assets: `theme_asset()` / `theme_image()` — `innopacks/common/helpers.php:1542+`
- Theme views: `inno_view()` — `innopacks/common/helpers.php:652`
- Theme SCSS alias `@front` → `innopacks/front/resources` (per official docs)
- Theme build: `THEME=<code> npm run build` → `public/static/themes/<code>/`
- Theme boot: `boot.php` at theme root; `setup/boot.php` is **deprecated** (`FrontServiceProvider.php:314` logs a warning)

### 1.5 Internationalisation — ✅ Indonesian already supported

`innopacks/common/lang/` contains **`id`** alongside `ar de en es fr it ja km ko …`. The `id` locale is present out of the box; Indonesian is not a localisation project, it is a configuration switch. The aicore module also ships an `id` locale.

---

## 2. Goal 1 — Build to Vercel

> ✅ **RESOLVED 2026-11-21 — Vercel dropped as the deployment target.** The owner decided to stage on localhost and deploy to a conventional production server instead. **The entire section below is now historical context**, but the findings are retained because they explain *why* the decision was correct, and because several of them (the installer writing to disk, the `media` disk, the `.env` gate) still shape the production-server plan. See [`03-development-goals.md §5`](03-development-goals.md) for the revised architecture.
>
> **What changed in practice:** the Container runtime, the `vercel.json` crons, the CLI install bypass, the S3 media requirement and the `LOG_CHANNEL=stderr` workaround are all **dropped**. In their place: Nginx + PHP-FPM + MySQL + Redis + Supervisor, which is the topology InnoShop's own documentation prescribes — so Horizon, Reverb and the scheduler all work as shipped, and media lives on local disk. Monthly cost falls from ≈ USD 60–150 to ≈ USD 12–24.

### 2.1 The runtime problem (external, decisive)

Vercel's official runtime list — <https://vercel.com/docs/functions/runtimes> — is: **Node.js, Bun, Python, Rust, Go, Ruby, Wasm, Edge, Container.** *There is no PHP.*

The community PHP runtime (`vercel-community/php`, npm `vercel-php`) is **archived** (`"archived": true` via GitHub API; 1,468 stars; last push 2026-10-29). Its own README states:

> `vercel-php` is a **community-maintained custom runtime**. It is not an official Vercel runtime and is not supported by Vercel.
> For new projects, we recommend deploying PHP on Vercel using a **container runtime** instead.

Vercel's official guide (<https://vercel.com/kb/guide/deploy-php-on-vercel-with-docker>) prescribes **FrankenPHP + Caddy + Dockerfile**, selected via the Container preset.

**⇒ The only Vercel-supported path for InnoShop is a Container runtime (FrankenPHP).**

### 2.2 The process problem (internal, decisive)

InnoShop's own documentation (<https://docs.innoshop.cn/en/development/supervisor.html>) prescribes this production topology:

| Process | Purpose | Vercel-compatible? |
|---|---|---|
| **Horizon** (Redis queue) | Async jobs | ❌ Needs a long-running worker |
| **Reverb** :8080/:8081 | WebSockets | ❌ Needs a long-running worker |
| `schedule:work` / `schedule:run` | Scheduler | ⚠️ Only via **Vercel Cron** → HTTP endpoint |
| Redis | Shared | ⚠️ Replace with a managed Redis (Upstash) or use the DB driver |

Required env per InnoShop docs: `QUEUE_CONNECTION=redis`, `BROADCAST_CONNECTION=reverb`.

**⇒ Vercel + Horizon + Reverb is not a deployable combination. Something has to give.**

### 2.3 The installer problem (internal, decisive)

`public/index.php` gates every request on a **filesystem marker**:

```php
$installed = file_exists(__DIR__.'/../storage/installed');
if ((! $installed || ! $envValid) && ! $isInstallRoute && ! $isDebugbar) {
    header('Location: /install'); exit;
}
```

And the installer itself **writes to disk**:
- `innopacks/install/src/Libraries/Creator.php:94` → `file_put_contents(base_path('.env'), $envFileData)`
- `innopacks/install/src/Libraries/Installer.php:227` → `file_put_contents($this->basePath.'/.env', $envContent)`
- `innopacks/install/src/Libraries/Installer.php:248` → `mkdir(dirname($this->installedFile))` (`storage/installed`, line 40)
- `innopacks/install/src/Libraries/Checker.php:91` → `file_put_contents($path, $defaultEnv)` + `chmod`
- `innopacks/install/src/Libraries/Installer.php:232` → `Artisan::call('migrate:fresh', ['--force' => true])`
- `innopacks/install/src/Libraries/Creator.php:106,123` → `migrate:fresh` + `db:seed`

**⇒ The browser-driven install wizard cannot run on Vercel** (read-only filesystem, ephemeral container, no persistent `storage/installed`). The database must be seeded by a **CLI/one-off job** against a managed database, and the app must be made to believe it is installed.

### 2.4 Storage & state

| Concern | Default in `.env.example` | Vercel reality | Required change |
|---|---|---|---|
| Database | `mysql` | Vercel has no MySQL | → managed MySQL (PlanetScale/Aiven) — see risk §7.2 |
| Session | `database` | ✅ fine | none |
| Cache | `database` | ✅ fine | none |
| Queue | `database` | ✅ workable (no worker needed if jobs are dispatched inline) | none *if* DB queue + cron drain is acceptable |
| Filesystem | `local` (`storage/app`) | ❌ ephemeral, read-only-ish | → S3 disk (`config/filesystems.php:48` already defines an `s3` disk) |
| Media | `Storage::disk('media')` — `innopacks/common/src/Models/MediaFile.php:71`, `Services/VideoPosterService.php:75` | ❌ | → point `media` disk at S3/R2/Vercel Blob |
| Logs | `single` → `storage/logs` | ❌ ephemeral | → `LOG_CHANNEL=stderr` (FrankenPHP) or a log sink |
| Broadcast | `log` | ✅ fine (default) | keep `log`; do **not** enable Reverb |
| Install marker | `storage/installed` | ❌ | create in the image, or patch the gate |

### 2.5 What else the Vercel build must handle

- **Asset build**: `npm run build` (Vite 8) must run in the Vercel build step; `public/build/{front,panel,install}` already exist locally. Theme assets additionally need `THEME=wondersoul npm run build` → `public/static/themes/wondersoul/`.
- **`public/` also ships `nginx.conf`** (a standard InnoShop nginx vhost) — **now directly reusable** for the production server: it already carries the rewrite rules, static caching, security headers, the 100M upload limit, and the denies on `/vendor` and `/storage` PHP execution. Only the server name, root and `fastcgi_pass` socket path need adjusting.
- **`.env` on Vercel** is injected as env vars, not a file. `public/index.php` requires `file_exists('.env')` **and** `str_contains(file_get_contents('.env'), 'APP_KEY=')`. `$_ENV`/`getenv()` alone will not satisfy this gate → **the image must contain a minimal `.env`**, or `index.php` must be patched.
- **`config/filesystems.php`** defines `local`, `public`, `s3` — but **no `media` disk**, even though `MediaFile` and `VideoPosterService` call `Storage::disk('media')`. That disk must be defined. ❓ Where it is currently defined was not located in `config/*.php` — verify before relying on it.

### 2.6 Goal 1 verdict

**Not a code problem — an architecture decision that must be made explicitly.** Three coherent options are costed in `03-development-goals.md` §5.

> **Outcome (2026-11-21):** the decision was made — **Vercel dropped**, production moves to a conventional server. All the findings above then become *assets*: InnoShop's documented Supervisor topology works unmodified, the browser installer works (writable filesystem), media can live on local disk, and no code has to be patched for serverless constraints. The evaluation confirmed that Vercel was the wrong target for this codebase — the finding was right even though the conclusion changed.

**Risk — database engine:** InnoShop's schema is MySQL-specific (`DB_PREFIX=inno_`, raw MySQL DDL, `utf8mb4_*` collations, MySQL-flavoured migrations). Switching to Postgres is **not** a configuration change and is likely a multi-week migration. **Recommend: MySQL** — on a self-hosted VPS the MySQL-vs-Postgres cost difference disappears entirely, so there is no reason to pay the migration.

---

## 3. Goal 2 — Meta Pixel + CAPI

### 3.1 Current state: nothing exists ✅ (confirmed by exhaustive grep)

A repository-wide grep across `innopacks/`, `plugins/` (`*.php`, `*.blade.php`, `*.js`, `*.json`) for
`facebook|fbq|meta_pixel|conversion_api|googletagmanager|gtag|dataLayer|umami|matomo|analytics` returned **only**:

- `innopacks/common/src/Repositories/Customer/SocialRepo.php:26` → `'facebook'` (OAuth social **login**, unrelated)
- `innopacks/front/resources/views/account/_social.blade.php` → Facebook login button
- `innopacks/common/src/Console/Commands/TagBots.php:32` → `'facebot'`, `'facebookexternalhit'` (UA bot filtering)
- `innopacks/common/src/Services/BotClassifier.php:79` → `'meta-externalagent'` (SEO bot classification)
- `innopacks/common/src/Services/EventTrackingService.php`, `VisitRepo.php`, `VisitStatisticsService.php` → InnoShop's **own** first-party visit/event analytics (internal panel reporting, not Meta)

**Conclusion:** there is no pixel, no CAPI, no GTM, no GA4, and no outbound marketing-tracking code. There is also **no analytics table** to piggyback on for Meta events.

### 3.2 What the platform gives us for free

**The injection point exists and is exact:**

```
innopacks/front/resources/views/layouts/app.blade.php:51
    @hookinsert('front.layout.app.head.bottom')
```

This is inside `<head>`, after the app's own CSS/JS. **This is the correct place for the Meta Pixel base code (`fbq('init')` + `fbq('track', 'PageView')`).** ✅

**The hook API** (official docs <https://docs.innoshop.cn/en/development/hook.html>):

```php
listen_hook_filter('hook.name', fn($data) => $data, $priority);   // data hook
listen_hook_action('hook.name', fn($data) => null, $priority);     // process hook
listen_blade_insert('point.name', fn($data) => 'html');            // template insert
listen_blade_update('point.name', fn($output, $data) => 'html');   // template update
```

Verified hook names relevant to tracking (from `fire_hook_filter`/`listen_hook_*` across `innopacks/` + `plugins/`):

| Hook | Use for Meta Pixel |
|---|---|
| `service.cart.add.after` | ✅ **AddToCart** — fires on cart add |
| `service.cart.add.before` | ViewContent fallback |
| `service.cart.delete.after` | cart removal |
| `service.checkout.confirm.after` | ✅ **Purchase / InitiateCheckout** — fires on order creation |
| `service.checkout.checkout.result` | checkout completion |
| `service.checkout.billing.methods` | checkout step |
| `order.completed` | ✅ post-purchase / fulfilment |
| `order.cancelled` | cancellation |
| `order.refund.completed` | refund |
| `model.cart.item.price` / `model.order.item.type_label` | data enrichment |

**Data available for CAPI `Purchase`:** the `Order` model (`innopacks/common/src/Models/Order.php`) carries `billing_method_code`, `total`, `number`, customer/shipping fields (`shipping_city`, `shipping_address_1/2`, `shipping_zipcode`). `PaymentRepo` / `Models/Order/Payment.php` carry the payment records. ✅ Sufficient for `value`, `currency`, `content_ids`, `content_type`, `order_id`.

**Data missing for CAPI Advanced Matching:** ❓ No SHA-256 hashing helper for `em`/`ph`/`fn` was found. It must be added (trivial — `hash('sha256', mb_strtolower(trim($email)))`).

**Settings storage for plugin config:** ✅ verified — `plugin_setting($code, $key)` / `plugin_setting_array()` at `innopacks/plugin/helpers.php:58,84`; definitions live in the plugin's `fields.php` and are persisted by the plugin system. This is exactly how the Pixel ID / CAPI token should be stored.

### 3.3 Goal 2 verdict

**Lowest-effort goal of the four.** A `plugins/MetaPixel/` plugin needs:

| File | Purpose |
|---|---|
| `config.json` | `"type": "marketing"`, code `meta_pixel` |
| `fields.php` | `pixel_id`, `access_token`, `test_event_code`, `enabled` — via `label_key` (per repo CLAUDE.md) |
| `Boot.php` | `listen_blade_insert('front.layout.app.head.bottom', …)` for base code; `listen_hook_action('service.cart.add.after', …)`, `listen_hook_action('service.checkout.confirm.after', …)` for events |
| `Services/MetaPixelService.php` | Build payloads + POST to the Graph API `<PIXEL_ID>/events` endpoint |
| `Services/CapiClient.php` | HTTP client, `event_id` = order number for browser↔server dedup |
| `Controllers/` | optional: a panel "test events" page |
| `Lang/{en,id}/common.php` | required |
| `Tests/Feature/` | signature/dedup test (mirror `plugins/Stripe/Tests/Feature/WebhookSecurityTest.php`) |

⚠️ **Browser↔server dedup requires the browser `fbq()` call to carry the *same* `event_id` as the server event.** This means the theme (or the plugin's injected JS) must set `eventID` on `fbq('track', …)`. This is the most commonly-botched part of CAPI and must be specified up front, not retrofitted.

⚠️ **Meta's developer docs were not retrievable from this network** (JS-gated; `developers.facebook.com/docs/meta-pixel` returned a Facebook error page; `/docs/marketing-api/conversions-api` → HTTP 400). The Graph API version, endpoint shape, and the `event_id` dedup field must be re-verified against live Meta docs before implementation.

---

## 4. Goal 3 — Midtrans payment gateway

### 4.1 The payment-plugin contract (fully verified)

Core resolution happens in `innopacks/front/src/Services/PaymentService.php`:

```php
$originCode  = $this->order->billing_method_code;      // e.g. 'midtrans'
$paymentCode = Str::studly($originCode);                // e.g. 'Midtrans'
$viewPath    = fire_hook_filter("service.payment.pay.$originCode.view", "$paymentCode::payment");
...
$paymentData = [
    'order'           => $this->order,
    'payment_setting' => plugin_setting($paymentCode),
];
$paymentData = fire_hook_filter("service.payment.pay.$originCode.data", $paymentData);
```

**So a payment plugin needs exactly three things:**
1. `config.json` with `"type": "payment"` and a `code` (the code is the `billing_method_code`).
2. A view at `plugins/Midtrans/Views/payment.blade.php` — resolved by default via the `"$paymentCode::payment"` convention (`Midtrans::payment`). `view()->exists()` is checked; a missing view throws.
3. A `Boot.php` listener on `service.payment.pay.midtrans.data` to inject the gateway payload.

**Stripe as the reference implementation** (`plugins/Stripe/`), file by file:

```
plugins/Stripe/
├── Boot.php                    # listen_hook_filter('service.payment.mobile_pay.data', …)
├── composer.json               # ⚠️ legacy — repo CLAUDE.md says plugins must NOT have one
├── config.json                 # "type": "payment", "code": "stripe"
├── Controllers/StripeController.php
├── fields.php                  # publishable_key / secret_key / webhook_secret / test_mode / payment_mode
├── Lang/{en,zh-cn}/common.php
├── Public/{css,image,js}/
├── README.md, README.zh-cn.md  # ⚠️ dot notation required
├── Routes/front.php
├── Services/StripeService.php
├── Tests/Feature/WebhookSecurityTest.php
└── Views/
    ├── payment.blade.php
    └── partials/{checkout,elements}.blade.php
```

Payment-relevant hook names available: `service.payment.pay.<code>.view`, `service.payment.pay.<code>.data`, `service.payment.api.<code>.data`, `service.payment.api.bank_transfer.data`, `service.payment.mobile_pay.data`.

Payment methods surface to the storefront via `innopacks/plugin/src/Resources/Checkout/PaymentMethodItem.php`, which serialises `{type, code, name, description, icon}` from the plugin resource. ✅ So a Midtrans plugin appears in the checkout method list automatically, with its `config.json` icon.

### 4.2 Midtrans integration specifics (from official docs)

Midtrans publishes a machine-readable index: **`https://docs.midtrans.com/llms.txt`** — appending `.md` to any doc URL returns markdown. Verified available pages include:

- `snap.md` / `snap-snap-integration-guide.md` — Snap (built-in checkout UI). Server-side call is `POST https://app.midtrans.com/snap/v1/transactions` with `server_key` Basic auth, body `{transaction_details, customer_details, item_details, enabled_payments, callbacks}`; browser side is `snap.embed('<snap_token>', {embedId})` with `https://app.midtrans.com/snap/js/snap.js`
- `https-notification-webhooks.md` — the **server-to-server notification** (webhook) endpoint, which is what actually marks an order paid
- `get-status-api-requests.md`, `transaction-status-cycle.md` — status polling
- `api-authorization-headers.md` — `Authorization: Basic base64(server_key:)`
- `error-code-and-response-code.md`, `switching-to-production-mode.md`, `testing-payment-on-sandbox.md`
- `install-cms-plugins.md` — CMS plugin patterns

**Snap's callbacks** (`callbacks.finish`, `callbacks.unfinish`, `callbacks.error`) are browser redirects and are **untrustworthy**; the **notification webhook** is the authoritative payment signal and must be signature-verified.

⚠️ **The webhook signature algorithm was not fully extracted** from the downloaded doc (the fetch returned the full 109 KB page but my grep targeted the wrong anchors). It must be read directly from `https://docs.midtrans.com/docs/https-notification-webhooks.md` before implementation — specifically the `signature_key` / `hash` verification formula. **Do not implement Midtrans webhooks without it.**

### 4.3 Gap: COD (bayar di tempat) — ❌ absent

An exhaustive grep for `cash.?on.?delivery|cod` across `innopacks/common/src`, `innopacks/front/src` and `plugins/` found **no COD payment method** — only an unrelated comment at `innopacks/common/src/Services/StateMachineService.php:458`.

For an Indonesian fashion label, COD is typically the dominant payment method. ⚠️ If Wondersoul sells COD today via WhatsApp, **a COD payment plugin must be built** — it is not a config toggle.

### 4.4 Goal 3 verdict

**Medium effort, low risk, well-understood.** The contract is clean, Stripe is a complete reference, and Midtrans' docs are unusually good. Roughly: 1 plugin, ~8–12 files, 2–3 days of work including webhook signature verification and a sandbox test.

---

## 5. Goal 4 — MCP, plugin mechanisms, theme templates

### 5.1 MCP — ✅ built in, and genuinely good

`innopacks/mcp/` ships a complete **Streamable HTTP MCP server** (JSON-RPC 2.0):

| Concern | Implementation |
|---|---|
| Server | `innopacks/mcp/src/Server/InnoShopMcpServer.php` |
| Transport / routes | `innopacks/mcp/routes/mcp.php` — endpoint **`/mcp`** |
| Controller | `innopacks/mcp/src/Http/Controllers/McpController.php` |
| Auth | Laravel Sanctum (`auth:sanctum`), bearer token from `POST /api/panel/login` |
| Origin validation | `innopacks/mcp/src/Http/Middleware/ValidateMcpOrigin.php` (anti DNS-rebinding) |
| Feature toggle | `EnsureMcpEnabled` middleware + `system_setting('mcp_enabled', true)`; **disabled ⇒ 404** |
| Locale | `SetMcpLocale` middleware |
| Tool bridge | `innopacks/mcp/src/Tools/RegistryToolAdapter.php` |
| Welcome page | `GET /mcp` — public, no auth, lists the full tool catalogue |

Per the official docs, core ships **45+ built-in tools** across products, orders, customers, categories, brands, shipping, analytics, content, files, reviews, attributes, localisation and tax. The same tools are exposed in the admin panel's AI chat via `LaravelAiToolAdapter`.

**Client wiring** (from official docs):
```bash
claude mcp add --transport http innoshop https://your-store.com/mcp \
  --header "Authorization: Bearer YOUR_TOKEN"
```

### 5.2 AI core — ✅ built in

`innopacks/aicore/` (namespace `InnoShop\Aicore\`) provides the AI layer: `laravel/ai` ^0.10.1, a `ToolRegistry` singleton, `BaseTool`, and the `ToolInterface` contract, plus per-locale settings and a panel UI.

> ⚠️ **Version drift:** the official MCP/AI docs reference `innopacks/ai/src/...` and namespace `InnoShop\AI\`, but this working copy has `innopacks/aicore/src/...` and namespace `InnoShop\Aicore\`. The docs are ahead of (or behind) this checkout. Verify the exact namespace in the copy before writing tools.

### 5.3 The plugin → MCP/AI tool extension point — ✅ exact

`innopacks/aicore/src/Services/ToolRegistry.php:113`:
```php
fire_hook_filter('ai.tools', $this);
```

Plugin registration (official docs, namespace adjusted to this checkout):
```php
// plugins/RecentOrders/Boot.php
namespace Plugin\RecentOrders;

class Boot
{
    public function init(): void
    {
        add_hook_filter('ai.tools', function ($registry) {
            $registry->register(new Tools\RecentOrdersTool);
            return $registry;
        });
    }
}
```

```php
// plugins/RecentOrders/Tools/RecentOrdersTool.php
namespace Plugin\RecentOrders\Tools;

use InnoShop\Aicore\Tools\BaseTool;   // ← namespace differs from docs (InnoShop\AI\Tools\BaseTool)
use InnoShop\Common\Repositories\OrderRepo;

class RecentOrdersTool extends BaseTool
{
    public function name(): string { return 'recent_orders'; }
    public function description(): string { return 'Get the most recent orders with optional status filter.'; }
    public function inputSchema(): array { /* JSON Schema */ }
    public function requiredPermission(): ?string { return 'orders_index'; }
    public function execute(array $arguments): mixed { /* thin adapter over the repo */ }
}
```

Key properties (per docs): the hook fires **lazily** on first registry read, so plugin boot order never matters; tools appear automatically in `tools/list`, `tools/call`, the `/mcp` welcome page, **and** the panel AI chat; permission slugs are enforced per-tool (`requiredPermission()` → `$user->can($permission)`).

⚠️ The docs use both `listen_hook_filter` and `add_hook_filter` — they are aliases of the same registration; use the form the local code already uses.

### 5.4 Plugin system — ✅ mature

- Discovery: auto-scan of `plugins/`, each with `config.json` (not `composer.json`)
- Namespace: `Plugin\` → `plugins/` (root `composer.json:63`)
- Required files: `Boot.php`, `config.json`, `Lang/en/common.php`, `Lang/zh-cn/common.php`
- PascalCase dirs: `Routes/`, `Migrations/`, `Lang/`, `Views/`
- Types: `feature`, `marketing`, `payment`, `shipping`, `fee`, `social`, `language`, `translator`, `intelli`
- Enabling/disabling via the admin panel; per-plugin settings via `fields.php` + `plugin_setting()`
- Plugins can ship PHPUnit tests (`plugins/Stripe/Tests/…`)

### 5.5 Theme system — ✅ capable, ❌ nothing to start from

- `themes/{code}/` with `config.json`, `views/`, `assets/scss`, `assets/js`, `boot.php`, `demo/Seeder.php`
- SCSS layering: `base/` → `layout/` → `components/` → `modules/` → `pages/`, with the `@front` alias
- Two integration strategies: **override** (import defaults, then override) or **replace** (own implementation)
- `theme_asset()`, `theme_image()`, `inno_view()`; build via `THEME=<code> npm run build`
- Theme boot: `boot.php` returning a callable; helpers must be theme-prefixed and `function_exists`-guarded (they live in the global namespace)

**❗ As noted in §1.4, `themes/` is empty in this checkout and in upstream.** A `wondersoul` theme must be authored from scratch — there is no upstream theme to fork.

### 5.6 Goal 4 verdict

**The strongest fit.** MCP, AI, the hook system, the plugin system and the theme system all ship working, with 45+ tools and full docs. Remaining work is *authoring*: a `wondersoul` theme, a Midtrans plugin, a MetaPixel plugin, and a small set of brand-specific MCP tools (e.g. `drop_release_list`, `stock_by_colorway`, `best_sellers`).

---

## 6. Consolidated gap table

| Requirement | Status | Gap | Est. effort |
|---|---|---|---|
| InnoShop baseline | ✅ | — | — |
| Indonesian locale | ✅ | config only | trivial |
| Plugin mechanism | ✅ | — | — |
| Theme mechanism | ✅ | — | — |
| MCP server + 45 tools | ✅ | enable + token | trivial |
| MCP custom tools | ✅ | author brand tools | 1–2 days |
| **Midtrans payment** | ❌ | build `plugins/Midtrans` | 2–3 days |
| **COD payment** | ❌ | build `plugins/Cod` | 1–2 days |
| **Meta Pixel base + events** | ❌ | build `plugins/MetaPixel` | 2–3 days |
| **CAPI server events + dedup** | ❌ | same plugin + hash helper | 1–2 days |
| **Vercel container runtime** | ❌ | ❌ **Dropped** with the Vercel target | — |
| **Vercel-safe install/seed** | ❌ | ❌ **Dropped** — the browser installer works on a real server | — |
| **S3/R2 media disk** | ⚠️ | ⚠️ **Simplified** — media on local disk; only an offsite *backup* bucket needed | 0.5 day |
| **Worker replacement** | ⚠️ | ✅ **Not needed** — Horizon + Reverb + scheduler run under Supervisor, exactly as InnoShop documents | 1 day |
| **Managed MySQL** | ⚠️ | ✅ **Not needed** — MySQL 8 on the same VPS | 1 day + ongoing cost |
| **`themes/wondersoul`** | ❌ | author from scratch | **2–4 weeks** |
| **Brand content & catalog** | ❌ | owner data (88 IG posts) | blocked on owner |

---

## 7. Risks & findings requiring the owner's decision

1. **✅ RESOLVED — Vercel + InnoShop was an architectural mismatch, and the deployment target is now a conventional production server.** The analysis stands: no official PHP runtime; InnoShop wants Horizon + Reverb + Supervisor; the installer writes to disk. On a VPS none of those are problems — the documented topology works unmodified, the installer works, media stays on local disk, and no serverless workaround is needed. The platform-responsibility risks (uptime, backups, TLS, patching) are now **ours**, and are on the risk register in `03-development-goals.md §10`.
2. **🟢 RESOLVED — MySQL vs Postgres.** InnoShop's schema is MySQL-specific, but on a self-hosted VPS the cost difference vanishes, so **MySQL 8 on the same server** — one engine, one set of migrations, one backup procedure, and no porting work.
3. **🟠 The `themes/` directory is empty upstream.** No theme to fork; the `wondersoul` theme is from-scratch work and is the largest single line item in the plan.
4. **🟠 Repo `CLAUDE.md` is stale** (says Laravel Mix / PHP 8.2 / `webpack.mix.js`). It will actively mislead an AI agent or a new developer. It should be rewritten or deleted early.
5. **🟠 `plugins/Stripe/composer.json` still exists**, contradicting the repo's own rule that plugins must not carry `composer.json`. Minor, but it is the reference implementation a developer will copy.
6. **🟡 Meta's and InnoShop's docs have drifted from this checkout** (`InnoShop\AI\` vs `InnoShop\Aicore\`; `innopacks/ai` vs `innopacks/aicore`). Pin the actual APIs from the code, not the docs.
7. **🟡 Meta's developer docs were unreachable from this network.** CAPI endpoint shape and Graph API version must be re-verified before implementation.
8. **🟡 No COD in core.** If Wondersoul sells COD (likely for local fashion), it is a new plugin, not a setting.
9. **🟡 The `media` disk is used but not defined in `config/filesystems.php`.** Verify where it is configured before pointing it at S3.
10. **🟠 No local PHP/Composer toolchain** (neither `php` nor `composer` is on PATH on this machine). Local development, migrations, seeding and tests cannot run until a PHP 8.3 + Composer environment exists. Node 26 / npm 11 are present.

---

## 8. Source-of-truth references

| Topic | URL |
|---|---|
| InnoShop README | <https://github.com/innocommerce/innoshop/blob/main/README.md> |
| InnoShop docs (en) | <https://docs.innoshop.cn/en/> |
| — Architecture | <https://docs.innoshop.cn/en/introduction/architecture.html> |
| — Plugin Development | <https://docs.innoshop.cn/en/development/plugin.html> |
| — Hook System | <https://docs.innoshop.cn/en/development/hook.html> |
| — MCP & AI | <https://docs.innoshop.cn/en/development/mcp.html> |
| — Theme | <https://docs.innoshop.cn/en/development/theme.html> |
| — Theme Boot | <https://docs.innoshop.cn/en/development/theme-boot.html> |
| — Supervisor (processes) | <https://docs.innoshop.cn/en/development/supervisor.html> |
| — Localization | <https://docs.innoshop.cn/en/development/localization.html> |
| — REST API | <https://docs.innoshop.cn/en/restapi/introduction.html> |
| InnoShop releases | <https://github.com/innocommerce/innoshop/releases> (latest **v1.0.1**, 2026-11-18) |
| Midtrans doc index (agent-readable) | <https://docs.midtrans.com/llms.txt> |
| — Snap integration | <https://docs.midtrans.com/docs/snap-snap-integration-guide.md> |
| — Webhook notification | <https://docs.midtrans.com/docs/https-notification-webhooks.md> |
| — Transaction status | <https://docs.midtrans.com/docs/transaction-status-cycle.md> |
| — Sandbox testing | <https://docs.midtrans.com/docs/testing-payment-on-sandbox.md> |
| Vercel runtimes (no PHP) | <https://vercel.com/docs/functions/runtimes> |
| Vercel PHP + Docker guide | <https://vercel.com/kb/guide/deploy-php-on-vercel-with-docker> |
| Archived community PHP runtime | <https://github.com/vercel-community/php> |
| Meta Pixel docs | <https://developers.facebook.com/docs/meta-pixel> (⚠️ unreachable from the build network) |

---

## 9. How this evaluation was produced

- **Source code:** direct reading of `laravel_innoshop` — `composer.json`, `config/`, `public/index.php`, `bootstrap/`, `plugins/`, `innopacks/{common,front,panel,plugin,aicore,mcp,install}/`.
- **Upstream docs:** <https://docs.innoshop.cn> (full sitemap fetched; pages read for architecture, plugins, hooks, MCP/AI, theme, theme-boot, supervisor, localization, REST API).
- **Upstream README:** <https://raw.githubusercontent.com/innocommerce/innoshop/main/README.md> — confirmed **identical** to the local `README.md`, so the local checkout matches upstream `main` for that file.
- **External platform research:** Vercel docs + KB, Midtrans docs (incl. `llms.txt`), GitHub API for `vercel-community/php` and `innocommerce/innoshop` contents/releases.
- **Brand research:** Instagram + Facebook public metadata, YouTube search, GitHub search — see `01-product-research.md`.

**Reproducing the greps:** the audit relied on `grep -rnE` over `*.php` / `*.blade.php` / `*.js` / `*.json` in `innopacks/` and `plugins/`, and `find` for file inventories. Every file path cited above was verified to exist at the stated line.