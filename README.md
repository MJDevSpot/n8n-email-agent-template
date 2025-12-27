# N8N Email Agent Template

An intelligent email automation workflow for N8N that uses AI to draft responses, schedule meetings, and manage inbound emails.

## Features

- **AI-Powered Email Responses**: Automatically drafts contextual replies using Anthropic's AI
- **Smart Meeting Scheduling**: Checks calendar availability and offers meeting times
- **Booking Automation**: Creates calendar events when recipients confirm a time
- **Thread Context Awareness**: Understands email conversation history
- **Low-Confidence Handling**: Routes uncertain bookings for human review
- **Multi-Channel Notifications**: Alerts via Slack when drafts are ready
- **Airtable Integration**: Tracks all emails and drafts for review

## Architecture

```
Gmail Trigger → Get Thread → AI Router
                                ↓
              ┌─────────────────┴─────────────────┐
              ↓                                   ↓
        New Inquiry                        Booking Reply
              ↓                                   ↓
        AI Agent                          Parse Booking
      (draft response)                          ↓
              ↓                      ┌──────────┴──────────┐
        Save to Airtable             ↓                    ↓
              ↓                 High Confidence      Low Confidence
        Notify Slack                 ↓                    ↓
              ↓               Create Event         Save + Notify
        Send Response         Send Confirmation    (Human Review)
              ↓                     ↓
        Mark Processed         Mark Processed
```

## Prerequisites

- N8N instance (cloud or self-hosted)
- Google Workspace account (Gmail + Calendar)
- Anthropic API account
- Airtable account
- Slack workspace

## Setup Instructions

### 1. Create Gmail Label

In Gmail, create a label called `processed-by-agent`. This prevents duplicate processing.

### 2. Set Up Airtable Base

Create an Airtable base with a table containing these fields:

| Field | Type |
|-------|------|
| Email ID | Single line text |
| From Email | Email |
| From Name | Single line text |
| Subject | Single line text |
| Body | Long text |
| Status | Single select: New, Draft Ready, Approved, Sent, Declined, Needs Review |
| Classification | Single select: Pricing Inquiry, Technical Question, General Info, Scheduling, Other |
| Agent Draft | Long text |
| Final Response | Long text |
| Notes | Long text |

### 3. Import Workflow

1. Open your N8N instance
2. Go to Workflows → Import from File
3. Select `workflow-template.json`

### 4. Configure Credentials

Create these credentials in N8N (Settings → Credentials):

| Credential | Required Scopes |
|------------|-----------------|
| Gmail OAuth2 | gmail.readonly, gmail.send, gmail.modify, gmail.labels |
| Google Calendar OAuth2 | calendar.readonly, calendar.events |
| Anthropic API | API key from console.anthropic.com |
| Airtable Personal Access Token | Read/write access to your base |
| Slack OAuth2 | chat:write, channels:read |

### 5. Update Placeholders

Search for `YOUR_` in the workflow and replace with your values:

| Placeholder | Description |
|-------------|-------------|
| `YOUR_GMAIL_CREDENTIAL_ID` | N8N credential ID for Gmail |
| `YOUR_GOOGLE_CALENDAR_CREDENTIAL_ID` | N8N credential ID for Google Calendar |
| `YOUR_ANTHROPIC_CREDENTIAL_ID` | N8N credential ID for Anthropic |
| `YOUR_AIRTABLE_CREDENTIAL_ID` | N8N credential ID for Airtable |
| `YOUR_SLACK_CREDENTIAL_ID` | N8N credential ID for Slack |
| `YOUR_AIRTABLE_BASE_ID` | Airtable base ID (starts with `app`) |
| `YOUR_AIRTABLE_TABLE_ID` | Airtable table ID (starts with `tbl`) |
| `YOUR_SLACK_CHANNEL_ID` | Slack channel ID (starts with `C`) |
| `YOUR_CALENDAR_EMAIL` | Calendar to book meetings on |
| `YOUR_CALENDAR_EMAIL_1`, `_2` | Calendars to check for conflicts |
| `YOUR_GMAIL_LABEL_ID` | Gmail label ID for `processed-by-agent` |

### 6. Customize AI Prompts

Edit the **AI Agent** node's system message to reflect your:
- Business description and services
- Scheduling availability
- Communication tone
- Special instructions

### 7. Customize Availability

In the **format_available_times** node, update the config:

```javascript
const config = {
  workingDays: [2, 3, 4],  // 0=Sun through 6=Sat
  startHour: 10,           // Start time (24h)
  endHour: 15,             // End time (24h)
  meetingDuration: 30,     // Minutes
  bufferMinutes: 15        // Buffer between meetings
};
```

### 8. Update Email Signatures

Edit these nodes to customize the email sign-off:
- **Send Email Response**
- **Send Booking Confirmation Email**

### 9. Test the Workflow

1. Use N8N's "Test Workflow" feature with pinned data
2. Send a test email to the monitored inbox
3. Verify the workflow triggers and processes correctly

### 10. Activate

Once tested, activate the workflow for production use.

## Workflow Paths

### New Inquiry Path
Email → AI drafts response → Saves to Airtable → Notifies Slack → Sends reply → Marks processed

### High-Confidence Booking Path
Email confirms time → Creates calendar event → Sends confirmation → Marks processed

### Low-Confidence Booking Path
Unclear booking → Saves to Airtable as "Needs Review" → Notifies Slack → Marks processed

## Customization Tips

### Adding New Email Categories
1. Update the classification options in the AI Agent prompt
2. Add handling logic in the Route By Action node
3. Update Airtable field options

### Changing AI Model
Update both Anthropic Chat Model nodes to use a different model.

### Removing Integrations
- **No Airtable**: Delete Save to Airtable nodes, connect directly to next step
- **No Slack**: Delete Notify Slack nodes, connect directly to next step

## Troubleshooting

### "Node hasn't been executed" Error
Ensure nodes run sequentially, not in parallel from the trigger.

### Gmail Label Not Working
Verify the label exists and the label ID is correct (use Gmail API to find ID).

### Calendar Not Showing Available Times
Check that all calendar email addresses in `check_calendar_availability` are correct.

See [Troubleshooting Playbook](docs/internal/TROUBLESHOOTING_PLAYBOOK.md) for comprehensive issue resolution.

## Documentation

Complete documentation is available in the `/docs` folder:

| Category | Documents |
|----------|-----------|
| **Client-Facing** | [Capability Overview](docs/client-facing/CAPABILITY_OVERVIEW.md) · [Integration Requirements](docs/client-facing/INTEGRATION_REQUIREMENTS.md) · [SLA & Support](docs/client-facing/SLA_AND_SUPPORT.md) · [ROI Calculator](docs/client-facing/ROI_CALCULATOR.md) |
| **Internal** | [Configuration Matrix](docs/internal/CONFIGURATION_MATRIX.md) · [Deployment Checklist](docs/internal/DEPLOYMENT_CHECKLIST.md) · [Troubleshooting](docs/internal/TROUBLESHOOTING_PLAYBOOK.md) |
| **Technical** | [Architecture](docs/technical/ARCHITECTURE_OVERVIEW.md) · [Node Reference](docs/technical/NODE_REFERENCE.md) · [API Dependencies](docs/technical/API_DEPENDENCIES.md) · [Disaster Recovery](docs/technical/DISASTER_RECOVERY.md) |
| **Templates** | [Client Onboarding Form](docs/templates/client-onboarding-form.md) · [Handoff Document](docs/templates/handoff-document-template.md) |

## License

MIT License - Feel free to use and modify for your needs.
