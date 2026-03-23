# Digital Waterfall — Complete Product Package

**Product:** Matrix Digital Waterfall
**Owner:** Chase Butler, Healthware Systems
**Version:** 1.1
**Date:** March 23, 2026
**Updated:** March 23, 2026 — Added Analytics Dashboard + Cost Analysis
**Status:** Prototype Complete / Ready for Engineering Handoff

---

## Table of Contents

1. [Executive Summary](#1-executive-summary)
2. [Problem Statement](#2-problem-statement)
3. [Competitive Analysis](#3-competitive-analysis)
4. [Product Architecture](#4-product-architecture)
5. [Waterfall Model & Gate Logic](#5-waterfall-model--gate-logic)
6. [Configurable Levers](#6-configurable-levers)
7. [File Tracker System](#7-file-tracker-system)
8. [Security & Compliance](#8-security--compliance)
9. [Role-Based Access Control](#9-role-based-access-control)
10. [Audit Log System](#10-audit-log-system)
11. [Analytics Dashboard](#11-analytics-dashboard)
12. [Cost Analysis Engine](#12-cost-analysis-engine)
13. [UI/UX Specification](#13-uiux-specification)
14. [Technical Architecture](#14-technical-architecture)
15. [Database Schema](#15-database-schema)
16. [API Specification](#16-api-specification)
17. [Build Phases & Roadmap](#17-build-phases--roadmap)
18. [Key Metrics](#18-key-metrics)
19. [Open Questions](#19-open-questions)
20. [Appendix: Prototype Files](#20-appendix-prototype-files)

---

## 1. Executive Summary

Matrix Digital Waterfall is a configurable omnichannel communication platform that automates message delivery across SMS, email, and print channels. Unlike competitors (RevSpring, Renkim) who offer black-box or fixed-sequence workflows, Digital Waterfall gives the sales team full autonomy to configure per-client escalation timing, retry logic, and channel prioritization — without engineering involvement.

**Three core problems solved:**
1. No client configurability — every customization requires dev work today
2. No file-level tracking — messages get lost as they move between channels and combined systems
3. No compliance visibility — audit trails and access controls are manual or absent

**Origin:** Based on the existing Matrix omnichannel workflow (SMS → Email → Print), this product transforms a linear pass/fail system into a signal-driven, configurable waterfall with real-time tracking and SOC 2 compliance.

---

## 2. Problem Statement

### Current State
- Matrix operates a linear pass/fail omnichannel flow: SMS fails → email → email bounces → print
- Escalation timing is hardcoded — no per-client adjustment
- File tracking across combined batches is manual and error-prone
- Sales cannot demonstrate or configure flexibility during the sales cycle
- No centralized audit trail for SOC 2 or HIPAA compliance

### Desired State
- Sales configures per-client waterfall timing through a self-service UI
- Every message is tracked in real-time from intake through final delivery
- Escalation is signal-driven (not just pass/fail) — silence, time, and engagement are triggers
- Full audit logging and role-based access for SOC 2 Type II readiness

---

## 3. Competitive Analysis

### RevSpring (eVoke Digital First)
- **Approach:** "Digital first" — prioritize email/text, print only if action isn't taken
- **Secret sauce:** "OmniBrain" — predictive models that pick channel, frequency, and messaging per-patient persona
- **What they DON'T expose:** Timing levers to clients. Their AI decides. Clients don't configure the waterfall
- **Weakness:** Black box. Client has no autonomy. No transparency into why a message went to a specific channel
- **Scale:** 1.5 billion communications, 300 million digital annually
- **Products:** eVoke Digital First, Text to Pay, IVR Advantage, PersonaPay
- **URL:** https://revspringinc.com/

### Renkim
- **Approach:** Email → Text → Print (different order than Matrix — email first)
- **Key insight:** "Every day of delay reduces engagement and payment performance" — speed is everything
- **Data-first:** They validate/score contact data before entering the waterfall. Bad data never enters the flow
- **Unified:** "One data flow, one compliance framework, one accountable partner"
- **Weakness:** No configurable timing. One-size-fits-all sequence
- **URL:** https://renkim.com/

### Matrix Digital Waterfall Positioning

| Capability | RevSpring | Renkim | Matrix Digital Waterfall |
|---|---|---|---|
| Channel sequence | AI-decided | Email → Text → Print (fixed) | **Configurable per client** |
| Client autonomy | Low — trust the algorithm | Low — one workflow | **High — sales configures** |
| Timing control | None exposed | None exposed | **Per-gate, per-channel** |
| File-level tracking | Unknown | Single data flow | **Real-time per-message** |
| Signal sensitivity | Persona-based (OmniBrain) | Not specified | **Configurable (delivery, open, click)** |
| Compliance | Assumed | Assumed | **SOC 2, HIPAA, TCPA built-in** |
| Sales enablement | Limited demo capability | Limited | **Live config during sales call** |

**Key Differentiator:** RevSpring says "trust our AI." Renkim says "one flow fits all." Matrix says "here are the levers — tune it to your business."

### What We Borrowed From Competitors
1. **Data validation gate (Renkim):** Tier 0 pre-processing that validates phone/email before the waterfall starts. Bad data skips straight to print.
2. **Persona-based defaults (RevSpring):** Industry templates as starting points, with future potential for demographic scoring.

---

## 4. Product Architecture

### System Overview

```
┌─────────────────────────────────────────────────────────────────┐
│                    Matrix Digital Waterfall                       │
├──────────────┬──────────────┬──────────────┬───────────────────┤
│   Config UI  │  Waterfall   │   File       │   Compliance      │
│   (Sales)    │  Engine      │   Tracker    │   & Audit         │
├──────────────┴──────────────┴──────────────┴───────────────────┤
│                        API Layer                                 │
├──────────────┬──────────────┬──────────────┬───────────────────┤
│  Client      │  Gate        │  Message     │   Audit           │
│  Profiles    │  Rules       │  Queue       │   Log             │
├──────────────┴──────────────┴──────────────┴───────────────────┤
│                      Database Layer                              │
├──────────────┬──────────────┬──────────────┬───────────────────┤
│  PostgreSQL  │  Redis       │  S3/Blob     │   Elasticsearch   │
│  (Config)    │  (Queue)     │  (Files)     │   (Audit/Search)  │
└──────────────┴──────────────┴──────────────┴───────────────────┘
```

### Core Components

1. **Config UI** — Web application where sales/admin configure client waterfall settings
2. **Waterfall Engine** — Backend service that processes messages through gate logic
3. **File Tracker** — Real-time tracking of every message through the pipeline
4. **Compliance & Audit** — Immutable logging, RBAC, encryption, reporting

---

## 5. Waterfall Model & Gate Logic

### The Waterfall

```
Input File
    │
    ▼
┌──────────────┐
│  Tier 0      │  Data validation — verify phone/email before entering flow
│  Pre-Process │  Bad data skips directly to print
└──────┬───────┘
       │
       ▼
┌──────────────┐
│  Tier 1      │  Fastest, cheapest, highest engagement
│  SMS         │
└──────┬───────┘
       │
   [ Gate 1 ]──── Configurable: wait time, retry count, fail action
       │
       ▼
┌──────────────┐
│  Tier 2      │  Broader reach, more content capacity
│  Email       │
└──────┬───────┘
       │
   [ Gate 2 ]──── Configurable: bounce handling, open tracking, escalation delay
       │
       ▼
┌──────────────┐
│  Tier 3      │  Guaranteed delivery, regulatory proof
│  Print/Mail  │
└──────────────┘
```

### Gate Logic (The Core Innovation)

Each gate between tiers evaluates multiple signals — not just pass/fail:

**RELEASE to next tier WHEN:**
- `hard_fail` — immediate (invalid number, hard bounce)
- `soft_fail_exhausted` — after N retries with backoff
- `no_delivery_receipt` — after X minutes/hours
- `delivered_no_open` — after X hours (email only, if tracking enabled)
- `delivered_no_action` — after X hours (clicked/responded)
- `time_elapsed` — max wait regardless of signals

**HOLD at current tier WHEN:**
- `within_quiet_hours` — don't release during restricted windows (TCPA)
- `pending_retry` — soft fail, retry scheduled
- `engagement_detected` — opened/clicked, stop the waterfall

**SKIP tier entirely WHEN:**
- `channel_disabled` — client doesn't use this channel
- `no_contact_data` — no phone number or email on file
- `opt_out` — consumer preference / regulatory requirement

### Signal-Driven vs Binary

| Current (Binary) | Digital Waterfall |
|---|---|
| SMS fails → send email | SMS sent → wait for engagement window → if no delivery OR no response within threshold → release to email |
| Email bounces → print mail | Email delivered but unopened after X hours → release to mail |
| One path, no branching | Multiple signals: delivered, opened, clicked, responded, failed, opted-out |

**"Failed" is no longer the only trigger. Silence is a trigger. Time is a trigger. The client defines what "good enough" means at each tier.**

---

## 6. Configurable Levers

### Per-Channel Settings

**SMS Tier**
| Setting | Type | Default (Healthcare) | Range | Guardrail |
|---|---|---|---|---|
| Enabled | Toggle | On | On/Off | — |
| Max Retries | Number | 2 | 1–3 | Max 3 |
| Escalation Delay | Minutes | 10 | 2–60 | Min 2 min |
| Quiet Hours Start | Time | 9:00 PM | — | Non-overridable (TCPA) |
| Quiet Hours End | Time | 8:00 AM | — | Non-overridable (TCPA) |
| Hard Fail Action | Select | Escalate Immediately | Escalate/Hold/Stop | — |

**Email Tier**
| Setting | Type | Default (Healthcare) | Range | Guardrail |
|---|---|---|---|---|
| Enabled | Toggle | On | On/Off | — |
| Soft Bounce Retries | Number | 2 | 1–3 | Max 3 |
| Escalation Delay | Hours | 8 | 1–72 | — |
| Track Opens | Toggle | On | On/Off | — |
| No-Open Escalation | Hours | 24 | 4–168 | — |

**Print / Mail Tier**
| Setting | Type | Default (Healthcare) | Range | Guardrail |
|---|---|---|---|---|
| Enabled | Toggle | On | On/Off | — |
| Batch Cutoff | Time | 6:00 PM | — | — |
| Hold Before Print | Days | 1 | 0–7 | Max 7 days |
| Priority Override | Select | Critical = Same Day | Same Day/Standard Batch | — |
| Return Mail Scan | Toggle | On | On/Off | — |

### Industry Templates

| Industry | SMS Delay | SMS Retries | Email Delay | Email Retries | Notes |
|---|---|---|---|---|---|
| Healthcare | 10 min | 2 | 8 hrs | 2 | HIPAA, TCPA enforced |
| Collections | 15 min | 3 | 12 hrs | 2 | Reg F timing rules |
| Utilities | 15 min | 2 | 24 hrs | 1 | Seasonal volume |
| Custom | — | — | — | — | Blank slate |

### Priority Tiers
- **Critical** — Fastest escalation (appointment in 24 hrs, past-due final notice). SMS 5 min, Email 4 hrs, Print same-day.
- **Standard** — Use industry template defaults.
- **Low** — Batched/delayed (marketing, general notices). Extended windows.

### Guardrails (Operations-Set)
| Guardrail | Purpose | Enforcement |
|---|---|---|
| Min SMS escalation delay | Prevent instant-fire to email | 2 min minimum, hard block |
| Max print hold | Ensure letters eventually mail | 7 days maximum |
| TCPA quiet hours | Regulatory, non-negotiable | 9 PM–8 AM, cannot override |
| Max retries per tier | Prevent infinite loops | 3 maximum |

---

## 7. File Tracker System

### Problem
Files get combined, split, and handed off between systems. No single view shows where a message is right now.

### Solution
Every message gets a unique waterfall ID (`WF-YYYYMMDD-NNNN`) and is tracked through every stage.

### Pipeline Stages
1. Intake / Pre-Processing
2. Data Validation (Tier 0)
3. SMS Sent → SMS Delivered / SMS Failed
4. Gate 1 (waiting — countdown visible)
5. Email Queued → Email Sent → Email Opened / Email Bounced
6. Gate 2 (waiting — countdown visible)
7. Print Queue → Batched → Mailed
8. Complete

### Tracked Fields Per Message
| Field | Description |
|---|---|
| File ID | Unique waterfall identifier (WF-YYYYMMDD-NNNN) |
| Recipient | Name (PHI masked for non-admin roles) |
| Message Type | Appt Reminder, Statement, Balance Due, Pre-Visit, Past Due |
| Current Stage | Where the message is right now |
| Current Tier | Tier 1/2/3 or Gate 1/2 |
| Progress | Visual percentage through current stage |
| Time in Stage | How long at current stage |
| Next Action | What happens next and when (countdown) |
| Full History | Every state change, timestamped |

### UI Features
- Real-time table with status badges, progress bars, time-in-stage
- Filterable: All / In Transit / Escalated / Failed
- Searchable by File ID, recipient name, message type
- Each row links to full message journey audit trail
- Sortable columns

---

## 8. Security & Compliance

### SOC 2 Type II Requirements

| Trust Criteria | Implementation |
|---|---|
| Security | MFA required, SSO (Okta/Azure AD/SAML 2.0), IP allowlisting |
| Availability | 99.9% uptime SLA, redundant message queues, failover |
| Processing Integrity | Every gate decision logged, message checksums, idempotent processing |
| Confidentiality | AES-256 encryption at rest and in transit, PHI masking by role |
| Privacy | Opt-out respected, data retention policies per client, TCPA enforcement |

### HIPAA Compliance
- PHI (patient name, contact info) encrypted at rest (AES-256)
- PHI masked in UI for non-admin roles (shows "J*** M*****")
- BAA (Business Associate Agreement) required per client
- Minimum necessary access principle enforced via RBAC
- Audit log captures all PHI access attempts (including blocked)

### TCPA Compliance
- Quiet hours enforced system-wide (non-overridable by any role)
- Opt-out processing within 24 hours
- Consent tracking per recipient per channel
- All SMS includes required opt-out language
- Guardrail prevents sales from overriding quiet hours

---

## 9. Role-Based Access Control

### Role Definitions

| Role | Description | Primary Use |
|---|---|---|
| **Admin** | Full platform access, manages users, sets guardrails, overrides | Platform owner (Chase Butler) |
| **Sales** | Creates client profiles, configures gate timing, exports reports | Client onboarding + tuning |
| **Operations** | Sets guardrails, views audit logs, pauses waterfall, monitors | Day-to-day oversight |
| **Client** | View-only access to their own file tracker and delivery reports | Client transparency |

### Permission Matrix

| Permission | Admin | Sales | Operations | Client |
|---|---|---|---|---|
| Edit gate timing | Yes | Yes | No | No |
| Set guardrails (min/max) | Yes | No | Yes | No |
| Create/edit client profiles | Yes | Yes | No | No |
| View file tracker | Yes | Yes | Yes | Yes (own) |
| Export reports | Yes | Yes | Yes | No |
| View audit log | Yes | No | Yes | No |
| Manage users & roles | Yes | No | No | No |
| Pause/resume waterfall | Yes | No | Yes | No |
| Access PHI/PII (unmasked) | Yes | Masked | Masked | Masked |
| Change industry template | Yes | Yes | No | No |
| Override guardrails | Yes | No | No | No |

### Authentication
- SSO required (Okta, Azure AD, SAML 2.0)
- MFA enforced for all roles
- Session timeout: 30 min inactive
- IP allowlisting available per account
- Failed login lockout after 3 attempts

---

## 10. Audit Log System

### Every action is logged immutably:

| Field | Description |
|---|---|
| Timestamp | UTC, millisecond precision |
| Actor | User name + role |
| Action | What was done (config change, login, export, toggle, gate edit, etc.) |
| Target | Which account/message/setting was affected |
| IP Address | Source IP of the action |
| Previous Value | For config changes — what it was before |
| New Value | For config changes — what it is now |
| Result | Success / Blocked / Failed |

### Retention
- 90 days online (searchable, filterable)
- 7 years archived (SOC 2 requirement)
- Exportable as CSV/PDF

### Alert Events (logged + flagged)
- Failed login attempts (lock after 3)
- PHI access by non-admin roles (blocked + logged)
- Guardrail enforcement (sales tried to exceed bounds)
- Bulk export of message data
- Role/permission changes
- Waterfall pause/resume
- Client profile creation/deletion

### Audit Log Categories
- **Config** — Setting changes, profile creation, template switches
- **Auth** — Logins, logouts, failed attempts, SSO events
- **Alert** — Guardrail blocks, PHI access denied, security events
- **Export** — Report downloads, data exports

---

## 11. Analytics Dashboard

### Purpose
Give sales, operations, and clients real-time visibility into how the waterfall is performing — where messages resolve, how fast they deliver, and where failures happen. Data-driven proof that the configuration is working.

### KPI Bar (Top of Page)
| KPI | Value (Sample) | Trend | Why It Matters |
|---|---|---|---|
| Total Delivered | 97.4% | +1.2% vs last month | Overall waterfall effectiveness |
| Avg Delivery Time | 14 min | -3 min improvement | Speed of first successful touch |
| SMS Success Rate | 78.4% | +2.1% vs last month | Tier 1 resolution — cheapest channel |
| Bounce Rate | 4.8% | +0.3% vs last month | Data quality indicator |
| Print Fallthrough | 4.1% | -1.4% vs last month | Most expensive channel — lower is better |

### Charts (7 visualizations, all powered by Chart.js)

**1. Daily Delivery Volume** (Stacked Bar)
- X-axis: Days of week
- Stacks: SMS Delivered, Email Delivered, Print Queued, Failed
- Shows daily throughput and channel distribution at a glance
- Range selector: 7 Days / 30 Days / 90 Days

**2. Tier Resolution Breakdown** (Donut)
- SMS 78.4%, Email 16.2%, Print 4.1%, Undeliverable 1.3%
- Inner cutout shows total messages processed
- The single most important chart — proves the waterfall works

**3. Escalation Rate Over Time** (Area Line)
- Two lines: "Escalated to Email" and "Escalated to Print"
- Shows trends over 7/30/90 days
- Downward trend = waterfall is improving
- Directly tied to gate timing changes — shows cause and effect

**4. Average Time to Delivery** (Horizontal Bar)
- SMS: 2.4 minutes, Email: 48 minutes, Print: 24 hours
- Visual proof of why SMS-first saves time
- Helps clients understand the urgency argument

**5. Channel Performance Trend** (Multi-Line)
- 30-day success rate per channel (SMS, Email, Print)
- SMS typically 76-80%, Email 82-87%, Print 95-97%
- Print is high because it's the final fallback — always "delivered"

**6. Failure Reasons** (Horizontal Bar)
- Ranked: Invalid Number, Hard Bounce, Soft Bounce, Carrier Block, Opt-Out, No Address, Bad Email
- Identifies data quality issues — drives Tier 0 validation improvements
- Actionable: "Fix 842 invalid numbers to reduce email escalation by 6%"

**7. Hourly Message Volume** (Bar)
- 24-hour distribution of message sends
- Highlights TCPA quiet hours gap (9 PM – 8 AM = near zero)
- Shows peak send windows for capacity planning
- Color intensity indicates volume level

### Access by Role
| Role | Can View Analytics | Can Export |
|---|---|---|
| Admin | All clients | Yes |
| Sales | Assigned clients | Yes |
| Operations | All clients | Yes |
| Client | Own account only | No |

---

## 12. Cost Analysis Engine

### Purpose
The single most powerful sales tool in the platform. Shows clients exactly how much they spend per channel and how tuning the waterfall saves money. Print is 67% of spend but only 4% of messages — shifting even 1% from print to SMS saves ~$890/month.

### KPI Bar
| KPI | Value (Sample) | Context |
|---|---|---|
| Total Spend (MTD) | $18,432 | March 2026 |
| Cost Per Delivery | $0.047 | Blended across all channels |
| SMS Cost | $4,829 | $0.012 per message |
| Email Cost | $1,247 | $0.008 per message |
| Print Cost | $12,356 | $0.890 per letter |

### Charts (3 visualizations + cost table + savings calculator)

**1. Cost by Channel** (Donut)
- SMS: $4,829 (26.2%), Email: $1,247 (6.8%), Print: $12,356 (67.0%)
- The visual shock: print is 67% of spend for 4% of messages
- This chart alone sells the optimization conversation

**2. Cost Trend** (Stacked Area)
- 6-month trend showing print costs declining as SMS improves
- Oct: $22,400 total → Mar: $18,432 total
- Proves ROI of waterfall optimization over time

**3. Unit Cost Comparison** (Bar, Log Scale)
- SMS: $0.012, Email: $0.008, Print: $0.890
- Log scale because print is 74x more expensive than SMS
- Makes the channel economics viscerally clear

### Cost Breakdown Table
| Channel | Messages | Unit Cost | Total | % of Spend |
|---|---|---|---|---|
| SMS | 402,180 | $0.012 | $4,826 | 26.2% |
| Email | 155,875 | $0.008 | $1,247 | 6.8% |
| Print / Mail | 13,883 | $0.890 | $12,356 | 67.0% |

### Savings Calculator (The Sales Killer)
Highlighted prominently at the bottom of the cost page:

> **$6,240 estimated monthly savings** by increasing SMS resolution from 78% to 85%.
> Every 1% shift from print to SMS saves ~$890/month.

**How it works:**
- Current: 78.4% SMS → 16.2% Email → 4.1% Print
- Target: 85% SMS → 12% Email → 2% Print
- Delta: ~2,700 fewer print letters/month × $0.89 = $2,400 saved
- Plus reduced email volume = additional ~$300 saved
- Plus faster delivery = improved collection rates (not quantified yet)

**This is how sales closes the deal:** "Let me show you what happens to your costs when we tune these gates."

### Range Selectors
- Monthly / Quarterly / Yearly views
- Per-client data when switching accounts
- Exportable as PDF/CSV for client presentations

---

## 13. UI/UX Specification

### Design Language
- Clean white background with #f8fafc content area
- Card-based layout with 1px #e8ecf1 borders, 14px border-radius
- Inter font family, weights 400/500/600/700
- Primary accent: #0891b2 (teal)
- Channel colors: SMS #0891b2 (teal), Email #7c3aed (violet), Print #d97706 (amber)
- Status colors: Success #10b981 (green), Error #ef4444 (red), Warning #f59e0b (amber)
- 220px fixed sidebar, white background
- Generous whitespace (32-40px content padding)

### Pages (9 total, all functional in prototype)
1. **Dashboard** — Stats, waterfall flow visualization, message journey timeline
2. **Waterfall Builder** — Visual flow with clickable gates
3. **File Tracker** — Searchable/filterable table with real-time status
4. **Channel Settings** — Per-channel config cards with industry template tabs
5. **Delivery Rates** — 5 KPIs + 7 Chart.js visualizations (volume, tier breakdown, escalation trend, delivery time, channel performance, failure reasons, hourly volume)
6. **Cost Analysis** — 5 KPIs + 3 charts + cost breakdown table + savings calculator
7. **Compliance** — SOC 2/HIPAA/TCPA/encryption status badges
8. **Users & Roles** — Team list + full permissions matrix
9. **Audit Log** — Filterable event stream with timestamps and IPs

### Interactive Elements (all functional in prototype)
- Sidebar navigation switches between 9 pages
- Client dropdown switches accounts (stats, waterfall, charts all update)
- Toggle switches enable/disable channels (tiers gray out)
- Config inputs update waterfall and timeline in real-time
- Gate nodes open edit modal
- Industry template tabs swap all defaults
- File tracker search and filter tabs
- Audit log filter tabs (All/Config/Auth/Alerts) + live event logging
- Analytics range selectors (7 Day / 30 Day / 90 Day)
- Cost range selectors (Monthly / Quarterly / Yearly)
- 10 Chart.js charts across analytics and cost pages (bar, donut, line, area, horizontal bar)
- New Client modal creates accounts and adds to dropdown
- Guardrail enforcement (blocks SMS delay below 2 min with audit entry)
- Toast notifications on every action
- Role-based access permissions matrix
- Compliance status badges (SOC 2, HIPAA, TCPA, AES-256)

---

## 14. Technical Architecture

### Recommended Stack

| Layer | Technology | Rationale |
|---|---|---|
| Frontend | React + TypeScript | Component-based, type-safe, large ecosystem |
| UI Framework | Tailwind CSS | Matches the clean design system already established |
| State Management | Zustand or Redux Toolkit | Client config state, real-time updates |
| Backend | .NET 8 / C# or Node.js | Depends on existing Matrix stack |
| API | REST + WebSocket | REST for config CRUD, WebSocket for real-time file tracker |
| Database | PostgreSQL | Client configs, user/roles, gate rules |
| Queue | Redis / RabbitMQ | Message processing pipeline, retry scheduling |
| Search | Elasticsearch | Audit log search, file tracker search |
| File Storage | Azure Blob / S3 | Print files, export reports |
| Auth | Okta / Azure AD (SAML 2.0) | SSO, MFA |
| Hosting | Azure / AWS | SOC 2 compliant infrastructure |
| Monitoring | Datadog / Application Insights | Uptime, performance, alerting |

### Key Architectural Decisions
- **Event-sourced audit log** — Every state change is an immutable event. Never update, only append.
- **Account-scoped everything** — Every query, every API call is scoped to a client account. No cross-account data leaks.
- **Idempotent message processing** — Messages can be safely retried without duplicate sends.
- **Gate logic as configuration, not code** — Gate rules stored as data. No deploys needed for new client configs.

---

## 15. Database Schema

### Core Tables

```sql
-- Client accounts
CREATE TABLE accounts (
  id UUID PRIMARY KEY,
  name VARCHAR(255) NOT NULL,
  industry VARCHAR(50) NOT NULL,
  created_at TIMESTAMP DEFAULT NOW(),
  created_by UUID REFERENCES users(id),
  is_active BOOLEAN DEFAULT TRUE
);

-- Waterfall configuration per account
CREATE TABLE waterfall_config (
  id UUID PRIMARY KEY,
  account_id UUID REFERENCES accounts(id),
  -- SMS
  sms_enabled BOOLEAN DEFAULT TRUE,
  sms_max_retries INT DEFAULT 2 CHECK (sms_max_retries BETWEEN 1 AND 3),
  sms_escalation_delay_min INT DEFAULT 10 CHECK (sms_escalation_delay_min >= 2),
  sms_quiet_start TIME DEFAULT '21:00',
  sms_quiet_end TIME DEFAULT '08:00',
  sms_hard_fail_action VARCHAR(20) DEFAULT 'escalate',
  -- Email
  email_enabled BOOLEAN DEFAULT TRUE,
  email_max_retries INT DEFAULT 2 CHECK (email_max_retries BETWEEN 1 AND 3),
  email_escalation_delay_hrs INT DEFAULT 8,
  email_track_opens BOOLEAN DEFAULT TRUE,
  email_no_open_escalation_hrs INT DEFAULT 24,
  -- Print
  print_enabled BOOLEAN DEFAULT TRUE,
  print_batch_cutoff TIME DEFAULT '18:00',
  print_hold_days INT DEFAULT 1 CHECK (print_hold_days BETWEEN 0 AND 7),
  print_priority_override VARCHAR(20) DEFAULT 'same_day',
  print_return_scan BOOLEAN DEFAULT TRUE,
  -- Meta
  updated_at TIMESTAMP DEFAULT NOW(),
  updated_by UUID REFERENCES users(id)
);

-- Messages in the waterfall
CREATE TABLE messages (
  id UUID PRIMARY KEY,
  file_id VARCHAR(50) UNIQUE NOT NULL, -- WF-YYYYMMDD-NNNN
  account_id UUID REFERENCES accounts(id),
  recipient_name VARCHAR(255),
  recipient_phone VARCHAR(20),
  recipient_email VARCHAR(255),
  message_type VARCHAR(50), -- appt_reminder, statement, balance_due, etc.
  priority VARCHAR(10) DEFAULT 'standard', -- critical, standard, low
  current_stage VARCHAR(50),
  current_tier INT,
  created_at TIMESTAMP DEFAULT NOW(),
  completed_at TIMESTAMP
);

-- Message state transitions (event sourced)
CREATE TABLE message_events (
  id UUID PRIMARY KEY,
  message_id UUID REFERENCES messages(id),
  event_type VARCHAR(50), -- sms_sent, sms_delivered, sms_failed, gate1_enter, email_queued, etc.
  event_data JSONB,
  created_at TIMESTAMP DEFAULT NOW()
);

-- Users
CREATE TABLE users (
  id UUID PRIMARY KEY,
  email VARCHAR(255) UNIQUE NOT NULL,
  name VARCHAR(255) NOT NULL,
  role VARCHAR(20) NOT NULL, -- admin, sales, operations, client
  account_id UUID REFERENCES accounts(id), -- for client role, scopes access
  is_active BOOLEAN DEFAULT TRUE,
  created_at TIMESTAMP DEFAULT NOW()
);

-- Audit log (immutable, append-only)
CREATE TABLE audit_log (
  id UUID PRIMARY KEY,
  actor_id UUID REFERENCES users(id),
  actor_role VARCHAR(20),
  action VARCHAR(100),
  target_type VARCHAR(50), -- account, config, message, user
  target_id UUID,
  previous_value JSONB,
  new_value JSONB,
  ip_address INET,
  result VARCHAR(20), -- success, blocked, failed
  created_at TIMESTAMP DEFAULT NOW()
);

-- Guardrails (set by operations/admin)
CREATE TABLE guardrails (
  id UUID PRIMARY KEY,
  setting_key VARCHAR(50), -- sms_min_delay, print_max_hold, etc.
  min_value INT,
  max_value INT,
  is_overridable BOOLEAN DEFAULT FALSE,
  updated_by UUID REFERENCES users(id),
  updated_at TIMESTAMP DEFAULT NOW()
);

-- Daily delivery stats (materialized for fast analytics)
CREATE TABLE daily_stats (
  id UUID PRIMARY KEY,
  account_id UUID REFERENCES accounts(id),
  date DATE NOT NULL,
  total_messages INT DEFAULT 0,
  sms_delivered INT DEFAULT 0,
  sms_failed INT DEFAULT 0,
  email_delivered INT DEFAULT 0,
  email_bounced INT DEFAULT 0,
  print_queued INT DEFAULT 0,
  print_mailed INT DEFAULT 0,
  undeliverable INT DEFAULT 0,
  avg_delivery_time_sec INT, -- mean seconds to first successful delivery
  UNIQUE(account_id, date)
);

-- Hourly volume (for hourly distribution chart)
CREATE TABLE hourly_volume (
  id UUID PRIMARY KEY,
  account_id UUID REFERENCES accounts(id),
  date DATE NOT NULL,
  hour INT NOT NULL CHECK (hour BETWEEN 0 AND 23),
  message_count INT DEFAULT 0,
  UNIQUE(account_id, date, hour)
);

-- Cost tracking per channel per month
CREATE TABLE monthly_costs (
  id UUID PRIMARY KEY,
  account_id UUID REFERENCES accounts(id),
  month DATE NOT NULL, -- first of month
  sms_count INT DEFAULT 0,
  sms_cost DECIMAL(10,2) DEFAULT 0,
  email_count INT DEFAULT 0,
  email_cost DECIMAL(10,2) DEFAULT 0,
  print_count INT DEFAULT 0,
  print_cost DECIMAL(10,2) DEFAULT 0,
  UNIQUE(account_id, month)
);

-- Failure reasons (aggregated daily)
CREATE TABLE failure_stats (
  id UUID PRIMARY KEY,
  account_id UUID REFERENCES accounts(id),
  date DATE NOT NULL,
  reason VARCHAR(50), -- invalid_number, hard_bounce, soft_bounce, carrier_block, opt_out, no_address, bad_email
  count INT DEFAULT 0,
  UNIQUE(account_id, date, reason)
);
```

---

## 16. API Specification

### Accounts
| Method | Endpoint | Role | Description |
|---|---|---|---|
| GET | /api/accounts | Admin, Sales | List all accounts |
| POST | /api/accounts | Admin, Sales | Create new account |
| GET | /api/accounts/:id | All (scoped) | Get account detail |
| PUT | /api/accounts/:id | Admin, Sales | Update account |

### Waterfall Config
| Method | Endpoint | Role | Description |
|---|---|---|---|
| GET | /api/accounts/:id/config | Admin, Sales, Ops | Get waterfall config |
| PUT | /api/accounts/:id/config | Admin, Sales | Update config (guardrails enforced server-side) |
| POST | /api/accounts/:id/config/apply-template | Admin, Sales | Apply industry template |

### Messages / File Tracker
| Method | Endpoint | Role | Description |
|---|---|---|---|
| GET | /api/accounts/:id/messages | All (scoped) | List messages (paginated, filterable) |
| GET | /api/accounts/:id/messages/:fileId | All (scoped) | Get message detail + event history |
| WS | /ws/accounts/:id/messages | All (scoped) | Real-time message status updates |

### Analytics
| Method | Endpoint | Role | Description |
|---|---|---|---|
| GET | /api/accounts/:id/analytics/delivery | Admin, Sales, Ops, Client | Delivery stats (tier resolution, volume, trends) |
| GET | /api/accounts/:id/analytics/escalation | Admin, Sales, Ops | Escalation rates over time |
| GET | /api/accounts/:id/analytics/failures | Admin, Sales, Ops | Failure reason breakdown |
| GET | /api/accounts/:id/analytics/hourly | Admin, Sales, Ops | Hourly volume distribution |
| GET | /api/accounts/:id/analytics/channel-perf | Admin, Sales, Ops | Per-channel success rates over time |

### Cost Analysis
| Method | Endpoint | Role | Description |
|---|---|---|---|
| GET | /api/accounts/:id/costs/summary | Admin, Sales, Ops | Cost KPIs and channel breakdown |
| GET | /api/accounts/:id/costs/trend | Admin, Sales, Ops | Cost trend over time by channel |
| GET | /api/accounts/:id/costs/savings | Admin, Sales | Projected savings from optimization |
| GET | /api/accounts/:id/costs/export | Admin, Sales, Ops | Export cost report as PDF/CSV |

### Audit Log
| Method | Endpoint | Role | Description |
|---|---|---|---|
| GET | /api/audit | Admin, Ops | Query audit log (filterable by type, date, actor) |
| GET | /api/audit/export | Admin, Ops | Export audit log as CSV |

### Users
| Method | Endpoint | Role | Description |
|---|---|---|---|
| GET | /api/users | Admin | List users |
| POST | /api/users/invite | Admin | Invite user with role |
| PUT | /api/users/:id/role | Admin | Change user role |
| DELETE | /api/users/:id | Admin | Deactivate user |

---

## 17. Build Phases & Roadmap

### Phase 1 — Foundation (MVP)
**Goal:** Core waterfall with configurable timing and file tracking.
**Estimated scope:** 8-12 weeks with 2-3 engineers

- Database schema + migrations
- Account CRUD + client profiles
- Waterfall config per account (all channel settings)
- Industry templates (Healthcare, Collections, Utilities, Custom)
- Gate logic engine (timer-based escalation + hard fail escalation)
- File tracker with real-time status (WebSocket)
- Basic RBAC (4 roles)
- Authentication (SSO + MFA via Okta/Azure AD)
- AES-256 encryption at rest and in transit
- Basic audit logging (config changes, logins)
- Config UI (React) — settings page, file tracker, dashboard

### Phase 2 — Visual Builder + Analytics + Compliance
**Goal:** Sales-friendly UI, analytics dashboard, and SOC 2 readiness.
**Estimated scope:** 8-10 weeks

- Visual waterfall canvas with clickable gate editing
- Timeline simulator ("what happens to a message sent now?")
- Priority tiers (Critical / Standard / Low)
- **Delivery analytics dashboard** (7 charts — volume, tier breakdown, escalation trend, delivery time, channel performance, failure reasons, hourly volume)
- **Cost analysis engine** (3 charts + cost table + savings calculator)
- **KPI bars** with trend indicators across analytics and cost pages
- **Range selectors** (7D/30D/90D for analytics, Monthly/Quarterly/Yearly for costs)
- Full SOC 2 audit log with IP tracking, alert events, 90-day retention
- Compliance dashboard with exportable reports
- PHI masking for non-admin roles
- TCPA quiet hours enforcement (non-overridable guardrail)
- Guardrail enforcement engine (server-side validation)
- Notification system (toast + email alerts for ops)

### Phase 3 — Intelligence + Optimization
**Goal:** AI-driven optimization and client self-service.
**Estimated scope:** 8-10 weeks

- **Savings simulator** — interactive "what if" modeling (drag SMS target to see projected savings)
- Timing optimization suggestions based on historical data
- A/B testing framework for different gate configurations
- Persona-based defaults (demographic scoring for channel preference)
- Data validation gate (Tier 0) — verify contact data before entering waterfall
- Predictive model: suggest optimal timing per client based on outcomes
- Client self-service portal (view-only with upgrade path)
- Custom report builder with scheduled email delivery

---

## 18. Key Metrics

### Delivery Metrics
| Metric | Description | Target |
|---|---|---|
| Tier 1 Resolution Rate | % of messages delivered via SMS without escalation | > 75% |
| Tier 2 Resolution Rate | % caught at email | < 20% |
| Tier 3 Fallthrough Rate | % reaching print | < 8% |
| Undeliverable Rate | % not delivered on any channel | < 2% |
| Mean Time to Delivery | Average from intake to confirmed delivery | < 30 min (SMS), < 12 hrs (email) |
| Bounce Rate | Hard + soft bounces as % of total | < 5% |
| Hourly Peak Throughput | Max messages processed per hour | Capacity planning |

### Cost Metrics
| Metric | Description | Target |
|---|---|---|
| Blended Cost Per Delivery | Total spend / total messages | < $0.05 |
| SMS Unit Cost | Cost per SMS message | ~$0.012 |
| Email Unit Cost | Cost per email | ~$0.008 |
| Print Unit Cost | Cost per printed letter | ~$0.89 |
| Print Spend % | Print cost as % of total spend | < 60% (declining) |
| Monthly Savings | Cost reduction from waterfall optimization | Track month-over-month |
| Print-to-SMS Shift Rate | % of messages moved from print to SMS via tuning | Growth = ROI proof |

### Business Metrics
| Metric | Description | Target |
|---|---|---|
| Config Change Frequency | How often sales adjusts client settings | Growth = product-market fit |
| Sales Demo Conversion | % of demos using waterfall builder that convert | Track post-launch |
| Client Retention | Clients staying after first renewal | > 95% |
| Audit Log Coverage | % of actions with complete audit trail | 100% |
| System Uptime | Platform availability | 99.9% |

---

## 19. Open Questions

1. **Channel ordering** — Should we support non-standard sequences (email-first for older demographics, print-first for regulatory)?
2. **Persona scoring** — Do we want demographic-based channel defaults in Phase 3?
3. **Real-time architecture** — WebSocket vs polling for file tracker? Cost/complexity tradeoff.
4. **Client self-service** — Should clients eventually adjust their own timing within guardrails?
5. **EHR integration** — What systems need first-party integration for intake files?
6. **Pricing model** — Per message? Per channel? Per client flat fee?
7. **Existing Matrix integration** — How does this layer onto the current Letter Manager pipeline?
8. **Mobile** — Do sales reps need a mobile-responsive version for field demos?

---

## 20. Appendix: Prototype Files

### Delivered Files
| File | Location | Description |
|---|---|---|
| Interactive Mockup | `digital-waterfall-mockup.html` | Fully functional HTML/CSS/JS/Chart.js prototype — 9 pages, 10 charts, all interactive |
| Product Spec (v1) | `Digital_Waterfall_Product_Spec.md` | Initial product specification document |
| Complete Package (v1.1) | `Digital_Waterfall_Complete_Package.md` | This document — comprehensive product, technical, and design specification |

### Prototype Features (all functional)
- Sidebar navigation between 9 pages
- Client account switching with per-client stats (all charts + waterfall update)
- Toggle switches that enable/disable channel tiers (gray out in waterfall)
- Config inputs update waterfall visualization + timeline in real-time
- Gate edit modal with save/cancel + audit trail entry
- Industry template switching (Healthcare/Collections/Utilities/Custom)
- File tracker with search + filter tabs (All/In Transit/Escalated/Failed)
- Audit log with filter tabs (All/Config/Auth/Alerts) + live event logging
- New Client creation modal (adds to dropdown + audit log)
- Guardrail enforcement (blocks SMS delay below 2 min with audit entry)
- Toast notifications on every action
- Role-based access permissions matrix
- Compliance status badges (SOC 2, HIPAA, TCPA, AES-256)
- **10 Chart.js visualizations:**
  1. Daily Delivery Volume (stacked bar)
  2. Tier Resolution Breakdown (donut)
  3. Escalation Rate Over Time (area line)
  4. Average Time to Delivery (horizontal bar)
  5. Channel Performance Trend (multi-line, 30-day)
  6. Failure Reasons (horizontal bar, ranked)
  7. Hourly Message Volume (bar with quiet hours visualization)
  8. Cost by Channel (donut)
  9. Cost Trend (stacked area, 6-month)
  10. Unit Cost Comparison (bar, log scale)
- Cost breakdown table with per-channel unit economics
- Savings calculator: "$6,240/month by shifting 7% from print to SMS"
- Analytics range selectors (7 Day / 30 Day / 90 Day)
- Cost range selectors (Monthly / Quarterly / Yearly)

---

*Document generated March 23, 2026. All prototype files saved to C:\Users\cbutler\Downloads\*
*For questions contact Chase Butler, Healthware Systems.*
