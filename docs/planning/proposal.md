# Proposal  
## Delegated Insurance Coverage Assurance  
*Learning-first • Local-first • Prosocial by design*

---

## 1. What problem or tension are we addressing?

Individuals delegate health insurance administration to institutions (employers, COBRA administrators, insurers), yet remain exposed to the consequences of silent failure.

Coverage or eligibility can lapse or diverge across systems without timely notification. These discrepancies are typically discovered only after harm occurs—denied claims, unexpected bills, or disrupted access to care—at which point recovery is slower, more stressful, and more costly.

The current state is **ad-hoc discovery**:
- A bill arrives → investigation reveals a denied claim → support explains coverage was “inactive,” despite payment.
- An email announces dependents are no longer covered → the individual coordinates between insurer, employer, and COBRA administrator → days later the issue is “fixed.”

This matters now because:
- The primary risk is **inability to access care**
- Secondary risks include financial loss and ongoing anxiety
- Doing nothing perpetuates a vigilance tax on individuals and reinforces institutional power asymmetry

---

## 2. What decision are we trying to make?

> **Decide to run a complete product lifecycle (Discovery → Proposal → Plan → Delivery) using GenAI as a first-class collaborator, by designing and delivering an individual-level insurance eligibility assurance system, built local-first on DXOS, in order to learn how to develop real, prosocial software under meaningful architectural constraints.**

This decision commits to:
- Completing the *entire* product lifecycle, not stopping at design or prototypes
- Using GenAI across discovery, synthesis, planning, and delivery
- Anchoring learning in a real, personally meaningful problem
- Accepting local-first architecture as a learning constraint, not an optimization
- Prioritizing confidence, assurance, and accessibility over speed, scale, or polish

### Explicit non-decisions (what this proposal is *not* doing)

- Not deciding commercial intent, pricing, or go-to-market
- Not attempting to fix, override, or automate institutional systems
- Not automating appeals or enrollment changes
- Not solving all insurance problems
- Not addressing plan or benefit correctness in the initial phase
- Not committing to specific UI patterns, notification mechanisms, or automation depth
- Not optimizing for cloud-first architectures or distribution
- Not committing yet to other insurance types, domains, or multi-user scale

If these boundaries are crossed, this proposal is no longer being implemented as approved.

---

## 3. What does “better” look like?

*(Written as if looking back after success)*

Because of this solution, my family and I are no longer worrying about whether our insurance is in order. Regardless of what happens with any of the institutions, we know we will be notified if there is a problem, provided with the evidence we need to push it toward resolution, and informed by our trusted system when it is resolved.

---

## 4. What principles or constraints shape the solution?

- **Learning over optimization**  
  This work is not optimized for speed, size, or distribution; it is optimized for learning.

- **Observability over authority**  
  The system supports justified confidence rather than asserting correctness it cannot prove.

- **Rebalancing asymmetry**  
  The design must actively reduce the power asymmetry between individuals and institutions by increasing individual confidence and observability.

- **Local-first as constraint, not feature**  
  Architectural constraints are embraced as part of the learning goal.

Trade-offs are expected and accepted where they support confidence, transparency, and learning.

---

## 5. What is non-negotiable?

The following are non-negotiable boundaries. If any are violated, the proposal is no longer being implemented:

1. Coverage discrepancies that affect care **must be detected within 1 day**.
2. Conflicting institutional states **must always be surfaced**, never hidden.
3. **Individual-level eligibility** must be visible independently.
4. The system **must not require continuous manual vigilance**.
5. The system **must be usable by non-technical people**.
6. The system **must not assert correctness it cannot justify**.
7. **Local-first data ownership must not be compromised**.

---

## 6. What is in scope — and what is deliberately out of scope?

### In scope
- Eligibility / active coverage assurance
- Early detection of discrepancies
- Evidence production for correction
- Confidence-building through transparency
- Personal and family use, with intentional abstraction for reuse

### Out of scope (for this proposal)
- Automatic correction or appeals
- Claims optimization or reimbursement strategy
- Plan or benefit correctness
- Commercialization and distribution
- Cross-domain generalization beyond learnings

---

## 7. What needs to be true for this to be considered successful?

Success is defined through **enduring questions**, not metrics:

- How does local-first actually work in practice on DXOS?
- Is DXOS sufficient for building a usable system for early adopters like me?
- Can I perform effective end-to-end product work using GenAI, and what do I think about that experience?
- How often does my manual review agree with the system’s automated results?

If these questions can be answered credibly through lived experience and inspection of artifacts, the work is considered worthwhile.

---

## 8. What are the known risks, assumptions, or open questions?

### Assumptions
- Necessary information can be accessed or scraped in an automated way.
- DXOS is mature enough to support meaningful local-first application development.
- I can become sufficiently fluent in GenAI-assisted, end-to-end product development to assess trade-offs.

### Open questions
- Where does GenAI meaningfully accelerate thinking and delivery, and where does it introduce friction or false confidence?
- Where do local-first constraints improve design quality, and where do they slow progress?
- What can only be learned by attempting this end-to-end?

These uncertainties are treated as learning objectives, not risks to eliminate upfront.

---

## Proposal Status

This proposal is complete and approved for transition into **Plan**, with this document serving as the immutable reference for intent, constraints, and learning goals.

*End of Proposal*
