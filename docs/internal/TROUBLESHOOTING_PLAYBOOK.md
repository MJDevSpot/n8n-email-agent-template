# Troubleshooting Playbook

> Comprehensive guide for diagnosing and resolving Email Co-Pilot issues.

## How to Use This Document

When an issue occurs:
1. Identify the symptom in the index below
2. Follow the diagnostic steps in order
3. Apply the solution
4. Document any new findings in the relevant section

---

## Issue Index

1. [Gmail Trigger Not Firing](#1-gmail-trigger-not-firing)
2. [Calendar Times Not Showing](#2-calendar-times-not-showing)
3. [AI Agent Stuck in Loop](#3-ai-agent-stuck-in-loop)
4. [Booking Confirmation Not Parsed](#4-booking-confirmation-not-parsed)
5. [Calendar Event Not Created](#5-calendar-event-not-created)
6. [Slack Notifications Failing](#6-slack-notifications-failing)
7. [Airtable Write Errors](#7-airtable-write-errors)
8. [Email Response Contains Metadata](#8-email-response-contains-metadata)
9. [Wrong Times Offered](#9-wrong-times-offered)
10. ["Node hasn't been executed" Errors](#10-node-hasnt-been-executed-errors)
11. [Duplicate Email Processing](#11-duplicate-email-processing)
12. [Email Not Marked as Processed](#12-email-not-marked-as-processed)
13. [AI Response Quality Issues](#13-ai-response-quality-issues)
14. [OAuth Token Expired](#14-oauth-token-expired)
15. [Workflow Suddenly Stops](#15-workflow-suddenly-stops)

---

## 1. Gmail Trigger Not Firing

### Symptoms
- Workflow never activates when emails arrive
- No executions showing in N8N history
- New emails visible in Gmail but not processed

### Diagnostic Steps

**Step 1: Check Workflow is Active**
- Open workflow in N8N
- Look for green "Active" toggle in top right
- If gray/inactive, activate it

**Step 2: Verify Gmail OAuth Connection**
- Go to Credentials in N8N
- Click on Gmail credential
- Click "Test" button
- If error, click "Reconnect" and re-authenticate

**Step 3: Check Polling Configuration**
- Open Gmail Trigger node
- Verify Poll Times is configured (every 5 minutes recommended)
- Check that polling mode is enabled

**Step 4: Verify Label Filter**
- Check filter query: `-label:processed-by-agent`
- Ensure no typos in label name
- Verify the label exists in Gmail

**Step 5: Check for Gmail Errors**
- Send a test email to monitored inbox
- Check N8N execution history for failed attempts
- Look for 401 (auth) or 403 (permission) errors

### Solutions

| Root Cause | Solution |
|------------|----------|
| Workflow inactive | Activate workflow |
| OAuth expired | Reconnect credential |
| Label typo | Fix label name in filter |
| Label missing | Create label in Gmail |
| Permission denied | Re-authorize with full scopes |

### Prevention
- Monitor execution count weekly
- Set up alert for 0 executions in 24 hours

---

## 2. Calendar Times Not Showing

### Symptoms
- AI offers to schedule but no times appear in response
- Response says something like "I'll check my calendar" with no times
- FreeBusy API returns empty or error

### Diagnostic Steps

**Step 1: Test FreeBusy API Directly**
- Run workflow in test mode
- Check output of `check_calendar_availability` node
- Look for `calendars` object in response

**Step 2: Check OAuth Scopes**
- Go to N8N Credentials > Google Calendar
- Verify scopes include:
  - `https://www.googleapis.com/auth/calendar.readonly`
  - `https://www.googleapis.com/auth/calendar.events`

**Step 3: Verify Calendar IDs**
- Open `check_calendar_availability` node
- Check the `items` array in JSON body
- Calendar IDs must be exact email addresses

**Step 4: Check Calendar Sharing**
- Log into Google Calendar
- Verify the authenticated account can see all conflict calendars
- Check sharing permissions

**Step 5: Verify Timezone**
- Check timezone in FreeBusy request matches client timezone
- Verify `format_available_times` uses same timezone

### Solutions

| Root Cause | Solution |
|------------|----------|
| Missing OAuth scopes | Recreate credential with full scopes |
| Wrong calendar IDs | Update to correct email addresses |
| No calendar access | Share calendars with authenticated account |
| Timezone mismatch | Align all timezone references |
| API error | Check Google Calendar service status |

### Prevention
- Test calendar check during deployment
- Document all calendar IDs in Configuration Matrix

---

## 3. AI Agent Stuck in Loop

### Symptoms
- Workflow fails with "Maximum iterations exceeded"
- AI Agent node shows 10+ tool calls in execution
- Extremely long execution time before failure

### Diagnostic Steps

**Step 1: Review Execution Details**
- Open failed execution in N8N
- Expand AI Agent node output
- Count the number of tool calls

**Step 2: Check System Prompt**
- Open AI Agent node settings
- Look for explicit STOP instructions
- Check for conflicting directives

**Step 3: Review Tool Descriptions**
- Check each tool's description field
- Look for clarity on single-use

**Step 4: Analyze Input Email**
- What was the email that triggered this?
- Is it ambiguous or complex?
- Does it contain multiple scheduling requests?

### Solutions

**Add Explicit Stop Instructions to System Prompt:**
```
## CRITICAL: Tool Usage Rules
1. Call check_calendar_availability ONCE when someone requests a meeting
2. Call format_available_times ONCE after checking calendar
3. After formatting times, draft your response and STOP
4. DO NOT call any tool more than once per email
5. If unsure, respond without tools rather than looping
```

**Simplify Tool Descriptions:**
```
// Before (vague)
"Check calendar availability for scheduling"

// After (explicit)
"Check calendars for busy times. Use ONCE when someone requests a meeting. Returns raw busy time data that must be formatted with format_available_times."
```

### Prevention
- Always include tool usage limits in system prompts
- Test with complex scheduling requests during deployment

---

## 4. Booking Confirmation Not Parsed

### Symptoms
- User confirms a time but system doesn't book
- Email Router returns "new_inquiry" instead of "booking_confirmation"
- Parse Booking Details returns null or undefined

### Diagnostic Steps

**Step 1: Check Router Output**
- Run workflow in test mode
- Examine Email Router node output
- Check the `action` field value

**Step 2: Verify Thread Context**
- Check if Merge Thread Context includes previous emails
- Look for `thread_history` in output
- Router needs conversation context

**Step 3: Check JSON Parsing**
- Look at Parse Booking Details node
- Check if router output is wrapped in markdown code blocks
- Verify regex extraction is working

**Step 4: Review Confidence Level**
- If action is "booking_confirmation", check confidence
- "low" confidence goes to manual review path
- This may be working as designed

### Solutions

| Root Cause | Solution |
|------------|----------|
| Missing thread context | Check Get Thread Context node |
| Router misclassification | Update router system prompt |
| JSON wrapped in markdown | Update parsing to strip backticks |
| Low confidence | Expected behavior - review manually |

**Update Router Prompt to Better Detect Confirmations:**
```
Look for confirmation patterns:
- "That works for me"
- "Let's do [time]"
- "I'm available [time]"
- "Sounds good, [time] is great"
- "[Day] at [time] please"
- "See you [day/time]"
```

### Prevention
- Include common confirmation phrases in router prompt
- Test with variety of confirmation styles

---

## 5. Calendar Event Not Created

### Symptoms
- Booking detected as high confidence
- Workflow proceeds past confidence check
- No calendar event appears
- Error in Create Calendar Event node

### Diagnostic Steps

**Step 1: Check Node Error Message**
- Open failed execution
- Expand Create Calendar Event node
- Read error details

**Step 2: Verify Calendar Permissions**
- Check OAuth credential for Calendar
- Ensure scope includes `calendar.events`
- Test credential connection

**Step 3: Check Event Data**
- Look at input to Create Calendar Event
- Verify `selected_datetime` is valid ISO format
- Check timezone handling

**Step 4: Verify Calendar ID**
- Check which calendar is configured
- Ensure authenticated account can create events there

### Solutions

| Root Cause | Solution |
|------------|----------|
| Invalid datetime format | Fix Parse Booking Details output |
| Missing calendar permission | Re-authorize with events scope |
| Wrong calendar | Update calendar selection |
| Past datetime | Validation issue - check routing |

### Prevention
- Validate datetime format in Parse Booking Details
- Add error handling for past dates

---

## 6. Slack Notifications Failing

### Symptoms
- Workflow completes but no Slack messages appear
- Slack node shows error in execution
- Channel is empty

### Diagnostic Steps

**Step 1: Check Slack Credential**
- Go to N8N Credentials > Slack
- Click Test
- If failed, reconnect

**Step 2: Verify Bot is in Channel**
- Open Slack workspace
- Go to target channel
- Check if bot is a member

**Step 3: Check Channel ID**
- Verify channel ID starts with "C"
- Check for typos
- Confirm it matches actual channel

**Step 4: Check Permission Scopes**
- Bot needs `chat:write` scope
- Bot needs access to the channel

### Solutions

| Root Cause | Solution |
|------------|----------|
| Credential expired | Reconnect Slack OAuth |
| Bot not in channel | Run `/invite @botname` in channel |
| Wrong channel ID | Get correct ID from Slack URL |
| Insufficient permissions | Reinstall app with correct scopes |

### Prevention
- Document channel ID in Configuration Matrix
- Test Slack notification during deployment

---

## 7. Airtable Write Errors

### Symptoms
- Emails processed but not logged
- 422 or 403 errors in Airtable nodes
- Missing records in Airtable

### Diagnostic Steps

**Step 1: Check Error Message**
- Open failed execution
- Read Airtable node error details
- Note specific error code

**Step 2: Verify PAT is Valid**
- Personal Access Tokens expire
- Check if token was regenerated
- Test credential in N8N

**Step 3: Check Base/Table IDs**
- Verify base ID matches (starts with "app")
- Verify table ID matches (starts with "tbl")
- Check for copy/paste errors

**Step 4: Verify Field Mapping**
- Field names must match exactly (case-sensitive)
- Single select options must exist
- Field types must be compatible

### Solutions

| Root Cause | Solution |
|------------|----------|
| Expired PAT | Generate new token, update credential |
| Wrong base/table | Correct IDs from Airtable URL |
| Field name mismatch | Match exactly including case |
| Missing select option | Add option in Airtable |
| Rate limit (429) | Add delay between operations |

### Prevention
- Use Airtable's descriptive field IDs
- Validate schema during deployment

---

## 8. Email Response Contains Metadata

### Symptoms
- Sent emails include "[CLASSIFICATION: X]" tags
- Response includes "DRAFT EMAIL RESPONSE:" headers
- AI commentary visible in sent message

### Diagnostic Steps

**Step 1: Check Clean Agent Response**
- Open the node in test mode
- Verify regex patterns are working
- Check for unhandled patterns

**Step 2: Review AI Agent Output**
- See what patterns the AI is using
- Identify new patterns not being cleaned

### Solutions

**Update Clean Agent Response Code:**
```javascript
let response = item.json.output || '';

// Remove classification tags (multiple patterns)
response = response.replace(/\[CLASSIFICATION:.*?\]/gi, '');
response = response.replace(/CLASSIFICATION:.*$/gim, '');

// Remove headers
response = response.replace(/^DRAFT EMAIL RESPONSE:?\s*/i, '');
response = response.replace(/^TO:.*$/gim, '');
response = response.replace(/^FROM:.*$/gim, '');
response = response.replace(/^SUBJECT:.*$/gim, '');

// Remove dividers
response = response.replace(/^---+.*$/gm, '');

// Trim
response = response.trim();
```

**Update AI Agent Prompt:**
```
CRITICAL: Your response should ONLY contain the email body text.
- Do NOT include any headers (TO, FROM, SUBJECT, etc.)
- Do NOT include any preamble or commentary
- Start directly with the greeting
- End before any classification tags
```

### Prevention
- Test with various email types during deployment
- Regularly review sent emails for quality

---

## 9. Wrong Times Offered

### Symptoms
- Times offered don't match client's availability
- Weekends or blocked days appearing
- Wrong timezone displayed

### Diagnostic Steps

**Step 1: Check format_available_times Config**
- Open the node
- Verify `workingDays` array (0=Sun, 6=Sat)
- Verify `startHour` and `endHour`
- Check `bufferMinutes`

**Step 2: Verify Timezone Settings**
- Check timezone in config
- Check timezone in toLocaleString
- Compare to client's timezone

**Step 3: Check FreeBusy Response**
- Run calendar check in test mode
- Verify busy times are being returned
- Check all calendars are included

### Solutions

| Root Cause | Solution |
|------------|----------|
| Wrong working days | Update array to correct days |
| Wrong hours | Update startHour/endHour |
| Wrong timezone | Update all timezone references |
| Calendar not checked | Add missing calendar to items array |

### Prevention
- Document availability in Configuration Matrix
- Test across multiple scenarios

---

## 10. "Node hasn't been executed" Errors

### Symptoms
- Code node fails with "Node 'X' hasn't been executed"
- Data references return undefined
- Workflow stops at Code node

### Diagnostic Steps

**Step 1: Verify Node Name**
- Node names in `$('Node Name')` are case-sensitive
- Check for exact spelling including spaces
- Check for renamed nodes

**Step 2: Check Execution Order**
- Data can only reference nodes that run BEFORE current node
- Verify workflow connections
- Check for parallel branches

**Step 3: Check Branch Paths**
- If using IF node, opposite branch data isn't available
- Verify you're on the correct branch

### Solutions

**Use Defensive Coding Pattern:**
```javascript
// Safe node reference
const nodeData = $('Node Name').first();
if (!nodeData || !nodeData.json) {
  // Handle missing data
  return [{ json: { error: 'Required data not available' } }];
}
const value = nodeData.json.field;
```

**For Optional Data:**
```javascript
const thread = $input.first() && $input.first().json ? $input.first().json : {};
const messages = Array.isArray(thread.messages) ? thread.messages : [];
```

### Prevention
- Always use null checks in Code nodes
- Test with various input scenarios

---

## 11. Duplicate Email Processing

### Symptoms
- Same email processed multiple times
- Multiple Airtable records for one email
- Multiple responses sent

### Diagnostic Steps

**Step 1: Check Label Application**
- Is the email getting labeled as processed?
- Check Mark as Processed node execution
- Verify label exists in Gmail

**Step 2: Check Timing**
- Did workflow fail before marking processed?
- Is polling interval very short?

**Step 3: Check Label Filter**
- Verify trigger filter excludes processed label
- Check for typos

### Solutions

| Root Cause | Solution |
|------------|----------|
| Label not applied | Fix Mark as Processed node |
| Label wrong | Correct label name/ID |
| Workflow failed early | Mark processed earlier in flow |
| Very short poll | Increase poll interval |

### Prevention
- Consider marking processed earlier in workflow
- Add duplicate detection in Airtable (unique Email ID)

---

## 12. Email Not Marked as Processed

### Symptoms
- Emails processed repeatedly
- processed-by-agent label not appearing
- Mark as Processed node failing silently

### Diagnostic Steps

**Step 1: Check Node Execution**
- Does Mark as Processed show in execution?
- Check for errors

**Step 2: Verify Label ID**
- Label ID must be exact
- Try using label name instead of ID

**Step 3: Check Gmail Permissions**
- Must have gmail.modify scope
- Verify credential has this permission

### Solutions

| Root Cause | Solution |
|------------|----------|
| Wrong label ID | Update to correct ID or use name |
| Missing permission | Re-authorize with modify scope |
| Node not reached | Fix upstream workflow issues |

---

## 13. AI Response Quality Issues

### Symptoms
- Responses don't match business context
- Tone is off
- Wrong information provided

### Diagnostic Steps

**Step 1: Review System Prompt**
- Is business context complete?
- Are services accurately described?
- Is tone guidance clear?

**Step 2: Check Input Data**
- What email triggered the poor response?
- Is thread context being passed?

**Step 3: Review Model Selection**
- Verify Claude Sonnet 4.5 is selected
- Check for model configuration issues

### Solutions

**Improve System Prompt:**
- Add more business context
- Include example good responses
- Add explicit "do not" rules

**Improve Input:**
- Ensure full email body is passed
- Include thread history for context

### Prevention
- Test with 10+ email variations during deployment
- Regular quality review of responses

---

## 14. OAuth Token Expired

### Symptoms
- Multiple nodes failing with 401 errors
- "Token has been expired or revoked"
- Workflow was working, now failing

### Diagnostic Steps

**Step 1: Identify Which Credential**
- Check which service is failing
- Gmail and Calendar use different credentials

**Step 2: Check N8N Auto-Refresh**
- N8N should refresh automatically
- If not working, manual reconnect needed

**Step 3: Check Google Security**
- Has user revoked access?
- Are there security alerts in Google account?

### Solutions

1. Go to N8N Credentials
2. Click on affected credential
3. Click "Reconnect" or "Test"
4. Complete OAuth flow
5. Verify workflow works

### Prevention
- Monitor for auth errors weekly
- Set up alerts for credential failures

---

## 15. Workflow Suddenly Stops

### Symptoms
- Was working, now no executions
- No errors in N8N
- Emails not being processed

### Diagnostic Steps

**Step 1: Check Workflow Status**
- Is workflow still active?
- Check for recent deactivation

**Step 2: Check N8N Status**
- Is N8N platform operational?
- Check https://status.n8n.io

**Step 3: Check Credentials**
- Test all credentials
- Any recently expired?

**Step 4: Check Billing**
- N8N Cloud subscription active?
- API credits available?

### Solutions

| Root Cause | Solution |
|------------|----------|
| Workflow deactivated | Reactivate |
| N8N outage | Wait for resolution |
| Credential expired | Reconnect |
| Billing issue | Update payment |

---

## Escalation Path

If issue not resolved with this playbook:

1. **Check platform status pages:**
   - https://status.n8n.io
   - https://status.anthropic.com
   - https://www.google.com/appsstatus

2. **Contact Michael:** (205) 966-6053

3. **External support:**
   - N8N: support@n8n.io
   - Anthropic: support@anthropic.com

---

## Document Updates

When new issues are resolved:
1. Add to this playbook
2. Include symptoms, diagnostics, and solution
3. Update prevention section
4. Note in VERSION_CHANGELOG.md
