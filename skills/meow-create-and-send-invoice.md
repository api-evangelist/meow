---
name: Create and send an invoice
description: Set up a customer and product, create an invoice with line items and accepted payment methods, and retrieve it.
api: openapi/meow-openapi.yaml
operations:
  - create_invoicing_customer_billing_customers_post
  - create_product_billing_products_post
  - create_invoice_billing_invoices_post
  - get_invoice_billing_invoices__invoice_id__get
  - list_billing_accounts_billing_accounts_get
auth: x-api-key header with billing:*:write scopes; MCP scope meow.billing
base_url: https://api.meow.com/v1 (sandbox https://api.sandbox.meow.com/v1)
---

# Create and send an invoice

Issue an invoice with the Billing API. Creating an invoice is a request for
payment — it does not move funds and needs no dashboard approval.

## Rules
- Writes require the matching billing scope (`billing:customers:write`,
  `billing:products:write`, `billing:invoices:write`).
- Recurring invoices use RFC 2445 RRULE scheduling.

## Steps
1. **Create the customer** — `create_invoicing_customer_billing_customers_post`
   (`POST /billing/customers`) with name and address. Keep the `customer_id`.
2. **Create products** — `create_product_billing_products_post`
   (`POST /billing/products`) with custom pricing. Keep each `product_id`.
3. **Choose a collection account** — `list_billing_accounts_billing_accounts_get`
   (`GET /billing/accounts`) to see where payments collect.
4. **Create the invoice** — `create_invoice_billing_invoices_post`
   (`POST /billing/invoices`) with line items (referencing products), discounts,
   notes, and accepted payment method types (ACH, wire, international wire, card,
   USDC). Keep the `invoice_id`.
5. **Verify** — `get_invoice_billing_invoices__invoice_id__get`
   (`GET /billing/invoices/{invoice_id}`); lifecycle fields like `sent_at`,
   `paid_at`, `voided_at` track status.
