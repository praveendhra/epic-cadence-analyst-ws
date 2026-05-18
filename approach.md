# Approach Notes: Epic Cadence Analyst II — BMCHS

*Private research doc. Do not publish if repo is public.*

---

## 1. Role Breakdown by Section

### POSITION SUMMARY
- Business specialist + technology generalist
- Aligns technology solutions with business strategy
- Informs and advises customers on IS tech: functionality, costs, benefits, implementation
- Acts as technical liaison with vendors
- Must build credibility with BOTH customer management and IT personnel
- Forward-thinking: proactively seeks to apply technology to business processes

### JOB REQUIREMENTS SUMMARY
- **Education:** Associate's (or equivalent) required; Bachelor's preferred
- **Experience:** 2+ years in business systems analysis or business unit experience with significant IT involvement
- **Epic:** Cadence certification required within 6 weeks of last class
- **HL7:** Knowledge is a plus

---

## 2. Hard Requirements

| Requirement | Status |
|---|---|
| 2+ years business systems analysis | Customize with your experience |
| Epic Cadence certification (or ability to obtain within 6 weeks) | Required on hire |
| Microsoft Office proficiency | Standard |
| Cross-functional project planning and execution | Demonstrate in work sample |
| Strong written communication, project documentation | Demonstrated in work sample |
| Healthcare domain experience | Demonstrate HIPAA, scheduling workflows |

---

## 3. Soft Requirements (Culture Signals)

- **"Ownership":** Epic analysts own their application end-to-end (build, validation, training, ongoing ops)
- **"Documentation":** Config decisions documented, training materials maintained, build tracker current
- **"Cross-functional":** Code-switch between clinical staff, department administrators, and IT engineers
- **"Forward-thinking":** Bring ideas to the team before problems force them; position as a strategic partner
- **"Credibility with both management and IT":** Same person translates cost-benefit to a VP and a scheduler in the same day

---

## 4. Domain Specifics: Epic Cadence

### What Cadence Is
Epic Cadence is Epic's scheduling and patient access application. It manages:
- Appointment booking (phone, web, walk-in, MyChart self-scheduling)
- Provider template management (when/where/how long each visit type is available)
- Visit type configuration (duration, rules, questionnaires, check-in type)
- Scheduling rules (who can schedule what, via which channel, with what lead time)
- Wait list and bump list management
- Referral-to-scheduling workflow
- Resource scheduling (rooms, equipment, interpreters)

### Key Cadence Concepts

| Concept | What it is |
|---|---|
| Visit Types (EVT) | Define appointments: duration, slot requirements, instructions, check-in behavior, questionnaires |
| Scheduling Rules / Rule Groups | Control which visit types staff can book, via which channels, with what lead time and constraints |
| Provider Templates | Time grid defining when a provider sees patients; contains slot types, hold pools, and preferences |
| Slot Types | Categories of appointment slots (new patient, follow-up, procedure, urgent, etc.) |
| Hold Types / Hold Pools | Reserve slots for specific purposes: urgent, same-day, care gap, referral |
| Overbook Rules | Define when and how many appointments can exceed template capacity; can require supervisor approval |
| Open Scheduling | Template-independent scheduling for low-complexity visits (e.g. phone consults, telehealth) |
| Wait List Entries (WLE) | Manage patients needing earlier slots; automated or manual bump processing |
| Work Queues (WQ) | Scheduling WQs for unfulfilled/recall appointments; referral routing queues; aging alerts |
| In Basket | Epic's internal messaging system; integrates with scheduling for task routing and referral alerts |
| MyChart Scheduling | Patient self-scheduling portal; visit types must be flagged self-schedulable and meet slot hold rules |
| SmartLinks / SmartText | Used in reminder messaging and scheduling context notes |
| 3rd Next Available (3NA) | Key access metric: how far out is the 3rd open slot for a new patient appointment |

### Reporting Tools
- **Crystal Reports / Reporting Workbench:** Template utilization, no-show rates, scheduling cycle time, visit type usage, 3NA trending
- **Epic Radar Dashboard:** Real-time operational view for scheduling supervisors
- **SlicerDicer:** Self-service reporting on scheduling data for clinic managers
- **Slicer Trending:** Long-period metric analysis for leadership dashboards

---

## 5. Gap Analysis

### Strong Areas
- Healthcare IT business systems analysis and project coordination
- Cross-functional collaboration (clinical staff, IT, administration, vendors)
- Documentation: technical writing, build trackers, training materials
- HL7 v2.x message structure (SIU scheduling messages, ADT flows)
- No-show and access metrics (3NA, cycle time, utilization)

### Areas to Brush Up On
- Epic Cadence certification content: Templates, Visit Types, Rule Groups, WQ setup
- MyChart self-scheduling configuration specifics (patient-facing rules)
- Epic Bridges HL7 engine: sending application setup, segment mapping, error queue management
- Epic's predictive analytics for no-show risk scoring
- HIPAA administrative safeguards specific to scheduling workloads (minimum necessary, audit trails)

---

## 6. Cert Roadmap

### Epic Cadence Certification Path
1. **Instructor-led training** at Epic (Verona, WI) or remote: 1-2 weeks
2. **Hands-on build exercises** in Epic's training environment
3. **Certification exam:** must pass within 6 weeks of last class
4. **Annual proficiency:** maintain after major Epic upgrades

### Related Certifications Worth Having
| Cert | Why |
|---|---|
| Epic MyChart | Self-scheduling configuration is Cadence-adjacent |
| Epic Bridges | HL7 SIU/REF interface builds — JD calls HL7 a plus |
| CPHIMS (HIMSS) | Broad healthcare IT credibility |
| PMP or CAPM | Multi-site rollout project management |
| CHDA | Analytics and reporting angle |

---

## 7. Domain-Specific Considerations: Healthcare

### HIPAA
- **Minimum Necessary Rule:** Schedulers see only the PHI required to complete the scheduling task; no clinical documentation access
- **Audit trails:** All Cadence access logged in Epic; role-based access reviewed quarterly; deactivate accounts within 24 hours of role change
- **Break the Glass:** Emergency schedule overrides: every instance requires documentation and is reviewed by the security officer
- **Patient identity verification:** MPI matching before scheduling to prevent duplicate MRNs; name, DOB, address, last 4 of phone
- **Appointment reminders:** Patient communication preferences honored (opt-out respected); voicemail scripts reviewed for minimum necessary PHI; text messages follow TCPA
- **PHI in HL7 messages:** SIU messages contain PII/PHI; Bridges interfaces require TLS 1.2+; no routing to cleartext endpoints

### HL7 Specifics for Scheduling
| Message | Trigger |
|---|---|
| SIU^S12 | New appointment booked |
| SIU^S14 | Appointment modified |
| SIU^S15 | Appointment cancelled |
| SIU^S17 | Appointment deleted |
| REF^I12 | Patient referral inbound from external provider |
| ADT^A08 | Patient information update (can trigger downstream scheduling actions) |

### Compliance and Regulatory
- **CMS Conditions of Participation:** Same-day access standards for primary care under certain federal programs
- **Joint Commission:** Patient access standards tied to accreditation surveys
- **PCMH:** Advanced access and open scheduling requirements for recognized practices
- **Prior authorization:** Scheduling must align with auth requirements before procedures go on book

---

## 8. KPIs and Metrics for Work Sample

| Metric | Baseline | Target | Industry Benchmark |
|---|---|---|---|
| No-show rate | 18% | 8% | 5-10% |
| Template utilization | 62% | 85% | 80-90% |
| Avg scheduling cycle time | 4.2 days | 1.8 days | 1-3 days |
| Referral capture rate | 71% | 92% | 85-95% |
| 3rd Next Available (3NA) | 12 days | 4 days | 3-7 days |
| Scheduling WQ avg age | 9.3 days | 2 days | under 3 days |
| Patient self-scheduling rate | 12% | 35% | 25-40% |
| Appointment reminder opt-in | 67% | 90% | 80-95% |

---

## 9. Tooling Stack (Employer Environment Inference)

| Tool | Purpose |
|---|---|
| Epic Cadence (Hyperspace) | Core scheduling application |
| Epic MyChart | Patient self-scheduling portal |
| Epic Bridges | HL7 interface engine (SIU, REF, ADT flows) |
| Epic Crystal Reports / Workbench | Scheduling analytics and operational reports |
| Epic SlicerDicer | Self-service ad hoc reporting for clinic managers |
| Epic In Basket | Internal task routing and referral notification |
| Epic Radar | Real-time scheduling dashboard |
| ServiceNow | ITSM: change requests, incident tracking, CAB workflow |
| Microsoft Teams / SharePoint | Project documentation, super user communication |
| Microsoft Excel / Access | Config tracking spreadsheets, visit type audit tables |
| Visio / Lucidchart | Process flow documentation |
| Zoom / WebEx | Remote training delivery |

---

## 10. Top 3 Pain Points (Inferred from JD + Domain Knowledge)

1. **Visit type and template sprawl:** No governance means hundreds of near-duplicate visit types; schedulers waste time finding the right one; provider workload imbalances because templates were built inconsistently by clinic

2. **Referral leakage:** External referrals arriving by fax or phone, not captured in Epic; patients lost to competitor systems; no closed-loop tracking back to the referring provider

3. **Super user network operating as silos:** Each clinic makes local Cadence decisions without standards; the analyst spends time firefighting instead of improving; incidents happen because super users have edit access they should not have

---

*For personal use only. Combine with resume facts before finalizing the About Me section of index.html.*
