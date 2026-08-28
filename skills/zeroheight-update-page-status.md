---
name: zeroheight-update-page-status
description: Move a zeroheight documentation page's status tag (To do, In progress, Ready, Deprecated) from an automated workflow, safely and reversibly — the only write operation the zeroheight REST API exposes.
api: zeroheight API
generated: '2026-08-28'
method: generated
source: openapi/zeroheight-open-api-v2.yml, conventions/zeroheight-conventions.yml, errors/zeroheight-problem-types.yml
operations:
  - listStyleguidePages
  - getPageStatus
  - updatePageStatus
---

# Update a zeroheight page status

Keeps design system documentation status in sync with a project tracker — a page moves to "Ready"
when its ticket closes, back to "In progress" when it reopens.

This is the **only write operation** on the zeroheight REST API. Treat it accordingly.

## Before you write: capture the undo

There is no reversal endpoint, because there does not need to be one — this operation is a state
assignment, and you undo it by calling it again with the old value. But that only works if you kept
the old value.

1. `getPageStatus` — `GET /pages/{page_id}/status`. **Record what it returns.** This is your only undo.
2. `updatePageStatus` — `PATCH /pages/{page_id}/status` with the new status.
3. If the change was wrong, `updatePageStatus` again with the value from step 1.

There is no expiry window on this. The status you saved in step 1 is restorable at any point.

## Finding the page

There is no search on the REST API. Use `listStyleguidePages` —
`GET /styleguides/{styleguide_id}/pages` — and match on the page you want.

Do **not** use the Zapier "Find Page" action as a model for matching: it returns the first page whose
name *contains* the search term, in navigation order, so a search for `button` will happily return
`Split buttons`. Match exactly, or ask the user.

## Authentication

```
X-API-CLIENT: <Client ID>
X-API-KEY:    <Access Token>
Content-Type: application/json
```

The token needs **write** access. A token created with read-only access will 401 on the PATCH even
though the GETs succeeded — if reads work and the write does not, the access level is the reason,
not the credentials.

## Status values

Status tags are per-styleguide and customisable. zeroheight's defaults are `New`, `Ready`,
`In progress`, `To do`, `Deprecated`, `BETA`, `N/A`, but a team can relabel them and there are at
most 7. **Read the styleguide's own tags rather than assuming the defaults**, and never invent a
status string.

## Rules

- Confirm the page and the new status with the user before the first write of a session.
- Retrying a `PATCH` is safe — setting the same status twice leaves the same state. There is no
  idempotency key and none is needed.
- Rate limit is 30 requests per 30 seconds per key; a bulk status sync must throttle itself.
- The request body shape is not published by zeroheight. Confirm it against the live API on a single
  test page before running a batch, and stop the batch on the first unexpected response.
