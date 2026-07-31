---
name: Draft an ACH transfer for approval
description: Pick a funding account and counterparty, create an ACH transfer (which enters a pending-approval state), and track it to settlement.
api: openapi/meow-openapi.yaml
operations:
  - list_accounts_accounts_get
  - list_contacts_contacts_get
  - create_ach_transfer_accounts__account_id__ach_post
  - get_approval_approvals__approval_id__get
  - get_ach_transfer_accounts__account_id__achs__ach_transfer_id__get
auth: x-api-key header with transfers:ach:write; MCP scope meow.transfers
base_url: https://api.meow.com/v1 (sandbox https://api.sandbox.meow.com/v1)
---

# Draft an ACH transfer for approval

Move money by ACH to an external counterparty. Programmatic transfers flow
through the entity's approval policy: the transfer is created in a
**pending-approval** state and no funds move until a human approves it on the
Meow dashboard.

## Rules
- Requires the `transfers:ach:write` scope. Through the MCP server all transfer
  tools are draft-only and require `meow.transfers`.
- **Send an `Idempotency-Key` header** (1-50 chars) on the create call so a retry
  never duplicates the transfer. See conventions/meow-conventions.yml.
- Amounts are strings (e.g. `"1500.00"`).

## Steps
1. **Pick the funding account** — `list_accounts_accounts_get` (`GET /accounts`);
   keep the `account_id`.
2. **Pick the counterparty** — `list_contacts_contacts_get` (`GET /contacts`);
   keep the `contact_id` (or create one first).
3. **Create the transfer** — `create_ach_transfer_accounts__account_id__ach_post`
   (`POST /accounts/{account_id}/ach`) with the counterparty, amount, and an
   `Idempotency-Key`. It returns the transfer and, when approval is required, an
   approval reference.
4. **Check approval** — `get_approval_approvals__approval_id__get`
   (`GET /approvals/{approval_id}`) to see whether the draft has been approved.
5. **Track settlement** — `get_ach_transfer_accounts__account_id__achs__ach_transfer_id__get`
   (`GET /accounts/{account_id}/achs/{ach_transfer_id}`); dispatch on `status`
   (`pending`, `processing`, `sent`, `returned`, `canceled`, `error`). Prefer a
   webhook subscription (see meow-subscribe-webhooks.md) over polling.
