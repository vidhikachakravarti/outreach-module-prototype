# Lillia Outreach Module V3.0 - Complete Documentation

## Overview

This document provides complete documentation for the **Outreach Module V3.0**, including the revised architecture, integration flows, and implementation details based on product review feedback.

**Version**: 3.0 (Post-Feedback Revision)
**Date**: February 2026
**Status**: Production-Ready Specification

---

## Table of Contents

1. [What's New in V3.0](#whats-new-in-v30)
2. [Architecture Overview](#architecture-overview)
3. [Agent Creation Flow](#agent-creation-flow)
4. [System Components](#system-components)
5. [Integration Details](#integration-details)
6. [Agent Execution Flow](#agent-execution-flow)
7. [API Reference](#api-reference)
8. [Deployment Guide](#deployment-guide)

---

## What's New in V3.0

### Major Architectural Changes

#### 1. One Channel Per Agent
**Previous**: Multi-channel configuration within a single agent with automatic fallback
**V3.0**: Each agent is configured for exactly ONE channel

**Rationale**: Different channels have fundamentally different capabilities (AI vs non-AI), content requirements (character limits, templates), and conversation flows. Single-channel agents provide:
- Simpler configuration (no cross-channel compatibility issues)
- Clearer analytics (performance attributable to one channel)
- Better predictability (no fallback logic ambiguity)

**Multi-Channel Strategy**: Use the **Duplicate** feature to create channel variants of existing agents

#### 2. Role Selection
**New in V3.0**: Agents explicitly target one of three roles:
- **Patients**: Enrolled patients or members
- **Caregivers**: Family members or designated caregivers
- **Care Providers**: Nurses, coaches, care managers

**Impact**:
- Determines available triggers (patient events vs provider events)
- Affects variable availability (patient variables vs provider variables)
- May restrict certain knowledge base content (clinical details for providers only)

#### 3. Pre-Built Knowledge Base Templates
**New in V3.0**: Three starter templates available out-of-the-box:

| Template | Priority | Contents |
|----------|----------|----------|
| **Safety Guardrails** | Highest | Medical advice restrictions, emergency protocols, escalation triggers, liability disclaimers |
| **Healthcare Compliance** | Highest | HIPAA language, consent reminders, recording disclosures, TCPA/GDPR/TRAI requirements |
| **Communication Best Practices** | Base | Empathetic language patterns, active listening, cultural sensitivity |

**Knowledge Priority**: Custom Knowledge > Safety Guardrails > Healthcare Compliance > Communication Best Practices

Safety and Compliance always override other sources.

#### 4. Prompt Building Step
**Previous**: Missing from flow
**V3.0**: Dedicated **Step 5** for writing agent instructions

**For AI Channels**: Write full conversational prompts with:
- Goal definition
- Conversation flow steps
- Data collection requirements
- Escalation conditions

**For Non-AI Channels**: Write message templates with variable placeholders

#### 5. Three Trigger Modes
**Previous**: Dual scheduling (Journey-linked or independent)
**V3.0**: Three distinct trigger modes:

1. **API / Webhook**: External systems trigger via API call
2. **Internal Ecosystem**: Platform events trigger agent (patient onboarded, alert triggered, etc.)
3. **Journey Task Linkage**: Agent runs when journey reaches specific task

#### 6. Product Gating
**New in V3.0**: Document Templates are **product-gated**

If client doesn't have Documents module:
- Step 6 shows disabled state with explanation
- Upsell message with "Learn More" link
- Agent can still be created without templates

#### 7. Programs Now Optional
**Previous**: Programs were required and hierarchical
**V3.0**: Programs are **optional** grouping/filtering mechanisms

Agents can be created without program association for:
- Simpler use cases
- Cross-program outreach
- Generic notifications

#### 8. Agent Duplication
**New in V3.0**: First-class duplication feature

- Duplicate button appears on agent card hover
- Creates copy with all settings
- User can immediately change channel
- Incompatible settings flagged for review

---

## Architecture Overview

### High-Level Architecture

```
┌─────────────────────────────────────────────────────────────────┐
│                         PORTAL UI                               │
│                      (Agent CRUD)                               │
└────────────────────┬────────────────────────────────────────────┘
                     │
          ┌──────────┼──────────┐
          │          │          │
          ▼          ▼          ▼
    ┌─────────┐ ┌─────────┐ ┌──────────┐
    │ Agent   │ │Knowledge│ │Document  │
    │ Config  │ │  Base   │ │Templates │
    │ Engine  │ │  Layer  │ │          │
    └────┬────┘ └────┬────┘ └────┬─────┘
         │           │           │
         └───────────┼───────────┘
                     │
                     ▼
           ┌─────────────────────┐
           │  AGENT FRAMEWORK    │◄──────────┐
           │  (Conversation AI)  │           │
           └──────────┬──────────┘           │
                      │                      │
       ┌──────────────┼──────────────┐       │
       │              │              │       │
       ▼              ▼              ▼       │
┌──────────┐   ┌──────────┐   ┌──────────┐  │
│ TRIGGER  │   │ CHANNELS │   │ESCALATION│──┘
│   MGMT   │   │  LAYER   │   │  SYSTEM  │
└────┬─────┘   └────┬─────┘   └──────────┘
     │              │
     │              ▼
     │         ┌──────────┐
     │         │   RATE   │
     │         │ LIMITING │
     │         └────┬─────┘
     │              │
     │              ▼
     │         ┌──────────┐
     │         │ CHANNEL  │
     │         │PROVIDERS │
     │         └──────────┘
     │
     ▼
┌──────────┐        ┌──────────┐
│ IDENTITY │◄──────►│ CONSENT  │
│   &      │        │   MGMT   │
│ PROFILE  │        └──────────┘
└────┬─────┘
     │
     ▼
┌───────────────────────────────────────────┐
│       EXECUTION LOGGING LAYER             │
│  [Transcripts][Status][Outcomes][Audit]   │
└────────────────┬──────────────────────────┘
                 │
                 ▼
┌───────────────────────────────────────────┐
│      ANALYTICS & INSIGHTS LAYER           │
│  [Funnels][Effectiveness][Dashboards]     │
└───────────────────────────────────────────┘
```

### Component Categories

**UI Layer** (User-facing):
- Portal UI (Agent CRUD)
- Analytics & Insights Dashboards

**Core Layer** (Essential functionality):
- Agent Configuration Engine
- Agent Framework (Conversation AI)
- Knowledge Base Layer
- Trigger Management System
- Channels Layer

**Auto-Enabled Layer** (Always active):
- Identity & Profile Resolution
- Consent Management Engine
- Rate Limiting Layer
- Escalation System
- Execution Logging Layer

**Integration Layer** (External connections):
- Channel Providers (Retell AI, WhatsApp API, SMS gateways)
- Document Templates (optional, gated)
- Journey Tracker (optional)

---

## Agent Creation Flow

### 8-Step Configuration Process

#### Step 1: Channel Selection
**Purpose**: Select ONE communication channel

**Options**:
- 📞 **Voice Call** (AI-enabled, via Retell AI)
- 💬 **WhatsApp** (AI-enabled, via WhatsApp Business API)
- 📱 **SMS** (Non-AI, simple text delivery)
- 📧 **Email** (Non-AI, email delivery)

**Key Concept**: Single-channel per agent. For multi-channel, use Duplicate feature.

**Implications**: Channel selection affects all subsequent steps (KB availability, config options, prompt type).

---

#### Step 2: Agent Basics
**Purpose**: Define agent identity and target audience

**Fields**:
- **Agent Name**: Descriptive name indicating purpose and channel
  *Example: "Medication Reminder — Voice"*

- **Agent Description**: What this agent does

- **Target Role**: Who receives this outreach?
  - 👤 **Patients**: Medication reminders, appointments, check-ins, surveys
  - 👨‍👩‍👧 **Caregivers**: Updates, care instructions, coordination
  - ⚕️ **Care Providers**: Alerts, patient status, task notifications

- **Program** (Optional): Link to care program or leave blank

**Design Decision**: Programs are now optional to support simpler use cases.

---

#### Step 3: Knowledge Base
**Purpose**: Select knowledge sources for AI guidance

**Availability**: Only for AI-enabled channels (Voice, WhatsApp). Skipped for SMS/Email.

**Pre-Built Templates**:
1. **Safety Guardrails** (Highest Priority)
   - What AI should never say (medical advice, diagnoses, prescriptions)
   - Emergency protocols (detect distress, provide crisis resources)
   - Escalation triggers (when to hand off immediately)
   - Liability disclaimers

2. **Healthcare Compliance** (Highest Priority)
   - HIPAA-safe language patterns
   - Consent reminders and verification
   - Recording disclosures
   - TCPA/GDPR/TRAI requirements

3. **Communication Best Practices** (Base)
   - Empathetic language patterns
   - Active listening cues
   - Cultural sensitivity guidelines
   - Handling difficult emotions

**Custom Knowledge Bases**:
- Program-specific protocols (Diabetes Clinical Protocols, Cardiac Rehab Guidelines)
- Partner terminology and branding
- Client-specific escalation contacts

**Priority Resolution**: When conflicts arise, higher priority wins. Safety > Compliance > Custom > Base.

---

#### Step 4: Agent Configuration
**Purpose**: Configure channel-specific behavior

**For AI Channels**:
- **Conversation Tone**:
  - Friendly & Warm (casual, approachable)
  - Professional (formal, business-appropriate)
  - Empathetic & Supportive (gentle, emotional support)
  - Concise & Direct (brief, minimal small talk)

- **Behavior Settings**:
  - Allow interruptions during speech (Voice only)
  - Enable escalation to human agent
  - Record conversation for review
  - Maximum conversation duration (minutes)
  - Silence timeout before re-prompting (seconds)

**For Non-AI Channels**:
- Sender ID / From address
- Character limit (SMS: single vs multi-part)
- Track delivery status
- Track open/read receipts

---

#### Step 5: Prompt / Message Building
**Purpose**: Define what the agent actually does

**For AI Channels**: Write **agent instructions**
- What is the goal? (remind about medication, onboard to program, collect survey)
- What steps should it follow?
- What information to collect?
- How to handle different responses?
- When to escalate?

**Sample Prompt**:
```
You are a medication reminder assistant for {{program_name}}. Your goal is to:

1. Greet the patient warmly using their name ({{patient_name}})
2. Remind them about their {{medication_name}} scheduled for {{medication_time}}
3. Ask if they have taken it today
4. If not, gently encourage them and ask if there are any barriers
5. Log their response and thank them

Always be supportive, never judgmental. If they express distress, follow escalation protocol.
```

**For Non-AI Channels**: Write **message template**
```
Hi {{patient_name}}, this is a reminder about your appointment with {{provider_name}}
on {{appointment_date}} at {{appointment_time}}. Reply YES to confirm or call us to reschedule.
```

**Available Variables**:
- `{{patient_name}}`, `{{patient_age}}`, `{{preferred_language}}`
- `{{program_name}}`, `{{program_duration}}`, `{{enrollment_date}}`
- `{{medication_name}}`, `{{medication_dosage}}`, `{{medication_time}}`
- `{{appointment_date}}`, `{{appointment_time}}`, `{{provider_name}}`
- `{{coach_name}}`, `{{care_team_phone}}`

---

#### Step 6: Document Templates
**Purpose**: Link templates for structured data capture

**Product Gating**: Only available if client has **Documents module**

**If Available**: Select from existing templates
- Medication Adherence Log (fields: adherence_status, barriers_mentioned, follow_up_needed)
- Onboarding Summary (fields: preferred_time, questions_asked, emergency_contact)
- Follow-up Notes (fields: patient_sentiment, concerns_raised, action_items)
- Survey Response Collection

**Automatic Field Mapping**: Agent's prompt is enhanced to collect data for template fields

**If Not Available**: Disabled state shown with explanation and upsell link

---

#### Step 7: Trigger Configuration
**Purpose**: Define how/when agent runs

**Mode 1: API / Webhook**
- External system calls API endpoint
- Use case: EHR, scheduling software triggers
- Provides agent ID for integration
- Supports batch triggering

**Sample API Call**:
```bash
POST https://api.lillia.com/v1/outreach/agents/ag_12345/trigger
{
  "user_id": "usr_67890",
  "variables": {
    "medication_name": "Metformin 500mg",
    "medication_time": "8:00 AM"
  }
}
```

**Mode 2: Internal Ecosystem Triggers**
- Agent triggered by platform events
- **Patient Events**: New onboarding, missed check-in, glucose threshold exceeded, adherence drops, appointment reminder
- **Provider Events**: New patient assigned, patient alert, daily digest, escalation routed

**Mode 3: Journey Task Linkage**
- Agent linked to specific Journey Tracker task
- Runs when journey reaches that task
- Inherits timing from journey
- Reports completion back to Journey Tracker
- Requires Journey Tracker access

**Design Philosophy**: Outreach remains independent from Journey Tracker. Task linkage is integration option, not requirement.

---

#### Step 8: Review & Create
**Purpose**: Validate and activate

**Configuration Summary**: Shows all settings
- Channel, name, role, program
- Knowledge bases selected
- Document template linked
- Trigger mode configured

**Pre-Activation Validation**:
- ✓ Channel configuration complete
- ✓ Agent basics provided
- ✓ Knowledge bases active
- ✓ Prompt meets minimum requirements
- ✓ Document template field mapping valid
- ✓ Trigger configuration valid

**If Critical Checks Fail**: Activation blocked with error messages

**Active System Layers**: Shows which layers are enabled
- Configured layers (user-controlled)
- Auto-enabled layers (always on)

**Activation**: Creates agent and makes it live for triggering

---

## System Components

### 1. Portal UI (Agent CRUD)
**Type**: User Interface Layer

**Responsibilities**:
- Present 8-step configuration flow
- Validate user inputs
- Communicate with Agent Configuration Engine
- Display agent list with stats
- Provide duplication feature

**Technology**: React/Vue.js with state management

---

### 2. Agent Configuration Engine
**Type**: Core Layer

**Responsibilities**:
- Store agent configurations
- Version-lock configurations (prevent mid-execution changes)
- Validate configuration completeness
- Manage agent lifecycle (draft, active, archived)
- Serve configurations to Agent Framework

**Data Model**:
```json
{
  "agent_id": "ag_12345",
  "version": 3,
  "channel": "voice",
  "role": "patients",
  "name": "Medication Reminder",
  "program_id": "prog_diabetes",
  "knowledge_bases": ["kb_safety", "kb_compliance", "kb_diabetes"],
  "configuration": {
    "tone": "friendly_warm",
    "max_duration": 600,
    "allow_interruptions": true,
    "enable_escalation": true,
    "record_conversation": true
  },
  "prompt": "You are a medication reminder assistant...",
  "document_template_id": "tpl_adherence",
  "trigger": {
    "mode": "internal",
    "event": "missed-checkin"
  },
  "status": "active",
  "created_at": "2026-02-01T10:00:00Z",
  "updated_at": "2026-02-01T10:00:00Z"
}
```

---

### 3. Knowledge Base Integration Layer
**Type**: Core Layer

**Responsibilities**:
- Store KB content (documents, guidelines, rules)
- Assemble knowledge before each conversation
- Merge multiple KBs with priority resolution
- Inject KB into agent prompts

**Priority System**:
1. **Highest**: Safety Guardrails, Healthcare Compliance
2. **Medium**: Custom program/client KBs
3. **Base**: Communication Best Practices, agent default

**Conflict Resolution**: Higher priority source always wins

**Example**:
- Base KB says: "Be friendly and conversational"
- Safety KB says: "Never provide medical advice"
- → Agent follows Safety (never provides advice) while being friendly

---

### 4. Agent Framework (Conversation AI)
**Type**: Core Layer

**Responsibilities**:
- Execute conversation prompts
- Conduct real-time conversations (Voice, WhatsApp)
- Extract structured data from conversations
- Monitor for escalation triggers
- Handle timeouts and interruptions

**Powered By**: Retell AI (for Voice and WhatsApp)

**Data Extraction**: Uses NLP to identify and extract:
- Medication adherence status
- Barriers or concerns mentioned
- Sentiment and emotional state
- Specific data points for template fields

---

### 5. Trigger Management System
**Type**: Core Layer

**Responsibilities**:
- Accept API trigger requests
- Listen for internal platform events
- Integrate with Journey Tracker for task triggers
- Queue execution requests
- Rate limit trigger requests per user

**API Endpoints**:
- `POST /v1/outreach/agents/{agent_id}/trigger` - Trigger agent for user
- `POST /v1/outreach/agents/batch-trigger` - Trigger for multiple users
- `GET /v1/outreach/executions/{execution_id}` - Check execution status

**Event Subscriptions**: Subscribes to internal event bus for:
- `patient.onboarded`
- `patient.missed_checkin`
- `patient.alert_triggered`
- `provider.patient_assigned`
- `journey.task_due`

---

### 6. Identity & Profile Resolution
**Type**: Auto-Enabled Layer

**Responsibilities**:
- Resolve user_id to full user profile
- Load contact information (phone, WhatsApp ID, email)
- Verify user exists and is active
- Provide user context to Agent Framework

**Always Active**: Cannot be disabled

---

### 7. Consent Management Engine
**Type**: Auto-Enabled Layer

**Responsibilities**:
- Track user consent per channel
- Block outreach if consent missing or revoked
- Enforce opt-out requests immediately
- Comply with TCPA, GDPR, TRAI DND regulations

**Consent Check Flow**:
1. Execution requested for user_id + channel
2. Query consent: `hasConsent(user_id, channel)`
3. If `false`: Abort execution, log reason
4. If `true`: Proceed to next layer

**Always Active**: Cannot be disabled (regulatory requirement)

---

### 8. Channels Layer
**Type**: Core Layer

**Responsibilities**:
- Format content for each channel
- Apply channel-specific constraints (SMS character limits, WhatsApp template requirements)
- Send messages via channel providers
- Handle delivery failures and retry logic

**Channel-Specific Logic**:
- **Voice**: Initiate call via Retell AI, stream conversation
- **WhatsApp**: Use approved template for initial message, freeform for follow-ups
- **SMS**: Enforce 160/1600 character limits, split multi-part
- **Email**: Apply HTML/plaintext formatting, add unsubscribe link

---

### 9. Rate Limiting Layer
**Type**: Auto-Enabled Layer

**Responsibilities**:
- Enforce max messages per user per channel per day
- Prevent spam and user fatigue
- Queue rate-limited requests for later retry

**Default Limits**:
- Voice: 3 calls per day
- WhatsApp: 5 messages per day
- SMS: 10 messages per day
- Email: No limit (has unsubscribe)

**Always Active**: Cannot be disabled

---

### 10. Channel Providers
**Type**: Integration Layer

**Retell AI** (Voice, WhatsApp):
- Conducts AI-powered conversations
- Streams real-time conversation data
- Returns transcript + extracted entities

**WhatsApp Business API**:
- Sends messages via official API
- Requires approved message templates
- Handles inbound message webhooks

**SMS Gateways** (Twilio, MessageBird):
- Sends SMS messages
- Provides delivery receipts

**SMTP Servers** (SendGrid, Mailgun):
- Sends emails
- Tracks open/click rates

---

### 11. Escalation System
**Type**: Auto-Enabled Layer

**Responsibilities**:
- Monitor conversations for escalation keywords
- Detect user distress or confusion
- Route to human agents when needed
- Provide context to human agent (transcript, user profile)

**Escalation Triggers**:
- User explicitly requests human ("I want to talk to a person")
- Distress keywords detected ("help", "emergency", "pain")
- Conversation stuck (same question repeated 3+ times)
- Agent can't answer question (knowledge base gap)
- Conversation exceeds max duration

**Always Active**: Cannot be disabled

---

### 12. Document Templates Integration
**Type**: Integration Layer (Optional, Product-Gated)

**Responsibilities**:
- Link agents to document templates
- Extract structured data from conversations
- Populate template fields automatically
- Create document instance in Documents module

**Availability**: Only if client has **Documents module**

**Field Mapping Example**:
- Conversation: "Yes, I took my medicine this morning"
- Extracted: `adherence_status: "taken"`
- Template field: `adherence_status` populated with "taken"

---

### 13. Execution Logging Layer
**Type**: Auto-Enabled Layer

**Responsibilities**:
- Log every execution event (trigger received, consent check, delivery, outcome)
- Store full conversation transcripts (Voice, WhatsApp)
- Maintain audit trail for compliance
- Provide data for analytics

**Logged Events**:
- `execution.triggered` (timestamp, user_id, agent_id)
- `consent.checked` (result: approved/denied)
- `delivery.attempted` (channel, status)
- `conversation.started` (Voice/WhatsApp only)
- `conversation.completed` (outcome, duration, extracted data)
- `execution.completed` (final status: success/failed/escalated/timeout)

**Always Active**: Required for compliance and analytics

---

### 14. Analytics & Insights Layer
**Type**: UI Layer (Auto-Enabled)

**Responsibilities**:
- Aggregate execution logs into metrics
- Generate performance dashboards
- Provide channel effectiveness comparisons
- Calculate engagement funnels
- Alert on anomalies (high failure rate, low response rate)

**Available Metrics**:
- Total runs, success rate, escalation rate
- Average conversation duration
- Channel effectiveness (Voice vs WhatsApp vs SMS vs Email)
- Document completion rate (if templates linked)
- Time-to-resolution
- User engagement rate (response rate)

**Dashboards**:
- Per-agent performance
- Program-level rollups
- Channel comparison
- User engagement funnels

---

### 15. Journey Tracker Integration
**Type**: Integration Layer (Optional)

**Responsibilities**:
- Link agents to journey tasks
- Receive task-due events from Journey Tracker
- Trigger agents when task becomes due
- Report task completion back to Journey Tracker

**Independence**: Outreach Module operates fully independently. Journey integration is opt-in.

**Use Case**: "Run medication reminder when patient reaches Day 7 of Diabetes Care Journey"

**Data Flow**:
1. Journey Tracker: Patient reaches "Day 7: Medication Check" task
2. Journey Tracker emits: `journey.task_due` event
3. Trigger Management receives event, initiates agent
4. Agent completes conversation
5. Trigger Management sends: `task.completed` to Journey Tracker
6. Journey Tracker advances patient to next task

---

## Integration Details

### Retell AI Integration (Voice Channel)

**How It Works**:
1. Agent Framework sends API request to Retell AI:
   ```json
   {
     "phone_number": "+1234567890",
     "prompt": "You are a medication reminder assistant...",
     "knowledge_base": "... merged KB content ...",
     "variables": {
       "patient_name": "Rahul",
       "medication_name": "Metformin 500mg"
     }
   }
   ```

2. Retell AI initiates outbound phone call

3. Real-time conversation streamed back to Agent Framework via WebSocket

4. Agent Framework monitors stream for:
   - Escalation keywords
   - Structured data extraction
   - Conversation completion

5. When call ends, Retell AI returns:
   ```json
   {
     "call_id": "call_abc123",
     "duration": 257,
     "transcript": "... full conversation ...",
     "extracted_entities": {
       "adherence_status": "taken",
       "barriers_mentioned": "none"
     },
     "outcome": "completed"
   }
   ```

6. Document Template Integration Layer receives extracted data, populates template

**Data Flow**: Agent Framework → Retell AI API → Telephony Provider → User → Retell AI → Agent Framework → Document Templates

---

### WhatsApp Business API Integration

**How It Works**:
1. Agent Framework formats message using **approved WhatsApp template**:
   ```json
   {
     "template_name": "medication_reminder_1",
     "language_code": "en",
     "parameters": [
       {"type": "text", "text": "Rahul"},
       {"type": "text", "text": "Metformin 500mg"}
     ]
   }
   ```

2. Channels Layer sends via WhatsApp Business API

3. User responds (now in 24-hour freeform window)

4. WhatsApp webhook hits our Channels Layer with response

5. Agent Framework processes response, determines next message

6. Two-way conversation continues until goal achieved or timeout

7. Structured data extracted and logged

**Template Requirement**: All **initial** WhatsApp messages must use pre-approved templates (WhatsApp policy). Follow-up messages within 24-hour window can be freeform.

---

### Journey Tracker Integration

**How It Works**:
1. **Configuration Phase**: User selects "Journey Task Link" in Step 7
   - Searches for journey task: "Day 7: Initial Follow-up Call"
   - Agent Config Engine stores: `trigger.mode = "journey_task"`, `trigger.journey_task_id = "task_789"`

2. **Runtime Phase**: Patient progresses through journey
   - Journey Tracker: Patient reaches "Day 7: Initial Follow-up Call"
   - Journey Tracker emits event: `journey.task_due(patient_id: usr_123, task_id: task_789)`

3. **Trigger Management** receives event:
   - Looks up agents with `trigger.journey_task_id = "task_789"`
   - Finds our agent: `ag_medication_reminder`
   - Initiates execution: `trigger_agent(ag_medication_reminder, usr_123)`

4. **Agent Execution** completes conversation

5. **Completion Signal**: Trigger Management sends to Journey Tracker:
   ```json
   {
     "event": "task.completed",
     "patient_id": "usr_123",
     "task_id": "task_789",
     "completed_by": "ag_medication_reminder",
     "outcome": "success"
   }
   ```

6. Journey Tracker marks task complete, advances patient to next task

**Independence Maintained**: If Journey Tracker is down/unavailable, agents with API or Internal triggers continue working. Only Journey-linked agents are affected.

---

### Document Templates Integration

**How It Works**:
1. **Configuration Phase**: User links document template in Step 6
   - Selects "Medication Adherence Log"
   - Template has fields: `adherence_status`, `barriers_mentioned`, `follow_up_needed`

2. **Prompt Augmentation**: Agent Config Engine enhances prompt:
   ```
   Original Prompt: "Ask if they took their medication..."

   Augmented Prompt: "Ask if they took their medication.
   Extract the following information:
   - adherence_status: Did they take it? (taken/not_taken/missed)
   - barriers_mentioned: Any obstacles they mentioned? (text)
   - follow_up_needed: Do they need additional support? (yes/no)"
   ```

3. **Conversation Execution**: Agent conducts conversation, Retell AI extracts data

4. **Post-Conversation**: Document Template Integration Layer receives:
   ```json
   {
     "template_id": "tpl_adherence",
     "extracted_data": {
       "adherence_status": "taken",
       "barriers_mentioned": "none",
       "follow_up_needed": "no"
     }
   }
   ```

5. **Document Creation**: Calls Documents module API:
   ```json
   POST /v1/documents
   {
     "template_id": "tpl_adherence",
     "user_id": "usr_123",
     "fields": {
       "adherence_status": "taken",
       "barriers_mentioned": "none",
       "follow_up_needed": "no"
     },
     "source": "outreach_agent_ag_12345",
     "conversation_id": "exec_abc123"
   }
   ```

6. Document created, linked to user profile and conversation transcript

**Product Gating**: If client doesn't have Documents module, Step 6 shows disabled state. Agent can still be created, but no document auto-population.

---

## Agent Execution Flow

### Complete Lifecycle: Trigger → Completion

```
1. TRIGGER RECEIVED
   │
   ├─ API Call: External system triggers via POST /v1/outreach/agents/{id}/trigger
   ├─ Internal Event: Platform emits event (patient.onboarded, patient.alert_triggered)
   └─ Journey Task: Journey Tracker emits journey.task_due event
   │
   ▼
2. IDENTITY RESOLUTION
   │
   ├─ Resolve user_id → full user profile
   ├─ Load contact info for channel (phone, WhatsApp ID, email)
   └─ Verify consent for this channel
   │
   ├─ Consent OK → Continue
   └─ No Consent → ABORT + LOG("No consent")
   │
   ▼
3. AGENT INITIALIZATION
   │
   ├─ Load agent configuration (version-locked)
   ├─ Assemble knowledge base (priority-merged)
   ├─ Resolve variables ({{patient_name}} → "Rahul")
   └─ Inject prompt with resolved values
   │
   ▼
4. RATE LIMIT CHECK
   │
   ├─ Check: User reached max messages for channel today?
   ├─ If YES → Queue for tomorrow + LOG("Rate limited")
   └─ If NO → Continue
   │
   ▼
5. CHANNEL DELIVERY
   │
   ├─ Format content for channel
   ├─ Apply rate limiting
   └─ Send via channel provider (Retell AI, WhatsApp API, SMS gateway)
   │
   ├─ Delivered → Continue
   └─ Failed → Retry per policy → Max retries → FAIL + LOG + ALERT
   │
   ▼
6. CONVERSATION EXECUTION (AI channels only)
   │
   ├─ Agent conducts conversation per prompt
   ├─ Collects structured data
   └─ Monitors for escalation triggers
   │
   ├─ Completed → Continue
   ├─ Escalated → Route to Human
   └─ Timeout → LOG + Retry later
   │
   ▼
7. POST-CONVERSATION PROCESSING
   │
   ├─ Populate document template (if linked)
   ├─ Update user profile with collected data
   ├─ Send completion signal to Journey Tracker (if linked)
   └─ Log full execution trace
   │
   ▼
8. EXECUTION COMPLETE
   │
   └─ Outcome logged, analytics updated
```

---

## API Reference

### Trigger Agent Execution

```bash
POST https://api.lillia.com/v1/outreach/agents/{agent_id}/trigger

Headers:
  Content-Type: application/json
  Authorization: Bearer {api_key}

Body:
{
  "user_id": "usr_67890",
  "variables": {
    "patient_name": "Rahul Kumar",
    "medication_name": "Metformin 500mg",
    "medication_time": "8:00 AM",
    "program_name": "Diabetes Care Management"
  },
  "schedule": {
    "type": "immediate"  // or "scheduled" with "run_at" timestamp
  }
}

Response (200 OK):
{
  "status": "accepted",
  "execution_id": "exec_abc123",
  "agent_id": "ag_67890",
  "user_id": "usr_67890",
  "scheduled_for": "2026-02-03T08:00:00Z",
  "message": "Agent execution scheduled successfully"
}
```

---

### Check Execution Status

```bash
GET https://api.lillia.com/v1/outreach/executions/{execution_id}

Headers:
  Authorization: Bearer {api_key}

Response (200 OK):
{
  "execution_id": "exec_abc123",
  "agent_id": "ag_67890",
  "user_id": "usr_67890",
  "status": "completed",  // pending, in_progress, completed, failed, escalated
  "channel": "voice",
  "started_at": "2026-02-03T08:00:15Z",
  "completed_at": "2026-02-03T08:04:32Z",
  "outcome": {
    "success": true,
    "conversation_duration": 257,
    "escalated": false,
    "structured_data": {
      "adherence_status": "taken",
      "barriers_mentioned": "none",
      "follow_up_needed": false
    }
  },
  "transcript_url": "https://api.lillia.com/v1/transcripts/exec_abc123"
}
```

---

### Batch Trigger

```bash
POST https://api.lillia.com/v1/outreach/agents/{agent_id}/batch-trigger

Body:
{
  "user_ids": ["usr_123", "usr_456", "usr_789"],
  "variables_template": {
    "medication_name": "Metformin 500mg",
    "medication_time": "8:00 AM"
  }
}

Response (200 OK):
{
  "status": "accepted",
  "batch_id": "batch_xyz",
  "scheduled_executions": 3,
  "message": "Batch trigger queued successfully"
}
```

---

## Deployment Guide

### Prerequisites

**Infrastructure**:
- Kubernetes cluster (production) or Docker Compose (staging)
- PostgreSQL database (agent configs, execution logs)
- Redis (rate limiting, execution queue)
- Message queue (RabbitMQ/Kafka for event bus)

**External Services**:
- Retell AI account + API key (for Voice channel)
- WhatsApp Business API access (for WhatsApp channel)
- SMS gateway account (Twilio, MessageBird)
- SMTP service (SendGrid, Mailgun)

### Architecture Components to Deploy

1. **Portal UI** (React/Vue.js app)
   - Build: `npm run build`
   - Deploy: Static hosting (S3 + CloudFront, Netlify, Vercel)

2. **Agent Configuration API** (Node.js/Python service)
   - Endpoints: CRUD for agents, KB management
   - Database: PostgreSQL

3. **Trigger Management Service** (Node.js/Python service)
   - Endpoints: POST /trigger, batch trigger
   - Event subscriptions: Internal event bus
   - Queue: RabbitMQ/Kafka

4. **Agent Framework Service** (Python service with NLP)
   - Integrates with Retell AI
   - Handles conversation execution
   - Extracts structured data

5. **Channels Service** (Node.js/Python service)
   - Integrates with WhatsApp Business API
   - SMS gateway integration
   - Email SMTP integration

6. **Execution Logging Service** (Node.js/Python service)
   - Writes logs to time-series database (InfluxDB, TimescaleDB)
   - Provides query API for analytics

7. **Analytics Service** (Python service)
   - Aggregates metrics
   - Generates dashboards
   - Alerts on anomalies

### Environment Variables

```env
# Database
DATABASE_URL=postgresql://user:pass@host:5432/outreach_db

# Redis
REDIS_URL=redis://host:6379

# Message Queue
RABBITMQ_URL=amqp://user:pass@host:5672

# External Services
RETELL_AI_API_KEY=sk_retell_xxx
WHATSAPP_BUSINESS_API_KEY=xxx
TWILIO_ACCOUNT_SID=xxx
TWILIO_AUTH_TOKEN=xxx
SENDGRID_API_KEY=xxx

# Feature Flags
DOCUMENTS_MODULE_ENABLED=true
JOURNEY_TRACKER_ENABLED=true

# Rate Limits
RATE_LIMIT_VOICE_PER_DAY=3
RATE_LIMIT_WHATSAPP_PER_DAY=5
RATE_LIMIT_SMS_PER_DAY=10
```

### Database Schema

**agents** table:
```sql
CREATE TABLE agents (
  agent_id UUID PRIMARY KEY,
  version INTEGER NOT NULL,
  channel VARCHAR(50) NOT NULL,
  role VARCHAR(50) NOT NULL,
  name VARCHAR(255) NOT NULL,
  description TEXT,
  program_id UUID,
  knowledge_bases JSONB,
  configuration JSONB,
  prompt TEXT,
  document_template_id UUID,
  trigger JSONB,
  status VARCHAR(50) NOT NULL,
  created_at TIMESTAMP NOT NULL,
  updated_at TIMESTAMP NOT NULL
);
```

**executions** table:
```sql
CREATE TABLE executions (
  execution_id UUID PRIMARY KEY,
  agent_id UUID REFERENCES agents(agent_id),
  user_id UUID NOT NULL,
  channel VARCHAR(50) NOT NULL,
  status VARCHAR(50) NOT NULL,
  started_at TIMESTAMP,
  completed_at TIMESTAMP,
  outcome JSONB,
  transcript TEXT,
  created_at TIMESTAMP NOT NULL
);

CREATE INDEX idx_executions_agent_id ON executions(agent_id);
CREATE INDEX idx_executions_user_id ON executions(user_id);
CREATE INDEX idx_executions_status ON executions(status);
```

---

## File Structure

```
/Outreach Module Prototype/
├── index.html                              # Home page with links to views
├── outreach-module-v3.html                 # Main V3 prototype (8-step flow)
├── system-architecture-v3.html             # V3 architecture visualization
├── README-V3.md                            # This file
├── V2-CHANGES.md                           # V2 changelog (previous version)
├── logo.png                                # Lillia logo
│
├── outreach-module-prototype.html          # V1 (original, 5-step)
├── outreach-module-v2.html                 # V2 (6-step with multi-program)
└── system-architecture-visualization.html  # V1 architecture
```

---

## Migration from V2 to V3

### Breaking Changes

1. **Multi-Channel → Single-Channel**:
   - V2 agents with multiple channels must be split into separate agents
   - Use Duplicate feature to create channel variants

2. **Channel Fallback Removed**:
   - V2 had automatic fallback (Voice → WhatsApp → SMS)
   - V3 has retry within same channel only
   - For fallback behavior, create multiple agents with different triggers

3. **Programs No Longer Required**:
   - V2 agents always linked to program
   - V3 agents can exist without program association
   - Existing agents: program link preserved

4. **Trigger Configuration Changed**:
   - V2 had "Independent" vs "Journey-Controlled" toggle
   - V3 has three distinct modes (API, Internal, Journey Task)
   - Migration: "Journey-Controlled" → "Journey Task Link", "Independent" → "API" or "Internal"

### Migration Steps

1. **Export V2 Agents**: Export all agent configurations from V2

2. **For Each Multi-Channel Agent**:
   - Create primary agent (most-used channel)
   - Duplicate for each additional channel
   - Adjust channel-specific settings (SMS character limits, etc.)

3. **Update Trigger Configuration**:
   - Journey-controlled agents → "Journey Task Link" mode
   - Scheduled agents → "Internal Ecosystem" mode with time-based event
   - External-triggered agents → "API/Webhook" mode

4. **Add Role Selection**:
   - Review agent purpose
   - Assign role: Patients, Caregivers, or Care Providers

5. **Test in Staging**:
   - Trigger each migrated agent
   - Verify conversation flow
   - Check document template population

6. **Deploy to Production**:
   - Activate migrated agents
   - Monitor first 24 hours closely
   - Compare metrics with V2 baseline

---

## Support & Feedback

For questions about this prototype or the Outreach Module V3 architecture:
- **Product Team**: Product feedback and feature requests
- **Engineering Team**: Technical implementation questions
- **Documentation**: Refer to V3 specification document

---

**Built with care for healthcare.**

*Version 3.0 | February 2026 | Lillia Health*
