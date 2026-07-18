# UpCart — overview

## What UpCart is

UpCart (by AfterSell) is a Shopify **slide-out cart drawer** app. When it is installed,
it **replaces the store's cart UI** — the drawer/mini-cart that opens when a shopper adds
an item, plus its rewards bar, upsells, and checkout button. It renders on the storefront
and is configured almost entirely in the **UpCart dashboard**, not in theme code.

## Architecture — where things live

1. **UpCart dashboard (most of it).** Layout, colors, modules, and any bespoke markup are
   set in the app: the **Custom CSS** field and **Custom HTML** modules (see
   `customization.md`). TF **cannot** edit the dashboard — dashboard work is a **manual
   step** for the merchant/developer (list it in the plan, do not try to emit it as theme
   files).
2. **Theme (small surface).** The theme only holds the UpCart **app embed** (installed by
   the app) and, optionally, a store-chosen CSS/JS hook file if the team keeps cart styling
   in the theme instead of the dashboard field. Everything the drawer shows is UpCart's.

By default UpCart renders in the **light DOM** ("Render Cart in Shadow DOM" is OFF), so its
elements are in the normal page and theme CSS can reach them. If a store turns Shadow DOM
ON, the cart is isolated (see `gotchas.md`).

## CRITICAL — what to do when a task touches the cart

- **Do NOT build a native cart.** If the store uses UpCart, there is already a cart drawer.
  Never scaffold a theme mini-cart / slide cart / cart-drawer section, and never restyle an
  existing theme cart, when the intent is the shopper-facing cart — that produces two carts
  or a dead one. Extend UpCart instead.
- **Do NOT duplicate UpCart features.** UpCart already provides the free-shipping/rewards
  meter, in-cart upsells ("Frequently Bought Together"), and subscription selectors. Don't
  add a second one in the theme; configure or restyle UpCart's.
- **Route cart changes to the right layer, and ship them as dashboard-paste files.** Visual /
  markup / behaviour changes belong in UpCart **Custom CSS / Custom HTML**, delivered as
  paste-ready files under `docs/` plus an implementation guide — see **`output-contract.md`**
  for the exact file set, naming, and guide contents. Do **NOT** wire cart UI into the theme:
  no `snippets/upcart-*.liquid` rendered from `layout/theme.liquid`, no
  `config/settings_schema.json` panel, no `frontend/` entrypoint or lib, no compiled
  `assets/sc--*`. Write **theme** code only for the narrow non-cart exceptions in
  `output-contract.md` (e.g. an add-to-cart trigger that must open the UpCart drawer).
- **Use UpCart's own selectors, verified live.** Target the documented `.upcart-*` classes;
  confirm against the live drawer before relying on any class (see `gotchas.md`).

## What the agent MUST do

1. Detect that the cart is UpCart before touching anything cart-related; if a task says
   "cart drawer / mini-cart / slide cart," assume UpCart owns it.
2. Deliver cart customizations as **paste-ready `docs/` files + an implementation guide**, one
   artifact per UpCart dashboard field / Custom HTML slot, and **flag the paste itself as a
   manual dashboard step**. Follow **`output-contract.md`** exactly.
3. Root any cart-querying JS at `window.upcartDocumentOrShadowRoot || document`, and target
   documented `.upcart-*` / confirmed V1 `styles_*__` classes — **verified against the live
   drawer**, not guessed with `[class*="…"]` fallback chains.
4. React to cart changes with the `upcartSubscribe*` API (or the
   `aftersell-upcart:public-events:*` events), and re-apply injected DOM on each cart event
   (UpCart rebuilds the line DOM on every change).

## What the agent MUST NOT do

- Build or restyle a competing native theme cart / mini-cart / slide cart.
- **Wire UpCart cart customizations into the theme** — a `snippets/upcart-*.liquid` rendered
  from `layout/theme.liquid`, a `config/settings_schema.json` panel for cart config, a
  `frontend/` entrypoint/lib + import-map entry, compiled `assets/sc--*` bundles, or dumped
  Figma SVGs in `assets/`. That is the single biggest failure mode: cart config goes to the
  **dashboard** as `docs/` paste files (see `output-contract.md`), not into the theme bundle.
- Assume Liquid works inside UpCart Custom HTML (it does not — fetch `/cart.js` instead).
- Use the deprecated single-assignment `window.upcartOnCartLoaded` / `upcartOnCartUpdated`
  callbacks (they clobber each other and other apps — use `upcartSubscribe*`).
- Hard-depend on unstable `styles_*__` build classes when a `.upcart-*` hook exists, or guess
  selectors with `[class*="…"]` chains instead of verifying against the live drawer.
