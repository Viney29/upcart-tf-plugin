# UpCart — gotchas

## Shadow DOM vs light DOM
- "Render Cart in Shadow DOM" (Settings → Advanced Settings) is **OFF by default** → the
  cart is in the **light DOM**, theme CSS reaches it, and `document.querySelector` finds
  cart elements.
- If a store turns it **ON**, the cart is isolated: theme CSS no longer styles it (put cart
  CSS in the UpCart **Custom CSS** field) and `document.querySelector` will **not** find
  cart elements.
- **Always** root cart JS at the drawer root so it works in either mode:
  ```js
  var root = window.upcartDocumentOrShadowRoot || document;
  root.querySelector('.upcart-product-item');
  ```
  Only use plain `document` for elements **outside** the cart (theme header, etc.).

## Selectors change between versions
- `.upcart-*` are documented, comparatively stable hooks — prefer them.
- Hashed `styles_*__` build classes (V1) are internal and can change when UpCart updates the
  cart build. Don't hard-depend on them; verify against the live drawer.
- **Don't guess selectors.** Chains like `root.querySelector('[class*="emptyCart"]') || …` are a
  smell — they mean the live DOM wasn't inspected. Confirm the real class against the live V1
  drawer and target that. A guessed selector that silently matches nothing produces a block
  that "runs" but injects nowhere.
- **V1 → V2 is a breaking rename.** V1 uses `styles_*__` + `.upcart-*`; a V2 migration moves to
  `.upcart-public-*` / `data-*`. State the version in the implementation guide and re-audit all
  CSS + block scripts if the store is ever upgraded.

## Data-URI SVGs can smuggle scripts
- Icons pasted into the Custom CSS field or a Custom HTML block as `data:image/svg+xml,…` are
  live markup. A copied blob can carry an inline `<script>` / event handler alongside the path.
  **Audit every `data:` SVG before shipping it** — keep only the `<svg>`/`<path>` shape, strip
  anything executable. (A real UpCart CSS field was found shipping a dropdown-chevron data-URI
  with an embedded geolocation-spoofer script.)

## Deprecated single-assignment callbacks
- `window.upcartOnCartLoaded`, `upcartOnCartUpdated`, `upcartOnItemRemoved`,
  `upcartOnAddUpsell`, `upcartOnAddToCart` are **assignments** — the last script to set one
  wins, silently disabling any earlier handler (yours or another app's). Use the
  `upcartSubscribe*` equivalents instead.

## DOM re-render wipes injected markup
- UpCart rebuilds the line-item DOM on every cart change. Anything you inject (a badge, a
  row, a custom control) disappears on the next render. Re-apply it inside your
  `upcartSubscribe*` callback and guard against double-inject (e.g. a claimed `data-*` flag),
  plus a `MutationObserver` on the drawer root as a fallback for renders not tied to a
  subscribed event.

## Cart line identity
- A line item's DOM element `id` equals its Shopify **cart line key** (`<variantId>:<hash>`),
  which is what `POST /cart/change.js` expects as `id`. You can map a rendered row to its
  cart line without a separate lookup.

## No Liquid in Custom HTML
- Modules render raw HTML/JS/CSS — `{{ ... }}` never interpolates. See `customization.md`.

## Coexistence with free-gift / GWP logic
- Free-gift-with-purchase logic (theme scripts or Shopify Flow) that adds `$0` lines will
  fight the cart if not accounted for: exclude gift/bundle handles from upsells via
  `upcartModifyListOfUpsells`, and **never ADD items** from a cart-load/update handler
  (adding on every load loops). Removing/capping is safe; adding is not.

## Subscription (selling-plan) pricing
- A subscription discount is a selling-plan price adjustment, **not** `compare_at_price` and
  **not** a line discount. If you compute a "was/now" saving, the pre-discount unit price is
  in `selling_plan_allocation.compare_at_price`, not `compare_at_price` / `original_price`.

## The meter is visual only
- UpCart's free-shipping/rewards meter does not grant shipping. A matching real
  free-shipping rate (or automatic discount) must exist in Shopify Admin, or the bar
  promises something checkout won't honor.
