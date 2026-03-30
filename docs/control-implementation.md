# Control Implementation Guide

**System:** Customer Onboarding & Fraud Risk Scoring AI  
**Classification:** 🔴 HIGH RISK — EU AI Act Annex III §5(b)  
**Document Owner:** AI Governance Committee  
**Date:** March 2026  

---

## Purpose

This document provides detailed implementation specifications for each mandatory control imposed by the AI Governance Committee. These controls are conditions of deployment approval — the system cannot go live until all controls are implemented and verified.

---

## Control 1: Mandatory Human-in-the-Loop

### Objective

Ensure that no AI output becomes a binding decision without meaningful human review and approval.

### Implementation Requirements

| Requirement | Specification |
|---|---|
| **UI labelling** | All AI outputs must be labelled as "AI Recommendation" — never "Decision" |
| **Approval workflow** | System must require human approval action before any recommendation is actioned |
| **Rejection blocking** | System must prevent rejection actions without analyst sign-off |
| **Audit trail** | Every decision must capture the human approver's identity and timestamp |

### Technical Specification

```
┌─────────────────────────────────────────────────────────────┐
│                    AI Scoring Engine                         │
│                                                              │
│  Input: Application Data                                     │
│  Output: Risk Score + Recommendation                         │
│  Status: ADVISORY ONLY                                       │
└─────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────┐
│                    Human Review Queue                        │
│                                                              │
│  Reviewer sees:                                              │
│  - Application summary                                       │
│  - AI risk score (0-100)                                     │
│  - AI recommendation (Approve/EDD/Reject)                    │
│  - Top 5 contributing factors                                │
│  - Confidence level                                          │
│                                                              │
│  Reviewer must:                                              │
│  - Select final decision (Approve/EDD/Reject)                │
│  - Enter rationale (mandatory free text)                     │
│  - Acknowledge accountability                                │
│                                                              │
│  System blocks:                                              │
│  - Any action without rationale                              │
│  - Any action without explicit selection                     │
│  - "Auto-approve" or batch approval                          │
└─────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────┐
│                    Decision Record                           │
│                                                              │
│  Captured:                                                   │
│  - Application ID                                            │
│  - AI recommendation                                         │
│  - Human decision                                            │
│  - Override flag (if different)                              │
│  - Rationale text                                            │
│  - Reviewer ID                                               │
│  - Timestamp                                                 │
│  - Session metadata                                          │
└─────────────────────────────────────────────────────────────┘
```

### Acceptance Criteria

- [ ] No path exists in the system to action a decision without human approval
- [ ] UI clearly distinguishes AI recommendations from human decisions
- [ ] Audit trail captures all required fields for every decision
- [ ] Batch approval functionality is disabled at the system level

### EU AI Act Alignment

This control satisfies **Article 14** — Human oversight requirements for High-Risk AI systems.

---

## Control 2: Tiered Decision Model

### Objective

Route AI recommendations to appropriate human review tiers based on risk level, ensuring proportionate scrutiny.

### Implementation Requirements

| Risk Tier | Score Range | Reviewer Level | SLA | Escalation Path |
|---|---|---|---|---|
| 🟢 Low Risk | 0–19 | Junior Analyst | 24 hours | Senior Analyst if uncertain |
| 🟡 Medium Risk | 20–70 | Analyst | 48 hours | Senior Analyst if override |
| 🔴 High Risk | 71–100 | Senior Analyst | 72 hours | Team Lead if exception |

### Routing Logic

```
IF risk_score < 20:
    route_to: junior_analyst_queue
    sla: 24_hours
    escalation: senior_analyst
    
ELIF risk_score >= 20 AND risk_score <= 70:
    route_to: analyst_queue
    sla: 48_hours
    escalation: senior_analyst
    
ELIF risk_score > 70:
    route_to: senior_analyst_queue
    sla: 72_hours
    escalation: team_lead
```

### SLA Monitoring

| Metric | Threshold | Action |
|---|---|---|
| Cases approaching SLA | 80% of SLA elapsed | Amber alert to reviewer |
| Cases breaching SLA | 100% of SLA elapsed | Red alert to team lead |
| Chronic SLA breaches | >5% breach rate per week | Capacity review triggered |

### Acceptance Criteria

- [ ] Routing logic correctly assigns cases based on risk score
- [ ] SLA tracking is active for all cases
- [ ] Escalation paths are functional
- [ ] Reporting dashboard shows tier distribution and SLA compliance

---

## Control 3: Transparency & Evidence Logging

### Objective

Capture comprehensive evidence for every AI recommendation and human decision to support audit, complaint handling, and regulatory inquiry.

### Data Elements

| Category | Data Element | Retention |
|---|---|---|
| **Input Data** | Application ID | 7 years |
| | Customer identifier (hashed) | 7 years |
| | Input features used for scoring | 7 years |
| | Data source timestamps | 7 years |
| **AI Output** | Risk score | 7 years |
| | Recommendation (Approve/EDD/Reject) | 7 years |
| | Confidence level | 7 years |
| | Top contributing factors | 7 years |
| | Model version | 7 years |
| **Human Decision** | Reviewer ID | 7 years |
| | Final decision | 7 years |
| | Override flag | 7 years |
| | Rationale text | 7 years |
| | Decision timestamp | 7 years |
| **Outcome** | Final onboarding status | 7 years |
| | Subsequent fraud events (if any) | 7 years |

### Log Architecture

```
┌─────────────────────────────────────────────────────────────┐
│                    Decision Log Entry                        │
├─────────────────────────────────────────────────────────────┤
│  entry_id: uuid                                              │
│  timestamp: ISO8601                                          │
│  application_id: string                                      │
│  customer_id_hash: sha256                                    │
│                                                              │
│  ai_output:                                                  │
│    risk_score: int (0-100)                                   │
│    recommendation: enum [APPROVE, EDD, REJECT]               │
│    confidence: float (0.0-1.0)                               │
│    model_version: string                                     │
│    contributing_factors: array[5]                            │
│      - factor_name: string                                   │
│      - factor_value: string                                  │
│      - factor_weight: float                                  │
│                                                              │
│  human_decision:                                             │
│    reviewer_id: string                                       │
│    reviewer_tier: enum [JUNIOR, ANALYST, SENIOR]             │
│    final_decision: enum [APPROVE, EDD, REJECT]               │
│    override: boolean                                         │
│    rationale: string (min 10 chars)                          │
│    decision_timestamp: ISO8601                               │
│    time_to_decision_seconds: int                             │
│                                                              │
│  metadata:                                                   │
│    session_id: uuid                                          │
│    ip_address: string (masked)                               │
│    user_agent: string                                        │
└─────────────────────────────────────────────────────────────┘
```

### Access Controls

| Role | Access Level |
|---|---|
| Reviewers | Own decisions only |
| Team Leads | Team decisions |
| Audit | Full read access |
| Compliance | Full read access |
| IT Operations | Metadata only (no decision content) |

### Acceptance Criteria

- [ ] All required data elements captured for every decision
- [ ] Logs are immutable (append-only)
- [ ] 7-year retention implemented and tested
- [ ] Access controls enforced per role matrix
- [ ] Log retrieval tested for audit scenarios

### EU AI Act Alignment

This control satisfies **Article 12** — Automatic logging requirements for High-Risk AI systems.

---

## Control 4: Acceptable Use Policy Enforcement

### Objective

Ensure all staff interacting with the AI system understand their responsibilities and the system's limitations.

### Policy Requirements

All users must acknowledge the following before accessing the system:

| Requirement | Acknowledgement |
|---|---|
| AI outputs are advisory only | ☐ I understand |
| I am accountable for final decisions | ☐ I understand |
| I must document my rationale | ☐ I understand |
| I should override AI when my judgment disagrees | ☐ I understand |
| I will report suspected AI errors | ☐ I understand |

### Enforcement Mechanisms

| Mechanism | Implementation |
|---|---|
| **Onboarding** | Policy acknowledgement required before system access granted |
| **Annual renewal** | Re-acknowledgement required annually |
| **Audit trail** | Acknowledgement records retained |
| **Violation response** | Policy violations escalated to line management |

### Override Encouragement

The policy explicitly encourages overrides when professional judgment disagrees with AI:

> *"You are expected to exercise independent professional judgment. If you disagree with an AI recommendation, override it and document your reasoning. Overrides are not penalised — they are a sign of meaningful human oversight."*

### Override Rate Monitoring

| Metric | Expected Range | Action if Outside Range |
|---|---|---|
| Overall override rate | 10–25% | Review if <5% or >35% |
| Override rate by reviewer | Within 2σ of mean | Coaching if outlier |
| Override rate by risk tier | Higher for medium risk | Review if pattern inverted |

### Acceptance Criteria

- [ ] Policy published and accessible to all users
- [ ] Acknowledgement workflow implemented
- [ ] Acknowledgement records captured and retained
- [ ] Override rate monitoring dashboard active

---

## Control 5: Ongoing Monitoring & Re-Assessment

### Objective

Detect model drift, bias emergence, and control effectiveness degradation through continuous monitoring.

### Monitoring Schedule

| Activity | Frequency | Owner | Output |
|---|---|---|---|
| **Fairness testing** | Quarterly | Data Science | Fairness report |
| **Drift detection** | Monthly | Data Science | Drift dashboard |
| **Override rate analysis** | Monthly | Operations | Override report |
| **SLA compliance** | Weekly | Operations | SLA dashboard |
| **Incident review** | As needed | Risk | Incident report |
| **Full governance review** | Annually | AI Governance Committee | Re-assessment |

### Fairness Testing Specification

Fairness testing evaluates outcome disparities across protected characteristic proxies:

| Metric | Definition | Threshold |
|---|---|---|
| **Demographic parity** | Approval rate ratio between groups | >0.8 (80% rule) |
| **Equalised odds** | True positive rate ratio between groups | >0.8 |
| **Predictive parity** | Precision ratio between groups | >0.8 |

Protected characteristic proxies tested:
- Age bands (derived from date of birth)
- Postcode-based demographic indices
- Name-based ethnicity estimates (where legally permitted)

### Drift Detection Specification

| Drift Type | Detection Method | Threshold |
|---|---|---|
| **Input drift** | Distribution shift in input features | KL divergence >0.1 |
| **Output drift** | Change in score distribution | Mean shift >5 points |
| **Concept drift** | Change in feature-outcome relationships | Accuracy drop >3% |

### Mandatory Re-Assessment Triggers

The system must undergo full governance re-assessment if any of the following occur:

| Trigger | Rationale |
|---|---|
| Model architecture change | New model may have different risk profile |
| Training data source change | New data may introduce new biases |
| Automation level change | Reduced human oversight requires re-approval |
| Regulatory guidance update | Obligations may have changed |
| Fairness threshold breach | Bias has emerged and must be addressed |
| Material incident | Root cause may require control changes |

### Acceptance Criteria

- [ ] All monitoring dashboards operational
- [ ] Alert thresholds configured
- [ ] Escalation paths tested
- [ ] Quarterly fairness testing scheduled
- [ ] Re-assessment triggers documented and understood

### EU AI Act Alignment

This control satisfies **Article 9** — Risk management system requirements, including post-market monitoring.

---

## Implementation Verification

Before deployment, the following verification must be completed:

| Control | Verified By | Sign-Off |
|---|---|---|
| Control 1: Human-in-the-Loop | QA + Compliance | ☐ |
| Control 2: Tiered Decision Model | QA + Operations | ☐ |
| Control 3: Transparency & Evidence | QA + Audit | ☐ |
| Control 4: Acceptable Use Policy | Compliance + HR | ☐ |
| Control 5: Ongoing Monitoring | Data Science + Risk | ☐ |

**Final deployment approval:** AI Governance Committee

---

*← Back to [Main Project](../README.md)*
