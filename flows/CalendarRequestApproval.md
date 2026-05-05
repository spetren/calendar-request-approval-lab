# CalendarRequestApproval (automated flow)

Routes a new request through approval and onto the correct destination calendar list.

## Trigger

**SharePoint – When an item is created** in `CalendarRequests`.

## Steps

1. **Start and wait for an approval**
   - Approval type: *Approve/Reject – First to respond*
   - Title: `concat('Calendar request: ', triggerOutputs()?['body/Title'])`
   - Assigned to: `<APPROVER_EMAIL>` (configure)
   - Details: include `RequestType`, `Subject`, `StartTime`, `EndTime`, `Requestor`, `Notes`

2. **Condition** — `outcome == 'Approve'`

   **If yes:**

   3. **Switch** on `triggerOutputs()?['body/RequestType/Value']`:

      - **Case `PTO`** or **Case `OOO`** → *Create item* in `TeamCalendar`
        ```
        Title     = triggerOutputs()?['body/Subject']
        EntryType = triggerOutputs()?['body/RequestType/Value']
        StartTime = triggerOutputs()?['body/StartTime']
        EndTime   = triggerOutputs()?['body/EndTime']
        Person    = triggerOutputs()?['body/Requestor/Claims']
        SourceId  = triggerOutputs()?['body/ID']
        ```
      - **Case `Event`** → *Create item* in `EventCalendar` (mirrors above; `Owner` instead of `Person`)
      - **Case `Company`** → *Create item* in `CompanyCalendar` (mirrors above; `Audience` defaulted to `All`)
      - **Default** → terminate as failed with message "Unknown RequestType"

   4. **Update item** in `CalendarRequests`:
      - Status = `Approved`
      - ApproverNote = approver comments

   **If no:**

   3. **Update item** in `CalendarRequests`:
      - Status = `Rejected`
      - ApproverNote = approver comments

## Notes

- Configure the approver in step 1 — keep it as a flow parameter or environment variable so the export is portable.
- For multi-stage approval (e.g. manager + facilities), chain additional approval actions before the Switch.
