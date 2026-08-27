# Human-Approved AI Lead Operations

This project demonstrates a controlled lead-operations system built with GoHighLevel and n8n.

Customer enquiries enter through a GoHighLevel form and become authoritative CRM Opportunities before AI processing begins. n8n reloads the current CRM state, validates the intake contract, applies durable replay controls, and generates a structured operational recommendation.

The AI recommendation does not directly control revenue routing. GoHighLevel applies deterministic business rules to the submitted CRM data, and ambiguous or conflicting cases are held for human review.

Approved leads can enter a second n8n workflow that validates the human decision, reloads the intended recipient, generates a constrained follow-up draft, creates an unsent Gmail draft, and creates an internal GoHighLevel review task.

The public repository focuses on architecture, execution evidence, control boundaries, human review, and operational reporting. Environment-specific workflow exports, credentials, internal IDs, and private webhook configuration are intentionally excluded.

## What this system handles

- Accepts enquiries through a designated GoHighLevel form
- Creates authoritative CRM Opportunities before AI processing
- Generates and validates unique event-correlation identifiers
- Reloads current CRM state before making automation decisions
- Uses n8n for AI-assisted intake interpretation
- Restricts AI output through structured schemas and allowed vocabularies
- Persists processing state in durable n8n Data Tables
- Detects duplicate webhook deliveries
- Supports controlled recovery of stale or failed processing
- Routes eligible enquiries using deterministic CRM rules
- Holds ambiguous or conflicting enquiries for human review
- Records explicit human review decisions
- Requires approval before follow-up preparation
- Validates the intended email recipient before drafting
- Creates an unsent Gmail draft rather than sending automatically
- Creates an internal GoHighLevel review task
- Persists downstream completion state
- Provides scheduled operational reporting

## System flow

```mermaid
flowchart TD
    A[GoHighLevel Enquiry Form] --> B[01A Lead Intake & Opportunity Creation]
    B --> C[(Authoritative Intake & Review Opportunity)]

    C --> D[01B AI Handoff]
    D --> E[01 n8n AI Recommendation Orchestrator]

    E --> F[Authoritative CRM Reload]
    F --> G[Idempotency & Replay Control]
    G --> H[Structured AI Recommendation]
    H --> I[AI Contract Validation]
    I --> J[Validated CRM Handoff]

    J --> K[02 Deterministic Lead Validation & Revenue Routing]

    K -->|Eligible| L[Approved Revenue Route]
    K -->|Ambiguous / Conflict / Invalid| M[Human Review Required]

    M --> N[03A Human Review Decision Capture]

    N -->|Approved / Corrected & Approved| O[Ready to Route]
    N -->|Needs More Information / Rejected| P[Controlled Hold]

    O --> Q[02 n8n Approved Follow-Up Orchestrator]
    Q --> R[Recipient Validation]
    R --> S[Controlled AI Draft]
    S --> T[Unsent Gmail Draft]
    T --> U[Internal GHL Review Task]
    U --> V[(Completed Follow-Up Ledger)]

    C -. Operational state .-> W[Weekly Operations Reporting]
    M -. Review queue .-> W
    O -. Routing readiness .-> W