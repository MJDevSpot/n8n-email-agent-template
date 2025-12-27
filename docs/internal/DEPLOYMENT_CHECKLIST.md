# Email Agent Deployment Checklist

> Step-by-step guide to deploy a new client instance in under 2 hours.

**Target Time:** 2 hours or less
**Prerequisites:** Completed [Client Onboarding Form](../templates/client-onboarding-form.md)
**Reference:** [Configuration Matrix](./CONFIGURATION_MATRIX.md) for all variable values

---

## Pre-Deployment Requirements

Before starting deployment, verify:

- [ ] Client onboarding form completed and reviewed
- [ ] Client has Google Workspace (Gmail + Calendar)
- [ ] Client can provide admin access OR authorize OAuth apps
- [ ] Slack workspace access confirmed (or alternative notification method)
- [ ] Airtable workspace ready (or will create)
- [ ] Anthropic API credits available

**Stop here if any requirement is missing.** Contact client to resolve before proceeding.

---

## Phase 1: Credential Setup (30 minutes)

### 1.1 Gmail OAuth Configuration

**Time Estimate:** 10 minutes

1. [ ] Log into N8N Cloud instance
2. [ ] Go to **Credentials** > **Add Credential**
3. [ ] Search for "Gmail OAuth2"
4. [ ] Name the credential: `[ClientName] Gmail`
5. [ ] Click **Sign in with Google**
6. [ ] Log in with client's monitored email address (e.g., info@company.com)
7. [ ] Grant all requested permissions:
   - View and manage email
   - Send email on behalf
   - Manage labels
8. [ ] Click **Save**
9. [ ] Test connection by clicking **Test**

**Expected Result:** Green checkmark, "Connection successful"

**If Error:** Check that the Google account has 2FA enabled and less secure apps aren't blocked

---

### 1.2 Google Calendar OAuth Configuration

**Time Estimate:** 5 minutes

1. [ ] Go to **Credentials** > **Add Credential**
2. [ ] Search for "Google Calendar OAuth2"
3. [ ] Name the credential: `[ClientName] Calendar`
4. [ ] Click **Sign in with Google**
5. [ ] Log in with the same account or primary calendar owner
6. [ ] Grant calendar permissions:
   - View and edit calendars
   - View free/busy information
7. [ ] Click **Save**
8. [ ] Test connection

**Expected Result:** Green checkmark, "Connection successful"

**Note:** For FreeBusy API to work with multiple calendars, the authenticated account must have at least read access to all conflict calendars.

---

### 1.3 Anthropic API Key Configuration

**Time Estimate:** 3 minutes

1. [ ] Go to **Credentials** > **Add Credential**
2. [ ] Search for "Anthropic"
3. [ ] Name the credential: `Anthropic account` (can be shared) OR `[ClientName] Anthropic` (dedicated)
4. [ ] Enter API key from console.anthropic.com
5. [ ] Click **Save**
6. [ ] Test connection

**Expected Result:** Green checkmark

**Note:** If using shared Anthropic account, verify sufficient credits for new client volume.

---

### 1.4 Airtable Base Setup

**Time Estimate:** 10 minutes

#### Create Base (if new):

1. [ ] Log into Airtable
2. [ ] Create new base: `[ClientName] Email Agent`
3. [ ] Create table: `Inbound Emails`
4. [ ] Add required fields:

| Field Name | Type | Configuration |
|------------|------|---------------|
| Email ID | Single line text | |
| From Email | Email | |
| From Name | Single line text | |
| Subject | Single line text | |
| Body | Long text | |
| Status | Single select | Options: New, Draft Ready, Approved, Sent, Declined, Needs Review |
| Classification | Single select | Options: Pricing Inquiry, Technical Question, General Info, Scheduling, Other |
| Agent Draft | Long text | |
| Final Response | Long text | |
| Notes | Long text | |

5. [ ] Note the Base ID from URL: `airtable.com/appXXXXXXX/...`
6. [ ] Note the Table ID from URL: `airtable.com/appXXX/tblXXXXXXX/...`

#### Create N8N Credential:

7. [ ] In Airtable, go to **Account** > **Developer Hub** > **Personal Access Tokens**
8. [ ] Create new token with scopes:
   - `data.records:read`
   - `data.records:write`
   - Access to the specific base
9. [ ] Copy the token
10. [ ] In N8N, go to **Credentials** > **Add Credential**
11. [ ] Search for "Airtable Personal Access Token"
12. [ ] Name: `[ClientName] Airtable`
13. [ ] Paste token
14. [ ] Save and test

**Expected Result:** Connection successful, can see base in dropdown

---

### 1.5 Slack Integration

**Time Estimate:** 5 minutes

#### Create Channel:

1. [ ] In client's Slack workspace, create channel: `#email-agent-alerts`
2. [ ] Note the Channel ID from URL or channel details

#### Create N8N Credential:

3. [ ] In N8N, go to **Credentials** > **Add Credential**
4. [ ] Search for "Slack OAuth2"
5. [ ] Name: `[ClientName] Slack`
6. [ ] Click **Connect**
7. [ ] Authorize in client's Slack workspace
8. [ ] Grant permissions:
   - Post to channels
   - View channels
9. [ ] Save and test

**Expected Result:** Can send test message to channel

---

### 1.6 Gmail Label Setup

**Time Estimate:** 2 minutes

1. [ ] Open Gmail for the monitored email account
2. [ ] Go to **Settings** (gear icon) > **See all settings** > **Labels**
3. [ ] Scroll to bottom, click **Create new label**
4. [ ] Name: `processed-by-agent`
5. [ ] Click **Create**
6. [ ] Note the label exists (ID will be found automatically or via API)

**Expected Result:** Label visible in Gmail sidebar

---

## Phase 2: Workflow Import & Configuration (45 minutes)

### 2.1 Import Base Workflow

**Time Estimate:** 5 minutes

1. [ ] In N8N, go to **Workflows**
2. [ ] Click **Add Workflow** > **Import from File**
3. [ ] Select `workflow-export-template.json` from templates folder
4. [ ] Rename workflow: `[ClientName] Email Agent`
5. [ ] Save workflow (do not activate yet)

**Expected Result:** Workflow imported with all nodes visible

---

### 2.2 Update Credential References

**Time Estimate:** 15 minutes

Update credentials in each node. Work through systematically:

#### Gmail Nodes (5 nodes):

| Node Name | Credential Field | Set To |
|-----------|-----------------|--------|
| Gmail Trigger | gmailOAuth2 | [ClientName] Gmail |
| Get Full Email | gmailOAuth2 | [ClientName] Gmail |
| Get Thread Context | gmailOAuth2 | [ClientName] Gmail |
| Send Email Response | gmailOAuth2 | [ClientName] Gmail |
| Mark as Processed | gmailOAuth2 | [ClientName] Gmail |
| Send Booking Confirmation Email | gmailOAuth2 | [ClientName] Gmail |
| Mark Booking Processed | gmailOAuth2 | [ClientName] Gmail |
| Mark Low Confidence Processed | gmailOAuth2 | [ClientName] Gmail |

1. [ ] Gmail Trigger - credential updated
2. [ ] Get Full Email - credential updated
3. [ ] Get Thread Context - credential updated
4. [ ] Send Email Response - credential updated
5. [ ] Mark as Processed - credential updated
6. [ ] Send Booking Confirmation Email - credential updated
7. [ ] Mark Booking Processed - credential updated
8. [ ] Mark Low Confidence Processed - credential updated

#### Calendar Nodes (2 nodes):

| Node Name | Credential Field | Set To |
|-----------|-----------------|--------|
| check_calendar_availability | googleCalendarOAuth2Api | [ClientName] Calendar |
| Create Calendar Event | googleCalendarOAuth2Api | [ClientName] Calendar |

1. [ ] check_calendar_availability - credential updated
2. [ ] Create Calendar Event - credential updated

#### Anthropic Nodes (2 nodes):

| Node Name | Credential Field | Set To |
|-----------|-----------------|--------|
| Anthropic Chat Model | anthropicApi | Anthropic account |
| Anthropic Chat Model1 | anthropicApi | Anthropic account |

1. [ ] Anthropic Chat Model - credential updated
2. [ ] Anthropic Chat Model1 - credential updated

#### Airtable Nodes (2 nodes):

| Node Name | Credential Field | Set To |
|-----------|-----------------|--------|
| Save to Airtable | airtableTokenApi | [ClientName] Airtable |
| Save Low Confidence to Airtable | airtableTokenApi | [ClientName] Airtable |

1. [ ] Save to Airtable - credential updated
2. [ ] Save Low Confidence to Airtable - credential updated

#### Slack Nodes (2 nodes):

| Node Name | Credential Field | Set To |
|-----------|-----------------|--------|
| Notify Slack | slackOAuth2Api | [ClientName] Slack |
| Notify Low Confidence | slackOAuth2Api | [ClientName] Slack |

1. [ ] Notify Slack - credential updated
2. [ ] Notify Low Confidence - credential updated

---

### 2.3 Configure Client-Specific Variables

**Time Estimate:** 15 minutes

#### Airtable Configuration:

1. [ ] Open **Save to Airtable** node
2. [ ] Set Base: Select client's base from dropdown
3. [ ] Set Table: Select "Inbound Emails" table
4. [ ] Verify field mappings match Airtable schema
5. [ ] Repeat for **Save Low Confidence to Airtable** node

#### Slack Configuration:

1. [ ] Open **Notify Slack** node
2. [ ] Set Channel: Select `#email-agent-alerts` from dropdown
3. [ ] Update Airtable URL in message template to client's base URL
4. [ ] Repeat for **Notify Low Confidence** node

#### Gmail Label Configuration:

1. [ ] Open **Mark as Processed** node
2. [ ] Update Label ID to client's `processed-by-agent` label
   - If using label name: enter `processed-by-agent`
   - If using label ID: enter `Label_XXXXXXXXXX`
3. [ ] Repeat for **Mark Booking Processed** and **Mark Low Confidence Processed**

#### Calendar Configuration:

1. [ ] Open **check_calendar_availability** node
2. [ ] Update the JSON body with client's calendar emails:
```json
{
  "timeMin": "{{ new Date().toISOString() }}",
  "timeMax": "{{ new Date(Date.now() + 30*24*60*60*1000).toISOString() }}",
  "timeZone": "[CLIENT_TIMEZONE]",
  "items": [
    {"id": "[PRIMARY_CALENDAR_EMAIL]"},
    {"id": "[CONFLICT_CALENDAR_1]"},
    {"id": "[CONFLICT_CALENDAR_2]"}
  ]
}
```

3. [ ] Open **Create Calendar Event** node
4. [ ] Set Calendar to client's primary calendar email
5. [ ] Update event title template if needed
6. [ ] Update event description template if needed

#### Availability Configuration:

1. [ ] Open **format_available_times** node
2. [ ] Update the config object:
```javascript
const config = {
  workingDays: [/* CLIENT WORKING DAYS */],
  startHour: /* CLIENT START HOUR */,
  endHour: /* CLIENT END HOUR */,
  meetingDuration: /* MEETING DURATION */,
  bufferMinutes: /* BUFFER MINUTES */
};
```
3. [ ] Update timezone in toLocaleString call

---

## Phase 3: AI Prompt Customization (30 minutes)

### 3.1 Update AI Agent System Prompt

**Time Estimate:** 20 minutes

1. [ ] Open **AI Agent** node
2. [ ] Click on **Options** > **System Message**
3. [ ] Update the following sections:

#### Business Context Section:
```
You are the [CLIENT_NAME] Email Assistant.

BUSINESS CONTEXT:
[CLIENT_BUSINESS_DESCRIPTION]

SERVICES:
[CLIENT_SERVICES_LIST]
```

#### Scheduling Section:
```
SCHEDULING:
The [CLIENT_NAME] team is available for [MEETING_DURATION]-minute [MEETING_TYPE_NAME] calls on [WORKING_DAYS] from [START_HOUR] to [END_HOUR] [TIMEZONE].
```

#### Tone Section:
```
TONE:
[CLIENT_TONE_GUIDELINES]
```

#### Deferral Section:
```
DEFERRAL TOPICS:
When asked about [DEFERRAL_TOPICS], respond: "I'd recommend discussing that directly with our team. Would you like to schedule a call?"
```

4. [ ] Save changes

### 3.2 Update Email Router System Prompt

**Time Estimate:** 5 minutes

1. [ ] Open **Email Router** node
2. [ ] Verify system prompt references are generic (no client-specific changes usually needed)
3. [ ] Save if any changes made

### 3.3 Update Email Signatures

**Time Estimate:** 5 minutes

1. [ ] Open **Send Email Response** node
2. [ ] Update signature block:
```html
<p>Best regards,<br>
<strong>[CLIENT_SENDER_NAME]</strong><br>
<em>on behalf of [CLIENT_COMPANY_NAME]</em></p>
```

3. [ ] Open **Send Booking Confirmation Email** node
4. [ ] Update signature block to match
5. [ ] Save changes

---

## Phase 4: Testing & Validation (15 minutes)

### 4.1 Test Email Flow

**Time Estimate:** 5 minutes

1. [ ] Send a test email to the monitored address from a personal account
2. [ ] Subject: "Test inquiry - please ignore"
3. [ ] Body: "Hi, I'm interested in learning more about your services. Can we schedule a call?"
4. [ ] Wait for workflow to trigger (up to 5 minutes based on polling interval)
5. [ ] Verify in N8N execution history:
   - [ ] Gmail Trigger fired
   - [ ] Email Router classified as "new_inquiry"
   - [ ] AI Agent generated response
   - [ ] Airtable record created
   - [ ] Slack notification sent

**Expected Result:** Complete execution, draft ready in Airtable

### 4.2 Verify Calendar Check

**Time Estimate:** 3 minutes

1. [ ] Check AI Agent execution output
2. [ ] Verify calendar availability was checked
3. [ ] Verify available times were formatted
4. [ ] Confirm times offered match client's availability settings

**Expected Result:** 3-4 available time slots in response

### 4.3 Test Booking Flow

**Time Estimate:** 5 minutes

1. [ ] Reply to the agent's email with a time selection
2. [ ] Example: "Tuesday at 2 PM works for me"
3. [ ] Wait for workflow to process
4. [ ] Verify:
   - [ ] Email Router classified as "booking_confirmation"
   - [ ] Confidence level is "high"
   - [ ] Calendar event created
   - [ ] Confirmation email sent
   - [ ] Email marked as processed

**Expected Result:** Calendar event visible, confirmation email received

### 4.4 Confirm Notifications

**Time Estimate:** 2 minutes

1. [ ] Check Slack channel for notifications
2. [ ] Verify notification includes:
   - [ ] Sender information
   - [ ] Subject line
   - [ ] Link to Airtable
3. [ ] Check Airtable for records

**Expected Result:** All notifications received, records created

---

## Phase 5: Activation & Handoff

### 5.1 Activate Workflow

**Time Estimate:** 1 minute

1. [ ] In N8N, toggle workflow to **Active**
2. [ ] Verify green "Active" indicator shows
3. [ ] Check execution history starts populating (if emails arrive)

**Expected Result:** Workflow active and processing

### 5.2 Document Client Settings

**Time Estimate:** 5 minutes

1. [ ] Fill out [Handoff Document Template](../templates/handoff-document-template.md)
2. [ ] Include:
   - [ ] Airtable dashboard URL
   - [ ] Slack channel name
   - [ ] Support contact information
   - [ ] Summary of configuration
3. [ ] Save completed handoff document to client folder

### 5.3 Send Welcome Email to Client

**Time Estimate:** 5 minutes

1. [ ] Compose email to primary contact
2. [ ] Include:
   - [ ] Confirmation that Email Co-Pilot is live
   - [ ] Link to Airtable dashboard
   - [ ] Slack channel confirmation
   - [ ] Quick start guide
   - [ ] Support contact information
3. [ ] Attach completed handoff document

---

## Post-Deployment Verification (Day 2)

Within 24 hours of deployment:

- [ ] Review N8N execution history for any errors
- [ ] Check Airtable for processed emails
- [ ] Verify no duplicate processing occurred
- [ ] Confirm label is being applied correctly
- [ ] Review AI response quality in Airtable
- [ ] Check with client for any issues

---

## Deployment Checklist Summary

| Phase | Est. Time | Status |
|-------|-----------|--------|
| Pre-Deployment Requirements | 5 min | [ ] |
| Phase 1: Credential Setup | 30 min | [ ] |
| Phase 2: Workflow Configuration | 45 min | [ ] |
| Phase 3: AI Prompt Customization | 30 min | [ ] |
| Phase 4: Testing & Validation | 15 min | [ ] |
| Phase 5: Activation & Handoff | 15 min | [ ] |
| **Total** | **~2 hours** | |

---

## Related Documents

- [Configuration Matrix](./CONFIGURATION_MATRIX.md) - All configuration values
- [Troubleshooting Playbook](./TROUBLESHOOTING_PLAYBOOK.md) - If issues arise
- [Client Onboarding Form](../templates/client-onboarding-form.md) - Information gathering
- [Handoff Document Template](../templates/handoff-document-template.md) - Client documentation
