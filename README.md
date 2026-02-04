# Lillia Outreach Module - Interactive Prototype

## Overview

This interactive prototype demonstrates the **Outreach Module** interface design and user flow based on the building blocks document. It shows how healthcare organizations can configure AI-powered outreach agents across multiple channels (Voice, WhatsApp, SMS, Email) with integrated knowledge bases, document templates, and scheduling systems.

## How to View the Prototype

1. Open `outreach-module-prototype.html` in any modern web browser
2. The prototype is fully interactive - click through the interface to explore different sections
3. No server or installation required - it's a standalone HTML file

## Design System

The prototype follows the **Lillia Brand Guidelines**:

### Typography
- Font Family: **Lato** (Light, Regular, Medium, SemiBold, Bold, Black)
- Clean, modern, healthcare-appropriate typeface

### Color Palette
- **Primary Purple**: `#7848FE` - Brand color for primary actions and active states
- **Purple Variants**: `#9F7BFF`, `#D3B9F9`, `#280470`, `#1405D6`
- **Lavender**: `#EADEFC` - Background highlights and selected states
- **Peach**: `#F9EAE4`, `#FFD2BB` - Accent colors for warmth
- **Greys**: Neutral palette for text and borders

## User Flow

### 1. Main Portal View
**Location**: Side panel navigation

- Users access the **Outreach** module from the left sidebar
- The navigation shows all major portal sections (Dashboard, Users, Programs, Outreach, Analytics)
- Outreach is highlighted as the active section

### 2. Agent List View
**Location**: Main content area when "Outreach" is selected

**Features**:
- Displays all existing agents as cards
- Each card shows:
  - Agent name and type
  - Status (Active/Draft)
  - Enabled channels (Voice, WhatsApp, SMS, Email)
  - Performance stats (Total Runs, Success Rate)
- **"+ Create Agent"** button in the header to start new agent creation
- Click any agent card to enter configuration mode

**Building Blocks Represented**:
- Agent Framework (each card represents a configured agent)
- Outreach Channels Layer (channel badges show which channels are enabled)
- Execution Logging & Monitoring Layer (stats come from this layer)

### 3. Agent Configuration View
**Location**: Activated when user clicks on an agent or creates a new one

The configuration flow is broken into **5 progressive steps**, each representing different system layers:

---

#### Step 1: Basic Info
**Purpose**: Define the agent's identity and purpose

**Fields**:
- Agent Name
- Agent Type (Onboarding, Reminder, Follow-up, Survey, Closure)
- Program (links to which care program this agent serves)
- Description

**Building Blocks Represented**:
- **Agent Framework** - Core agent configuration
- **Program Configuration Layer** - Program selection constrains what's available in later steps
- **Agent Configuration Engine** - All these settings are managed here

---

#### Step 2: Channels
**Purpose**: Select which communication channels the agent can use

**Features**:
- Visual channel cards for Voice, WhatsApp, SMS, Email
- Multi-select (agent can use multiple channels)
- Info box explaining **Channel Fallback** behavior
- Channels are attempted in priority order: Voice → WhatsApp → SMS → Email

**Building Blocks Represented**:
- **Outreach Channels Layer** - The delivery infrastructure
- **Channel Fallback Engine** - Automatically tries next channel if primary fails
- **Agent Configuration Engine** - Stores channel preferences per agent

**Key Concept Demonstrated**:
If WhatsApp fails to deliver, the system automatically tries SMS, then Email. This prevents message loss and ensures patients are reached.

---

#### Step 3: Knowledge Base
**Purpose**: Select which knowledge sources guide the agent's conversations

**Features**:
- List of available knowledge bases
- Each KB shows name, description, and priority level
- Priority levels: Highest (Compliance) → Medium (Program/Client) → Base (Agent default)
- Multi-select with visual checkbox indicators

**Knowledge Bases Shown**:
1. **Compliance & Safety Guardrails** (Highest Priority) - HIPAA, safety protocols, prohibited statements
2. **Diabetes Program Clinical Protocols** (Medium) - Treatment guidelines, medication info
3. **Partner-Specific Terminology** (Medium) - Branding, terminology, escalation contacts
4. **Base Reminder Agent Knowledge** (Base) - General reminder handling, conversation flows

**Building Blocks Represented**:
- **Knowledge Base Integration Layer** - Assembles knowledge before each conversation
- **Program Configuration Layer** - Determines which KBs are mandatory for the program
- **Agent Configuration Engine** - Manages KB selections and priority

**Key Concept Demonstrated**:
When there's a conflict between knowledge sources (e.g., base agent says one thing, compliance says another), the higher priority source always wins. This ensures regulatory compliance cannot be overridden.

---

#### Step 4: Templates
**Purpose**: Configure document generation from conversation data

**Features**:
- Select post-conversation template
- Preview of field mapping (shows which conversation data maps to which document fields)
- Examples: "Patient Confirmation → agent.response.confirmed"

**Building Blocks Represented**:
- **Document Template Integration Layer** - Generates structured documents from conversation data
- **Agent Configuration Engine** - Links templates to agents
- **Variable Injection Framework** - Maps dynamic values to template fields

**Key Concept Demonstrated**:
After an agent completes a conversation (e.g., medication reminder call), it automatically fills out an "Adherence Report" template with the captured information. Care teams get structured documentation without manual data entry.

---

#### Step 5: Scheduling
**Purpose**: Define when and how the agent executes

**Features**:
- **Toggle**: Journey Tracker Integration vs. Independent Scheduling
- Two distinct modes shown with different UI states

**Mode 1: Independent Scheduling** (Journey Tracker OFF)
- Execution Frequency (one-time, daily, weekly, custom cron)
- Execution Window (9am-6pm in user's timezone)
- Retry Policy (how many retry attempts, with what intervals)
- Stop Conditions (when to stop trying)

**Mode 2: Journey-Controlled** (Journey Tracker ON)
- Agent executes when care journey reaches specific task
- Scheduling managed by journey orchestrator
- Select which journey task triggers this agent

**Building Blocks Represented**:
- **Journey Tracker Integration Layer** (when toggle is ON)
- **Outreach Scheduler** (when toggle is OFF)
- **Trigger Management System** - Determines what starts the agent
- **Agent Configuration Engine** - Stores scheduling preferences

**Key Concept Demonstrated**:
An agent can either run independently (e.g., "send medication reminder every day at 9am") or be controlled by a care journey (e.g., "call patient when they reach Day 7 of diabetes program"). The system supports both modes, but an agent can only be in one mode at a time.

---

### 4. Layer Integration Visualization
**Location**: Bottom of Step 5 (Scheduling)

**Purpose**: Show which system layers are active for this configured agent

**Visual Design**:
- List of all system layers
- Active layers are highlighted with colored badges
- Each layer shows status (Configured, Auto-enabled, Disabled)

**Layers Shown**:
1. Agent Framework - ✓ Configured
2. Outreach Channels - ✓ Configured (with selected channels)
3. Knowledge Base Integration - ✓ 2 sources selected
4. Document Templates - ✓ Template mapped
5. Identity & Profile Resolution - Auto-enabled (always on)
6. Consent Management - Auto-enabled (always on)
7. Channel Fallback Engine - Auto-enabled (always on)
8. Journey Tracker Integration - Status changes based on toggle
9. Execution Logging & Monitoring - Auto-enabled (always on)

**Building Blocks Represented**:
All major system layers are shown, demonstrating how they integrate when an agent is configured.

**Key Concept Demonstrated**:
Some layers are **manually configured** (channels, knowledge bases, templates), while others are **automatically enabled** for every agent (consent checks, logging, fallback). The user doesn't configure these, but the visualization shows they're part of the system.

---

## Interactive Elements

### Clickable Components
- **Side navigation** - Navigate between portal sections
- **Agent cards** - Click to enter configuration mode
- **"+ Create Agent" button** - Start new agent creation
- **Progress steps** - Jump between configuration steps
- **Channel cards** - Select/deselect channels (visual selection state)
- **Knowledge base items** - Select/deselect KBs (checkbox interaction)
- **Journey Tracker toggle** - Switch between scheduling modes (changes UI below)
- **Navigation buttons** - Previous/Next to move through steps
- **"Back to Agents" button** - Return to agent list

### Visual States
- **Hover effects** on cards and buttons
- **Selected states** for channels and knowledge bases
- **Active/Completed states** for progress steps
- **Toggle animation** for Journey Tracker switch
- **Layer activation** visual feedback in the integration diagram

---

## Building Blocks Mapping

This prototype demonstrates the user-facing interface for configuring these **Outreach Module Building Blocks**:

| Building Block | Where It Appears in Prototype |
|----------------|-------------------------------|
| **Agent Framework** | Core of entire configuration flow; agent cards in list view |
| **Outreach Channels Layer** | Step 2: Channel selection grid |
| **Agent Configuration Engine** | Backend for all configuration steps |
| **Identity & Profile Resolution System** | Auto-enabled layer (shown in visualization) |
| **Outreach Scheduler** | Step 5: Independent scheduling mode |
| **Knowledge Base Integration Layer** | Step 3: KB selection and priority |
| **Document Template Integration Layer** | Step 4: Template selection and field mapping |
| **Journey Tracker Integration Layer** | Step 5: Journey-controlled mode toggle |
| **Trigger Management System** | Implied in scheduling configuration |
| **Execution Logging & Monitoring Layer** | Stats in agent cards; auto-enabled layer |
| **Program Configuration Layer** | Step 1: Program dropdown |
| **Variable Injection Framework** | Step 4: Template field mapping preview |
| **Consent Management Engine** | Auto-enabled layer (not configured by user) |
| **Channel Fallback Engine** | Info box in Step 2 explaining fallback behavior |

**NEW Elements** (from gap analysis) are represented as **auto-enabled layers** or **info boxes**:
- Consent Management, Channel Fallback, Escalation Management, Analytics are shown as auto-enabled
- Multi-language support would appear as a language selector in Step 1 (not shown in this prototype)
- Pre-activation validation would happen when user clicks "Activate Agent"
- Testing framework would be a separate view/panel (not shown in this prototype)

---

## Design Patterns Used

### Cards
- **Agent cards**: Summary view with key info and stats
- **Channel cards**: Large, icon-driven selection cards
- **Knowledge base items**: List-based cards with checkbox selection

### Progressive Disclosure
- 5-step configuration flow prevents overwhelming users
- Each step focuses on one aspect of agent configuration
- Progress indicator shows where user is in the flow

### Visual Hierarchy
- **Primary actions** (Create Agent, Activate Agent) use brand purple
- **Secondary actions** (Save Draft, Previous) use outlined style
- **Stats and metrics** use bold typography for emphasis

### Feedback & Affordance
- Selected states are visually distinct (highlighted borders, background color change)
- Hover states indicate clickability
- Toggle switches show clear on/off states
- Help text and info boxes explain complex concepts

### Responsive Layout
- Grid-based layouts adapt to different screen sizes
- Side panel can be collapsed on smaller screens
- Card grids reflow from multi-column to single-column

---

## Key User Experience Decisions

### 1. Progressive Configuration
Rather than a single massive form, the configuration is split into 5 focused steps. This:
- Reduces cognitive load
- Makes the process feel less overwhelming
- Allows users to focus on one concern at a time
- Matches the mental model of "setting up" an agent piece by piece

### 2. Visual Channel Selection
Channels are presented as large, icon-driven cards rather than checkboxes because:
- It makes the options more scannable
- It emphasizes that this is an important choice
- It allows space for descriptive text about each channel
- It feels more modern and engaging

### 3. Knowledge Base Priority Visualization
Priority levels are shown with colored badges (High = red, Medium = yellow, Low = grey) because:
- Priority conflicts are a complex concept
- Visual color coding makes it immediately clear which KB "wins"
- It reinforces the compliance-first approach

### 4. Journey Tracker Toggle
The toggle between Journey-controlled and Independent scheduling is prominent because:
- This is a fundamental decision that changes how the entire system operates
- The UI needs to adapt significantly based on this choice
- Users need to understand these are mutually exclusive modes

### 5. Layer Integration Visualization
Showing all integrated layers at the end serves multiple purposes:
- It builds confidence that the system is "complete" and ready
- It educates users about what's happening behind the scenes
- It surfaces auto-enabled features (like consent management) that users might not know exist
- It creates transparency about the system architecture

---

## Next Steps for Development

This prototype demonstrates the **UI/UX layer** for the Outreach Module. To build the production system, these components would need to be developed:

### Frontend
- React/Vue.js implementation of these screens
- State management for multi-step form
- API integration for saving configurations
- Real-time validation and error handling
- Accessibility improvements (ARIA labels, keyboard navigation)

### Backend
- Agent configuration API endpoints
- Knowledge base management system
- Template engine and field mapping
- Scheduling engine with timezone support
- Channel integration with telephony, WhatsApp, SMS, email providers
- Consent tracking and enforcement
- Execution logging and monitoring

### Additional Views Not Shown in Prototype
- Agent execution history and logs
- Analytics dashboard for agent performance
- User consent management panel
- Testing and simulation environment
- Escalation queue for human review
- Knowledge base creation and editing interface

---

## Feedback and Iteration

This prototype is designed for demonstration and stakeholder review. Key questions for feedback:

1. **Is the 5-step flow intuitive?** Or would a different organization work better?
2. **Is the Journey Tracker toggle clear?** Do users understand the difference between modes?
3. **Is the knowledge base priority system understandable?** Or does it need more explanation?
4. **Are there missing configuration options** that should be surfaced in this flow?
5. **Is the visual design appropriate** for a healthcare enterprise product?

---

## Technical Notes

### Browser Compatibility
- Works in all modern browsers (Chrome, Firefox, Safari, Edge)
- Uses CSS Grid and Flexbox for layout
- No external dependencies except Google Fonts (Lato)
- Fully functional without JavaScript (except interactive states)

### Performance
- Lightweight (single HTML file, ~50KB)
- No external API calls
- Fast load time
- No build process required

### Customization
The prototype can be easily customized by modifying:
- **Colors**: CSS custom properties in `:root`
- **Content**: Agent names, descriptions, knowledge bases
- **Layout**: Grid and flexbox properties
- **Branding**: Logo, typography

---

## File Structure

```
/Outreach Module Prototype/
├── outreach-module-prototype.html    # Main interactive prototype
├── README.md                          # This documentation
└── Lillia Mini Brand Manual.pdf      # Brand guidelines (reference)
```

---

## Version History

**Version 1.0** - January 2026
- Initial prototype release
- 5-step agent configuration flow
- Layer integration visualization
- Lillia brand compliance
- Fully interactive UI states

---

## Contact & Support

For questions about this prototype or the Outreach Module architecture, refer to:
- **Building Blocks Document**: Detailed specification of all system elements
- **Brand Manual**: Visual design guidelines and requirements
- **Product Team**: For feedback and feature requests

---

**Built with care for healthcare.**
