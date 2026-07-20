# UpCart — customization model & Public API

Everything below is configured in the **UpCart dashboard**. Deliverables follow
**`output-contract.md`** (paste-ready `docs/` files + implementation guide, no theme wiring).
Name the exact field / Custom HTML location for every artifact you produce.

## Extension points

### Custom CSS
- **Where:** UpCart → Settings → **Custom CSS** (bottom of the Settings tab).
- **Use for:** restyling native cart elements. Target documented `.upcart-*` classes, e.g.
  `.upcart-header-text`, `.upcart-product-title`, `.upcart-item-price`,
  `.upcart-checkout-button`, `.upcart-upsells-title`, `.upcart-upsell-item-card`.
- Prefer `.upcart-*` hooks over V1's hashed `styles_*__` build classes, and verify every
  selector against the live drawer — see `gotchas.md` ("Selectors change between versions").

### Custom HTML modules
- **Where:** UpCart → Cart Editor / Settings → **Custom HTML**. Supports `<style>` and
  `<script>`; **no Liquid** — read data with `/cart.js` and `/products/{handle}.js`, mutate
  with `POST /cart/change.js`.
- **Placement locations** (choose per feature):
  1. Above announcements/rewards
  2. Below header/announcements/rewards
  3. Between each line item
  4. Above footer/add-ons
  5. Above checkout button
  6. Below checkout button
  7. Bottom of cart
  8. On empty cart screen
  9. **Scripts (Before Load)** — invisible; JS that must run **before** the cart initializes
     (e.g. registering `upcartModifyListOfUpsells` or subscribe callbacks).
- Global scripts (event listeners) work from any location; visual blocks must go in the slot
  where they should appear.

## Public JS API (the essentials)

Full reference: UpCart Public API — https://aftersell.notion.site/Upcart-Public-API-7a0f6d75cb044871bdb6da5d99cfc755

- **Root every cart query** at `window.upcartDocumentOrShadowRoot || document` so it works
  whether Shadow DOM is on or off — see `gotchas.md`.
- **React to cart changes with the `upcartSubscribe*` family** (multiple listeners are
  safe): e.g. `window.upcartSubscribeCartLoaded(fn)`, `upcartSubscribeCartUpdated(fn)`,
  `upcartSubscribeItemRemoved(fn)`, `upcartSubscribeUpsellsAddedToCart(fn)`. Check the API
  reference for the exact set. Equivalent DOM events also fire as
  `aftersell-upcart:public-events:*` if `addEventListener` fits better. Never the deprecated
  `upcartOn*` callbacks — see `gotchas.md`.
- **Re-injection:** UpCart wipes injected DOM on every re-render — re-apply on each cart
  event; see `gotchas.md` ("DOM re-render wipes injected markup").
- **Force a re-render** after your own `/cart/change.js` mutation: `window.upcartRefreshCart()`.
- **Filter in-cart upsells** ("Frequently Bought Together"):
  ```js
  window.upcartModifyListOfUpsells = (upsells) =>
    upsells.filter((item) => !EXCLUDED_HANDLES.includes(item.handle));
  ```

## Where a change goes — quick map

| Task | Extension point |
|---|---|
| Recolor / restyle native cart elements | Custom CSS |
| Add net-new markup (banner, trust row, note) | Custom HTML at the matching location |
| Setup JS before the cart renders | Custom HTML → Scripts (Before Load) |
| Exclude products from upsells | `upcartModifyListOfUpsells` (Before Load) |
| React to add/remove/update | `upcartSubscribe*` + re-apply injected DOM |
| Change a native module (rewards, upsells, subscriptions, discount input) | that module's settings in the dashboard |
