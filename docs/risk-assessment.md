# Responsible AI Risk Assessment

**System:** Customer Onboarding & Fraud Risk Scoring AI  
**Classification:** 🔴 HIGH RISK — EU AI Act Annex III §5(b)  
**Analyst:** Francisco Fonseca  
**Date:** March 2026  

---

## Assessment Methodology

This risk assessment follows a structured methodology aligned with the EU AI Act requirements and industry-standard Responsible AI frameworks. The assessment evaluates risk across six dimensions, each rated on a three-tier scale:

| Rating | Definition |
|---|---|
| 🟢 **Low** | Risk is minimal; standard controls sufficient |
| 🟡 **Medium** | Risk is material; enhanced controls required |
| 🔴 **High** | Risk is significant; mandatory controls and governance oversight required |

The overall classification follows the highest individual dimension rating — a single High-rated dimension triggers High-Risk classification for the entire system.

---

## Dimension 1: Business Impact

**Rating:** 🔴 High

### Assessment

The AI system directly affects individuals' access to financial services. A rejection recommendation, if actioned, denies a person the ability to open an account, access credit, or use payment services. This is not a convenience impact — it affects economic participation.

| Impact Factor | Assessment |
|---|---|
| Decision type | Access to financial services |
| Affected population | Natural persons (UK & EU residents) |
| Reversibility | Decisions can be appealed, but initial impact is immediate |
| Scale | Thousands of applications processed monthly |

### Regulatory Significance

Under the EU AI Act, AI systems used to evaluate creditworthiness or establish credit scores of natural persons are explicitly classified as High-Risk (Annex III §5(b)). This is not discretionary — it is a legal classification that triggers mandatory compliance obligations.

### Conclusion

Business impact alone would classify this system as High-Risk. The remaining dimensions determine the specific controls required.

---

## Dimension 2: Data & Privacy

**Rating:** 🔴 High

### Assessment

The system processes sensitive personal and financial data across multiple categories:

| Data Category | Examples | Sensitivity |
|---|---|---|
| Identity data | Name, date of birth, address, ID documents | Personal data |
| Financial data | Income, employment, existing accounts | Sensitive financial |
| Behavioural data | Device fingerprint, session patterns, geolocation | Potentially sensitive |
| Fraud indicators | Historical fraud patterns, mule account signals | Sensitive operational |

### Special Category Data Risk

While the system does not explicitly request special category data (race, ethnicity, religion, etc.), there is a risk that proxy indicators could encode protected characteristics:

- Postcode data may correlate with demographic composition
- Name patterns may correlate with ethnicity
- Device and behavioural data may correlate with age or disability

These proxy correlations must be monitored and mitigated through the fairness testing controls.

### GDPR Considerations

- **Lawful basis:** Legitimate interest (fraud prevention) and contract performance (onboarding)
- **DPIA required:** Yes — automated decision-making affecting individuals
- **Data minimisation:** Only data necessary for scoring should be processed
- **Retention:** 7-year retention for regulatory compliance

### Conclusion

Data sensitivity is High. Privacy-by-design principles must be embedded, and a DPIA must be completed before deployment.

---

## Dimension 3: Fairness

**Rating:** 🔴 High

### Assessment

The AI system presents significant fairness risks due to the nature of its training data and decision impact:

| Fairness Risk | Description |
|---|---|
| **Historical bias** | Training data reflects past decisions, which may encode historical discrimination |
| **Proxy discrimination** | Neutral-seeming features may correlate with protected characteristics |
| **Feedback loops** | Rejected applicants cannot demonstrate creditworthiness, reinforcing initial patterns |
| **Disparate impact** | Even without discriminatory intent, outcomes may disproportionately affect protected groups |

### Protected Characteristics at Risk

Under UK Equality Act 2010 and EU non-discrimination law, the following characteristics require monitoring:

- Age
- Race and ethnicity
- Sex and gender
- Disability
- Religion or belief
- Sexual orientation
- Pregnancy and maternity

### Required Fairness Controls

| Control | Description |
|---|---|
| Pre-deployment bias audit | Test for disparate impact before go-live |
| Ongoing demographic monitoring | Track approval/rejection rates by protected characteristic proxies |
| Fairness metric thresholds | Define acceptable variance (e.g., 80% rule for adverse impact) |
| Bias incident response | Procedure for responding to detected fairness violations |

### Conclusion

Fairness risk is High. Quarterly fairness testing and continuous demographic monitoring are mandatory controls.

---

## Dimension 4: Accountability

**Rating:** 🟡 Medium

### Assessment

Accountability structures have been defined, reducing this dimension from High to Medium:

| Accountability Element | Status |
|---|---|
| System owner identified | ✅ Head of Digital |
| Escalation path defined | ✅ To AI Governance Committee |
| Incident response owner | ✅ Head of Operational Risk |
| Regulatory liaison | ✅ Head of Compliance |
| Decision liability | ✅ Remains with human approver |

### Accountability Chain

```
AI Recommendation → Human Reviewer → Decision Logged → Outcome Attributed to Human
```

The human-in-the-loop control ensures that no AI output becomes a binding decision without human accountability. This is critical for both regulatory compliance and incident response.

### Gaps Identified

- **Vendor accountability:** If third-party models are used, vendor liability must be contractually defined
- **Model developer accountability:** Data science team accountability for model quality must be formalised

### Conclusion

Accountability is Medium. Structures are in place, but vendor and developer accountability should be formalised before deployment.

---

## Dimension 5: Transparency

**Rating:** 🔴 High

### Assessment

The AI system presents significant transparency challenges:

| Transparency Element | Assessment |
|---|---|
| **Model explainability** | Risk scores are not inherently explainable to affected individuals |
| **Decision rationale** | Applicants cannot understand why they were flagged or rejected |
| **Regulatory disclosure** | EU AI Act requires transparency about AI use in high-risk decisions |
| **Internal transparency** | Reviewers need to understand AI reasoning to exercise meaningful oversight |

### EU AI Act Transparency Requirements

Under Articles 13–14, deployers of High-Risk AI systems must:

1. Inform affected individuals that AI is being used in their assessment
2. Provide meaningful information about the logic involved
3. Enable human oversight that can understand and override AI outputs

### Required Transparency Controls

| Control | Description |
|---|---|
| Customer disclosure | Clear notification that AI assists in application assessment |
| Explainability layer | SHAP or similar explanations generated for each decision |
| Reviewer dashboard | Human reviewers see top contributing factors to AI score |
| Appeals information | Customers informed of right to request human review |

### Conclusion

Transparency risk is High. Explainability controls and customer disclosures are mandatory before deployment.

---

## Dimension 6: Safety & Misuse

**Rating:** 🔴 High

### Assessment

The AI system presents safety risks related to over-reliance and automation bias:

| Safety Risk | Description |
|---|---|
| **Automation bias** | Reviewers may defer to AI recommendations without critical assessment |
| **Over-reliance** | As trust builds, oversight may become nominal rather than meaningful |
| **Adversarial manipulation** | Fraudsters may probe the system to identify evasion patterns |
| **Scope creep** | System may be extended to higher-stakes decisions without re-assessment |

### Automation Bias Mitigation

Research consistently shows that humans tend to over-rely on automated recommendations, particularly under time pressure. The tiered review model and override tracking controls are designed to counteract this:

- Override rates below 5% trigger review — may indicate rubber-stamping
- Override rates above 30% trigger review — may indicate model degradation
- Rationale documentation requirement forces active engagement

### Adversarial Robustness

Fraud detection systems are inherently adversarial — bad actors actively probe for weaknesses. Controls required:

| Control | Description |
|---|---|
| Input validation | Detect anomalous input patterns that may indicate probing |
| Model rotation | Periodic model updates to prevent pattern learning |
| Red team testing | Simulated adversarial attacks before deployment |

### Conclusion

Safety risk is High. Automation bias controls and adversarial robustness measures are mandatory.

---

## Overall Risk Classification

### Dimension Summary

| Dimension | Rating |
|---|---|
| Business Impact | 🔴 High |
| Data & Privacy | 🔴 High |
| Fairness | 🔴 High |
| Accountability | 🟡 Medium |
| Transparency | 🔴 High |
| Safety & Misuse | 🔴 High |

### Classification Decision

**Overall Classification: 🔴 HIGH RISK**

The system is classified as High-Risk based on:

1. **Legal classification:** EU AI Act Annex III §5(b) — creditworthiness evaluation
2. **Impact severity:** Five of six risk dimensions rated High
3. **Population affected:** Natural persons seeking financial services
4. **Decision type:** Access to essential financial services

### Mandatory Compliance Requirements

Under EU AI Act Articles 9–15, the following are mandatory:

| Article | Requirement |
|---|---|
| Article 9 | Establish and maintain a risk management system |
| Article 10 | Ensure data governance and bias mitigation |
| Article 11 | Produce and maintain technical documentation |
| Article 12 | Implement automatic logging of system operations |
| Article 13 | Ensure transparency to deployers and affected persons |
| Article 14 | Enable meaningful human oversight |
| Article 15 | Ensure accuracy, robustness, and cybersecurity |

---

## Recommended Controls

Based on this assessment, the following controls are recommended for Governance Committee approval:

| Control | Risk Addressed | Priority |
|---|---|---|
| Human-in-the-loop (mandatory) | Safety, Accountability | 🔴 Critical |
| Tiered decision model | Safety, Transparency | 🔴 Critical |
| Transparency & evidence logging | Transparency, Accountability | 🔴 Critical |
| Acceptable use policy | Safety, Accountability | 🟡 High |
| Quarterly fairness testing | Fairness | 🔴 Critical |
| Ongoing drift monitoring | Safety, Fairness | 🟡 High |

---

## Assessor Certification

This assessment was conducted in accordance with the organisation's AI Governance Policy and EU AI Act requirements. The findings represent my professional judgment based on the information available at the time of assessment.

**Assessor:** Francisco Fonseca  
**Date:** March 2026  
**Next Review:** March 2027 (or upon material change)

---

*← Back to [Main Project](../README.md)*
