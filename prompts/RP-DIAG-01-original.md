# RP-DIAG-01 — Unified NinjaTrader Runtime Diagnostic & Acceptance AddOn

## 1. Role and Mission

Act as a Senior NinjaTrader 8 AddOn Engineer, C#/.NET Framework Engineer, NinjaTrader API Specialist, Market Data Systems Engineer, and Independent Software Auditor.

Your assignment is to inspect the supplied RiskPilot.BOT source archive and build an independent, read-only NinjaTrader 8 Diagnostic AddOn.

The AddOn must collect real runtime evidence from NinjaTrader, correlate it with available RiskPilot.BOT diagnostic information, and produce a comprehensive technical evidence package.

This is an engineering and validation tool, NOT a trading system.

The primary objective is to answer unresolved technical questions using actual NinjaTrader runtime data rather than assumptions, simulated behavior, or source-code inspection alone.

You do not have GitHub access. Work exclusively with the supplied source archive and the locally available NinjaTrader installation and assemblies.

Do not assume that the supplied archive represents the latest main branch. Identify and report its actual source revision using available Git metadata, commit files, manifests, or other reliable evidence. If the exact commit cannot be established, explicitly report it as unknown.

---

## 2. Project Context

Repository:

`Ai00120/RiskPilot.BOT`

Expected baseline: the source contains the merged changes through PR #275, which addressed a NinjaTrader compilation issue involving the unavailable `NinjaTrader.Cbi.Provider.Rithmic` member.

PR #275 replaced direct enum-member references with string-based Provider detection.

The merged PR must be independently inspected.

In particular, verify whether the runtime value returned by the relevant Provider objects can actually be identified as `Rithmic` using the new comparison logic.

Do not assume that PR #275 is behaviorally correct merely because a stub-based CI build passed.

Relevant project context:

- NinjaTrader 8.1.8.4 was used in the earlier test.
- NinjaTrader 8.1.9.0 is the intended next validation environment.
- .NET Framework 4.8 is the expected target framework; verify the actual installation rather than assuming it.
- Market data provider: Rithmic.
- Primary market: CME ES.
- Relevant contract in the supplied test recording: ES DEC26 / ES 12-26.
- Sim account used in the previous test: `SimRP.BOT`.
- Automated trading must remain disabled throughout all diagnostic operations.

The previous test archive may contain NinjaTrader logs, market recordings, DOM data, and order lifecycle evidence.

### Important distinction about the previous Sim test

The user manually submitted a test trade using NinjaTrader Chart Trader / ATM on a Sim account.

The recorded trade must be treated as a user-initiated manual Sim trade, NOT as a RiskPilot.BOT-generated trade.

The previous evidence indicates a Sell entry of quantity 2, followed by two quantity-1 target executions and a final Flat position.

RiskPilot's trading state was Disabled.

Do not attribute those executions to the bot. Do not infer that the bot submitted any orders.

Use the previous trade only as a real-world test case for validating NinjaTrader order, execution, position, and Account.Orders behavior.

---

## 3. Non-Negotiable Safety Constraints

These requirements override every other implementation objective.

### 3.1 Read-only operation

The AddOn must never:

- Submit, modify, cancel, or close orders.
- Create, modify, or close positions.
- Change account settings.
- Change connection settings.
- Connect or disconnect a broker or market data provider.
- Enable automated trading.
- Change RiskPilot strategy settings.
- Modify RiskPilot risk parameters.
- Modify D-078 approval decisions.
- Change instrument specifications or registry entries.
- Change account balances or PnL.
- Invoke any trading or broker mutation API.

Permitted operations:

- Read accessible NinjaTrader runtime properties.
- Enumerate accounts, connections, orders, executions, instruments, and positions where supported by documented APIs.
- Subscribe to documented read-only runtime events.
- Take read-only snapshots.
- Record timestamps and event sequences.
- Read accessible RiskPilot diagnostic interfaces.
- Write diagnostic files to a dedicated output directory.

The user will manually perform any required Sim trade through NinjaTrader Chart Trader or ATM.

### 3.2 Trading safety invariant

`Trading = Disabled`

This invariant must never be modified or bypassed.

The AddOn must not contain an order submission mechanism, execution engine, strategy activation mechanism, or hidden trading fallback.

### 3.3 Architectural isolation

The diagnostic tool must be independent of RiskPilot.BOT production trading logic.

Do not refactor, rewrite, or modify existing RiskPilot strategy, execution, portfolio, risk, or order-management code.

Do not change production behavior merely to make the diagnostic tool work.

Do not use private-field manipulation, unsafe memory access, or undocumented internal runtime manipulation.

Do not use reflection to bypass access restrictions or invoke hidden methods.

If required diagnostic information is not exposed through supported public APIs or existing diagnostic interfaces, report the limitation instead of inventing an unsafe workaround.

---

## 4. Initial Source Audit — Mandatory Before Implementation

Before writing code, inspect the complete supplied source archive.

Identify:

1. Repository revision and available Git metadata.
2. Solution and project structure.
3. Target frameworks and build configurations.
4. NinjaTrader references and dependency paths.
5. Existing NT8 AddOns and integration points.
6. Existing Market Data adapters.
7. Existing DOM adapters and snapshot synchronization logic.
8. Existing Account and Portfolio adapters.
9. Existing Order and Execution mapping.
10. Instrument Registry and Attestation implementation.
11. D-026 and D-078 documentation.
12. Stage 2, Stage 3, and Stage 4 diagnostic sources.
13. Existing logging, telemetry, and event correlation mechanisms.
14. Existing Trading=Disabled and Static Safety enforcement.

Pay particular attention to:

- `Nt8SimPortfolioHostReader`
- `Nt8SimAccountAdapterFactory`
- `Nt8SimPortfolioProviderFactory`
- `Nt8HostInstrumentMetadataProvider`
- `SimPortfolioProvider`
- `SimPortfolioCaptureBuilder`
- `PortfolioStatePublisher`
- `DomMarketDataAdapter`
- The actual implementation responsible for DOM Reset and Snapshot Synchronization
- The TradeDelta and OrderFlow readiness paths
- Instrument Attestation and D-078 approval logic

Verify actual paths and class names against the supplied archive. Do not assume they exist exactly as described.

Create a source-to-diagnostic mapping table identifying which runtime questions can be answered directly by NinjaTrader, which require existing RiskPilot telemetry, and which currently cannot be observed.

Do not begin implementation until this inspection is complete.

---

## 5. Diagnostic AddOn Architecture

Build an independent NinjaTrader AddOn named:

`RiskPilot NT8 Diagnostic Center`

Suggested namespace:

`RiskPilot.Diagnostics.Nt8`

Use an independent project and output directory.

The AddOn should be accessible from the NinjaTrader Control Center through its own menu entry or dedicated window.

The UI should provide:

- Quick Diagnostic
- Full Diagnostic
- Post-Trade Audit
- Stop Recording
- Generate Final Report
- Open Output Folder
- Current diagnostic status
- Current environment information

The AddOn must not interfere with the RiskPilot production AddOn.

Use a modular architecture. Each diagnostic module must be independently startable, stoppable, and report its own status.

Event handlers must be lightweight. Do not perform large file operations, expensive serialization, UI operations, or blocking work directly inside market data or account event callbacks.

Use a thread-safe queue or equivalent bounded collection for event processing.

Respect NinjaTrader threading and event lifecycle requirements. Verify these against the actual assemblies and supported API documentation.

Avoid unbounded memory growth. Implement configurable recording limits, rotation, and orderly shutdown.

---

## 6. Required Diagnostic Modules

### Module 1 — Environment and API Compatibility

Capture:

- Exact NinjaTrader version and build.
- Application version information.
- .NET runtime version.
- Operating system version.
- Process architecture.
- Assembly names, versions, and file locations for relevant loaded NinjaTrader assemblies.
- Relevant Rithmic-related assembly information where accessible.
- Diagnostic AddOn version.
- RiskPilot source revision or explicitly Unknown.
- Diagnostic session ID.
- Session start and end timestamps in UTC.
- Local time and timezone information.

Inspect the actual loaded API and verify the members used by the AddOn.

Do not report a successful real NinjaTrader build when only stubs or mock assemblies were used.

If NinjaTrader 8.1.9.0 is installed, validate against that installation. If a different build is installed, report the actual version and explain the mismatch.

Produce an API compatibility report identifying required members that were verified and those that could not be verified.

### Module 2 — Connection and Provider Identity

This module is especially important because PR #275 changed Provider detection.

For each accessible connection and relevant account, collect:

- Display name.
- Connection status.
- Actual Provider object type.
- Raw Provider value.
- Provider.ToString() result.
- Numeric Enum value, if applicable.
- Assembly and type information.
- Account-to-connection association, where exposed.
- Price connection state.
- Order connection state.
- Broker connection state.

Evaluate Provider detection independently at the account and connection levels.

Specifically test the logic introduced in PR #275:

`string.Equals(provider.ToString(), "Rithmic", StringComparison.OrdinalIgnoreCase)`

Determine whether the actual runtime representation satisfies this comparison.

Do not simply assume that it does.

Record the result for each relevant Factory:

- Nt8SimAccountAdapterFactory
- Nt8SimPortfolioProviderFactory
- Nt8HostInstrumentMetadataProvider

Where these Factory methods cannot safely be invoked independently without modifying project state, inspect their decision logic and report the limitation. Do not force their execution.

The report must distinguish:

- Provider identity observed.
- Provider identity inferred.
- Provider identity accepted by RiskPilot.
- Provider identity unavailable.

These are different facts.

### Module 3 — Account, Order, Execution, and Position Lifecycle

This module must directly address F-05.

Identify the actual supported NinjaTrader API members for:

- Account enumeration.
- Account.Orders.
- Order updates.
- Execution updates.
- Position updates.

Verify the actual event signatures and property availability against the installed NinjaTrader assemblies.

Do not invent API names or assume event signatures.

For every accessible order, record:

- Timestamp UTC.
- Diagnostic session ID.
- Account identity.
- Instrument full name.
- Order ID raw value classification.
- Order ID value, with appropriate redaction if needed.
- Order state.
- Order action.
- Order type.
- Quantity.
- Filled quantity.
- Average fill price where accessible.
- Time-in-force where accessible.
- Relevant name or ATM/strategy label where available.
- Event source.
- Whether the observation came from Account.Orders or an Order Update event.

Classify OrderId as:

- NULL
- EMPTY
- WHITESPACE
- VALID
- UNAVAILABLE

Record the exact observed value separately when safe.

Take repeated snapshots of Account.Orders:

- Before any user test trade.
- During order submission.
- After order acceptance.
- While orders are Working.
- During partial execution if it occurs.
- After Filled.
- After Cancelled.
- After the position becomes Flat.

Do not create test orders.

The user will decide when to perform the manual Sim test.

The key F-05 questions are:

1. Does Account.Orders retain Filled orders?
2. Does it retain Cancelled orders?
3. Does it retain Rejected orders?
4. Are terminal orders removed from the collection?
5. Can terminal orders have null, empty, or whitespace OrderId values?
6. Are Working and PendingSubmit orders always represented with valid identifiers?
7. Does Account.Orders change immediately after order state transitions?
8. Does Account.Orders contain orders for multiple instruments?
9. Does Account.Orders contain orders associated with other relevant strategies or ATM instances?
10. Are event notifications and Account.Orders snapshots temporally consistent?
11. Can collection enumeration race with order updates?

Produce a time-ordered order lifecycle record linking order, execution, and position events when reliable identifiers exist.

Never invent an association when identity is ambiguous.

If event and snapshot information disagree, preserve both observations and flag the discrepancy.

The report must explicitly answer whether the current F-05 check in `Nt8SimPortfolioHostReader` can incorrectly reject a terminal order before its state is mapped.

Do not modify F-05 production code.

### Module 4 — Instrument Metadata and Attestation / D-078

Collect actual NinjaTrader metadata for relevant instruments:

- Instrument full name.
- Master instrument identity.
- Exchange.
- Expiry.
- Tick size.
- Point value.
- Currency.
- Relevant trading hours information where available.
- Current contract identity.
- Instrument object/type identity.

Compare actual observed metadata with the relevant RiskPilot Instrument Registry and D-078 definitions, using existing public/read-only interfaces where available.

Report every field independently:

- Available.
- Missing.
- Mismatched.
- Not observable.
- Approval pending.
- Approved, only when a valid existing approval record proves it.

Investigate the known failure reasons:

- `HOST_UNAVAILABLE`
- `D078_FIELD_APPROVAL_NOT_APPROVED`
- `MISSING_INSTRUMENT_SPECIFICATION`

Determine whether these are caused by missing runtime metadata, Provider detection, unavailable Host state, an incomplete Registry, or an independent approval requirement.

Do not automatically approve any field.

Do not manufacture approval records.

Do not infer D-078 approval from the fact that an instrument is visible in NinjaTrader.

### Module 5 — DOM and Market Data Diagnostics

Inspect the actual NinjaTrader MarketDepth and market data APIs used by the project.

Collect, where supported:

- Instrument identity.
- Bid/Ask depth snapshots.
- Native Bid level count.
- Native Ask level count.
- Internal Bid level count.
- Internal Ask level count.
- Depth operation type.
- Depth position.
- Price.
- Volume.
- Event timestamps.
- Update sequence.
- Reset events.
- Snapshot synchronization events.
- Snapshot synchronization failures.
- Out-of-bounds events.
- Last data age.
- Feed availability.
- Event processing latency where measurable.

Specifically investigate the previous recording's observations:

- 265 Snapshot Sync events.
- 265 Snapshot Sync resets.
- 264 Update Out of Bounds events.
- 1 Snapshot Sync failure.
- 64 native Bid levels.
- 64 native Ask levels.
- 10 internal Bid levels.
- 6 internal Ask levels.

Do not assume these counts identify the root cause.

Determine whether the observed mismatch is caused by native snapshot depth, internal depth configuration, index translation, event ordering, reset handling, or another supported explanation.

Correlate native NinjaTrader events with existing RiskPilot DOM telemetry when possible.

Preserve raw event evidence and avoid silently normalizing or discarding invalid events.

Do not change DOM adapter behavior.

### Module 6 — TradeDelta and OrderFlow Readiness

Inspect existing RiskPilot diagnostic interfaces and log sources.

Capture, where available:

- TradeDelta status.
- TradeDelta completeness.
- Reasons for incomplete data.
- OrderFlow readiness.
- OrderFlow availability.
- Reset-related status.
- Required upstream data availability.
- Quote and Trade event counts.
- Relevant event processing failures.
- Time ranges affected by incomplete data.

Investigate the previously observed conditions:

- `TradeDelta status is Incomplete`
- `OrderFlow overall status is Unavailable`
- `RESET_OPERATION_RECEIVED`

Determine whether these failures correlate with DOM reset activity, market data gaps, event ordering, initialization timing, or another observed cause.

Do not declare a causal relationship unless the evidence supports it.

### Module 7 — Strategy, Candidate, and Risk Gate Diagnostics

For all five configured strategies, capture existing diagnostic information where accessible:

- Strategy identity.
- Enabled/disabled state.
- Evaluation count.
- Candidate count.
- Valid candidate count.
- Invalid candidate count.
- Candidate rejection reasons.
- Missing capability reasons.
- Stage 2 decisions.
- Stage 3 decisions.
- Stage 4 decisions.
- Position sizing result.
- Risk Gate result.
- Final NoTrade reason.

Investigate the previous recording's observations:

- Five active strategies.
- Zero valid final candidates.
- `ALL_CANDIDATES_INVALID`
- `ANALYTICAL_NO_TRADE`
- `NoTradeNoValidCandidates`
- Incomplete TradeDelta.
- Unavailable OrderFlow.

Avoid incorrectly interpreting repeated diagnostic events as independent strategy failures.

Aggregate repeated events while preserving the original counts and representative timestamps.

If the strategy state is not exposed through a supported diagnostic interface, report `NOT_OBSERVABLE`.

Do not add invasive hooks or modify strategy behavior simply to collect these values.

### Module 8 — Safety and Readiness Audit

Verify the diagnostic AddOn's own implementation and inspect available RiskPilot safety information.

The report must confirm:

- Trading remains Disabled.
- No order submission API is used.
- No order modification API is used.
- No cancellation API is used.
- No position mutation API is used.
- No account mutation API is used.
- No connection mutation API is used.
- No RiskPilot trading logic was modified.
- No trading state was enabled.
- All diagnostic subscriptions are disposed correctly on shutdown.
- Recording can be stopped without affecting NinjaTrader or RiskPilot operation.

Do not claim runtime Trading=Disabled based solely on a source-code search or configuration file. Require actual runtime evidence or report `INCONCLUSIVE`.

---

## 7. Diagnostic Session Workflow

The AddOn must support three modes.

### A. Quick Diagnostic

A short read-only inspection of:

- Environment.
- Loaded assemblies.
- Provider identity.
- Connection state.
- Account availability.
- Instrument metadata.
- Basic safety status.

It should not require a user trade.

### B. Full Diagnostic

Starts a controlled diagnostic session.

1. Generate a unique session ID.
2. Capture environment and source identity.
3. Subscribe to supported read-only events.
4. Record initial account and instrument snapshots.
5. Begin DOM and market data monitoring.
6. Monitor RiskPilot diagnostics where supported.
7. Preserve raw event sequence and timestamps.
8. Allow the user to perform a manual Sim trade.
9. Continue recording order, execution, position, DOM, and strategy observations.
10. Allow the user to stop recording.
11. Generate the complete evidence package.

The AddOn must clearly show that it is only recording observations and is not performing trading operations.

### C. Post-Trade Audit

Analyze the order, execution, and position lifecycle from a completed manual Sim trade.

Compare:

- Account.Orders snapshots.
- Order events.
- Execution events.
- Position events.
- Terminal order retention.
- Raw OrderId values.
- Final position state.

The audit must not assume that the existence of a manual Sim execution means the automated bot placed an order.

---

## 8. User Interface Requirements

Provide a practical, responsive diagnostic window in NinjaTrader.

Display:

- NinjaTrader version.
- Connection.
- Provider identity.
- Selected Account.
- Selected Instrument.
- Trading safety state.
- Current diagnostic session.
- Recording status.
- Last event time.
- Module status.
- Error count.

Provide buttons:

- Quick Diagnostic
- Start Full Diagnostic
- Stop Recording
- Post-Trade Audit
- Generate Report
- Open Output Folder

The UI must be informational only.

Never add a trading button, order control, account modification control, or strategy activation control.

Do not create unnecessary visual complexity.

---

## 9. Evidence Package and Reporting

Every completed diagnostic session must generate an independently reviewable evidence package.

Suggested structure:

`RiskPilot-Diagnostic/<SessionId>/`

```text
Report/
    EXECUTIVE_SUMMARY.html
    FULL_TECHNICAL_REPORT.html
    ACCEPTANCE_MATRIX.csv

Environment/
    platform_version.json
    assemblies.json
    provider_identity.json

Account/
    account_snapshots.csv
    order_lifecycle.csv
    execution_events.csv
    position_events.csv

Instrument/
    instrument_metadata.json
    attestation_report.json
    d078_status.json

MarketData/
    market_data_statistics.json
    dom_events.csv
    dom_resets.csv
    dom_statistics.json

OrderFlow/
    tradedelta_status.json
    orderflow_readiness.json

Strategies/
    candidate_statistics.csv
    strategy_failures.csv
    risk_gate_events.csv

Safety/
    safety_audit.json

Logs/
    diagnostic.log

manifest.json
```

Files may be omitted when a module is genuinely not applicable, but the manifest must explain why.

The report must use four result states:

- PASS: directly observed and supported by sufficient evidence.
- FAIL: directly observed failure.
- INCONCLUSIVE: insufficient or conflicting evidence.
- NOT_APPLICABLE: the test does not apply to the current environment.

Every finding must include:

- Finding ID.
- Severity.
- Result state.
- Exact observation.
- Timestamp.
- Evidence source.
- Relevant module or source component.
- Method used to obtain the evidence.
- Limitations.
- Recommended next action.

Every conclusion must be traceable to raw evidence.

Do not report a module as PASS merely because no exception occurred.

Do not convert missing information into a successful result.

Do not claim full Stage 4 acceptance simply because the diagnostic AddOn completed successfully.

Include a manifest containing:

- Session ID.
- Platform version.
- AddOn version.
- Source revision or Unknown.
- File names.
- File sizes.
- SHA-256 checksums.
- Session start and end times.
- Module completion statuses.

Redact credentials, passwords, API keys, access tokens, and other secrets. Never collect or write connection credentials to the diagnostic output.

---

## 10. Build and Compatibility Requirements

The AddOn must be built against the actual NinjaTrader installation whenever available.

Preferred validation target: NinjaTrader 8.1.9.0.

Inspect the actual local installation paths and reference the real DLLs.

Do not redistribute proprietary NinjaTrader DLLs or bundle them into the deliverable ZIP.

Do not use mock assemblies as a substitute for real compilation.

If actual NinjaTrader assemblies are unavailable in your environment:

1. Produce the source project.
2. Produce a clearly documented build configuration.
3. Identify the exact assemblies required.
4. Document the expected reference paths.
5. Report compilation as NOT VERIFIED.
6. Do not claim successful NinjaTrader compilation.

If compilation fails, provide:

- Exact compiler error.
- File and line.
- Actual API member involved.
- Why the member is unavailable or incompatible.
- Proposed correction.
- Confirmation that the correction does not weaken safety.

Do not silently substitute fake API members or stub behavior.

---

## 11. Testing Requirements

Create automated tests for logic that can be tested independently of NinjaTrader.

At minimum:

- OrderId classification: null, empty, whitespace, valid.
- Order state classification.
- Terminal order detection.
- Account.Orders snapshot comparison.
- Event ordering and correlation.
- Duplicate event handling.
- Missing event handling.
- Session ID generation.
- Timestamp handling.
- CSV and JSON output.
- Manifest checksum generation.
- Recording shutdown.
- Bounded queue behavior.
- Secret redaction.
- Result-state classification.

Clearly separate unit tests from actual NinjaTrader Host Acceptance tests.

A passing unit test is not proof of successful runtime behavior.

A stub-based build is not proof of compatibility with the actual NinjaTrader installation.

---

## 12. Implementation Boundaries

The supplied RiskPilot.BOT source archive is for inspection and integration analysis.

Do not directly modify the existing RiskPilot.BOT production source.

Place all new code in an independent project and a separate output directory.

If an existing public diagnostic interface is available, use it.

If an interface does not exist:

- Do not use private reflection or invasive access.
- Do not change production code without authorization.
- Report the missing observability.
- Propose the smallest possible future read-only integration interface as a separate recommendation.

Do not make unrelated refactors.

Do not change existing tests to make the AddOn appear successful.

Do not remove or weaken safety checks.

---

## 13. Required Deliverables

Deliver all of the following:

1. Complete independent AddOn source code.
2. Solution and project files.
3. Build instructions.
4. NinjaTrader installation instructions.
5. Diagnostic UI.
6. Automated unit tests.
7. API compatibility report.
8. Source-to-diagnostic mapping report.
9. Safety review.
10. Sample output package using clearly identified sample data only.
11. Final ZIP containing the independent AddOn source and required non-proprietary files.
12. A final technical report describing what is verified, what is incomplete, and what requires actual NinjaTrader execution.

Do not include fake runtime evidence in the sample report.

Clearly label all sample or simulated data.

---

## 14. Final Acceptance Criteria

The task is complete only when the deliverable satisfies the applicable items below:

- Independent AddOn project exists.
- Production trading source remains unchanged.
- Read-only restrictions are enforced.
- Actual NinjaTrader API compatibility has been inspected.
- Real DLL compilation has been completed, or the exact limitation is documented.
- Provider identity diagnostics are implemented.
- Account.Orders snapshots are implemented.
- Order, execution, and position lifecycle recording is implemented.
- Raw OrderId classification is implemented.
- Instrument metadata diagnostics are implemented.
- DOM diagnostics are implemented.
- TradeDelta and OrderFlow diagnostics are implemented where supported.
- Strategy and Risk Gate diagnostics are implemented where supported.
- Evidence output is structured and reproducible.
- Result states are evidence-based.
- No trading API is invoked.
- Automated tests pass.
- Installation and operation instructions are complete.

The final report must distinguish three separate outcomes:

1. AddOn implementation and build status.
2. Runtime diagnostic coverage and evidence quality.
3. RiskPilot.BOT acceptance status.

These must never be merged into one generic PASS result.

A successful diagnostic AddOn build does NOT mean RiskPilot.BOT has passed Stage 4 acceptance.

---

## 15. Final Working Instructions

Work systematically.

First inspect the source archive and the previous NinjaTrader evidence.

Then establish the exact API and integration constraints.

Then implement the independent AddOn.

Do not stop after producing a design document if implementation is feasible.

Do not invent NinjaTrader API members.

Do not make assumptions about Provider representations, OrderState behavior, event signatures, or Account.Orders retention.

Verify uncertain API details from the actual installed assemblies or authoritative NinjaTrader documentation.

When something cannot be verified, say so explicitly.

Do not modify the RiskPilot.BOT production code.

Do not submit or simulate any trade.

Do not enable trading.

At completion, provide a concise but technically complete handoff report explaining:

- What was built.
- What was compiled.
- Which NinjaTrader version was used.
- Which tests passed.
- Which tests could not be performed.
- What evidence the tool can collect.
- What remains unobservable.
- How the user should install and run it.
- Which exact files should be returned for independent review.

The priority is reliable evidence, correct API usage, strict isolation, and safe execution — not the number of files produced or the appearance of a successful build.