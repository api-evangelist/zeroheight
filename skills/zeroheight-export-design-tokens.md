---
name: zeroheight-export-design-tokens
description: Pull design tokens out of zeroheight into a build pipeline — list token sets over the API, then consume the stable per-set Style Dictionary or W3C DTCG export URL.
api: zeroheight API
generated: '2026-08-28'
method: generated
source: openapi/zeroheight-open-api-v2.yml, conformance/zeroheight-conformance.yml
operations:
  - listTokenSets
---

# Export zeroheight design tokens into code

## The shape of this, before you start

`listTokenSets` — `GET /token_sets` — tells you which token sets exist. It does **not** give you the
tokens. The actual token payload is served from a separate, stable export URL that a human generates
once per token set in the zeroheight UI. There is no API route that enumerates tokens: `GET
/token_sets/{id}/tokens` returns 404.

So the flow is: a person creates the export URL once, and from then on your pipeline just fetches it.

## Getting the export URL (one-time, human step)

In zeroheight: open the token set → **Export** → **Style Dictionary export** → choose **Public URL**
or **Private URL**.

- **Public URL** — anyone with the URL can read the formatted tokens. No headers needed.
- **Private URL** — requires a Client ID and Access Token, shown once at creation. Send them as
  `X-API-CLIENT` and `X-API-KEY`, the same headers as the REST API. Create the token with the
  **Style Dictionary Exports** use case.

The URL is stable. It does not change when tokens are republished, which is what makes it usable as a
pipeline endpoint.

## Formats

- **Style Dictionary (v5)** — pick a target format in the export dropdown. This is the build-system path.
- **W3C DTCG JSON** — Export → **Download JSON**. Use this to move tokens between tools. Token sets
  derived from Figma variables download as a **zip with one file per collection**, not a single file —
  handle both shapes.

## Rules

- **Publish before you expect changes.** Token edits made in zeroheight are not in the export until
  they are published. The exception: token sets synced from Figma variables publish automatically.
- **Composite tokens are not supported** in platform-specific exports. If a set uses them, export
  Style Dictionary format and transform downstream, and say so rather than silently dropping them.
- If the set was imported from a git provider, changes can be pushed back as a pull request from
  inside zeroheight — that is a UI action, not an API one.
- Rate limit is 30 requests per 30 seconds per key. A pipeline should fetch each export URL once per
  build, not per package.
