# Data Migration & Cutover Integration

## Data Migration Integration Model

![SAP Cutover Data Migration Integration](assets/cutover-data-migration-integration-flow.png)

Most cutover frameworks treat data migration as a prerequisite. Something that happens before the cutover starts, managed by a separate workstream, handed off with a completion sign-off and a green status.

That model works until it doesn't — which is usually during the downtime window, when a reconciliation discrepancy surfaces, a migration job runs longer than estimated, or a data validation sign-off is required from someone who is not in the war room.

This document treats data migration as an integrated component of cutover execution, not a dependency that was resolved before the window opened.

---

## Why the separation fails

Data migration and cutover share the same execution window, the same critical path, and the same go-live gate. What they do not always share is the same command structure, the same issue log, or the same definition of "done."

When migration runs behind a separate workstream with its own escalation path, the Cutover Lead is managing a dependency they cannot directly observe. The migration team reports status through their own channel. The Cutover Lead infers readiness from those reports — often without visibility into what "reconciliation in progress" actually means in terms of time, risk, and impact on the go-live sequence.

The integration failure is governance, not technical. The fix is structural.

---

## The integrated model

Data migration activities are sequenced inside the cutover plan — not referenced from it. The Data Migration Lead is in the war room. Reconciliation checkpoints are formal gates in the go-live confirmation sequence. Migration sign-off is a named prerequisite for the go-live declaration, with the same status as Basis sign-off or functional smoke test completion.

This does not mean the migration team is subordinate to the Cutover Lead during execution. It means their critical path activities are visible on the same board, their escalation path runs through the same war room, and their completion criteria are agreed before the window opens.

---

## Migration phases inside the cutover window

### Pre-cutover preparation (T-2 weeks to T-48h)

This phase happens before the downtime window but determines what is possible inside it.

**Object scoping confirmation** — the list of migration objects is frozen and signed off. Any object added after this point requires explicit Cutover Lead approval and a documented impact assessment on the migration timeline. Late scope additions to data migration are a disproportionate source of cutover overruns.

**Migration cycle completion** — the final dress rehearsal migration (typically Mock 3 or equivalent) runs in a production-representative environment with production-representative data volumes. Duration is measured, not estimated. If Mock 3 takes 6 hours and the cutover window allocates 4 hours for migration, that is a planning crisis, not a timing issue.

**Reconciliation tooling validation** — the reconciliation approach (automated reports, manual count checks, financial balance comparisons) is tested and confirmed in the dress rehearsal. Reconciliation that is being run for the first time during the live window is a risk that should not exist.

**Go-live migration scope vs. post-go-live migration scope** — not everything migrates on Day 1. Historical data, archived objects, and non-critical master data can move post-go-live. The boundary between what must be live on Day 1 and what can follow is a business decision that needs to be made and documented before the window opens, not discovered when the migration runs long.

### Downtime window — migration execution

**Sequencing within the cutover plan** — migration jobs are sequenced against technical activities. The typical sequence: system freeze → backup → Basis preparation → migration job execution → reconciliation → functional smoke tests → go-live. The migration jobs do not start until Basis confirms the environment is ready. Functional smoke tests do not start until the Data Migration Lead confirms reconciliation sign-off.

Violations of this sequence — functional teams starting smoke tests before reconciliation is complete, or migration jobs running in parallel with transport activities that share the same objects — are a primary source of data consistency issues discovered post go-live.

**Migration job monitoring** — each migration object (master data, open items, balances, transactional data) has a defined execution duration based on dress rehearsal results, an owner, and a buffer threshold. When a job exceeds its buffer threshold, it is escalated immediately to the Cutover Lead — not after it finishes, not when it misses the checkpoint.

In programs with 10+ SAP modules (FI, CO, MM, SD, QM, PP, PM, EWM, TM, HR), migration jobs run across multiple workstreams simultaneously. A delay in FI open item migration has direct impact on CO reconciliation and downstream impact on the financial smoke test sequence. These cascades are only manageable if the dependency map was built in advance.

**Parallel migration streams** — some objects can migrate in parallel. Most cannot. The sequencing logic follows the same principle as the cutover plan: parallel where failure modes do not intersect, sequential where they do. Master data before transactional data. Organizational structures before dependent objects. Financial balances before operational documents.

The Data Migration Lead owns the sequencing logic and is accountable for it during execution.

### Reconciliation as a formal gate

Reconciliation is not a step that follows migration. It is the gate that determines whether migration is complete.

A migration job that finished with zero errors is not a complete migration. It is a migration that ran without technical failures. Whether the data is correct — counts match, balances reconcile, critical master data records are present and accurate — is determined by reconciliation, not by job completion status.

**Reconciliation scope** — defined before the window opens. At minimum: record counts by object type, financial balance comparison (opening balances in S/4HANA match closing balances from source system), and a sample validation of critical master data records (customer master, vendor master, material master, cost center hierarchy).

In programs with regulatory requirements (Tax Reform, eSocial, financial reporting), reconciliation scope expands to include compliance-specific validations — tax classification data, social security contribution bases, financial account mappings. These are not optional. A go-live with unreconciled tax-relevant data is a compliance exposure, not just a data quality issue.

**Reconciliation sign-off authority** — named, confirmed before the window opens. In most programs, this is a joint sign-off: the Data Migration Lead confirms technical reconciliation (counts, balances), the relevant business representative confirms business-layer validation (the data makes operational sense). Both sign-offs are required. One without the other is not complete.

**Reconciliation discrepancy protocol** — pre-agreed thresholds and decision rules for what constitutes an acceptable discrepancy versus a blocking one. A 0.01% variance in record counts on a non-critical object is different from a balance discrepancy in a GL account that will be used in the first financial posting. The difference should be defined before the window, not negotiated inside it.

---

## Data Migration Lead in the war room

The Data Migration Lead attends every war room status sync during the migration execution phase. Their update follows the same format as every other workstream: current status against plan, active blockers, next checkpoint.

They have direct escalation authority to the Cutover Lead for any migration issue that affects the critical path. They do not route migration escalations through a separate workstream lead or a separate channel.

Their sign-off is a named line item in the go-live confirmation sequence. The Cutover Lead does not declare go-live until that line item is confirmed.

---

## Post-go-live: migration tail activities

Not everything that needs to happen with data happens before go-live. Some activities run during hypercare:

**Delta migration** — in programs with a live source system running during the downtime window (common in phased cutovers), delta records generated after the final migration load need to be captured and loaded into S/4HANA. This is a planned activity, not a remediation. It has an owner, a timeline, and a completion gate.

**Historical data migration** — historical documents, archived transactions, and reporting data that are not required for Day 1 operations migrate post-go-live on a defined schedule. The schedule is agreed before go-live and managed as a hypercare work item, not an open risk.

**Master data corrections** — discrepancies identified during reconciliation that were classified as non-blocking go-live items are resolved post-go-live with defined SLAs. Each correction has an owner and a business impact assessment. They are tracked in the hypercare issue log, not in informal channels.

The migration is not complete when the go-live is declared. It is complete when all migration activities — including post-go-live tail work — are closed and signed off.

---

## Common failure patterns specific to migration-cutover integration

**Migration duration underestimated from dress rehearsal**

The dress rehearsal ran against a subset of production data volume. The live migration runs against full volume and takes 40% longer. The cutover plan had no contingency for this scenario because the dress rehearsal result was treated as a confirmed duration, not an estimate with variance.

Mitigation: dress rehearsals run against full production data volume, or duration estimates include an explicit variance factor derived from volume differences. The cutover plan includes a migration duration buffer that is sized based on actual rehearsal variance, not optimism.

**Reconciliation discrepancy discovered after smoke tests started**

Functional teams began smoke tests while reconciliation was still running — because the migration jobs finished on time and the assumption was that reconciliation would follow quickly. Reconciliation found a balance discrepancy in a GL account that is used in the first smoke test scenario. The smoke test is now blocked. The reconciliation team and the functional team are both waiting on each other.

Mitigation: functional smoke tests have a hard dependency on reconciliation sign-off in the cutover plan. Not a soft dependency. Not an assumption. A named gate that the Execution Lead enforces.

**Data Migration Lead not in the war room**

Migration status is reported through a separate channel. The Cutover Lead gets updates every hour from the migration workstream lead, who is summarizing what the migration team told them. By the time a job overrun is escalated to the Cutover Lead, it has consumed 45 minutes of buffer that was not visible in the war room.

Mitigation: the Data Migration Lead is in the war room. Their status is reported directly, on the same cadence as every other workstream.

**Post-go-live migration tail not planned**

Go-live is declared. The migration team disperses. Two weeks later, a business user reports that historical purchase orders are not visible in the system. The historical data migration was discussed during planning but never formally scheduled. Nobody owns it. Nobody has a timeline.

Mitigation: post-go-live migration activities are planned, owned, and scheduled before go-live — not after. They are tracked as hypercare work items with defined completion dates and business sign-off criteria.

---

## Integration checklist

### Before the downtime window opens

- [ ] Migration object scope frozen and signed off
- [ ] Dress rehearsal completed with production-representative data volume
- [ ] Migration durations confirmed from dress rehearsal (not estimated)
- [ ] Reconciliation approach tested and validated
- [ ] Reconciliation sign-off authority confirmed (name, not role)
- [ ] Reconciliation discrepancy thresholds and decision rules documented
- [ ] Go-live vs. post-go-live migration scope boundary agreed and documented
- [ ] Post-go-live tail activities planned, owned, and scheduled
- [ ] Data Migration Lead confirmed for war room attendance
- [ ] Migration activities sequenced in the cutover plan with dependencies to technical and functional activities

### During the downtime window

- [ ] Migration jobs started only after Basis confirms environment readiness
- [ ] Migration status reported in war room on standard cadence
- [ ] Job duration tracked against dress rehearsal baseline — buffer consumed reported proactively
- [ ] Reconciliation started immediately upon job completion — not queued
- [ ] Reconciliation sign-off obtained before functional smoke tests begin
- [ ] Any discrepancy classified against pre-agreed thresholds — blocking vs. non-blocking determined in real time
- [ ] Data Migration Lead sign-off confirmed as part of go-live declaration sequence

### Post go-live

- [ ] Delta migration executed and confirmed (if applicable)
- [ ] Non-blocking discrepancies logged as hypercare items with owners and SLAs
- [ ] Historical data migration scheduled and tracked
- [ ] Migration workstream formally closed when all tail activities are complete

## Notas Relacionadas
- [[SAP Cutover Framework]]
- [[S4HANA]]
- [[ABI-S4Upgrade]]
- [[Hypercare]]
- [[RAID-Log]]
- [[sap-cutover-framework/cutover-flow]]
- [[sap-cutover-framework/cutover-checklist]]
- [[sap-cutover-framework/pre-sum-preparation]]
