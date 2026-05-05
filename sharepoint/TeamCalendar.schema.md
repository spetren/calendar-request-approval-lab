# TeamCalendar — list schema

Approved personal / team entries (PTO, OOO).

| Column      | Type                | Notes                                |
| ----------- | ------------------- | ------------------------------------ |
| Title       | Single line of text | Mirrors `CalendarRequests.Subject`   |
| EntryType   | Choice              | `PTO`, `OOO`                         |
| StartTime   | Date and time       |                                      |
| EndTime     | Date and time       |                                      |
| Person      | Person or Group     | The requestor                        |
| SourceId    | Number              | Originating `CalendarRequests` ID    |
