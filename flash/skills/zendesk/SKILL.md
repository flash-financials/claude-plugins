---
name: zendesk
description: Query the Zendesk Sell REST API to explore CRM data — deals, contacts, leads, pipelines, tasks, and more. Use when the user mentions Zendesk Sell, CRM, deals, leads, contacts, sales pipeline, Sell API, or sales data.
---

# Zendesk Sell API

## Context

- GCP auth: !`gcloud auth print-access-token 2>/dev/null | head -c4 && echo "...OK" || echo "NOT AUTHENTICATED"`
- Token accessible: !`gcloud secrets versions access latest --secret=zendesk-sell-token --project=zendesk-integration-sync 2>/dev/null | head -c4 && echo "...OK" || echo "NOT ACCESSIBLE"`

## Connection Details

- **Base URL**: `https://api.getbase.com/v2`
- **Auth**: Bearer token from GCP Secret Manager
- **GCP Project**: `zendesk-integration-sync`
- **Secret Name**: `zendesk-sell-token`

### Token Retrieval

```bash
TOKEN=$(gcloud secrets versions access latest --secret=zendesk-sell-token --project=zendesk-integration-sync)
```

Use this token in all curl commands as: `-H "Authorization: Bearer $TOKEN"`

---

## API Resources

All endpoints are relative to `https://api.getbase.com/v2`. All list endpoints support pagination: `page`, `per_page` (max 100), `sort_by`.

| Resource | Endpoint | Filters |
|----------|----------|---------|
| **Accounts** | `/accounts/self` | *(single resource, no list)* |
| **Associated Contacts** | `/deals/{deal_id}/associated_contacts` | *(nested under deal)* |
| **Calls** | `/calls` | `ids`, `resource_type`, `resource_id` |
| **Call Outcomes** | `/call_outcomes` | *(pagination only)* |
| **Collaborations** | `/collaborations` | `ids`, `resource_type`, `resource_id` |
| **Contacts** | `/contacts` | `ids`, `creator_id`, `owner_id`, `is_organization`, `name`, `first_name`, `last_name`, `email`, `phone`, `mobile`, `address[city]`, `address[postal_code]`, `address[state]`, `address[country]` |
| **Custom Fields** | `/custom_fields` | `resource_type` |
| **Deals** | `/deals` | `ids`, `creator_id`, `owner_id`, `contact_id`, `organization_id`, `stage_id`, `source_id`, `hot`, `name` |
| **Deal Sources** | `/deal_sources` | `ids`, `name` |
| **Deal Unqualified Reasons** | `/deal_unqualified_reasons` | `ids`, `name` |
| **Documents** | `/documents` | `ids`, `resource_type`, `resource_id` |
| **Leads** | `/leads` | `ids`, `creator_id`, `owner_id`, `source_id`, `first_name`, `last_name`, `organization_name`, `status`, `email`, `phone`, `mobile`, `address[city]`, `address[postal_code]`, `address[state]`, `address[country]` |
| **Lead Conversions** | `/lead_conversions` | `ids`, `lead_id` |
| **Lead Sources** | `/lead_sources` | `ids`, `name` |
| **Lead Unqualified Reasons** | `/lead_unqualified_reasons` | *(pagination only)* |
| **Line Items** | `/orders/{order_id}/line_items` | `ids` *(nested under order)* |
| **Loss Reasons** | `/loss_reasons` | `ids`, `name` |
| **Notes** | `/notes` | `ids`, `creator_id`, `q`, `resource_type`, `resource_id` |
| **Orders** | `/orders` | `ids`, `deal_id` |
| **Pipelines** | `/pipelines` | `ids`, `name`, `disabled` |
| **Products** | `/products` | `ids`, `name`, `sku`, `active` |
| **Sequence Enrollments** | `/sequence_enrollments` | `ids`, `sequence_id`, `resource_type`, `resource_id`, `state` |
| **Sequences** | `/sequences` | `ids`, `name` |
| **Stages** | `/stages` | `ids`, `pipeline_id`, `name`, `active` |
| **Tags** | `/tags` | `ids`, `creator_id`, `name`, `resource_type` |
| **Tasks** | `/tasks` | `ids`, `creator_id`, `owner_id`, `q`, `type`, `resource_type`, `resource_id`, `completed`, `overdue`, `remind` |
| **Text Messages** | `/text_messages` | `ids`, `resource_type`, `resource_id` |
| **Users** | `/users` | `ids`, `name`, `email`, `role`, `role_id`, `status`, `confirmed`, `zendesk_user_ids`, `identity_type` |
| **Visit Outcomes** | `/visit_outcomes` | *(pagination only)* |
| **Visits** | `/visits` | `outcome_id`, `creator_id`, `resource_type`, `resource_id`, `rep_location_verification_status` |

---

## Key Data Models

### Contact
`id`, `creator_id`, `owner_id`, `is_organization` (bool), `name`, `first_name`, `last_name`, `title`, `description`, `industry`, `website`, `email`, `phone`, `mobile`, `fax`, `twitter`, `facebook`, `linkedin`, `skype`, `organization_name`, `parent_organization_id`, `customer_status`, `prospect_status`, `address` (object: line1, city, postal_code, state, country), `tags` (array), `custom_fields` (object)

### Deal
`id`, `creator_id`, `owner_id`, `name`, `value` (number), `currency`, `hot` (bool), `stage_id`, `source_id`, `loss_reason_id`, `unqualified_reason_id`, `estimated_close_date`, `contact_id`, `organization_id`, `last_stage_change_at`, `last_activity_at`, `tags` (array), `custom_fields` (object), `customized_win_likelihood`

### Lead
`id`, `creator_id`, `owner_id`, `first_name`, `last_name`, `organization_name`, `title`, `description`, `industry`, `website`, `email`, `phone`, `mobile`, `fax`, `twitter`, `facebook`, `linkedin`, `skype`, `status`, `source_id`, `unqualified_reason_id`, `address` (object), `tags` (array), `custom_fields` (object)

### User
`id`, `name`, `email`, `status`, `invited`, `confirmed`, `phone_number`, `role`, `roles` (array: id, name), `team_name`, `group` (object: id, name), `reports_to`, `timezone`, `identity_type`, `zendesk_user_id`

### Task
`id`, `creator_id`, `owner_id`, `content`, `due_date`, `remind_at`, `completed` (bool), `completed_at`, `overdue` (bool), `resource_type`, `resource_id`

### Note
`id`, `creator_id`, `resource_type`, `resource_id`, `content`, `is_important` (bool), `tags` (array), `type`

### Product
`id`, `name`, `description`, `sku`, `active` (bool), `max_discount`, `max_markup`, `cost`, `cost_currency`, `prices` (array: amount, currency)

### Stage
`id`, `name`, `category`, `active` (bool), `position`, `likelihood`, `pipeline_id`

### Pipeline
`id`, `name`, `disabled` (bool)

---

## Sample curl Commands

### Get account info
```bash
TOKEN=$(gcloud secrets versions access latest --secret=zendesk-sell-token --project=zendesk-integration-sync)
curl -sf -H "Authorization: Bearer $TOKEN" https://api.getbase.com/v2/accounts/self | python3 -m json.tool
```

### List deals (first page)
```bash
curl -sf -H "Authorization: Bearer $TOKEN" "https://api.getbase.com/v2/deals?per_page=25" | python3 -m json.tool
```

### Get a specific deal by ID
```bash
curl -sf -H "Authorization: Bearer $TOKEN" https://api.getbase.com/v2/deals/12345 | python3 -m json.tool
```

### Filter contacts by name
```bash
curl -sf -H "Authorization: Bearer $TOKEN" "https://api.getbase.com/v2/contacts?name=Acme" | python3 -m json.tool
```

### Filter deals by stage
```bash
curl -sf -H "Authorization: Bearer $TOKEN" "https://api.getbase.com/v2/deals?stage_id=42" | python3 -m json.tool
```

### Search notes by text
```bash
curl -sf -H "Authorization: Bearer $TOKEN" "https://api.getbase.com/v2/notes?q=meeting" | python3 -m json.tool
```

### List contacts in a city (bracket notation)
```bash
curl -sf -H "Authorization: Bearer $TOKEN" "https://api.getbase.com/v2/contacts?address%5Bcity%5D=Chicago" | python3 -m json.tool
```

### Paginate through all deals
```bash
curl -sf -H "Authorization: Bearer $TOKEN" "https://api.getbase.com/v2/deals?page=1&per_page=100&sort_by=created_at:desc" | python3 -m json.tool
```

### List stages for a pipeline
```bash
curl -sf -H "Authorization: Bearer $TOKEN" "https://api.getbase.com/v2/stages?pipeline_id=1" | python3 -m json.tool
```

### List associated contacts for a deal (nested resource)
```bash
curl -sf -H "Authorization: Bearer $TOKEN" https://api.getbase.com/v2/deals/12345/associated_contacts | python3 -m json.tool
```

### List line items for an order (nested resource)
```bash
curl -sf -H "Authorization: Bearer $TOKEN" https://api.getbase.com/v2/orders/67890/line_items | python3 -m json.tool
```

### List custom fields for deals
```bash
curl -sf -H "Authorization: Bearer $TOKEN" "https://api.getbase.com/v2/custom_fields?resource_type=deal" | python3 -m json.tool
```

### Get incomplete tasks
```bash
curl -sf -H "Authorization: Bearer $TOKEN" "https://api.getbase.com/v2/tasks?completed=false" | python3 -m json.tool
```

---

## Operations

### `connect` -- Verify API access
Fetch the token and call `/accounts/self` to confirm the connection works.

### `list <resource>` -- List resources
Call the appropriate endpoint. Default `per_page=25`. Use filters from the table above.

### `get <resource> <id>` -- Get a single resource
Call `/<resource>/<id>` to retrieve a specific record.

### `search <resource> <query>` -- Search (where supported)
Notes and tasks support the `q` parameter for text search.

### `count <resource>` -- Count resources
Call the list endpoint with `per_page=1` and read `meta.total_count` from the response.

### `stages` -- List pipeline stages
List all pipelines and their stages to understand the sales funnel.

### `custom-fields <resource_type>` -- List custom field definitions
Call `/custom_fields?resource_type=<type>` where type is `contact`, `deal`, `lead`, etc.

### `paginate <resource>` -- Fetch all pages
Loop through pages using `page=N&per_page=100` until `meta.links.next_page` is null.

---

## Response Format

All API responses wrap data in an `items` array (for lists) or `data` object (for single resources):

```json
// List response
{
  "items": [
    { "data": { "id": 1, "name": "..." } },
    { "data": { "id": 2, "name": "..." } }
  ],
  "meta": {
    "type": "collection",
    "count": 2,
    "total_count": 150,
    "links": {
      "next_page": "https://api.getbase.com/v2/deals?page=2&per_page=25",
      "prev_page": null
    }
  }
}

// Single resource response
{
  "data": { "id": 1, "name": "..." },
  "meta": { "type": "deal" }
}
```

Use `jq '.items[].data'` to unwrap list results. Use `jq '.data'` for single resources.

---

## Notes

- **403 errors** are expected for enterprise-only resources (sequences, sequence_enrollments, visits, visit_outcomes, text_messages, collaborations, documents). Skip gracefully.
- **Nested resources** use parent ID in the URL: `/deals/{id}/associated_contacts`, `/orders/{id}/line_items`
- **Address filters** use bracket notation: `address[city]=Chicago` (URL-encode brackets as `%5B` and `%5D` in curl)
- **Sorting**: use `sort_by=field:asc` or `sort_by=field:desc`
- **Rate limits**: the API may return 429 if you hit rate limits. Back off and retry.
- **Related project**: `/Users/scottwinkler/Desktop/feb-28/zendesk-integration/` -- Go SDK and Cloud Run sync job
