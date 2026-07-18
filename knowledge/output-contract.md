# UpCart — output contract (what TF MUST ship)

This is the most important rule in the plugin. Read it before writing any cart file.

When a task customizes the UpCart cart — styling, banners, the free-shipping/rewards meter,
custom blocks, per-line controls, upsell/gift logic — the deliverable is **paste-ready
dashboard files + an implementation guide**, *not* theme code.

UpCart is configured in its **dashboard**, and TF cannot edit the dashboard. So TF's job is to
produce the exact CSS/HTML/JS the developer pastes into each UpCart field, plus a guide that
says where each piece goes and what to toggle. Ship the content as files so it is reviewable in
the PR and version-controlled — but those files are **source-of-truth copies for pasting**,
not theme assets that get wired into the running theme.

## Files to emit (mirror this naming)

Write these into the repo's docs folder (`docs/`). Emit **one file per block/artifact** — note
that several blocks may target the **same** Custom HTML slot (in the reference cart, Blocks F
and G are separate files both pasted into "Bottom of cart," one after the other):

| File | Pastes into |
|---|---|
| `docs/upcart-custom-css-field.css` | UpCart → Settings → **Custom CSS** (paste the whole file) |
| `docs/upcart-before-load-scripts.html` | UpCart → Settings → **custom code _before cart loads_** |
| `docs/upcart-block-<x>-<name>.html` | UpCart → Custom HTML → **\<named location\>** (one file per block) |
| `docs/upcart-cart-design-implementation.md` | the human guide — not pasted anywhere |

- **Give each block a stable letter + name** (Block B = top panel, Block C = trust row,
  Block E = empty cart, Block F = frequency select, Block G = subtotal "was", …) and name the
  exact slot in its header comment.
- **The first line of every file is a paste header**, in the comment syntax native to that file
  type — `/* … */` for the `.css` field file, `<!-- … -->` for the HTML block / before-load
  files — naming the exact UpCart destination. Examples:
  `<!-- Block C — trust row. Paste at UpCart Custom HTML "Below checkout button". -->` /
  `/* UpCart → Settings → Custom CSS — paste this whole file into that field. */`.
  The developer must never have to guess where a file goes.
- The Custom CSS field file styles UpCart's **native** elements, **plus** any injected-element
  styling that has to interleave with native selectors (e.g. a combinator like
  `.styles_Footer__cartSubtotalValue__ + .ln-subtotal-was`, or overrides that must beat native
  specificity). When a block's styling lives in the CSS field, that block **omits its own
  `<style>`** and says so in a comment. Otherwise, self-contained net-new markup carries its
  own `<style>` inside its block file — say so, so nobody double-pastes it.

## The implementation guide MUST contain

1. **Paste-map table** — every file → its exact UpCart field / Custom HTML location.
2. **Module on/off matrix** — every native UpCart module set ON or OFF, with the reason.
   Leaving a native module ON next to a custom block that reproduces it causes duplicates
   (two free-shipping bars) or stray off-design elements (a discount-code input).
3. **Dashboard settings that aren't code** — translations (button/label text), Design-module
   tokens (colors, corner radius, "strikethrough price"/"subtotal line" toggles), and the
   **real Shopify free-shipping rate** the meter depends on (the meter is visual only).
4. **Preflight facts** — UpCart version (V1/V2), Shadow-DOM on/off (light DOM ⇒ plain
   `document` works), subscription app (Skio/etc.), and any theme-side coupling (e.g. a GWP
   script that adds gift lines).
5. **Per-file description + maintenance/gotchas** — the V1→V2 selector break, the light-DOM
   assumption, and which single-assignment callbacks are already owned (see `gotchas.md`).

## Config values — hardcode, don't build settings

Bake per-store values (free-ship threshold, excluded product handles, brand hex, copy)
**directly into the block scripts / CSS**, and list them in the guide under "confirm / adjust
before publishing." Do **NOT** add theme settings or a `config/settings_schema.json` panel to
configure the cart — that couples cart config to the theme, which is exactly what this plugin
exists to avoid. A single documented constant at the top of a script (e.g.
`var THRESHOLD = 75;`) is the right level of "configurable."

## PROHIBITED for cart UI (this is how the plugin fails)

To customize the UpCart cart, **never** create or edit any of these theme surfaces — each one
wrongly pushes UpCart config into the theme:

- ❌ A theme snippet (`snippets/upcart-*.liquid`) — whether or not it is rendered from `layout/theme.liquid`.
- ❌ A theme **section** (`sections/*.liquid`) or **theme-app-extension block** (`blocks/*`).
  An empty-cart message, a trust row, or a per-line selector is a **Custom HTML block**, not a
  theme section/app-block.
- ❌ `config/settings_schema.json` (a cart settings panel) **or** `config/settings_data.json`
  (its saved values — editing it also risks clobbering unrelated app-embed blocks, as a past
  run did to a fraud-protection embed).
- ❌ A `frontend/entrypoints/*` or `frontend/scripts/lib/*` module (or an import-map entry, or a
  compiled `assets/sc--*` bundle) for the cart.
- ❌ Figma-exported SVGs / logos dumped into `assets/`. Inline icons as `<svg>` or a `data:` URI
  **inside the block that uses them** — they render in the dashboard drawer, not the theme.
- ❌ Liquid inside any block. Dashboard Custom HTML is **not** Liquid; read live data with
  `/cart.js` and `/products/{handle}.js`, mutate with `POST /cart/change.js`.

**Hard STOP self-check:** if you are about to create or edit **any theme file for the cart** —
anything under `snippets/`, `sections/`, `blocks/`, `layout/`, `config/`, `frontend/`, or a
cart `assets/*` — stop. That output belongs in a `docs/` file for the dashboard.

## The only theme-side exceptions

1. **A pre-existing theme cart-CSS file** (e.g.
   `frontend/styles/components/upcart-cart-drawer.css`). The restyle's sole home is
   `docs/upcart-custom-css-field.css`; do **not** create a new `frontend/styles/*` file when
   none exists, and do **not** copy the restyle into a theme file. If such a file already
   exists, it may only receive a "moved to the UpCart Custom CSS field" pointer/comment (or be
   emptied) — the **Custom CSS field file is canonical**; the theme copy is legacy and must not
   fork or duplicate the styling.
2. Genuinely theme-side behavior UpCart cannot own: an **add-to-cart trigger that opens the
   UpCart drawer**, or a theme element **outside** the cart. Never the cart's own UI.
