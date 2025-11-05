# CE-23681: QTIgroup Voice AI Busy Signal Investigation

## Overview
Investigation and resolution of busy signal issue reported by QTIgroup when candidate attempted to call back Voice AI number.

**JIRA Ticket:** CE-23681 (https://sensehq.atlassian.net/browse/CE-23681)  
**Zendesk Ticket:** #112435 (https://sensehq.zendesk.com/agent/tickets/112435)  
**Agency:** QTIgroup (ID: 2557730192181840260)  
**Workflow:** Applicant Screening (https://qtigroup.sensehq.com/automation-workflow/17)
**Voiceflow:** QTI_post_application_bot  
**Candidate:** Melissa Wilson (Recipient ATS ID: 178149256)  
**Report Date:** 11/3/2025

---

## Problem Statement
Candidate reported receiving a busy signal when attempting to call back the Voice AI number on 11/3/2025.

### Initial Report Screenshot
![Candidate Report](https://github.com/user-attachments/assets/b7da53e6-6af1-421d-9118-9d7f9eadf2cd)

---

## Root Cause Analysis

### Probable Causes Investigated
1. **Webhook API Configuration Issue** - Inbound webhook not properly configured
2. **Call Window Expiration** - Incoming call placed beyond maximum mapping window

---

## Investigation Steps

### 1. Webhook Configuration Verification

**Objective:** Verify that the inbound webhook is properly configured in Retell

**Steps:**
1. Navigate to Retell dashboard
2. Go to Phone Numbers section
3. Locate agency's Sense phone number
4. Verify inbound webhook configuration

**Expected Configuration:**
```
https://business.sensehq.com/api/v1/voice-ai/incoming-call?agency_id=2557730192181840260
```

**Result:** Webhook properly configured with correct agency ID

#### Webhook Configuration Screenshot
![Retell Webhook Configuration](https://github.com/user-attachments/assets/2cfb7649-6f61-480b-aa37-cbe8702699d9)

**Verification Status:** PASSED - Webhook is configured correctly

---

### 2. Call Window Timing Verification

**Objective:** Determine if the incoming call was placed beyond the maximum mapping window

#### Step 2.1: Check Last Outbound Call

**Database:** QTIgroup Database  
**Table:** voice_ai_call  
**Query:**
```sql
SELECT * 
FROM voice_ai_call 
WHERE recipient_ats_id = 178149256 
ORDER BY time_created DESC;
```

**Findings:**
- Last outbound call date: **10/30/2025**
- Call status: Went to voicemail (candidate did not pick up)
- Incoming call from candidate: **11/3/2025**
- Time difference: **3 days**

#### Call Logs Screenshot
![Voice AI Call Logs](https://github.com/user-attachments/assets/cc98b39b-7bc4-4620-89ba-5b84d549accf)

#### Step 2.2: Check Agency Configuration

**Objective:** Verify the maximum call mapping window configured for the agency

**Steps:**
1. Navigate to Admin panel for QTIgroup
2. Go to Agency Config section
3. Locate call mapping window settings

#### Agency Configuration Screenshot
![Agency Config Settings](https://github.com/user-attachments/assets/7c4444c6-e17e-4bc1-a3f4-e8a3c5f6e7dd)

**Configuration Found:**
- Maximum window to map incoming call to outbound call: **3 days**

---

## Resolution

### Root Cause
The incoming call was placed **exactly at the 3-day threshold** after the last outbound call. Since the maximum mapping window is configured as 3 days, and the call came in after this window had elapsed, the system did not map the incoming call to the previous outbound session, resulting in a busy signal.

### Expected Behavior Analysis

**Timeline:**
- Outbound call placed: **10/30/2025**
- Call outcome: Voicemail (not picked up)
- Incoming call received: **11/3/2025**
- Time elapsed: **3 days**
- Maximum mapping window: **3 days**
- System behavior: **Call not mapped → Busy signal**

**Conclusion:** The system functioned as designed according to the configured parameters.

---

## Final Resolution Comments

```
What I've found is that the latest outbound call was placed on 30/10/25 to Melissa, 
which went straight to voicemail, indicating that she didn't pick up.

Melissa Wilson then made a call to the Voice AI number three days later.

Since the maximum window we use to map an incoming call to an outbound call is three days, 
this behavior is expected.

Therefore, I'm closing this ticket as the system is functioning as intended.
```

---

## Technical Reference

### Database Schema
**Table:** voice_ai_call

**Key Fields:**
- recipient_ats_id: Unique identifier for the candidate
- time_created: Timestamp of the call
- call_status: Status of the call attempt

### Configuration Parameters
**Agency Config Location:** Admin Panel > Agency Config

**Key Settings:**
- Call Mapping Window: Maximum time window to associate incoming calls with outbound calls

### API Endpoints
**Inbound Call Webhook:**
```
https://business.sensehq.com/api/v1/voice-ai/incoming-call?agency_id={agency_id}
```

### Retell Integration
**Configuration Path:** Retell Dashboard > Phone Numbers > Inbound Webhook

---

## Troubleshooting Guide

### For Similar Issues

1. **Check Webhook Configuration**
   - Verify webhook URL is correct
   - Ensure agency_id parameter is properly set
   - Confirm webhook is active in Retell

2. **Verify Call Timing**
   - Query voice_ai_call table for last outbound call
   - Calculate time difference between outbound and inbound calls
   - Compare against agency's configured mapping window

3. **Review Agency Configuration**
   - Check call mapping window setting in Admin panel
   - Verify setting aligns with business requirements
