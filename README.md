\# Human-Approved AI Lead Operations



> Portfolio demo / reference implementation built with GoHighLevel, n8n, OpenAI API, and Gmail.



A controlled lead-operations system for intake, AI-assisted interpretation, deterministic routing, human review, approved follow-up preparation, replay control, and operational reporting.



\*\*AI recommendations are advisory. CRM state, deterministic business rules, and human decisions remain authoritative. No customer-facing follow-up is sent automatically.\*\*



\---



\## Overview



This project demonstrates a production-style lead operations architecture where GoHighLevel acts as the CRM and authoritative system of record, while n8n handles orchestration, AI assistance, validation, idempotency, recovery, and approved downstream actions.



The system is designed around one core principle:



> AI may assist with interpretation and drafting, but it does not independently control revenue routing or customer communication.



\### End-to-end flow



```text

Customer Enquiry

&#x20;     ↓

GoHighLevel Intake

&#x20;     ↓

Authoritative CRM Opportunity

&#x20;     ↓

n8n AI Recommendation

&#x20;     ↓

Structured Output Validation

&#x20;     ↓

Deterministic CRM Routing

&#x20;     ↓

&#x20;┌────────────────────┬─────────────────────┐

&#x20;│ Eligible to Route  │ Human Review Needed │

&#x20;└─────────┬──────────┴──────────┬──────────┘

&#x20;          ↓                     ↓

&#x20;  Revenue Pipeline        Reviewer Decision

&#x20;                                ↓

&#x20;                        Explicit Approval

&#x20;                                ↓

&#x20;                    n8n Approved Follow-Up

&#x20;                                ↓

&#x20;                        Unsent Gmail Draft

&#x20;                                ↓

&#x20;                         GHL Review Task

&#x20;                                ↓

&#x20;                          Human Final Send

