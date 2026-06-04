# US-001 Ticket Submission Schema Design

## Database Schema Design

This document translates the US-001 domain model into a production-ready relational schema for HelpHub. It validates entities, relationships, and business rules while providing DDL-ready guidance for PostgreSQL-style deployment.

---

## Assumptions

- `ticket_id` is a stable business identifier formatted as `TKT-YYYYMMDD-XXXXX`.
- `employee_id` originates from the employee directory and is stable enough to be a reference key.
- Email is the primary notification channel.
- `notification_status` on `ticket` is a derived summary state, with authoritative detail stored in `notification` records.
- Attachment limits and allowed file types are application-enforced defaults.
- `status` values follow the workflow: `Open`, `In Progress`, `Resolved`, `Closed`.

## Open Questions

- Should `ticket_id` remain the primary key or should a surrogate `id` be the true PK?
- What is the precise duplicate submission deduplication window and fingerprint strategy?
- Must the schema store manual submitter contact details when directory lookup fails?
- Should `ticket.notification_status` be stored separately or derived from `notification` records?
- Are there additional notification channels beyond email (SMS, in-app)?

## Entity Summary

| Entity | Type |
|---|---|
| `Ticket` | Core |
| `Employee` | Reference |
| `Category` | Lookup |
| `Attachment` | Transaction |
| `Notification` | Transaction |

## Tables

### Table: `category`

| Column | Type | Nullable | Default | Constraints |
|---|---|---|---|---|
| `category_id` | UUID | false | `gen_random_uuid()` | PK |
| `name` | VARCHAR(64) | false |  | UNIQUE |
| `description` | TEXT | true |  |  |
| `created_at` | TIMESTAMP WITH TIME ZONE | false | `NOW()` |  |

### Table: `employee`

| Column | Type | Nullable | Default | Constraints |
|---|---|---|---|---|
| `employee_id` | VARCHAR(64) | false |  | PK |
| `name` | VARCHAR(255) | false |  |  |
| `email` | VARCHAR(320) | false |  | UNIQUE |
| `department` | VARCHAR(128) | true |  |  |
| `directory_source` | VARCHAR(32) | false | `'directory'` |  |
| `created_at` | TIMESTAMP WITH TIME ZONE | false | `NOW()` |  |

### Table: `ticket`

| Column | Type | Nullable | Default | Constraints |
|---|---|---|---|---|
| `id` | UUID | false | `gen_random_uuid()` | PK |
| `ticket_id` | VARCHAR(24) | false |  | UNIQUE |
| `description` | TEXT | false |  |  |
| `category_id` | UUID | false |  | FK -> `category(category_id)` |
| `priority` | `ticket_priority_type` | false |  |  |
| `status` | `ticket_status_type` | false |  |  |
| `created_at` | TIMESTAMP WITH TIME ZONE | false | `NOW()` |  |
| `submitter_employee_id` | VARCHAR(64) | false |  | FK -> `employee(employee_id)` |
| `notification_status` | `ticket_notification_status_type` | false | `'queued'` |  |
| `duplicate_key` | VARCHAR(128) | true |  | UNIQUE WHERE `duplicate_key` IS NOT NULL |
| `needs_enrichment` | BOOLEAN | false | `FALSE` |  |

### Table: `attachment`

| Column | Type | Nullable | Default | Constraints |
|---|---|---|---|---|
| `attachment_id` | UUID | false | `gen_random_uuid()` | PK |
| `ticket_id` | UUID | false |  | FK -> `ticket(id)` |
| `filename` | VARCHAR(255) | false |  |  |
| `content_type` | VARCHAR(100) | false |  |  |
| `size_bytes` | INTEGER | false |  | `CHECK (size_bytes > 0 AND size_bytes <= 10485760)` |
| `created_at` | TIMESTAMP WITH TIME ZONE | false | `NOW()` |  |

### Table: `notification`

| Column | Type | Nullable | Default | Constraints |
|---|---|---|---|---|
| `notification_id` | UUID | false | `gen_random_uuid()` | PK |
| `ticket_id` | UUID | false |  | FK -> `ticket(id)` |
| `to_email` | VARCHAR(320) | false |  |  |
| `status` | `notification_status_type` | false |  |  |
| `attempts` | INTEGER | false | `0` | `CHECK (attempts >= 0)` |
| `last_attempt_at` | TIMESTAMP WITH TIME ZONE | true |  |  |
| `created_at` | TIMESTAMP WITH TIME ZONE | false | `NOW()` |  |

## Relationships

| Source | Target | Type | Cardinality |
|---|---|---|---|
| `Employee` | `Ticket` | one-to-many | `Employee(1)` → `Ticket(0..*)` |
| `Category` | `Ticket` | one-to-many | `Category(1)` → `Ticket(0..*)` |
| `Ticket` | `Attachment` | one-to-many | `Ticket(1)` → `Attachment(0..*)` |
| `Ticket` | `Notification` | one-to-many | `Ticket(1)` → `Notification(0..*)` |

## Constraints

### Primary Keys

- `category(category_id)`
- `employee(employee_id)`
- `ticket(id)`
- `attachment(attachment_id)`
- `notification(notification_id)`

### Foreign Keys

- `ticket(category_id)` → `category(category_id)`
- `ticket(submitter_employee_id)` → `employee(employee_id)`
- `attachment(ticket_id)` → `ticket(id)`
- `notification(ticket_id)` → `ticket(id)`

### Unique Constraints

- `category(name)`
- `employee(email)`
- `ticket(ticket_id)`
- `ticket(duplicate_key)` where `duplicate_key` IS NOT NULL

### Check Constraints

- `ticket.ticket_id` follows the pattern `TKT-YYYYMMDD-XXXXX`
- `ticket.priority` values are restricted to `Low`, `Medium`, `High`, `Critical`
- `ticket.status` values are restricted to `Open`, `In Progress`, `Resolved`, `Closed`
- `ticket.notification_status` values are restricted to `queued`, `sent`, `failed`
- `notification.status` values are restricted to `queued`, `sent`, `failed`
- `attachment.size_bytes` is positive and at most 10 MB

## Junction Tables

- No many-to-many junction tables are required for the current US-001 domain model.

## Normalization Review

### 1NF

- Satisfied: all columns store atomic values and there are no repeating groups.

### 2NF

- Satisfied: non-key columns depend on the whole primary key of each table.

### 3NF

- Satisfied with one caveat: `ticket.notification_status` is a derived summary field.
  - Recommendation: remove it or keep it synchronized with `notification` records via application logic or a trigger.

### BCNF

- Satisfied under current schema and key selection.

## Index Strategy

| Table | Index | Reason |
|---|---|---|
| `ticket` | `ticket(ticket_id)` | Fast business lookup by ticket number |
| `ticket` | `ticket(submitter_employee_id)` | Query tickets by submitter |
| `ticket` | `ticket(category_id)` | Filter by category |
| `ticket` | `ticket(status)` | Dashboard filtering by status |
| `ticket` | `ticket(duplicate_key)` WHERE `duplicate_key` IS NOT NULL | Efficient duplicate detection |
| `notification` | `notification(ticket_id)` | Retrieve notifications for a ticket |
| `notification` | `notification(ticket_id, status)` | Support retry and failure reporting |
| `attachment` | `attachment(ticket_id)` | Fetch attachments by ticket |
| `employee` | `employee(email)` | Enforce email uniqueness and lookup submitter |
| `category` | `category(name)` | Lookup category values quickly |

## Audit Columns

Recommended audit fields:

- `created_at` on all tables
- `updated_at` on mutable tables such as `ticket`, `attachment`, and `notification`
- `created_by` and `updated_by` where actor tracking is required

## Soft Delete Strategy

Recommended for recoverability:

- Add `is_deleted BOOLEAN DEFAULT FALSE` and `deleted_at TIMESTAMP WITH TIME ZONE` to `ticket`
- Optionally add soft delete columns to `attachment`
- Preserve notifications for audit/history rather than soft delete them

## Scalability Recommendations

- Expect initial system growth to be small to medium.
- For high volume, partition `ticket` by `created_at` or `status`.
- Archive resolved/closed tickets to read-only or archive tables for reporting.
- Keep `notification` and `attachment` metadata separate from core ticket reads.
- Avoid storing large file blobs in the database; keep attachment content external if possible.

## DDL Skeleton

```sql
CREATE TYPE ticket_priority_type AS ENUM ('Low', 'Medium', 'High', 'Critical');
CREATE TYPE ticket_status_type AS ENUM ('Open', 'In Progress', 'Resolved', 'Closed');
CREATE TYPE ticket_notification_status_type AS ENUM ('queued', 'sent', 'failed');
CREATE TYPE notification_status_type AS ENUM ('queued', 'sent', 'failed');

CREATE TABLE category (
    category_id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    name VARCHAR(64) NOT NULL UNIQUE,
    description TEXT,
    created_at TIMESTAMP WITH TIME ZONE NOT NULL DEFAULT NOW()
);

CREATE TABLE employee (
    employee_id VARCHAR(64) PRIMARY KEY,
    name VARCHAR(255) NOT NULL,
    email VARCHAR(320) NOT NULL UNIQUE,
    department VARCHAR(128),
    directory_source VARCHAR(32) NOT NULL DEFAULT 'directory',
    created_at TIMESTAMP WITH TIME ZONE NOT NULL DEFAULT NOW()
);

CREATE TABLE ticket (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    ticket_id VARCHAR(24) NOT NULL UNIQUE,
    description TEXT NOT NULL,
    category_id UUID NOT NULL REFERENCES category(category_id),
    priority ticket_priority_type NOT NULL,
    status ticket_status_type NOT NULL,
    created_at TIMESTAMP WITH TIME ZONE NOT NULL DEFAULT NOW(),
    submitter_employee_id VARCHAR(64) NOT NULL REFERENCES employee(employee_id),
    notification_status ticket_notification_status_type NOT NULL DEFAULT 'queued',
    duplicate_key VARCHAR(128),
    needs_enrichment BOOLEAN NOT NULL DEFAULT FALSE,
    CHECK (ticket_id ~ '^TKT-[0-9]{8}-[0-9]{5}$')
);

CREATE UNIQUE INDEX ticket_duplicate_key_uq
    ON ticket (duplicate_key)
    WHERE duplicate_key IS NOT NULL;

CREATE INDEX ticket_submitter_idx ON ticket (submitter_employee_id);
CREATE INDEX ticket_category_idx ON ticket (category_id);
CREATE INDEX ticket_status_idx ON ticket (status);

CREATE TABLE attachment (
    attachment_id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    ticket_id UUID NOT NULL REFERENCES ticket(id),
    filename VARCHAR(255) NOT NULL,
    content_type VARCHAR(100) NOT NULL,
    size_bytes INT NOT NULL CHECK (size_bytes > 0 AND size_bytes <= 10485760),
    created_at TIMESTAMP WITH TIME ZONE NOT NULL DEFAULT NOW()
);

CREATE INDEX attachment_ticket_idx ON attachment (ticket_id);

CREATE TABLE notification (
    notification_id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    ticket_id UUID NOT NULL REFERENCES ticket(id),
    to_email VARCHAR(320) NOT NULL,
    status notification_status_type NOT NULL,
    attempts INT NOT NULL DEFAULT 0 CHECK (attempts >= 0),
    last_attempt_at TIMESTAMP WITH TIME ZONE,
    created_at TIMESTAMP WITH TIME ZONE NOT NULL DEFAULT NOW()
);

CREATE INDEX notification_ticket_idx ON notification (ticket_id);
CREATE INDEX notification_ticket_status_idx ON notification (ticket_id, status);
```

## Risks and Concerns

- `ticket.notification_status` is derived and may become inconsistent if not synchronized with `notification` records.
- `duplicate_key` depends on a consistent deduplication fingerprint; algorithm changes can cause false duplicates or missed duplicates.
- Using `ticket_id` as a visible business identifier is valuable, but the schema uses a surrogate `id` for relational joins and performance.
- The `employee` table may need an alternate fallback path for manual contact entry when directory lookup fails.
- Attachment validation is best enforced in the application layer; the database can only enforce size and metadata constraints.

## Agent Description

The Schema Designer agent translates domain model output into a normalized relational schema design, producing table definitions, constraints, normalization analysis, indexing strategy, and implementation-ready DDL guidance.
