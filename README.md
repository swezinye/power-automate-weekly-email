# 📅 Weekly Calendar Email Automation
**Microsoft Power Automate | University of Portland — Marketing Services**

---

## 📌 Overview
An automated weekly email workflow built with Microsoft Power Automate that retrieves shared Outlook calendar events and delivers a formatted HTML summary email every Monday to the Marketing Services team — eliminating manual reporting.

---

## ✨ Features
- 🔄 **Fully automated** — runs every Monday via Recurrence trigger
- 📆 **Smart week detection** — calculates Monday–Sunday boundaries without unsupported functions
- 🕐 **Timezone aware** — handles Pacific Time (PDT/PST)
- 📊 **4-column HTML table** — Day | Date | Time | Description
- 🗂️ **Chronological sorting** — events ordered by start date/time
- ✅ **All-day event support** — displays "All Day" instead of incorrect timestamps
- 📧 **Professional email format** — Aptos font, University of Portland branding

---

## 🏗️ Flow Architecture

| Step | Action | Description |
|------|--------|-------------|
| 1 | Recurrence Trigger | Fires every Monday |
| 2 | Compose - WeekStart | Calculates Monday of current week |
| 3 | Compose - WeekEnd | Calculates Sunday of current week |
| 4 | Get Calendar View of Events (V3) | Retrieves events from shared Outlook calendar |
| 5 | Filter Array | Filters events to current week only |
| 6 | Compose - Sort | Sorts events chronologically by start date |
| 7 | Select | Formats each event as HTML table row |
| 8 | Compose - Join All Lines | Builds complete HTML table |
| 9 | Send an email (V2) | Delivers formatted email to recipients |

---

## 📧 Email Output Format

```
Hello Marketing Team,

Happy Monday!

Please see below for this week's Time-Off/WFH Summary for Marketing Services

| Day       | Date          | Time                    | Description              |
|-----------|---------------|-------------------------|--------------------------|
| Monday    | May 11 2026   | 12:30 PM to 04:00 PM   | WRK | Swe Zin             |
| Wednesday | May 13 2026   | All Day                 | WFH | Sophie Peterson     |
| Friday    | May 15 2026   | All Day                 | WFH | Dhara Brown         |
```

---

## ⚙️ Key Technical Decisions

### Week Boundary Calculation
`startOfWeek()` is not supported in Power Automate. Instead, `dayOfWeek()` with nested `if/else` offsets calculates Monday and Sunday:

```
// Monday offset per day
Sunday(0)=-6, Monday(1)=0, Tuesday(2)=-1, 
Wednesday(3)=-2, Thursday(4)=-3, Friday(5)=-4, Saturday(6)=-5
```

### Timestamp Handling
The V3 connector returns timestamps in format `2026-05-11T12:30:00.0000000`.
`substring(item()?['start'], 0, 19)` trims the microseconds for correct `formatDateTime()` processing.

### All-Day Events
All-day events use `item()?['isAllDay']` flag to display `All Day` instead of converting a midnight timestamp which causes incorrect timezone shifts.

### HTML Table in Email
The complete HTML table is built inside `Compose - Join All Lines` using:
```
concat('<table>...<header row>...', join(body('Select'), ''), '</table>')
```
This ensures the full table renders correctly when referenced in the email body.

---

## 🚧 Known Limitations

| Limitation | Workaround |
|---|---|
| `startOfWeek()` not supported | `dayOfWeek()` nested if/else offsets |
| `mul()` / `sub()` not supported in Filter Array | Nested `if/equals` expressions |
| V3 connector doesn't expose attendees directly | Requires separate `Get Event (V3)` per event |
| University firewall blocks Power Platform URLs | Edit flows from external network |
| Copied flows lose connection references | Reconnect manually after copying |

---

## 🗺️ Roadmap

- [x] Phase 1 — Personal calendar automation (Test_Weekly)
- [x] Phase 2 — Marketing Services shared calendar (Mktg_Test_Weekly)
- [ ] Phase 3 — Management calendar extension
- [ ] Phase 4 — Attendee-based email distribution
- [ ] Phase 5 — Dynamic recipient list from SharePoint
- [ ] Phase 6 — Error handling and retry logic
- [ ] Phase 7 — Supervisor approval flow before sending

---

## 🛠️ Tech Stack

![Power Automate](https://img.shields.io/badge/Power%20Automate-0066FF?style=flat&logo=microsoft&logoColor=white)
![Outlook](https://img.shields.io/badge/Outlook-0078D4?style=flat&logo=microsoft-outlook&logoColor=white)
![Office 365](https://img.shields.io/badge/Office%20365-D83B01?style=flat&logo=microsoft-office&logoColor=white)
![HTML](https://img.shields.io/badge/HTML-E34F26?style=flat&logo=html5&logoColor=white)

- **Platform:** Microsoft Power Automate
- **Connectors:** Office 365 Outlook (V3)
- **Language:** Power Automate Expression Language (TEL)
- **Email Format:** HTML with inline CSS

---

## 📄 Documentation

- [PRD — Product Requirements Document](./PRD.md)

---

## 👤 Author

**Swe Zin Ye**
University of Portland — Marketing Services Office Assistant
