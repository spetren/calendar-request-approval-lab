# Setup Guide

End-to-end provisioning for the Calendar Request Approval lab.

## Prerequisites

- A Microsoft 365 tenant where you can create SharePoint sites, Power Automate flows, and Copilot Studio agents
- Two test users: a **requestor** and an **approver**
- Power Automate Premium or appropriate license for approvals
- Copilot Studio license

## 1. Provision SharePoint lists

Create the following lists at the root site (or any site of your choosing). Schemas are in [`sharepoint/`](../sharepoint/).

| List              | Purpose                                |
| ----------------- | -------------------------------------- |
| CalendarRequests  | Incoming requests from the agent       |
| TeamCalendar      | Approved team / personal entries       |
| EventCalendar     | Approved project / event entries       |
| CompanyCalendar   | Approved org-wide entries              |

Apply the column definitions in:

- [sharepoint/CalendarRequests.schema.md](../sharepoint/CalendarRequests.schema.md)
- [sharepoint/TeamCalendar.schema.md](../sharepoint/TeamCalendar.schema.md)
- [sharepoint/EventCalendar.schema.md](../sharepoint/EventCalendar.schema.md)
- [sharepoint/CompanyCalendar.schema.md](../sharepoint/CompanyCalendar.schema.md)

## 2. Build Power Automate flows

### CreateCalendarRequest (instant / manual trigger)

- Trigger: **Manually trigger a flow** with inputs:
  `RequestType`, `Subject`, `StartTime`, `EndTime`, `Requestor`, `Notes`
- Action: **SharePoint → Create item** in `CalendarRequests` mapping inputs to columns
- Set `Status = Pending`
- Return the new item ID

See [flows/CreateCalendarRequest.md](../flows/CreateCalendarRequest.md).

### CalendarRequestApproval (automated)

- Trigger: **When an item is created** in `CalendarRequests`
- Action: **Start and wait for an approval** (assign to approver)
- Condition: response is *Approve*
- **Switch** on `RequestType`:
  - `PTO` / `OOO` → Create item in `TeamCalendar`
  - `Event`       → Create item in `EventCalendar`
  - `Company`     → Create item in `CompanyCalendar`
- Update `CalendarRequests` item with `Status = Approved` (or `Rejected`)

See [flows/CalendarRequestApproval.md](../flows/CalendarRequestApproval.md).

## 3. Build the Copilot Studio agent

- Create a new agent named e.g. **Calendar Management Agent**
- Add instructions from [agent/instructions.md](../agent/instructions.md)
- Add a **Topic** that calls `CreateCalendarRequest` with extracted slots
- Connect the flow as an action (Power Automate connector)
- Publish the agent

See [agent/README.md](../agent/README.md).

## 4. Test

1. Open the agent's test pane.
2. Say: *"Book PTO next Friday"*.
3. Confirm a row appears in `CalendarRequests` with `Status = Pending`.
4. The approver receives an approval task — approve it.
5. Confirm the item lands in `TeamCalendar`.

## 5. Troubleshooting

See [docs/troubleshooting.md](troubleshooting.md).
