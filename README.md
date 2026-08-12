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

On every push to `master`, `.github/workflows/publish.yml`:

1. **Notify customers (placeholder)** — a no-op hook where the email-on-change notification will be wired. See issue **#1**.
2. **Trigger UI update** — sends a `repository_dispatch` (`event_type: toc-updated`) to `thiis-is/thiis-is-ui`, whose deploy workflow pulls this repo's content, renders the pages, and redeploys.

### Prerequisite

A repo secret **`UI_DISPATCH_TOKEN`** — a GitHub token (fine-grained PAT or App installation token) with **Actions: write** (and Contents: read) on `thiis-is/thiis-is-ui` — so the dispatch can trigger the UI build.
