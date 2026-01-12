# Discovery: Delegated Health Coverage & Claims Assurance  
*Problem-driven • Instance-grounded • Pattern-level • Prosocial*

---

## 1. Context (Anonymized)

A household transitioned to continuation health coverage (COBRA) following a job change.

Over approximately mid-2025 through early-2026, the household interacted with:
- A former employer (benefits sponsor)
- A COBRA administrator
- One health insurer in Year 1 and a different insurer in Year 2
- Multiple healthcare providers, including an emergency hospitalization
- Provider billing departments and insurer claims/EOB portals

COBRA premiums were paid on time each month when payment paths were available.

---

## 2. Core Tension

Individuals delegate health coverage, enrollment, and claims administration to institutions, yet remain fully exposed to the consequences of failure.

Institutions:
- Control status and state transitions  
- Operate across fragmented systems  
- Do not reliably surface errors early  

Individuals:
- Bear care, financial, and emotional risk  
- Discover problems late  
- Must coordinate corrections across institutions  

This creates a **structural asymmetry of power, observability, and responsibility**.

---

## 3. Primary Risk Ordering

1. **Prevent disruption to care**  
2. **Prevent financial loss**  
3. **Reduce stress and uncertainty**

This ordering governs all prioritization.

---

## 4. Observed Problems (from Discovery)

### A. Billing & Reconciliation Failures
- Provider bills cannot be paid because charges cannot be reconciled with insurer EOBs.
- Provider statements aggregate charges, while EOBs contain many service-line items.
- Itemized bills are required but not automatically provided.
- Multiple bills are marked past due while reconciliation is in progress.

### B. Claims Denied Due to Coverage State
- Multiple claims denied with reasons indicating coverage was terminated.
- Insurer indicates “manual review” is required.
- Resolution is slow and requires active monitoring.

### C. COBRA Payment Path Failures
- At times, no reliable way exists to pay for upcoming COBRA coverage periods.
- Payment path failures create risk of future coverage lapse.

### D. Claims Visibility & Submission Gaps
- Providers report claims rejected or not accepted.
- Corresponding claims are not visible in insurer portals.
- Individuals are asked to relay filing instructions to providers.
- Final status of some claims is unknown.

### E. Coverage & Dependent State Drift
- Coverage initially inactive due to incomplete enrollment.
- Dependents dropped from coverage multiple times across consecutive days.
- Coverage restored after multi-party coordination, but recurrence occurred.

### F. Data Quality Errors
- Incorrect dependent identity data (e.g., name mismatch) discovered in enrollment records.
- Such errors risk claim rejection and eligibility confusion.

### G. Coding-Driven Coverage Variance
- Identical services covered on one date and not another due to coding differences.
- Resolution requires provider resubmission or insurer review.
- Non-obvious to patients.

### H. Reimbursement Workflow Ambiguity
- Self-paid services require reimbursement submission.
- Individuals must determine what constitutes an acceptable “superbill” or itemized receipt.
- Confirmation of successful submission is unclear.

### I. Low-Signal Anomalies
- EOBs occasionally contain unexpected service categories.
- Impact unclear without investigation.

---

## 5. Current State of Problems

### Unresolved / Active
- Provider bills pending reconciliation (past due)
- Claims denied due to coverage termination
- Inability to pay upcoming COBRA coverage
- Missing or incomplete claim submissions (final status unknown)

### Resolved (but recurring risk)
- Dependents dropped from coverage
- Initial COBRA enrollment incomplete
- Identity/data errors in enrollment
- Coding-driven coverage variance
- Reimbursement workflow confusion

---

## 6. Detection & Timing Requirements

- Detection of discrepancies affecting care or claims must occur within **days**, not weeks.
- Discovery after harm (denials, past-due bills, collections risk) is unacceptable.

---

## 7. Trust & Evidence Model

- No single institutional source is fully authoritative.
- Confidence requires **cross-checking signals**, not blind trust.
- Evidence must be:
  - Inspectable
  - Portable
  - Sufficient to support correction without reconstructing history from memory

---

## 8. Desired End State (Discovery-Level)

If this problem were solved:
- The household would not need to proactively “check” coverage or claims.
- Problems would surface early, with context and evidence.
- Resolution would be tracked to closure.
- Silence would be meaningful.
- Confidence would be justified, not assumed.

---

## 9. Pattern-Level Insight

This instance reflects a broader class of problems:

> **Delegated institutional systems where individuals bear consequences but lack continuous observability.**

Examples include:
- Health insurance eligibility and claims
- COBRA administration
- School enrollment
- Benefits administration
- Licensing and certification

This discovery intentionally operates at a level suitable for reuse.

---

## 10. Discovery Status

Discovery is considered **complete**.

Open questions are explicitly acknowledged rather than assumed resolved, and are suitable inputs to planning rather than further discovery.

*End of Discovery Document*
