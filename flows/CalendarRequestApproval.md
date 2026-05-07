# CalendarRequestApproval (automated flow)

Routes a new request through approval (PTO only) and onto the correct destination calendar list, then creates a matching Outlook event.

## Trigger

**SharePoint – When an item is created** in `CalendarRequests`.

## Steps

1. **Condition – needs approval** — `triggerOutputs()?['body/RequestType/Value'] == 'PTO'`

   **If yes (PTO):**

   1a. **Start and wait for an approval**
      - Approval type: *Approve/Reject – First to respond*
      - Title: `concat('Calendar request: ', triggerOutputs()?['body/Title'])`
      - Assigned to: `<APPROVER_EMAIL>` (configure)
      - Details: include `RequestType`, `Subject`, `StartTime`, `EndTime`, `Requestor`, `Notes`

   1b. **Condition** — `outcome == 'Approve'`
      - On **Reject**: update `CalendarRequests` Status = `Rejected` with comments → **terminate**.
      - On **Approve**: continue to step 2.

   **If no (OOO / Event / Company):** continue directly to step 2 (no approval needed).

2. **Switch** on `triggerOutputs()?['body/RequestType/Value']`:

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

3. **Create event (V4)** — *Office 365 Outlook* connector
   - Calendar id: requestor's default calendar (or a shared mailbox for `Company` requests)
   - Subject: `triggerOutputs()?['body/Subject']`
   - Start / End time: from the trigger
   - Time zone: tenant default
   - Body: `concat('Filed via Calendar Management Agent. Source request #', triggerOutputs()?['body/ID'])`
   - Required attendees: requestor (and any others derived from `Notes` if you parse them)

4. **Update item** in `CalendarRequests`:
   - Status = `Approved`
   - ApproverNote = approver comments (or "Auto-approved" for non-PTO types)

## Notes

- Configure the approver in step 1a — keep it as a flow parameter or environment variable so the export is portable.
- Step 3 (Outlook) is what makes the entry visible in the user's actual Outlook calendar in addition to the SharePoint list. If you want the Outlook event to appear before approval (tentative), insert it before step 1.
- For multi-stage approval (e.g. manager + facilities), chain additional approval actions inside the PTO branch.
