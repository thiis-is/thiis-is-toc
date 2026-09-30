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

Bump `meta.json.version` (and `effectiveDate`) whenever the Terms/Privacy change **materially**. This must match the backend `CurrentTermsVersion`, and it triggers the change-notification email (see below).

There is no re-acceptance step. Users accept the Terms at registration (the API records that version and date once), and continued use after a new version's `effectiveDate` constitutes acceptance. The email only informs.

## How it reaches the UI

This repo is **public**. The landing app (`thiis-is-ui`) pulls the content at **build time** — its
`scripts/fetch-toc.mjs` fetches `content/*.md` + `meta.json` from this repo's `master`, fills the
`{{company.*}}` tokens, and writes a committed `src/legal/toc.json` snapshot that the legal pages
render. So **changes here land in the UI on its next deploy** (i.e. the next merge to `thiis-is-ui`
`master`). There is no cross-repo trigger or token to manage.

## Change-notification email

`.github/workflows/publish.yml` emails users about a new version (issue **#1**). The API does the
sending, from `terms@thiisis.com`, via `POST /internal/notifications/terms-updated`.

To announce a material change:

1. Bump `version` and set `effectiveDate` to a date **after** the email will go out. Users must be
   told before the change takes effect (terms.md §15).
2. Set `"notify": true` and write a one- or two-sentence `summary` of what changed. It goes into the
   email body verbatim. Leave `notify` false for typo-level edits.
3. Deploy the API with the matching `CurrentTermsVersion` **first**. Until then the endpoint returns
   409 and the workflow fails.
4. Merge to `master`. The push run sends up to 200 emails; a daily scheduled run sends the rest.
   It stops early on a Cloudflare rate limit. Reruns never email anyone twice. Use **Run workflow**
   (`workflow_dispatch`) to send or resume by hand.

Users who registered on the new version, and users with an unverified email, are skipped.

Repo secrets: `API_BASE_URL` (e.g. `https://api.thiisis.com`) and `INTERNAL_NOTIFY_SECRET`, which
must match the API's env var of the same name.
