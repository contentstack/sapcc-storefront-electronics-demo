# SAP Composable Storefront (Spartacus) + Contentstack — Electronics Demo

A B2C electronics storefront built with **Angular 21** and **SAP Composable Storefront
(Spartacus `221121.15.1`)**, using the
[`@contentstack/contentstack-spartacus-connector`](https://github.com/contentstack/contentstack-spartacus-connector)
to drive its CMS layer from **Contentstack**.

Commerce data (products, pricing, cart, checkout) comes from a SAP Commerce Cloud (OCC) backend;
page content — the homepage, navigation, footer, and marketing slots — comes from Contentstack,
rendered as a hybrid over the OCC base (unauthored pages/slots fall back to OCC automatically).

> **ℹ️ About the Contentstack stack**
> This demo only renders content that already exists in a Contentstack stack. This repo ships that
> content, so **Step 2 (Seed your Contentstack stack)** below gets you an electronics-demo
> stack in one import command. Follow the steps top to bottom and the app runs.

## Prerequisites

| Requirement | Notes |
| --- | --- |
| Node.js | `^22.22.0` (older 22.x prints non-fatal `EBADENGINE` warnings) |
| Angular CLI 21 | `npx -p @angular/cli@21 ng ...` — no global install required |
| SAP RBSC registry access | Needed to install `@spartacus/*` packages — see below |
| A Contentstack stack | Delivery token + API key (read-only) |

## Step 1 — Install dependencies

`@spartacus/*` packages are served from SAP's RBSC registry, not public npm. Create a `.npmrc` in
the project root:

```ini
@spartacus:registry=https://<YOUR_RBSC_REGISTRY_HOST>/
//<YOUR_RBSC_REGISTRY_HOST>/:_auth=<YOUR_RBSC_BASE64_AUTH>
legacy-peer-deps=true
```

This file is gitignored — it holds an organization credential and must never be committed. Then:

```bash
npm install
```

## Step 2 — Seed your Contentstack stack

The demo content (17 content types, 509 entries, 294 assets, 4 locales) ships in this repo as a
Contentstack CLI export at
[`import-electronics-demo-content/`](import-electronics-demo-content/). Import it into
an **empty stack** whose master locale is **English - United States (`en-us`)**:

```bash
npm install -g @contentstack/cli
csdx config:set:region US              # or EU | AZURE-NA | ... to match your org
csdx auth:login                        # dev machine only — provisioning credential
csdx cm:stacks:import \
  --stack-api-key <YOUR_STACK_API_KEY> \
  --data-dir ./import-electronics-demo-content \
  --yes
```

Then, in the Contentstack UI:

1. **Publish** the imported entries **and assets** to the `development` environment.
2. Create a **Delivery token** for `development`.

Keep your **stack API key** and the **delivery token** — you'll put them in `.env` next. (Full
details are in the pack's [README](import-electronics-demo-content/README.md).)

> Already have the configured demo stack? Skip the import and just use its API key + delivery
> token in Step 3.

## Step 3 — Configure and run

```bash
cp .env.example .env
# fill in the stack API key + delivery token from Step 2:
#   CS_API_KEY=<YOUR_STACK_API_KEY>
#   CS_DELIVERY_TOKEN=<YOUR_DELIVERY_TOKEN>
#   CS_ENVIRONMENT=development
#   CS_REGION=US            # match the region you imported into
npm start
```

Open `http://localhost:4200/` — the storefront now renders the imported content.

`npm start` (and `npm run build`) automatically generate
`src/environments/contentstack.environment.ts` from your `.env` via
`scripts/generate-contentstack-env.js` — that generated file is gitignored too, so nothing
credential-bearing ever needs to live in source control. See `.env.example` for all supported
variables.

On a hosting platform, set the same variables (`CS_API_KEY`, `CS_DELIVERY_TOKEN`, `CS_ENVIRONMENT`,
`CS_REGION`, `CS_LIVE_PREVIEW`) as real environment variables instead of a `.env` file — the
generator script reads `process.env` either way.

## Development server

```bash
ng serve
```

Then open `http://localhost:4200/`.

## Build

```bash
ng build
```

Build artifacts are written to `dist/`.

## Notes on credentials

`CS_API_KEY`/`CS_DELIVERY_TOKEN` are **read-only, environment-scoped Contentstack Delivery API
tokens** — safe to ship in the client bundle (they can only read already-published content, not
write or delete).
