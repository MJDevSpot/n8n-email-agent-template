# API Dependencies

> Complete reference for all external services used by the Email Co-Pilot.

## Service Overview

| Service | Provider | Purpose | Authentication |
|---------|----------|---------|----------------|
| Gmail | Google | Email read/send | OAuth 2.0 |
| Calendar | Google | Availability & booking | OAuth 2.0 |
| Claude | Anthropic | AI classification & drafting | API Key |
| Airtable | Airtable | Data logging & review | PAT |
| Slack | Slack | Notifications | OAuth 2.0 |

---

## Gmail API

### Service Details

| Attribute | Value |
|-----------|-------|
| Provider | Google Cloud |
| API Version | v1 |
| Base URL | `https://gmail.googleapis.com/gmail/v1` |
| Documentation | https://developers.google.com/gmail/api |
| Console | https://console.cloud.google.com |

### Required OAuth Scopes

| Scope | Purpose | Required |
|-------|---------|----------|
| `https://www.googleapis.com/auth/gmail.readonly` | Read emails and threads | Yes |
| `https://www.googleapis.com/auth/gmail.modify` | Add/remove labels | Yes |
| `https://www.googleapis.com/auth/gmail.send` | Send responses | Yes |

### Endpoints Used

| Endpoint | Method | Purpose |
|----------|--------|---------|
| `/users/me/messages` | GET | List messages |
| `/users/me/messages/{id}` | GET | Get full message |
| `/users/me/threads/{id}` | GET | Get thread messages |
| `/users/me/messages/{id}/modify` | POST | Add labels |
| `/users/me/messages/send` | POST | Send email |

### Rate Limits

| Limit Type | Value | Notes |
|------------|-------|-------|
| Quota Units/Day | 1,000,000,000 | Per project |
| Quota Units/User/Second | 250 | Per user |
| Messages List | 5 quota units | Per request |
| Messages Get | 5 quota units | Per request |
| Messages Send | 100 quota units | Per request |

### Usage Per Email

| Operation | Quota Cost |
|-----------|------------|
| Trigger poll | 5 units |
| Get full email | 5 units |
| Get thread | 5 units |
| Send response | 100 units |
| Add label | 5 units |
| **Total per email** | **~120 units** |

### Error Codes

| Code | Meaning | Resolution |
|------|---------|------------|
| 400 | Invalid request | Check request format |
| 401 | Invalid credentials | Re-authenticate OAuth |
| 403 | Rate limit/permission | Check quotas, verify scopes |
| 404 | Message not found | Message was deleted |
| 429 | Too many requests | Implement backoff |
| 500 | Server error | Retry with backoff |

### Token Management

- **Access Token Lifetime:** 1 hour
- **Refresh Token:** Long-lived, used for renewal
- **Auto-Refresh:** N8N handles automatically
- **Revocation:** User can revoke at https://myaccount.google.com/permissions

---

## Google Calendar API

### Service Details

| Attribute | Value |
|-----------|-------|
| Provider | Google Cloud |
| API Version | v3 |
| Base URL | `https://www.googleapis.com/calendar/v3` |
| Documentation | https://developers.google.com/calendar/api |

### Required OAuth Scopes

| Scope | Purpose | Required |
|-------|---------|----------|
| `https://www.googleapis.com/auth/calendar.readonly` | Read free/busy | Yes |
| `https://www.googleapis.com/auth/calendar.events` | Create events | Yes |

### Endpoints Used

| Endpoint | Method | Purpose |
|----------|--------|---------|
| `/freeBusy` | POST | Query availability |
| `/calendars/{id}/events` | POST | Create event |

### FreeBusy API Details

**Request Format:**
```json
{
  "timeMin": "2024-12-20T00:00:00Z",
  "timeMax": "2024-12-27T00:00:00Z",
  "timeZone": "America/Chicago",
  "items": [
    {"id": "primary"},
    {"id": "secondary@gmail.com"}
  ]
}
```

**Response Format:**
```json
{
  "kind": "calendar#freeBusy",
  "timeMin": "2024-12-20T00:00:00Z",
  "timeMax": "2024-12-27T00:00:00Z",
  "calendars": {
    "primary": {
      "busy": [
        {
          "start": "2024-12-20T09:00:00Z",
          "end": "2024-12-20T10:00:00Z"
        }
      ]
    }
  }
}
```

### Event Creation Details

**Request Format:**
```json
{
  "summary": "Meeting with Contact Name",
  "start": {
    "dateTime": "2024-12-20T10:00:00",
    "timeZone": "America/Chicago"
  },
  "end": {
    "dateTime": "2024-12-20T10:30:00",
    "timeZone": "America/Chicago"
  },
  "attendees": [
    {"email": "contact@example.com"}
  ],
  "conferenceData": {
    "createRequest": {
      "requestId": "unique-id",
      "conferenceSolutionKey": {"type": "hangoutsMeet"}
    }
  }
}
```

### Rate Limits

| Limit Type | Value |
|------------|-------|
| Queries/Day | 1,000,000 |
| Queries/User/100s | 500 |

### Error Codes

| Code | Meaning | Resolution |
|------|---------|------------|
| 401 | Auth failed | Re-authenticate |
| 403 | Calendar access denied | Check sharing settings |
| 404 | Calendar not found | Verify calendar ID |
| 409 | Conflict | Time slot already taken |

---

## Anthropic Claude API

### Service Details

| Attribute | Value |
|-----------|-------|
| Provider | Anthropic |
| API Version | 2023-06-01 |
| Base URL | `https://api.anthropic.com/v1` |
| Documentation | https://docs.anthropic.com |
| Console | https://console.anthropic.com |

### Authentication

| Method | Format |
|--------|--------|
| Type | API Key |
| Header | `x-api-key: sk-ant-...` |
| Additional | `anthropic-version: 2023-06-01` |

### Endpoints Used

| Endpoint | Method | Purpose |
|----------|--------|---------|
| `/messages` | POST | Create chat completion |

### Request Format

```json
{
  "model": "claude-sonnet-4-5-20250514",
  "max_tokens": 1000,
  "system": "System prompt here...",
  "messages": [
    {
      "role": "user",
      "content": "Email content..."
    }
  ]
}
```

### Response Format

```json
{
  "id": "msg_...",
  "type": "message",
  "role": "assistant",
  "content": [
    {
      "type": "text",
      "text": "Response content..."
    }
  ],
  "model": "claude-sonnet-4-5-20250514",
  "stop_reason": "end_turn",
  "usage": {
    "input_tokens": 500,
    "output_tokens": 200
  }
}
```

### Models

| Model | Use Case | Cost (per 1M tokens) |
|-------|----------|---------------------|
| claude-sonnet-4-5-20250514 | Classification & drafting | Input: $3 / Output: $15 |
| claude-3-5-haiku-latest | Fast, cheap tasks | Input: $1 / Output: $5 |

### Rate Limits

| Tier | Requests/Min | Tokens/Min | Tokens/Day |
|------|-------------|------------|------------|
| Tier 1 | 50 | 40,000 | 1,000,000 |
| Tier 2 | 1,000 | 80,000 | 2,500,000 |
| Tier 3 | 2,000 | 160,000 | 5,000,000 |
| Tier 4 | 4,000 | 400,000 | 10,000,000 |

### Usage Per Email

| Component | Input Tokens | Output Tokens |
|-----------|--------------|---------------|
| Email Router | ~500 | ~50 |
| AI Agent | ~1000 | ~300 |
| **Total** | **~1500** | **~350** |

**Estimated cost per email:** $0.01-0.02

### Error Codes

| Code | Meaning | Resolution |
|------|---------|------------|
| 400 | Invalid request | Check request format |
| 401 | Invalid API key | Verify key is correct |
| 403 | Permission denied | Check account status |
| 429 | Rate limited | Implement backoff |
| 500 | Server error | Retry |
| 529 | Overloaded | Wait and retry |

### Best Practices

1. **Use structured output** for classification
2. **Set temperature low** (0.3) for consistent classification
3. **Set temperature higher** (0.7) for creative responses
4. **Limit max_tokens** to reduce cost
5. **Include stop sequences** to prevent runaway generation

---

## Airtable API

### Service Details

| Attribute | Value |
|-----------|-------|
| Provider | Airtable |
| API Version | v0 |
| Base URL | `https://api.airtable.com/v0` |
| Documentation | https://airtable.com/developers/web/api |
| Console | https://airtable.com/create/tokens |

### Authentication

| Method | Format |
|--------|--------|
| Type | Personal Access Token (PAT) |
| Header | `Authorization: Bearer pat...` |

### Required Scopes

| Scope | Purpose |
|-------|---------|
| `data.records:read` | Read existing records |
| `data.records:write` | Create/update records |
| `schema.bases:read` | Access base schema |

### Endpoints Used

| Endpoint | Method | Purpose |
|----------|--------|---------|
| `/{baseId}/{tableId}` | POST | Create record |
| `/{baseId}/{tableId}` | PATCH | Update record |
| `/{baseId}/{tableId}` | GET | List records |

### Create Record Format

**Request:**
```json
{
  "fields": {
    "Email ID": "msg_123",
    "From Email": "contact@example.com",
    "Subject": "Question about services",
    "Body": "Email content...",
    "Status": "Draft Ready",
    "Agent Draft": "AI response..."
  }
}
```

**Response:**
```json
{
  "id": "rec123",
  "createdTime": "2024-12-20T10:00:00.000Z",
  "fields": {
    "Email ID": "msg_123",
    ...
  }
}
```

### Rate Limits

| Limit Type | Value |
|------------|-------|
| Requests/Second | 5 |
| Records/Request | 10 |

### Error Codes

| Code | Meaning | Resolution |
|------|---------|------------|
| 401 | Unauthorized | Check PAT is valid |
| 403 | Forbidden | Verify PAT scopes |
| 404 | Not found | Check base/table IDs |
| 422 | Invalid request | Check field names/types |
| 429 | Rate limited | Add delay between requests |

### Field Type Requirements

| Field Type | Value Format |
|------------|--------------|
| Single line text | String |
| Long text | String |
| Email | Valid email string |
| Single select | Must match existing option |
| Date | ISO 8601 format |
| Number | Numeric value |

---

## Slack API

### Service Details

| Attribute | Value |
|-----------|-------|
| Provider | Slack |
| API Version | Web API |
| Base URL | `https://slack.com/api` |
| Documentation | https://api.slack.com |
| App Directory | https://api.slack.com/apps |

### Authentication

| Method | Format |
|--------|--------|
| Type | OAuth 2.0 Bot Token |
| Header | `Authorization: Bearer xoxb-...` |

### Required Scopes

| Scope | Purpose |
|-------|---------|
| `chat:write` | Send messages |
| `channels:read` | Access channel info |

### Endpoints Used

| Endpoint | Method | Purpose |
|----------|--------|---------|
| `/chat.postMessage` | POST | Send message |

### Message Format

**Request:**
```json
{
  "channel": "C0123456789",
  "text": "New email draft ready",
  "blocks": [
    {
      "type": "section",
      "text": {
        "type": "mrkdwn",
        "text": "*New Email Draft Ready*"
      }
    }
  ]
}
```

**Response:**
```json
{
  "ok": true,
  "channel": "C0123456789",
  "ts": "1234567890.123456",
  "message": {...}
}
```

### Rate Limits

| Limit Type | Value |
|------------|-------|
| Tier 1 (chat.postMessage) | 1+ per second |
| Burst | 120 per minute |

### Error Codes

| Error | Meaning | Resolution |
|-------|---------|------------|
| `not_authed` | Invalid token | Re-authorize OAuth |
| `channel_not_found` | Invalid channel | Check channel ID |
| `not_in_channel` | Bot not added | Invite bot to channel |
| `ratelimited` | Too many requests | Implement backoff |

---

## N8N Cloud

### Service Details

| Attribute | Value |
|-----------|-------|
| Provider | N8N GmbH |
| Documentation | https://docs.n8n.io |
| Status | https://status.n8n.io |
| Support | support@n8n.io |

### Execution Limits (by Plan)

| Plan | Executions/Month | Active Workflows |
|------|------------------|------------------|
| Starter | 2,500 | 5 |
| Pro | 10,000 | 15 |
| Enterprise | Unlimited | Unlimited |

### Credential Storage

- All credentials encrypted at rest
- OAuth tokens auto-refreshed
- No credential sharing between workflows

---

## Dependency Health Monitoring

### Status Pages

| Service | Status URL |
|---------|------------|
| Google | https://www.google.com/appsstatus |
| Anthropic | https://status.anthropic.com |
| Airtable | https://status.airtable.com |
| Slack | https://status.slack.com |
| N8N | https://status.n8n.io |

### Health Check Schedule

| Check | Frequency | Method |
|-------|-----------|--------|
| Gmail connection | Daily | Test credential |
| Calendar connection | Daily | Test credential |
| Anthropic API | Per request | Monitor errors |
| Airtable connection | Daily | Test credential |
| Slack connection | Daily | Test credential |

### Alerting Thresholds

| Metric | Warning | Critical |
|--------|---------|----------|
| Consecutive failures | 2 | 5 |
| Error rate | 5% | 10% |
| Response time | 30s | 60s |

---

## Cost Estimation

### Per-Email Costs

| Service | Cost/Email | Notes |
|---------|------------|-------|
| Gmail API | $0.00 | Included in Google Workspace |
| Calendar API | $0.00 | Included in Google Workspace |
| Anthropic | $0.01-0.02 | ~1850 tokens average |
| Airtable | $0.00 | Included in plan |
| Slack | $0.00 | Included in plan |
| **Total** | **~$0.015** | Per email processed |

### Monthly Cost Estimates

| Email Volume | AI Cost | Total |
|--------------|---------|-------|
| 100 emails | $1.50 | $1.50 |
| 500 emails | $7.50 | $7.50 |
| 1000 emails | $15.00 | $15.00 |
| 2000 emails | $30.00 | $30.00 |

---

## Related Documents

- [Architecture Overview](./ARCHITECTURE_OVERVIEW.md) - System design
- [Node Reference](./NODE_REFERENCE.md) - N8N node details
- [Disaster Recovery](./DISASTER_RECOVERY.md) - Failure handling
- [Troubleshooting Playbook](../internal/TROUBLESHOOTING_PLAYBOOK.md) - Issue resolution
