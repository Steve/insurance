# Proposal: Delegated Health Coverage & Claims Assurance  
*Learning-first • Local-first • Prosocial • Problem-driven*

---

## 1. Problem or Tension

Individuals must rely on multiple institutions to administer health coverage, enrollment, and claims correctly, yet bear the full consequences when those systems drift, disagree, or fail silently.

Problems are typically discovered only after harm occurs:
- claims are denied,
- provider bills become past due,
- coverage appears terminated,
- dependents disappear from enrollment,
- or payment paths fail.

The result is preventable financial exposure, administrative burden, and persistent uncertainty.

---

## 2. Decision

> **Decide to run a complete product lifecycle (Discovery → Proposal → Plan → Delivery) using GenAI as a first-class collaborator, by designing and delivering an individual-level health coverage and claims assurance system, built local-first on DXOS, in order to learn how to develop real, prosocial software under meaningful architectural constraints—while producing immediate, practical value for a household facing active administrative risk.**

### Explicit Non-Decisions

This proposal does **not** decide to:
- Build a commercial product or define monetization
- Replace or control institutional systems
- Automate appeals or negotiations
- Guarantee correctness of insurer or provider decisions
- Solve all benefit or plan correctness
- Optimize for scale, speed, or distribution
- Expand beyond the defined problem set in early phases

---

## 3. What “Better” Looks Like

*(Written retrospectively)*

Because of this solution, the household no longer worries whether coverage or claims are in order. Regardless of institutional failures, problems surface early, with evidence, and progress toward resolution is visible. When issues are fixed, the system confirms closure. Confidence is restored without constant vigilance.

---

## 4. Principles & Constraints

- **Learning over optimization**  
  Speed, scale, and polish are secondary to insight and understanding.

- **Rebalance asymmetry**  
  The system must increase individual certainty and observability in environments dominated by institutional control.

- **Observability over authority**  
  Conflicts and uncertainty are surfaced, not hidden or resolved implicitly.

- **Constraint-first progression**  
  Work is sequenced to remove the most constraining problem at any given time, prioritizing resolution of active harm before prevention or optimization. The current constraint is re-evaluated after each increment.

- **Smallest useful increment**  
  Every milestone must deliver immediate, real-world value.

- **Local-first integrity**  
  Data ownership and inspectability are preserved.

- **Accessible by default**  
  The system must be usable by non-technical people.

---

## 5. Non-Negotiables

If any of the following are violated, the proposal is no longer being implemented:

1. Discrepancies affecting care or claims must be detected within days.
2. Conflicting institutional states must always be surfaced.
3. Individual-level coverage must be independently visible.
4. Continuous manual vigilance must not be required in steady state.
5. The system must be usable by non-technical users.
6. The system must not assert correctness it cannot justify.
7. Local-first data ownership must not be compromised.

---

## 6. Scope

### In Scope
- Coverage and dependent state assurance
- Claims denial detection and resolution tracking
- Provider billing and EOB reconciliation readiness
- Evidence capture and timeline construction
- COBRA payment continuity awareness
- Reimbursement and missing-claim workflow visibility

### Out of Scope (Early Phases)
- Automated appeals or corrections
- Legal, financial, or medical advice
- Full plan/benefit optimization
- Broad productization or distribution

---

## 7. Success Criteria (as Questions)

Success is evaluated through enduring questions:

- How does local-first work in practice on DXOS?
- Is DXOS sufficient for early-adopter, real-world use?
- Can end-to-end product work be done effectively with GenAI?
- How often does manual review agree with automated detection?
- Does this measurably reduce vigilance, risk, and uncertainty?

---

## 8. Risks, Assumptions, Open Questions

### Assumptions
- Required information can be accessed or captured.
- DXOS is sufficiently mature for this scope.
- GenAI can assist meaningfully across the lifecycle.

### Open Questions
- Where does GenAI help vs hinder judgment?
- Where do local-first constraints improve or slow progress?
- Which unknowns can only be resolved through delivery?

---

## 9. Sequencing (Problem-Driven)

Sequencing follows a constraint-first progression: unresolved problems that cause active harm are addressed before prevention or optimization.

### Phase 1 — Resolve Active Failures (Unresolved Problems)

1. **Unable to pay provider bills due to unreconcilable charges**  
   (past-due bills from a hospitalization)

2. **Claims denied due to coverage marked terminated**  
   (manual review pending)

3. **Inability to pay upcoming COBRA coverage**  
   (payment path unavailable)

4. **Missing or incomplete claim submissions**  
   (final status unknown)

### Phase 2 — Prevent Recurrence (Resolved Problems)

5. Dependents repeatedly dropped from coverage  
6. COBRA enrollment failures  
7. Identity/data errors in enrollment  
8. Coding-driven coverage variance  
9. Reimbursement workflow ambiguity  
10. Low-signal EOB anomalies  

This ordering prioritizes:
1) resolving open harm,  
2) preventing disruption to care,  
3) preventing financial loss,  
4) reducing stress and uncertainty.

---

## 10. Proposal Status

This proposal is **approved for transition to Plan**.

Discovery is complete, priorities are explicit, unknowns are acknowledged, and sequencing is problem-driven. The Plan should operationalize constraint-first progression, producing value in days rather than weeks.

---

# Appendix A — Problem → Detection Mapping

Each section maps a discovered problem to the steps in the sequence that surface it.

---

## A1. Unable to Pay Provider Bills Due to Unreconcilable Charges

**Related sequence steps**
- Phase 1, Step 1
- Phase 1, Step 2

**One-sentence summary**

> Provider bills cannot be paid because aggregated charges cannot be reconciled with insurer EOBs, creating financial and collections risk despite intent to pay.

**Failure narrative (compressed)**

Provider statements list a small number of aggregated charges while insurer EOBs list many detailed service lines. Without itemized bills, patient responsibility cannot be verified and bills become past due during reconciliation.

**Signal (compressed)**

- A provider bill exists  
- Corresponding EOB has materially more line items  
- Itemized detail required to reconcile is missing

---

## A2. Claims Denied Due to Coverage Marked Terminated

**Related sequence steps**
- Phase 1, Step 2
- Phase 1, Step 3

**One-sentence summary**

> Claims are denied for lack of coverage even though coverage was intended to be active, indicating administrative state drift.

**Failure narrative (compressed)**

Coverage was expected to be active, yet claims were denied with reasons indicating coverage termination. Denials were discovered only after bills accumulated or during manual review.

**Signal (compressed)**

- Claim status indicates denial  
- Denial reason references coverage or eligibility  
- Date of service falls within an expected coverage period

---

## A3. Inability to Pay Upcoming COBRA Coverage

**Related sequence steps**
- Phase 1, Step 3

**One-sentence summary**

> The household cannot reliably pay for an upcoming coverage period, creating imminent risk of coverage lapse.

**Failure narrative (compressed)**

As a new coverage period approaches, no functional payment path exists through the COBRA administrator, requiring multi-party coordination and leaving continuity uncertain.

**Signal (compressed)**

- Upcoming coverage period exists  
- No working payment path is available  
- No confirmation of future coverage continuity

---

## A4. Missing or Incomplete Claim Submissions

**Related sequence steps**
- Phase 1, Step 4

**One-sentence summary**

> Claims may never be adjudicated because submissions are rejected, incomplete, or not visible in insurer systems.

**Failure narrative (compressed)**

Providers report claims as rejected or missing, while insurer portals show no corresponding records. Final status remains unclear.

**Signal (compressed)**

- Service occurred  
- Provider reports rejection or submission  
- No corresponding claim appears, or submission is incomplete

---

## A5. Dependents Dropped from Coverage

**Related sequence steps**
- Phase 2, Step 5

**One-sentence summary**

> Dependents are removed from coverage unexpectedly, sometimes repeatedly, risking denial of care.

**Failure narrative (compressed)**

Dependent coverage was removed, restored, and removed again within a short period, requiring repeated institutional coordination.

**Signal (compressed)**

- Dependent previously covered  
- Dependent no longer appears as covered  
- No intentional enrollment change

---

## A6. COBRA Enrollment Incomplete or Incorrect

**Related sequence steps**
- Phase 2, Step 6
- Phase 2, Step 7

**One-sentence summary**

> Coverage was inactive or incorrect due to enrollment or data errors despite intent to enroll correctly.

**Failure narrative (compressed)**

Enrollment steps were incomplete or identity data incorrect, leaving coverage inactive or misrepresented until manually corrected.

**Signal (compressed)**

- Enrollment intent exists  
- Coverage inactive or incorrect  
- Enrollment or identity data mismatches expectations

---

## A7. Coding-Driven Coverage Variance

**Related sequence steps**
- Phase 2, Step 8

**One-sentence summary**

> Identical services are covered on one date and not another due to coding differences.

**Failure narrative (compressed)**

The same service was fully covered once and denied later due to diagnostic versus preventive coding differences.

**Signal (compressed)**

- Same service across claims  
- Different coverage outcomes  
- Difference corresponds to coding variation

---

## A8. Reimbursement Workflow Ambiguity

**Related sequence steps**
- Phase 2, Step 9

**One-sentence summary**

> Reimbursement for self-paid services is uncertain due to unclear documentation and submission confirmation.

**Failure narrative (compressed)**

Self-paid services require reimbursement submission, but acceptable documentation and confirmation of receipt are unclear.

**Signal (compressed)**

- Service paid out-of-pocket  
- Reimbursement requires documentation  
- Submission requirements or confirmation unclear

---

## A9. Low-Signal EOB Anomalies

**Related sequence steps**
- Phase 2, Step 10

**One-sentence summary**

> EOBs occasionally contain unexpected or questionable service categories.

**Failure narrative (compressed)**

An EOB lists service categories that do not align with known care received, creating uncertainty about correctness.

**Signal (compressed)**

- EOB contains unexpected classification  
- Classification does not align with known care

---

# Appendix B — Open Questions by Problem

This appendix lists unresolved questions that must be answered to understand or close each problem, independent of whether resolution is manual or automated.

---

## B1. Unable to Pay Provider Bills Due to Unreconcilable Charges
- Have itemized bills been issued for all provider statements?
- Which EOB line items map to each aggregated charge?
- Are any charges pending reprocessing or review?
- Has a billing hold been confirmed?

---

## B2. Claims Denied Due to Coverage Marked Terminated
- What coverage state was used at adjudication time?
- Does the denial apply to the full date of service or part?
- What evidence shows coverage should have been active?
- Are additional claims from the same period affected?

---

## B3. Inability to Pay Upcoming COBRA Coverage
- Is the payment failure due to system, enrollment, or admin error?
- Does the administrator recognize enrollment for the upcoming period?
- Has any alternate payment path been offered?
- What evidence exists that coverage will remain active?

---

## B4. Missing or Incomplete Claim Submissions
- Were missing claims ever resubmitted?
- Do insurer records show receipt or adjudication?
- What documentation was requested and provided?
- What evidence reconstructs submission attempts?

---

## B5. Dependents Dropped from Coverage
- What events triggered dependent removal?
- Why did removals recur?
- Do insurer and administrator records now agree?
- What safeguards exist against recurrence?

---

## B6. COBRA Enrollment Incomplete or Incorrect
- Which enrollment steps failed?
- Were confirmations issued, and do they match system state?
- Are other enrollment fields unvalidated?
- How can enrollment completeness be verified proactively?

---

## B7. Coding-Driven Coverage Variance
- Which codes were used in each instance?
- Were differences expected under plan rules?
- Were corrected claims submitted?
- Does this pattern affect other services?

---

## B8. Reimbursement Workflow Ambiguity
- What documentation qualifies for reimbursement?
- Has reimbursement ever been successfully completed?
- How is submission receipt confirmed?
- Are deadlines or limits involved?

---

## B9. Low-Signal EOB Anomalies
- Do anomalies affect coverage or cost sharing?
- Are they benign artifacts or errors?
- Has the pattern recurred?
- What threshold warrants escalation?

---

*End of Proposal*
