\# Data Dictionary



This document describes the operational fields, controlled values, and persistent processing states used by the Human-Approved AI Lead Operations system.



Environment-specific GoHighLevel custom-field IDs, pipeline IDs, stage IDs, Data Table IDs, credential references, and internal account identifiers are intentionally excluded.



\---



\## 1. Lead intake fields



These fields originate from the enquiry and become part of the authoritative CRM Opportunity.



| Field | Purpose |

|---|---|

| `Revenue Stream` | Customer-selected business category used by deterministic routing |

| `Requested Location` | Customer-selected preferred operating location |

| `Service / Program Interest` | Free-text description of the service, course, or business interest |

| `Preferred Contact Method` | Customer-selected communication preference |

| `Submitted Enquiry Message` | Original customer enquiry text |

| `Source Event ID` | Generated identifier for the original intake event |

| `External Record Key` | Durable correlation and idempotency key for the intake |

| `AI Processing Status` | Current state of AI-assisted intake processing |



\### Allowed revenue-stream values



\- `Infusion Services`

\- `Courses`

\- `Business Opportunities`

\- `Other / Unknown`



\### Allowed location values



\- `Melbourne`

\- `Perth`

\- `Sydney`

\- `Gold Coast / Byron Bay`

\- `Brisbane`

\- `Sunshine Coast`

\- `Toowoomba`

\- `Other / Unsupported`

\- `Unknown`



\---



\## 2. Event identity



Each valid intake is associated with two correlated identifiers.



| Field | Purpose |

|---|---|

| `Source Event ID` | Identifies the originating form event |

| `External Record Key` | Durable external processing identity used for replay control |



The system validates that both identifiers reference the same generated event before AI processing begins.



This provides a traceable relationship between:



```text

Form Submission

&#x20;     ↓

CRM Opportunity

&#x20;     ↓

n8n Processing

&#x20;     ↓

Persistent Ledger State

