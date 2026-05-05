# CalendarRequests — list schema

The inbox for all incoming requests.

| Column        | Type                          | Notes                                              |
| ------------- | ----------------------------- | -------------------------------------------------- |
| Title         | Single line of text (default) | Short summary, e.g. "PTO – next Friday"            |
| RequestType   | Choice                        | `PTO`, `OOO`, `Event`, `Company`                   |
| Subject       | Single line of text           | Free-form subject from the requestor               |
| StartTime     | Date and time                 | Event start                                        |
| EndTime       | Date and time                 | Event end                                          |
| Requestor     | Person or Group               | Defaults to the user who triggered the agent       |
| Notes         | Multiple lines of text        | Optional context                                   |
| Status        | Choice                        | `Pending` (default), `Approved`, `Rejected`        |
| ApproverNote  | Multiple lines of text        | Filled by the approval flow                        |
