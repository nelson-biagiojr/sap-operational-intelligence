# SAP War Room Model (Enterprise)

## War Room Command Structure

![Cutover War Room](assets/cutover-war-room-structure.png)

```mermaid 
graph LR
    subgraph Governance
        Exec[Executive Committee]
    end
    
    subgraph Global_Command
        PD[Program Director / War Room Central]
    end
    
    subgraph Regional_Hubs
        AMER[Americas Hub]
        EMEA[EMEA Hub]
        APAC[APAC Hub]
    end

    Exec <--> PD
    PD <--> AMER
    PD <--> EMEA
    PD <--> APAC
    
    AMER -- Handoff --> EMEA
    EMEA -- Handoff --> APAC
    APAC -- Handoff --> AMER

    style PD fill:#bbf,stroke:#333,stroke-width:2px```

---------------------------------------------------------------------

## What a war room actually is
A war room is not a meeting room with a dashboard. It is a temporary command structure that replaces the normal program governance model during the highest-risk window of the program — from the start of the downtime window through the confirmation of business readiness post go-live.
During that window, the regular escalation hierarchy is too slow. Decisions that would normally take hours need to happen in minutes. The war room compresses the decision chain by putting the right people in the same space (physical or virtual) with a shared view of what is happening, a clear escalation protocol, and pre-agreed decision authority.
Programs that skip the war room structure don't avoid the war. They just fight it without a command center.

## Command structure
Cutover Lead (overall authority during the window)
Single point of command. Not a coordinator — the person with authority to make go/no-go calls, approve rollback, and escalate directly to steering committee. In multi-region programs, this role has to be explicitly separated from regional execution leads to prevent conflicting decisions across time zones.
Owns: critical path status, rollback trigger criteria, SteerCo communication.

## Execution Lead (technical operations)
Manages the sequencing of cutover activities against the plan. Tracks task completion, flags delays before they hit the critical path, and coordinates directly with workstream leads. In a large program, this is a dedicated role — not something the Cutover Lead does while also managing executive communication.
Owns: task log, sequencing dependencies, escalation to Cutover Lead when buffer is consumed.

## SAP Basis Lead
Technical ownership of the SAP environments during the cutover window. Transport management, system monitoring, client settings, background job scheduling, and interface activation are all in this scope. The Basis Lead is the last technical authority before any system-level decision — including rollback.
In multi-system programs (Aurora ran across 8 integrated environments), Basis coordination across systems is a parallel track that requires its own synchronization rhythm, separate from the functional execution stream.
Owns: system health, transport queue, technical go/no-go input.

## Functional Leads (by module)
Each functional area — FI, CO, MM, SD, QM, PP, PM, EWM, TM, HR — has a designated lead accountable for smoke test execution and business validation within their domain. They do not manage the overall cutover. They own their slice of the validation checklist and escalate immediately when a test fails or a dependency is blocked.
In practice, the functional leads are the early warning system. If FI smoke tests are taking longer than the buffer allows, that information needs to reach the Execution Lead before it becomes a critical path problem.
Owns: module-level smoke tests, functional validation sign-off.

## Integration Lead
In any modern SAP program, the system boundary is not the SAP system — it is the integration layer connecting SAP to middleware, third-party platforms, and legacy systems. Integration failures during cutover are disproportionately common and disproportionately disruptive because they are harder to diagnose quickly and often require coordination across vendor boundaries.
The Integration Lead monitors interface activation, end-to-end message flow, and error queues throughout the cutover window. This role is not optional in programs with 3+ integrated systems.
Owns: interface activation sequence, middleware monitoring, integration incident triage.

## Data Migration Lead
In cutovers where data migration is part of the execution (not pre-loaded), the Data Migration Lead tracks migration job completion, reconciliation results, and data validation checkpoints. They are the gate between technical data load completion and business readiness confirmation.
Reconciliation sign-off from the Data Migration Lead is a mandatory input to the go-live decision — not an assumption.
Owns: migration job status, reconciliation checkpoints, data validation sign-off.

## Business Representatives
One representative per critical business domain (Finance, Supply Chain, Operations, HR as applicable). Their role during the cutover window is not advisory — it is active validation. They execute business-layer smoke tests, confirm that the system is behaving as expected from an operational standpoint, and provide the business sign-off that precedes the go-live declaration.
The business representative is also the escalation path when a functional issue requires a business decision (workaround acceptance, scope deferral, manual process activation).
Owns: business-layer validation, sign-off authority for their domain.

## Incident Manager
In a cutover with 50+ active tasks running in parallel, issue tracking cannot be managed informally. The Incident Manager owns the centralized issue log, classifies incidents by severity in real time, assigns owners, tracks resolution progress, and surfaces blockers to the Execution Lead before they age.
The incident log is the single source of truth during the war room. If it is not in the log, it does not officially exist — and no decision gets made based on information that is not in the log.
Owns: issue log, severity classification, resolution tracking, blocker escalation.

# Operating rhythm
Pre-cutover (T-48h to T-0)

Final war room briefing: roles confirmed, contact list distributed, escalation protocol reviewed
Communication channels activated (dedicated Teams/Slack channel for the war room only — separate from the general project channel)
Issue tracking tool configured with cutover-specific categories
Rollback criteria and decision authority formally confirmed — in writing, before the window opens

## During the downtime window

Status sync every 30 minutes minimum (every 15 during critical path activities)
Format: Execution Lead reports task progress against plan, flags delays and blockers; each workstream lead confirms status; Incident Manager reports open incidents and resolution status
All decisions logged with timestamp, owner, and rationale — no verbal-only decisions during the window
Cutover Lead holds the rollback trigger: if the rollback decision point is reached before critical activities are complete, the decision is made then, not after the window expires

## Go-live confirmation
Go-live is declared when all of the following are confirmed — not assumed:

Technical go/no-go: Basis Lead confirms system health, transport consistency, and interface activation
Data validation: Data Migration Lead confirms reconciliation sign-off
Functional validation: all module leads confirm smoke test completion within scope
Business validation: business representatives confirm operational readiness in their domains
Cutover Lead declares go-live and notifies SteerCo

Any one of these missing = go-live not declared. The decision is sequential, not parallel.

## Incident management during the war room
![Hypercare Incident Flow](assets/hypercare-incident-management-flow.png)

## Severity classification
SeverityDefinitionResponseCriticalBlocks go-live or triggers rollback evaluationImmediate — Cutover Lead engaged within 5 minutesHighDelays critical path activity or blocks module sign-offExecution Lead engaged within 15 minutesMediumIssue with workaround available — does not block go-liveFunctional Lead resolves within 1 hourLowCosmetic or post-go-live candidateLogged, deferred to hypercare

## Escalation protocol
The escalation chain is not hierarchical by seniority — it is hierarchical by decision scope.
Functional issue within one module → Functional Lead resolves. If unresolved within defined buffer: Execution Lead engaged. If it affects critical path: Cutover Lead notified. If it triggers rollback criteria: Cutover Lead decides, SteerCo notified.
Cross-module or integration issue → Integration Lead or Execution Lead coordinates immediately. Buffer is shorter because these issues propagate.
Business decision required (workaround acceptance, scope change, process deferral) → Business Representative decides within their domain. If cross-domain: Cutover Lead facilitates, business owners decide jointly.

## Communication model
Internal war room channel (real-time)
Single channel for all war room participants. No parallel threads. Updates follow a structured format:

[TIME] [ROLE] STATUS: Brief status. BLOCKERS: None / [description]. NEXT CHECKPOINT: [time].

No narrative. No context. Facts and status only.
Executive updates (scheduled)
Steering committee and executive sponsors receive structured updates at pre-agreed intervals — typically every 2 hours during the downtime window, and at go-live declaration.
Format: current status against plan, active incidents and resolution status, go-live readiness assessment, next update time.
The executive update is prepared by the Cutover Lead or a designated communications owner — not improvised based on the last thing said in the war room channel.
Stakeholder notifications (milestone-triggered)
Broader stakeholder communication (business users, regional operations teams, IT support) is triggered by milestones — not by time. System freeze confirmation, go-live declaration, hypercare activation. These messages are drafted in advance and released when the milestone is confirmed.

## Multi-region considerations
In programs spanning multiple time zones (Project Aurora covered 14 countries), the war room model requires explicit adaptation:
Follow-the-sun coordination: designate regional execution leads with defined handoff points. The Cutover Lead maintains overall authority but cannot personally monitor all regions simultaneously. Regional leads have pre-agreed escalation thresholds — they do not wait for the Cutover Lead to ask.
Single issue log across regions: a regional incident that is not visible in the central log is a risk that the Cutover Lead cannot manage. Non-negotiable.
Time zone-aware critical path: the sequencing of activities across regions has to account for business hours, regional support availability, and time-zone-driven dependencies. A region going live at 06:00 local time with no local support available at that hour is a governance failure, not a scheduling detail.
Rollback scope by region: in phased multi-region go-lives, rollback decisions may be regional. The criteria for regional rollback versus global rollback must be pre-agreed before the window opens.

## What the war room does not replace
The war room is a crisis command structure. It operates well under pressure precisely because it has a narrow scope and a clear mandate.
It does not replace the cutover plan — the plan is the war room's operational baseline. It does not replace hypercare — hypercare is what comes after the war room stands down. And it does not replace governance — the SteerCo continues to exist; the war room simply compresses the escalation path to it during the critical window.
When the go-live is declared and the system is stable, the war room stands down. Hypercare takes over. The incident log is handed off. The command structure reverts to normal program governance.
The goal of the war room is to make itself unnecessary as quickly as possible.