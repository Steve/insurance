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

The following decisions have been resolved based on review:

### 1. Data acquisition method
**Decision**: **Option B - Automated scraping of institutional web portals**

Automated scraping is the chosen approach, with extensive architectural support for maintainability, testability, and user confidence.

**Implementation requirements**:

1. **Portal change detection and notification**: When a scraper detects that a portal's structure has changed (scraping fails or extracts unexpected data), treat this as an assurance problem and notify the user, similar to coverage discrepancy alerts. This ensures users know when monitoring may be compromised.

2. **Independent scraper components**: Each institutional scraper (e.g., Sana Benefits scraper, UnitedHealthcare scraper) must be architecturally independent and runnable as a standalone component, separate from the larger system. This supports:
   - Independent development and testing
   - Selective deployment (only run scrapers for user's specific institutions)
   - Easier maintenance when portal changes occur

3. **TDD structure for scrapers**: Develop test-driven development scaffolding for scrapers that can:
   - Test against captured portal data (HTML snapshots, API responses)
   - Validate extraction logic without live portal access
   - Detect breaking changes when portal structure changes
   - Support regression testing as scrapers evolve

4. **Manual scraper execution scaffold**: Build tooling to manually trigger a scraper against a live portal and observe:
   - What data is being extracted
   - How extraction logic is interpreting portal structure
   - Where extraction may be failing or producing unexpected results
   - This supports debugging and gives users confidence in what's happening

5. **Headed scraping option**: Consider architecture supporting "headed" browser automation where the user can optionally watch (in a visible browser window) as the system impersonates them to log in and scrape data. This increases:
   - **Technical confidence**: User sees exactly what the system is doing
   - **Security confidence**: User observes credential usage and can verify no unexpected actions
   - **Privacy confidence**: User can audit what data is being accessed
   - Implementation note: Default to headless (background) operation, but support headed mode for transparency

**Decision rationale**: Automated scraping delivers the highest value (zero vigilance tax) and fits the learning-first goal. The extensive scaffolding requirements mitigate scraper fragility risk and maintain user trust through transparency.

---

### 2. Notification delivery mechanism
**Decision**: **Option B (email) as minimum viable notification; Option C (push notifications) as stretch goal if DXOS patterns exist**

Email notifications sent from the local DXOS client meet the "no vigilance" constraint. Push notifications remain a stretch goal pending DXOS capability investigation.

**Decision rationale**: Email is reliable, universally accessible, and achievable within local-first constraints. Push notifications would improve UX but require DXOS-compatible infrastructure.

---

### 3. Multi-user (family) data architecture
**Decision**: **Option C - Hierarchical with documented future migration path**

Primary user Space contains family member data. Document architecture to support future migration where adult dependents could gain independent Space control.

**Decision rationale**: Pragmatic for initial family use while respecting future autonomy needs. Avoids premature complexity of per-person Spaces.

---

### 4. Discrepancy detection algorithm approach
**Decision**: **Option C - Rule-based with confidence scoring**

Detection uses transparent, rule-based logic with explicit confidence scoring.

**Additional requirement**: **Rules visibility for users** - The system must provide current or future visibility into detection rules so users can understand why alerts fired. This could be:
- In-app rule viewer showing active detection logic
- Per-alert explanation of which rule triggered and why
- Documentation of rule logic in plain language

**Decision rationale**: Maintains transparency and justified confidence constraints. Users must be able to audit the system's reasoning. ML-based approaches would violate this requirement.

---

### 5. What constitutes "active coverage" across sources
**Decision**: **Option C for internal model, with simplified Option B presentation**

Internally, model multi-dimensional status (enrollment status, payment status, effective date status). Present to users with institutional terminology preserved: "Insurer says X, Employer says Y."

**Decision rationale**: Most accurate internal representation while presenting information in user-understandable format tied to institutional sources.

---

## Chosen approach, workstreams & responsibilities

### Implementation approach

This system will be built as a **local-first DXOS application** with four primary layers, using a **loosely-coupled, tuple-space-inspired architecture** for service composition.

#### Architectural pattern: Linda-like tuple space with DXOS

To achieve loose coupling between system components (especially external service integrations like scraping and email), we adopt a **Linda-like tuple space pattern** using DXOS shared spaces:

**Pattern overview**:
- Multiple independent DXOS instances run concurrently, all accessing the same shared DXOS Space
- Each instance provides specific capabilities (scraping, email notification, detection, UI)
- Instances watch the Space for tuple-like data structures representing work requests (e.g., "scrape source X", "send email notification Y")
- When an instance detects a request matching its capabilities, it **atomically grabs** the tuple, executes the function, stores results in the Space, and writes a completion tuple
- This decouples components: the system requesting scraping doesn't need to know how scrapers work or manage their lifecycle

**Example flow - Scraping**:
1. Detection scheduler writes tuple: `{type: "SCRAPE_REQUEST", sourceId: "sana-benefits", timestamp: ...}`
2. Scraping instance (running independently) detects this tuple and atomically claims it
3. Scraper executes, extracts coverage data from Sana portal
4. Scraper writes result tuple: `{type: "SCRAPE_COMPLETE", sourceId: "sana-benefits", snapshot: {...}, status: "success"}`
5. Detection engine detects completion tuple, processes new snapshot

**Example flow - Email notification**:
1. Detection engine generates alert, writes tuple: `{type: "NOTIFICATION_REQUEST", alertId: "123", channel: "email", ...}`
2. Email instance detects tuple, atomically claims it
3. Email sender composes and sends email
4. Writes completion: `{type: "NOTIFICATION_COMPLETE", alertId: "123", status: "sent", timestamp: ...}`

**Benefits of this architecture**:
- **Loose coupling**: Scraper development/maintenance is independent of detection logic
- **Independent deployment**: Can run scraper instance on different schedule, machine, or not at all
- **Testability**: Each capability instance can be tested in isolation
- **Transparency**: Tuple history in DXOS provides full audit trail of system actions
- **Resilience**: If scraper instance fails, request tuples remain in Space; retry logic is explicit

**Must review before implementation**: DXOS documentation on atomic operations, Space querying patterns, and multi-instance coordination.

---

#### Layer descriptions

**1. Data persistence layer (DXOS)**

Stores institutional data snapshots, user configuration, detection history, evidence chains, and work request/completion tuples. All system state lives in DXOS Space(s).

**Must review**: DXOS documentation on Spaces, Echo schema patterns, atomic operations, and data replication.

---

**2. Integration layer (revised architecture)**

The integration layer consists of **independent capability instances** that respond to work requests in the DXOS Space.

**Scraper capability instances**:
- Each institutional scraper (Sana, UnitedHealthcare, employer-specific portals) is an independent component
- Scraper watches for `SCRAPE_REQUEST` tuples for its source type
- Executes scraping (headless or headed mode), generates timestamped CoverageSnapshot
- Writes snapshot to DXOS, writes `SCRAPE_COMPLETE` tuple

**Scraper architecture requirements** (per Unknown #1):
- **Independent execution**: Each scraper runs as standalone process/service
- **TDD scaffolding**: Test scrapers against captured portal HTML/JSON snapshots
- **Manual execution mode**: CLI/UI to trigger scraper manually and observe extraction
- **Headed mode support**: Optional visible browser for user transparency
- **Portal change detection**: If scraper fails or extracts unexpected structure, write `PORTAL_CHANGE_ALERT` tuple (treated like coverage discrepancy)

**Email notification capability instance**:
- Watches for `NOTIFICATION_REQUEST` tuples
- Composes and sends email from local SMTP configuration
- Writes `NOTIFICATION_COMPLETE` tuple with delivery status

**Third-party component considerations**:
- **For notification management**: Consider using existing libraries (e.g., Nodemailer for email, third-party notification services for push)
  - *Constraint impact*: If using external notification service (e.g., Firebase Cloud Messaging), this introduces cloud dependency and may compromise "local-first data ownership" (Constraint #7) for notification metadata (but not core coverage data)
  - *Trade-off*: Evaluate whether notification routing metadata constitutes sensitive data requiring local-first protection
  - *Recommendation*: Start with local SMTP for email (fully local-first); if push notifications require cloud service, document as acceptable cloud dependency for non-sensitive operational data

**General pattern for external service integration**:
- Any integration with external services (scraping, email, future API calls) should follow this tuple-based pattern
- Isolates external dependencies into independent capability instances
- Maintains core system independence from external service availability

---

**3. Detection engine**

Compares snapshots across sources and over time using rule-based logic with confidence scoring. Produces alerts when discrepancies exceed threshold. Must be transparent and debuggable (no black boxes).

**Third-party component considerations**:
- **For rules engine**: Consider using existing rules engines (e.g., json-rules-engine, node-rules, or similar)
  - *Constraint impact*: Third-party rules engines typically don't compromise local-first constraints (run locally), but must evaluate:
    - **Transparency** (Constraint #2, #6): Can users understand rules? Can rules be exported/displayed in human-readable format?
    - **Justified confidence** (Constraint #6): Does the engine support confidence scoring or only binary true/false?
  - *Trade-off*: Existing rules engine may accelerate development but may require wrapper layer to expose rules for user visibility (per Unknown #4 requirement)
  - *Recommendation*: Evaluate json-rules-engine or similar; if it supports human-readable rule export and can be extended with confidence scoring, adopt it; otherwise, build custom rule evaluator with transparency as first-class requirement

Detection engine watches for `SCRAPE_COMPLETE` tuples, processes new snapshots, and writes `ALERT` tuples when discrepancies detected.

---

**4. User interface layer (minimal viable approach)**

DXOS-compatible UI (likely React-based given DXOS patterns) displaying **minimal set of information** with option to expand later.

**Minimal viable UI components** (prioritize these):
- **Current status view**: Show per-person, per-source status (green/yellow/red) with most recent snapshot timestamp
- **Active alerts list**: Display unresolved alerts with confidence level and basic description
- **Alert detail**: Show which rule triggered, confidence breakdown, affected sources, timestamp
- **Evidence viewer**: Display snapshot comparison for user to share with institutions

**Deferred to future expansion**:
- Historical trend graphs (can start with simple timeline, defer interactive visualizations)
- Advanced filtering and search across alerts
- Customizable dashboards
- PDF export (start with copy-paste text, defer styled PDF generation)

**Design principle**: Start with minimum information needed to answer "Is my coverage okay?" and "What's wrong?" Everything else can be added progressively based on real usage feedback.

---

### Why not alternative approaches

**Cloud-first SaaS**: Would violate local-first learning goal and data ownership constraint. Rejected.

**Mobile-native app**: DXOS compatibility unclear; would split effort between native mobile and DXOS patterns. Could be future work after DXOS learning is complete.

**Browser extension only**: Insufficient for multi-source monitoring and notification; doesn't meet "no vigilance" constraint. Could be complementary data source later.

**Blockchain/Web3**: Overkill for single-user problem; doesn't address institutional data access; violates learning focus (would become crypto learning vs. GenAI + local-first learning).

---

### Workstreams

#### 1. DXOS foundation & local-first architecture
Establish DXOS development environment, design data schema using Echo (including tuple structures for work requests), implement Space management for user and family member data, and establish patterns for local-first tuple-space architecture.

**Key artifacts to review before implementation**:
- DXOS documentation: Getting Started, Echo data schema patterns, Space and Identity management
- DXOS atomic operations and multi-instance coordination patterns
- DXOS example applications (if available): examine data modeling patterns, UI integration, state management
- Must review: DXOS notification/background task patterns (if they exist)

**Coherent slices**:
- Development environment setup and DXOS CLI familiarity
- Core schema design (User, InsuranceSource, CoverageSnapshot, DiscrepancyAlert, plus tuple schemas: ScrapeRequest, ScrapeComplete, NotificationRequest, etc.)
- Space initialization and family member model
- Basic CRUD operations on local data
- Tuple pattern prototype (write request, read/claim, write completion)

---

#### 2. Data ingestion & institutional source integration (scraper-first)
Build independent scraper capability instances for institutional data sources, following architectural requirements from Unknown #1. Each scraper produces normalized, timestamped snapshots stored immutably in DXOS.

**Key patterns to establish**:
- Scraper independence (each source as standalone component)
- TDD scaffolding for scraper testing against captured portal data
- Manual execution scaffold for scraper observation
- Headed/headless browser automation modes
- Portal change detection and alerting
- Tuple-based work request pattern (ScrapeRequest → scraper execution → ScrapeComplete)
- Snapshot versioning and immutability
- Schema for institutional terminology mapping

**Coherent slices**:
- Scraper framework and tuple-based integration pattern
- First scraper: Sana Benefits (or user's primary insurer)
  - TDD test suite with captured portal HTML
  - Manual execution CLI tool
  - Headed mode support for user observation
  - Portal change detection logic
- Second scraper: Employer portal (or secondary institutional source)
  - Reuse framework established with first scraper
  - Document portal-specific authentication and structure
- Snapshot storage and retrieval from DXOS
- Scraper scheduling (writes ScrapeRequest tuples on defined intervals)

---

#### 3. Discrepancy detection engine
Implement rule-based consistency checking across sources and over time. Produce alerts when discrepancies meet confidence threshold. Maintain full audit trail of detection logic.

**Third-party component evaluation**:
- Evaluate json-rules-engine or similar for rule evaluation
- If adopted, implement wrapper for confidence scoring and user-visible rule export
- If transparency requirements can't be met with third-party, build custom evaluator

**Key patterns to establish**:
- Rule definition format (must be human-readable and user-visible per Unknown #4)
- Confidence scoring based on data freshness, source agreement, historical patterns
- Alert generation and priority assignment
- Evidence artifact generation (what data led to this alert?)
- Integration with tuple pattern (watches ScrapeComplete, writes Alert tuples)

**Coherent slices**:
- Rules engine selection and integration (or custom rule evaluator implementation)
- Core detection rules (cross-source consistency)
- Confidence scoring algorithm
- Alert generation and persistence
- Evidence artifact generation (shareable with institutions)
- Rules visibility UI (users can view active rules and understand logic)

---

#### 4. User interface & evidence presentation (minimal viable)
Build DXOS-compatible UI showing minimal set of information to answer "Is coverage okay?" and "What's wrong?"

**Key patterns to establish**:
- Status dashboard (current state per person, per source) - minimal design
- Alert presentation (clear, actionable, not alarming)
- Evidence view (exportable as text, timestamped, understandable)
- Settings for scraper configuration and notification preferences

**Coherent slices** (prioritized for minimal viable):
- Dashboard layout with status cards (per person, per source, color-coded)
- Alert inbox (list view with confidence and timestamp)
- Alert detail view (rule that triggered, confidence breakdown, affected sources)
- Evidence artifact viewer (snapshot comparison in plain text or simple table)
- Settings page (family members, institutional sources, scraper schedule, notification preferences)

**Deferred slices** (expand later based on usage):
- Historical trend visualization (graphs over time)
- PDF export for evidence artifacts
- Advanced filtering and search
- Customizable notification rules

---

#### 5. Notification & alerting system (email capability instance)
Implement email notification capability as independent instance following tuple pattern. Meets "no vigilance" constraint.

**Third-party component evaluation**:
- Use Nodemailer or similar for email sending (local SMTP, no cloud dependency)
- If push notifications pursued (stretch goal), evaluate constraint impact of Firebase Cloud Messaging or similar (introduces cloud dependency for notification routing metadata)

**Key patterns to establish**:
- Notification capability instance (watches NotificationRequest tuples)
- Email composition from alert data
- SMTP configuration and credential management
- Notification history (NotificationComplete tuples)
- User preferences for notification triggers and quiet hours

**Coherent slices**:
- Notification capability instance framework
- Email delivery using Nodemailer (or similar local SMTP library)
- Email template (plain-language alert summary with link to app)
- User notification preferences UI
- Notification audit log (view past notifications)
- (Stretch) Push notification capability instance (pending DXOS patterns and constraint evaluation)

---

## Third-party components: constraint trade-offs

**Philosophy**: Prefer existing third-party components where they fit, rather than building from scratch. However, evaluate constraint impacts before adoption.

### Evaluation framework

For each third-party component being considered, assess impact on constraints:

1. **Local-first data ownership** (Constraint #7): Does it require sending data to cloud services?
2. **Transparency** (Constraint #2, #9): Can users understand what it's doing?
3. **Justified confidence** (Constraint #6): Does it support explainability?
4. **Non-technical usability** (Constraint #5): Does it add complexity visible to users?

### Specific component evaluations

#### Rules engine (e.g., json-rules-engine)
- **Constraint impact**: Low - runs locally, no data leaves system
- **Transparency**: Must verify rules can be exported in human-readable format
- **Recommendation**: **Adopt if transparency requirement met**; significantly accelerates detection engine development
- **Mitigation**: Build wrapper layer for confidence scoring and user-visible rule display

#### Notification management (e.g., Nodemailer for email)
- **Constraint impact**: Low for email (local SMTP), Medium for push (cloud routing metadata)
- **Transparency**: Email sending is transparent; push may introduce opacity
- **Recommendation**: **Adopt Nodemailer for email** (no constraint violations); **evaluate push carefully** (document cloud dependency as acceptable for operational metadata if pursued)

#### Web scraping frameworks (e.g., Puppeteer, Playwright)
- **Constraint impact**: Low - runs locally, browser automation doesn't compromise data ownership
- **Transparency**: Can support headed mode for user visibility
- **Recommendation**: **Adopt Puppeteer or Playwright** (mature, well-tested, supports headed/headless modes, accelerates scraper development)

#### UI component libraries (e.g., React, Material-UI, Chakra UI)
- **Constraint impact**: None - purely local rendering
- **Non-technical usability**: Can improve UX if used thoughtfully
- **Recommendation**: **Adopt React-based UI library** compatible with DXOS patterns; prioritize accessibility and plain-language presentation

### Constraint pivot considerations

If a third-party component provides significant value but conflicts with constraints, document the trade-off and consider whether constraint should be relaxed:

**Example**: If push notifications require cloud service (Firebase Cloud Messaging), document:
- **Constraint affected**: #7 (local-first data ownership)
- **Data leaving local system**: Notification routing metadata (device tokens, notification timestamps), NOT coverage data or personal health information
- **Risk**: Cloud provider sees notification patterns (frequency, timing), not content
- **Mitigation**: Encrypt notification content; cloud service only routes messages
- **Pivot decision**: Is notification routing metadata acceptable cloud dependency given UX benefit?

---

## Risks, assumptions, and mitigations

### Risks

1. **DXOS maturity and capability gaps**
   - *What could go wrong*: DXOS may lack necessary features (background tasks, atomic operations for tuple pattern, notifications, data export), causing architectural rework or forcing compromises on local-first constraints.
   - *Mitigation*: Early spike (Epic 1, de-risking action #1) to validate DXOS capabilities, especially atomic operations needed for tuple pattern, before deep implementation. Document workarounds and gaps as learning artifacts.
   - *Detection*: If DXOS spike takes >2 weeks or reveals blocking limitations for tuple pattern, escalate architectural decision (may need to simplify to single-instance architecture).

2. **Scraper fragility and portal changes**
   - *What could go wrong*: Institutional portals change structure frequently, breaking scrapers; anti-scraping protections block automation; 2FA requirements prevent unattended scraping.
   - *Mitigation*:
     - TDD structure with captured portal data allows rapid detection of breakage
     - Portal change detection alerts user when scraper fails (treats as assurance problem)
     - Independent scraper architecture isolates failures (one broken scraper doesn't break system)
     - Manual execution scaffold allows user to diagnose and report issues
     - Headed mode allows user to verify scraper behavior and maintain trust even when troubleshooting
   - *Detection*: If scrapers break >1x per month per source, this is expected; mitigation is fast repair cycle. If scraping becomes impossible (anti-bot), fall back to manual input (document as learning outcome).

3. **False positive alerts erode trust**
   - *What could go wrong*: Detection rules trigger on benign differences (e.g., timing delays between systems), causing alert fatigue and user disengagement.
   - *Mitigation*: Conservative confidence thresholds initially (prefer false negatives to false positives during learning phase). Implement alert feedback loop. Start with real family data for ground truth. Rules visibility helps users understand and trust logic.
   - *Detection*: If >30% of alerts are false positives in first month of real use, tighten detection rules.

4. **Non-technical usability failure**
   - *What could go wrong*: UI or terminology is too technical; users don't understand what action to take when alert fires; system increases anxiety rather than confidence.
   - *Mitigation*: User test with non-technical family members before considering "done." Plain language throughout. Include "what does this mean?" explainers. Validation criteria include comprehension check. Minimal UI approach reduces cognitive load.
   - *Detection*: If test user cannot explain alert meaning or next action within 30 seconds, UI revision required.

5. **1-day detection window missed**
   - *What could go wrong*: Scraper scheduling runs too infrequently, or processing lag causes alerts to fire >24 hours after actual discrepancy.
   - *Mitigation*: Design for 6-hour scrape intervals (4x daily) to ensure 24-hour detection. Add explicit latency monitoring. Make scrape frequency configurable. Tuple pattern provides audit trail of request → execution → completion timing.
   - *Detection*: If any real discrepancy takes >24 hours from occurrence to alert, this is a critical failure requiring immediate fix.

6. **Local-first constraints make system unusable**
   - *What could go wrong*: Requirement to run local DXOS node becomes friction point; users forget to run it; scraping fails because local client is offline.
   - *Mitigation*: This is partially accepted as learning constraint. Document friction as learning artifact. Consider always-on deployment option (Raspberry Pi, NAS, always-on laptop) as workaround while maintaining local-first control. Tuple pattern's asynchronous nature helps: scrape requests queue when offline, execute when back online.
   - *Detection*: If real-world usage shows <80% uptime, either improve always-on deployment or document as DXOS learning limitation.

7. **Tuple pattern coordination complexity**
   - *What could go wrong*: Multiple instances competing for same tuple cause race conditions; tuple cleanup becomes complex; debugging distributed system is harder than monolithic.
   - *Mitigation*: DXOS atomic operations should prevent race conditions (validate in spike). Tuple history provides audit trail for debugging. Start with single instance of each capability type (one scraper instance per source, one email instance) to reduce coordination complexity. Document tuple lifecycle (request → claim → complete → archive).
   - *Detection*: If tuple pattern causes bugs that take >1 day to diagnose, consider simplifying to monolithic architecture (acceptable trade-off against learning goal).

### Assumptions

1. **Necessary information is accessible via scraping**: Assumes employer and insurer portals surface active/inactive status in scrapable form (HTML, JSON, or visible browser state).
   - *Validation approach*: Early portal access spike (de-risking action #2) tests real scrapers against real portals.

2. **DXOS supports atomic operations for tuple pattern**: Assumes DXOS Echo can support atomic claim operations needed for Linda-like coordination.
   - *Validation approach*: Epic 1 spike validates atomic operation support; if unavailable, simplify architecture.

3. **GenAI can accelerate end-to-end development**: Assumes GenAI tools (including this planning process) meaningfully reduce time and cognitive load vs. manual development.
   - *Validation approach*: This is a meta-learning goal; tracked through reflection artifacts, not metrics.

4. **Family scale is sufficient for learning**: Assumes 2-5 covered individuals is enough to validate data model and detection logic without needing multi-tenant architecture.
   - *Validation approach*: Real usage with actual family for 3+ months.

5. **Headless scraping is sufficient, headed mode is transparency feature**: Assumes scrapers can operate unattended (headless) for routine monitoring; headed mode is for user confidence and debugging, not required for every scrape.
   - *Validation approach*: Build headless first, add headed mode as transparency feature; validate with real usage.

---

## Detail the work: Dependencies & sequencing

### Epic 1: DXOS foundation & tuple-space architecture
**Dependencies**: None (foundational work)

**Tasks**:

1. **Environment setup & DXOS installation**
   - Install DXOS CLI and development dependencies
   - Initialize DXOS project structure
   - Verify local node can start and persist data
   - Document setup steps for reproducibility

2. **DXOS capability spike (expanded)**
   - Survey DXOS documentation for: data schemas (Echo), Space management, Identity, **atomic operations** (critical for tuple pattern), UI integration patterns, background tasks/scheduling (if supported), notification mechanisms
   - Build proof-of-concept: create Space, define simple schema, persist data, retrieve data, render in UI
   - **Test tuple pattern**: Write request tuple, have second process atomically claim it, write completion tuple, verify no race conditions
   - Document findings: what's supported, what's missing, what's unclear (especially atomic operation support)
   - Decision gate: Is DXOS viable for this project? Does it support tuple pattern?

3. **Core schema design (including tuple schemas)**
   - Design Echo schema for core entities: User (family members), InsuranceSource (employer, insurer, COBRA), CoverageSnapshot (timestamped status records), DiscrepancyAlert (detected issues), EvidenceArtifact (supporting data for alerts)
   - Design tuple schemas: ScrapeRequest, ScrapeComplete, NotificationRequest, NotificationComplete, PortalChangeAlert
   - Define relationships: User has many CoverageSnapshots per InsuranceSource; tuples reference entities by ID
   - Implement schemas in DXOS Echo format
   - Validate with test data

4. **Space and family member model**
   - Implement Space initialization for primary user
   - Design family member data structure (hierarchical - per Unknown #3)
   - Implement CRUD operations for family members
   - Test: Add/edit/remove family member records

5. **Basic UI scaffolding (minimal)**
   - Set up React (or DXOS-recommended UI framework)
   - Implement DXOS data binding to UI components
   - Create minimal navigation structure (Dashboard, Alerts, Settings)
   - Verify data flow: DXOS → UI rendering

6. **Tuple pattern prototype**
   - Implement utility functions: writeTuple, claimTuple (atomic), readCompletedTuples
   - Build simple proof-of-concept: one process writes work request, another claims and completes
   - Verify: tuples are claimed atomically (no double-processing), completion tuples are queryable
   - Document tuple lifecycle and cleanup strategy

**Testing approach**:
- **Unit tests**: Schema validation, CRUD operations, tuple lifecycle functions
- **Integration tests**: DXOS data persistence and retrieval across sessions, tuple pattern with multiple processes
- **Manual testing**: Can create Space, add family member, see data persist after restart; two processes can coordinate via tuples

**Validation criteria**:
- DXOS development environment is operational and documented
- Core schemas (entities + tuples) are implemented and can store/retrieve test data
- Basic UI can render DXOS data for at least one family member
- Tuple pattern is validated: atomic claim works, completion tuples are queryable, no race conditions observed
- Architecture decision on family member model is documented (hierarchical)
- DXOS capability gaps (if any) are documented as constraints for later epics
- If tuple pattern is not viable, simplified architecture approach is documented

---

### Epic 2: Scraper capability instances & institutional source integration
**Dependencies**: Epic 1 (requires schema, tuple pattern, and DXOS foundation)

**Tasks**:

1. **Scraper framework design**
   - Define scraper interface: responds to ScrapeRequest tuple, writes ScrapeComplete tuple
   - Design scraper base class/module with common functionality:
     - Tuple watching and atomic claiming
     - Headless/headed mode support
     - Portal change detection (structure validation)
     - Error handling and retry logic
     - Snapshot generation
   - Select browser automation library: Puppeteer or Playwright (evaluate both)
   - Document scraper architecture and independence requirements

2. **TDD scaffolding for scrapers**
   - Build test framework: load captured portal HTML/JSON, run scraper extraction logic, validate output
   - Create fixture storage: directory structure for portal snapshots (versioned by date)
   - Implement portal snapshot capture tool: manually save portal HTML for testing
   - Write example test: scraper extracts correct fields from captured Sana Benefits portal HTML
   - Document TDD workflow: capture portal → write test → implement scraper → validate

3. **Manual scraper execution scaffold**
   - Build CLI tool: `scraper-run --source=sana --mode=headed --observe`
   - Features:
     - Triggers single scraper execution (not via tuple, direct invocation)
     - Headed mode shows browser window (user watches automation)
     - Logs extraction steps (navigating to page X, extracting field Y)
     - Displays extracted data for user verification
   - Test: Run tool against real portal, verify user can observe and understand process

4. **First scraper: Sana Benefits (or primary insurer)**
   - Implement Sana scraper:
     - Authentication flow (username/password, handle 2FA if present)
     - Navigate to coverage overview page
     - Extract: covered individuals, status per person, effective dates, plan details
     - Generate CoverageSnapshot with extracted data
     - Detect portal changes: validate expected DOM structure, flag if missing
   - Write TDD test suite with captured Sana HTML fixtures
   - Test with manual execution scaffold (headed mode)
   - Integrate with tuple pattern: watches ScrapeRequest{sourceId: "sana"}, writes ScrapeComplete
   - Document: authentication requirements, extraction logic, known fragilities

5. **Second scraper: Employer portal (or secondary institutional source)**
   - Implement employer portal scraper (reuse framework from Sana scraper)
   - Adapt to employer-specific portal structure
   - Write TDD test suite with employer portal fixtures
   - Test with manual execution scaffold
   - Integrate with tuple pattern
   - Document: authentication requirements, extraction logic

6. **Institutional source configuration UI**
   - Create UI for defining InsuranceSource records
   - Fields: source name, source type, scraper type (sana, employer, etc.), credentials (encrypted), URL, scrape schedule
   - Implement source CRUD operations
   - Test: Add Sana source with credentials, add employer source

7. **Scrape scheduler**
   - Implement scheduling logic: every 6 hours (configurable), write ScrapeRequest tuple for each configured source
   - Can be simple cron-like scheduler or DXOS background task if supported
   - Test: Scheduler writes ScrapeRequest tuples at correct intervals
   - Verify: Scraper instances pick up requests and execute

8. **Portal change detection and alerting**
   - When scraper detects portal structure change (extraction fails, unexpected DOM), write PortalChangeAlert tuple
   - UI displays portal change alerts like coverage discrepancy alerts
   - Alert includes: source name, what failed, last successful scrape, suggested action ("Portal structure changed, scraper needs update")
   - Test: Manually break scraper test fixture (remove expected DOM element), verify alert generated

9. **Snapshot history and retrieval**
   - Implement query: get all snapshots for source X
   - Implement query: get most recent snapshot per source
   - Implement UI: view snapshot history timeline (minimal - simple list with timestamps)
   - Test: Run scrapers multiple times, verify snapshot history displays

**Testing approach**:
- **Unit tests**: Scraper extraction logic with TDD fixtures, tuple integration, snapshot generation
- **Integration tests**: End-to-end scraping flow (ScrapeRequest → scraper execution → snapshot stored → ScrapeComplete tuple)
- **Manual testing**: Use manual execution scaffold with headed mode on real portals, verify extraction correctness
- **Portal change tests**: Modify fixtures to simulate portal changes, verify detection and alerting

**Validation criteria**:
- Scraper framework supports headless/headed modes, TDD testing, and tuple-based integration
- At least two institutional scrapers (Sana/insurer, employer) are implemented and working
- TDD test suites exist for each scraper with captured portal fixtures
- Manual execution scaffold allows user to observe scraper in headed mode
- Scrape scheduler writes ScrapeRequest tuples on defined intervals
- Scrapers respond to tuples, execute, and write ScrapeComplete tuples
- Portal change detection generates alerts when scraper fails
- Snapshots are stored immutably with timestamps and queryable
- User can configure institutional sources (including credentials) via UI

---

### Epic 3: Discrepancy detection engine
**Dependencies**: Epic 2 (requires snapshot data from scrapers)

**Tasks**:

1. **Rules engine evaluation and selection**
   - Evaluate json-rules-engine: can it export rules in human-readable format? Can it be extended with confidence scoring?
   - Evaluate alternatives if json-rules-engine doesn't meet transparency requirements
   - Decision: adopt third-party rules engine OR build custom evaluator
   - If adopting third-party: implement wrapper for confidence scoring and user-visible rule export
   - Document decision and rationale

2. **Detection rule definition format**
   - Design human-readable rule syntax
     - If using json-rules-engine: use its JSON format, document how to translate to plain language
     - If custom: design format like "IF employer.status = 'active' AND insurer.status = 'inactive' THEN alert with confidence based on data freshness"
   - Implement rule parser/evaluator (or wrapper around third-party engine)
   - Create initial rule set for cross-source consistency
   - Document rule logic for transparency (must be auditable by users)

3. **Core consistency rules**
   - Rule: Cross-source status mismatch (employer says active, insurer says inactive, or vice versa)
   - Rule: Coverage lapse detected (status changed from active to inactive without user action)
   - Rule: Dependent coverage mismatch (employer shows N dependents, insurer shows M dependents, N ≠ M)
   - Rule: Effective date inconsistency (employer and insurer report different effective dates)
   - Rule: Payment status discrepancy (if COBRA: payment recorded but coverage shows inactive)
   - Test each rule with synthetic snapshot data

4. **Confidence scoring algorithm**
   - Implement scoring based on:
     - Data freshness (recent snapshot = higher confidence)
     - Source agreement (2+ sources agree = higher confidence)
     - Historical stability (consistent over time = higher confidence)
     - Anomaly significance (large discrepancy = higher confidence than small)
   - Define confidence threshold for alert generation (e.g., >70% confidence triggers alert)
   - Test: Vary snapshot timestamps and source agreement, verify confidence scores differentiate scenarios

5. **Alert generation and persistence**
   - Detection engine watches for ScrapeComplete tuples (new snapshots available)
   - When rule fires above confidence threshold, generate DiscrepancyAlert
   - Alert fields: trigger rule, confidence score, affected person, affected sources, timestamp, status (new/acknowledged/resolved)
   - Store alert in DXOS (as regular entity, not tuple)
   - Test: Trigger detection with discrepant snapshots, verify alert created with correct confidence

6. **Evidence artifact generation**
   - For each alert, bundle supporting evidence:
     - Snapshot data from each source (affected fields)
     - Confidence breakdown (why this confidence score?)
     - Rule that triggered (plain-language explanation)
     - Timeline (when did discrepancy first appear?)
   - Store as EvidenceArtifact linked to alert
   - Test: Generate alert, verify evidence artifact contains necessary data for user to share with HR/insurer

7. **Detection scheduling**
   - Detection engine runs after each ScrapeComplete tuple processed
   - Can also run on schedule (every 6 hours) to catch any missed tuples
   - Log detection runs and results
   - Test: Run detection multiple times with same snapshots, verify alerts don't duplicate

8. **Alert resolution workflow**
   - User action: acknowledge alert (seen but not resolved)
   - User action: mark resolved (institutional issue fixed)
   - Auto-resolution: If subsequent snapshots show consistency (discrepancy cleared), suggest alert is resolved (user confirms)
   - Test: Mark alert resolved, verify status updates; inject consistent snapshots after discrepancy, verify auto-resolution suggestion

9. **Rules visibility UI (per Unknown #4)**
   - Settings page: "View Detection Rules"
   - Display active rules in plain language
     - Example: "Alert if employer shows coverage active but insurer shows inactive (confidence threshold: 70%)"
   - Per-alert: show which rule triggered and why (link from alert detail to rule explanation)
   - Test: User can view rules, understand logic, and trace alert back to triggering rule

**Testing approach**:
- **Unit tests**: Individual detection rules, confidence scoring, alert generation, evidence artifact creation
- **Integration tests**: End-to-end detection (snapshots → rule evaluation → alert creation → evidence generation)
- **Scenario tests**: Create specific snapshot sequences (e.g., active→inactive→active) and verify correct alerts with correct confidence
- **Rules visibility test**: User can explain why alert fired based on rules UI

**Validation criteria**:
- Detection rules are documented in human-readable format and visible to users
- Core consistency rules (cross-source, lapse, dependent mismatch, dates, payment) are implemented and tested
- Confidence scoring produces differentiated scores based on data quality factors
- Alerts are generated and persisted when confidence exceeds threshold
- Evidence artifacts contain sufficient data to support institutional follow-up (plain language, timestamps, source comparison)
- Detection runs after each scrape completion (real-time detection)
- False positive rate on test scenarios is <20%
- Users can view active detection rules and understand logic via UI

---

### Epic 4: User interface & evidence presentation (minimal viable)
**Dependencies**: Epic 3 (requires alerts and evidence artifacts)

**Tasks**:

1. **Dashboard layout and status cards (minimal)**
   - Design minimal dashboard: one status card per family member
   - Status card shows:
     - Family member name
     - Overall status: green (all sources consistent), yellow (low-confidence discrepancy), red (high-confidence discrepancy)
     - Most recent snapshot timestamp per source (Employer: checked 2 hours ago, Insurer: checked 3 hours ago)
   - Click card → expand to show per-source status
   - Test: Display dashboard with test data for 3 family members

2. **Alert inbox (minimal)**
   - Alerts page: list all unresolved alerts
   - Sort by confidence score (highest first)
   - Alert card shows: affected person, one-line summary, confidence %, timestamp
   - Click alert → navigate to detail view
   - Test: Display 5 test alerts, verify sorting and navigation

3. **Alert detail view**
   - Show full alert details:
     - Affected family member
     - Which sources have discrepancy
     - Plain-language explanation: "Employer reported coverage active on [date], but insurer reported coverage inactive on [date]"
     - Confidence score with breakdown (why this score?)
     - Link to rule that triggered (links to rules visibility UI)
   - Action buttons: "Acknowledge" (dismiss from inbox but keep record), "Mark Resolved" (discrepancy fixed)
   - Link to evidence artifact view
   - Test: Open alert, verify all data displays correctly, click actions and verify state changes

4. **Evidence artifact viewer (minimal)**
   - Display snapshot comparison in simple format:
     - Table with columns: Source, Status, Effective Date, Dependents, Timestamp
     - Row per source involved in discrepancy
     - Highlight discrepant fields (different background color)
   - Plain-language summary at top: "These sources disagree about coverage status as of [date]"
   - "Copy as Text" button (copies formatted text for email/ticket)
   - Test: View evidence for alert, verify comprehensibility, test copy function

5. **Settings and configuration UI (minimal)**
   - Family members: list, add, edit, remove
   - Institutional sources: list, add (name, type, credentials), edit, remove
   - Scraper settings: schedule frequency (every 6 hours default, configurable)
   - Notification settings: email address, enable/disable, confidence threshold for notification
   - Test: Change settings, verify they persist and affect system behavior

6. **Simple timeline view (deferred: historical graphs)**
   - For each family member + source: show list of snapshots (timestamp, status)
   - Purpose: help user identify when discrepancy began ("It was consistent until 3 days ago")
   - Simple list format (not interactive graph yet)
   - Test: Display 30 days of snapshot data in list, verify chronological order

**Testing approach**:
- **Unit tests**: Component rendering with mock data
- **Integration tests**: UI interactions (click alert → see detail → mark resolved → alert disappears from inbox)
- **Usability tests**: Non-technical family member can understand dashboard and alerts without explanation

**Validation criteria**:
- Dashboard clearly shows current coverage status per person with minimal cognitive load
- Alerts are sorted by priority (confidence score) and easy to navigate
- Alert detail view contains sufficient information to understand the discrepancy in plain language
- Evidence viewer presents comparison in format suitable for sharing with institutions (copy as text works)
- Users can configure family members, sources, and notification preferences
- Non-technical user can explain what an alert means and what action to take (validated through user test)
- Deferred features (graphs, PDF export) are documented for future expansion

---

### Epic 5: Notification capability instance (email)
**Dependencies**: Epic 3 (requires alert generation), Epic 4 (requires evidence presentation for notification content)

**Tasks**:

1. **Notification capability instance framework**
   - Design notification instance: watches NotificationRequest tuples, atomically claims, executes, writes NotificationComplete
   - Implement tuple watching logic
   - Implement notification trigger logic: when alert generated with confidence >threshold, write NotificationRequest tuple
   - Test: Generate alert, verify NotificationRequest tuple written

2. **Email delivery using Nodemailer**
   - Install and configure Nodemailer (third-party library for email)
   - Implement SMTP configuration: user provides email credentials in settings (stored encrypted in DXOS)
   - Notification instance reads NotificationRequest, composes email, sends via SMTP
   - Email template:
     - Subject: "Insurance Coverage Alert: [Affected Person]"
     - Body: Plain-language summary (affected person, source discrepancy, confidence level, timestamp)
     - Link to open app and view alert detail
   - Test: Trigger alert, verify email received with correct content

3. **Notification trigger logic (expanded)**
   - Trigger email for:
     - High-confidence coverage discrepancy alerts (confidence >70%)
     - Portal change alerts (scraper failure)
   - Suppress duplicate notifications: don't re-notify for same alert
   - Implement quiet hours: don't notify between 10pm-7am (configurable)
   - Test: Generate multiple alerts, verify first triggers notification, subsequent don't (deduplication)

4. **User notification preferences**
   - Settings UI section: "Notifications"
   - Fields: enable/disable email notifications, email address, SMTP credentials, quiet hours (start/end), confidence threshold for notification
   - Test: Disable notifications, verify NotificationRequest tuples written but email not sent; re-enable, verify email sent

5. **Notification history and audit log**
   - NotificationComplete tuples serve as audit log
   - UI: "Notification History" page showing past notifications (timestamp, alert, recipient, delivery status)
   - Test: Send notifications, view history, verify all logged

6. **(Stretch) Push notification capability instance**
   - Research DXOS-compatible push notification service (e.g., Firebase Cloud Messaging, Pushover)
   - Evaluate constraint impact (per "Third-party components" section): cloud dependency for routing metadata
   - If acceptable and DXOS supports: implement push notification instance (watches NotificationRequest with channel="push")
   - Test: Trigger alert, verify push notification received on mobile device
   - Document: If DXOS doesn't support or constraint impact too high, log as limitation

**Testing approach**:
- **Unit tests**: Trigger logic, notification suppression rules, email composition
- **Integration tests**: End-to-end flow (alert generated → NotificationRequest tuple → email instance claims → email sent → NotificationComplete tuple)
- **Manual tests**: Real email delivery with actual SMTP credentials to multiple email providers (Gmail, Outlook, etc.)

**Validation criteria**:
- Notification capability instance responds to NotificationRequest tuples and sends emails
- Email content is clear and actionable (includes affected person, discrepancy summary, link to app)
- Duplicate notifications are suppressed (same alert doesn't trigger multiple emails)
- Quiet hours are respected (no emails between 10pm-7am)
- User can configure notification preferences (enable/disable, email address, SMTP credentials, quiet hours, confidence threshold)
- Notification delivery is logged for audit purposes (NotificationComplete tuples)
- (If push implemented) Push notifications work on at least one mobile platform
- (If push not implemented) Limitation is documented with reasons (constraint impact or DXOS limitations)

---

### Epic 6: End-to-end testing & validation with real data
**Dependencies**: All previous epics (requires complete system)

**Tasks**:

1. **Real family data ingestion via scrapers**
   - Configure institutional sources for real employer and insurer
   - Provide real credentials (encrypted in DXOS)
   - Trigger scrapes (manually via scaffold first, then via scheduler)
   - Verify snapshots are stored correctly
   - Test: Compare stored data to actual portal screenshots, verify extraction accuracy

2. **Baseline consistency check**
   - Run detection engine on initial real data
   - Verify no false positives (real data should be consistent initially, or if inconsistent, verify it's real discrepancy)
   - If alerts generated: investigate whether real discrepancy or false positive
   - Adjust detection rules if false positives found
   - Test: Baseline detection should show accurate state (green if consistent, alert if real discrepancy)

3. **Scraper reliability validation**
   - Run scrapers on schedule (every 6 hours) for 7 days
   - Log: successful scrapes, failures, portal change alerts
   - Measure: scraper success rate, time to detect and alert portal changes
   - Test: Scraping runs reliably without manual intervention

4. **Headed mode transparency test**
   - Use manual execution scaffold in headed mode (visible browser)
   - Have non-technical family member observe scraper logging in and extracting data
   - Questions: "Do you understand what the system is doing?" "Do you trust it with your credentials?"
   - Document: feedback on transparency and trust

5. **Simulated discrepancy injection**
   - Manually create discrepant snapshot (e.g., edit DXOS data to simulate employer=active, insurer=inactive)
   - Run detection, verify alert generated with appropriate confidence
   - Verify evidence artifact accurately represents discrepancy
   - Verify email notification sent
   - Test: Alert should fire with high confidence, notification delivered within minutes

6. **Notification delivery validation**
   - Verify email notification from simulated discrepancy
   - Check: subject line clear, body plain-language, link works, notification logged
   - Test: Family member without context receives email and can understand what action to take

7. **Non-technical user testing**
   - Recruit non-technical family member as test user
   - Task 1: "Tell me what your current coverage status is" (tests dashboard comprehension)
   - Task 2: "There's an alert—what does it mean and what should you do?" (tests alert comprehension)
   - Task 3: "Show me evidence to send to HR" (tests evidence viewer usability)
   - Task 4: "Look at the detection rules—which rule would trigger if employer and insurer disagree?" (tests rules visibility)
   - Record: time to complete, comprehension, confusion points
   - Test: User completes tasks without assistance in <5 minutes each

8. **24-hour detection validation**
   - Inject discrepancy at T=0 (via manual data edit or wait for real discrepancy)
   - Verify scraper runs within 6 hours (T=6 or earlier)
   - Verify detection runs after scrape
   - Verify alert generated by T=6 hours (worst case, next scheduled scrape)
   - Test: Detection SLA is met (discrepancy → alert within 24 hours, typically within 6 hours)

9. **Multi-day stability test**
   - Run complete system continuously for 7 days with real data
   - Monitor: scraper instances, detection engine, notification instance, UI
   - Log: scrape runs, detection runs, alerts generated, notifications sent, false positive rate, system uptime, tuple processing delays
   - Identify: any crashes, data corruption, missed detections, performance issues
   - Test: System remains stable over extended period without manual intervention

10. **Retrospective and learning capture**
    - Document: What worked well? What was difficult?
    - GenAI experience: Where did AI help? Where did it hinder? What surprised you?
    - DXOS experience: Where did local-first help? Where did it constrain? Was tuple pattern valuable or over-complex?
    - Scraper experience: How often did scrapers break? Was TDD/headed mode valuable? Did portal change detection work?
    - Answer Proposal's success questions (Proposal section 7):
      - How does local-first actually work in practice on DXOS?
      - Is DXOS sufficient for building a usable system for early adopters like me?
      - Can I perform effective end-to-end product work using GenAI?
      - How often does my manual review agree with the system's automated results?
    - Create retrospective document: lessons learned, recommendations for similar projects

**Testing approach**:
- **Real-world scenario testing**: Using actual family insurance data and credentials
- **Usability testing**: Non-technical family members as participants
- **Reliability testing**: Multi-day continuous operation without intervention
- **Transparency testing**: Headed mode observation, rules visibility
- **Retrospective**: Qualitative learning capture against Proposal's success criteria

**Validation criteria**:
- Real family data is ingested via automated scrapers and stored correctly
- Scrapers run reliably on schedule for 7 consecutive days (>80% success rate)
- Detection engine runs without false positives on baseline real data (or <10% false positive rate)
- Simulated discrepancies generate alerts with correct confidence levels
- Notifications are delivered reliably and are comprehensible to non-technical recipients
- Non-technical users can complete core tasks (understand status, interpret alerts, access evidence, understand rules) without assistance
- 24-hour detection requirement is met in practice (discrepancy → alert within 24 hours)
- System runs stably for 7 consecutive days (>80% uptime, no data corruption)
- Headed mode and rules visibility provide sufficient transparency for user trust
- Retrospective artifacts document learning outcomes per Proposal's success criteria, including specific findings about GenAI, DXOS, and local-first architecture

---

## Early de-risking actions

The following actions should be completed within the first 2 weeks to reduce the biggest risks:

### 1. DXOS proof-of-concept with tuple pattern validation (4-5 days)
**Goal**: Validate that DXOS is mature enough to support this project, especially the tuple-space coordination pattern, before significant investment.

**Tasks**:
- Install DXOS CLI and initialize project
- Create simple schema (User, CoverageSnapshot, plus ScrapeRequest/ScrapeComplete tuple schemas)
- Persist data, retrieve data, verify it survives restart
- **Critical: Test atomic operations** - Can DXOS support atomic claim of tuples by competing processes?
- Build minimal UI that renders DXOS data
- Prototype tuple coordination: Process A writes ScrapeRequest, Process B atomically claims it, executes mock scrape, writes ScrapeComplete
- Research: background tasks, notification patterns, data export options, multi-instance coordination
- Document: what works, what doesn't, what's uncertain (especially tuple pattern viability)

**Success criteria**: Can store and retrieve structured insurance data using DXOS; basic UI renders data; tuple pattern works (atomic claim prevents race conditions); architectural gaps (if any) are documented.

**If this fails**:
- If DXOS works but tuple pattern doesn't (no atomic operations): Simplify to monolithic architecture (single process handles scraping, detection, notification sequentially)
- If DXOS has blocking limitations: Consider architectural pivot (e.g., SQLite + local-first UI, still respecting data ownership) OR accept DXOS limitations and adjust scope

---

### 2. First scraper proof-of-concept with Puppeteer/Playwright (3-4 days)
**Goal**: Validate that automated scraping is viable for real institutional portals before building full scraper framework.

**Tasks**:
- Select browser automation library: test both Puppeteer and Playwright, choose one
- Manually log in to real primary insurer portal (e.g., Sana Benefits)
- Document: portal structure, authentication flow, what data is visible, anti-scraping protections
- Build minimal scraper: automate login, navigate to coverage page, extract status fields
- Test both headless (background) and headed (visible browser) modes
- Capture portal HTML snapshot for TDD testing
- Document: auth complexity (2FA?), extraction challenges, scraper fragility assessment

**Success criteria**: Can successfully scrape real portal in both headless and headed modes; extraction logic produces correct CoverageSnapshot; TDD test with captured HTML passes; challenges and fragilities are documented.

**If this reveals high difficulty**:
- If scraping is impossible (strong anti-bot, impossible 2FA): Pivot to manual input with optimized UX
- If scraping works but is highly fragile: Accept fragility, invest in portal change detection and fast repair cycle
- Document findings to inform Epic 2 scope

---

### 3. Detection algorithm prototype with real-world scenarios (2-3 days)
**Goal**: Validate that discrepancy detection logic produces accurate results before building full engine.

**Tasks**:
- Create synthetic dataset based on real insurance scenarios:
  - 3 family members × 2 sources (employer, insurer) × 10 timestamps = 60 snapshots
  - Inject realistic discrepancies: employer=active + insurer=inactive, coverage lapse (active→inactive), dependent mismatch, effective date inconsistency
  - Include edge cases: timing delays (employer updated, insurer not yet), partial information (one source missing data)
- Implement basic rule-based detection (no confidence scoring yet, just binary alert/no alert)
- Run detection on synthetic data, verify alerts generated for injected discrepancies
- Measure: false positive rate, false negative rate, does detection logic match intuition?
- Have non-technical person review results: "Does this alert make sense given the data?"

**Success criteria**: Detection prototype identifies injected discrepancies with <20% false positive rate; logic is understandable and matches non-technical person's intuition; edge cases are handled reasonably.

**If this reveals issues**:
- If false positive rate >20%: Refine rule logic, add confidence scoring earlier than planned
- If rules are too complex to explain: Simplify detection (e.g., cross-source only, no temporal analysis initially)
- Document findings to inform Epic 3 rule design

---

### 4. Non-technical comprehension test with low-fidelity mockups (1 day)
**Goal**: Ensure core concepts (coverage status, discrepancy, alert, evidence, rules) are understandable before building full UI.

**Tasks**:
- Create paper mockups or Figma wireframes of:
  - Dashboard (status cards per family member)
  - Alert inbox and detail view
  - Evidence viewer (snapshot comparison)
  - Rules visibility (list of detection rules)
- Show to non-technical family member (spouse, parent, friend)
- Ask:
  - "What does this screen tell you?"
  - "What would you do if you saw this alert?"
  - "Why do you think this alert was triggered?" (after showing rules)
- Document: confusion points, terminology issues, suggested changes

**Success criteria**: Non-technical person can explain status and alerts without help; identified terminology/UX issues inform Epic 4 design; person understands why alert fired when shown rules.

**If this reveals confusion**:
- Iterate on plain-language explanations (e.g., avoid terms like "snapshot," "tuple," "confidence score" without explanation)
- Simplify visual design (less information per screen)
- Improve rules visibility presentation (plain language, examples)
- Repeat test with revised mockups until comprehension achieved

---

### 5. End-to-end critical path prototype (3-4 days)
**Goal**: Prove the full flow (scrape → storage → detection → alert → notification) works before building full feature set.

**Tasks**:
- Minimal implementation connecting all pieces:
  - Scraper: use first scraper from de-risking action #2, triggered manually (not via tuple yet)
  - Storage: write CoverageSnapshot to DXOS
  - Detection: run simple detection rule from de-risking action #3
  - Alert: generate DiscrepancyAlert if discrepancy found
  - Notification: send email using Nodemailer (not via tuple, direct call)
- Use real family data for one person, one institutional source
- Inject discrepancy (manually edit DXOS data after scrape to simulate mismatch)
- Run detection, verify alert generated
- Verify email sent with alert details
- Measure: time from discrepancy injection to notification delivery

**Success criteria**: End-to-end flow works; notification delivered; timing meets 24-hour requirement with large headroom (should be minutes, not hours); can demonstrate working system end-to-end even if unpolished.

**If this reveals issues**:
- Identify and fix bottlenecks (e.g., DXOS queries slow, scraping takes too long, email delivery fails)
- Simplify architecture if coordination is too complex (e.g., defer tuple pattern to later)
- Adjust scope if any piece is fundamentally blocked
- Document findings to inform Epic sequencing and priorities

---

## Readiness checklist

Before converting this Plan into work items (Jira/Linear/etc.), confirm:

### Detail completeness
- [ ] Are epics broken down into specific, independently understandable tasks?
- [ ] Are dependencies between epics and tasks clearly identified?
- [ ] Are validation criteria defined for each epic?
- [ ] Are testing approaches specified (unit/integration/E2E)?
- [ ] Are architectural decisions (tuple pattern, minimal UI, scraper-first) reflected in tasks?

### Constraint adherence
- [ ] Are the 10 constraints from the Proposal reflected in epic/task design?
- [ ] Are non-negotiables (1-day detection, non-technical usability, local-first ownership, transparency) explicitly validated?
- [ ] Are guardrails clear enough to be referenced during code review?
- [ ] Are third-party component decisions evaluated against constraints?

### Decision clarity
- [ ] Are the 5 unknowns resolved with decisions documented?
- [ ] Are additional requirements from Unknown #1 (scraper architecture) incorporated into Epic 2?
- [ ] Are third-party component trade-offs documented (rules engine, email, browser automation)?

### Risk mitigation
- [ ] Are the 7 risks (including tuple pattern coordination, scraper fragility) documented with mitigations?
- [ ] Are early de-risking actions (5 spikes) scheduled before dependent work?
- [ ] Are detection mechanisms in place for each risk?
- [ ] Is scraper fragility accepted as expected condition with mitigation strategy?

### Learning focus
- [ ] Is learning captured as an explicit outcome (not just delivery)?
- [ ] Are GenAI, DXOS, and local-first experiences documented as meta-goals?
- [ ] Are retrospectives and reflection built into Epic 6?
- [ ] Is tuple pattern evaluated as learning experiment (can be simplified if over-complex)?

### Handoff readiness
- [ ] Can a new team read this Plan and understand what to build?
- [ ] Is the architectural pattern (tuple-space, scraper independence, minimal UI) clearly described?
- [ ] Are references to DXOS docs, third-party libraries, and evaluation criteria included?
- [ ] Is the Proposal linked for full context?

---

## Plan status

This Plan is **ready for implementation**.

Decisions from review are incorporated:
- **Data acquisition**: Automated scraping (Option B) with extensive architectural scaffolding
- **Notification**: Email (Option B), push as stretch (Option C)
- **Multi-user**: Hierarchical (Option C)
- **Detection**: Rule-based with confidence (Option C), rules visible to users
- **Status representation**: Multi-dimensional internal (Option C), institutional terminology external (Option B)
- **Architecture**: Linda-like tuple pattern for loose coupling, scraper independence, minimal UI
- **Third-party components**: Evaluated and recommended where appropriate (Puppeteer/Playwright, Nodemailer, json-rules-engine)

**Next steps**:
1. Begin Epic 1: DXOS foundation and tuple pattern proof-of-concept (de-risking action #1)
2. Parallel: Begin first scraper proof-of-concept (de-risking action #2)
3. After de-risking actions complete (week 2-3), create Epic 2-6 tickets in planning tool
4. Continuous: Document learning experiences (GenAI, DXOS, scrapers) as retrospective input

**Ticket creation**: Once de-risking actions validate architecture, convert epics and tasks into tickets with:
- Constraints and validation criteria as acceptance criteria
- Third-party component decisions and evaluations documented
- Links to this Plan and Proposal for context

---

*This Plan implements the Proposal documented in [docs/planning/proposal.md](./proposal.md) with decisions from review incorporated.*

*End of Plan*
