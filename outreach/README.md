# Outreach: article-ready drafts (not platform publications)

These Markdown files are publication drafts for **independent technical audiences**. They are included in the public GitHub repository for transparent review and easy copying; **their presence here does not mean they have been posted to Habr or DEV**.

| Destination | Draft | Language | Status |
| --- | --- | --- | --- |
| Habr | [habr-ru.md](habr-ru.md) | Russian | Ready for author review; **not submitted** |
| DEV Community | [dev-community-en.md](dev-community-en.md) | English | Ready for author review; `published: false`; **not submitted** |

## Before submitting

1. **Operator fact-check:** confirm the article still accurately states the October 10, 2026 iPhone/mobile observation and any additional longer-term evidence. In the absence of more data, leave long-term stability as **unconfirmed**.
2. **Privacy/security:** double-check that the public-facing copies include no SSH credentials, internal addresses, server file paths, private issue links, raw logs, customer information or secret values. Our code examples use the placeholder `example.com` and leave certificate paths unspecified.
3. **Technical honesty:** do not say HTTP/1.1, Safari, the carrier, or DPI is *proven defective*. The protocol change coincided with a graceful Nginx reload, so causality isn't completely isolated.
4. **Editorial purpose:** teach a diagnostic method, not promote a commercial website, build SEO backlinks, or mass-syndicate near-duplicates. Keep the link to the open technical case study relevant and minimal.
5. **AI disclosure:** leave the assistance statement visible. DEV's [AI-assisted article guidelines](https://dev.to/guidelines-for-ai-assisted-articles-on-dev/) specifically require disclosure; authors should understand and stand behind the claims before publishing.

## Habr

- Open [Habr](https://habr.com/ru/) while signed into an account with article publishing privileges; select **«Написать» → «Статья»**.
- Use the heading from `habr-ru.md` as the platform title and paste the remaining Markdown as the body (avoid duplicating the heading).
- Select **relevant, existing** topic hubs (1–5); add meaningful tags, run the preview, and ensure Markdown/code fences render correctly.
- The article is an original incident narrative, **not a translation** of somebody else's work.
- Habr has account/karma restrictions for publishing articles. If the account is not eligible, do not work around platform restrictions.
- Source guidance: [Habr — publications](https://habr.com/ru/docs/help/publications/) and [How to write a case study](https://habr.com/ru/docs/authors/case/).

## DEV Community

- Open [Write a Post](https://dev.to/new) when signed in.
- Use `dev-community-en.md`: it contains **DEV Basic Markdown front matter** with title, description, four tags, and `published: false`. Paste it into the *Basic Markdown* editor or transfer title/tags separately if using the Rich + Markdown editor.
- Preview and personally verify the text. Only when ready, change to `published: true` and save or click the platform **Publish** control.
- The DEV article is separately adapted for English-language readers rather than a byte-for-byte republication of GitHub README or Habr. Therefore **no `canonical_url` is set**. If later cross-posting a *duplicate* of an already published article, set its actual canonical URL using DEV's editor.
- Don't use DEV posts to promote a product/business or for SEO link building; the article's only off-site project link leads to its technical documentation.
- Source guidance: [DEV writing and publishing](https://dev.to/help/writing-editing-scheduling), [editor guide](https://dev.to/p/editor_guide/), [AI-assisted article guidelines](https://dev.to/guidelines-for-ai-assisted-articles-on-dev/).

## After publishing

Replace the table's **not submitted** status only after independently confirming the live article URLs (do not invent them). Update this file with those URLs and publication dates through a documentation PR. Respond to technical corrections and update the case if longer observation disproves the initial result.

**Scope:** Public documentation only. Nothing here changes Nginx, production DNS, web app code, certificates, or internal issue tracker permissions.
