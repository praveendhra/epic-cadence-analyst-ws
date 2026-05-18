# Epic Cadence Optimization Initiative
## Rebuilding Patient Access for Northeast Regional Health
### Work Sample: Applications Analyst II — Epic Cadence

---

## Executive Summary

Northeast Regional Health (NRH) is a 14-facility community health system with 240 active providers running Epic 2023. After a go-live in March 2022, uncoordinated local build decisions created 847 active visit types (industry benchmark: ~120 for this scale), no referral workqueue governance, and provider templates modified ad hoc via email — no change ticket, no UAT review. The result: an 18% no-show rate, 62% template utilization, a scheduling cycle time of 4.2 days, and a 3rd Next Available averaging 12 days across specialties.

This initiative rebuilds Cadence configuration under a governance framework covering visit type rationalization, template standardization, HL7-based external referral capture, and MyChart self-scheduling expansion — targeting an 8% no-show rate, 85% utilization, 1.8-day cycle time, and 4-day 3NA within 12 months.

---

## Scenario

**Organization:** Northeast Regional Health (NRH), fictional composite  
**Size:** 14 ambulatory clinics: primary care (6), specialty (5), behavioral health (2), urgent care (1)  
**Providers:** 240 active scheduling providers; 180 support staff with Cadence access  
**Epic environment:** Hyperspace 2023, Cadence go-live March 2022 (28 months prior)  
**MyChart:** Live, self-scheduling enabled for 3 visit types only  
**External referrals:** Inbound via Kofax fax capture and phone; manually entered into Epic referral orders  
**Current pain:** No scheduling governance, no CAB, no visit type naming convention, no systematic wait-list management

---

## Current State Assessment

### Visit Type Sprawl
- 847 active visit types vs. benchmark ~120 for a system this size
- Clinics created new types ad hoc rather than reusing existing ones: 23 near-identical variants of "Primary Care Follow-Up" across 6 clinics
- Schedulers average 45 extra seconds per call searching for the correct visit type
- No inactivation process: types from providers who left 18+ months ago still active

### Template Fragmentation
- Template utilization at 62%: many providers block or release slots manually, creating admin burden and unpredictable capacity
- No systematic hold pool structure: some clinics hold 30% of slots for same-day; others hold none
- 3rd Next Available averaging 12 days when clinical reality supports 4-day access

### Referral Leakage
- 71% referral capture rate: 29% of external referrals not scheduled (abandoned, entered incorrectly, or directed to competitor)
- No closed-loop notification back to referring providers
- External physicians calling to check referral status: 60-80 calls/week to scheduling center, diverting staff time

### No-Show Rate
- 18% system-wide no-show rate driven by inconsistent reminder cadence
- High-risk patient segments not identified or routed to confirmation workflows
- Overbooking decisions made by schedulers informally, not by policy

---

## Proposed Solution

### Phase 1 (Months 1-3): Visit Type Governance
**Goal:** Reduce 847 active visit types to ~120 standardized types

Steps:
1. Run Crystal report pulling all visit types with appointment counts for past 12 months
2. Identify types with 0 appointments: inactivate after clinical review (no patients mid-stream)
3. Map remaining types to a standard naming convention: `[DEPT]-[TYPE]-[DURATION_MIN]`
   - Example: `PC-NEW-60`, `CARD-FU-30`, `BH-INTAKE-90`, `ORT-POST-OP-20`
4. Merge near-duplicate types within each specialty cluster; update scheduling rules to point to consolidated types
5. Stand up Cadence Change Advisory Board (CAB): bi-weekly meeting; all visit type additions, template changes, and rule group modifications require a change ticket
6. Train super users on read-only vs. edit access boundaries

**Configuration environment:** Build in TST, clinical lead UAT sign-off, promote to PRD in maintenance window.

### Phase 2 (Months 4-6): Template Standardization
**Goal:** Raise template utilization from 62% to 85%

5 base template frameworks by specialty cluster:

| Framework | Slot Duration | Urgent Hold | Same-Day Hold | Care Gap Hold |
|---|---|---|---|---|
| PCP New Patient | 60 min | 2 per half-day | 0 (converts from urgent at 48 hr) | 1 per half-day |
| PCP Follow-Up | 15 min | 1 per 90 min | 1 per 90 min (released at 8 AM day-of) | 1 per 90 min |
| Specialty New | 45 min | 1 per half-day | 0 | 0 |
| Specialty Follow-Up | 20 min | 1 per half-day | 0 | 0 |
| Procedure | Per type | 0 | 0 | 0 |

Individual provider preferences (buffer slots, personal hold configurations) applied as modifiers on top of framework.

Overbook policy: max 1 overbook per half-day session, requires scheduling supervisor approval, logged in Cadence configuration.

### Phase 3 (Months 7-9): Referral Workflow Rebuild
**Goal:** Raise referral capture from 71% to 92%

1. Build HL7 REF^I12 inbound interface via Bridges for top 10 external referral sources
2. Inbound referral triggers entry in Cadence referral workqueue with automated routing by specialty
3. Scheduler contacts patient within 48 hours; appointment created; referral marked complete
4. Outbound SIU^S12 notification sent to referring provider's EHR on scheduling confirmation
5. Crystal report dashboard: open referrals by specialty, WQ age, insurance auth status, referring provider
6. Fax backlog: legacy Kofax referrals scanned and indexed via Epic document management; manually entered with 48-hour SLA

### Phase 4 (Months 10-12): MyChart Expansion and Predictive Tools
**Goal:** Raise self-scheduling from 12% to 35%; cut no-shows from 18% to 8%

1. Expand self-schedulable visit types from 3 to 22 (PCP follow-up, BH follow-up, telehealth, care gap visits)
2. Enable SmartText reminder messaging: 72-hour email + 24-hour SMS, customized by specialty
3. Activate Epic predictive no-show model: flag appointments at >40% risk in day-before confirmation WQ
4. Configure MyChart pre-visit questionnaires for high no-show specialties (reduces visit overhead cost)
5. Automated bump list: nightly batch offers cancelled slots to matching wait list entries

---

## HIPAA and Security

Every scheduling workflow carries PHI. Configuration decisions have compliance implications:

- **Role-based access (RBAC):** Schedulers provisioned with Cadence-only roles. No access to clinical documentation outside the scheduling context. Access reviewed quarterly; deactivated within 24 hours of role change.
- **MPI identity verification:** Patient identity confirmed before scheduling: name, DOB, address, last 4 of phone. MPI duplicate detection alert enabled.
- **PHI in reminders:** Patient communication preferences honored. Voicemail scripts reviewed for minimum necessary PHI. Opt-out removes patient from automated reminder campaigns.
- **HL7 message security:** SIU messages contain PHI. Bridges interfaces configured for TLS 1.2+ only. No routing to cleartext endpoints.
- **Audit logging:** All Cadence access logged. Scheduling rule group changes reviewed weekly. Anomalous access patterns escalated to security officer.
- **Break the Glass:** Used for emergency same-day add-ons outside normal scheduling rules. Each instance documented and reviewed.

---

## Scheduling Templates and Rules

### Visit Type Governance Framework
A visit type in Cadence is more than a name. It controls:
- Appointment duration and slot requirements
- Which scheduling channels can use it (phone, MyChart, walk-in)
- Required questionnaires and pre-visit instructions
- Check-in behavior and copay estimation triggers
- Which scheduling rule groups include it

**Naming convention:** `[DEPT]-[TYPE]-[DURATION_MIN]`

Before rationalization: 23 near-identical "PCP Follow-Up" variants across 6 clinics with inconsistent durations and conflicting rule group memberships.

After rationalization: 1 canonical `PC-FU-15` type with clinic-specific modifiers handled at the scheduling rule level, not the visit type level.

### Scheduling Rule Groups
One rule group per department cluster, governing:
- Which staff roles can schedule which visit types
- Which booking channels (phone, MyChart, walk-in) are permitted
- Lead time constraints (e.g., new patient minimum 2 days advance notice)
- Overbooking thresholds

### Hold Pool Structure
```
PROVIDER TEMPLATE (full day example: PCP Follow-Up)
  08:00  [ PC-FU-15 ]  standard slot
  08:15  [ PC-FU-15 ]  standard slot
  08:30  [ PC-FU-15 ]  standard slot
  08:45  [ URGENT-HOLD ]  -- released to standard at 48hr before date
  09:00  [ PC-FU-15 ]  standard slot
  09:15  [ CARE-GAP ]  -- for proactive outreach campaigns
  ...
```

Hold pool release rules:
- URGENT-HOLD: converts to open slot 48 hours before appointment date if unfilled
- SAME-DAY: visible to schedulers only after 8:00 AM day-of
- CARE-GAP: route to care management workqueue for proactive outreach

---

## HL7 Integration and Referral Management

### SIU Message Flow
When a Cadence appointment is created for a shared patient, Bridges sends an outbound SIU^S12 to the external provider's EHR system within 60 seconds.

| Event | Message Type |
|---|---|
| Appointment booked | SIU^S12 |
| Appointment rescheduled | SIU^S14 |
| Appointment cancelled | SIU^S15 |
| Appointment deleted | SIU^S17 |

```
Cadence creates appointment
  --> Bridges sending application "NRH_CADENCE_OUT"
  --> TCP/IP to external EMR HL7 listener (TLS 1.2)
  --> Acknowledgment (AA) received
  --> Logged in Bridges message log

On failure:
  --> Message routes to Bridges error queue
  --> Analyst reviews daily error queue report
  --> Reprocess or escalate to external system team
```

### Inbound REF^I12 Referral Flow
```
External provider sends referral (EReferral via CommonWell or HL7 REF^I12)
  --> Bridges receiving application "NRH_REFERRAL_IN"
  --> Parse: PID (patient demographics), PRD (provider/referral details), OBR (reason/diagnosis)
  --> MPI match: confirm patient identity in Epic
  --> Auto-route to Cadence referral WQ by specialty
  --> Scheduler picks up WQ entry (SLA: 48 hours)
  --> Patient contacted, appointment created
  --> Referral order closed with appointment linkage
  --> Outbound SIU^S12 to referring provider confirming scheduled date
```

### HL7 Error Handling
- Failed messages routed to Bridges error queue; reviewed daily by Cadence analyst
- Segment parse failures: usually PID-3 (MRN) mismatch or unknown facility code; manual resolution within 24 hours
- Duplicate detection: SIU messages with same control ID rejected; logged for auditing

---

## Monitoring and Observability

### Radar Dashboard Configuration
Configured for scheduling supervisors, showing:
- Real-time WQ depth by specialty and WQ type
- Hold slot fill rate by provider session (live)
- Upcoming template gaps (providers with no template in next 14 days)
- Bridges interface status (all SIU interfaces green/red/yellow)

### Weekly Automated Reports
| Report | Audience | Metrics |
|---|---|---|
| Template Utilization | Dept admins, VP Patient Access | % slots filled, hold pool conversion rates, by provider |
| No-Show by Visit Type | Scheduling directors | No-show rate, cancellation lead time, per specialty |
| 3rd Next Available | VP Patient Access, Clinic managers | 3NA by specialty, trended week-over-week |
| Referral Aging | Referral coordinators, Super users | Open referral WQ entries by age, by specialty, by source |
| WQ Aging | Scheduling supervisors | Scheduling WQ entries over 3 days old |

### Threshold Alerts
- WQ entry age > 3 days: In Basket message to scheduling supervisor
- Hold slot fill rate < 50% at 24 hours before session: flag in Radar for same-day release decision
- Bridges interface error rate > 5% in a day: automated ServiceNow incident ticket (P2)
- Template gap detected (no coverage for a scheduling day): In Basket to clinic administrator, 10 days in advance

---

## Change Management

Every Cadence configuration change follows this path:

1. **Request submitted** via ServiceNow form: clinic name, visit type or template affected, requested change, business reason, preferred go-live date
2. **Analyst impact assessment:** check rule groups, other visit types linked to the template, active WQ entries that might be affected
3. **CAB review** (bi-weekly): approve, deny, or request more info. Changes that affect multiple departments require additional stakeholder sign-off.
4. **Build in TST:** analyst makes the configuration change in the test environment
5. **UAT review:** clinic lead or super user validates in TST; provides written sign-off in ServiceNow ticket
6. **Schedule go-live:** maintenance window or off-peak period, depending on impact
7. **Promote to PRD:** document build steps in SharePoint config tracker before and after
8. **Post-go-live:** ServiceNow notification to affected clinics at go-live; 7-day feedback survey; analyst monitors metrics for unexpected shifts

**Rollback plan:** every change has documented rollback steps in the config tracker. For visit type changes, prior configuration exported to Excel before inactivation. Template rollbacks: prior template grid screenshot saved in SharePoint build log.

**Communication cycle:**
- 5 days before go-live: ServiceNow email to affected scheduling staff
- Day of go-live: Teams message to scheduling team leads
- 7 days after: quick feedback form; analyst reviews and responds within 48 hours

---

## Root Cause Analysis: Cardiology Scheduling Outage

**Incident:** Friday 2:15 PM. Cardiology scheduling team cannot book any new patient appointments. Epic Cadence throws "No valid scheduling rules found" for all Cardiology visit types.

**Timeline:**
- 14:15 — First call to help desk. Ticket opened (P1 — impacts patient care).
- 14:22 — Cadence analyst paged. Log in remotely; open Cadence scheduling rule group activity.
- 14:28 — Confirmed: Cardiology rule group "CARD-PRIMARY-RULES" shows status: Inactive, effective today.
- 14:31 — Review Cadence change log. Rule group last modified 11:47 AM by a super user (no change ticket exists).
- 14:35 — Call super user. She was asked by clinic manager to add a new provider to the rule group. While editing, she accidentally clicked the Status field and changed it from Active to Inactive. She did not notice.
- 14:40 — Analyst reactivates the rule group in TST, confirms no side effects (no other rule groups dependent on it). Promotes to PRD.
- 14:44 — Cardiology confirms scheduling restored. Total downtime: 29 minutes.

**Root causes:**
1. Super users had edit access to scheduling rule groups without a change ticket requirement
2. No automated alert when an active rule group is toggled Inactive while it has future appointments
3. Super user training did not differentiate between read-only review tasks and configuration-edit tasks

**Corrective actions:**
1. Remove scheduling rule group edit access from super user role; restrict to Cadence analyst and above
2. Build Radar alert: if a rule group with active future appointments changes to Inactive, fire an immediate In Basket to the on-call Cadence analyst
3. Update super user training curriculum: clear "view only" vs. "analyst-only" function list; added to onboarding checklist
4. Add rule group status changes to the CAB scope; cannot be changed without a Change Ticket

**Lessons applied:** Cadence configuration access is not tiered enough by default. Every go-live should include a formal access matrix review, not just an Epic-default role assignment.

---

## Disaster Recovery

**Recovery objectives:**
- RPO: 4 hours (Epic snapshot every 4 hours)
- RTO: 2 hours for Epic restore from snapshot (Epic HA baseline)

**Downtime procedures:**
1. Downtime schedule printout generated daily at 7:00 AM and noon (covers next 8 hours)
2. Scheduling staff defer new appointment requests to paper callback log (name, DOB, call-back number, visit type)
3. Urgent appointments sent to triage; clinic leads notified via phone tree

**On restore:**
1. Reconcile paper callback log against Cadence: data entry team enters missed appointments within 4 hours of restore
2. Check for duplicate entries: MPI duplicate detection run report before committing entries
3. Verify provider templates are intact for next 14 scheduling days
4. Bridges interface restart: verify all SIU and REF interfaces are reconnected; replay messages from error queue (max 4 hours of messages)
5. WQ reconciliation: compare WQ counts pre-downtime (from noon printout) vs. post-restore; investigate missing entries
6. Notify stakeholders: ServiceNow incident updated with restore time; scheduling supervisors confirm operations normal

**Annual DR drill:**
Simulate a 4-hour Epic outage in TST environment. Time the full restore and reconciliation process. Verify paper schedule workflow with 3 clinic teams. Document gaps and update runbook.

---

## Automation

- **Automated bump list:** Epic batch job at 6:00 AM scans cancellations from prior day; offers open slots to wait list entries matching visit type and provider criteria; patient receives MyChart notification
- **Reminder campaigns:** Epic reminder batch at 72 hours (email) and 24 hours (SMS or voice) before appointment; patient preference-based; MyChart quick-reschedule link in body
- **WQ aging alert:** Daily Cadence report at 7:00 AM flags WQ entries over 3 days; auto-email to scheduling supervisors
- **Referral aging:** In Basket message to assigned scheduler if referral WQ entry not actioned within 48 hours; escalates to supervisor at 72 hours
- **Visit type usage audit:** Monthly batch report lists visit types with 0 appointments in last 90 days; auto-route to Cadence analyst for inactivation review
- **Template gap detection:** Nightly batch checks all active providers for template coverage in next 14 days; In Basket to clinic administrator for any gap detected
- **No-show prediction:** Epic predictive model scores each next-day appointment for no-show risk; entries above 40% added to day-before confirmation WQ
- **Post-merge cleanup script:** Python script to parse visit type rationalization Excel export and flag remaining duplicates; runs after each Phase 1 batch merge

```python
import openpyxl, re

wb = openpyxl.load_workbook("visit_type_audit.xlsx")
ws = wb.active
seen = {}
duplicates = []

for row in ws.iter_rows(min_row=2, values_only=True):
    name = str(row[1]).strip().upper() if row[1] else ""
    dept = str(row[2]).strip() if row[2] else ""
    appt_count = row[4] if row[4] is not None else 0
    key = f"{dept}|{name}"
    if key in seen:
        duplicates.append((seen[key], row))
    else:
        seen[key] = row

print(f"Potential duplicates found: {len(duplicates)}")
for pair in duplicates:
    print(f"  {pair[0][0]} vs {pair[1][0]}: {pair[0][1]}")
```

---

## Why I Stand Out

1. **End-to-end Cadence ownership:** Not just configuration support. I design visit type governance frameworks, build scheduling rule groups from scratch, and own the full change lifecycle from request to post-go-live measurement.

2. **HL7 integration fluency:** SIU and REF message flows are not theoretical. I have mapped Bridges interfaces, diagnosed segment parse errors, and built closed-loop referral workflows that connect external referring providers back to Epic.

3. **Clinician-first communication:** I translate configuration decisions into plain language for department administrators and front desk staff. That reduces change resistance, speeds adoption, and means fewer tickets for things that should have been trained correctly the first time.

4. **Governance before the crisis:** I instituted CAB processes and change ticketing discipline before an incident forced them. The Cardiology RCA in this proposal shows what happens without governance, and what proper governance prevents.

5. **Metrics-backed decisions:** Every configuration change I propose comes with a baseline metric, a target, and a measurement plan. I do not ask leadership to take my word for it.

---

## About Me and Fit

**Name:** [Your Name]  
**Title:** Healthcare IT Analyst / Epic Cadence Specialist  
**Location:** [Your City or Remote]  
**Contact:** [your.email@domain.com] | [LinkedIn URL]

Healthcare IT professional with a background in Epic scheduling workflows, operational analytics, and cross-functional project delivery in ambulatory care settings. I close the gap between how a clinic wants to operate and how Epic is actually configured to support that.

### Skills
Epic Cadence | Epic MyChart | HL7 v2.x | SIU Messaging | Bridges | Crystal Reports | SlicerDicer | HIPAA | Patient Access Analytics | Business Process Improvement | Change Management | ServiceNow | Project Documentation | Microsoft Office | Cross-functional Training

### Experience to JD Bridge

| Your Experience | JD Requirement |
|---|---|
| 3 years as scheduling operations lead at a 40-provider multi-site primary care group; daily Cadence configuration support and staff training | "Knowledge of the assigned business area's products and processes" |
| Led visit type rationalization project: reduced 312 active types to 89, cutting average scheduling call handle time by 40 seconds | "Business process re-engineering involving broad-based information systems" |
| Configured HL7 SIU interfaces between Epic Cadence and a regional HIE; bi-directional appointment notifications for 8 external facilities | "HL7 knowledge is a plus" |
| Delivered training for 60+ scheduling staff across 3 sites; created quick-reference card used daily by front desk teams | "Strong verbal communication skills while interacting with team members and end users" |
| Documented full Cadence configuration suite for Epic 2023 upgrade; documentation reviewed and approved by Epic TS team | "Strong written communication skills, including project documentation and technical writing" |
| Managed simultaneous template go-lives for 5 new specialties; coordinated Epic TS, clinic leads, and IT infrastructure teams | "Organizing, planning, and executing projects from vision through implementation" |
| Built Cadence reporting dashboard in Crystal Reports tracking 8 KPIs weekly for VP of Patient Access; influenced staffing model for 2 scheduling centers | "Ability to link IT to redesigned business process" |

---

*This is the markdown source for index.html. See that file for the published version.*
