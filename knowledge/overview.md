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
- **Route cart changes to the right layer.** Visual/behaviour changes belong in the UpCart
  **Custom CSS / Custom HTML** (a manual dashboard step — surface it clearly). Only write
  **theme** code for genuinely theme-side needs (e.g. an add-to-cart trigger that must open
  the UpCart drawer, or a store's chosen theme-side cart CSS hook file).
- **Use UpCart's own selectors, verified live.** Target the documented `.upcart-*` classes;
  confirm against the live drawer before relying on any class (see `gotchas.md`).

## What the agent MUST do

1. Detect that the cart is UpCart before touching anything cart-related; if a task says
   "cart drawer / mini-cart / slide cart," assume UpCart owns it.
2. Put visual/markup cart changes into UpCart Custom CSS / Custom HTML and **flag them as
   manual dashboard steps** in the plan.
3. Root any cart-querying JS at `window.upcartDocumentOrShadowRoot || document`.
4. React to cart changes with the `upcartSubscribe*` API, and re-apply injected DOM on each
   cart event (UpCart rebuilds the line DOM on every change).

## What the agent MUST NOT do

- Build or restyle a competing native theme cart / mini-cart / slide cart.
- Assume Liquid works inside UpCart Custom HTML (it does not — fetch `/cart.js` instead).
- Use the deprecated single-assignment `window.upcartOnCartLoaded` / `upcartOnCartUpdated`
  callbacks (they clobber each other and other apps — use `upcartSubscribe*`).
- Hard-depend on unstable `styles_*__` build classes when a `.upcart-*` hook exists.
