# UpCart — customization model & Public API

Everything below is configured in the **UpCart dashboard**. TF's deliverable is the
**paste-ready content for each field, as `docs/` files, plus an implementation guide** — see
**`output-contract.md`** for the required file set, block naming, and the theme-wiring
prohibitions. Name the exact field / Custom HTML location for every artifact you produce.

## Extension points

### Custom CSS
- **Where:** UpCart → Settings → **Custom CSS** (bottom of the Settings tab).
- **Use for:** restyling native cart elements. Target documented `.upcart-*` classes, e.g.
  `.upcart-header-text`, `.upcart-product-title`, `.upcart-item-price`,
  `.upcart-checkout-button`, `.upcart-upsells-title`, `.upcart-upsell-item-card`.
- Prefer `.upcart-*` hooks. V1 builds also expose hashed `styles_*__` build classes; those
  change between UpCart versions — only use them when no `.upcart-*` hook exists, and note
  the fragility.

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

- **Root every cart query at the drawer root**, so it works whether Shadow DOM is on or off:
  ```js
  var root = window.upcartDocumentOrShadowRoot || document;
  root.querySelector('.upcart-product-item');
  ```
- **React to cart changes with the `upcartSubscribe*` family** (modern; multiple listeners
  are safe): e.g. `window.upcartSubscribeCartLoaded(fn)`, `upcartSubscribeCartUpdated(fn)`,
  `upcartSubscribeItemRemoved(fn)`, `upcartSubscribeUpsellsAddedToCart(fn)`. Check the API
  reference for the exact set. **Do not** use the deprecated single-assignment
  `window.upcartOnCartLoaded` / `upcartOnCartUpdated` (last writer wins — they overwrite
  each other and other apps' handlers).
- **Re-injection:** UpCart re-renders the line DOM on every change, so any DOM you inject
  must be re-applied on each cart event (subscribe callback) and, defensively, via a
  `MutationObserver` on the drawer root.
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
