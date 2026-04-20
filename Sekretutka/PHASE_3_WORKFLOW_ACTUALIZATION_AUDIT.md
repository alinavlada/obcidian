# Phase 3: Workflow JSON Actualization Audit
**Date**: 2026-04-10  
**Status**: Audit Complete — Ready for Surgical Updates

---

## Executive Summary

Analyzed all 6 core workflow JSON files against Phase 2 documentation. **Most critical functionality is already correctly implemented**. Only **2 surgical updates** needed:

1. **finance.json** — Add session_id/user_id input parameters
2. **finance.json** — Add GAME_LOG event logging (GOLD_GAINED/GOLD_LOST)

All other workflows are actualized and aligned with documentation.

---

## Detailed Findings

### ✅ **1.1 Sekretutka_ENGINE_updated.json** (24 nodes)
**Status: FULLY ACTUALIZED**

**Findings:**
- ✅ STUCK_TIME detection implemented (hoursBetween() function, 25-hour threshold)
- ✅ STUCK_TIME_DETECTED event logging with proper context_json
- ✅ ATTEMPT_FAILED logic (silentDays >= failThresholdDays check)
- ✅ EVENT_TYPE validation present
- ✅ GAME_LOG event construction with: TS, EVENT_TYPE, CONTEXT_JSON, ENTITY_GROUP, metadata
- ✅ State transition tracking (transitionCodes array)
- ✅ Multiple event types logged: STUCK_TIME_DETECTED, ATTEMPT_FAILED, STUCK_TRIGGERED, THERAPIST_REQUIRED, STAGE_CHANGED, CONSISTENCY_CHECKED, LEVEL_READY_REACHED

**Updates Needed:** NONE

**Cross-reference:** Compute Engine State code node (1500+ lines)

---

### ✅ **1. Sekretutka_main.json** (24 nodes)
**Status: FULLY ACTUALIZED**

**Findings:**
- ✅ THERAPIST_WORKFLOW_ID: fxeZXplAm5MfdSGDdi338 (verified in line 580-584)
- ✅ Quest_Architect_WORKFLOW_ID: bVNR2xmUUo_m1_PB7382n (verified in line 789)
- ✅ Proper routing logic for 5 intent types: therapist, finance, habit, quest, rules
- ✅ Session context extraction: chat_id, user_id properly extracted from Telegram trigger
- ✅ Routing rules include all documented paths

**Session ID Transmission:**
- ✅ Therapist call: Passes session_id + user_id
- ✅ Quest_Architect call: Passes session_id + user_id
- ✅ Game Rules call: Passes session_id + user_id
- ❌ Finance call: MISSING session_id + user_id (see below)
- ⚠️ Get Schedule Context: Helper only, doesn't need session context

**Updates Needed:** 1 surgical fix (see finance.json)

---

### ✅ **1.3 Quest_Architect_Daily_Planner.json** (37 nodes)
**Status: FULLY ACTUALIZED**

**Findings:**
- ✅ GOAL_DECOMPOSER mode implemented with proper cell targeting
- ✅ Cell targets correctly defined:
  - Text columns: G, L, Q, V, AA, AF, AK, AP, AU, AZ (max 10 goals)
  - Checkbox columns: F, K, P, U, Z, AE, AJ, AO, AT, AY
  - Deadline columns: H, M, R, W, AB, AG, AL, AQ, AV, BA
- ✅ Step normalization and text/checkbox/deadline value mapping
- ✅ Shared deadline and reward suggestion handling
- ✅ Other modes supported: TASK_CAPTURE, PLANNER, EVENING_REVIEW, TASK_STATUS
- ✅ Proper session_id transmission from main.json

**Mode Routing:** 
- Parse Response node handles mode detection and validation
- Modes: TASK_CAPTURE (default), PLANNER, GOAL_DECOMPOSER, EVENING_REVIEW, TASK_STATUS, HABIT_LOG

**Updates Needed:** NONE

---

### ✅ **1.0 Sekretutka_Game Rules.json** (26 nodes)
**Status: FULLY ACTUALIZED**

**Findings:**
- ✅ GAME_LOG event logging implemented
- ✅ Events logged: ONBOARDING_CHECKPOINT, ACTIVATION, RULE_UPDATED
- ✅ Proper append node structure: "Append Onboarding Checkpoint GAME_LOG", "Append Activation GAME_LOG", "Append RULE_UPDATED GAME_LOG"
- ✅ Rules management FSM router with proper state handling
- ✅ Onboarding checkpoint logic implemented
- ✅ Rule activation/deactivation workflow
- ✅ Session_id transmission from main.json

**Event Types Validated:**
- ONBOARDING_CHECKPOINT: Appended when user completes onboarding step
- ACTIVATION: Appended when a rule is activated  
- RULE_UPDATED: Appended when a rule is modified

**Updates Needed:** NONE

---

### ✅ **1.5 Sekretutka_Therapist.json** (19 nodes)
**Status: FULLY ACTUALIZED**

**Findings:**
- ✅ GAME_LOG append node: "Append GAME_LOG Rows"
- ✅ Multi-step workflow with AI therapist core logic
- ✅ Proper conditional nodes for rule updates, belief writes, and GAME_LOG writes
- ✅ JSON validation and parsing: "Parse + Validate JSON"
- ✅ Rule management: "Deactivate Old Rules", "Append New Rules"
- ✅ Belief management: "Append BELIEFS"
- ✅ Session_id transmission from main.json

**Event Logging Structure:**
- Has GAME_LOG append node for event logging
- Supports rule deactivations and rule appends
- Supports belief appends with proper context

**Updates Needed:** NONE

---

### ⚠️ **1.4 Sekretutka_finance.json** (4 nodes)
**Status: NEEDS ACTUALIZATION**

**Findings:**
- ✅ Basic financial transaction logging to WALLET_Monthly Budget sheet
- ❌ MISSING: session_id and user_id parameters
- ❌ MISSING: GAME_LOG event logging (should log GOLD_GAINED or GOLD_LOST events)
- ⚠️ Only 4 nodes (very minimal), missing event logging infrastructure

**Current Structure:**
```
When Executed by Another Workflow
  ↓
Calc Row Number (computes target row based on existing entries)
  ↓
Get row(s) in sheet (reads existing transactions)
  ↓
Append or update row in sheet (writes to WALLET_Monthly Budget)
```

**Missing Components:**
1. Session context parameters (session_id, user_id, chat_id)
2. GAME_LOG event append node for GOLD_GAINED/GOLD_LOST events
3. Event validation and context_json generation

**Updates Needed:**
1. Add session_id, user_id, chat_id to workflowInputs in main.json finance call
2. Add GAME_LOG append node to finance.json with proper event typing
3. Implement event selection logic: GOLD_GAINED vs GOLD_LOST based on TYPE field

**Cross-reference:** Appendix A (update details below)

---

### ✅ **1.2 Sekretutka_Get Schedule Context.json** (8 nodes)
**Status: HELPER WORKFLOW — NO CHANGES NEEDED**

**Findings:**
- Simple context provider (not a primary workflow)
- Reads different data sheets based on source parameter
- Routes to: Личный (Personal), Развитие (Development), Семья (Family), profile, Quest, wallet
- No session context needed (helper role)
- No event logging required

**Updates Needed:** NONE

---

## Critical Issues Fixed vs Remaining

### Phase 1 Critical Fixes Status (all ✅ completed):
1. ✅ **THERAPIST_WORKFLOW_ID**: fxeZXplAm5MfdSGDdi338 (verified in ENGINE and main.json)
2. ✅ **EVENT_TYPE Registry**: Created with 15 comprehensive types
3. ✅ **STUCK_TIME Detection**: Implemented in ENGINE with 25-hour threshold

### Phase 2 Documentation Sync Status (all ✅ completed):
1. ✅ README.md: Added STUCK_TIME rule and EVENT_TYPE Registry
2. ✅ AUDIT_2026-04-10_REVISED.md: Marked critical issues #1-3 as ИСПРАВЛЕНО
3. ✅ Sekretutka_PRD_v1.md: Added STUCK_TIME subsection and expanded STUCK detection

### Phase 3 Remaining Issues (NEW):
1. ⚠️ **finance.json Session Context**: Missing session_id/user_id parameters
2. ⚠️ **finance.json Event Logging**: Missing GAME_LOG event logging for transactions

---

## Appendix A: Finance.json Required Updates

### Update 1: main.json finance workflow call parameters

**Location:** main.json → "Call '1.4. Sekretutka_finance'" node → parameters.workflowInputs.value

**Current (incomplete):**
```json
{
  "AMOUNT": "={{ $json.route_payload.amount || 0 }}",
  "TYPE": "={{ $json.route_payload.finance_type || '' }}",
  "CATEGORY": "={{ $json.route_payload.category || '' }}",
  "CURRENCY": "={{ $json.route_payload.currency || 'THB' }}",
  "DETAILS": "={{ $json.route_payload.details || $json.user_message || '' }}",
  "DATE": "={{ $json.route_payload.date || $json.today_bkk }}",
  "ACCOUNT": "={{ $json.route_payload.account || 'BKK-bank' }}"
}
```

**Required Addition:**
```json
{
  "AMOUNT": "={{ $json.route_payload.amount || 0 }}",
  "TYPE": "={{ $json.route_payload.finance_type || '' }}",
  "CATEGORY": "={{ $json.route_payload.category || '' }}",
  "CURRENCY": "={{ $json.route_payload.currency || 'THB' }}",
  "DETAILS": "={{ $json.route_payload.details || $json.user_message || '' }}",
  "DATE": "={{ $json.route_payload.date || $json.today_bkk }}",
  "ACCOUNT": "={{ $json.route_payload.account || 'BKK-bank' }}",
  "user_id": "={{ String($json.message?.from?.id || $json.user_id || 'unknown') }}",
  "session_id": "={{ String($json.message?.chat?.id || $json.chat_id || 'default-session') }}",
  "chat_id": "={{ String($json.message?.chat?.id || $json.chat_id || '') }}"
}
```

### Update 2: finance.json add GAME_LOG event node

**Location:** finance.json → Add new node after "Append or update row in sheet"

**New Node:** "Append GAME_LOG Event"
- Type: n8n-nodes-base.googleSheets.appendOrUpdate
- Sheet: GAME_LOG
- Columns to append:
  - TS: Current timestamp (ISO format with BKK timezone)
  - EVENT_TYPE: GOLD_GAINED or GOLD_LOST (based on TYPE field logic)
  - CONTEXT_JSON: JSON with amount, currency, category, details
  - ENTITY_GROUP: "WALLET"
  - VALUE: Amount (as string)
  - RESOURCE_TYPE: Currency code (e.g., "THB")
  - USER_ID: from input parameter
  - CHAT_ID: from input parameter
  - SESSION_ID: from input parameter

**Logic:**
```
If TYPE == "income" or "earnings" or "gain" → GOLD_GAINED
Otherwise → GOLD_LOST
```

---

## Summary Table

| Workflow | Nodes | Status | Updates | Critical Issues |
|----------|-------|--------|---------|-----------------|
| ENGINE_updated.json | 24 | ✅ Actualized | 0 | None |
| main.json | 24 | ⚠️ Partial | 1 | Finance routing missing context |
| Quest_Architect.json | 37 | ✅ Actualized | 0 | None |
| Game Rules.json | 26 | ✅ Actualized | 0 | None |
| Therapist.json | 19 | ✅ Actualized | 0 | None |
| finance.json | 4 | ⚠️ Minimal | 2 | Missing context + GAME_LOG |
| Get Schedule Context.json | 8 | ✅ Helper | 0 | N/A |

---

## Next Steps

1. ✅ **IMMEDIATE:** Update main.json finance call with session context parameters (Appendix A, Update 1)
2. ✅ **IMMEDIATE:** Add GAME_LOG append node to finance.json (Appendix A, Update 2)
3. 📋 **VERIFICATION:** Test finance workflow with session context transmission
4. 📋 **DOCUMENTATION:** Update Phase 3 completion summary

---

**Audit Complete: 2026-04-10**  
**Actualization Ready: YES**  
**Estimated Update Time: 15-20 minutes**
