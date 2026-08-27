# Data Dictionary

This document describes the public semantic data contracts, controlled values, and persistent processing states used by the Human-Approved AI Lead Operations system.

GoHighLevel custom-field IDs, pipeline and stage IDs, Data Table IDs, webhook IDs, credential references, account identifiers, CRM object IDs, Gmail draft IDs, task IDs, and environment-specific configuration are intentionally excluded.

---

## 1. Lead intake and CRM control fields

The intake flow combines authoritative GoHighLevel Opportunity state with limited handoff context.

n8n reloads the current Opportunity directly from GoHighLevel before validating the intake. Handoff fields may provide additional enquiry context, but they are not treated as sufficient authority for business-state decisions.

### Authoritative Opportunity fields

| Field                        | Purpose                                                                      |
| ---------------------------- | ---------------------------------------------------------------------------- |
| `Revenue Stream`             | Submitted business category used by deterministic routing                    |
| `Requested Location`         | Submitted preferred operating location                                       |
| `Service / Program Interest` | Free-text description of the requested service, course, or business interest |
| `Source Event ID`            | Correlation identity for the originating intake event                        |
| `External Record Key`        | Durable external identity used for correlation and replay control            |
| `AI Processing Status`       | Current state of AI-assisted intake processing                               |

### Handoff context fields

| Field                       | Purpose                                                                |
| --------------------------- | ---------------------------------------------------------------------- |
| `Submitted Enquiry Message` | Original enquiry text supplied to the controlled AI-processing context |
| `Preferred Contact Method`  | Submitted communication preference supplied as processing context      |

These handoff values may contribute context, but the webhook does not independently authorize AI processing or downstream routing.

### Allowed revenue-stream values

* `Infusion Services`
* `Courses`
* `Business Opportunities`
* `Other / Unknown`

### Allowed location values

* `Melbourne`
* `Perth`
* `Sydney`
* `Gold Coast / Byron Bay`
* `Brisbane`
* `Sunshine Coast`
* `Toowoomba`
* `Other / Unsupported`
* `Unknown`

The initial accepted intake state uses `AI Processing Status = Queued`.

A validated AI result is stored as advisory CRM data before the Opportunity advances to the AI-interpreted routing boundary. Invalid intake or AI-contract failure does not authorize a revenue route and instead enters controlled review handling.

---

## 2. Event identity and correlation

Every accepted intake receives two correlated identities:

| Field                 | Purpose                                                              |
| --------------------- | -------------------------------------------------------------------- |
| `Source Event ID`     | Identifies the originating intake event                              |
| `External Record Key` | Provides the durable processing identity used by n8n replay controls |

The system verifies that both identifiers refer to the same generated event before normal AI processing can continue.

Conceptually:

```text
Form Submission
      ↓
Authoritative CRM Opportunity
      ↓
n8n Processing
      ↓
Persistent Intake Ledger
```

The actual identifier values and environment-specific prefixes are not part of the public repository.

---

## 3. AI recommendation contract

AI output is advisory and must satisfy a structured contract before it can be written back to GoHighLevel.

| Field                      | Contract                              | Purpose                               |
| -------------------------- | ------------------------------------- | ------------------------------------- |
| `intake_summary`           | Required text                         | Concise summary of the enquiry        |
| `suggested_revenue_stream` | Controlled value                      | AI-recommended business category      |
| `suggested_location`       | Controlled value                      | AI-recommended operating location     |
| `confidence`               | Integer from `0` through `100`        | Confidence in the recommendation      |
| `suggested_priority`       | Controlled value                      | Suggested operational priority        |
| `suggested_next_action`    | Required text, maximum 180 characters | Recommended next operational action   |
| `recommendation_reason`    | Required text, maximum 220 characters | Concise reason for the recommendation |

### Allowed AI priority values

* `Low`
* `Normal`
* `High`

The AI revenue-stream and location vocabularies use the same controlled values defined in the intake contract.

A successful model call alone is not sufficient. The returned structure is independently validated by n8n before the recommendation is accepted.

AI does not have authority to:

* override authoritative CRM truth
* authorize a revenue route
* approve an Opportunity
* authorize customer communication
* send customer-facing email

---

## 4. Routing and human-review controls

GoHighLevel owns the authoritative routing decision after AI interpretation.

Important semantic control fields include:

| Field                       | Purpose                                                                                    |
| --------------------------- | ------------------------------------------------------------------------------------------ |
| `Automation Review Status`  | Indicates whether the Opportunity is ready for automation or requires controlled attention |
| `Automation Control Reason` | Records the reason for review, hold, validation failure, or another control action         |
| `Human Review Decision`     | Records the explicit reviewer decision used by the approved-follow-up boundary             |

The deterministic routing layer separates Opportunities into these outcomes:

* `Eligible Revenue Route`
* `Human Review Required`
* `Validation Fail-Safe`
* `Unexpected Decision Fail-Safe`

### Human-review decisions

Supported decisions are:

* `Approved`
* `Corrected & Approved`
* `Needs More Information`
* `Rejected`
* invalid review submission handling

Only `Approved` and `Corrected & Approved` can progress to `Ready to Route`.

`Needs More Information`, `Rejected`, and invalid review submissions remain in controlled hold handling.

---

## 5. Intake processing ledger

n8n maintains a persistent ledger for AI intake processing.

The public semantic contract is:

| Field                       | Purpose                                                    |
| --------------------------- | ---------------------------------------------------------- |
| `External Record Identity`  | Durable identity of the intake being processed             |
| `Source Event Identity`     | Correlates processing to the original intake event         |
| `Opportunity Reference`     | Internal reference to the authoritative CRM Opportunity    |
| `Contact Reference`         | Internal reference to the associated CRM contact           |
| `Processing Status`         | Current durable processing state                           |
| `First Execution Reference` | Execution that originally reserved the processing identity |
| `Last Execution Reference`  | Most recent execution associated with the record           |
| `Last Seen At`              | Timestamp of the latest processing observation             |
| `Replay Count`              | Number of observed repeat or recovery attempts             |
| `Result Code`               | Machine-readable processing outcome                        |
| `Failure Reason`            | Persisted reason when processing cannot complete normally  |

Actual CRM and execution identifier values are intentionally private.

### Replay-state interpretation

The ledger distinguishes between:

* **Completed / non-retryable** — suppress duplicate processing
* **Stale / recoverable** — allow controlled recovery
* **Active processing** — do not duplicate work

A newly reserved intake enters active processing before AI execution.

Successful AI processing ends with:

```text
Processing Status = COMPLETED
Result Code = AI_RESULT_STORED_AND_HANDED_OFF
```

This means the validated advisory result was stored and handed back to the CRM routing layer.

It does not mean that the lead was approved or that customer communication occurred.

---

## 6. Human-approved follow-up contract

The approved-follow-up workflow does not trust the incoming webhook as authorization by itself.

n8n reloads the authoritative Opportunity and verifies the current CRM state before follow-up preparation can continue.

The approval contract requires, conceptually:

| Requirement             | Expected state                       |
| ----------------------- | ------------------------------------ |
| CRM location            | Expected intake-and-review pipeline  |
| Routing state           | `Ready to Route`                     |
| Opportunity status      | Open                                 |
| Human decision          | `Approved` or `Corrected & Approved` |
| Automation review state | `Ready`                              |
| Event identity          | Valid and correlated                 |
| Required business data  | Present and usable                   |

If this contract fails, follow-up preparation does not continue.

---

## 7. Approved follow-up processing ledger

Approved follow-up uses a separate persistent ledger because it is a different operational transaction from intake interpretation.

The public semantic contract is:

| Field                       | Purpose                                                        |
| --------------------------- | -------------------------------------------------------------- |
| `Approval Idempotency Key`  | Durable identity of the approved business-state snapshot       |
| `External Record Identity`  | Links the approval transaction to the original intake          |
| `Source Event Identity`     | Preserves source-event correlation                             |
| `Opportunity Reference`     | Internal reference to the approved CRM Opportunity             |
| `Contact Reference`         | Internal reference to the intended CRM contact                 |
| `Review Decision`           | Approved human decision being acted upon                       |
| `Processing Status`         | Current follow-up processing state                             |
| `First Execution Reference` | Execution that first reserved the approval transaction         |
| `Last Execution Reference`  | Most recent execution associated with the transaction          |
| `Last Seen At`              | Timestamp of the latest processing observation                 |
| `Replay Count`              | Number of repeated or recovery observations                    |
| `Result Code`               | Machine-readable follow-up outcome                             |
| `Gmail Draft Reference`     | Internal reference proving the draft artifact was created      |
| `GHL Review Task Reference` | Internal reference proving the review task was created         |
| `Completed At`              | Timestamp when all required follow-up artifacts were confirmed |
| `Failure Reason`            | Persisted reason when follow-up cannot complete normally       |

Actual Gmail, task, CRM, and execution identifiers are not published.

### Completion contract

Approved follow-up is complete only after all required controls and downstream artifacts are confirmed:

```text
Human Approval = Valid
        ↓
Recipient = Valid
        ↓
Draft = Valid
        ↓
Unsent Gmail Draft = Confirmed
        ↓
GHL Review Task = Confirmed
        ↓
Processing Status = COMPLETED
        ↓
Result Code = FOLLOW_UP_PREPARED
```

`FOLLOW_UP_PREPARED` means the follow-up package is ready for human review.

It does **not** mean that an email was sent.

Final customer-send authority remains human.

---

## 8. Failure and hold semantics

The workflow architecture distinguishes deterministic failure from uncertainty around external side effects.

### `FAILED`

Used when a reserved processing transaction reaches a confirmed failure and no uncertain external write needs to be protected.

Examples include:

* AI provider failure or invalid AI output after intake reservation
* invalid recipient
* invalid follow-up draft

A known-invalid contract is not treated as successful and does not continue normally.

Authoritative-intake validation and human-approval validation occur before their respective ledger reservations. Those paths fail closed, but they do not create a persisted `FAILED` ledger row.

### `HOLD`

Used when an external side effect may have occurred but the workflow cannot safely confirm the result.

Examples include:

* uncertain Gmail draft creation result
* uncertain GoHighLevel review-task creation result

A `HOLD` state prevents blind replay of the uncertain external action and reduces the risk of duplicate drafts or duplicate review tasks.

---

## 9. Operational reporting terms

Scheduled reporting is separate from transactional processing.

The portfolio reporting view uses these operational terms:

| Term                  | Meaning                                                              |
| --------------------- | -------------------------------------------------------------------- |
| `Open Opportunities`  | Current open CRM Opportunities included in the operational view      |
| `New Intake Contacts` | Intake activity represented in the reporting period                  |
| `Human Review Queue`  | Opportunities currently requiring human attention                    |
| `Ready to Route`      | Opportunities that have reached the approved routing-readiness state |

Reporting reads operational CRM state. It does not determine routing and does not authorize follow-up.

Any counts shown in portfolio evidence are demonstration-record counts, not client performance metrics.

---

## 10. Data authority and public boundary

The system deliberately separates data authority.

**GoHighLevel is authoritative for:**

* contacts
* Opportunities
* submitted CRM data
* pipeline and routing state
* automation review state
* human-review decisions
* internal review tasks

**n8n is responsible for:**

* orchestration
* authoritative CRM reload
* contract validation
* event correlation
* replay and idempotency control
* stale recovery
* AI orchestration
* recipient safety
* controlled Gmail draft creation
* persistent processing ledgers

**AI remains advisory.**

The public repository documents these semantic contracts without exposing reusable secrets, account-specific IDs, private URLs, raw payloads, test records, or environment-specific configuration.
