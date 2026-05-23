# Product Requirements Document
## Weekly Calendar Email Automation
**University of Portland — Marketing Services**
**Version 1.0 | May 2026**

---

## 1. Overview
This document defines the requirements for an automated weekly email system built using Microsoft Power Automate. The system retrieves calendar events from a shared Outlook calendar and delivers a formatted summary email every Monday to the Marketing Services team at the University of Portland.

---

## 2. Problem Statement
The Marketing Services team previously sent weekly time-off and work-from-home (WFH) summaries manually each Monday. This process was:
- Time-consuming for the sender
- Error-prone and inconsistent in format
- Dependent on an individual remembering to send
- Occasionally missed or delayed

---

## 3. Objectives
- Automate the weekly delivery of a time-off and WFH summary email every Monday morning
- Pull event data directly from a shared Marketing Services Outlook calendar
- Present events in a clear, consistently formatted HTML table
- Ensure correct timezone display (Pacific Time — PDT/PST)
- Eliminate manual effort and reduce risk of missed communications
- Support future extension to additional calendars (e.g. Management calendar)

---

## 4. Scope

### 4.1 In Scope
- Weekly recurring trigger (every Monday)
- Outlook Calendar event retrieval via Power Automate (Get Calendar View of Events V3)
- Event filtering for current week (Monday–Sunday, Pacific Time)
- Date sorting in chronological order
- HTML formatted email with 4-column table layout
- Support for all-day events and timed events
- Manual recipient list (specified email addresses)

### 4.2 Out of Scope
- Attendee-based email distribution (planned Phase 4)
- Integration with non-Outlook calendars
- Real-time or on-demand email triggers
- Event creation or modification

---

## 5. Users & Stakeholders

| Role | Name | Involvement |
|------|------|-------------|
| Flow Owner | Swe Zin Ye | Builds and maintains the automation flow |
| Supervisor | Dhara Brown | Reviews output, approves format, email sender identity |
| Recipients | Marketing Team | Receives the weekly summary email every Monday |
| IT Department | University of Portland IT | Manages firewall/Conditional Access for Power Platform |

---

## 6. Functional Requirements

### 6.1 Trigger
- The flow must run automatically every Monday
- Trigger type: **Recurrence** — Frequency: Week, Interval: 1, Day: Monday

### 6.2 Calendar Data Retrieval
- Connector: `Get Calendar View of Events (V3)` — Office 365 Outlook
- Calendar: Marketing Services Office Assistant (shared calendar)
- Start Time: Monday 00:00:00 of current week (Pacific Time)
- End Time: Sunday 23:59:59 of current week (Pacific Time)
- Week calculation uses `dayOfWeek()` offsets (`startOfWeek()` not supported)

### 6.3 Event Filtering
- Filter Array action applied after retrieval
- Events filtered to current week Monday–Sunday using WeekStart and WeekEnd Compose actions
- Filter uses `greaterOrEquals` and `lessOrEquals` on `item()?['start']` field
- Advanced mode expression required (basic mode does not support AND conditions)

```
@and(
  greaterOrEquals(
    formatDateTime(item()?['start'], 'yyyy-MM-dd'),
    outputs('Compose-WeekStart')
  ),
  lessOrEquals(
    formatDateTime(item()?['start'], 'yyyy-MM-dd'),
    outputs('Compose_-_WeekEnd')
  )
)
```

### 6.4 Sorting
- Events sorted chronologically by start date/time
- `Compose(Sort)` action uses `sort(body('Filter_array'), 'start')`
- Select action references Compose(Sort) output as its From source

### 6.5 Data Formatting
- Select action formats each event into an HTML table row
- Four columns: **Day of Week | Date | Time | Description**
- All-day events display `All Day` in the Time column
- Timed events display `hh:mm AM/PM to hh:mm AM/PM` format
- `substring(item()?['start'], 0, 19)` used to trim timestamp for formatting
- `dayOfWeek()` nested if/else converts numeric day to day name

**Select Map Expression:**
```
concat(
  '<tr><td style="width:100px; padding:2px 8px;">',
  [Day of Week],
  '</td><td style="width:150px; padding:2px 8px;">',
  [Date],
  '</td><td style="width:180px; padding:2px 8px;">',
  [Time or All Day],
  '</td><td style="padding:2px 8px;">',
  item()?['subject'],
  '</td></tr>'
)
```

### 6.6 Email Composition
- Compose Join All Lines builds complete HTML table including header row
- `join(body('Select'), '')` concatenates all event rows
- Email body uses HTML with Aptos font, 14px size
- Table headers styled with University of Portland navy color (`#1F3864`)
- **Is HTML = Yes** must be set in Send an email (V2) advanced parameters

### 6.7 Email Delivery
- `Send an email (V2)` action — Office 365 Outlook connector
- **To:** Specified recipient list (manual for current phase)
- **Subject:** `Weekly Time-Off & WFH Summary — [date]`
- **Sender identity:** Dhara Brown (brownd@up.edu)

---

## 7. Non-Functional Requirements

| Requirement | Detail |
|---|---|
| Reliability | Flow runs without manual intervention every Monday |
| Timezone | All times displayed in Pacific Time (PDT/PST auto-adjusted) |
| Compatibility | Email renders correctly in Outlook desktop and web |
| Maintainability | Actions named clearly for easy debugging |
| Scalability | Architecture supports adding additional calendars |

---

## 8. Flow Architecture

| # | Action | Type | Description |
|---|--------|------|-------------|
| 1 | Recurrence | Trigger | Fires every Monday |
| 2 | Compose-WeekStart | Compose | Calculates Monday date of current week |
| 3 | Compose-WeekEnd | Compose | Calculates Sunday date of current week |
| 4 | Get Calendar View (V3) | Outlook Connector | Retrieves events from shared calendar |
| 5 | Filter Array | Data Operations | Filters events to current week only |
| 6 | Compose(Sort) | Compose | Sorts events chronologically by start |
| 7 | Select | Data Operations | Formats each event as HTML table row |
| 8 | Compose Join All Lines | Compose | Builds complete HTML table |
| 9 | Send an email (V2) | Outlook Connector | Delivers formatted email to recipients |

---

## 9. Known Limitations

| Limitation | Workaround |
|---|---|
| `startOfWeek()` not supported | `dayOfWeek()` nested if/else offsets |
| `mul()` / `sub()` not in Filter Array | Nested `if/equals` expressions |
| V3 connector no attendee data | Separate `Get Event (V3)` per event |
| University firewall blocks Power Platform | Edit flows from external network |
| Copied flows lose connection references | Reconnect manually after copying |
| All-day event timezone shift | Use `item()?['start']` without conversion |

---

## 10. Future Enhancements

| Phase | Feature |
|---|---|
| Phase 3 | Extend to Management calendar |
| Phase 4 | Attendee-based email distribution |
| Phase 5 | Dynamic recipient list from SharePoint |
| Phase 6 | Error handling and retry logic |
| Phase 7 | Supervisor approval flow before sending |

---

