# SAP Composable Storefront (Spartacus) + Contentstack — Electronics Demo

A working SAP Composable Storefront (Spartacus) whose **entire content layer is managed in
Contentstack**. Commerce data — products, pricing, cart, checkout — still comes from the SAP
Commerce Cloud (OCC) backend, but everything an author touches — the homepage, navigation, footer,
and marketing slots — is created and served from Contentstack through the
[`@contentstack/contentstack-spartacus-connector`](https://github.com/contentstack/contentstack-spartacus-connector).

The point of the demo: you can run a complete SAP storefront's content from Contentstack. It renders
as a hybrid over the OCC base, so any page or slot you haven't authored in Contentstack falls back to
SAP automatically — you configure as much or as little of the site as you want.

Built with **Angular 21** and **SAP Composable Storefront (Spartacus `221121.15.1`)**.

## Prerequisites

| Requirement | Notes |
| --- | --- |
| Node.js | `^22.22.0` (older 22.x prints non-fatal `EBADENGINE` warnings) |
| Angular CLI 21 | `npx -p @angular/cli@21 ng ...` — no global install required |
| SAP RBSC registry access | Needed to install `@spartacus/*` packages — see Step 1 |
| A Contentstack stack | An empty stack to import the demo content into |

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

## Step 2 — Import the demo content into your stack

The demo content (17 content types, 509 entries, 294 assets, 4 locales) ships with the repo as a
Contentstack CLI export at
[`import-electronics-demo-content/`](import-electronics-demo-content/). Import it into an **empty
stack** whose master locale is **English - United States (`en-us`)**:

```bash
npm install -g @contentstack/cli
csdx config:set:region US              # or EU | AZURE-NA | ... to match your org
csdx auth:login                        # dev machine only — provisioning credential
csdx cm:stacks:import \
  --stack-api-key <YOUR_STACK_API_KEY> \
  --data-dir ./import-electronics-demo-content \
  --yes
```

The import also publishes the entries to the `development` environment. Then, in the Contentstack UI,
create a **Delivery token** for `development`. Keep your **stack API key** and the **delivery token**
— you'll put them in `.env` next. (Full details are in the pack's
[README](import-electronics-demo-content/README.md).)

## Step 3 — Configure and run

```bash
cp .env.example .env
# fill in the values from Step 2:
#   CS_API_KEY=<YOUR_STACK_API_KEY>
#   CS_DELIVERY_TOKEN=<YOUR_DELIVERY_TOKEN>
#   CS_ENVIRONMENT=development
#   CS_REGION=US            # match the region you imported into
npm start
```

Open `http://localhost:4200/` — the storefront now renders content from your stack.

> **⚠️ Accept the backend's self-signed certificate first**
> Commerce data comes from the SAP Commerce Cloud (OCC) backend configured in
> [`src/app/spartacus/spartacus-configuration.module.ts`](src/app/spartacus/spartacus-configuration.module.ts),
> which serves a **self-signed TLS certificate**. Browsers block it by default, so the storefront
> loads as a **blank page** with a `net::ERR_CERT_AUTHORITY_INVALID` error in the console. Fix it
> once per browser: open the backend URL directly — e.g.
> `https://40.76.109.9:9002/occ/v2/electronics-spa/basesites` — click **Advanced → Proceed anyway**
> to accept the certificate, then reload `http://localhost:4200/`.

`npm start` (and `npm run build`) automatically generate
`src/environments/contentstack.environment.ts` from your `.env` via
`scripts/generate-contentstack-env.js` — that generated file is gitignored too, so nothing
credential-bearing ever needs to live in source control. See `.env.example` for all supported
variables.

On a hosting platform, set the same variables as real environment variables instead of a `.env`
file — the generator script reads `process.env` either way.

## Live Preview (optional)

Contentstack [Live Preview](https://www.contentstack.com/docs/developers/set-up-live-preview) lets
authors see edits render in the storefront in real time. To enable it, add these to your `.env`:

```bash
CS_LIVE_PREVIEW=true
CS_PREVIEW_TOKEN=<YOUR_PREVIEW_TOKEN>        # Stack → Settings → Tokens → create a Preview token
CS_PREVIEW_HOST=rest-preview.contentstack.com  # match your region (see table below)
```

| Region | `CS_PREVIEW_HOST` |
| --- | --- |
| US | `rest-preview.contentstack.com` |
| EU | `eu-rest-preview.contentstack.com` |
| AZURE-NA | `azure-na-rest-preview.contentstack.com` |
| AZURE-EU | `azure-eu-rest-preview.contentstack.com` |

Then restart (`npm start`) and, in the Contentstack UI, go to **Settings → Live Preview**, enable
it, and set the preview URL to `http://localhost:4200`. Open any entry and use the **Live Preview**
pane to see your edits render live.

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
