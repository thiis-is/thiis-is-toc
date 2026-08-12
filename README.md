# thiis-is-toc

Source of truth for the **thiis.is** legal documents — Terms of Service, Privacy Policy, and Cookie Policy.

The content lives here as Markdown so it can be reviewed and versioned on its own, and is published to the landing app (`thiis-is-ui`) automatically on merge to `master`.

## Layout

```
content/
  terms.md      # Terms of Service
  privacy.md    # Privacy Policy
  cookies.md    # Cookie Policy
meta.json       # version, effective date, document index, and company/entity details
.github/workflows/publish.yml   # on master → notify hook (placeholder) + trigger UI update
```

## Entity details / placeholders

The Markdown uses `{{company.*}}` tokens (e.g. `{{company.name}}`, `{{company.address}}`). Their values live once in `meta.json → company` and are substituted when the UI renders each document. Fill the bracketed placeholders (`address`, `regNo`) before publishing.

> ⚠️ These are drafted templates, **not legal advice**. Have a qualified lawyer / DPO review them before they go public.

## Versioning

Bump `meta.json.version` (and `effectiveDate`) whenever the Terms/Privacy change **materially**. This must match the backend `CurrentTermsVersion` so the dashboard can prompt users to re-accept, and it gates the customer-notification hook.

## How it reaches the UI

This repo is **public**. The landing app (`thiis-is-ui`) pulls the content at **build time** — its
`scripts/fetch-toc.mjs` fetches `content/*.md` + `meta.json` from this repo's `master`, fills the
`{{company.*}}` tokens, and writes a committed `src/legal/toc.json` snapshot that the legal pages
render. So **changes here land in the UI on its next deploy** (i.e. the next merge to `thiis-is-ui`
`master`). There is no cross-repo trigger or token to manage.

On every push to `master`, `.github/workflows/publish.yml` runs one step: a **placeholder
customer-notification hook** — a no-op today, the intended home for the email-on-change notification
(issue **#1**).
