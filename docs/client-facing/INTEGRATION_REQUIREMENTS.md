# Integration Requirements

> What you need to provide for Email Co-Pilot setup.

## Overview

Setting up your Email Co-Pilot requires access to a few systems. This document outlines exactly what we need, what permissions are required, and how to provide them.

**Time to gather everything:** 15-20 minutes

---

## Required: Google Workspace Access

### Gmail

We need access to the email inbox you want the Co-Pilot to monitor.

**What we need:**
- [ ] The email address to monitor (e.g., info@yourcompany.com)
- [ ] Ability to authorize our application with that account

**Permissions we'll request:**
| Permission | Why We Need It |
|------------|---------------|
| Read emails | To see incoming messages |
| Send emails | To send responses on your behalf |
| Manage labels | To track which emails have been processed |

**How authorization works:**
1. During setup, you'll see a Google sign-in screen
2. Log in with the email account to be monitored
3. Review and approve the permission request
4. Google creates a secure connection—we never see your password

---

### Google Calendar

We need access to check your availability and create meeting events.

**What we need:**
- [ ] Primary calendar where meetings should be booked
- [ ] Any additional calendars to check for conflicts

**Common configurations:**
- Single owner: Just your primary calendar
- Shared calendars: Your calendar + team calendar
- Multiple personal calendars: Work + personal to prevent conflicts

**Permissions we'll request:**
| Permission | Why We Need It |
|------------|---------------|
| View free/busy | To check availability across calendars |
| Create events | To book meetings when confirmed |
| Send invitations | To notify attendees |

**Note:** We can only see free/busy information on conflict calendars—not event details.

---

## Required: Business Information

Help us configure the AI to represent your business accurately.

### Basic Information

- [ ] **Business Name:** How should the AI refer to your company?
- [ ] **Brief Description:** 1-2 sentences about what you do
- [ ] **Primary Services:** What should the AI discuss?

### Scheduling Preferences

- [ ] **Available Days:** Which days can meetings be scheduled?
- [ ] **Available Hours:** What time range? (e.g., 9 AM - 5 PM)
- [ ] **Meeting Length:** How long are typical meetings? (usually 30 min)
- [ ] **Buffer Time:** Minimum gap between meetings? (usually 15 min)
- [ ] **Your Timezone:** Important for accurate scheduling

### Communication Guidelines

- [ ] **Meeting Type Name:** What do you call your intro meetings?
  - Examples: "Initial Consultation", "Discovery Call", "Intro Meeting"

- [ ] **Topics for Human Handling:** What should NOT be answered by AI?
  - Usually: pricing, contracts, legal matters, complaints

- [ ] **Communication Tone:** How should emails sound?
  - Professional and formal
  - Professional but friendly
  - Casual and conversational

---

## Optional: Slack Integration

We can send you real-time notifications when the Co-Pilot drafts responses or books meetings.

**If you want Slack notifications:**
- [ ] Access to your Slack workspace (we'll request authorization)
- [ ] A channel for notifications (or we'll create one)

**Benefits:**
- Instant awareness of new inquiries
- Quick link to review drafts
- Booking confirmations in real-time

**Alternative:** If you don't use Slack, we can configure email notifications instead.

---

## What We Provide

After setup, you'll receive:

### Airtable Dashboard
- View all processed emails
- See AI drafts and sent responses
- Track classifications and status
- Add notes for your records

### Slack Channel (if applicable)
- Real-time notifications
- Quick links to Airtable records

### Documentation
- Your configuration summary
- Support contact information
- Quick reference guide

---

## Data & Security

We take security seriously. Here's how your data is protected:

### Data Stays in Your Accounts
- Emails remain in Gmail
- Calendar events in Google Calendar
- Records in your Airtable
- We don't store your emails on our servers

### Secure Connections
- OAuth 2.0 authentication (industry standard)
- Encrypted connections throughout
- No password sharing required

### You're in Control
- Revoke access anytime from Google settings
- We can deactivate immediately upon request
- All access can be audited

### Compliance
- GDPR-aware processing
- CAN-SPAM compliant responses
- No data sold or shared

---

## Pre-Setup Checklist

Before our setup call, please have:

**Essential:**
- [ ] Login access to the Gmail account being monitored
- [ ] Login access to Google Calendar
- [ ] Your business information ready (see above)

**If using Slack:**
- [ ] Admin or installer access to your Slack workspace
- [ ] Decided on a notification channel name

**Helpful but not required:**
- [ ] Sample emails you'd want the AI to handle well
- [ ] Examples of your current email response style
- [ ] Any specific phrases or terms unique to your industry

---

## The Setup Process

Here's what happens after you provide access:

### Day 1: Configuration (We do this)
1. Connect all credentials securely
2. Set up your Airtable dashboard
3. Configure Slack notifications
4. Train AI on your business context

### Day 2: Testing (Together)
1. We send test emails through the system
2. You review AI response quality
3. We make any needed adjustments
4. You approve the final configuration

### Day 3: Launch
1. System goes live
2. We monitor the first 24 hours
3. You receive handoff documentation
4. You start capturing more leads

---

## Questions?

**Contact:**
Michael Johnson
michael@updraftrev.com
(205) 966-6053

We're happy to walk through any of these requirements on a call.
