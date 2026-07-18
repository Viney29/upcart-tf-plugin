# Contributing

This is a **knowledge-only** Theme Factory (TF) integration plugin — no `tools.ts`, no
`storeSettings`. It teaches TF how to handle a store whose cart is **UpCart (AfterSell)**.
Changes are almost always edits to the Markdown under `knowledge/` plus the manifest.

## The core invariant (please don't break it)

For any cart task, the deliverable TF must produce is **dashboard-paste files under `docs/`
plus an implementation guide — never theme-wiring**. This is spelled out in
[`knowledge/output-contract.md`](knowledge/output-contract.md). Any change that reopens a
theme-side escape hatch (a `snippets/` / `sections/` / `blocks/` file, a `settings_schema.json`
panel, a `frontend/` entrypoint for the cart, etc.) defeats the plugin's whole purpose and will
be rejected.

Two more rules:

- Keep knowledge **general and reusable** — no store-specific IDs, handles, thresholds, or hex.
- Ground new factual claims in UpCart's official docs (linked in [`README.md`](README.md)).

## Repo layout

- `plugins.json` — manifest. The `description` is what the LLM selector reads; bump `version`
  (semver) on every substantive change.
- `knowledge/` — `overview.md`, `output-contract.md`, `customization.md`, `gotchas.md` (the
  payload TF actually consumes).
- `README.md` / `USAGE.md` — human docs.

## Develop & test

```bash
tf run <task.md> --with-plugin /path/to/upcart-tf-plugin
# confirm "[plugins] Engaged: upcart" in the output / logs/<runId>.log
```

Before committing: make sure `plugins.json` is valid JSON, and that a representative cart task
still yields `docs/` paste files — **not** theme edits.

## Proposing a change

1. Branch from `main`.
2. Make the edit and bump `plugins.json` `version`.
3. Verify the output contract still holds (see above).
4. Open a PR with a clear before/after and the reasoning.

## Commits

Short, staged, descriptive messages (e.g. `pass-1: …`, `fix feedback: …`, `docs: …`).
