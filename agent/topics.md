# Topics

## File a calendar request (single topic, branches by type)

**Trigger phrases**

- "book pto"
- "i'll be out"
- "schedule an event"
- "add a company event"
- "out of office"
- "request time off"

**Slots**

| Slot        | Entity                | Prompt                                                  |
| ----------- | --------------------- | ------------------------------------------------------- |
| RequestType | Choice (PTO/OOO/Event/Company) | "What kind of calendar entry is this?"         |
| Subject     | Text                  | "What's the subject?"                                   |
| StartTime   | Date / DateTime       | "When does it start?"                                   |
| EndTime     | Date / DateTime       | "When does it end?"                                     |
| Notes       | Text (optional)       | "Any notes? (you can say *no*)"                         |

**Confirm step**

> "Filing a **{RequestType}** request: *{Subject}* from **{StartTime}** to **{EndTime}**. Notes: {Notes}. Submit?"

If user confirms, call action **CreateCalendarRequest** with:

```
RequestType = RequestType
Subject     = Subject
StartTime   = StartTime
EndTime     = EndTime
Requestor   = User.Email
Notes       = Notes
```

**Final message**

> "Submitted. Request ID **{action.RequestId}** — pending approval. You'll be notified once it's approved."

## Help / fallback

If the agent doesn't recognize the intent, route to a help topic that lists the four supported request types and asks which one the user wants.
