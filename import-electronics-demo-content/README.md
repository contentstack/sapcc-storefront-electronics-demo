# Electronics Demo Content

A full **Contentstack CLI (`csdx`) export** of the demo stack that backs this storefront.
Import it into an **empty stack of your own** to get the exact same content this demo renders —
then point the app at your stack and run it. No need to ask the repo owner for stack access.

> This is a complete export data-dir (every module folder is present, as `csdx` import requires).
> It contains **no secrets** — only the source stack's public API key and environment URLs. The
> runtime **delivery token** is never part of an export; you create your own after import (below).

## What's inside

| Module | Count | Notes |
| --- | --- | --- |
| Content types | **17** | Page types (`landing_page`, `product_page`, `category_page`, `content_page`), `global_slots` shell, nav (flat + legacy), and marketing components |
| Entries | **509** | Across `en-us` + localized locales |
| Assets | **294** | 147 files + metadata |
| Locales | **4** | `en-us` (master) + `de-de`, `ja-jp`, `zh-cn` (fall back to `en-us`) |
| Environments | **1** | `development` (recreated on import) |

## Import into your own stack

Create an **empty stack** whose master locale is **English - United States (`en-us`)**, then:

```bash
npm install -g @contentstack/cli
csdx config:set:region US              # or EU | AZURE-NA | ... to match your org
csdx auth:login                        # dev machine only — provisioning credential
csdx cm:stacks:import \
  --stack-api-key <YOUR_STACK_API_KEY> \
  --data-dir ./import-electronics-demo-content \
  --yes
```

One command imports the **17 content types**, **4 locales**, **509 entries**, **294 assets**,
and creates the **`development`** environment.

Then, in the Contentstack UI:
1. **Publish** the entries **and assets** to the `development` environment.
2. Create a **delivery token** for `development` — this is the only runtime credential the
   storefront needs.

## Run the storefront against your stack

Copy `.env.example` to `.env` (gitignored) in the repo root and fill in your values:

```bash
CS_API_KEY=<YOUR_STACK_API_KEY>
CS_DELIVERY_TOKEN=<YOUR_DELIVERY_TOKEN>   # from the step above
CS_ENVIRONMENT=development
CS_REGION=US                             # match the region you imported into
CS_LIVE_PREVIEW=false
```

Then:

```bash
npm install
npm start
```

The app's `prestart` hook generates the Contentstack environment config from `.env`, so the
storefront boots reading content from **your** stack — identical to this demo.

## Notes

- **Branches** are not enabled on the source stack (the export notes this — informational, not an
  error).
- Locales `ja-jp` / `zh-cn` may be intentionally unlocalized for some entries and fall back to
  `en-us` — set `includeFallback: true` on the storefront to exercise fallback everywhere.
