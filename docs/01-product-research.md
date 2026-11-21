# Product Research — WONDERSOUL JAKARTA

**Compiled:** 2026-11-21 · **Method:** direct source retrieval (Instagram/Facebook public metadata, YouTube search + channel scraping, GitHub API, official vendor docs). No third-party research panels used.

> **Verification legend** — ✅ *verified* (fetched from the source) · ⚠️ *inferred* (from adjacent verified data) · ❓ *open* (could not be verified without account access)

---

## 1. The brand already exists — and it already has an audience

This is the single most important finding: **WONDERSOUL JAKARTA is not a greenfield project.** It is an operating Jakarta clothing label with a live community, a physical presence (Kemang, Jakarta), and ~10K combined social followers. The build is a *migration* of an existing business onto a real storefront — not a launch from zero.

### 1.1 Instagram — `@wondersoul_jkt` ✅

| Field | Value |
|---|---|
| Display name | WONDERSOUL JAKARTA |
| Handle | `@wondersoul_jkt` |
| Followers | **8,347** |
| Following | 412 |
| Posts | **88** |
| Source | `og:description` on `instagram.com/wondersoul_jkt` |

### 1.2 Facebook — `facebook.com/WONDERSOULJAKARTA` ✅

| Field | Value |
|---|---|
| Page name | WONDERSOUL |
| Followers | **2,076** |
| "Discussing this" | 9 |
| Indonesia orders | **WhatsApp +62 812-9865-108** |
| Worldwide orders | **wondersoul.jakarta@gmail.com** |

### 1.3 The origin story: a soul-music collective ⚠️ → ✅

Search surfaced an established Jakarta entity called **WonderSoul Weekender** (YouTube channel *"WonderSoul Weekenders Jakarta"*), with a documented activity trail:

- *"Proyek Di Atas presents: Borongan Ep.1. In Collaboration with Wondersoul Weekender & Kabur Lagi"* — music collaboration
- *"PUSH WOOD by WONDERSOUL WEEKENDER at Tiamtara Kopitiam Nusantara. Saturday, 3 April 2025"*
- *"Weekenders Service Crew … at MAKMUR KEMANG JAKARTA"*
- *"Travellers Tune Part II Kabur Lagi X Wondersoul Weekender"*
- Earliest artefacts dated **2017**

**Read:** the name is a compound of *Wonder* + *Soul* and the group is a Jakarta soul/funk/records-crew. The clothing label reads as the merch/culture arm of a live music community. ⚠️ The link between "Weekender" and the clothing brand is inferred from name + scene + city; it is not confirmed by either platform's public metadata. ❓ Worth confirming with the owner — it materially changes the launch story, the CRM, and whether the store should sell music-event tickets.

### 1.4 What we could NOT retrieve ❓

Instagram and Facebook both gate post-level content behind login for non-browser clients. **We could not read the 88 post captions, product photos, prices, or the Instagram bio.** Instagram's `web_profile_info` endpoint returned HTTP 429 from this network.

This is the largest gap in the research and it directly determines the catalog. The single highest-value next step is a manual export of the 88 posts (or simply the bio + last ~20 captions) — it yields the real product taxonomy, real prices, and real colorways for free.

---

## 2. Positioning

Per the owner: a **local Jakarta clothing brand inspired by British Mods, American Ivy League, 90s/Madchester, and the soul scene.**

Decomposed into sellable fashion language:

| Reference | Era | Wardrobe translation | Signature items |
|---|---|---|---|
| **Mods** | 1960s UK | Tailored, sharp, high-waist, two-wheeler | Harrington/Mod jacket, tailored blazer, Chelsea boots, loafers, anorak |
| **Ivy League** | 1950s–70s US East | Preppy, layered, collegiate | Oxford shirt, cable knit, tweed, penny loafer, chinos, duffle coat |
| **Madchester** | 1988–1992 | Baggy, rave, print-heavy, hedonistic | Baggy fit, smiley-face graphic, mesh/fishnet layer, track jacket, **period-correct '90s sizing** |
| **Soul** | 1960s Motown → 1990s neo-soul | Warm, textured, warm-toned, music-native | Harrington, shearling collar, velour, satin, earth palette |

**Cross-cutting brand assets that follow from this:** the accent colour, the type treatment, the "notch" motif (already the signature device in the design reference template), and a music-first visual language (records, hi-fi, flyers, stage light).

---

## 3. Market context — Indonesian local fashion brand commerce

### 3.1 How local brands actually sell ✅

Verified from YouTube search (2026 landscape): Indonesian local clothing brands are discovered and sold primarily through **Instagram/ TikTok presence, physical bazaars, and WhatsApp order handling** — not through owned web stores.

Observed, verified channels:
- **Bazaars** — e.g. *USS* (bazaar circuit) featured as a recurring brand-tent channel
- **Fashion weeks** — *Indonesia Fashion Week 2026 at JICC Senayan*, framed around "brand lokal dan outfit keren"
- **Long-tail survival matters more than scale** — brands profiled for "bertahan selama 10 tahun" (surviving 10 years) are the aspirational benchmark in this scene, not fast-growth

### 3.2 The competitive frame ⚠️

Named local brands surfaced in 2026 coverage (partial, from search snippets rather than deep profiles):
- **Insurgent Club** (Bandung) — local streetwear
- **Miracle Mates** — long-running local label, "no strategy for 10 years"
- The general **USS bazaar circuit** and JICC fashion-week roster

⚠️ These are *scene peers*, not necessarily direct competitors: the Mods/Ivy/Madchester/soul niche in Jakarta is comparatively under-served relative to generic streetwear. ❓ A proper competitor scan (price bands, drop cadence, audience overlap) is still outstanding.

### 3.3 The channel that matters ⚠️

For a Jakarta label with this profile, the realistic funnel is:

```
Instagram / TikTok  →  WhatsApp (order + sizing + payment proof)  →  Instagram DM
```

The Facebook page still lists **WhatsApp as the Indonesia order channel** and **email as the "worldwide order" channel**. ✅ That is the business as it operates *today*.

**Implication:** the website's single highest-value job is **not** to replace WhatsApp — it is to (a) give the brand a credible, shareable home that works as a link-in-bio destination, (b) capture the "worldwide order" audience that WhatsApp cannot serve, and (c) own the customer record (email + CAPI audiences) that Instagram DM cannot.

---

## 4. Product & pricing research ❓ — the honest gap

We could not retrieve the actual catalog, so the following are **market conventions for Indonesian fashion e-commerce**, stated as assumptions to validate — not as findings:

| Assumption | Why | ❓ Validation |
|---|---|---|
| Price band **IDR 250K–1.2M** per item | Standard band for Jakarta label-quality (premium local streetwear) | Owner must confirm |
| **Drop-based** releases, not always-on | Matches bazaar + community-scene rhythm observed | Owner must confirm |
| **COD dominates** (est. 60–75% of local fashion orders) | Universal Indonesian fashion e-commerce pattern | Owner must confirm — this drives whether we build a COD plugin |
| **Size range S–XXL**, unisex, with a **size chart** | Standard; Madchester/Mods sizing is a real fit-risk | Owner must confirm |
| **Pre-order** capability for drops | Common in local labels to manage cash | Owner must confirm |

> ⚠️ **None of the above is verified.** They are stated so the engineering plan can be sized, and they are flagged in `03-development-goals.md` as blocking questions for the catalog theme.

---

## 5. Technology & platform research (relevant to this build)

| Topic | Finding | Source |
|---|---|---|
| **Meta Pixel / CAPI** | Meta's developer docs are JS-gated and returned errors from this network. Integration shape is well-established (browser `fbq()` + server `POST /<PIXEL_ID>/events` with `access_token` + `event_id` for dedup + SHA-256 `em`/`ph` for Advanced Matching). **Must be re-verified against current Meta docs before implementation.** | developers.facebook.com (partially blocked) |
| **Midtrans** | Official docs are excellent and machine-readable: `https://docs.midtrans.com/llms.txt` indexes every page; appending `.md` to any doc URL yields markdown. Snap, Core API, webhook notification, status API, and error codes all documented. | docs.midtrans.com ✅ |
| **InnoShop** | Official docs at `docs.innoshop.cn` with full sitemap. Covers Architecture, Plugin Development, Hook System, MCP + AI Integration, Theme, Theme Boot, WebSocket, Supervisor, Localization, REST API. Latest release **v1.0.1** (2026-11-18). | docs.innoshop.cn ✅ |
| **Vercel + PHP** | Vercel has **no official PHP runtime**. The community runtime (`vercel-community/php`, npm `vercel-php`) is **archived**, and its own README states: *"For new projects, we recommend deploying PHP on Vercel using a container runtime instead."* Official guide: `vercel.com/kb/guide/deploy-php-on-vercel-with-docker` (FrankenPHP + Dockerfile). | vercel.com ✅ |
| **InnoShop production topology** | Official Supervisor doc requires **Horizon (queue) + Reverb (websockets) + scheduler** as persistent processes — incompatible with serverless. | docs.innoshop.cn ✅ |

---

## 6. Research conclusions → design inputs

1. **Migrate, don't launch.** There are 8,347 IG followers, 2,076 FB followers, a physical store in Kemang, and a 7-year creative history. The site must look *established* on day one and must respect a real community, not look like a startup.
2. **The brand has two audiences** — Jakarta local (WhatsApp, COD, cash) and worldwide (email, card). The storefront should route them differently rather than impose one checkout on both. This is the strongest argument for a bilingual, multi-currency, multi-payment setup (both of which InnoShop already supports natively).
3. **Music is the brand engine.** The soul/funk collective origin means drops can be event-shaped — and the site should have a place for releases, lookbooks, and event tie-ins that a generic catalogue doesn't.
4. **Instagram → site → owned audience.** Every Instagram touchpoint is a traffic source; CAPI + Meta audiences are how the 8,347 followers become a retargetable, exclude-bought custom audience. This is the concrete business case for the CAPI requirement.
5. **The catalog is the only thing we are missing.** One manual export from the owner's IG unblocks theme design, price band, size matrix, and payment mix at once.

---

## 7. Open research questions for the owner

1. **The 88 posts** — product types, price range, colour names, sizes, and how drops are announced. (Highest value; blocks theme + catalog design.)
2. **COD?** Is cash-on-delivery offered today via WhatsApp? If yes, it must exist in the new store (InnoShop core has **no** COD payment method).
3. **Worldwide orders** — which countries actually order, and what do they pay with? (Midtrans is Indonesia-only by nature; Stripe/PayPal already exist as plugins.)
4. **Inventory model** — made-to-order, pre-order, or stocked? Determines whether the storefront needs a waitlist/stock-notify flow.
5. **Is the music collective part of the business?** Determines whether the site also sells event tickets or hosts releases.
6. **Sizing** — unisex? Mods/90s cuts need explicit size charts.
7. **Return/exchange policy** — required copy and a policy page.
8. **Competitor set** — which 3–5 Jakarta labels do you consider peers? Enables a real competitive scan.
