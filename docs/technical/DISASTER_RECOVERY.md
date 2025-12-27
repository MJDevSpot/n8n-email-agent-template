# Disaster Recovery & Business Continuity

> Procedures for handling system failures and service restoration.

## Recovery Time Objectives

| Scenario | RTO | RPO | Priority |
|----------|-----|-----|----------|
| Complete workflow failure | 2 hours | 0 (no data loss) | Critical |
| Single service outage | 4 hours | 0 | High |
| Credential expiration | 1 hour | 0 | High |
| AI degradation | 24 hours | 0 | Medium |
| Logging failure | 48 hours | 24 hours | Low |

**RTO** = Recovery Time Objective (max acceptable downtime)
**RPO** = Recovery Point Objective (max acceptable data loss)

---

## Failure Scenarios

### 1. N8N Cloud Outage

**Symptoms:**
- No workflow executions
- N8N dashboard inaccessible
- Status page shows outage

**Impact:**
- All email processing stops
- No responses sent
- Emails queue in Gmail (not lost)

**Response:**

| Step | Action | Time |
|------|--------|------|
| 1 | Check https://status.n8n.io | Immediate |
| 2 | Notify client of delay | 15 min |
| 3 | Monitor status page | Ongoing |
| 4 | Verify workflow activates post-recovery | Upon resolution |
| 5 | Review any missed emails | 30 min post-recovery |

**Mitigation:**
- Emails remain in Gmail inbox
- Processed label prevents duplicates
- System catches up automatically

---

### 2. Gmail OAuth Token Revoked

**Symptoms:**
- 401 errors on Gmail operations
- Workflow fails at trigger or send
- No new executions starting

**Impact:**
- Cannot read new emails
- Cannot send responses
- Calendar and other services unaffected

**Response:**

| Step | Action | Time |
|------|--------|------|
| 1 | Identify credential failure | Immediate |
| 2 | Contact client for re-authorization | 30 min |
| 3 | Re-establish OAuth connection | 15 min |
| 4 | Test credential | 5 min |
| 5 | Verify workflow active | 5 min |

**Prevention:**
- Weekly credential health checks
- Monitor for auth errors in executions
- Document re-authorization process for clients

---

### 3. Anthropic API Failure

**Symptoms:**
- 500/503 errors from Anthropic
- AI Agent node failing
- Responses not generated

**Impact:**
- Emails received but not responded to
- Manual intervention required
- Logging continues

**Response:**

| Step | Action | Time |
|------|--------|------|
| 1 | Check https://status.anthropic.com | Immediate |
| 2 | If temporary, wait for resolution | Monitor |
| 3 | If extended, notify client | 30 min |
| 4 | Consider manual response queue | 1 hour |
| 5 | Process backlog post-recovery | Upon resolution |

**Workaround Options:**
1. Manual response drafting
2. Template-based responses
3. Pause workflow until resolved

---

### 4. Calendar Integration Failure

**Symptoms:**
- FreeBusy returns errors
- No availability times in responses
- Booking flow broken

**Impact:**
- Cannot offer specific times
- Scheduling becomes manual
- General responses still work

**Response:**

| Step | Action | Time |
|------|--------|------|
| 1 | Verify Calendar credential | Immediate |
| 2 | Check calendar sharing settings | 10 min |
| 3 | Re-authorize if needed | 15 min |
| 4 | Test availability check | 5 min |

**Workaround:**
- AI can suggest "let me check availability and get back to you"
- Manual scheduling until resolved

---

### 5. Airtable Logging Failure

**Symptoms:**
- Airtable nodes show errors
- No records appearing in base
- 401/422 errors in execution

**Impact:**
- No audit trail
- Email processing continues
- Notifications still work

**Response:**

| Step | Action | Time |
|------|--------|------|
| 1 | Check Airtable PAT validity | Immediate |
| 2 | Verify base/table IDs | 10 min |
| 3 | Regenerate PAT if expired | 15 min |
| 4 | Backfill missed records | 1-2 hours |

**Recovery:**
- Execution history contains all data
- Can export from N8N and import to Airtable
- No data lost, just delayed logging

---

### 6. Slack Notifications Failing

**Symptoms:**
- No messages in Slack channel
- Slack nodes show errors
- Bot shows offline

**Impact:**
- No real-time alerts
- Email processing unaffected
- Can check Airtable for status

**Response:**

| Step | Action | Time |
|------|--------|------|
| 1 | Check Slack credential | Immediate |
| 2 | Verify bot is in channel | 5 min |
| 3 | Reinstall if needed | 15 min |
| 4 | Send test message | 5 min |

**Workaround:**
- Check Airtable directly
- Enable email notifications as backup

---

### 7. Workflow Deactivated

**Symptoms:**
- Zero executions
- Workflow shows "Inactive"
- No trigger activity

**Impact:**
- All processing stops
- Emails queue in inbox

**Response:**

| Step | Action | Time |
|------|--------|------|
| 1 | Identify workflow is inactive | Immediate |
| 2 | Investigate cause | 10 min |
| 3 | Fix any underlying issues | Varies |
| 4 | Reactivate workflow | 1 min |
| 5 | Process backlogged emails | 30 min |

**Common Causes:**
- N8N subscription issue
- Manual deactivation
- Credential failure cascade
- Execution limit reached

---

## Recovery Procedures

### Complete System Restore

If workflow must be rebuilt from scratch:

**Phase 1: Infrastructure (30 min)**
1. Access N8N Cloud
2. Create new workflow or import backup
3. Verify workflow structure

**Phase 2: Credentials (45 min)**
1. Gmail OAuth
   - Create connection
   - Authorize with client
   - Verify scopes
2. Google Calendar OAuth
   - Create connection
   - Authorize with client
   - Verify scopes
3. Anthropic API
   - Enter API key
   - Verify connection
4. Airtable
   - Generate new PAT
   - Configure base access
5. Slack
   - Install app
   - Authorize workspace
   - Select channel

**Phase 3: Configuration (30 min)**
1. Load client configuration from Matrix
2. Update all client-specific values
3. Configure AI prompts

**Phase 4: Testing (30 min)**
1. Send test email
2. Verify trigger fires
3. Check classification
4. Verify response draft
5. Test calendar integration
6. Confirm Slack notification
7. Verify Airtable logging

**Phase 5: Activation (5 min)**
1. Activate workflow
2. Monitor first live email
3. Confirm all systems operational

---

### Credential Rotation Emergency

If credentials are compromised:

| Step | Action | Owner | Time |
|------|--------|-------|------|
| 1 | Revoke compromised credential immediately | Admin | 5 min |
| 2 | Assess scope of exposure | Admin | 15 min |
| 3 | Generate new credentials | Admin | 15 min |
| 4 | Update in N8N | Admin | 10 min |
| 5 | Verify workflow operation | Admin | 10 min |
| 6 | Document incident | Admin | 30 min |
| 7 | Notify client if required | Admin | 15 min |

---

## Backup & Export

### What to Back Up

| Item | Location | Backup Method | Frequency |
|------|----------|---------------|-----------|
| Workflow JSON | N8N Cloud | Export to file | After changes |
| Configuration Matrix | Internal docs | Version control | After changes |
| System prompts | Workflow nodes | Document separately | After changes |
| Client info | Configuration docs | Encrypted storage | After changes |

### Export Workflow

1. Open workflow in N8N
2. Click three-dot menu
3. Select "Download"
4. Save JSON file with date

### Backup Storage

- Store backups in Git repository
- Client-specific configs in encrypted storage
- Retain last 5 versions minimum

---

## Communication Templates

### Client Notification - Service Disruption

```
Subject: Email Co-Pilot - Temporary Service Notice

[Client Name],

We've detected an issue with your Email Co-Pilot service:

Issue: [Brief description]
Impact: [What's affected]
Started: [Time]
Estimated Resolution: [Time or "Investigating"]

What's happening:
- Emails are being queued safely in your inbox
- No data has been lost
- We're actively working on resolution

Next update: [Time]

If you need immediate assistance: (205) 966-6053

-- UpdraftRev Support
```

### Client Notification - Service Restored

```
Subject: Email Co-Pilot - Service Restored

[Client Name],

Good news - your Email Co-Pilot is back to normal operation.

Issue: [Brief description]
Duration: [Start] to [End]
Resolution: [What was fixed]

What happened:
- [X] emails queued during outage have been processed
- All systems verified operational
- No data was lost

If you notice any issues, please reach out.

-- UpdraftRev Support
```

---

## Incident Response Checklist

### Detection
- [ ] Identify symptom
- [ ] Determine scope
- [ ] Check service status pages
- [ ] Review recent executions

### Assessment
- [ ] Identify root cause
- [ ] Determine impact level
- [ ] Estimate resolution time
- [ ] Decide on communication

### Response
- [ ] Notify client if needed
- [ ] Implement fix or workaround
- [ ] Test resolution
- [ ] Verify full operation

### Recovery
- [ ] Process any backlogged items
- [ ] Verify all integrations
- [ ] Update monitoring
- [ ] Document incident

### Post-Incident
- [ ] Write incident summary
- [ ] Update playbook if needed
- [ ] Implement prevention measures
- [ ] Schedule follow-up review

---

## Monitoring Recommendations

### Daily Checks
- [ ] At least one execution in last 24 hours
- [ ] No error executions in N8N
- [ ] Slack notifications arriving

### Weekly Checks
- [ ] All credentials test successfully
- [ ] Execution count normal for volume
- [ ] Airtable records matching emails

### Monthly Checks
- [ ] Review execution trends
- [ ] Check API usage vs limits
- [ ] Verify backup is current
- [ ] Test recovery procedure

---

## Emergency Contacts

### Internal
- **Michael Johnson:** (205) 966-6053
- **Email:** michael@updraftrev.com

### Service Providers
- **N8N Support:** support@n8n.io
- **Anthropic:** support@anthropic.com
- **Google Cloud:** Via Cloud Console

### Status Pages
- N8N: https://status.n8n.io
- Anthropic: https://status.anthropic.com
- Google: https://www.google.com/appsstatus
- Airtable: https://status.airtable.com
- Slack: https://status.slack.com

---

## Related Documents

- [Troubleshooting Playbook](../internal/TROUBLESHOOTING_PLAYBOOK.md) - Issue resolution
- [API Dependencies](./API_DEPENDENCIES.md) - Service details
- [Configuration Matrix](../internal/CONFIGURATION_MATRIX.md) - Client config
- [Deployment Checklist](../internal/DEPLOYMENT_CHECKLIST.md) - Full setup
