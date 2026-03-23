# Digital Waterfall — Product Specification

**Product:** Matrix Digital Waterfall
**Owner:** Chase Butler, Healthware Systems
**Version:** 1.0 Draft
**Date:** March 23, 2026
**Status:** Discovery / Design

---

## 1. Executive Summary

Matrix Digital Waterfall is a configurable omnichannel communication platform that automates message delivery across SMS, email, and print channels. Unlike competitors (RevSpring, Renkim) who offer black-box or fixed-sequence workflows, Digital Waterfall gives the sales team full autonomy to configure per-client escalation timing, retry logic, and channel prioritization — without engineering involvement.

The product solves three core problems:
1. **No client configurability** — today every customization requires dev work
2. **No file-level tracking** — messages get lost as they move between channels and systems
3. **No compliance visibility** — audit trails and access controls are manual or absent

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

## 3. Competitive Landscape

| Capability | RevSpring (eVoke) | Renkim | Matrix Digital Waterfall |
|---|---|---|---|
| Channel sequence | AI-decided | Email → Text → Print (fixed) | Configurable per client |
| Client autonomy | Low — trust the algorithm | Low — one workflow | High — sales configures |
| Timing control | None exposed | None exposed | Per-gate, per-channel |
| File-level tracking | Unknown | Single data flow | Real-time per-message |
| Signal sensitivity | Persona-based (OmniBrain) | Not specified | Configurable (delivery, open, click) |
| Compliance | Assumed | Assumed | SOC 2, HIPAA, TCPA built-in |

### Key Differentiator
RevSpring says "trust our AI." Renkim says "one flow fits all." Matrix says "here are the levers — tune it to your business."

---

## 4. Product Architecture

### 4.1 The Waterfall Model

Communication flows downward through channel tiers. Each tier has a **gate** — configurable logic that determines when a message spills over to the next level.

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

### 4.2 Gate Logic

Each gate between tiers evaluates multiple signals:

**RELEASE to next tier WHEN:**
- `hard_fail` — immediate (invalid number, hard bounce)
- `soft_fail_exhausted` — after N retries with backoff
- `no_delivery_receipt` — after X minutes/hours
- `delivered_no_open` — after X hours (email only, if tracking enabled)
- `delivered_no_action` — after X hours (clicked/responded)
- `time_elapsed` — max wait regardless of signals

**HOLD at current tier WHEN:**
- `within_quiet_hours` — don't release during restricted windows
- `pending_retry` — soft fail, retry scheduled
- `engagement_detected` — opened/clicked, stop the waterfall

**SKIP tier entirely WHEN:**
- `channel_disabled` — client doesn't use this channel
- `no_contact_data` — no phone number or email on file
- `opt_out` — consumer preference / regulatory requirement

### 4.3 What the Current Flow Misses

| Current (Binary) | Digital Waterfall |
|---|---|
| SMS fails → send email | SMS sent → wait for engagement window → if no delivery OR no response within threshold → release to email |
| Email bounces → print mail | Email delivered but unopened after X hours → release to mail |
| One path, no branching | Multiple signals: delivered, opened, clicked, responded, failed, opted-out |

**"Failed" is no longer the only trigger. Silence is a trigger. Time is a trigger. The client defines what "good enough" means at each tier.**

---

## 5. Configurable Levers (Sales-Facing)

### 5.1 Per-Channel Settings

**SMS Tier**
| Setting | Type | Default (Healthcare) | Range |
|---|---|---|---|
| Enabled | Toggle | On | On/Off |
| Max Retries | Number | 2 | 1–3 |
| Escalation Delay | Minutes | 10 | 2–60 |
| Quiet Hours Start | Time | 9:00 PM | — |
| Quiet Hours End | Time | 8:00 AM | — |
| Hard Fail Action | Select | Escalate Immediately | Escalate / Hold / Stop |

**Email Tier**
| Setting | Type | Default (Healthcare) | Range |
|---|---|---|---|
| Enabled | Toggle | On | On/Off |
| Soft Bounce Retries | Number | 2 | 1–3 |
| Escalation Delay | Hours | 8 | 1–72 |
| Track Opens | Toggle | On | On/Off |
| No-Open Escalation | Hours | 24 | 4–168 |

**Print / Mail Tier**
| Setting | Type | Default (Healthcare) | Range |
|---|---|---|---|
| Enabled | Toggle | On | On/Off |
| Batch Cutoff | Time | 6:00 PM | — |
| Hold Before Print | Days | 1 | 0–7 |
| Priority Override | Select | Critical = Same Day | Same Day / Standard Batch |
| Return Mail Scan | Toggle | On | On/Off |

### 5.2 Industry Templates

Pre-configured defaults that sales selects as a starting point. Templates are editable, not locked.

| Industry | SMS Delay | Email Delay | Print Hold | Notes |
|---|---|---|---|---|
| Healthcare | 10 min | 8 hrs | 1 day | HIPAA, TCPA enforced |
| Collections | 15 min | 12 hrs | 1 day | Reg F timing rules |
| Utilities | 15 min | 24 hrs | 2 days | Seasonal volume spikes |
| Custom | — | — | — | Blank slate |

### 5.3 Priority Tiers

Message urgency overrides default timing:

- **Critical** — Fastest escalation (appointment in 24 hrs, past-due final notice). Override: SMS 5 min, Email 4 hrs, Print same-day.
- **Standard** — Use industry template defaults.
- **Low** — Batched/delayed (marketing, general notices). Extended windows.

### 5.4 Guardrails (Operations-Set)

Operations defines min/max bounds. Sales configures freely within them.

| Guardrail | Purpose | Example |
|---|---|---|
| Min SMS escalation delay | Prevent instant-fire to email | 2 minutes minimum |
| Max print hold | Ensure letters eventually mail | 7 days maximum |
| TCPA quiet hours | Regulatory, non-negotiable | 9 PM–8 AM, cannot override |
| Max retries per tier | Prevent infinite loops | 3 maximum |

---

## 6. File Tracker

### 6.1 Problem
Files get combined, split, and handed off between systems. No single view shows where a message is right now.

### 6.2 Solution
Every message gets a unique waterfall ID (`WF-YYYYMMDD-NNNN`) and is tracked through every stage:

**Stages:**
1. Intake / Pre-Processing
2. Data Validation (Tier 0)
3. SMS Sent → SMS Delivered / SMS Failed
4. Gate 1 (waiting)
5. Email Queued → Email Sent → Email Opened / Email Bounced
6. Gate 2 (waiting)
7. Print Queue → Batched → Mailed
8. Complete

### 6.3 File Tracker UI
- Real-time table with status badges, progress bars, time-in-stage
- Filterable: All / In Transit / Escalated / Failed
- Searchable by File ID, recipient name, message type
- Each row links to full message journey audit trail

### 6.4 Tracked Fields Per Message
| Field | Description |
|---|---|
| File ID | Unique waterfall identifier |
| Recipient | Name (PHI masked for non-admin roles) |
| Message Type | Appt Reminder, Statement, Balance Due, etc. |
| Current Stage | Where the message is right now |
| Current Tier | Tier 1/2/3 or Gate 1/2 |
| Time in Stage | How long at current stage |
| Next Action | What happens next and when |
| Full History | Every state change, timestamped |

---

## 7. Security & Compliance

### 7.1 SOC 2 Type II Requirements

**Trust Service Criteria addressed:**

| Criteria | Implementation |
|---|---|
| Security | MFA required, SSO via Okta/Azure AD, IP allowlisting |
| Availability | 99.9% uptime SLA, redundant message queues |
| Processing Integrity | Every gate decision logged, message checksums verified |
| Confidentiality | AES-256 encryption at rest and in transit, PHI masking |
| Privacy | Opt-out respected, data retention policies per client, TCPA enforcement |

### 7.2 HIPAA Compliance
- PHI (patient name, contact info) encrypted at rest
- PHI masked in UI for non-admin roles (shows "J*** M*****")
- BAA (Business Associate Agreement) required per client
- Minimum necessary access principle enforced via RBAC

### 7.3 TCPA Compliance
- Quiet hours enforced system-wide (non-overridable)
- Opt-out processing within 24 hours
- Consent tracking per recipient per channel
- All SMS includes required opt-out language

### 7.4 Audit Log

**Every action is logged immutably:**

| Log Field | Description |
|---|---|
| Timestamp | UTC, millisecond precision |
| Actor | User name + role |
| Action | What was done (config change, login, export, etc.) |
| Target | Which account/message/setting was affected |
| IP Address | Source IP of the action |
| Previous Value | For config changes — what it was before |
| New Value | For config changes — what it is now |
| Result | Success / Blocked / Failed |

**Retention:** 90 days online, 7 years archived (SOC 2 requirement).

**Alert Events:**
- Failed login attempts (lock after 3)
- PHI access by non-admin roles (blocked + logged)
- Guardrail enforcement (sales tried to exceed bounds)
- Bulk export of message data
- Role/permission changes

---

## 8. Role-Based Access Control (RBAC)

### 8.1 Role Definitions

| Role | Description | Primary Use |
|---|---|---|
| **Admin** | Full platform access, manages users, sets guardrails | Platform owner |
| **Sales** | Creates client profiles, configures gate timing, exports reports | Client onboarding + tuning |
| **Operations** | Sets guardrails, views audit logs, pauses waterfall, monitors compliance | Day-to-day oversight |
| **Client** | View-only access to their own file tracker and delivery reports | Client transparency |

### 8.2 Permission Matrix

| Permission | Admin | Sales | Operations | Client |
|---|---|---|---|---|
| Edit gate timing | Yes | Yes | No | No |
| Set guardrails (min/max bounds) | Yes | No | Yes | No |
| Create/edit client profiles | Yes | Yes | No | No |
| View file tracker | Yes | Yes | Yes | Yes (own account) |
| Export reports | Yes | Yes | Yes | No |
| View audit log | Yes | No | Yes | No |
| Manage users & roles | Yes | No | No | No |
| Pause/resume waterfall | Yes | No | Yes | No |
| Access PHI/PII (unmasked) | Yes | Masked | Masked | Masked |
| Change industry template | Yes | Yes | No | No |
| Override guardrails | Yes | No | No | No |

### 8.3 Authentication
- SSO required (Okta, Azure AD, SAML 2.0)
- MFA enforced for all roles
- Session timeout: 30 min inactive
- IP allowlisting available per account

---

## 9. Build Phases

### Phase 1 — Foundation (MVP)
**Goal:** Core waterfall with configurable timing and file tracking.

- Data model: client accounts, channel configs, gate settings
- Industry templates (Healthcare, Collections, Utilities, Custom)
- Per-channel enable/disable + timing configuration
- Admin-set guardrails (min/max bounds)
- File tracker with real-time status
- Basic audit logging (config changes, logins)
- RBAC with 4 roles (Admin, Sales, Operations, Client)
- AES-256 encryption, MFA, SSO

### Phase 2 — Visual Builder + Compliance
**Goal:** Sales-friendly UI and SOC 2 readiness.

- Visual waterfall canvas with drag-and-drop gate editing
- Timeline simulator ("what happens to a message sent now?")
- Priority tiers (Critical / Standard / Low)
- Full SOC 2 audit log with IP tracking, alert events
- Compliance dashboard with exportable reports
- PHI masking for non-admin roles
- TCPA quiet hours enforcement (non-overridable)

### Phase 3 — Intelligence
**Goal:** Data-driven optimization.

- Delivery analytics per client (% resolved at each tier)
- Cost analysis (SMS vs email vs print cost per message)
- Timing optimization suggestions based on historical data
- A/B testing for different gate configurations
- Persona-based defaults (demographic scoring for channel preference)
- Data validation gate (Tier 0) — verify contact data before entering waterfall

---

## 10. Key Metrics

| Metric | Description | Target |
|---|---|---|
| Tier 1 Resolution Rate | % of messages delivered via SMS | > 75% |
| Tier 2 Resolution Rate | % caught at email | < 20% |
| Tier 3 Fallthrough Rate | % reaching print | < 8% |
| Undeliverable Rate | % not delivered on any channel | < 2% |
| Mean Time to Delivery | Average time from intake to confirmed delivery | < 30 min (SMS), < 12 hrs (email) |
| Config Change Frequency | How often sales adjusts client settings | Indicates product-market fit |
| Audit Log Coverage | % of actions logged | 100% |

---

## 11. Mockup

Interactive HTML mockup available at:
`digital-waterfall-mockup.html`

Includes:
- Dashboard with live delivery stats
- Waterfall flow visualization with gate controls
- Channel settings with industry template tabs
- File tracker table with real-time pipeline status
- Compliance status banner (SOC 2, HIPAA, TCPA, AES-256)
- Role-based access panel with full permissions matrix
- Audit log with timestamped, IP-tracked events
- Message journey timeline preview

---

## 12. Open Questions

1. **Channel ordering** — Should we support non-standard sequences (e.g., email-first for older demographics, print-first for regulatory)? Renkim uses Email → Text → Print.
2. **Persona scoring** — RevSpring uses AI to pick channels per-patient. Do we want demographic-based defaults in Phase 3?
3. **Real-time vs batch** — Is the file tracker real-time websocket or polling? Cost/complexity tradeoff.
4. **Client self-service** — Should clients (not just sales) eventually be able to adjust their own timing within guardrails?
5. **Integration** — What EHR/PMS systems need first-party integration for intake files?
6. **Pricing model** — Per message? Per channel? Flat fee per client account?

---

*Document generated March 23, 2026. For questions contact Chase Butler.*
