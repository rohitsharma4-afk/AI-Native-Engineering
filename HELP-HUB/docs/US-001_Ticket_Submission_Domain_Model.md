# US-001 Domain Model: Employee Ticket Submission

## Entities

### Ticket
- Attributes:
  - `ticket_id`: string (TKT-YYYYMMDD-XXXXX) [US-001]
  - `description`: text [US-001]
  - `category_id`: uuid [US-001]
  - `priority`: enum {Low, Medium, High, Critical} [US-001]
  - `status`: enum {Open, In Progress, Resolved, Closed} [US-002 dependency]
  - `created_at`: datetime [US-001]
  - `submitter_employee_id`: uuid [US-001]
  - `notification_status`: enum {queued, sent, failed} [US-001 Scenario 9]
  - `duplicate_key`: string? [US-001 Scenario 8]
  - `needs_enrichment`: boolean [US-001 Scenario 6]
- Responsibilities:
  - Validate required ticket fields.
  - Generate and persist unique ticket identifiers.
  - Trigger confirmation notification workflows.
  - Detect and prevent duplicate submissions.

### Employee
- Attributes:
  - `employee_id`: string [US-001]
  - `name`: string [US-001]
  - `email`: string [US-001]
  - `department`: string? [assumed]
- Responsibilities:
  - Submit tickets.
  - Provide contact information for ticket confirmation.
  - Be optionally auto-populated by the employee directory.

### Category
- Attributes:
  - `category_id`: uuid [US-001]
  - `name`: string {Hardware, Software, Network, Account Access, Other} [US-001]
- Responsibilities:
  - Define valid ticket categories.
  - Constrain ticket classification.

### Attachment
- Attributes:
  - `attachment_id`: uuid [US-001 Scenario 7]
  - `ticket_id`: uuid [US-001 Scenario 7]
  - `filename`: string [US-001 Scenario 7]
  - `content_type`: string [US-001 Scenario 7]
  - `size_bytes`: int [US-001 Scenario 7]
- Responsibilities:
  - Associate uploaded files with tickets.
  - Enforce file-size and file-type limits.

### Notification
- Attributes:
  - `notification_id`: uuid [US-001 Scenario 9]
  - `ticket_id`: uuid [US-001 Scenario 9]
  - `to_email`: string [US-001 Scenario 9]
  - `status`: enum {queued, sent, failed} [US-001 Scenario 9]
  - `attempts`: int [US-001 Scenario 9]
  - `last_attempt_at`: datetime [US-001 Scenario 9]
- Responsibilities:
  - Queue and send ticket confirmation notifications.
  - Record retry and failure status for reliability.

## Relationships

- `Employee` 1 → * `Ticket`
  - Type: one-to-many
  - Cardinality: Employee(1) → Ticket(0..*)
  - Directionality: Employee owns ticket submitter reference.

- `Ticket` * → 1 `Category`
  - Type: many-to-one
  - Cardinality: Ticket(0..*) → Category(1)
  - Directionality: Ticket references a single category.

- `Ticket` 1 → * `Attachment`
  - Type: one-to-many
  - Cardinality: Ticket(1) → Attachment(0..*)
  - Directionality: Ticket owns attachments.

- `Ticket` 1 → * `Notification`
  - Type: one-to-many
  - Cardinality: Ticket(1) → Notification(0..*)
  - Directionality: Ticket owns notification records.

- Duplicate submission deduplication is represented by `Ticket.duplicate_key` and may be enforced by application logic or a unique database constraint.

## Business Rules

- **Required ticket fields**
  - Source story ID: US-001
  - Rule statement: Ticket creation must include `category`, `priority`, and `description`.
  - Enforcement point: Application + Domain

- **Ticket ID format**
  - Source story ID: US-001
  - Rule statement: `ticket_id` must follow `TKT-YYYYMMDD-XXXXX` and be globally unique.
  - Enforcement point: Database + Application

- **Attachment validation**
  - Source story ID: US-001 Scenario 7
  - Rule statement: Attachments exceeding size limits or unsupported file types must be rejected.
  - Enforcement point: Application + Database/Storage

- **Duplicate submission idempotency**
  - Source story ID: US-001 Scenario 8
  - Rule statement: Duplicate ticket submissions within a short time window should not create multiple tickets; subsequent attempts must return the same confirmation.
  - Enforcement point: Application (+ optional Database uniqueness)

- **Notification reliability**
  - Source story ID: US-001 Scenario 9
  - Rule statement: Successful ticket creation must queue confirmation notifications and retry failures, marking persistent failures for operator review.
  - Enforcement point: Application + Background worker

- **Directory fallback**
  - Source story ID: US-001 Scenario 6
  - Rule statement: If the employee directory is unavailable, manual submitter contact entry must be allowed and the ticket flagged for enrichment.
  - Enforcement point: Application + Domain

- **Category constraint**
  - Source story ID: US-001 Scenario 3
  - Rule statement: Ticket category must match a valid `Category` entry.
  - Enforcement point: Database + Application

## ER Summary

| Entity | Primary Key | Important Foreign Keys | Notes |
|---|---|---|---|
| Ticket | `ticket_id` | `submitter_employee_id`, `category_id` | Central aggregate root for submissions |
| Employee | `employee_id` |  | Submitter identity source |
| Category | `category_id` |  | Valid category enumeration |
| Attachment | `attachment_id` | `ticket_id` | File metadata and validation |
| Notification | `notification_id` | `ticket_id` | Confirmation and retry tracking |

## Open Questions & Assumptions

- Assumed `employee_id` is provided by the employee directory; if not, manual contact fields should be stored and the ticket flagged for later enrichment.
- Assumed attachment policy defaults (e.g., max 10 MB and allowed types such as PDF, PNG, JPG) because exact limits are not specified.
- Assumed duplicate submission detection uses a short time window (e.g., 10–60 seconds); confirm the desired deduplication threshold.
- Assumed email is the only notification channel; confirm whether SMS or in-app messaging is needed.
- Assumed `status` lifecycle comes from US-002 and is not defined as part of US-001; confirm whether status values should be part of the US-001 domain model.

## Agent Notes

- Modeled from US-001: Employee Ticket Submission.
- Included entities and relationships required to satisfy scenarios around ticket creation, validation, notification, duplicate handling, and directory failure.
- The model is intentionally domain-focused; implementation details are captured as assumptions only.
- Next step: if approved, generate a state-lifecycle sketch for `Ticket.status` and a candidate DDL schema.
