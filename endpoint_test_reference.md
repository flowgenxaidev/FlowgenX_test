# HelpScout — UI Endpoint Test Reference
> Copy-paste each JSON block directly into the UI Body field.
> All 31 endpoints listed in order.

**Shared constants used throughout:**
| Variable | Value |
|----------|-------|
| Mailbox ID | `362302` |
| Mailbox Name | `FlowGenX AI` |
| Mailbox Email | `help@flowgenx-ai.helpscoutapp.com` |
| User ID | `901126` |
| User Email | `ssaha@flowgenx.ai` |
| Team ID | `901194` |
| Team Name | `flowgen team` |
| Customer ID | `860054407` |
| Customer First Name | `Jane` |
| Conversation ID | `3243095282` |
| Conversation Subject | `FlowGenX UI Test Conversation` |
| Thread ID (note) | `9884686817` |
| Tag: flowgenx-test | ID `16384639` |
| Tag: practice | ID `16383577` |
| Tag: welcome | ID `16383576` |

---

## GROUP 0 — Connection

### 1. `test_connection`
**Test HelpScout connection**
`GET /api/v1/helpscout/test-connection`

No body required. Just click **Run**.

**Expected response:**
```json
{ "success": true, "message": "Connection successful" }
```

---

## GROUP 1 — Identity

### 2. `get_me`
**Get the authenticated user profile**
`GET /api/v1/users/me`

No body required. Just click **Run**.

**Expected response keys:** `id`, `firstName`, `lastName`, `email`, `role`, `timezone`, `createdAt`, `updatedAt`

**Expected response (sample):**
```json
{
  "id": 901126,
  "firstName": "Sumon",
  "lastName": "Saha",
  "email": "ssaha@flowgenx.ai",
  "role": "owner",
  "timezone": "America/New_York",
  "createdAt": "2026-02-21T00:00:00Z",
  "updatedAt": "2026-02-21T00:00:00Z"
}
```

---

## GROUP 2 — Mailboxes

### 3. `list_mailboxes`
**List all mailboxes**
`GET /api/v1/mailboxes`

No body required (all parameters optional).

```json
{
  "page": 1
}
```

**Expected response keys:** `_embedded.mailboxes[]`, `page`

**Expected response (sample):**
```json
{
  "_embedded": {
    "mailboxes": [
      {
        "id": 362302,
        "name": "FlowGenX AI",
        "slug": "flowgenx-ai",
        "email": "help@flowgenx-ai.helpscoutapp.com",
        "createdAt": "2026-02-21T02:48:09Z",
        "updatedAt": "2026-02-21T02:48:09Z"
      }
    ]
  },
  "page": {
    "size": 50,
    "totalElements": 1,
    "totalPages": 1,
    "number": 1
  }
}
```

---

### 4. `get_mailbox`
**Get a mailbox by ID**
`GET /api/v1/mailboxes/{mailboxId}`

```json
{
  "mailbox_id": 362302
}
```

**Expected response keys:** `id`, `name`, `slug`, `email`, `createdAt`, `updatedAt`

**Expected response (sample):**
```json
{
  "id": 362302,
  "name": "FlowGenX AI",
  "slug": "flowgenx-ai",
  "email": "help@flowgenx-ai.helpscoutapp.com",
  "createdAt": "2026-02-21T02:48:09Z",
  "updatedAt": "2026-02-21T02:48:09Z"
}
```

---

## GROUP 3 — Conversations Read

### 5. `list_conversations`
**List conversations**
`GET /api/v1/conversations`

```json
{
  "status": "all",
  "page": 1
}
```

**Expected response keys:** `_embedded.conversations[]`, `page`

**Expected response (sample):**
```json
{
  "_embedded": {
    "conversations": [
      {
        "id": 3243095282,
        "number": 35,
        "subject": "FlowGenX UI Test Conversation",
        "status": "active",
        "type": "email",
        "mailboxId": 362302,
        "createdAt": "2026-02-27T00:00:00Z",
        "updatedAt": "2026-02-27T00:00:00Z",
        "tags": [{ "id": 16384639, "tag": "flowgenx-test" }]
      }
    ]
  },
  "page": {
    "size": 25,
    "totalElements": 1,
    "totalPages": 1,
    "number": 1
  }
}
```

**Optional filters:**
```json
{
  "mailbox": 362302,
  "status": "open",
  "tag": "flowgenx-test",
  "assigned_to": 901126,
  "sort_field": "modifiedAt",
  "sort_order": "desc",
  "page": 1
}
```

---

### 6. `get_conversation`
**Get a conversation by ID**
`GET /api/v1/conversations/{conversationId}`

```json
{
  "conversation_id": 3243095282
}
```

**Optional — embed threads inline:**
```json
{
  "conversation_id": 3243095282,
  "embed": "threads"
}
```

**Expected response keys:** `id`, `number`, `subject`, `status`, `type`, `mailboxId`, `assignee`, `customer`, `tags`, `customFields`, `createdAt`, `updatedAt`

**Expected response (sample):**
```json
{
  "id": 3243095282,
  "number": 35,
  "subject": "FlowGenX UI Test Conversation",
  "status": "active",
  "type": "email",
  "mailboxId": 362302,
  "tags": [{ "id": 16384639, "tag": "flowgenx-test" }],
  "createdAt": "2026-02-27T00:00:00Z",
  "updatedAt": "2026-02-27T00:00:00Z"
}
```

---

## GROUP 4 — Conversations Write

### 7. `create_conversation`
**Create a new conversation**
`POST /api/v1/conversations`

> NOTE: This creates a real conversation in mailbox 362302. Use `imported: true` to suppress email notifications.

```json
{
  "subject": "UI Test — New Support Request",
  "type": "email",
  "mailbox_id": 362302,
  "status": "open",
  "customer": { "email": "test@example.com" },
  "threads": [
    {
      "type": "customer",
      "text": "Hello, I need help with a billing issue on my account.",
      "customer": { "email": "test@example.com" }
    }
  ],
  "tags": ["flowgenx-test"],
  "imported": true
}
```

**Expected response keys:** `_resource_id` (new conversation ID)

**Expected response (sample):**
```json
{ "_resource_id": 3243100000 }
```

Save the `_resource_id` for `update_conversation`, `update_conversation_tags`, `update_conversation_custom_fields`, and `delete_conversation`.

---

### 8. `update_conversation`
**Update a conversation (JSON Patch)**
`PATCH /api/v1/conversations/{conversationId}`

> NOTE: Sends a single JSON Patch operation object — NOT an array. HelpScout accepts exactly one operation per request. To update multiple fields, call once per field.

Change status to `closed`:
```json
{
  "conversation_id": 3243095282,
  "op": "replace",
  "path": "/status",
  "value": "closed"
}
```

Re-open conversation:
```json
{
  "conversation_id": 3243095282,
  "op": "replace",
  "path": "/status",
  "value": "open"
}
```

Assign to user:
```json
{
  "conversation_id": 3243095282,
  "op": "replace",
  "path": "/assignTo",
  "value": 901126
}
```

Change subject:
```json
{
  "conversation_id": 3243095282,
  "op": "replace",
  "path": "/subject",
  "value": "Updated Subject — Billing Issue Resolved"
}
```

**Expected response:** Empty dict `{}` — HTTP 204 No Content on success.

---

### 9. `update_conversation_tags`
**Replace all tags on a conversation**
`PUT /api/v1/conversations/{conversationId}/tags`

> NOTE: Full replacement — tags NOT in this list are removed. Use real tag names (not IDs).

```json
{
  "conversation_id": 3243095282,
  "tags": ["flowgenx-test", "practice"]
}
```

Remove all tags:
```json
{
  "conversation_id": 3243095282,
  "tags": []
}
```

**Expected response:** Empty dict `{}` — HTTP 204 No Content on success.

---

### 10. `update_conversation_custom_fields`
**Replace custom field values on a conversation**
`PUT /api/v1/conversations/{conversationId}/fields`

> NOTE: Full replacement — fields NOT in this list are cleared. Get valid field IDs from `get_mailbox`.
> This account has no custom fields defined, so this call will succeed but have no visible effect.

```json
{
  "conversation_id": 3243095282,
  "fields": []
}
```

With custom fields (if your mailbox has them defined):
```json
{
  "conversation_id": 3243095282,
  "fields": [
    { "id": 12345, "value": "enterprise" }
  ]
}
```

**Expected response:** Empty dict `{}` — HTTP 204 No Content on success.

---

### 11. `delete_conversation`
**Delete a conversation permanently**
`DELETE /api/v1/conversations/{conversationId}`

> WARNING: Permanent and irreversible. Use a conversation created specifically for this cleanup test. Use the `_resource_id` returned by `create_conversation` (step 7), NOT conversation `3243095282`.

```json
{
  "conversation_id": 3243095282
}
```

Replace `3243095282` with the ID returned by `create_conversation` before running this.

**Expected response:** Empty dict `{}` — HTTP 204 No Content on success.

---

## GROUP 5 — Threads

### 12. `list_threads`
**List all threads in a conversation**
`GET /api/v1/conversations/{conversationId}/threads`

```json
{
  "conversation_id": 3243095282,
  "page": 1
}
```

**Expected response keys:** `_embedded.threads[]`, `page`

**Expected response (sample):**
```json
{
  "_embedded": {
    "threads": [
      {
        "id": 9884686817,
        "type": "note",
        "status": "active",
        "body": "Internal note: This is a FlowGenX automated test.",
        "createdAt": "2026-02-27T00:00:00Z"
      }
    ]
  },
  "page": {
    "size": 50,
    "totalElements": 3,
    "totalPages": 1,
    "number": 1
  }
}
```

---

### 13. `create_reply`
**Create a reply thread in a conversation**
`POST /api/v1/conversations/{conversationId}/reply`

> NOTE: Creates a real outbound email reply. Use `imported: true` to suppress sending.

```json
{
  "conversation_id": 3243095282,
  "text": "Thank you for reaching out. Our team is reviewing your request and will follow up shortly.",
  "customer": { "id": 860054407 },
  "imported": true
}
```

**Expected response keys:** `_resource_id` (new thread ID)

**Expected response (sample):**
```json
{ "_resource_id": 9884700000 }
```

---

### 14. `create_note`
**Create an internal note in a conversation**
`POST /api/v1/conversations/{conversationId}/notes`

> NOTE: Internal notes are never sent to customers. Safe to create without `imported`.

```json
{
  "conversation_id": 3243095282,
  "text": "Internal note: Customer confirmed billing issue. Escalating to billing team."
}
```

**Expected response keys:** `_resource_id` (new thread ID)

**Expected response (sample):**
```json
{ "_resource_id": 9884700001 }
```

---

### 15. `update_thread`
**Update a thread (JSON Patch)**
`PATCH /api/v1/conversations/{conversationId}/threads/{threadId}`

> NOTE: Sends a single JSON Patch operation object — NOT an array. Only `replace` is supported. Paths: `/text` or `/hidden`.

Update thread text:
```json
{
  "conversation_id": 3243095282,
  "thread_id": 9884686817,
  "op": "replace",
  "path": "/text",
  "value": "Internal note: Updated — billing issue confirmed and escalated."
}
```

Hide a thread:
```json
{
  "conversation_id": 3243095282,
  "thread_id": 9884686817,
  "op": "replace",
  "path": "/hidden",
  "value": true
}
```

**Expected response:** Empty dict `{}` — HTTP 204 No Content on success.

---

### 16. `get_thread_source`
**Get the original source of a thread**
`GET /api/v1/conversations/{conversationId}/threads/{threadId}/original-source`

> NOTE: Only inbound email threads have source data. Threads created via the API or with `imported=true` may return 404 as they have no raw email source.

```json
{
  "conversation_id": 3243095282,
  "thread_id": 9884686817
}
```

**Expected response keys:** `body`, `type`

**Expected response (sample):**
```json
{
  "body": "From: help@helpscout.com\r\nSubject: FlowGenX UI Test Conversation\r\n\r\nThis is a test conversation...",
  "type": "email"
}
```

**Note:** Returns 404 for threads without raw source (imported/API-created threads). This is expected behavior.

---

## GROUP 6 — Customers Read

### 17. `list_customers`
**List customers**
`GET /api/v1/customers`

```json
{
  "page": 1
}
```

**With filters:**
```json
{
  "first_name": "Jane",
  "page": 1
}
```

**Expected response keys:** `_embedded.customers[]`, `page`

**Expected response (sample):**
```json
{
  "_embedded": {
    "customers": [
      {
        "id": 860054407,
        "firstName": "Jane",
        "photoType": "default",
        "conversationCount": 1,
        "createdAt": "2026-02-21T02:48:30Z",
        "updatedAt": "2026-02-22T14:04:45Z"
      }
    ]
  },
  "page": {
    "size": 50,
    "totalElements": 1,
    "totalPages": 1,
    "number": 1
  }
}
```

---

### 18. `get_customer`
**Get a customer by ID**
`GET /api/v1/customers/{customerId}`

```json
{
  "customer_id": 860054407
}
```

**Expected response keys:** `id`, `firstName`, `lastName`, `gender`, `organization`, `photoType`, `photoUrl`, `conversationCount`, `createdAt`, `updatedAt`

**Expected response (sample):**
```json
{
  "id": 860054407,
  "firstName": "Jane",
  "photoType": "default",
  "conversationCount": 1,
  "createdAt": "2026-02-21T02:48:30Z",
  "updatedAt": "2026-02-22T14:04:45Z"
}
```

---

## GROUP 7 — Customers Write

### 19. `create_customer`
**Create a new customer**
`POST /api/v1/customers`

> NOTE: Creates a real customer record. Use clearly test-labelled data.

```json
{
  "first_name": "FlowGenX",
  "last_name": "TestUser",
  "job_title": "QA Engineer",
  "organization": "FlowGenX Test Org",
  "emails": [{ "type": "work", "value": "flowgenx-test-user@example.com" }],
  "phones": [{ "type": "work", "value": "+1-555-000-0001" }]
}
```

**Expected response keys:** `_resource_id` (new customer ID)

**Expected response (sample):**
```json
{ "_resource_id": 860100000 }
```

Save the `_resource_id` for `update_customer` and `delete_customer`.

---

### 20. `update_customer`
**Update a customer record**
`PUT /api/v1/customers/{customerId}`

> NOTE: Use the `_resource_id` returned by `create_customer` (step 19). Updating customer `860054407` is not recommended.

```json
{
  "customer_id": 860100000,
  "job_title": "Senior QA Engineer",
  "organization": "FlowGenX Test Org Updated",
  "emails": [{ "type": "work", "value": "flowgenx-test-user-updated@example.com" }]
}
```

Replace `860100000` with the ID returned by `create_customer`.

**Expected response:** Empty dict `{}` — HTTP 204 No Content on success.

---

### 21. `delete_customer`
**Delete a customer record permanently**
`DELETE /api/v1/customers/{customerId}`

> WARNING: Permanent and irreversible. Customer contact info is deleted; associated conversations remain. Use the `_resource_id` returned by `create_customer` (step 19), NOT customer `860054407`.

```json
{
  "customer_id": 860100000
}
```

Replace `860100000` with the ID returned by `create_customer`.

**Expected response:** Empty dict `{}` — HTTP 204 No Content on success.

---

## GROUP 8 — Tags

### 22. `list_tags`
**List all tags**
`GET /api/v1/tags`

No body required.

```json
{
  "page": 1
}
```

**Expected response keys:** `_embedded.tags[]`, `page`

**Expected response (sample):**
```json
{
  "_embedded": {
    "tags": [
      { "id": 16384639, "slug": "flowgenx-test", "name": "flowgenx-test", "color": "none", "createdAt": "2026-02-22T14:04:42Z" },
      { "id": 16383577, "slug": "practice", "name": "practice", "color": "none", "createdAt": "2026-02-21T02:48:10Z" },
      { "id": 16383576, "slug": "welcome", "name": "welcome", "color": "none", "createdAt": "2026-02-21T02:48:10Z" }
    ]
  },
  "page": {
    "size": 50,
    "totalElements": 3,
    "totalPages": 1,
    "number": 1
  }
}
```

---

## GROUP 9 — Users

### 23. `list_users`
**List all users (agents)**
`GET /api/v1/users`

```json
{
  "page": 1
}
```

**Filter by mailbox:**
```json
{
  "mailbox": 362302,
  "page": 1
}
```

**Expected response keys:** `_embedded.users[]`, `page`

**Expected response (sample):**
```json
{
  "_embedded": {
    "users": [
      {
        "id": 901126,
        "firstName": "Sumon",
        "lastName": "Saha",
        "email": "ssaha@flowgenx.ai",
        "role": "owner",
        "timezone": "America/New_York",
        "createdAt": "2026-02-21T02:48:09Z",
        "updatedAt": "2026-02-21T02:48:09Z"
      }
    ]
  },
  "page": {
    "size": 50,
    "totalElements": 1,
    "totalPages": 1,
    "number": 1
  }
}
```

---

### 24. `get_user`
**Get a user (agent) by ID**
`GET /api/v1/users/{userId}`

```json
{
  "user_id": 901126
}
```

**Expected response keys:** `id`, `firstName`, `lastName`, `email`, `role`, `timezone`, `createdAt`, `updatedAt`

**Expected response (sample):**
```json
{
  "id": 901126,
  "firstName": "Sumon",
  "lastName": "Saha",
  "email": "ssaha@flowgenx.ai",
  "role": "owner",
  "timezone": "America/New_York",
  "createdAt": "2026-02-21T02:48:09Z",
  "updatedAt": "2026-02-21T02:48:09Z"
}
```

---

## GROUP 10 — Teams

### 25. `list_teams`
**List all teams**
`GET /api/v1/teams`

```json
{
  "page": 1
}
```

**Expected response keys:** `_embedded.teams[]`, `page`

**Expected response (sample):**
```json
{
  "_embedded": {
    "teams": [
      {
        "id": 901194,
        "name": "flowgen team",
        "createdAt": "2026-02-21T02:48:09Z",
        "updatedAt": "2026-02-21T02:48:09Z"
      }
    ]
  },
  "page": {
    "size": 50,
    "totalElements": 1,
    "totalPages": 1,
    "number": 1
  }
}
```

---

### 26. `list_team_members`
**List members of a team**
`GET /api/v1/teams/{teamId}/members`

```json
{
  "team_id": 901194,
  "page": 1
}
```

**Expected response keys:** `_embedded.users[]`, `page`

**Expected response (sample):**
```json
{
  "_embedded": {
    "users": [
      {
        "id": 901126,
        "firstName": "Sumon",
        "lastName": "Saha",
        "email": "ssaha@flowgenx.ai",
        "role": "owner"
      }
    ]
  },
  "page": {
    "size": 50,
    "totalElements": 1,
    "totalPages": 1,
    "number": 1
  }
}
```

---

## GROUP 11 — Webhooks (dependent chain — run in order)

> NOTE: Webhook endpoints 28-31 form a dependent chain. Run them in sequence: `create_webhook` (28) returns a webhook ID, which is needed for `get_webhook` (29), `update_webhook` (30), and `delete_webhook` (31).

### 27. `list_webhooks`
**List all webhook subscriptions**
`GET /api/v1/webhooks`

No body required (all parameters optional).

```json
{
  "page": 1
}
```

**Expected response keys:** `_embedded.webhooks` (array), `page` (pagination metadata)

**Expected response (sample):**
```json
{
  "_embedded": {
    "webhooks": [
      {
        "id": 80941,
        "url": "https://httpbin.org/post",
        "state": "enabled",
        "label": "FlowGenX UI Test Webhook",
        "events": ["convo.created", "convo.status"]
      }
    ]
  },
  "page": { "size": 25, "totalElements": 1, "totalPages": 1, "number": 1 }
}
```

---

### 28. `create_webhook`
**Create a webhook subscription**
`POST /api/v1/webhooks`

> NOTE: Creates a real webhook subscription. The `secret` field is required by the HelpScout API. Use `https://httpbin.org/post` as the test URL — it accepts all HTTP methods and echoes the request.

```json
{
  "url": "https://httpbin.org/post",
  "events": ["convo.created", "convo.status", "customer.created"],
  "state": "enabled",
  "secret": "flowgenx-test-secret-key-2026",
  "label": "FlowGenX UI Test Webhook"
}
```

**Expected response keys:** `_resource_id` (new webhook ID)

**Expected response (sample):**
```json
{ "_resource_id": 80941 }
```

Save the `_resource_id` as your `webhook_id` for steps 29-31.

---

### 29. `get_webhook`
**Get a webhook by ID**
`GET /api/v1/webhooks/{webhookId}`

> Dependency: Requires `webhook_id` from `create_webhook` (step 28).

```json
{
  "webhook_id": 80941
}
```

Replace `80941` with the `_resource_id` from `create_webhook`.

**Expected response keys:** `id`, `url`, `state`, `label`, `notification`, `events`, `secret`, `createdAt`, `updatedAt`

**Expected response (sample):**
```json
{
  "id": 80941,
  "url": "https://httpbin.org/post",
  "state": "enabled",
  "label": "FlowGenX UI Test Webhook",
  "notification": false,
  "events": ["convo.created", "convo.status", "customer.created"],
  "createdAt": "2026-02-27T00:00:00Z",
  "updatedAt": "2026-02-27T00:00:00Z"
}
```

---

### 30. `update_webhook`
**Update a webhook subscription**
`PUT /api/v1/webhooks/{webhookId}`

> Dependency: Requires `webhook_id` from `create_webhook` (step 28).
> NOTE: Full replacement — all fields including `secret` are required. Provide all desired values; omitted fields revert to defaults.

```json
{
  "webhook_id": 80941,
  "url": "https://httpbin.org/post",
  "events": ["convo.created", "convo.status", "convo.note.created", "customer.updated"],
  "state": "enabled",
  "secret": "flowgenx-test-secret-key-2026",
  "label": "FlowGenX UI Test Webhook (updated)"
}
```

Replace `80941` with the `_resource_id` from `create_webhook`.

**Expected response:** Empty dict `{}` — HTTP 204 No Content on success.

---

## GROUP 12 — Cleanup

### 31. `delete_webhook`
**Delete a webhook subscription permanently**
`DELETE /api/v1/webhooks/{webhookId}`

> Dependency: Requires `webhook_id` from `create_webhook` (step 28).
> WARNING: Permanent. HelpScout will stop delivering events to the webhook URL.

```json
{
  "webhook_id": 80941
}
```

Replace `80941` with the `_resource_id` from `create_webhook`.

**Expected response:** Empty dict `{}` — HTTP 204 No Content on success.

---

## Dependency Map

| Step | Endpoint | HTTP | Path | Saves | Needs |
|------|----------|------|------|-------|-------|
| 1 | `test_connection` | GET | `/api/v1/helpscout/test-connection` | — | — |
| 2 | `get_me` | GET | `/api/v1/users/me` | — | — |
| 3 | `list_mailboxes` | GET | `/api/v1/mailboxes` | mailbox_id=`362302` | — |
| 4 | `get_mailbox` | GET | `/api/v1/mailboxes/{mailboxId}` | — | mailbox_id |
| 5 | `list_conversations` | GET | `/api/v1/conversations` | conversation_id=`3243095282` | — |
| 6 | `get_conversation` | GET | `/api/v1/conversations/{conversationId}` | — | conversation_id |
| 7 | `create_conversation` | POST | `/api/v1/conversations` | new_conversation_id | mailbox_id |
| 8 | `update_conversation` | PATCH | `/api/v1/conversations/{conversationId}` | — | conversation_id |
| 9 | `update_conversation_tags` | PUT | `/api/v1/conversations/{conversationId}/tags` | — | conversation_id |
| 10 | `update_conversation_custom_fields` | PUT | `/api/v1/conversations/{conversationId}/fields` | — | conversation_id |
| 11 | `delete_conversation` | DELETE | `/api/v1/conversations/{conversationId}` | — | new_conversation_id (from step 7) |
| 12 | `list_threads` | GET | `/api/v1/conversations/{conversationId}/threads` | thread_ids | conversation_id |
| 13 | `create_reply` | POST | `/api/v1/conversations/{conversationId}/reply` | reply_thread_id | conversation_id, customer_id |
| 14 | `create_note` | POST | `/api/v1/conversations/{conversationId}/notes` | note_thread_id | conversation_id |
| 15 | `update_thread` | PATCH | `/api/v1/conversations/{conversationId}/threads/{threadId}` | — | conversation_id, thread_id |
| 16 | `get_thread_source` | GET | `/api/v1/conversations/{conversationId}/threads/{threadId}/original-source` | — | conversation_id, thread_id |
| 17 | `list_customers` | GET | `/api/v1/customers` | customer_id=`860054407` | — |
| 18 | `get_customer` | GET | `/api/v1/customers/{customerId}` | — | customer_id |
| 19 | `create_customer` | POST | `/api/v1/customers` | new_customer_id | — |
| 20 | `update_customer` | PUT | `/api/v1/customers/{customerId}` | — | new_customer_id (from step 19) |
| 21 | `delete_customer` | DELETE | `/api/v1/customers/{customerId}` | — | new_customer_id (from step 19) |
| 22 | `list_tags` | GET | `/api/v1/tags` | — | — |
| 23 | `list_users` | GET | `/api/v1/users` | user_id=`901126` | — |
| 24 | `get_user` | GET | `/api/v1/users/{userId}` | — | user_id |
| 25 | `list_teams` | GET | `/api/v1/teams` | team_id=`901194` | — |
| 26 | `list_team_members` | GET | `/api/v1/teams/{teamId}/members` | — | team_id |
| 27 | `list_webhooks` | GET | `/api/v1/webhooks` | — | — |
| 28 | `create_webhook` | POST | `/api/v1/webhooks` | webhook_id | — |
| 29 | `get_webhook` | GET | `/api/v1/webhooks/{webhookId}` | — | webhook_id (from step 28) |
| 30 | `update_webhook` | PUT | `/api/v1/webhooks/{webhookId}` | — | webhook_id (from step 28) |
| 31 | `delete_webhook` | DELETE | `/api/v1/webhooks/{webhookId}` | — | webhook_id (from step 28) |
