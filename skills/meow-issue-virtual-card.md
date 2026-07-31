---
name: Issue and manage a virtual card
description: Issue a single-merchant virtual card with a spend limit, retrieve its details, freeze or update it, revoke it, and review its transactions.
api: openapi/meow-openapi.yaml
operations:
  - create_card_cards_post
  - get_card_details_cards__card_id__details_get
  - update_card_cards__card_id__patch
  - revoke_card_cards__card_id__revoke_post
  - list_card_transactions_cards_transactions_get
auth: x-api-key header with the cards:write scope (cards:read for reads); MCP scope meow.cards
base_url: https://api.meow.com/v1 (sandbox https://api.sandbox.meow.com/v1)
---

# Issue and manage a virtual card

Use this skill to issue a Meow virtual card scoped to a single merchant and spend
limit, then operate it.

## Rules
- Authenticate every call with `x-api-key`. Add `x-entity-id` when the key spans
  multiple entities. Card issue/update/revoke need the `cards:write` scope.
- Cards are virtual and scoped to a single merchant and spend limit. By default a
  card is `single_use: true` and auto-cancels after the first authorization; set
  `single_use: false` for a multi-use card.
- Never log or display the PAN. Retrieve it only when required via the dedicated
  PAN endpoint.

## Steps
1. **Issue the card** — `create_card_cards_post` (`POST /cards`). Provide the
   merchant scope, spend limit, and `single_use`. Keep the returned `card_id`.
2. **Read operational details** — `get_card_details_cards__card_id__details_get`
   (`GET /cards/{card_id}/details`) for billing/delivery address and linked
   account.
3. **Update / freeze** — `update_card_cards__card_id__patch`
   (`PATCH /cards/{card_id}`): set `status` to freeze/unfreeze, replace
   `spending_controls`, or change allowed merchant categories. Omitted fields are
   left unchanged.
4. **Revoke** — `revoke_card_cards__card_id__revoke_post`
   (`POST /cards/{card_id}/revoke`) to immediately kill a card.
5. **Review spend** — `list_card_transactions_cards_transactions_get`
   (`GET /cards/transactions`), filtering by card, date range, status, or merchant.

## Errors
Handle the `{code, message, debug_message}` envelope. `602` = not authorized,
`703` = invalid input, `404`/`422` at the HTTP layer. See
errors/meow-problem-types.yml.
