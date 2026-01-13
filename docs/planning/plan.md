# Plan
## Delegated Health Coverage & Claims Assurance System
*Problem-driven, constraint-first implementation to resolve active household harm*

**Proposal reference**: [docs/planning/proposal.md](./proposal.md)

---

## The Brief

This plan implements a local-first health coverage and claims assurance system to resolve active administrative failures causing financial harm, care disruption, and persistent uncertainty for a household.

**Context**: The household is currently unable to:
1. Pay provider bills due to unreconcilable charges (past-due bills from hospitalization)
2. Resolve claims denied for "terminated coverage" despite active coverage intent
3. Pay upcoming COBRA coverage due to unavailable payment path
4. Determine status of missing or incomplete claim submissions

These are not theoretical problems—they represent **active harm** requiring immediate resolution. Secondary problems (dependents dropped, enrollment errors, coding variance, reimbursement ambiguity, EOB anomalies) will be addressed after active harm is resolved.

**This is a learning-first project** with three goals:
1. Deliver immediate, practical value (resolve active problems in days)
2. Learn GenAI as first-class collaborator across full product lifecycle
3. Learn local-first architecture (DXOS) through real-world constraints

**Implementation philosophy**: **Constraint-first progression**. Work is sequenced to remove the most constraining problem at any given time. Each increment must deliver immediate, real-world value. The current constraint is re-evaluated after each delivery.

**Important non-negotiables being respected** (from Proposal section 5):
1. Discrepancies affecting care or claims must be detected within days (not months)
2. Conflicting institutional states must always be surfaced (never hidden)
3. Individual-level coverage must be independently visible
4. Continuous manual vigilance must not be required in steady state
5. The system must be usable by non-technical users
6. The system must not assert correctness it cannot justify (show evidence, not conclusions)
7. Local-first data ownership must not be compromised (DXOS architecture)

**Scope**: Phase 1 addresses 4 active problems causing current harm. Phase 2 addresses 6 prevention problems to avoid recurrence. See Proposal Appendix A for detailed problem narratives and signals.

---

## Constraints

These implementation guardrails must be referenced during review and survive conversion into work items:

1. **Constraint-first sequencing**: Work must address the most constraining active problem first. Unresolved problems causing harm take priority over prevention or optimization. Re-evaluate constraint ordering after each increment.

2. **Smallest useful increment**: Every delivery must provide immediate, real-world value—even if imperfect. A manual workflow that resolves an active problem is more valuable than an automated system that addresses a future problem.

3. **Days, not weeks**: Phase 1 problems must see measurable progress within days of starting work. If an increment takes >5 days, it's too large.

4. **Transparency over certainty**: Conflicting data must be surfaced to the user with evidence. The system shows "Insurer says X, Provider says Y" rather than asserting which is correct.

5. **Justified confidence only**: The system must never claim correctness it cannot support with captured evidence. Uncertainty is explicit.

6. **Local-first integrity**: All data (coverage snapshots, claims, EOBs, bills, COBRA records) stored locally in DXOS. No cloud dependencies for core data.

7. **Non-technical accessibility**: The system must be usable by household members without technical expertise. Plain language, clear actions, minimal abstractions.

8. **Detection within days**: Coverage or claims discrepancies affecting care must be detected within days (manual capture acceptable initially; automation only after value proven).

9. **Learning over optimization**: When choosing between delivery speed and learning depth, prefer learning (DXOS patterns, GenAI collaboration, problem understanding) as long as it doesn't block resolving active harm.

10. **Problem-driven validation**: Success is measured by whether active problems are resolved, not by system completeness. If provider bills can be paid, claims get reprocessed, COBRA payment works, and missing claims are found—the system succeeded, even if incomplete.

---

## Unknowns requiring decision

The following decisions must be made before or during delivery:

### 1. Data capture method for Phase 1

**Question**: How should initial data (provider bills, EOBs, claim status, COBRA records) be captured given the need for value in days?

**Options**:
- **Option A**: Manual data entry forms (text fields, file upload for PDFs)
  - *Trade-off*: Slowest for ongoing use, but fastest to build and validate value
  - *Pro*: Can deliver working system in 2-3 days
  - *Con*: Requires manual effort every time; doesn't scale

- **Option B**: Automated scraping from institutional portals
  - *Trade-off*: Highest long-term value, but risky and slow to build
  - *Pro*: Zero vigilance tax after initial setup
  - *Con*: May take weeks; fragile to portal changes; may not unblock current harm

- **Option C**: Hybrid—manual entry for Phase 1, scraping for Phase 2+
  - *Trade-off*: Pragmatic sequencing; validates value before automation investment
  - *Pro*: Unblocks active problems immediately; defers automation until patterns clear
  - *Con*: Rebuilding data ingestion layer later

**Recommendation**: **Option C (hybrid)**. Start with manual entry to resolve Phase 1 problems in days. Add scraping in Phase 2 only if manual effort becomes constraining.

**Decision owner**: Implementation team; can pivot based on Phase 1 learnings.

**Rationale**: Constraint-first progression says "resolve active harm first." Manual entry unblocks bill reconciliation, claim tracking, and COBRA payment investigation immediately. Automation addresses future constraint (vigilance tax), not current constraint (lack of visibility).

---

### 2. Bill-to-EOB reconciliation approach

**Question**: How should the system help reconcile aggregated provider bills with detailed insurer EOBs given that itemized bills are missing?

**Options**:
- **Option A**: Manual matching UI (side-by-side comparison, user marks matches)
  - *Trade-off*: Labor-intensive but gives user full control and transparency
  - *Pro*: Works even when automated matching impossible; user learns patterns
  - *Con*: Tedious for large bill sets

- **Option B**: Automated matching by amount, date, provider (with confidence scoring)
  - *Trade-off*: Faster but may produce false positives/negatives
  - *Pro*: Reduces manual effort significantly
  - *Con*: Aggregated charges don't have 1:1 EOB line items; matching ambiguous

- **Option C**: Assisted matching (system suggests, user confirms/overrides)
  - *Trade-off*: Combines automation speed with user validation
  - *Pro*: Transparent (user sees suggestions and rationale); user can correct errors
  - *Con*: More complex to build than pure manual or pure automated

**Recommendation**: **Option A (manual) for Phase 1, Option C (assisted) if reconciliation becomes routine**.

**Decision owner**: After Phase 1 increment 1 (bill reconciliation prototype).

**Rationale**: The household has ~3-5 pending provider bills from one hospitalization. Manual matching UI can be built in 1-2 days and unblocks payment decisions immediately. If reconciliation becomes ongoing activity (multiple providers, recurring services), assisted matching adds value.

---

### 3. COBRA payment path investigation approach

**Question**: The household cannot pay upcoming COBRA coverage. How should the system help investigate and resolve this?

**Options**:
- **Option A**: Evidence capture tool (log contact attempts, responses, dates, people contacted)
  - *Trade-off*: Doesn't automate solution but provides audit trail for escalation
  - *Pro*: Can be built in hours; provides structure for multi-party coordination
  - *Con*: Doesn't solve payment path; just documents problem

- **Option B**: Automated COBRA administrator portal integration
  - *Trade-off*: Could enable direct payment if portal functional
  - *Pro*: If portal works, payment is resolved
  - *Con*: Portal may not work (that's the problem); integration takes days/weeks

- **Option C**: Hybrid—evidence capture + portal integration spike
  - *Trade-off*: Spike answers "is portal usable?"; evidence capture provides fallback
  - *Pro*: Rapid spike (4 hours) determines if automation viable; evidence tool useful regardless
  - *Con*: Spike may reveal portal unusable (but that's valuable information)

**Recommendation**: **Option C (hybrid)**. 4-hour spike to test COBRA portal payment flow. If broken/unusable, build evidence capture tool same day. If functional, integrate payment.

**Decision owner**: After COBRA payment spike (Phase 1 increment 3).

**Rationale**: Payment deadline approaching (constraint). Spike answers "is portal the problem or is process the problem?" quickly. Evidence tool supports manual resolution if needed.

---

### 4. Claim status tracking granularity

**Question**: What level of detail should the system track for claims given missing/incomplete claims and denied claims?

**Options**:
- **Option A**: High-level status only (submitted, pending, paid, denied)
  - *Trade-off*: Simpler to capture and display; may miss nuance
  - *Pro*: Fast to implement; sufficient for "is claim missing?" question
  - *Con*: Doesn't capture denial reasons, reprocessing state, appeals status

- **Option B**: Detailed tracking (denial codes, reprocessing, appeals, status history)
  - *Trade-off*: Comprehensive but complex data model
  - *Pro*: Answers Phase 1 Step 2 (denied claims) and Step 4 (missing claims) fully
  - *Con*: Takes longer to build; may include unused fields

- **Option C**: Progressive detail (start high-level, add fields as questions arise)
  - *Trade-off*: Incremental complexity growth
  - *Pro*: Delivers value quickly; expands based on real usage
  - *Con*: Schema evolution overhead

**Recommendation**: **Option C (progressive)**. Start with: claim ID, date of service, provider, amount, status (submitted/pending/paid/denied), denial reason (if denied), last updated. Add fields when real questions demand them.

**Decision owner**: Implementation team during Phase 1 increment 2 (claims tracking).

**Rationale**: Smallest useful increment. High-level status answers "is this claim missing?" and "why was it denied?" Additional detail (reprocessing attempts, appeal history) added when needed for resolution.

---

### 5. DXOS architecture pattern

**Question**: Should the system use the tuple-space pattern (from previous plan) or a simpler architecture given aggressive delivery timeline?

**Options**:
- **Option A**: Tuple-space pattern (multiple independent DXOS instances coordinating via tuples)
  - *Trade-off*: Elegant for automation and loose coupling, but adds complexity
  - *Pro*: Supports future automation (scrapers as independent services)
  - *Con*: Overkill for manual data entry; delays Phase 1 delivery

- **Option B**: Monolithic DXOS application (single process, direct function calls)
  - *Trade-off*: Simpler and faster to build; harder to add automation later
  - *Pro*: Can deliver Phase 1 in days; easier to debug; sufficient for manual workflows
  - *Con*: Refactoring needed for scraper independence (but that's Phase 2+ concern)

- **Option C**: Monolithic with abstraction layers (anticipate future tuple pattern)
  - *Trade-off*: Compromise between delivery speed and future flexibility
  - *Pro*: Clean interfaces make refactoring easier
  - *Con*: More upfront design than pure monolithic

**Recommendation**: **Option B (monolithic) for Phase 1**, revisit for Phase 2. Constraint-first says "deliver value in days." Tuple pattern is elegant but unnecessary for manual workflows.

**Decision owner**: Implementation team during Epic 1 (DXOS foundation).

**Rationale**: Phase 1 problems don't require scraping or background automation—they require visibility and evidence capture. Monolithic architecture delivers this faster. Refactor to tuple pattern if/when automation becomes constraining.

---

### 6. Multi-user (family) data model

**Question**: How should data for multiple family members be organized in DXOS?

**Options**:
- **Option A**: Single user, all family data in one Space
  - *Trade-off*: Simplest; assumes single household administrator
  - *Pro*: Fastest to implement; matches current reality (one person handling all admin)
  - *Con*: Doesn't scale if family members want independent access later

- **Option B**: Per-person Spaces with sharing
  - *Trade-off*: Future-proof but premature for current need
  - *Pro*: Supports autonomy
  - *Con*: Complex; delays delivery; overkill for household of 2-3 managed by one person

- **Option C**: Single Space, hierarchical data, document migration path
  - *Trade-off*: Pragmatic with documented future
  - *Pro*: Simple now, clear evolution path
  - *Con*: Migration work deferred

**Recommendation**: **Option A for Phase 1** (single Space, all family data). Document Option C migration path for future.

**Decision owner**: Implementation team during Epic 1 (data schema design).

**Rationale**: Current constraint is resolving active problems for household. Complex multi-user architecture is premature optimization.

---

## Chosen approach, workstreams & responsibilities

### Implementation approach

This system will be built as a **local-first DXOS application** using a **monolithic architecture** for Phase 1, with **manual data entry** as primary input method, following **constraint-first progression** to deliver value in days.

**Architecture: Monolithic DXOS application (Phase 1)**

For Phase 1, we adopt a simple, single-process architecture:

**Core layers**:
1. **Data persistence (DXOS)**: Stores coverage snapshots, claims, EOBs, provider bills, COBRA records, reconciliation state, evidence artifacts. Single DXOS Space for household data.

2. **Data entry UI**: Manual input forms for:
   - Provider bills (PDF upload + manual field extraction)
   - EOBs (PDF upload + manual field extraction)
   - Claim status (from insurer portal, manually transcribed)
   - Coverage snapshots (from employer/insurer/COBRA portals, manually transcribed)
   - COBRA payment attempts (evidence capture log)

3. **Analysis logic**: Direct function calls (no tuple pattern) for:
   - Bill-to-EOB reconciliation (manual matching UI)
   - Claim status tracking (detect missing claims)
   - Coverage state comparison (detect "terminated" discrepancies)
   - COBRA payment path monitoring
   - Evidence timeline construction

4. **Display UI**: Dashboards per Phase 1 problem:
   - Bill reconciliation view (provider bill vs EOB comparison)
   - Claim tracking view (list of claims, highlight missing/denied)
   - Coverage status view (current state per person, per source)
   - COBRA payment tracking view (timeline of payment attempts, evidence)

**Why monolithic for Phase 1?**
- **Speed**: Can deliver working system in 2-5 days per problem
- **Simplicity**: Easier to debug; fewer moving parts; direct function calls
- **Sufficient**: Phase 1 problems don't require automation or background processing
- **Refactorable**: Clean interfaces allow evolution to tuple pattern in Phase 2 if automation becomes valuable

**Why manual data entry for Phase 1?**
- **Fastest to build**: Forms and file upload can be implemented in hours
- **Validates value**: Proves system helps before investing in automation
- **Unblocks immediately**: Household can start reconciling bills, tracking claims, documenting COBRA issues today
- **Defers complexity**: Scraping is fragile, takes weeks, and may not unblock current problems

**Must review before implementation**: DXOS documentation on Spaces, Echo schema patterns, data persistence, file storage (for PDFs).

---

### Why not alternative approaches

**Automated scraping first**: Would delay resolving active problems by weeks. Constraint-first says solve active harm now, optimize later.

**Tuple-space architecture**: Elegant for future automation but premature for manual workflows. Adds days to delivery with no Phase 1 benefit.

**Cloud-first SaaS**: Violates local-first learning goal and data ownership constraint. Rejected.

**Mobile-first**: DXOS compatibility unclear; desktop sufficient for initial household use. Could be Phase 3+.

**Blockchain/distributed ledger**: Overkill; doesn't solve data access or reconciliation problems.

---

### Workstreams

#### Workstream 1: DXOS foundation & data schema (foundational)

Establish DXOS development environment, design schema for coverage, claims, bills, EOBs, COBRA records, and evidence artifacts. Implement single-Space household data model.

**Key artifacts to review before implementation**:
- DXOS documentation: Getting Started, Echo schema patterns, Space and Identity management
- DXOS file storage patterns (for PDF uploads)
- DXOS query patterns (for reconciliation matching)

**Coherent slices**:
- Development environment setup
- Core schema design (Person, CoverageSnapshot, Claim, EOB, ProviderBill, COBRARecord, EvidenceLog, ReconciliationMatch)
- Space initialization for household
- Basic CRUD operations on local data
- PDF file upload and storage

---

#### Workstream 2: Bill reconciliation (Phase 1, Problem 1)

Build UI to manually match provider bills with insurer EOBs, enabling household to determine which charges are valid and which require itemized bills.

**Key patterns to establish**:
- Bill and EOB data entry (manual or PDF upload with field extraction)
- Side-by-side comparison UI (bill charges vs EOB line items)
- Manual matching workflow (user marks which EOB lines correspond to bill charges)
- Reconciliation state tracking (matched, unmatched, pending itemized detail)
- "What's blocking payment" summary view

**Coherent slices**:
- Provider bill entry form (date, provider, total amount, aggregated charges, PDF attachment)
- EOB entry form (claim number, date of service, provider, service lines, patient responsibility, PDF attachment)
- Side-by-side reconciliation view (bill on left, EOB on right, user draws connections)
- Reconciliation state summary ("Bill A: $X matched, $Y unmatched, requires itemized detail")
- Evidence export (PDF generation or copy-as-text for provider communication)

---

#### Workstream 3: Claims tracking & denial detection (Phase 1, Problem 2 & 4)

Build UI to track claim status, detect claims denied for "terminated coverage," and identify missing claims.

**Key patterns to establish**:
- Claim data entry (claim ID, date of service, provider, amount, status, denial reason)
- Coverage snapshot entry (employer, insurer, COBRA - status per person per date)
- Cross-reference logic (denied claim date vs coverage snapshot date → discrepancy detection)
- Missing claim detection (expected service occurred but no claim in system)
- Denial reason pattern analysis (flag "coverage" or "eligibility" reasons)

**Coherent slices**:
- Claim entry form (claim number, date of service, provider, billed amount, status, denial reason if applicable)
- Coverage snapshot entry form (source, person, effective dates, status, screenshot/PDF evidence)
- Claim list view (sortable by status, filterable by person/provider/date range)
- Denial detection alert ("Claim X denied for coverage reason, but coverage snapshot shows active on date of service")
- Missing claim detection ("Service on DATE at PROVIDER has no corresponding claim in system")
- Evidence timeline (show coverage state at time of claim adjudication)

---

#### Workstream 4: COBRA payment tracking (Phase 1, Problem 3)

Build evidence capture tool for COBRA payment attempts, enabling household to document failed payment paths and coordinate resolution with administrator.

**Key patterns to establish**:
- Evidence log entry (date, contact method, person contacted, outcome, notes, attachments)
- COBRA period tracking (upcoming coverage periods, payment deadlines)
- Payment path status (attempted, failed, alternate offered, resolved)
- Escalation support (timeline view for case building)

**Coherent slices**:
- COBRA period configuration (start date, end date, payment amount, deadline)
- Evidence log form (timestamp, contact type, summary, detailed notes, attachments)
- Payment attempt tracking ("Attempted payment via portal on DATE - failed")
- Timeline view (chronological log of all payment attempts and responses)
- Escalation summary ("X attempts since DATE, no working payment path confirmed")

---

#### Workstream 5: Minimal UI & evidence presentation

Build simple, accessible UI showing status per Phase 1 problem with plain-language explanations and evidence export.

**Key patterns to establish**:
- Problem-centric dashboard (one card per Phase 1 problem showing current state)
- Evidence artifacts (exportable summaries for provider/insurer communication)
- Plain-language explanations (no jargon; clear next actions)

**Coherent slices**:
- Dashboard layout (4 cards: Bill Reconciliation, Claims Tracking, COBRA Payment, Overall Status)
- Problem status indicators (green = resolved, yellow = in progress, red = blocked)
- Evidence export per problem (e.g., "Claim X: denied for coverage, but coverage was active - here's proof")
- Settings (family members, institutional sources, manual data refresh prompts)

---

## Risks, assumptions, and mitigations

### Risks

1. **Manual data entry is too tedious; household abandons system**
   - *What could go wrong*: Data entry takes >15 minutes per session; household stops using system before problems resolved
   - *Mitigation*: Optimize forms for speed (sensible defaults, auto-fill where possible, PDF upload with manual extraction assistance). Measure time per entry; if >10 minutes, simplify.
   - *Detection*: Track usage frequency. If entries stop before problems resolved, interview household to understand friction.

2. **Provider bills cannot be reconciled without itemized detail**
   - *What could go wrong*: Even with manual matching UI, aggregated charges vs detailed EOB lines don't map; payment decisions still blocked
   - *Mitigation*: System explicitly flags "requires itemized bill" rather than forcing incorrect match. Produces evidence artifact: "Provider billed $X aggregated; EOB shows $Y detailed lines; cannot verify patient responsibility without itemization."
   - *Detection*: If reconciliation view used but no payment decisions made, matching may be insufficient.

3. **DXOS maturity gaps block Phase 1 delivery**
   - *What could go wrong*: DXOS lacks features needed for PDF storage, data queries, or UI rendering, causing delays
   - *Mitigation*: Early spike (Epic 1, de-risking action #1) validates DXOS for required features. If blocking gaps found, pivot to SQLite + local-first UI (still respects data ownership).
   - *Detection*: If DXOS spike takes >2 days or reveals blocking limitations, architectural pivot required.

4. **Denied claims cannot be resolved through system visibility alone**
   - *What could go wrong*: Household sees denied claims + evidence of active coverage, but insurer won't reprocess; external escalation required beyond system scope
   - *Mitigation*: System provides evidence artifacts (coverage proof, denial details, timeline) for external escalation. Success = "household has tools to escalate," not "system auto-resolves."
   - *Detection*: If reprocessing requests sent but claims remain denied, system's role is evidence provision, not resolution.

5. **COBRA payment path issue is systemic, not data problem**
   - *What could go wrong*: Evidence capture reveals COBRA administrator's system is broken; no amount of data visibility resolves payment path
   - *Mitigation*: Evidence log supports escalation to employer/administrator with documented timeline. System success = "household has audit trail for escalation," not "payment path fixed."
   - *Detection*: If payment attempts fail despite documented evidence, problem is institutional (out of system scope).

6. **Phase 1 problems get resolved externally before system delivers**
   - *What could go wrong*: Insurer reprocesses claims, provider issues itemized bills, COBRA admin fixes payment, rendering system unnecessary
   - *Mitigation*: This is a success outcome for household (problems resolved). System still validates value by capturing what would have been needed. Learning goals (DXOS, GenAI) still achieved.
   - *Detection*: If external resolution occurs, document "what system would have shown" for retrospective validation.

7. **"Days not weeks" timeline is too aggressive**
   - *What could go wrong*: Increments take 5+ days each; Phase 1 total timeline stretches to 3-4 weeks, delaying resolution
   - *Mitigation*: Ruthlessly scope each increment to "smallest useful." If increment taking >3 days, cut features to accelerate. Imperfect delivery that helps is better than perfect delivery that's late.
   - *Detection*: Daily check-in: "Did this move toward resolving a Phase 1 problem?" If not, pivot.

### Assumptions

1. **Household can extract data from PDFs and portals**: Assumes ability to access insurer/provider/COBRA systems and manually transcribe or upload data.
   - *Validation approach*: Test data entry UI with real PDFs and portal screenshots during Epic 1.

2. **Bill-to-EOB matching is possible with manual judgment**: Assumes household member can identify which EOB lines might correspond to aggregated bill charges, even if ambiguous.
   - *Validation approach*: Build reconciliation UI prototype with real bills/EOBs, test with household member.

3. **DXOS is sufficiently mature for this scope**: Assumes DXOS can handle data persistence, file storage, queries, and UI rendering without blocking issues.
   - *Validation approach*: Epic 1 spike validates DXOS capabilities before deep implementation.

4. **Manual workflows are acceptable for Phase 1**: Assumes household willing to manually enter data for 2-4 weeks to resolve active problems before automation is prioritized.
   - *Validation approach*: Set expectation upfront: "manual entry for Phase 1, automation in Phase 2 if needed."

5. **Evidence artifacts support external resolution**: Assumes that providing household with clear evidence (coverage proof, denial details, reconciliation gaps, payment timeline) enables them to escalate issues with institutions.
   - *Validation approach*: Test evidence export (PDF or text) with real data; household reviews for "would this help in a phone call with insurer?"

---

## Detail the work: Dependencies & sequencing

### Phase 1 — Resolve Active Failures (Epic 1-5)

---

### Epic 1: DXOS foundation & minimal data schema
**Dependencies**: None (foundational work)

**Purpose**: Establish DXOS development environment and validate it can support Phase 1 requirements (data persistence, file storage, queries, UI rendering). Design minimal schema for Phase 1 entities.

**Tasks**:

1. **Environment setup & DXOS installation**
   - Install DXOS CLI and development dependencies
   - Initialize DXOS project structure
   - Verify local node can start and persist data
   - Document setup steps for reproducibility

2. **DXOS capability spike**
   - Survey DXOS documentation: Echo schema, Space management, file storage (PDFs), UI integration, data queries
   - Build proof-of-concept: create Space, define simple schema (Person, Claim), persist data, store PDF file, retrieve and display
   - Test query patterns (e.g., "find all claims for person X with status=denied")
   - Document findings: what's supported, what's missing, what's unclear
   - **Decision gate**: Is DXOS viable for Phase 1? If blocking gaps, pivot to SQLite + local-first UI.

3. **Minimal schema design (Phase 1 scope only)**
   - Design Echo schema for:
     - **Person** (household member)
     - **CoverageSnapshot** (source, person, effective dates, status, evidence file)
     - **Claim** (claim ID, person, date of service, provider, amount, status, denial reason, notes)
     - **EOB** (claim ref, service lines, amounts, patient responsibility, PDF file)
     - **ProviderBill** (provider, date, total amount, line items/aggregated charges, PDF file)
     - **ReconciliationMatch** (bill line ↔ EOB line mapping, match confidence, notes)
     - **COBRARecord** (coverage period, payment deadline, payment attempts, status)
     - **EvidenceLog** (timestamp, problem ref, log type, summary, details, attachments)
   - Implement schemas in DXOS Echo format
   - Validate with test data

4. **Space initialization for household**
   - Implement Space creation for single household
   - Add household members (Person entities)
   - Implement basic CRUD operations (create, read, update, delete for all entity types)
   - Test: Add person, add claim, retrieve claims for person

5. **Minimal UI scaffolding**
   - Set up React (or DXOS-recommended UI framework)
   - Implement DXOS data binding to UI components
   - Create minimal navigation: Dashboard, Bills, Claims, COBRA, Settings
   - Verify data flow: DXOS → UI rendering

**Testing approach**:
- **Unit tests**: Schema validation, CRUD operations
- **Integration tests**: DXOS data persistence across sessions, file storage and retrieval
- **Manual testing**: Can create Space, add person, add claim with PDF, retrieve and display

**Validation criteria**:
- DXOS development environment operational and documented
- Phase 1 schemas implemented and can store/retrieve test data with PDF attachments
- Basic UI can render DXOS data for household members
- DXOS capability gaps (if any) documented; pivot decision made if blocking
- Time to complete: ≤3 days (if >3 days, DXOS may be too immature; consider pivot)

---

### Epic 2: Bill reconciliation tool (Phase 1, Problem 1)
**Dependencies**: Epic 1 (requires DXOS foundation and schema)

**Purpose**: Enable household to reconcile provider bills with EOBs, identify unmatched charges, and determine which bills can be paid vs which require itemized detail.

**Tasks**:

1. **Provider bill entry form**
   - Build form: provider name, bill date, total amount, aggregated line items (description + amount), notes
   - PDF upload: attach provider bill PDF
   - Store ProviderBill entity in DXOS with file reference
   - Test: Enter real provider bill from hospitalization, verify storage

2. **EOB entry form**
   - Build form: claim number, date of service, provider, detailed service lines (procedure, amount billed, amount allowed, amount paid, patient responsibility), notes
   - PDF upload: attach EOB PDF
   - Link to Claim entity if exists (optional for Phase 1)
   - Store EOB entity in DXOS with file reference
   - Test: Enter real EOB, verify storage

3. **Bill list view**
   - Display all provider bills (provider, date, total amount, reconciliation status: unreconciled/partial/complete)
   - Click bill → open reconciliation view
   - Test: Display 3 provider bills from real data

4. **Reconciliation view (side-by-side comparison)**
   - Left panel: Provider bill (aggregated charges)
   - Right panel: Available EOBs (detailed service lines)
   - User workflow:
     - Select bill charge (left) and EOB service line (right)
     - Click "Match" → creates ReconciliationMatch entity
     - Mark bill charge as "Requires Itemized Detail" if no match possible
   - Show totals: Bill total, Matched amount, Unmatched amount, EOB patient responsibility
   - Test: Match real bill charges to real EOB lines, verify match persistence

5. **Reconciliation status summary**
   - For each bill, show:
     - Total billed amount
     - Amount matched to EOBs (with confidence/notes)
     - Amount unmatched (requires itemized detail or further investigation)
     - Patient responsibility according to matched EOBs
     - Next action: "Can pay $X" or "Request itemized bill for $Y"
   - Test: View summary for 3 real bills, verify accuracy

6. **Evidence export (plain text summary)**
   - Generate exportable summary for provider communication:
     - "Bill from [Provider] on [Date] for $[Amount]"
     - "Matched charges: [list with EOB references]"
     - "Unmatched charges: [list]"
     - "Action required: Itemized bill needed for charges X, Y, Z"
   - Copy-as-text button (PDF export deferred)
   - Test: Generate summary for real bill, household reviews for usefulness

**Testing approach**:
- **Unit tests**: Form validation, CRUD operations for ProviderBill/EOB/ReconciliationMatch
- **Integration tests**: End-to-end flow (enter bill → enter EOB → match → view summary)
- **Usability test**: Household member uses reconciliation view with real bills/EOBs, completes matching in <15 minutes

**Validation criteria**:
- Household can enter 3 real provider bills and corresponding EOBs
- Household can match bill charges to EOB lines (or mark "requires itemized detail")
- Reconciliation summary clearly shows what can be paid vs what's blocked
- Evidence export provides useful text for provider communication
- **Problem resolution**: Household can make payment decisions for at least 1 provider bill

---

### Epic 3: Claims tracking & denial detection (Phase 1, Problems 2 & 4)
**Dependencies**: Epic 1 (requires DXOS foundation); partially overlaps Epic 2 (can start after Epic 1)

**Purpose**: Track claim status, detect claims denied for "terminated coverage" despite active coverage evidence, and identify missing claims.

**Tasks**:

1. **Claim entry form**
   - Build form: claim number, person, date of service, provider, billed amount, claim status (submitted/pending/paid/denied), denial reason (if denied), notes
   - Optional: link to EOB entity if exists
   - Store Claim entity in DXOS
   - Test: Enter 5 real claims (mix of paid, pending, denied)

2. **Coverage snapshot entry form**
   - Build form: source (employer/insurer/COBRA), person, effective start date, effective end date, coverage status (active/inactive/pending), notes
   - Screenshot or PDF upload: evidence from portal
   - Store CoverageSnapshot entity in DXOS with file reference
   - Test: Enter coverage snapshots for 2 family members across 3 sources

3. **Claim list view**
   - Display all claims: claim number, person, date of service, provider, amount, status, denial reason
   - Sortable by date, person, status
   - Filterable: show only denied claims, show only missing claims
   - Click claim → claim detail view
   - Test: Display 10 claims with filtering and sorting

4. **Denial detection logic**
   - For each denied claim:
     - Check denial reason for keywords: "coverage", "eligibility", "terminated", "inactive"
     - Query CoverageSnapshot entities for same person on date of service
     - If coverage snapshot shows "active" but claim denied for coverage reason → flag discrepancy
   - Display alert: "Claim X denied for coverage on [date], but coverage snapshot shows active"
   - Test: Enter denied claim (coverage reason) + active coverage snapshot, verify discrepancy flagged

5. **Missing claim detection (manual prompt)**
   - User workflow: "Did service occur on [date] that should have a claim?"
   - If yes, mark as "Expected claim - missing from system"
   - Track expected claims (date of service, provider, estimated amount, notes)
   - List view: show expected claims not found in insurer portal
   - Test: Enter expected service, verify it appears in "missing claims" list

6. **Claim detail view & evidence timeline**
   - Show claim details (all fields)
   - Show related coverage snapshots at time of service (from all sources)
   - Show discrepancy if coverage-related denial + active coverage
   - Evidence export: "Claim [ID] denied on [date] with reason '[text]'. Coverage snapshot from [source] shows active coverage on [DOS]. Reprocessing requested."
   - Test: View denied claim detail, verify evidence is clear and actionable

7. **Coverage status dashboard**
   - For each person, show current coverage status per source (employer, insurer, COBRA)
   - Highlight discrepancies across sources (e.g., employer shows active, insurer shows terminated)
   - Click person → coverage history (list of snapshots over time)
   - Test: Display coverage for 2 people, verify discrepancies highlighted

**Testing approach**:
- **Unit tests**: Claim CRUD, CoverageSnapshot CRUD, denial detection logic
- **Integration tests**: Enter claim + coverage → detect discrepancy → generate evidence
- **Scenario tests**:
  - Denied claim (coverage reason) + active coverage → discrepancy detected
  - Denied claim (other reason) + active coverage → no false alarm
  - Expected service + no claim in system → missing claim detected

**Validation criteria**:
- Household can enter 10 real claims with current status
- Household can enter coverage snapshots for family members (employer, insurer, COBRA)
- System detects and flags claims denied for coverage reasons when coverage snapshots show active
- Missing claims can be tracked (expected service not found in insurer system)
- Evidence export provides clear proof for insurer reprocessing request
- **Problem resolution**: Household has evidence to request reprocessing for at least 2 denied claims

---

### Epic 4: COBRA payment tracking & evidence log (Phase 1, Problem 3)
**Dependencies**: Epic 1 (requires DXOS foundation); can run parallel to Epic 2/3

**Purpose**: Track COBRA payment attempts, document failed payment paths, and build evidence timeline for escalation to employer/administrator.

**Tasks**:

1. **COBRA coverage period configuration**
   - Build form: coverage period (start date, end date), monthly premium amount, payment deadline
   - Store COBRARecord entity in DXOS
   - Test: Enter upcoming COBRA period (e.g., February 2026, $X premium, due Jan 31)

2. **Payment attempt log entry form**
   - Build form: timestamp, payment method attempted (portal/mail/phone), amount, outcome (success/failed/pending), error details, notes
   - Attachment upload: screenshots of errors, confirmation emails, etc.
   - Link to COBRARecord (which period this attempt was for)
   - Store as EvidenceLog entity in DXOS with attachments
   - Test: Log 3 real payment attempts with different outcomes

3. **Contact log entry form**
   - Build form: timestamp, contact method (phone/email/portal message), person/department contacted, summary, detailed notes, response received
   - Attachment upload: email screenshots, case numbers, etc.
   - Link to COBRARecord
   - Store as EvidenceLog entity
   - Test: Log 2 real contact attempts with administrator

4. **COBRA timeline view**
   - Chronological display of all payment attempts and contact logs for a coverage period
   - Show: date, event type (payment attempt/contact), summary, outcome
   - Click event → full details and attachments
   - Visual timeline (can be simple list with timestamps initially)
   - Test: Display timeline for February COBRA period with 5 logged events

5. **COBRA status summary**
   - For each coverage period, show:
     - Payment deadline
     - Payment status (paid/pending/failed/no working path)
     - Number of payment attempts
     - Number of contacts with administrator
     - Days until deadline (if unpaid)
     - Next action: "Escalate to employer" / "Retry payment" / "Resolved"
   - Test: View status for upcoming period, verify urgency is clear

6. **Escalation evidence export**
   - Generate summary for employer/administrator escalation:
     - "COBRA coverage for [period] due [date]"
     - "Payment attempts: [count] attempts since [first attempt date], all failed"
     - "Administrator contacts: [count] attempts, no resolution"
     - "Evidence attached: [list of screenshots/emails]"
     - "Action required: Immediate resolution or alternate payment path"
   - Copy-as-text (PDF export deferred)
   - Test: Generate escalation summary, household reviews for completeness

**Testing approach**:
- **Unit tests**: COBRARecord CRUD, EvidenceLog CRUD, timeline generation
- **Integration tests**: End-to-end flow (configure period → log payment attempts → view timeline → export evidence)
- **Manual test**: Household logs real payment attempts and contacts, reviews timeline for accuracy

**Validation criteria**:
- Household can configure upcoming COBRA coverage period with deadline
- Household can log payment attempts (≥3 real attempts)
- Household can log contacts with administrator (≥2 real contacts)
- Timeline view shows chronological evidence of payment path failure
- Escalation evidence export provides clear case for employer/administrator resolution
- **Problem resolution**: Household has documented evidence for external escalation (even if payment path not fixed by system)

---

### Epic 5: Minimal dashboard & problem status overview
**Dependencies**: Epic 2, 3, 4 (requires problem-specific tools built)

**Purpose**: Provide household with simple, at-a-glance view of Phase 1 problem status and next actions.

**Tasks**:

1. **Dashboard layout (problem-centric)**
   - Create dashboard with 4 cards, one per Phase 1 problem:
     - **Card 1**: Bill Reconciliation Status
     - **Card 2**: Claims Tracking Status
     - **Card 3**: COBRA Payment Status
     - **Card 4**: Overall Household Status
   - Each card shows: status indicator (green/yellow/red), summary text, next action
   - Click card → navigate to detailed tool (Epic 2/3/4)
   - Test: Display dashboard with real data

2. **Bill Reconciliation status card**
   - Show: number of bills (total, reconciled, pending)
   - Show: total unmatched charges requiring itemized detail
   - Next action: "Review 2 unreconciled bills" or "All bills reconciled"
   - Status: green if all reconciled, yellow if pending, red if blocked >7 days
   - Test: Verify card reflects Epic 2 data accurately

3. **Claims Tracking status card**
   - Show: number of claims (total, denied with coverage discrepancy, missing expected claims)
   - Show: number of claims requiring reprocessing request
   - Next action: "Request reprocessing for 2 claims" or "All claims resolved"
   - Status: green if no discrepancies, yellow if action needed, red if denied claims blocking care
   - Test: Verify card reflects Epic 3 data accurately

4. **COBRA Payment status card**
   - Show: upcoming coverage period deadline (days remaining)
   - Show: payment status (failed X attempts, no working path)
   - Next action: "Escalate to employer" or "Payment confirmed"
   - Status: red if deadline <7 days and unpaid, yellow if deadline <14 days, green if paid
   - Test: Verify card reflects Epic 4 data accurately

5. **Overall Household status summary**
   - Aggregate view: "X problems resolved, Y in progress, Z blocked"
   - Priority: show most urgent next action across all problems
   - Link to evidence exports for all problems
   - Test: Verify overall status matches individual problem statuses

6. **Settings & configuration**
   - Household members: list, add, edit, remove (Person entities)
   - Institutional sources: list employer, insurer, COBRA administrator (for coverage snapshots)
   - Data refresh prompt: "When did you last check insurer portal?" (reminder for manual data entry)
   - Test: Add family member, update insurer name, verify persistence

**Testing approach**:
- **Unit tests**: Dashboard card rendering with mock data
- **Integration tests**: Dashboard reflects real data from Epic 2/3/4 tools
- **Usability test**: Household member views dashboard, answers "What problems are resolved? What's the next action?" in <30 seconds

**Validation criteria**:
- Dashboard shows at-a-glance status for all Phase 1 problems
- Status indicators (green/yellow/red) accurately reflect problem urgency
- Next actions are clear and actionable (non-technical language)
- Household can navigate from dashboard to detailed tools (Epic 2/3/4) in 1 click
- **Overall Phase 1 validation**: Household reports measurable progress on ≥2 of 4 active problems

---

## Early de-risking actions

The following actions should be completed in the **first 3 days** to reduce the biggest risks:

### 1. DXOS proof-of-concept (1-2 days)
**Goal**: Validate that DXOS can support Phase 1 requirements (data persistence, file storage for PDFs, queries, UI rendering) before committing to implementation.

**Tasks**:
- Install DXOS CLI and initialize project
- Create simple schema: Person, Claim, ProviderBill (with PDF file)
- Persist data, store PDF file, retrieve both, verify they survive restart
- Build minimal UI that renders DXOS data and displays PDF
- Test query: "Get all claims for person X where status=denied"
- Document: what works, what doesn't, what's unclear

**Success criteria**: Can store household data (claims, bills) with PDF attachments using DXOS; basic UI renders data; queries work for filtering (e.g., denied claims). Architectural gaps (if any) are documented.

**If this fails**: Pivot to SQLite + local-first UI. Still respects data ownership constraint, but loses DXOS learning goal. Acceptable trade-off if DXOS blocks delivery.

**Time budget**: 1-2 days max. If taking longer, DXOS may be too immature; pivot to SQLite.

---

### 2. Bill reconciliation prototype with real data (1 day)
**Goal**: Validate that manual bill-to-EOB matching is practical and helps household make payment decisions.

**Tasks**:
- Build minimal UI: left panel (provider bill charges), right panel (EOB lines), match button
- Load 1 real provider bill and corresponding EOB(s)
- Household member attempts to match charges to EOB lines
- Measure: time to complete, level of frustration, whether payment decision clearer after matching

**Success criteria**: Household member can complete matching in <15 minutes; can articulate "I can pay $X" or "I need itemized bill for $Y" after matching.

**If this reveals issues**: If matching is too ambiguous (can't determine correct matches), simplify to "flag unmatched" workflow: system shows bill vs EOB side-by-side, user marks unmatched charges as "needs itemized detail."

**Time budget**: 1 day (8 hours including UI building and testing).

---

### 3. Claims denial detection with real data (4 hours)
**Goal**: Validate that denied-claim-with-active-coverage detection logic identifies real discrepancies from household data.

**Tasks**:
- Load 2 real denied claims (from insurer portal screenshots)
- Load corresponding coverage snapshots (employer, insurer, COBRA) for dates of service
- Implement simple detection: if (claim.status === 'denied' && claim.denialReason.includes('coverage') && coverageSnapshot.status === 'active') → flag discrepancy
- Run detection, verify it flags the 2 real denied claims
- Generate evidence text: "Claim X denied for coverage, but coverage was active - see attached snapshot"

**Success criteria**: Detection logic correctly identifies 2 real denied claims with coverage discrepancies; evidence text is clear and actionable for reprocessing request.

**If this reveals issues**: If denial reasons don't include "coverage" keywords, expand keyword list. If coverage snapshots don't align with claim dates, add "coverage effective date" field to snapshots.

**Time budget**: 4 hours.

---

### 4. COBRA evidence log with real timeline (2 hours)
**Goal**: Validate that evidence logging provides useful escalation artifact for COBRA payment issue.

**Tasks**:
- Build minimal evidence log form (timestamp, summary, details, attachment)
- Household member logs 3 real payment attempts (with dates, outcomes, screenshots)
- Generate chronological timeline view
- Generate escalation summary text: "3 payment attempts since [date], all failed - evidence attached"

**Success criteria**: Timeline shows clear chronological record of payment failures; escalation summary provides useful artifact for employer/administrator escalation.

**If this reveals issues**: If timeline is unclear, improve visual presentation (add icons, color-code outcomes). If summary text is insufficient, add template fields (dates, contact names, case numbers).

**Time budget**: 2 hours.

---

### 5. End-to-end Phase 1 problem resolution simulation (4 hours)
**Goal**: Validate that Epic 2/3/4 tools, in combination, help household make progress on active problems.

**Tasks**:
- Use prototypes from de-risking actions #2, #3, #4
- Household member completes workflows:
  - Reconcile 1 provider bill (Epic 2 prototype)
  - Review denied claims, identify 1 with coverage discrepancy (Epic 3 prototype)
  - Review COBRA payment timeline, generate escalation summary (Epic 4 prototype)
- Interview: "Did these tools help? What's still unclear? What's missing?"
- Measure: Did household make progress (payment decision, reprocessing request drafted, escalation evidence ready)?

**Success criteria**: Household reports tools helped make progress on ≥2 of 4 Phase 1 problems; identifies specific next actions (e.g., "call insurer with this evidence," "request itemized bill from provider").

**If this reveals issues**: Prioritize gaps identified by household. If reconciliation most helpful, invest more in Epic 2. If claims tracking most helpful, invest more in Epic 3.

**Time budget**: 4 hours (assuming prototypes from #2/#3/#4 exist).

---

## Readiness checklist

Before converting this Plan into work items, confirm:

### Detail completeness
- [ ] Are epics broken down into specific, independently deliverable tasks?
- [ ] Are dependencies between epics clearly identified?
- [ ] Are validation criteria defined for each epic (observable outcomes)?
- [ ] Are testing approaches specified (unit/integration/usability)?
- [ ] Are de-risking actions scheduled in first 3 days?

### Constraint adherence
- [ ] Does sequencing follow constraint-first progression (Phase 1 problems before Phase 2)?
- [ ] Is each epic scoped as "smallest useful increment"?
- [ ] Are "days not weeks" timelines realistic (Epic 1: ≤3 days, Epic 2/3/4: ≤5 days each)?
- [ ] Are non-negotiables (transparency, justified confidence, local-first, non-technical usability) validated in each epic?
- [ ] Are manual workflows accepted for Phase 1 (automation deferred)?

### Decision clarity
- [ ] Are the 6 unknowns resolved with decisions documented?
- [ ] Are decision owners identified?
- [ ] Are decision points tied to specific epics/spikes?
- [ ] Is pivot path documented if DXOS fails (SQLite alternative)?

### Risk mitigation
- [ ] Are the 7 risks documented with realistic mitigations?
- [ ] Are detection mechanisms in place for each risk?
- [ ] Is "external resolution" accepted as success for some problems (e.g., COBRA payment)?
- [ ] Is "evidence provision" accepted as success when system can't auto-resolve?

### Problem-driven validation
- [ ] Is success measured by Phase 1 problem resolution, not system completeness?
- [ ] Are validation criteria tied to real household problems (not abstract metrics)?
- [ ] Is household involvement explicit in testing/validation (usability tests, interviews)?
- [ ] Is retrospective planned after Phase 1 to evaluate constraint-first approach?

### Learning focus
- [ ] Is learning captured as explicit outcome (GenAI collaboration, DXOS patterns)?
- [ ] Are pivot decisions documented as learning artifacts (e.g., if DXOS insufficient)?
- [ ] Is Proposal's success questions framework referenced for retrospective?

---

## Plan status

This Plan is **ready for implementation**.

**Key decisions incorporated**:
- **Data capture**: Manual entry (Option C hybrid) for Phase 1; defer scraping to Phase 2
- **Reconciliation**: Manual matching UI (Option A) for Phase 1; defer automation if not constraining
- **COBRA**: Evidence capture + portal spike (Option C); accept external resolution
- **Claim tracking**: Progressive detail (Option C); start high-level, expand as needed
- **Architecture**: Monolithic DXOS (Option B) for Phase 1; revisit tuple pattern for Phase 2
- **Multi-user**: Single Space (Option A); defer multi-user complexity

**Sequencing**: Constraint-first, problem-driven:
- Epic 1 (DXOS foundation): 2-3 days
- Epic 2 (Bill reconciliation): 3-5 days (solves Phase 1 Problem 1)
- Epic 3 (Claims tracking): 3-5 days (solves Phase 1 Problems 2 & 4)
- Epic 4 (COBRA tracking): 2-3 days (supports Phase 1 Problem 3 resolution)
- Epic 5 (Dashboard): 1-2 days (integrates Epic 2/3/4)

**Total Phase 1 timeline**: 11-18 days (target: 2 weeks)

**Next steps**:
1. **Days 1-3**: Complete all 5 de-risking actions; validate approach
2. **Days 4-6**: Epic 1 (DXOS foundation)
3. **Days 7-11**: Epic 2 (Bill reconciliation) — first problem resolved
4. **Days 12-16**: Epic 3 (Claims tracking) — second/fourth problems resolved
5. **Days 17-19**: Epic 4 (COBRA tracking) — third problem evidence ready
6. **Days 20-21**: Epic 5 (Dashboard) — integrated view
7. **Day 22**: Phase 1 retrospective — evaluate constraint-first progression, identify Phase 2 constraint

**Phase 2 planning**: After Phase 1 complete, re-evaluate constraints. If manual data entry becomes most constraining (taking >10 min/day), prioritize scraping automation. If prevention more valuable than ongoing resolution, address Phase 2 problems (dependents dropped, enrollment errors, etc.).

---

*This Plan implements the Proposal documented in [docs/planning/proposal.md](./proposal.md) using constraint-first progression to resolve active household harm in days.*

*End of Plan*
