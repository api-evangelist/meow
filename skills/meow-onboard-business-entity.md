---
name: Onboard a business entity (KYB)
description: Create a business entity, fill KYB business details, add and verify representatives, and submit the application for review.
api: openapi/meow-openapi.yaml
operations:
  - create_business_entity_entities_post
  - update_entity_business_details_entities__entity_id__business_details_patch
  - create_representative_entities__entity_id__representatives_post
  - submit_representative_kyc_data_entities__entity_id__representatives__representative_id__kyc_post
  - submit_entity_application_entities__entity_id__submit_post
  - get_entity_onboarding_entities__entity_id__get
auth: global x-api-key with onboarding:write / entity:create scopes
base_url: https://api.meow.com/v1 (sandbox https://api.sandbox.meow.com/v1)
---

# Onboard a business entity (KYB)

Open a Meow account for a business programmatically. Drive the flow off the
onboarding status object: check `next_step` before submitting.

## Rules
- Needs `entity:create` and `onboarding:write`. Exactly one representative must be
  primary before submission.
- Some documents/forms can only be completed in the web app; the API tells you
  which via due-diligence requirements.
- Submitting returns a hosted `consent_url` for the primary representative to
  accept agreements in-browser.

## Steps
1. **Create the entity** — `create_business_entity_entities_post`
   (`POST /entities`). Send an empty body or seed KYB details. Keep `entity_id`.
2. **Fill business details** — `update_entity_business_details_entities__entity_id__business_details_patch`
   (`PATCH /entities/{entity_id}/business-details`): legal name, incorporation,
   addresses, tax ID, website, industry.
3. **Add a representative** — `create_representative_entities__entity_id__representatives_post`
   (`POST /entities/{entity_id}/representatives`) with `is_primary` for the signer.
4. **Verify KYC** — `submit_representative_kyc_data_entities__entity_id__representatives__representative_id__kyc_post`
   (`POST /entities/{entity_id}/representatives/{representative_id}/kyc`), or mint a
   self-serve verification link instead.
5. **Submit** — check `next_step.can_submit` on the status, then
   `submit_entity_application_entities__entity_id__submit_post`
   (`POST /entities/{entity_id}/submit`), choosing the checking-account product.
6. **Poll status** — `get_entity_onboarding_entities__entity_id__get`
   (`GET /entities/{entity_id}`) for `kyc_status` and the returned `consent_url`.
