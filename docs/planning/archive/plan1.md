# Plan
## Delegated Insurance Coverage Assurance System
*Local-first eligibility monitoring to detect institutional failures before they cause harm*

**Proposal reference**: [docs/planning/proposal.md](./proposal.md)

---

## The Brief

This plan delivers a local-first insurance eligibility assurance system for individual and family use, built on DXOS. The system continuously monitors coverage status across institutional sources (employer, insurer, COBRA administrator) and alerts users to discrepancies within 1 day—before denied claims or disrupted care occur.

**Context**: Individuals delegate insurance administration to institutions yet remain exposed when coverage silently lapses or diverges. Current discovery is reactive: problems surface only after harm (denied claims, unexpected bills, care access issues). This system shifts detection earlier, reducing the vigilance tax on individuals and rebalancing the power asymmetry with institutions.

**This is a learning-first project** with three interconnected goals:
1. Understand GenAI as a first-class collaborator across the full product lifecycle (Discovery → Proposal → Plan → Delivery)
2. Learn local-first architectural constraints through DXOS implementation
3. Build a real, prosocial system that solves a personally meaningful problem

**Important non-negotiables being respected**:
- Coverage discrepancies affecting care must be detected within 1 day
- The system must be usable by non-technical people
- Local-first data ownership cannot be compromised
- The system must never assert correctness it cannot justify
- Conflicting institutional states must always be surfaced, never hidden

**Explicit boundaries**: This system does NOT automate correction/appeals, optimize claims, commercialize initially, or attempt to override institutional systems.

---

## Constraints

These implementation guardrails must be referenced during review and survive conversion into work items:

1. **1-day detection requirement**: Coverage discrepancies that could affect access to care must be detected and surfaced within 24 hours
2. **Transparency over certainty**: Conflicting institutional states must always be surfaced to the user; uncertainty must be explicit, not hidden
3. **Individual-level visibility**: Each covered individual's eligibility status must be independently observable
4. **Zero vigilance tax**: The system must operate without requiring continuous manual checking by the user
5. **Non-technical usability**: The system must be understandable and actionable by people without technical expertise
6. **Justified confidence only**: The system must never assert correctness beyond what its evidence supports; confidence levels must be explicit
7. **Inviolable local-first ownership**: User data must remain under local control; DXOS architecture must not be compromised for convenience
8. **Learning over optimization**: When choosing between learning opportunities and efficiency, prefer learning (e.g., prefer understanding DXOS deeply over using familiar but cloud-first alternatives)
9. **Observability over authority**: Design for helping users understand what's happening, not for making decisions on their behalf
10. **Asymmetry rebalancing**: Every design choice must be evaluated against whether it increases individual confidence and reduces institutional information advantage

---

## Unknowns requiring decision

The following decisions must be made before or during delivery:

### 1. Data acquisition method
**Options**:
- **Option A**: Manual input of coverage status at defined intervals (daily/weekly check-ins)
  - *Trade-off*: Lowest technical risk, still reduces vigilance vs. status quo, but reintroduces some manual work
- **Option B**: Automated scraping of institutional web portals (employer portal, insurer member portal)
  - *Trade-off*: True automation, highest value, but highest technical risk and fragility to portal changes
- **Option C**: Hybrid approach (API where available, manual input as fallback, scraping for specific high-value sources)
  - *Trade-off*: Balanced risk, allows progressive enhancement

**Recommendation**: Start with Option C (hybrid), beginning with manual input to validate detection logic, then progressively automate high-value sources.
**Decision owner**: Implementation team (can be revisited per Epic 2)

### 2. Notification delivery mechanism
**Options**:
- **Option A**: In-app only (requires user to open application)
  - *Trade-off*: Simplest, maintains local-first purity, but violates "no vigilance" constraint if user doesn't open app
- **Option B**: Email notifications sent from local DXOS client
  - *Trade-off*: Meets notification requirement, requires email config, creates local→cloud dependency
- **Option C**: Push notifications via DXOS-compatible notification service
  - *Trade-off*: Best UX, highest technical complexity, need to validate DXOS support

**Recommendation**: Option B (email) as minimum viable notification; Option C (push) as stretch goal if DXOS patterns exist.
**Decision owner**: After DXOS architecture spike (Epic 1)

### 3. Multi-user (family) data architecture
**Options**:
- **Option A**: Single DXOS space with all family members' data, single-user access control
  - *Trade-off*: Simplest, assumes single administrator, may not scale to independent teen/adult dependents
- **Option B**: Per-person DXOS spaces with explicit sharing/delegation model
  - *Trade-off*: More flexible, respects future autonomy, significantly more complex
- **Option C**: Hierarchical: primary user space contains dependent data with future migration path
  - *Trade-off*: Pragmatic for initial family use, documents migration plan for later independence

**Recommendation**: Option C (hierarchical with documented future path).
**Decision owner**: Implementation team during Epic 1 architecture work

### 4. Discrepancy detection algorithm approach
**Options**:
- **Option A**: Rule-based consistency checks (e.g., "employer shows active" AND "insurer shows inactive" = discrepancy)
  - *Trade-off*: Transparent, debuggable, requires manual rule definition, may miss subtle patterns
- **Option B**: ML-based anomaly detection trained on historical status patterns
  - *Trade-off*: Could detect subtle drift, but violates "justified confidence" constraint (black box), overkill for problem
- **Option C**: Rule-based with confidence scoring based on data freshness and source agreement
  - *Trade-off*: Transparent like Option A, adds nuance for "uncertain" states, maintains justifiability

**Recommendation**: Option C (rule-based with confidence scoring).
**Decision owner**: Implementation team during Epic 3

### 5. What constitutes "active coverage" across sources
**Question**: Different institutions use different terminology ("active," "enrolled," "eligible," "effective"). How do we normalize?
**Options**:
- **Option A**: Strict mapping table (insurer="active" → internal="active"; insurer="pending" → internal="inactive")
  - *Trade-off*: Clear, may misrepresent institutional nuance
- **Option B**: Preserve institutional terminology with human-readable explanations
  - *Trade-off*: Accurate, but requires user to reconcile differences
- **Option C**: Multi-dimensional status (enrollment status, payment status, effective date status) with synthesis view
  - *Trade-off*: Most accurate, most complex

**Recommendation**: Option C for internal model, with simplified Option B presentation ("Insurer says X, Employer says Y").
**Decision owner**: Implementation team during Epic 2 schema design

---

## Chosen approach, workstreams & responsibilities

### Implementation approach

This system will be built as a **local-first DXOS application** with four primary layers:

1. **Data persistence layer** (DXOS): Stores institutional data snapshots, user configuration, detection history, and evidence chains. Must review DXOS documentation on Spaces, Echo schema patterns, and data replication before implementation.

2. **Integration layer**: Fetches current status from institutional sources. Initial implementation uses manual input with structured schema, creating abstraction for future automation. Each source adapter produces timestamped, immutable snapshots.

3. **Detection engine**: Compares snapshots across sources and over time using rule-based logic with confidence scoring. Produces alerts when discrepancies exceed threshold. Must be transparent and debuggable (no black boxes).

4. **User interface layer**: DXOS-compatible UI (likely React-based given DXOS patterns) displaying current status, historical trends, discrepancy alerts, and evidence artifacts. Must be comprehensible to non-technical users.

This approach fits the Proposal by:
- Respecting local-first architecture as learning constraint (DXOS)
- Enabling 1-day detection through periodic automated checks
- Maintaining transparency (rule-based detection, explicit confidence)
- Supporting non-technical use (clear status displays, plain-language alerts)
- Preserving evidence for institutional follow-up (immutable snapshots)

### Why not alternative approaches

**Cloud-first SaaS**: Would violate local-first learning goal and data ownership constraint. Rejected.

**Mobile-native app**: DXOS compatibility unclear; would split effort between native mobile and DXOS patterns. Could be future work after DXOS learning is complete.

**Browser extension only**: Insufficient for multi-source monitoring and notification; doesn't meet "no vigilance" constraint. Could be complementary data source later.

**Blockchain/Web3**: Overkill for single-user problem; doesn't address institutional data access; violates learning focus (would become crypto learning vs. GenAI + local-first learning).

### Workstreams

#### 1. DXOS foundation & local-first architecture
Establish DXOS development environment, design data schema using Echo, implement Space management for user and family member data, and establish patterns for local-first application structure.

**Key artifacts to review before implementation**:
- DXOS documentation: Getting Started, Echo data schema patterns, Space and Identity management
- DXOS example applications (if available): examine data modeling patterns, UI integration, state management
- Must review: DXOS notification/background task patterns (if they exist)

**Coherent slices**:
- Development environment setup and DXOS CLI familiarity
- Core schema design (User, InsuranceSource, CoverageSnapshot, DiscrepancyAlert)
- Space initialization and family member model
- Basic CRUD operations on local data

#### 2. Data ingestion & institutional source integration
Build adapters for institutional data sources, starting with manual input forms, with architecture supporting future automation. Each adapter produces normalized, timestamped snapshots stored immutably in DXOS.

**Key patterns to establish**:
- Source adapter interface (must be consistent for manual vs. automated)
- Snapshot versioning and immutability
- Schema for institutional terminology mapping
- Data validation and confidence tagging

**Coherent slices**:
- Manual input UI for employer-provided coverage status
- Manual input UI for insurer member portal status
- Snapshot storage and retrieval from DXOS
- (Stretch) Automated scraper for one high-value source as proof-of-concept

#### 3. Discrepancy detection engine
Implement rule-based consistency checking across sources and over time. Produce alerts when discrepancies meet confidence threshold. Maintain full audit trail of detection logic.

**Key patterns to establish**:
- Rule definition format (must be human-readable)
- Confidence scoring based on data freshness, source agreement, historical patterns
- Alert generation and priority assignment
- Evidence artifact generation (what data led to this alert?)

**Coherent slices**:
- Core detection rules (cross-source consistency)
- Confidence scoring algorithm
- Alert generation and persistence
- Evidence artifact generation (shareable with institutions)

#### 4. User interface & evidence presentation
Build DXOS-compatible UI showing current status, discrepancy alerts, and supporting evidence. Must be accessible to non-technical users.

**Key patterns to establish**:
- Status dashboard (current state per person, per source)
- Alert presentation (clear, actionable, not alarming)
- Evidence view (exportable, timestamped, understandable)
- Historical trend visualization (when did discrepancy start?)

**Coherent slices**:
- Dashboard layout and status card components
- Alert inbox and detail views
- Evidence artifact viewer (timeline, source comparison)
- (Stretch) Export evidence as PDF for institutional communication

#### 5. Notification & alerting system
Implement notification delivery to meet "no vigilance" constraint. Starting point is email; stretch goal is push notifications if DXOS patterns support it.

**Key patterns to establish**:
- Notification trigger logic (when to send vs. when to suppress)
- Email composition from DXOS client
- Notification history and user preferences
- (Stretch) Push notification integration

**Coherent slices**:
- Notification trigger logic
- Email delivery from local client
- User notification preferences
- Notification audit log

---

## Risks, assumptions, and mitigations

### Risks

1. **DXOS maturity and capability gaps**
   - *What could go wrong*: DXOS may lack necessary features (background tasks, notifications, data export), causing architectural rework or forcing compromises on local-first constraints.
   - *Mitigation*: Early spike (Epic 1, de-risking action #1) to validate DXOS capabilities before deep implementation. Document workarounds and gaps as learning artifacts.
   - *Detection*: If DXOS spike takes >2 weeks or reveals blocking limitations, escalate architectural decision.

2. **Institutional data access automation infeasibility**
   - *What could go wrong*: Employer/insurer portals may use anti-scraping protections, require 2FA on every login, or have no accessible data export, making automation impractical.
   - *Mitigation*: Hybrid approach with manual input as fallback. Design adapter interface to support multiple acquisition methods. Early spike on one real portal (de-risking action #2).
   - *Detection*: If portal access spike shows >4 hour manual effort per scraper OR high brittleness, shift permanently to manual input with better UX.

3. **False positive alerts erode trust**
   - *What could go wrong*: Detection rules trigger on benign differences (e.g., timing delays between systems), causing alert fatigue and user disengagement.
   - *Mitigation*: Conservative confidence thresholds initially (prefer false negatives to false positives during learning phase). Implement alert feedback loop. Start with real family data for ground truth.
   - *Detection*: If >30% of alerts are false positives in first month of real use, tighten detection rules.

4. **Non-technical usability failure**
   - *What could go wrong*: UI or terminology is too technical; users don't understand what action to take when alert fires; system increases anxiety rather than confidence.
   - *Mitigation*: User test with non-technical family members before considering "done." Plain language throughout. Include "what does this mean?" explainers. Validation criteria include comprehension check.
   - *Detection*: If test user cannot explain alert meaning or next action within 30 seconds, UI revision required.

5. **1-day detection window missed**
   - *What could go wrong*: Data source checks run too infrequently, or processing lag causes alerts to fire >24 hours after actual discrepancy.
   - *Mitigation*: Design for 6-hour check intervals (4x daily) to ensure 24-hour detection. Add explicit latency monitoring. Make check frequency configurable.
   - *Detection*: If any real discrepancy takes >24 hours from occurrence to alert, this is a critical failure requiring immediate fix.

6. **Local-first constraints make system unusable**
   - *What could go wrong*: Requirement to run local DXOS node becomes friction point; users forget to run it; notifications fail because local client is offline.
   - *Mitigation*: This is partially accepted as learning constraint. Document friction as learning artifact. Consider always-on deployment option (Raspberry Pi, NAS) as workaround while maintaining local-first control.
   - *Detection*: If real-world usage shows <80% uptime, either improve always-on deployment or document as DXOS learning limitation.

### Assumptions

1. **Necessary information is accessible**: Assumes employer and insurer portals surface active/inactive status in human-readable form (even if not API-accessible).
   - *Validation approach*: Manual audit of real portals during Epic 2 design.

2. **DXOS is sufficiently mature**: Assumes DXOS can support persistent data, basic UI rendering, and local execution without blocking bugs.
   - *Validation approach*: Epic 1 spike answers this before major work begins.

3. **GenAI can accelerate end-to-end development**: Assumes GenAI tools (including this planning process) meaningfully reduce time and cognitive load vs. manual development.
   - *Validation approach*: This is a meta-learning goal; tracked through reflection artifacts, not metrics.

4. **Family scale is sufficient for learning**: Assumes 2-5 covered individuals is enough to validate data model and detection logic without needing multi-tenant architecture.
   - *Validation approach*: Real usage with actual family for 3+ months.

5. **Discrepancies are detectable through status snapshots**: Assumes institutions represent coverage state in observable fields, not just in backend databases.
   - *Validation approach*: Document what fields are actually available during Epic 2; adjust detection scope if needed.

---

## Detail the work: Dependencies & sequencing

### Epic 1: DXOS foundation & local-first architecture
**Dependencies**: None (foundational work)

**Tasks**:

1. **Environment setup & DXOS installation**
   - Install DXOS CLI and development dependencies
   - Initialize DXOS project structure
   - Verify local node can start and persist data
   - Document setup steps for reproducibility

2. **DXOS capability spike**
   - Survey DXOS documentation for: data schemas (Echo), Space management, Identity, UI integration patterns, background tasks/scheduling (if supported), notification mechanisms
   - Build proof-of-concept: create Space, define simple schema, persist data, retrieve data, render in UI
   - Document findings: what's supported, what's missing, what's unclear
   - Decision gate: Is DXOS viable for this project?

3. **Core schema design**
   - Design Echo schema for: User (family members), InsuranceSource (employer, insurer, COBRA), CoverageSnapshot (timestamped status records), DiscrepancyAlert (detected issues), EvidenceArtifact (supporting data for alerts)
   - Define relationships: User has many CoverageSnapshots per InsuranceSource
   - Implement schema in DXOS Echo format
   - Validate with test data

4. **Space and family member model**
   - Implement Space initialization for primary user
   - Design family member data structure (hierarchical vs. separate Spaces—see Unknowns #3)
   - Implement CRUD operations for family members
   - Test: Add/edit/remove family member records

5. **Basic UI scaffolding**
   - Set up React (or DXOS-recommended UI framework)
   - Implement DXOS data binding to UI components
   - Create navigation structure (Dashboard, Sources, Alerts, Settings)
   - Verify data flow: DXOS → UI rendering

**Testing approach**:
- **Unit tests**: Schema validation, CRUD operations
- **Integration tests**: DXOS data persistence and retrieval across sessions
- **Manual testing**: Can create Space, add family member, see data persist after restart

**Validation criteria**:
- DXOS development environment is operational and documented
- Core schemas are implemented and can store/retrieve test data
- Basic UI can render DXOS data for at least one family member
- Architecture decision on family member model is documented (hierarchical vs. separate Spaces)
- DXOS capability gaps (if any) are documented as constraints for later epics

---

### Epic 2: Data ingestion & institutional source integration
**Dependencies**: Epic 1 (requires schema and UI foundation)

**Tasks**:

1. **Source adapter interface design**
   - Define adapter interface: `getSnapshot(sourceId) → CoverageSnapshot`
   - Design for multiple implementations (manual, scraper, API future)
   - Define normalized snapshot format (independent of source-specific terminology)
   - Document institutional terminology mapping strategy (see Unknowns #5)

2. **Institutional source configuration**
   - Create UI for defining InsuranceSource records (employer, insurer, COBRA admin)
   - Fields: source name, source type, URL (for future automation), last checked timestamp
   - Implement source CRUD operations
   - Test: Add employer source, add insurer source

3. **Manual input adapter - employer status**
   - Build form for employer coverage status input
   - Fields: effective date, status (active/inactive/pending), dependent statuses, notes
   - Generate timestamped CoverageSnapshot on submission
   - Store snapshot immutably in DXOS
   - Test: Submit employer status, verify snapshot persists

4. **Manual input adapter - insurer status**
   - Build form for insurer portal status input
   - Fields: policy number, effective dates, covered individuals, status per person, notes
   - Generate timestamped CoverageSnapshot on submission
   - Store snapshot immutably in DXOS
   - Test: Submit insurer status, verify snapshot persists

5. **Manual input adapter - COBRA status (if applicable)**
   - Build form for COBRA administrator status input
   - Fields: coverage period, payment status, covered individuals, notes
   - Generate timestamped CoverageSnapshot on submission
   - Test: Submit COBRA status, verify snapshot persists

6. **Snapshot history and retrieval**
   - Implement query: get all snapshots for source X
   - Implement query: get most recent snapshot per source
   - Implement UI: view snapshot history timeline
   - Test: Submit multiple snapshots over time, verify timeline display

7. **(Stretch) Automated scraper proof-of-concept**
   - Select one high-value institutional portal for scraping experiment
   - Build scraper (headless browser or HTTP client)
   - Parse status fields and generate CoverageSnapshot
   - Document scraper brittleness, auth challenges, feasibility
   - Decision gate: Is automation viable for this source?

**Testing approach**:
- **Unit tests**: Snapshot generation, schema validation, adapter interface compliance
- **Integration tests**: End-to-end flow (input form → snapshot creation → DXOS storage → UI display)
- **Manual testing**: Real portal data entry (use actual family insurance data)

**Validation criteria**:
- At least two institutional sources (employer, insurer) can be configured
- Manual input adapters capture coverage status and generate snapshots
- Snapshots are stored immutably with timestamps
- Snapshot history is queryable and displayable
- Source adapter interface is documented and supports future automation (even if not implemented)
- (If scraper attempted) Automation feasibility is documented with specific findings

---

### Epic 3: Discrepancy detection engine
**Dependencies**: Epic 2 (requires snapshot data)

**Tasks**:

1. **Detection rule definition format**
   - Design human-readable rule syntax (e.g., "IF employer.status = active AND insurer.status = inactive THEN alert")
   - Implement rule parser/evaluator
   - Create initial rule set for cross-source consistency
   - Document rule logic for transparency (must be auditable)

2. **Core consistency rules**
   - Rule: Employer says active, insurer says inactive (or vice versa)
   - Rule: Coverage lapse detected (status changed from active to inactive without user action)
   - Rule: Dependent coverage mismatch (employer shows N dependents, insurer shows M dependents, N ≠ M)
   - Rule: Effective date inconsistency (employer and insurer report different effective dates)
   - Test each rule with synthetic snapshot data

3. **Confidence scoring algorithm**
   - Implement scoring based on: data freshness (recent snapshot = higher confidence), source agreement (2+ sources agree = higher confidence), historical stability (consistent over time = higher confidence)
   - Define confidence threshold for alert generation (e.g., >70% confidence triggers alert)
   - Test: Vary snapshot timestamps and source agreement, verify confidence scores

4. **Alert generation and persistence**
   - When rule fires above confidence threshold, generate DiscrepancyAlert
   - Alert fields: trigger rule, confidence score, affected person, affected sources, timestamp, status (new/acknowledged/resolved)
   - Store alert in DXOS
   - Test: Trigger detection with discrepant snapshots, verify alert created

5. **Evidence artifact generation**
   - For each alert, bundle supporting evidence: snapshot data from each source, confidence breakdown, rule that triggered
   - Store as EvidenceArtifact linked to alert
   - Test: Generate alert, verify evidence artifact contains necessary data

6. **Detection scheduling**
   - Implement periodic detection runs (every 6 hours to meet 24-hour requirement)
   - Design: DXOS background task if supported, or scheduled script invoking detection logic
   - Log detection runs and results
   - Test: Run detection multiple times, verify alerts don't duplicate

7. **Alert resolution workflow**
   - User action: acknowledge alert (seen but not resolved)
   - User action: mark resolved (institutional issue fixed)
   - Auto-resolution: If subsequent snapshots show consistency, suggest alert is resolved
   - Test: Mark alert resolved, verify status updates

**Testing approach**:
- **Unit tests**: Individual detection rules, confidence scoring, alert generation
- **Integration tests**: End-to-end detection (snapshots → rule evaluation → alert creation → evidence generation)
- **Scenario tests**: Create specific snapshot sequences (e.g., active→inactive→active) and verify correct alerts

**Validation criteria**:
- Detection rules are documented in human-readable format
- Core consistency rules (cross-source, lapse, dependent mismatch) are implemented and tested
- Confidence scoring produces differentiated scores based on data quality
- Alerts are generated and persisted when confidence exceeds threshold
- Evidence artifacts contain sufficient data to support institutional follow-up
- Detection runs every ≤6 hours (sufficient to meet 24-hour requirement)
- False positive rate on test scenarios is <20%

---

### Epic 4: User interface & evidence presentation
**Dependencies**: Epic 3 (requires alerts and evidence artifacts)

**Tasks**:

1. **Dashboard layout and status cards**
   - Design dashboard showing current status per family member
   - Status card per person: name, most recent snapshot per source, overall status (consistent/discrepancy/unknown)
   - Color coding: green (consistent), yellow (low-confidence discrepancy), red (high-confidence discrepancy)
   - Test: Display dashboard with test data

2. **Alert inbox and priority sorting**
   - Alerts page: list all active alerts, sorted by confidence score
   - Alert card: affected person, summary, confidence, timestamp
   - Filter options: by person, by source, by status
   - Test: Display multiple alerts, verify sorting and filtering

3. **Alert detail view**
   - Clicking alert shows full detail: rule that triggered, confidence breakdown, affected sources, snapshot data comparison
   - Action buttons: acknowledge, mark resolved
   - Link to evidence artifact
   - Test: Open alert, verify all data displays correctly

4. **Evidence artifact viewer**
   - Timeline view: show snapshot history for involved sources
   - Comparison view: side-by-side display of discrepant fields
   - Plain-language explanation: "Employer reported active on [date], but insurer reported inactive on [date]"
   - Test: View evidence for alert, verify comprehensibility

5. **Historical trend visualization**
   - Graph: coverage status over time per source
   - Highlight discrepancy periods
   - Purpose: help user understand when problem started
   - Test: Display 30 days of snapshot data, verify timeline accuracy

6. **Settings and configuration UI**
   - User preferences: notification settings, check frequency, confidence threshold
   - Family member management: add/edit/remove
   - Source management: add/edit/remove institutional sources
   - Test: Change settings, verify they persist

7. **(Stretch) Export evidence as PDF**
   - Generate PDF from evidence artifact
   - Formatted for sharing with HR or insurer (professional appearance)
   - Test: Export evidence, verify PDF is readable and complete

**Testing approach**:
- **Unit tests**: Component rendering with mock data
- **Integration tests**: UI interactions (click alert → see detail → mark resolved → alert disappears from inbox)
- **Usability tests**: Non-technical family member can understand dashboard and alerts without explanation

**Validation criteria**:
- Dashboard clearly shows current coverage status per person
- Alerts are sorted by priority (confidence score) and filterable
- Alert detail view contains sufficient information to understand the discrepancy
- Evidence viewer presents comparison in plain language
- Historical timeline helps user identify when discrepancy began
- Non-technical user can explain what an alert means and what action to take (validated through user test)

---

### Epic 5: Notification & alerting system
**Dependencies**: Epic 3 (requires alert generation), Epic 4 (requires evidence presentation for notification content)

**Tasks**:

1. **Notification trigger logic**
   - When new alert is generated with confidence >threshold, trigger notification
   - Suppress duplicate notifications (don't re-notify for same alert)
   - Implement quiet hours (don't notify between 10pm-7am)
   - Test: Generate alert, verify notification triggered once

2. **Email delivery from local client**
   - Configure SMTP from DXOS client (user provides email credentials in settings)
   - Email template: clear subject ("Insurance Coverage Alert"), plain-language body, link to open app
   - Include summary: affected person, source discrepancy, action needed
   - Test: Trigger alert, verify email received

3. **User notification preferences**
   - Settings UI: enable/disable email notifications, email address, quiet hours, confidence threshold for notification
   - Test: Disable notifications, verify no email sent; re-enable, verify email sent

4. **Notification history and audit log**
   - Log all notifications sent: timestamp, alert ID, recipient, delivery status
   - UI: view notification history
   - Test: Send notification, verify log entry

5. **(Stretch) Push notification integration**
   - Research DXOS-compatible push notification service
   - If available: implement push notification delivery
   - Test: Trigger alert, verify push notification received on mobile device
   - Document: If DXOS doesn't support this, log as limitation

**Testing approach**:
- **Unit tests**: Trigger logic, notification suppression rules
- **Integration tests**: End-to-end flow (alert generated → notification triggered → email sent → user receives)
- **Manual tests**: Real email delivery with actual SMTP credentials

**Validation criteria**:
- New high-confidence alerts trigger email notifications
- Notification content is clear and actionable (includes affected person, discrepancy summary, link to app)
- Duplicate notifications are suppressed
- User can configure notification preferences (enable/disable, quiet hours)
- Notification delivery is logged for audit purposes
- (If push implemented) Push notifications work on at least one mobile platform
- (If push not implemented) Limitation is documented with reasons

---

### Epic 6: End-to-end testing & validation with real data
**Dependencies**: All previous epics (requires complete system)

**Tasks**:

1. **Real family data ingestion**
   - Input actual employer coverage status for all family members
   - Input actual insurer portal status for all family members
   - Verify snapshots are stored correctly
   - Test: Compare stored data to actual portal screenshots

2. **Baseline consistency check**
   - Run detection engine on initial real data
   - Verify no false positives (real data should be consistent initially)
   - If alerts generated: investigate whether real discrepancy or false positive
   - Test: Baseline detection should show "all systems consistent"

3. **Simulated discrepancy injection**
   - Manually create discrepant snapshot (e.g., change employer status to inactive while insurer shows active)
   - Run detection, verify alert generated
   - Verify evidence artifact accurately represents discrepancy
   - Test: Alert should fire with high confidence

4. **Notification delivery validation**
   - Trigger alert from simulated discrepancy
   - Verify email notification received
   - Verify notification content is accurate and actionable
   - Test: Family member without context can understand email

5. **Non-technical user testing**
   - Recruit non-technical family member as test user
   - Task 1: "Tell me what your current coverage status is"
   - Task 2: "There's an alert—what does it mean and what should you do?"
   - Task 3: "Show me evidence to send to HR"
   - Record: time to complete, comprehension, confusion points
   - Test: User completes tasks without assistance in <5 minutes

6. **24-hour detection validation**
   - Configure detection to run every 6 hours
   - Inject discrepancy at T=0
   - Verify alert generated by T=6 hours
   - Test: Detection SLA is met

7. **Multi-day stability test**
   - Run system continuously for 7 days with real data
   - Log: detection runs, alerts generated, false positive rate, system uptime
   - Identify: any crashes, data corruption, missed detections
   - Test: System remains stable over extended period

8. **Retrospective and learning capture**
   - Document: What worked well? What was difficult?
   - GenAI experience: Where did AI help? Where did it hinder?
   - DXOS experience: Where did local-first help? Where did it constrain?
   - Answer Proposal's success questions (see Proposal section 7)

**Testing approach**:
- **Real-world scenario testing**: Using actual family insurance data
- **Usability testing**: Non-technical family members as participants
- **Stability testing**: Multi-day continuous operation
- **Retrospective**: Qualitative learning capture

**Validation criteria**:
- Real family data is ingested and stored correctly
- Detection engine runs without false positives on baseline real data
- Simulated discrepancies generate alerts with correct confidence levels
- Notifications are delivered reliably and are comprehensible
- Non-technical users can complete core tasks (understand status, interpret alerts, access evidence) without assistance
- 24-hour detection requirement is met in practice (discrepancy → alert within 24 hours)
- System runs stably for 7 consecutive days
- Retrospective artifacts document learning outcomes per Proposal's success criteria

---

## Early de-risking actions

The following actions should be completed within the first 2 weeks to reduce the biggest risks:

### 1. DXOS proof-of-concept (3-5 days)
**Goal**: Validate that DXOS is mature enough to support this project before significant investment.

**Tasks**:
- Install DXOS CLI and initialize project
- Create simple schema (User, CoverageSnapshot)
- Persist data, retrieve data, verify it survives restart
- Build minimal UI that renders DXOS data
- Research: background tasks, notification patterns, data export options
- Document: what works, what doesn't, what's uncertain

**Success criteria**: Can store and retrieve structured insurance data using DXOS; basic UI renders data; architectural gaps (if any) are documented.

**If this fails**: Consider architectural pivot (e.g., SQLite with local-first UI, still respecting data ownership) OR accept DXOS limitations and adjust scope.

---

### 2. Institutional portal access spike (2-3 days)
**Goal**: Understand feasibility of automated data acquisition from real employer/insurer portals.

**Tasks**:
- Manually log in to real employer portal and insurer member portal
- Document: what data is visible, how is it structured, what terminology is used
- Attempt basic scraping (headless browser, inspect network requests)
- Document: auth complexity (2FA?), anti-bot protections, data format (HTML, JSON, PDF)
- Estimate: effort to build and maintain scraper for each source

**Success criteria**: Clear assessment of automation feasibility per source; fallback to manual input if automation is impractical.

**If this reveals high difficulty**: Shift to manual input with optimized UX (quick form, mobile-friendly) rather than over-investing in fragile scrapers.

---

### 3. Detection algorithm prototype (2-3 days)
**Goal**: Validate that discrepancy detection logic is sound before building full engine.

**Tasks**:
- Create synthetic dataset: 5 family members × 2 sources × 10 timestamps = 100 snapshots
- Inject discrepancies: cross-source mismatch, status lapse, dependent count mismatch
- Implement basic rule-based detection (no confidence scoring yet)
- Run detection, verify alerts generated for injected discrepancies
- Measure: false positive rate, false negative rate

**Success criteria**: Detection prototype identifies injected discrepancies with <20% false positive rate; logic is understandable and debuggable.

**If this reveals issues**: Refine rule logic before building full engine; consider simpler detection initially (e.g., cross-source only, no temporal analysis).

---

### 4. Non-technical comprehension test (1 day)
**Goal**: Ensure core concepts (coverage status, discrepancy, alert, evidence) are understandable before building full UI.

**Tasks**:
- Create paper mockups or low-fidelity wireframes of dashboard and alert views
- Show to non-technical family member
- Ask: "What does this screen tell you?" "What would you do if you saw this alert?"
- Document: confusion points, terminology issues, suggested changes

**Success criteria**: Non-technical person can explain status and alerts without help; identified terminology/UX issues inform Epic 4 design.

**If this reveals confusion**: Iterate on plain-language explanations and visual design before implementing UI.

---

### 5. End-to-end critical path prototype (3-4 days)
**Goal**: Prove the full flow (data input → storage → detection → alert → notification) works before building polish.

**Tasks**:
- Minimal implementation: manual input form → DXOS snapshot → simple detection rule → console log alert → email sent
- Use real family data for one person, one source
- Inject discrepancy, verify email received
- Measure: time from discrepancy injection to notification delivery

**Success criteria**: End-to-end flow works; notification delivered; timing meets 24-hour requirement with headroom.

**If this reveals issues**: Identify and fix bottlenecks before scaling to full feature set.

---

## Readiness checklist

Before converting this Plan into work items (Jira/Linear/etc.), confirm:

### Detail completeness
- [ ] Are epics broken down into specific, independently understandable tasks?
- [ ] Are dependencies between epics and tasks clearly identified?
- [ ] Are validation criteria defined for each epic?
- [ ] Are testing approaches specified (unit/integration/E2E)?

### Constraint adherence
- [ ] Are the 10 constraints from the Proposal reflected in epic/task design?
- [ ] Are non-negotiables (1-day detection, non-technical usability, local-first ownership) explicitly validated?
- [ ] Are guardrails clear enough to be referenced during code review?

### Decision clarity
- [ ] Are the 5 unknowns requiring decision listed with options and trade-offs?
- [ ] Are decision owners identified?
- [ ] Are decision points tied to specific epics/tasks where they must be resolved?

### Risk mitigation
- [ ] Are the top 6 risks documented with mitigations?
- [ ] Are early de-risking actions (5 spikes) scheduled before dependent work?
- [ ] Are detection mechanisms in place for each risk?

### Learning focus
- [ ] Is learning captured as an explicit outcome (not just delivery)?
- [ ] Are GenAI and DXOS experiences documented as meta-goals?
- [ ] Are retrospectives and reflection built into Epic 6?

### Handoff readiness
- [ ] Can a new team read this Plan and understand what to build?
- [ ] Are references to existing code/docs included (or noted as "must review")?
- [ ] Is the Proposal linked for full context?

---

## Questions for the team

Before finalizing this Plan and creating tickets:

1. **DXOS familiarity**: Does the team have prior DXOS experience, or is this a learning journey for everyone? (This affects Epic 1 timeline.)

2. **Manual input vs. automation priority**: Should we commit to building at least one automated scraper in the initial delivery, or is manual input sufficient for learning goals?

3. **Non-technical user testing**: Who is available as a non-technical test user? (Need to schedule usability testing for Epic 4 and Epic 6.)

4. **Notification delivery**: Is email-from-local-client acceptable, or is push notification a requirement? (This affects Epic 5 scope.)

5. **Timeline expectations**: This Plan intentionally omits time estimates per your guidance. When do we need to have a deployable system for real family use? (This informs prioritization and stretch goals.)

6. **Ticket creation tool**: Where will this be tracked? (Jira, Linear, GitHub Projects, other?) This affects ticket formatting and integration.

---

## Plan status

This Plan is **ready for review and approval**.

Once approved, epics and tasks can be converted into tickets in the planning tool, with constraints and validation criteria referenced in ticket acceptance criteria.

**Next steps**:
1. Review this Plan with the team
2. Answer the 6 questions above
3. Resolve the 5 unknowns requiring decision (or assign decision points)
4. Create Epic 1 tickets and begin DXOS proof-of-concept (de-risking action #1)
5. After de-risking actions complete, create remaining epic tickets

---

*This Plan implements the Proposal documented in [docs/planning/proposal.md](./proposal.md).*

*End of Plan*
