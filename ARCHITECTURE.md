\# System Architecture



This system separates CRM authority, automation orchestration, AI assistance, deterministic business rules, and human decisions instead of combining them into one autonomous workflow.



The architecture is intentionally split between \*\*GoHighLevel\*\* and \*\*n8n\*\*:



\- \*\*GoHighLevel\*\* owns customer records, Opportunity state, pipeline stages, deterministic routing, and human-review decisions.

\- \*\*n8n\*\* owns cross-system orchestration, authoritative-state validation, AI processing, idempotency, recovery, and controlled follow-up preparation.

\- \*\*OpenAI\*\* provides structured recommendations and draft content within explicit contracts.

\- \*\*Gmail\*\* stores an unsent follow-up draft for human review.



The governing rule is simple:



> \*\*AI assists. Deterministic rules control routing. Humans authorize customer-facing follow-up.\*\*



\---



\## Architecture at a glance



```mermaid

flowchart LR

&#x20;   A\[Customer Enquiry] --> B\[GoHighLevel Intake]



&#x20;   B --> C\[(Authoritative CRM Opportunity)]



&#x20;   C --> D\[n8n AI Recommendation]



&#x20;   D --> E\[Structured Output Validation]



&#x20;   E --> F\[GoHighLevel Deterministic Routing]



&#x20;   F -->|Eligible| G\[Revenue Route]



&#x20;   F -->|Ambiguous / Conflict / Invalid| H\[Human Review]



&#x20;   H -->|Approved| I\[Ready to Route]



&#x20;   H -->|Needs Information / Rejected| J\[Controlled Hold]



&#x20;   I --> K\[n8n Approved Follow-Up]



&#x20;   K --> L\[Recipient Validation]



&#x20;   L --> M\[Controlled AI Draft]



&#x20;   M --> N\[Unsent Gmail Draft]



&#x20;   N --> O\[Internal GHL Review Task]



&#x20;   O --> P\[Human Final Review]

