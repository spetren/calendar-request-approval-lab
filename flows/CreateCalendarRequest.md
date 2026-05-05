# CreateCalendarRequest (instant flow)

Called by the Copilot Studio agent to insert a new request into the `CalendarRequests` list.

## Trigger

**Manually trigger a flow** — inputs:

| Name        | Type    | Required |
| ----------- | ------- | -------- |
| RequestType | Text    | yes      |
| Subject     | Text    | yes      |
| StartTime   | Date    | yes      |
| EndTime     | Date    | yes      |
| Requestor   | Email   | yes      |
| Notes       | Text    | no       |

## Steps

1. **Compose – Title** — `concat(triggerBody()['RequestType'], ' – ', triggerBody()['Subject'])`
2. **SharePoint – Create item** in `CalendarRequests`:
   - Title = output of step 1
   - RequestType = inputs.RequestType
   - Subject = inputs.Subject
   - StartTime = inputs.StartTime
   - EndTime = inputs.EndTime
   - Requestor Claims = `i:0#.f|membership|{inputs.Requestor}`
   - Notes = inputs.Notes
   - Status Value = `Pending`
3. **Respond to Power App or flow** — return:
   - RequestId = `outputs('Create_item')?['body/ID']`
   - Status = `Pending`

## Notes

- The agent passes `Requestor` as the email of the user that initiated the chat (Copilot Studio variable `User.Email`).
- Date inputs come from the agent's date entity. Convert to ISO 8601 if needed.
