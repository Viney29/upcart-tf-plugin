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
2. **Theme (tiny surface).** The theme only holds the UpCart **app embed** (installed by the
   app) and, at most, a **legacy, optional** cart-CSS hook file *if the store already keeps
   one* — the UpCart **Custom CSS field remains canonical** (see `output-contract.md`). TF does
   **not** add theme JS for the cart. Everything the drawer shows is UpCart's.

UpCart renders in the **light DOM** by default ("Render Cart in Shadow DOM" is OFF) — see
`gotchas.md` for the Shadow-DOM case and how to root cart JS.

## CRITICAL — what to do when a task touches the cart

- **Do NOT build a native cart.** If the store uses UpCart, there is already a cart drawer.
  Never scaffold a theme mini-cart / slide cart / cart-drawer section, and never restyle an
  existing theme cart, when the intent is the shopper-facing cart — that produces two carts
  or a dead one. Extend UpCart instead. If a task says "cart drawer / mini-cart / slide
  cart," assume UpCart owns it.
- **Do NOT duplicate UpCart features.** UpCart already provides the free-shipping/rewards
  meter, in-cart upsells ("Frequently Bought Together"), and subscription selectors. Don't
  add a second one in the theme; configure or restyle UpCart's.
- **Ship cart changes as dashboard-paste files, never as theme wiring.** The deliverable is
  paste-ready files under `docs/` plus an implementation guide, and the paste itself is a
  **manual dashboard step**. **`output-contract.md` is the authoritative contract** — the
  exact file set and naming, the prohibited theme surfaces, hardcoded config values, and
  the only theme-side exceptions (e.g. an add-to-cart trigger that opens the UpCart
  drawer). Read it before writing any cart file.

## Where the rules live

- **`output-contract.md`** — the deliverable: `docs/` paste files + implementation guide;
  the prohibited theme surfaces and hard STOP self-check; hardcode-don't-build-settings;
  the two theme-side exceptions.
- **`customization.md`** — UpCart's extension points (Custom CSS, the 9 Custom HTML
  locations incl. Scripts (Before Load), the Public JS API essentials) and the
  task → extension-point map.
- **`gotchas.md`** — Shadow vs light DOM and JS rooting, selector stability and the V1→V2
  break, deprecated `upcartOn*` callbacks vs `upcartSubscribe*`, re-injecting DOM after
  re-renders, data-URI SVG audits, cart line keys, GWP coexistence, selling-plan pricing,
  meter-is-visual-only.
