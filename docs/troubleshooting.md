# Troubleshooting

## Agent says "done" but nothing appears anywhere

The request may be stuck at one of three stages. Check each:

### 1. Did `CreateCalendarRequest` run?
Power Automate → My flows → **CreateCalendarRequest** → 28-day run history.
- **No runs** → the agent never called the flow. Check the agent topic / action wiring in Copilot Studio.
- **Failed** → open the run, inspect the failed step. Common causes: required column missing, choice column value not in allowed set.

### 2. Is the request in `CalendarRequests`?
Open the SharePoint list. New items should have `Status = Pending`.

### 3. Did `CalendarRequestApproval` run?
Power Automate → My flows → **CalendarRequestApproval** → 28-day run history.
- **Running** → waiting on approver. Approver should check Outlook / Approvals app.
- **Succeeded** → open the run, look at the **Switch** action to confirm it routed to the right calendar list.
- **Failed** → inspect the failed step (often a permissions issue on the destination list).

## Agent doesn't pick up dates correctly

Add explicit slot prompts in the topic ("What date does it start?") and use Copilot Studio's date entity. Validate parsed values before calling the flow.

## Approval emails not arriving

- Confirm the approver is licensed for Approvals.
- Check that the assignee email in the flow is correct.
- Inspect the run — the *Start an approval* action will show whether the request was sent.

## Cross-tenant connector errors

If your agent and your data live in different tenants, you need a **Custom Connector** with appropriate auth. The simplest fix is to put both in the same environment.
