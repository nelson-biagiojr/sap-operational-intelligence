# Cutover Execution Flow

```mermaid 
graph TD
    A[Pre-Cutover: Readiness Check] --> B[System Freeze & Downtime Window]
    B --> C[Data Migration & Reconciliation]
    C --> D{Phase Gate: Data Sign-off}
    D -- Erro Crítico --> E[Mitigação de Incidentes]
    E --> C
    D -- Sucesso --> F[Technical Cutover]
    F --> G[Business Readiness Validation]
    G --> H{Governance Moment: Go/No-Go}
    H -- No-Go --> I[Rollback Protocol]
    H -- Go --> J[System Open & Hypercare]
    
    style H fill:#f96,stroke:#333,stroke-width:4px
    style I fill:#f66,stroke:#333```

## Phase 1 – Preparation
- Planning and validation  
- Stakeholder alignment  
- Risk assessment  

## Phase 2 – Execution
- System lock  
- Technical activities  
- Monitoring  

## Phase 3 – Validation
- Business validation  
- Data checks  
- Issue resolution  

## Phase 4 – Hypercare
- Incident management  
- Stabilization  
- Transition to AMS  

# Cutover Execution Flow

![SAP Cutover Flow](assets/cutover-flow.png)