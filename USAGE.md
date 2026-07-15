# UpCart plugin — usage & manual steps

This plugin teaches Theme Factory that the store's cart is **UpCart** — so TF won't build a
competing native cart, and any cart work is routed to the right layer. Most UpCart
customization lives in the **UpCart dashboard**, which TF cannot edit; those are manual steps
for you, listed below (the same split as the Shoplift plugin's "what TF does NOT do").

## 0. Prerequisites (once per store)

- **UpCart app installed** from the Shopify App Store (it adds its own app-embed to the
  theme; the drawer is served by the app).
- **Know your Shadow DOM setting** — Settings → Advanced Settings → "Render Cart in Shadow
  DOM". Default is OFF (light DOM). If ON, cart CSS must live in UpCart's Custom CSS field
  and scripts must root at `window.upcartDocumentOrShadowRoot`.
- **A real free-shipping rate exists in Shopify** if you show a free-shipping meter — the
  meter is visual only.

## 1. What TF does (automatic, theme-side)

| What | How |
|---|---|
| Avoids building a native cart | Recognizes UpCart owns the drawer; won't scaffold/restyle a competing theme cart or mini-cart |
| Doesn't duplicate cart features | Won't add a second rewards bar, in-cart upsell, or subscription selector |
| Theme-side cart hooks (only if asked) | Add-to-cart triggers that open the UpCart drawer; a store's chosen theme-side cart CSS/JS hook file |
| Correct cart JS patterns | Roots queries at `upcartDocumentOrShadowRoot`; uses `upcartSubscribe*`; re-applies injected DOM on re-render |

## 2. What TF does NOT do (manual — in the UpCart dashboard)

| Task | Where |
|---|---|
| Paste Custom CSS (restyle native cart) | UpCart → Settings → Custom CSS |
| Add Custom HTML modules (banners, trust rows, custom controls) | UpCart → Custom HTML (pick a location — see below) |
| Setup-before-load scripts (upsell filters, subscribe callbacks) | UpCart → Custom HTML → Scripts (Before Load) |
| Turn native modules on/off (Rewards, Upsells, Subscription Upgrades, Discount Codes) | UpCart dashboard modules |
| Edit button/label text | UpCart → Settings → Translations |

TF will produce the CSS/HTML/JS **content** when a task calls for it and tell you exactly
which field/location to paste it into — but the paste itself is done in the dashboard.

## 3. Custom HTML locations (reference)

`Above announcements/rewards` · `Below header/announcements/rewards` · `Between each line item`
· `Above footer/add-ons` · `Above checkout button` · `Below checkout button` · `Bottom of cart`
· `On empty cart screen` · `Scripts (Before Load)`

Custom HTML supports `<script>`/`<style>` but **not Liquid** — fetch data via `/cart.js`.

## 4. Confirm the plugin engaged

Run with `--with-plugin` and look for `[plugins] Engaged: upcart` in the run output, plus an
`INTEGRATION PLUGINS` section in `logs/<runId>.log`.
