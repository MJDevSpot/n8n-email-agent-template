# Node Reference Guide

> Detailed documentation for every node in the Email Co-Pilot N8N workflow.

## Node Index

1. [Gmail Trigger](#1-gmail-trigger)
2. [Get Full Email](#2-get-full-email)
3. [Get Thread Context](#3-get-thread-context)
4. [Merge Thread Context](#4-merge-thread-context)
5. [Email Router](#5-email-router)
6. [Route By Action](#6-route-by-action)
7. [AI Agent](#7-ai-agent)
8. [check_calendar_availability](#8-check_calendar_availability)
9. [format_available_times](#9-format_available_times)
10. [Clean Agent Response](#10-clean-agent-response)
11. [Save to Airtable](#11-save-to-airtable)
12. [Notify Slack](#12-notify-slack)
13. [Filter Response](#13-filter-response)
14. [Send Email Response](#14-send-email-response)
15. [Mark as Processed](#15-mark-as-processed)
16. [Parse Booking Details](#16-parse-booking-details)
17. [Check Confidence](#17-check-confidence)
18. [Create Calendar Event](#18-create-calendar-event)
19. [Send Booking Confirmation](#19-send-booking-confirmation)
20. [Notify Low Confidence](#20-notify-low-confidence)

---

## 1. Gmail Trigger

### Purpose
Monitors the Gmail inbox for new, unprocessed emails at regular intervals.

### Node Type
`n8n-nodes-base.gmailTrigger`

### Configuration

| Setting | Value | Notes |
|---------|-------|-------|
| Poll Times | Every 5 minutes | Balance between responsiveness and API quota |
| Filters | `-label:processed-by-agent` | Excludes already-processed emails |
| Simple | False | Returns full message data |

### Input
None (trigger node)

### Output
```json
{
  "id": "message_id",
  "threadId": "thread_id",
  "labelIds": ["INBOX", "UNREAD"],
  "snippet": "Preview text...",
  "payload": {
    "headers": [...],
    "body": {...}
  }
}
```

### Credentials Required
- Gmail OAuth2 with scopes:
  - `https://www.googleapis.com/auth/gmail.readonly`
  - `https://www.googleapis.com/auth/gmail.modify`
  - `https://www.googleapis.com/auth/gmail.send`

### Common Issues
| Issue | Solution |
|-------|----------|
| Not triggering | Check workflow is active, verify OAuth |
| Duplicate processing | Verify label filter syntax |
| Missing emails | Check poll interval, verify label exists |

---

## 2. Get Full Email

### Purpose
Retrieves the complete email content including full body text.

### Node Type
`n8n-nodes-base.gmail`

### Configuration

| Setting | Value | Notes |
|---------|-------|-------|
| Operation | Get | Retrieve single message |
| Message ID | `{{ $json.id }}` | From trigger output |
| Format | Full | Include all parts |

### Input
Message ID from Gmail Trigger

### Output
```json
{
  "id": "message_id",
  "threadId": "thread_id",
  "subject": "Email subject",
  "from": "sender@example.com",
  "to": "recipient@example.com",
  "date": "RFC 2822 date",
  "text": "Plain text body",
  "html": "HTML body"
}
```

### Credentials Required
Same as Gmail Trigger

### Common Issues
| Issue | Solution |
|-------|----------|
| Empty body | Check email has text/plain part |
| 404 error | Message was deleted |

---

## 3. Get Thread Context

### Purpose
Fetches all messages in the email thread to provide conversation context to the AI.

### Node Type
`n8n-nodes-base.gmail`

### Configuration

| Setting | Value | Notes |
|---------|-------|-------|
| Operation | Get Many | List messages in thread |
| Thread ID | `{{ $json.threadId }}` | From previous node |
| Format | Full | Include message content |
| Return All | True | Get entire conversation |

### Input
Thread ID from Get Full Email

### Output
```json
{
  "messages": [
    {
      "id": "msg1",
      "snippet": "First message...",
      "payload": {...}
    },
    {
      "id": "msg2",
      "snippet": "Reply...",
      "payload": {...}
    }
  ]
}
```

### Credentials Required
Same as Gmail Trigger

### Common Issues
| Issue | Solution |
|-------|----------|
| Only one message | Normal for new threads |
| Too many messages | Consider limiting for long threads |

---

## 4. Merge Thread Context

### Purpose
Combines the current email with thread history into a unified structure for AI processing.

### Node Type
`n8n-nodes-base.code`

### Configuration

**JavaScript Code:**
```javascript
// Get current email
const email = $('Get Full Email').first().json;

// Get thread messages (if any)
const thread = $input.first() && $input.first().json ? $input.first().json : {};
const messages = Array.isArray(thread.messages) ? thread.messages : [];

// Format thread history (exclude current message)
let thread_history = '';
const otherMessages = messages.filter(m => m.id !== email.id);

if (otherMessages.length > 0) {
  thread_history = otherMessages.map(m => {
    const snippet = m.snippet || '';
    return `Previous message: ${snippet.substring(0, 200)}...`;
  }).join('\n\n');
}

return [{
  json: {
    ...email,
    thread_history: thread_history || 'No previous messages in thread',
    thread_length: messages.length
  }
}];
```

### Input
- Get Full Email output
- Get Thread Context output

### Output
```json
{
  "id": "message_id",
  "threadId": "thread_id",
  "subject": "Subject",
  "from": "sender@example.com",
  "text": "Email body",
  "thread_history": "Previous message: ...",
  "thread_length": 3
}
```

### Common Issues
| Issue | Solution |
|-------|----------|
| "Node hasn't been executed" | Check node names match exactly |
| Missing thread_history | Normal for new conversations |

---

## 5. Email Router

### Purpose
AI-powered classification that determines the email's intent and routes it appropriately.

### Node Type
`@n8n/n8n-nodes-langchain.lmChatAnthropic`

### Configuration

| Setting | Value | Notes |
|---------|-------|-------|
| Model | claude-sonnet-4-5-20250514 | Latest Sonnet model |
| Temperature | 0.3 | Low for consistent classification |
| Max Tokens | 500 | Classification doesn't need many |

**System Prompt (Key Sections):**
```
You are an email classification system. Analyze the incoming email and determine the appropriate action.

ACTIONS:
1. "new_inquiry" - New message needing a response
2. "booking_confirmation" - Someone confirming a meeting time

For booking confirmations, extract:
- selected_datetime (ISO 8601 format)
- timezone
- confidence (high/medium/low)

OUTPUT FORMAT (JSON only):
{"action": "new_inquiry"}
OR
{"action": "booking_confirmation", "selected_datetime": "2024-12-20T10:00:00", "timezone": "America/Chicago", "confidence": "high"}
```

### Input
Merged email data with thread context

### Output
```json
{
  "action": "new_inquiry"
}
// OR
{
  "action": "booking_confirmation",
  "selected_datetime": "2024-12-20T10:00:00",
  "timezone": "America/Chicago",
  "confidence": "high"
}
```

### Credentials Required
- Anthropic API Key

### Common Issues
| Issue | Solution |
|-------|----------|
| Wrong classification | Update system prompt with examples |
| JSON parsing errors | Add code to strip markdown backticks |
| Missing datetime | Lower confidence, goes to review |

---

## 6. Route By Action

### Purpose
Branches the workflow based on the Email Router's classification.

### Node Type
`n8n-nodes-base.if`

### Configuration

| Condition | Route |
|-----------|-------|
| `action` equals `booking_confirmation` | Booking path |
| Otherwise | New inquiry path |

**Expression:**
```
{{ $json.action === 'booking_confirmation' }}
```

### Input
Email Router output

### Output
Same as input, routed to appropriate branch

### Common Issues
| Issue | Solution |
|-------|----------|
| Wrong routing | Check action value casing |
| Both branches execute | Verify IF node logic |

---

## 7. AI Agent

### Purpose
Generates contextual email responses using AI with access to calendar tools.

### Node Type
`@n8n/n8n-nodes-langchain.agent`

### Configuration

| Setting | Value | Notes |
|---------|-------|-------|
| Model | claude-sonnet-4-5-20250514 | Latest Sonnet |
| Temperature | 0.7 | Balanced creativity |
| Max Tokens | 1000 | Enough for full response |
| Max Iterations | 5 | Prevent infinite loops |

**System Prompt (Key Sections):**
```
You are an email assistant for [BUSINESS_NAME].

BUSINESS CONTEXT:
[Description of services, value proposition]

RESPONSE GUIDELINES:
- Professional but friendly tone
- Offer specific meeting times when scheduling requested
- Suggest scheduling a call for pricing/contract questions
- Keep responses concise

TOOL USAGE:
1. Call check_calendar_availability ONCE when scheduling requested
2. Call format_available_times ONCE after checking calendar
3. After formatting, draft your response and STOP

OUTPUT:
Provide ONLY the email body text. No headers, no commentary.
```

### Tools Connected
- check_calendar_availability
- format_available_times

### Input
Email data with classification

### Output
```json
{
  "output": "Hi [Name],\n\nThank you for reaching out..."
}
```

### Credentials Required
- Anthropic API Key

### Common Issues
| Issue | Solution |
|-------|----------|
| Looping on tools | Add explicit STOP instructions |
| Generic responses | Improve business context in prompt |
| Metadata in output | Use Clean Agent Response node |

---

## 8. check_calendar_availability

### Purpose
Queries Google Calendar FreeBusy API to find busy times across configured calendars.

### Node Type
`n8n-nodes-base.httpRequest`

### Configuration

| Setting | Value |
|---------|-------|
| Method | POST |
| URL | `https://www.googleapis.com/calendar/v3/freeBusy` |
| Authentication | OAuth2 |

**Request Body:**
```json
{
  "timeMin": "{{ $now.toISO() }}",
  "timeMax": "{{ $now.plus({days: 14}).toISO() }}",
  "timeZone": "America/Chicago",
  "items": [
    {"id": "primary"},
    {"id": "secondary@example.com"}
  ]
}
```

### Input
Called by AI Agent when needed

### Output
```json
{
  "calendars": {
    "primary": {
      "busy": [
        {"start": "2024-12-20T09:00:00Z", "end": "2024-12-20T10:00:00Z"},
        {"start": "2024-12-20T14:00:00Z", "end": "2024-12-20T15:00:00Z"}
      ]
    }
  }
}
```

### Credentials Required
- Google Calendar OAuth2 with scopes:
  - `https://www.googleapis.com/auth/calendar.readonly`
  - `https://www.googleapis.com/auth/calendar.events`

### Common Issues
| Issue | Solution |
|-------|----------|
| Empty calendars | Check calendar IDs are correct |
| 403 error | Verify OAuth scopes |
| Wrong timezone | Update timeZone parameter |

---

## 9. format_available_times

### Purpose
Converts FreeBusy response into human-readable available time slots.

### Node Type
`n8n-nodes-base.code`

### Configuration

**JavaScript Code:**
```javascript
const config = {
  timezone: 'America/Chicago',
  workingDays: [1, 2, 3, 4, 5], // Mon-Fri
  startHour: 9,
  endHour: 17,
  slotDuration: 30,
  bufferMinutes: 15,
  maxSlots: 4
};

// Get busy times from FreeBusy response
const freeBusyData = $input.first().json;
const busyTimes = [];

// Extract all busy periods
if (freeBusyData.calendars) {
  Object.values(freeBusyData.calendars).forEach(cal => {
    if (cal.busy) {
      cal.busy.forEach(period => {
        busyTimes.push({
          start: new Date(period.start),
          end: new Date(period.end)
        });
      });
    }
  });
}

// Generate available slots (next 14 days)
const slots = [];
const now = new Date();
const endDate = new Date(now.getTime() + 14 * 24 * 60 * 60 * 1000);

let current = new Date(now);
current.setMinutes(0, 0, 0);
current.setHours(current.getHours() + 1);

while (current < endDate && slots.length < config.maxSlots) {
  const dayOfWeek = current.getDay();
  const hour = current.getHours();

  // Check if within working hours
  if (config.workingDays.includes(dayOfWeek) &&
      hour >= config.startHour &&
      hour < config.endHour) {

    const slotEnd = new Date(current.getTime() + config.slotDuration * 60000);

    // Check if conflicts with any busy time
    const isBusy = busyTimes.some(busy =>
      current < busy.end && slotEnd > busy.start
    );

    if (!isBusy) {
      slots.push({
        datetime: current.toISOString(),
        formatted: current.toLocaleString('en-US', {
          weekday: 'long',
          month: 'long',
          day: 'numeric',
          hour: 'numeric',
          minute: '2-digit',
          timeZone: config.timezone
        })
      });
    }
  }

  // Move to next slot
  current = new Date(current.getTime() + config.slotDuration * 60000);
}

return [{
  json: {
    available_slots: slots,
    formatted_text: slots.map((s, i) => `${i + 1}. ${s.formatted}`).join('\n')
  }
}];
```

### Input
FreeBusy API response

### Output
```json
{
  "available_slots": [
    {"datetime": "2024-12-20T10:00:00Z", "formatted": "Friday, December 20 at 10:00 AM"},
    {"datetime": "2024-12-20T14:00:00Z", "formatted": "Friday, December 20 at 2:00 PM"}
  ],
  "formatted_text": "1. Friday, December 20 at 10:00 AM\n2. Friday, December 20 at 2:00 PM"
}
```

### Client-Variable Settings

| Parameter | Description | Default |
|-----------|-------------|---------|
| timezone | Client's timezone | America/Chicago |
| workingDays | Array of working days (0=Sun) | [1,2,3,4,5] |
| startHour | First available hour | 9 |
| endHour | Last available hour | 17 |
| slotDuration | Meeting length (minutes) | 30 |
| bufferMinutes | Gap between meetings | 15 |
| maxSlots | Maximum times to offer | 4 |

### Common Issues
| Issue | Solution |
|-------|----------|
| No slots returned | Check working hours/days config |
| Weekend times | Verify workingDays excludes 0, 6 |
| Wrong timezone display | Update timezone in toLocaleString |

---

## 10. Clean Agent Response

### Purpose
Removes AI metadata, headers, and formatting artifacts from the response before sending.

### Node Type
`n8n-nodes-base.code`

### Configuration

**JavaScript Code:**
```javascript
let response = $input.first().json.output || '';

// Remove classification tags
response = response.replace(/\[CLASSIFICATION:.*?\]/gi, '');
response = response.replace(/CLASSIFICATION:.*$/gim, '');

// Remove email headers
response = response.replace(/^DRAFT EMAIL RESPONSE:?\s*/i, '');
response = response.replace(/^TO:.*$/gim, '');
response = response.replace(/^FROM:.*$/gim, '');
response = response.replace(/^SUBJECT:.*$/gim, '');
response = response.replace(/^DATE:.*$/gim, '');

// Remove dividers and commentary
response = response.replace(/^---+.*$/gm, '');
response = response.replace(/^\[.*?\]$/gm, '');

// Clean up whitespace
response = response.trim();
response = response.replace(/\n{3,}/g, '\n\n');

return [{
  json: {
    ...$input.first().json,
    cleaned_response: response
  }
}];
```

### Input
AI Agent output

### Output
```json
{
  "output": "Original AI output...",
  "cleaned_response": "Clean email body ready to send"
}
```

### Common Issues
| Issue | Solution |
|-------|----------|
| New patterns appearing | Add regex to handle new formats |
| Over-cleaning | Make patterns more specific |

---

## 11. Save to Airtable

### Purpose
Logs the email and AI draft to Airtable for review and tracking.

### Node Type
`n8n-nodes-base.airtable`

### Configuration

| Setting | Value |
|---------|-------|
| Operation | Create |
| Base ID | `app...` (client-specific) |
| Table ID | `tbl...` (Email Log table) |

**Field Mapping:**

| Airtable Field | Value |
|----------------|-------|
| Email ID | `{{ $json.id }}` |
| Thread ID | `{{ $json.threadId }}` |
| From Email | `{{ $json.from }}` |
| From Name | `{{ $json.fromName }}` |
| Subject | `{{ $json.subject }}` |
| Body | `{{ $json.text }}` |
| Status | "Draft Ready" |
| Classification | `{{ $json.classification }}` |
| Agent Draft | `{{ $json.cleaned_response }}` |
| Created | `{{ $now.toISO() }}` |

### Input
Cleaned response data

### Output
Created Airtable record with record ID

### Credentials Required
- Airtable Personal Access Token with scopes:
  - `data.records:read`
  - `data.records:write`

### Common Issues
| Issue | Solution |
|-------|----------|
| 422 error | Check field names match exactly |
| 403 error | Verify PAT has correct scopes |
| Missing fields | Verify all fields exist in base |

---

## 12. Notify Slack

### Purpose
Sends a notification to Slack when a new draft is ready for review.

### Node Type
`n8n-nodes-base.slack`

### Configuration

| Setting | Value |
|---------|-------|
| Operation | Send Message |
| Channel | `#email-copilot` (client-specific) |

**Message Template:**
```
📧 *New Email Draft Ready*

*From:* {{ $json.from }}
*Subject:* {{ $json.subject }}

*Draft Response:*
{{ $json.cleaned_response.substring(0, 500) }}...

<{{ $json.airtable_url }}|View in Airtable>
```

### Input
Airtable record data with draft

### Output
Slack message confirmation

### Credentials Required
- Slack OAuth2 with scopes:
  - `chat:write`
  - `channels:read`

### Common Issues
| Issue | Solution |
|-------|----------|
| Channel not found | Invite bot to channel |
| Rate limited | Add delay between messages |
| Formatting broken | Escape special characters |

---

## 13. Filter Response

### Purpose
Validates that the AI response is suitable for sending.

### Node Type
`n8n-nodes-base.filter`

### Configuration

**Conditions:**
```
{{ $json.cleaned_response.length > 50 }}
AND
{{ !$json.cleaned_response.includes('[ERROR]') }}
AND
{{ !$json.cleaned_response.includes('undefined') }}
```

### Input
Cleaned response

### Output
Passes through if valid, stops if not

### Common Issues
| Issue | Solution |
|-------|----------|
| Blocking good emails | Adjust length threshold |
| Allowing bad emails | Add more filter conditions |

---

## 14. Send Email Response

### Purpose
Sends the AI-drafted response via Gmail.

### Node Type
`n8n-nodes-base.gmail`

### Configuration

| Setting | Value |
|---------|-------|
| Operation | Reply |
| Message ID | `{{ $json.id }}` |
| Message | `{{ $json.cleaned_response }}` |

### Input
Validated response data

### Output
Sent message confirmation

### Credentials Required
- Gmail OAuth2 with `gmail.send` scope

### Common Issues
| Issue | Solution |
|-------|----------|
| 403 error | Re-authorize with send scope |
| Wrong thread | Verify message ID is correct |

---

## 15. Mark as Processed

### Purpose
Adds a label to the email to prevent re-processing.

### Node Type
`n8n-nodes-base.gmail`

### Configuration

| Setting | Value |
|---------|-------|
| Operation | Add Labels |
| Message ID | `{{ $json.id }}` |
| Labels | `processed-by-agent` |

### Input
Message ID

### Output
Label confirmation

### Common Issues
| Issue | Solution |
|-------|----------|
| Label not found | Create label in Gmail first |
| Still processing | Check trigger filter syntax |

---

## 16. Parse Booking Details

### Purpose
Extracts structured booking information from the Email Router output.

### Node Type
`n8n-nodes-base.code`

### Configuration

**JavaScript Code:**
```javascript
const routerOutput = $('Email Router').first().json;
const email = $('Merge Thread Context').first().json;

// Handle potential JSON in markdown code blocks
let parsed = routerOutput;
if (typeof routerOutput === 'string') {
  const jsonMatch = routerOutput.match(/```json?\s*([\s\S]*?)\s*```/);
  if (jsonMatch) {
    parsed = JSON.parse(jsonMatch[1]);
  } else {
    parsed = JSON.parse(routerOutput);
  }
}

return [{
  json: {
    action: parsed.action,
    selected_datetime: parsed.selected_datetime,
    timezone: parsed.timezone || 'America/Chicago',
    confidence: parsed.confidence || 'low',
    from_email: email.from,
    from_name: email.fromName || email.from,
    subject: email.subject,
    thread_id: email.threadId,
    message_id: email.id
  }
}];
```

### Input
Email Router output + original email data

### Output
```json
{
  "action": "booking_confirmation",
  "selected_datetime": "2024-12-20T10:00:00",
  "timezone": "America/Chicago",
  "confidence": "high",
  "from_email": "contact@example.com",
  "from_name": "Contact Name",
  "subject": "Re: Meeting",
  "thread_id": "thread_123",
  "message_id": "msg_456"
}
```

### Common Issues
| Issue | Solution |
|-------|----------|
| JSON parse error | Add markdown code block handling |
| Missing fields | Add default values |

---

## 17. Check Confidence

### Purpose
Routes bookings based on AI's confidence in the extracted datetime.

### Node Type
`n8n-nodes-base.if`

### Configuration

**Condition:**
```
{{ $json.confidence === 'high' }}
```

### Branches
| Branch | Condition | Destination |
|--------|-----------|-------------|
| True | High confidence | Create Calendar Event |
| False | Medium/Low | Notify Low Confidence |

### Input
Parsed booking details

### Output
Same data, routed appropriately

---

## 18. Create Calendar Event

### Purpose
Creates a Google Calendar event with Google Meet link.

### Node Type
`n8n-nodes-base.googleCalendar`

### Configuration

| Setting | Value |
|---------|-------|
| Operation | Create Event |
| Calendar | Primary (or specified) |
| Title | "Meeting with {{ $json.from_name }}" |
| Start | `{{ $json.selected_datetime }}` |
| End | `{{ DateTime.fromISO($json.selected_datetime).plus({minutes: 30}).toISO() }}` |
| Add Conference | Google Meet |
| Attendees | `{{ $json.from_email }}` |

### Input
Booking details with datetime

### Output
Created event with Meet link

### Credentials Required
- Google Calendar OAuth2 with `calendar.events` scope

### Common Issues
| Issue | Solution |
|-------|----------|
| Invalid datetime | Check ISO format |
| No Meet link | Enable in calendar settings |
| Permission denied | Re-authorize OAuth |

---

## 19. Send Booking Confirmation

### Purpose
Emails the meeting confirmation with details and video link.

### Node Type
`n8n-nodes-base.gmail`

### Configuration

| Setting | Value |
|---------|-------|
| Operation | Reply |
| Message ID | `{{ $json.message_id }}` |

**Message Template:**
```
Hi {{ $json.from_name.split(' ')[0] }},

Great! I've scheduled our meeting for {{ $json.formatted_datetime }}.

A calendar invitation with the video meeting link has been sent to your email.

Looking forward to speaking with you!

Best regards,
[SENDER_NAME]
```

### Input
Event creation result + booking details

### Output
Sent message confirmation

---

## 20. Notify Low Confidence

### Purpose
Alerts via Slack when a booking needs manual review.

### Node Type
`n8n-nodes-base.slack`

### Configuration

**Message Template:**
```
⚠️ *Booking Needs Review*

*From:* {{ $json.from_email }}
*Subject:* {{ $json.subject }}
*Detected Time:* {{ $json.selected_datetime }}
*Confidence:* {{ $json.confidence }}

Please review and manually book if appropriate.

<{{ $json.airtable_url }}|View in Airtable>
```

### Input
Low confidence booking data

### Output
Slack notification confirmation

---

## Node Connection Map

```
Gmail Trigger
    │
    ▼
Get Full Email
    │
    ▼
Get Thread Context
    │
    ▼
Merge Thread Context
    │
    ▼
Email Router
    │
    ▼
Route By Action ─────────────────────┐
    │                                │
    │ (new_inquiry)                  │ (booking_confirmation)
    ▼                                ▼
AI Agent                        Parse Booking Details
    │                                │
    ├─► check_calendar_availability  ▼
    │       │                   Check Confidence
    │       ▼                        │
    │   format_available_times       ├─► (high) ──► Create Calendar Event
    │       │                        │                    │
    ◄───────┘                        │                    ▼
    │                                │              Send Booking Confirmation
    ▼                                │                    │
Clean Agent Response                 │                    ▼
    │                                │              Mark as Processed
    ▼                                │
Save to Airtable                     └─► (low) ──► Notify Low Confidence
    │                                                    │
    ▼                                                    ▼
Notify Slack                                       Mark as Processed
    │
    ▼
Filter Response
    │
    ▼
Send Email Response
    │
    ▼
Mark as Processed
```

---

## Related Documents

- [Architecture Overview](./ARCHITECTURE_OVERVIEW.md) - System architecture
- [API Dependencies](./API_DEPENDENCIES.md) - External service details
- [Configuration Matrix](../internal/CONFIGURATION_MATRIX.md) - Client settings
- [Troubleshooting Playbook](../internal/TROUBLESHOOTING_PLAYBOOK.md) - Issue resolution
