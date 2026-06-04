# IT Helpdesk Tracking System - User Stories

**Project:** Internal IT Support Ticketing System  
**Date Created:** June 3, 2026  
**Prepared for:** Development Team  

---

## Overview

This document contains 5 developer-ready user stories for the IT Helpdesk Tracking System. Each story is scoped to be deliverable within a standard sprint and follows INVEST principles (Independent, Negotiable, Valuable, Estimable, Small, Testable).

---

## US-001: Employee Ticket Submission

### Summary
Employees need the ability to create IT support tickets through a simple web form with essential information (category, priority, description) so that IT staff can receive and track their requests.

### User Story Statement
As an **employee**, I want to **submit an IT support ticket with a category, priority level, and detailed description**, so that **my IT issues are properly documented and tracked by the IT team**.

### Acceptance Criteria

**Scenario 1: Employee Successfully Creates a Ticket**
- **Given** an employee is on the "Create Ticket" page
- **When** the employee fills in all required fields (category, priority, description) and clicks "Submit"
- **Then** the system creates a ticket and displays a confirmation message with a unique ticket ID

**Scenario 2: Validation on Ticket Form**
- **Given** an employee has partially filled out the form
- **When** the employee clicks "Submit" with missing required fields
- **Then** the system displays validation errors and prevents submission

**Scenario 3: Category and Priority Selection**
- **Given** the ticket form is loaded
- **When** the employee clicks the category dropdown
- **Then** the system displays predefined categories (Hardware, Software, Network, Account Access, Other)

**Scenario 4: Ticket Confirmation and Reference**
- **Given** an employee has submitted a ticket
- **When** the ticket is successfully created
- **Then** the employee receives a confirmation with ticket number, submission timestamp, and expected response timeframe

**Scenario 5: Unauthenticated or Unauthorized Submission**
- **Given** a visitor is not authenticated or lacks permission to submit
- **When** they attempt to submit the ticket form
- **Then** the system blocks submission, returns an authentication/authorization error, and prompts to sign in or request access

**Scenario 6: Employee Directory Unavailable / Auto-population Fallback**
- **Given** the employee directory service is unreachable when loading the form
- **When** the employee attempts to submit
- **Then** the form allows manual entry of name/contact, flags the submission for later enrichment, and logs the directory outage for operators

**Scenario 7: Invalid Input and Attachment Limits**
- **Given** the employee attaches a file exceeding allowed size or unsupported type
- **When** they click Submit
- **Then** the system rejects the attachment with a clear validation error and prevents submission until corrected

**Scenario 8: Duplicate or Rapid Submissions (Rate Limit)**
- **Given** the employee accidentally clicks Submit multiple times or scripts submit repeatedly
- **When** duplicate requests are received within a short window
- **Then** the system detects duplicates by comparing payload and timestamps, prevents duplicate ticket creation, and returns idempotent confirmation for the first submission

**Scenario 9: Email Notification Failure Handling**
- **Given** the ticket is created successfully but the email notification service fails
- **When** the notification attempt fails
- **Then** the system queues the notification for retry, logs the failure, and marks the ticket's notification status for operator review if retries fail

### Dependencies & Notes
- Category list must be pre-defined by IT team
- Priority options: Low, Medium, High, Critical
- Email notifications should be triggered upon ticket creation (see US-005 for integration point)
- Integration with employee directory for auto-population of employee name and contact info

### Definition of Done
- [ ] Ticket submission form created with all required fields
- [ ] Form validation implemented for required fields
- [ ] Unique ticket ID generation working (Format: TKT-YYYYMMDD-XXXXX)
- [ ] Confirmation page displayed with ticket details
- [ ] Ticket data persisted to database
- [ ] Unit tests cover all validation scenarios
- [ ] Form styling matches design specifications
- [ ] Accessibility requirements met (WCAG 2.1 AA)
- [ ] Tested in Chrome, Firefox, Safari, Edge

---

## US-002: IT Staff Ticket Management Dashboard

### Summary
IT staff need a centralized dashboard to view all support tickets, assign owners, update ticket status, and resolve issues so they can efficiently manage workload and prioritize support activities.

### User Story Statement
As an **IT support staff member**, I want to **view all tickets, filter by status, assign tickets to team members, and update ticket status**, so that **I can efficiently manage support requests and ensure timely resolution**.

### Acceptance Criteria

**Scenario 1: View All Tickets in Dashboard**
- **Given** an IT staff member logs into the system
- **When** they access the Ticket Dashboard
- **Then** the system displays a table with all open tickets showing: Ticket ID, Submitter, Category, Priority, Status, and Created Date

**Scenario 2: Assign Ticket to Team Member**
- **Given** an IT staff member is viewing a ticket in detail
- **When** they click "Assign" and select a team member from the dropdown
- **Then** the system updates the ticket ownership and notifies the assigned person via email

**Scenario 3: Update Ticket Status**
- **Given** an IT staff member is viewing a ticket
- **When** they change the status from one value to another (Open → In Progress → Resolved → Closed)
- **Then** the system updates the status, records a timestamp, and maintains an audit trail of the change

**Scenario 4: Add Internal Notes**
- **Given** an IT staff member is working on a ticket
- **When** they add internal notes and click "Save"
- **Then** the notes are stored and visible only to IT staff (not to the employee)

**Scenario 5: Bulk Actions**
- **Given** an IT staff member selects multiple tickets
- **When** they perform a bulk action (e.g., assign multiple tickets, change status)
- **Then** the system applies the action to all selected tickets and confirms completion

**Scenario 6: Unauthorized Access Attempt**
- **Given** a user without IT Staff role attempts to access the dashboard or perform assignment actions
- **When** they attempt the action
- **Then** the system denies access and logs the attempt for audit/review

**Scenario 7: Assignment Failures and Offline Assignee**
- **Given** the selected assignee is not available in the team directory or email notifications cannot be delivered
- **When** the IT staff assigns a ticket
- **Then** the system provides a clear error, suggests alternative assignees, and allows assignment to a fallback queue

**Scenario 8: Bulk Action Partial Failures**
- **Given** a bulk action affecting many tickets (100+)
- **When** an error occurs for a subset of tickets (e.g., validation or permission error)
- **Then** the system applies the action for the successful subset, reports the failed items with reasons, and provides retry or remediation options

**Scenario 9: Performance at Scale**
- **Given** the system contains >1000 tickets and multiple staff are concurrently accessing the dashboard
- **When** an IT staff loads the dashboard and performs common operations
- **Then** the system responds within performance targets (list load <2s) and maintains consistent pagination/filter behavior

### Dependencies & Notes
- Requires user authentication and role-based access control (Admin, IT Staff, Employee roles)
- Team member list must be maintained in system
- Status workflow: Open → In Progress → Resolved → Closed
- Internal notes should not be visible to employees
- Dependent on US-001 for ticket creation data

### Definition of Done
- [ ] Dashboard displays all tickets with required columns
- [ ] Ticket detail view implemented
- [ ] Assignment functionality working with email notifications
- [ ] Status update workflow implemented with audit trail
- [ ] Internal notes feature created
- [ ] Bulk action selection and processing working
- [ ] Role-based access control enforced (IT Staff only)
- [ ] Unit and integration tests for status updates and assignments
- [ ] Performance tested with 1000+ tickets
- [ ] Tested across specified browsers

---

## US-003: Real-Time Ticket Status Visibility

### Summary
Both employees and IT staff need real-time visibility of ticket status updates so they know the current state of support requests without manual polling or page refreshes.

### User Story Statement
As an **employee or IT staff member**, I want to **see real-time updates to ticket status without refreshing the page**, so that **I have immediate visibility into ticket progress and can make informed decisions**.

### Acceptance Criteria

**Scenario 1: Employee Views Ticket Status in Real-Time**
- **Given** an employee is on their ticket detail page
- **When** an IT staff member updates the ticket status or adds a comment
- **Then** the employee sees the update immediately without page refresh (within 2 seconds)

**Scenario 2: Status Change Notification**
- **Given** a ticket is being actively viewed
- **When** the ticket status changes to "Resolved"
- **Then** the system highlights the change and displays a notification banner

**Scenario 3: Multiple Users Viewing Same Ticket**
- **Given** multiple IT staff members are viewing the same ticket
- **When** one staff member updates the ticket status
- **Then** all other viewers see the update in real-time

**Scenario 4: WebSocket Connection Resilience**
- **Given** a real-time connection is active
- **When** network connectivity is temporarily lost and restored
- **Then** the system automatically reconnects and syncs any missed updates

**Scenario 5: Unauthorized User Viewing Real-Time Updates**
- **Given** a user without proper permissions (e.g., employee viewing another employee's ticket detail)
- **When** they attempt to access real-time updates
- **Then** the system enforces authorization, blocks the WebSocket connection, and logs the access attempt

**Scenario 6: Missed Updates and Out-of-Order Message Reconciliation**
- **Given** a client reconnects after a disconnect lasting >30 seconds
- **When** the client resumes the connection
- **Then** the system sends a full state snapshot of the ticket to ensure consistency, rather than relying on buffered messages alone, preventing stale/out-of-order updates

**Scenario 7: Multiple Concurrent Updates and Consistency**
- **Given** multiple staff members edit the same ticket concurrently (status, notes, assignment)
- **When** updates are broadcast in real-time
- **Then** the system ensures eventual consistency (e.g., last-write-wins for status, appended notes), displays clear conflict indicators, and provides audit trail of all updates

### Dependencies & Notes
- Requires WebSocket or Server-Sent Events (SSE) implementation
- Scalability requirement: Support 500+ concurrent users
- Real-time update latency target: < 2 seconds
- Fallback polling mechanism if WebSocket unavailable
- Dependent on US-001, US-002 for ticket creation and status updates

### Definition of Done
- [ ] WebSocket infrastructure implemented (or SSE alternative)
- [ ] Real-time status update mechanism working
- [ ] Real-time notification display for changes
- [ ] Connection resilience and auto-reconnection implemented
- [ ] Fallback polling mechanism as backup
- [ ] Load testing completed (500+ concurrent connections)
- [ ] Latency monitoring in place
- [ ] Unit tests for connection management
- [ ] Integration tests for multi-user scenarios
- [ ] Browser compatibility verified

---

## US-004: Ticket Tracking and SLA Management with Overdue Highlighting

### Summary
The system needs to track time-to-resolution and highlight overdue tickets so IT staff can prioritize urgent items and ensure SLA compliance, while management can monitor resolution performance metrics.

### User Story Statement
As an **IT staff member and IT manager**, I want to **track time spent on tickets, see resolution timelines, and identify overdue tickets**, so that **I can prioritize support activities and ensure SLA compliance**.

### Acceptance Criteria

**Scenario 1: Time Tracking per Ticket**
- **Given** a ticket is in "In Progress" status
- **When** IT staff updates the ticket status to "Resolved"
- **Then** the system calculates and displays time-to-resolution (submission time to resolution time)

**Scenario 2: Overdue Ticket Highlighting**
- **Given** an SLA target is defined (e.g., 24 hours for Critical, 48 hours for High)
- **When** the current time exceeds the SLA deadline for a ticket
- **Then** the ticket is highlighted in red/orange in all dashboards and list views, and a flag displays "Overdue"

**Scenario 3: Time Spent Tracking**
- **Given** an IT staff member is working on a ticket
- **When** they toggle a "Work Timer" or manually log time spent
- **Then** the time is tracked and accumulated on the ticket

**Scenario 4: SLA Status on Ticket Detail**
- **Given** a ticket detail page is open
- **When** the page loads
- **Then** the system displays: Creation date, First response deadline (SLA), and Time remaining (or "Overdue")

**Scenario 5: Historical Resolution Data**
- **Given** a ticket is resolved
- **When** the ticket is archived or moved to closed state
- **Then** the system stores historical data: Total time-to-resolution, number of status changes, and time spent by each team member

**Scenario 6: SLA Exception - Ticket On-Hold or Paused**
- **Given** a ticket is in "On-Hold" or "Paused" status (awaiting customer info or third-party action)
- **When** the ticket is on-hold
- **Then** the SLA clock pauses, the ticket is not highlighted as overdue, and time-to-resolution calculations exclude on-hold duration

**Scenario 7: Manual SLA Override or Extension**
- **Given** an IT manager manually extends an SLA deadline
- **When** the override is applied
- **Then** the new deadline is recorded, the override reason is logged, and the ticket's overdue status recalculates based on the new deadline

**Scenario 8: Timezone Awareness and DST Handling**
- **Given** a ticket is created in timezone A and monitored by staff in timezone B
- **When** Daylight Saving Time transitions occur
- **Then** the system correctly recalculates SLA deadlines and timestamps using UTC internally, displaying localized times to each user without drift or inconsistencies

**Scenario 9: SLA Breach Notification**
- **Given** a ticket exceeds its SLA deadline
- **When** the breach is detected
- **Then** the system sends an alert to the assigned staff and manager, logs the breach event, and updates SLA compliance metrics

### Dependencies & Notes
- SLA configuration must be defined per category and priority (e.g., Critical: 4 hrs, High: 24 hrs, Medium: 48 hrs, Low: 72 hrs)
- System clock must be synchronized and timezone-aware
- Time tracking can be automated (status-based) or manual (timer)
- Dependent on US-001 and US-002 for ticket creation and status management
- Integration with reporting system for analytics

### Definition of Done
- [ ] SLA configuration system created
- [ ] Time-to-resolution calculation implemented
- [ ] Overdue detection and highlighting working
- [ ] Time tracking mechanism (automatic or manual) implemented
- [ ] Ticket detail view displays SLA status
- [ ] Historical data collection and storage working
- [ ] Timezone handling verified
- [ ] Unit tests for time calculations and SLA logic
- [ ] Integration tests for time tracking across status changes
- [ ] Performance verified with large historical datasets

---

## US-005: Filtering and Reporting Dashboard

### Summary
IT managers and staff need comprehensive filtering and reporting capabilities so they can analyze ticket trends, monitor workload distribution, identify bottlenecks, and generate insights for capacity planning and process improvement.

### User Story Statement
As an **IT manager and IT support staff**, I want to **filter tickets by category, priority, status, owner, and date range, and generate reports on ticket metrics**, so that **I can monitor team performance, identify trends, and make data-driven decisions**.

### Acceptance Criteria

**Scenario 1: Multi-Dimensional Filtering**
- **Given** the ticket list or dashboard is displayed
- **When** a user applies filters (Category: "Hardware", Priority: "High", Status: "Open", Owner: "John Doe")
- **Then** the system displays only tickets matching all filter criteria

**Scenario 2: Date Range Filtering**
- **Given** the filter panel is open
- **When** a user selects a date range (e.g., Last 7 days, Last 30 days, or custom dates)
- **Then** the system filters tickets by creation date or resolution date within the selected range

**Scenario 3: Saved Filters**
- **Given** a user has created a complex filter
- **When** they click "Save Filter" and give it a name
- **Then** the filter is saved and can be reloaded from a dropdown list

**Scenario 4: Summary Report Generation**
- **Given** filters are applied or the user selects "Generate Report"
- **When** the user clicks "Export Report" or "View Summary"
- **Then** the system displays key metrics: Total tickets, Resolved tickets, Overdue tickets, Average resolution time, Tickets by category, Tickets by priority, Workload distribution by team member

**Scenario 5: Performance Dashboard**
- **Given** the performance dashboard is loaded
- **When** the dashboard displays
- **Then** it shows: SLA compliance rate (%), Average time-to-resolution, Tickets resolved today/this week/this month, Team member statistics (tickets assigned, resolved, average time)

**Scenario 6: Export Report to CSV/PDF**
- **Given** a report or filtered view is displayed
- **When** a user clicks "Export to CSV" or "Export to PDF"
- **Then** the system generates and downloads the report file with all displayed data

**Scenario 7: Scheduled Reports**
- **Given** an IT manager is setting up automated reporting
- **When** they configure a report to send weekly or monthly
- **Then** the system generates and emails the report at the scheduled time

**Scenario 8: Export Authorization and PII Redaction**
- **Given** a user exports a report (CSV/PDF) containing ticket data
- **When** the export is initiated
- **Then** the system enforces role-based export permissions (managers can export all data, staff cannot export data beyond their assigned tickets), and redacts or excludes sensitive PII based on user role (e.g., employee contact info hidden from non-managers)

**Scenario 9: Scheduled Report Failure Handling**
- **Given** a scheduled report is due to send
- **When** the report generation or email delivery fails
- **Then** the system logs the failure, retries the delivery up to 3 times, notifies the manager of the failure, and provides a manual retry option

**Scenario 10: Large Dataset Export Performance**
- **Given** a user exports a report spanning 10,000+ tickets
- **When** the export is initiated
- **Then** the system generates the export asynchronously (non-blocking), notifies the user via email when ready, and provides a temporary download link

### Dependencies & Notes
- Reporting requires aggregation of data from US-001, US-002, US-004
- Dashboard should display real-time metrics (refresh every 5-10 minutes minimum)
- Export functionality requires PDF/CSV generation library
- Scheduled reports require background job processing
- Role-based dashboard views (Manager vs. Staff)
- Filtering options depend on ticket schema finalization

### Definition of Done
- [ ] Multi-dimensional filtering UI created
- [ ] Filter logic implemented and tested
- [ ] Saved filters feature working with persistence
- [ ] Summary report generation implemented
- [ ] Performance dashboard with all required metrics
- [ ] Export to CSV and PDF working
- [ ] Scheduled report job configured
- [ ] Metrics calculation and aggregation queries optimized
- [ ] Unit tests for filter logic and report generation
- [ ] Performance tested with large datasets (10,000+ tickets)
- [ ] Dashboard refreshes correctly at configured intervals
- [ ] Tested across browsers and devices

---

## Dependencies Between Stories

```
US-001 (Ticket Submission)
  ↓
US-002 (IT Ticket Management) ← US-001
  ↓
US-003 (Real-Time Visibility) ← US-001, US-002
  ↓
US-004 (SLA & Tracking) ← US-001, US-002
  ↓
US-005 (Filtering & Reporting) ← US-001, US-002, US-004
```

---

## Recommended Sprint Sequencing

1. **Sprint 1:** US-001 (Employee Ticket Submission)
2. **Sprint 2:** US-002 (IT Ticket Management Dashboard)
3. **Sprint 3:** US-004 (SLA Tracking & Overdue Highlighting)
4. **Sprint 4:** US-003 (Real-Time Visibility) + US-005 (Filtering & Reporting)

---

## Success Metrics & Acceptance

The system is considered complete when:

- [ ] All 5 user stories are implemented and tested
- [ ] Employee ticket submission is intuitive (< 2 min to create ticket)
- [ ] IT staff can efficiently manage tickets (load all tickets < 2 sec)
- [ ] Real-time updates work within SLA latency targets (< 2 sec)
- [ ] SLA compliance rate is measurable and tracked
- [ ] All acceptance criteria in Given/When/Then format pass
- [ ] Performance benchmarks are met (1000+ concurrent users)
- [ ] Accessibility and browser compatibility verified
- [ ] Documentation complete for end users and IT staff
- [ ] UAT sign-off from IT leadership

---

## Notes for Development Team

- **Technology Stack Decision:** Awaiting confirmation on frontend framework, backend language, and database choice
- **Authentication:** Integrate with company identity provider (AD/Azure/OAuth)
- **Data Privacy:** Ensure GDPR/compliance requirements are met for ticket data
- **Scalability:** Design for future multi-site/multi-department expansion
- **API-First Approach:** Consider API endpoints for third-party integrations (monitoring tools, etc.)
- **Mobile Access:** Consider responsive design or native mobile app for ticket viewing

---

**End of User Stories Document**
