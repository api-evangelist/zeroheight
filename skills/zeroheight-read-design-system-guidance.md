---
name: zeroheight-read-design-system-guidance
description: Ground UI work in a team's zeroheight design system by finding the right styleguide, walking its navigation and fetching page guidance as Markdown — over the REST API when you hold API keys, or over the zeroheight MCP server when you do not.
api: zeroheight API
generated: '2026-08-28'
method: generated
source: openapi/zeroheight-open-api-v2.yml, mcp/zeroheight-mcp.yml, conventions/zeroheight-conventions.yml
operations:
  - listStyleguides
  - listStyleguideCategories
  - listStyleguidePages
  - getPage
---

# Read design system guidance from zeroheight

Use this before generating or reviewing UI, so what you produce matches the team's documented design
system instead of your priors.

## Pick your surface first

zeroheight has two, and they are not equivalent.

- **MCP** — `https://mcp.zeroheight.com/mcp`, or `npx -y @zeroheight/mcp-server@latest`. Available on
  every zeroheight plan. Has full-text search (`search-pages`) and asset fetch. **Prefer this.**
- **REST** — `https://zeroheight.com/open_api/v2`. Enterprise plan only. No search of any kind.

If an MCP connection is available, use it. Fall back to REST only when you hold a Client ID and
Access Token and no MCP connection.

## Authentication (REST)

Send **both** headers on every request:

```
X-API-CLIENT: <Client ID, prefix zhci_>
X-API-KEY:    <Access Token, prefix zhat_>
Accept:       application/json
```

Missing, malformed or invalid credentials return `401 {"status":"fail","message":"Unauthorized",...}`.
A 401 can also mean the account's plan does not include API access — check the plan before assuming
the key is wrong.

## Steps (REST)

1. **Find the styleguide.** `listStyleguides` — `GET /styleguides`. If more than one comes back, ask
   the user which design system applies rather than guessing.
2. **Get the shape of it.** `listStyleguideCategories` — `GET /styleguides/{styleguide_id}/categories`
   for the grouping layer, and `listStyleguidePages` — `GET /styleguides/{styleguide_id}/pages` for
   the pages. There is no search parameter, so this listing is your only index.
3. **Read the guidance.** `getPage` — `GET /pages/{page_id}?format=markdown`. Markdown is the format
   to ask for; it is what zeroheight's own Postman example uses and what you can reason over.
4. **Cite what you used.** Name the page you took a rule from when you report back.

## Steps (MCP)

1. `list-styleguides` — skip this if the connection is a per-viewer link; it is already scoped.
2. `search-pages` with the component or pattern name, if the team is on Enterprise with AI features.
   Otherwise `list-pages` and pick from the navigation tree.
3. `get-page` for the pages that matter.
4. `get-page-asset` for referenced images. Non-image attachments (PDF, Word, Excel, ZIP) are
   unreliable — if one fails, the tool hands you the source URL instead; pass that on rather than
   retrying.

## Rules

- **Rate limit: 30 requests per 30 seconds per API key.** On `429`, read `X-RateLimit-Reset` (UTC
  epoch seconds) and wait until then. There is no `Retry-After` header to lean on.
- **Do not paginate blindly.** zeroheight documents no pagination parameters. If a listing looks
  truncated, say so rather than inventing a `page` or `limit` parameter.
- **A 404 is ambiguous** — it means either "no such route" or "no such resource". Verify the path
  against the contract before concluding the resource is gone.
- **Every error carries `data.request_id`.** Include it verbatim when you report a failure.
- **Private pages** are readable over REST, over local MCP and over MCP-via-login. They are NOT
  readable over MCP-via-link. If content the user expects is missing, this is the first thing to check.
