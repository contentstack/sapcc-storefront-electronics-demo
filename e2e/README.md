# Contentstack connector sanity checks

Playwright specs that verify the connector is actually wired end-to-end, not just that the
Angular app builds. Run with:

```bash
npm run e2e
```

Requires `.env` filled in (same as `npm start`) and network access to both the OCC backend and
`cdn.contentstack.io`. `playwright.config.ts` sets `ignoreHTTPSErrors: true` for the OCC backend's
self-signed cert.

| Spec | Checks |
| --- | --- |
| `contentstack-connectivity.spec.ts` | The app calls the Delivery API, gets 200s with real entry bodies, and a non-CMS page (cart) doesn't fetch page content it doesn't own. |
| `contentstack-content-source.spec.ts` | A CS-owned slot renders the image Contentstack actually returned (not OCC/placeholder data); a non-CMS page still renders its OCC content. |
| `contentstack-live-preview.spec.ts` | No `data-cslp` tags leak into the DOM with Live Preview off (current default). Set `E2E_LIVE_PREVIEW_EXPECTED=true` to instead assert tags appear and match the active locale. |
| `contentstack-locale.spec.ts` | Switching language re-fetches Contentstack content for the mapped locale and drops the previous locale's assets. |

**Enabling Live Preview:** `scripts/generate-contentstack-env.js` writes `previewToken` (and
`previewHost`) into `contentstack.environment.ts` when `CS_PREVIEW_TOKEN` / `CS_PREVIEW_HOST` are
set, so `CS_LIVE_PREVIEW=true` + `CS_PREVIEW_TOKEN` turns Live Preview on in a dev build
([contentstack-client.service.ts](../../../../contentstack-spartacus-connector/src/client/contentstack-client.service.ts)
requires `livePreview` + `previewToken` + dev mode together). By default the suite asserts the
off-state; set `E2E_LIVE_PREVIEW_EXPECTED=true` when running against a build with Live Preview on to
assert the tags appear instead.
