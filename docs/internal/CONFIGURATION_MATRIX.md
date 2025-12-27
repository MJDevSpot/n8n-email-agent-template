# Configuration Matrix

> Single source of truth for all client-variable settings in the Email Co-Pilot system.

## How to Use This Document

This matrix contains every variable that changes between client deployments. When setting up a new client:

1. Gather all values using the [Client Onboarding Form](../templates/client-onboarding-form.md)
2. Fill in this matrix for the specific client
3. Reference this document during deployment
4. Store the completed matrix securely for future reference

---

## Client Information

| Field | Description | Example Value | Where Used |
|-------|-------------|---------------|------------|
| Client Name | Legal business name | "Apex Construction LLC" | AI Prompt, Email Signature |
| Client Display Name | Short name for UI | "Apex Construction" | Slack notifications, Airtable |
| Client Domain | Primary email domain | "apexconstruction.com" | Gmail Filter |
| Primary Contact Name | Main point of contact | "John Smith" | Support communications |
| Primary Contact Email | POC email address | "john@apexconstruction.com" | Support communications |
| Primary Contact Phone | POC phone number | "(555) 123-4567" | Escalation |
| Timezone | Client's primary timezone | "America/Chicago" | All date/time operations |

---

## Calendar Configuration

| Field | Description | Example Value | Where Used |
|-------|-------------|---------------|------------|
| Primary Calendar | Main calendar for booking meetings | "info@apexconstruction.com" | Calendar Check, Event Creation |
| Conflict Calendar 1 | Additional calendar to check | "john@apexconstruction.com" | Calendar Check (FreeBusy) |
| Conflict Calendar 2 | Additional calendar to check | "sales@apexconstruction.com" | Calendar Check (FreeBusy) |
| Conflict Calendar 3 | Additional calendar to check | "" | Calendar Check (FreeBusy) |
| Timezone | IANA timezone identifier | "America/Chicago" | All datetime operations |

**Notes:**
- Primary Calendar is where events are created
- Conflict Calendars are only checked for availability (read-only)
- Maximum 3 conflict calendars supported
- All calendars must be accessible by the authenticated Google account

---

## Availability Settings

| Field | Description | Example Value | Where Used |
|-------|-------------|---------------|------------|
| Working Days | Days available for meetings (0=Sun, 6=Sat) | `[1, 2, 3, 4, 5]` (Mon-Fri) | format_available_times |
| Start Hour | Earliest meeting time (24h format) | `9` (9 AM) | format_available_times |
| End Hour | Latest meeting END time (24h format) | `17` (5 PM) | format_available_times |
| Meeting Duration | Standard meeting length (minutes) | `30` | format_available_times, Event Creation |
| Buffer Minutes | Gap required between meetings | `15` | format_available_times |
| Lookahead Days | How far to look for availability | `30` | format_available_times |
| Max Slots Offered | Maximum meeting times to offer | `4` | format_available_times |

**Common Configurations:**

| Profile | Working Days | Hours | Duration | Buffer |
|---------|-------------|-------|----------|--------|
| Standard Business | Mon-Fri (1-5) | 9 AM - 5 PM | 30 min | 15 min |
| Contractor | Tue-Thu (2-4) | 10 AM - 3 PM | 30 min | 15 min |
| Executive | Mon-Wed (1-3) | 1 PM - 4 PM | 45 min | 30 min |
| Sales-Heavy | Mon-Fri (1-5) | 8 AM - 6 PM | 15 min | 10 min |

---

## AI Configuration

| Field | Description | Example Value | Where Used |
|-------|-------------|---------------|------------|
| Business Description | 1-2 sentence company description | "Commercial roofing contractor serving the Southeast with 25+ years of experience" | AI Agent System Prompt |
| Services Offered | Key services to mention | "Roof inspections, repairs, full replacements, preventive maintenance contracts" | AI Agent System Prompt |
| Industry Vertical | Client's industry | "Construction / Roofing" | AI Agent System Prompt |
| Tone Descriptor | Communication style | "Professional but friendly, straightforward, no jargon" | AI Agent System Prompt |
| Meeting Type Name | What to call scheduled meetings | "Initial Consultation" | AI Prompt, Calendar Event Title |
| Calendar Event Title Template | Format for event titles | "Initial Consultation - {client_name}" | Calendar Event Creation |
| Deferral Topics | Topics to route to human | `["pricing", "contracts", "legal", "warranty claims"]` | AI Agent System Prompt |
| Unique Selling Points | Key differentiators | "24-hour emergency response, licensed and insured, free inspections" | AI Agent System Prompt |
| Prohibited Topics | Things AI should never discuss | `["competitor pricing", "legal disputes"]` | AI Agent System Prompt |

---

## Email Signature Configuration

| Field | Description | Example Value | Where Used |
|-------|-------------|---------------|------------|
| Sender Name | Name in email signature | "Apex Email Assistant" | Send Email Response |
| On Behalf Of | Company attribution line | "Apex Construction" | Send Email Response |
| Closing Line | Final line before signature | "Looking forward to connecting!" | Send Email Response |
| Additional Signature Lines | Extra content | "" | Send Email Response |

**Signature Format:**
```
{closing_line}

Best regards,
{sender_name}
on behalf of {on_behalf_of}
```

---

## Integration Credentials

| Credential Type | N8N Credential Name Pattern | Required Scopes | Notes |
|-----------------|----------------------------|-----------------|-------|
| Gmail OAuth | "[Client] Gmail" | gmail.readonly, gmail.send, gmail.modify, gmail.labels | Per-client credential |
| Google Calendar OAuth | "[Client] Calendar" | calendar.readonly, calendar.events | Per-client credential |
| Anthropic API | "Anthropic account" | API key access | Can be shared across clients |
| Airtable PAT | "[Client] Airtable" | data.records:read, data.records:write | Per-client base |
| Slack OAuth | "[Client] Slack" | chat:write, channels:read | Per-client workspace |

**Credential Naming Convention:**
- Format: `[ClientShortName] ServiceName`
- Example: `[Apex] Gmail`, `[Apex] Calendar`, `[Apex] Airtable`

---

## Airtable Configuration

| Field | Description | Example Value | Where Used |
|-------|-------------|---------------|------------|
| Base ID | Airtable base identifier | "appXXXXXXXXXXXXXX" | Airtable nodes |
| Table ID | Email tracking table | "tblXXXXXXXXXXXXXX" | Airtable nodes |
| Base URL | Link for Slack notifications | "https://airtable.com/appXXXXXXX" | Slack notifications |
| View URL | Direct link to main view | "https://airtable.com/appXXX/tblXXX/viwXXX" | Handoff documentation |

**Required Airtable Fields:**

| Field Name | Field Type | Options (if applicable) |
|------------|-----------|------------------------|
| Email ID | Single line text | |
| From Email | Email | |
| From Name | Single line text | |
| Subject | Single line text | |
| Body | Long text | |
| Status | Single select | New, Draft Ready, Approved, Sent, Declined, Needs Review |
| Classification | Single select | Pricing Inquiry, Technical Question, General Info, Scheduling, Other |
| Agent Draft | Long text | |
| Final Response | Long text | |
| Created | Created time | Auto |
| Notes | Long text | |

---

## Slack Configuration

| Field | Description | Example Value | Where Used |
|-------|-------------|---------------|------------|
| Workspace Name | Slack workspace | "Apex Construction" | Reference only |
| Channel ID | Notification channel ID | "C0XXXXXXXXX" | Slack nodes |
| Channel Name | Human-readable name | "#email-agent-alerts" | Documentation |
| Bot Name | Slack app/bot name | "Updraft Email Bot" | Reference only |

**Finding Channel ID:**
1. Open Slack in browser
2. Navigate to the channel
3. Channel ID is in the URL: `slack.com/client/TXXXXX/CXXXXX`
4. The `CXXXXX` part is the Channel ID

---

## Gmail Configuration

| Field | Description | Example Value | Where Used |
|-------|-------------|---------------|------------|
| Monitored Email | Email address being monitored | "info@apexconstruction.com" | Gmail Trigger |
| Processed Label ID | Gmail label ID for processed emails | "Label_XXXXXXXXXX" | Mark as Processed nodes |
| Processed Label Name | Human-readable label name | "processed-by-agent" | Gmail setup |
| Excluded Labels | Labels to ignore (if any) | `[]` | Gmail Trigger filter |

**Finding Gmail Label ID:**
1. Open Gmail
2. Go to Settings > Labels
3. Create label "processed-by-agent" if needed
4. Use Gmail API or inspect network traffic to find ID
5. Or use the label name directly in N8N (some nodes support this)

---

## Variable Substitution Reference

When configuring a new client workflow, search for these placeholders and replace with actual values:

| Placeholder | Replace With | Configuration Matrix Section |
|-------------|--------------|------------------------------|
| `YOUR_GMAIL_CREDENTIAL_ID` | N8N credential ID | Integration Credentials > Gmail |
| `YOUR_GOOGLE_CALENDAR_CREDENTIAL_ID` | N8N credential ID | Integration Credentials > Calendar |
| `YOUR_ANTHROPIC_CREDENTIAL_ID` | N8N credential ID | Integration Credentials > Anthropic |
| `YOUR_AIRTABLE_CREDENTIAL_ID` | N8N credential ID | Integration Credentials > Airtable |
| `YOUR_SLACK_CREDENTIAL_ID` | N8N credential ID | Integration Credentials > Slack |
| `YOUR_AIRTABLE_BASE_ID` | Airtable base ID | Airtable Configuration > Base ID |
| `YOUR_AIRTABLE_TABLE_ID` | Airtable table ID | Airtable Configuration > Table ID |
| `YOUR_SLACK_CHANNEL_ID` | Slack channel ID | Slack Configuration > Channel ID |
| `YOUR_CALENDAR_EMAIL` | Primary calendar email | Calendar Configuration > Primary Calendar |
| `YOUR_CALENDAR_EMAIL_1` | First conflict calendar | Calendar Configuration > Conflict Calendar 1 |
| `YOUR_CALENDAR_EMAIL_2` | Second conflict calendar | Calendar Configuration > Conflict Calendar 2 |
| `YOUR_GMAIL_LABEL_ID` | Gmail label ID | Gmail Configuration > Processed Label ID |

---

## AI Prompt Variables

These placeholders appear in the AI Agent system prompt and need customization:

| Placeholder | Description | Example |
|-------------|-------------|---------|
| `{BUSINESS_DESCRIPTION}` | Company overview | "Commercial roofing contractor..." |
| `{SERVICES_LIST}` | Offered services | "Roof inspections, repairs..." |
| `{MEETING_TYPE}` | Meeting name | "Initial Consultation" |
| `{AVAILABILITY_DESCRIPTION}` | Available times | "Tuesday through Thursday, 10 AM to 3 PM Central" |
| `{DEFERRAL_TOPICS}` | Topics for humans | "pricing, contracts, legal matters" |
| `{TONE_GUIDELINES}` | Voice/tone notes | "Professional but friendly..." |
| `{COMPANY_NAME}` | Business name | "Apex Construction" |

---

## Quick Reference: Minimum Required Values

For a basic deployment, these are the absolute minimum values needed:

1. **Client Name**
2. **Monitored Email Address**
3. **Primary Calendar Email**
4. **Timezone**
5. **Working Days + Hours**
6. **Meeting Duration**
7. **Brief Business Description**
8. **Slack Channel ID** (or alternative notification method)
9. **All 5 credential IDs** (Gmail, Calendar, Anthropic, Airtable, Slack)
10. **Airtable Base + Table IDs**

---

## Related Documents

- [Deployment Checklist](./DEPLOYMENT_CHECKLIST.md) - Step-by-step deployment using these values
- [Client Onboarding Form](../templates/client-onboarding-form.md) - Template for gathering these values
- [Troubleshooting Playbook](./TROUBLESHOOTING_PLAYBOOK.md) - When configurations cause issues
