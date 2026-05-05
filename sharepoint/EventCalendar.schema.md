# EventCalendar — list schema

Approved project / event entries.

| Column      | Type                   | Notes                                |
| ----------- | ---------------------- | ------------------------------------ |
| Title       | Single line of text    | Event name                           |
| StartTime   | Date and time          |                                      |
| EndTime     | Date and time          |                                      |
| Location    | Single line of text    | Optional                             |
| Notes       | Multiple lines of text | Optional                             |
| Owner       | Person or Group        | The requestor                        |
| SourceId    | Number                 | Originating `CalendarRequests` ID    |
