# Flow Diagram

End-to-end request lifecycle. GitHub renders Mermaid natively.

## Key behaviors

- **Only `PTO` requires approval.** All other request types (`OOO`, `Event`, `Company`) are written directly to their destination calendar.
- **Every approved request produces two artifacts:** a row in the appropriate **SharePoint calendar list** AND a corresponding **Outlook calendar event** (created via the Office 365 Outlook connector).

## High-level sequence

```mermaid
sequenceDiagram
    autonumber
    actor U as User
    participant A as Copilot Studio<br/>Agent
    participant F1 as CreateCalendarRequest<br/>(instant flow)
    participant L as CalendarRequests<br/>(SharePoint list)
    participant F2 as CalendarRequestApproval<br/>(automated flow)
    actor AP as Approver
    participant SP as Destination SharePoint list<br/>(Team / Event / Company)
    participant OL as Outlook Calendar

    U->>A: "Book PTO next Friday"
    A->>A: Extract slots<br/>(type, subject, start, end, notes)
    A->>U: Confirm summary
    U->>A: Yes
    A->>F1: Invoke with slots
    F1->>L: Create item (Status = Pending)
    F1-->>A: RequestId
    A-->>U: "Submitted"
    L-->>F2: Item-created trigger
    alt RequestType == PTO
        F2->>AP: Start approval
        AP-->>F2: Approve / Reject
        alt Approved
            F2->>SP: Create item in TeamCalendar
            F2->>OL: Create Outlook event
            F2->>L: Status = Approved
        else Rejected
            F2->>L: Status = Rejected
        end
    else RequestType == OOO / Event / Company
        F2->>SP: Create item in destination list
        F2->>OL: Create Outlook event
        F2->>L: Status = Approved (auto)
    end
```

## State / decision flow

```mermaid
flowchart TD
    Start([User chats with agent]) --> Extract[Agent extracts<br/>RequestType, Subject,<br/>StartTime, EndTime, Notes]
    Extract --> Confirm{User confirms?}
    Confirm -- No --> Start
    Confirm -- Yes --> CreateFlow[CreateCalendarRequest flow]
    CreateFlow --> Inbox[(CalendarRequests<br/>Status = Pending)]
    Inbox --> Trigger[CalendarRequestApproval<br/>item-created trigger]
    Trigger --> NeedsApproval{RequestType<br/>== PTO ?}

    NeedsApproval -- No<br/>OOO / Event / Company --> Switch
    NeedsApproval -- Yes<br/>PTO --> Approval{Approver<br/>decision}
    Approval -- Reject --> Reject[Update Status = Rejected]
    Reject --> EndR([End])
    Approval -- Approve --> Switch{Switch on<br/>RequestType}

    Switch -- PTO / OOO --> Team[(TeamCalendar)]
    Switch -- Event --> Event[(EventCalendar)]
    Switch -- Company --> Company[(CompanyCalendar)]

    Team --> Outlook[Create Outlook event<br/>via O365 Outlook connector]
    Event --> Outlook
    Company --> Outlook

    Outlook --> Approved[Update Status = Approved]
    Approved --> EndA([End])

    classDef store fill:#e8f0ff,stroke:#3b6fb6,color:#0b2545;
    classDef flow fill:#fff4e0,stroke:#c97e10,color:#5a3a00;
    classDef decision fill:#f3e8ff,stroke:#7e3fc8,color:#3b125f;
    classDef outlook fill:#e0f7e9,stroke:#2e8b57,color:#0b3d20;
    class Inbox,Team,Event,Company store;
    class CreateFlow,Trigger flow;
    class NeedsApproval,Confirm,Approval,Switch decision;
    class Outlook outlook;
```
