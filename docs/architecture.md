# Architecture

## Data flow

1. **User → Agent** — natural-language request: "Book me PTO next Friday".
2. **Agent → Flow** — the Copilot Studio topic extracts slots (`RequestType`, `Subject`, `StartTime`, `EndTime`, `Requestor`, `Notes`) and invokes `CreateCalendarRequest` via the Power Automate connector.
3. **Flow → SharePoint** — `CreateCalendarRequest` creates a row in `CalendarRequests` with `Status = Pending`.
4. **Trigger fires** — `CalendarRequestApproval` (item-created trigger) picks up the new row.
5. **Approval** — the flow assigns an approval task to the configured approver and waits.
6. **Dispatch** — on approve, a `Switch` on `RequestType` writes to `TeamCalendar`, `EventCalendar`, or `CompanyCalendar`. On reject, only `CalendarRequests.Status` is updated.

## Why this shape?

- **Single inbox (`CalendarRequests`)** — easy to audit and report on. All requests, regardless of type or outcome, are visible in one place.
- **Separate destination lists** — different audiences, different permissions, different views (e.g. company calendar visible to all; team calendar scoped to a group).
- **Approval in flow, not in agent** — keeps the conversational layer thin and makes the policy auditable.
- **Switch by `RequestType`** — adding a new calendar type is a one-branch change, no agent retraining required.

## Extensibility

- **More calendar types** — add a list, add a `Switch` case, add a value to the `RequestType` choice column.
- **Conditional approvers** — replace the static approver with a lookup (e.g. manager via Office 365 Users connector).
- **Notifications** — add post-approval email/Teams steps after each `Create item`.
- **Outlook sync** — add a parallel branch that creates an actual Outlook calendar event for the requestor.
