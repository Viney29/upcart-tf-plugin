# upcart — Theme Factory plugin

A Theme Factory **integration plugin** that teaches the TF pipeline how a store using
**UpCart (AfterSell)** for its cart drawer should be handled — so the agents don't build a
competing native cart, and route cart work to the right layer (UpCart dashboard vs theme).

Knowledge-only (no `tools.ts`, no `storeSettings`), following the same shape as the
`shoplift` reference plugin.

## Design (what's in here and why)

UpCart is different from an SDK-style integration: it **owns the cart UI** and is configured
mostly in its **dashboard**, not in theme code. So the plugin's job is mainly *knowledge* —
tell the agents what UpCart is, that it owns the cart, how to extend it correctly, and **what
to ship**: paste-ready dashboard files under `docs/` + an implementation guide, *not* theme
wiring.

```
upcart/
  plugins.json              # manifest — the `description` is what the LLM selector reads
  knowledge/
    overview.md             # what UpCart is; owns the cart; CRITICAL rules + pointers into the detail files
    output-contract.md      # ★ the deliverable: docs/ paste files + guide; theme-wiring is PROHIBITED
    customization.md        # extension model (Custom CSS, Custom HTML x9, no Liquid) + Public JS API
    gotchas.md              # shadow/light DOM + JS rooting, selector stability, deprecated callbacks, re-render, GWP, pricing, data-URI safety
  USAGE.md                  # human notes: prerequisites + manual dashboard steps
  README.md
```

The `output-contract.md` doc is the key rule: for cart work TF must emit paste-ready files
(Custom CSS field, before-load scripts, one Custom HTML block per named slot) plus an
implementation guide — and must **not** create a theme snippet, a `settings_schema.json`
panel, or a `frontend/` entrypoint for the cart. See `knowledge/output-contract.md` →
"Files to emit" for the reference output shape (the pattern matches the hand-built Llama
Naturals cart PR).

All knowledge is intentionally **general and reusable** — no store-specific IDs, handles, or
thresholds (those are per-store dashboard values).

## Grounded in UpCart docs

- Getting Started with Customizations — https://docs.aftersell.com/upcart/getting_started_with_upcart_customizations
- Custom HTML (locations) — https://intercom.help/as-upcart/en/articles/9455952-custom-html
- Custom CSS — https://intercom.help/as-upcart/en/articles/9455929-custom-css
- Render Cart in Shadow DOM — https://intercom.help/as-upcart/en/articles/12658102-render-cart-in-shadow-dom-setting
- Public API — https://aftersell.notion.site/Upcart-Public-API-7a0f6d75cb044871bdb6da5d99cfc755

## Develop & test

```bash
tf run <task.md> --with-plugin /Users/b-mac/www/Tf-plugins/upcart-tf-plugin
# confirm: "[plugins] Engaged: upcart" in the output / logs/<runId>.log
```

## Install permanently

```bash
# from the parent dir, zip the plugin folder then install
cd /Users/b-mac/www/Tf-plugins && zip -r upcart.zip upcart-tf-plugin -x '*/.git/*' -x '*.DS_Store'
tf plugin install ./upcart.zip          # add --force to overwrite an existing install
# installs to ~/.theme-factory/plugins/upcart/ (named by manifest `name`, not the zip)
```

## Scope

Cart drawer / mini-cart / slide cart, in-cart upsells, rewards/free-shipping meter, in-cart
subscription selectors, in-cart free gift. **Not** Shopify checkout, and **not** the `/cart`
page template unless UpCart is disabled.
