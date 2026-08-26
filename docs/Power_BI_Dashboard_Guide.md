# Power BI Dashboard Guide

**Companion to:** `GIZ_Internal_Ticketing_System_Concept_Note_v2.docx` — Section 13 (Dashboard and KPI Framework)
**Data sources:** the **Tickets** and **Ticket History** MS Lists (see `MS_List_Schema.xlsx`), or the sample dataset `tickets_dataset_sample.csv` / the Colab prototype's synthetic data while no real data exists yet.
**Status:** Build guide, not a delivered `.pbix` — a Power BI file references live MS Lists in a specific GIZ SharePoint site the author cannot connect to. This guide gives everything a report builder needs to construct the same four pages described in Section 13.

---

## 1. Getting data in

- **Get Data → SharePoint Online List**, point at the site hosting the Tickets and Ticket History lists (or **Get Data → Text/CSV** against `tickets_dataset_sample.csv` for a first build/rehearsal).
- Set the connection to **Import** mode for the pilot (small data volume; DirectQuery is unnecessary and slower for MS Lists).
- Rename the queries `Tickets` and `TicketHistory` for clarity.

## 2. Data model

Two tables, one relationship:

```
Tickets[TicketID]  1 ────< *  TicketHistory[TicketID]
```

- Relationship: **one-to-many**, single direction (Tickets → TicketHistory).
- Mark `Tickets` as the **date table anchor**: create a calculated `Date` table (Power Query → New Source → Blank Query → `= List.Dates(...)`, or Power BI's built-in "Auto date/time") if trend-over-time visuals are needed; otherwise `RequestReceivedDate` alone is enough for the pilot's volume.
- Set `Category`, `SubCategoryRequestType`, `PriorityConfirmed`, `Status`, `ServiceQueue`, `AssignedProcessor` as **Text**; all date columns as **Date/Time**; `SLATargetHours`, `ClarificationCycles`, `ImpactScore`, `DeadlinePressureScore`, `DependencyScore` as **Whole Number**.

## 3. Suggested DAX measures

```dax
Total Tickets = COUNTROWS(Tickets)

Open Tickets =
CALCULATE(
    [Total Tickets],
    NOT( Tickets[Status] IN {"RESOLVED","CLOSED","REJECTED","CANCELLED"} )
)

Closed Tickets =
CALCULATE([Total Tickets], Tickets[Status] = "CLOSED")

Overdue Tickets =
CALCULATE([Total Tickets], Tickets[SLAStatus] = "Breached")

Escalated Tickets =
CALCULATE([Total Tickets], Tickets[Status] = "ESCALATED")

Avg Response Time (hrs) =
AVERAGEX(
    FILTER(Tickets, NOT(ISBLANK(Tickets[FirstResponseDate]))),
    DATEDIFF(Tickets[RequestReceivedDate], Tickets[FirstResponseDate], MINUTE) / 60
)

Avg Resolution Time (hrs) =
AVERAGEX(
    FILTER(Tickets, NOT(ISBLANK(Tickets[ResolutionDate]))),
    DATEDIFF(Tickets[RequestReceivedDate], Tickets[ResolutionDate], MINUTE) / 60
)

SLA Compliance % =
DIVIDE(
    CALCULATE([Total Tickets], Tickets[SLAStatus] = "On Track"),
    CALCULATE([Total Tickets], NOT(ISBLANK(Tickets[SLAStatus])))
)

Avg Clarification Cycles =
AVERAGE(Tickets[ClarificationCycles])

Aging (days, open only) =
AVERAGEX(
    FILTER(Tickets, NOT( Tickets[Status] IN {"RESOLVED","CLOSED","REJECTED","CANCELLED"} )),
    DATEDIFF(Tickets[RequestReceivedDate], TODAY(), DAY)
)

Pending Approvals =
CALCULATE([Total Tickets], Tickets[Status] = "PENDING APPROVAL")
```

All measures are written against **Section 13's indicator list** so each dashboard tile below maps to a named measure rather than an ad-hoc visual calculation.

## 4. Page-by-page layout

### Page 1 — Executive Overview
- KPI cards (top row): `[Total Tickets]`, `[Open Tickets]`, `[Closed Tickets]`, `[Overdue Tickets]`, `[Escalated Tickets]`.
- Second row: `[Avg Response Time (hrs)]`, `[Avg Resolution Time (hrs)]`, `[SLA Compliance %]` as gauge or card visuals.
- Add a **date range slicer** (`RequestReceivedDate`) at the top so the whole page can be filtered to the pilot period.

### Page 2 — Workload
- Clustered bar chart: ticket count by `Category`.
- Clustered bar chart: ticket count by `SubCategoryRequestType` (filtered/drilled from Category).
- Bar chart: ticket count by `ServiceQueue`.
- Bar chart: ticket count by `AssignedProcessor` (workload per person).
- Donut chart: ticket count by `PriorityConfirmed`.
- Matrix: `Status` (rows) × `ServiceQueue` (columns), values = count — gives an at-a-glance queue/status cross-tab.

### Page 3 — Performance
- Box-and-whisker or clustered column: `[Avg Resolution Time (hrs)]` by `PriorityConfirmed`.
- Donut: ticket count by `SLAStatus`.
- Card: `[Aging (days, open only)]`, with a table beneath listing open tickets sorted by age descending (`TicketID`, `Category`, `AssignedProcessor`, age in days).
- Card: `[Pending Approvals]`.
- Line chart: `[Avg Clarification Cycles]` over time (by month of `RequestReceivedDate`) if enough history exists.

### Page 4 — Process Improvement
- Bar chart: `[Avg Resolution Time (hrs)]` by `Category`, sorted descending — surfaces the slowest categories (Section 13, "longest-resolution categories").
- Bar chart: `[Avg Clarification Cycles]` by `Category` or `SubCategoryRequestType` — surfaces categories with the most back-and-forth.
- Table: top recurring `Comment` text from **TicketHistory** rows where `ActionType = "Clarification Requested"`, for manual review of recurring clarification reasons (free text is hard to aggregate automatically — treat this as a reading list, not a chart).

## 5. Prototype-vs-real-data labelling (Section 13 requirement)

Add a text box or a measure-driven indicator on every page reading either **"Prototype / synthetic data"** or **"Live operational data"**, driven by a manually-set parameter table (one row, one Yes/No field an administrator flips once real data replaces the sample dataset). This directly implements the concept note's requirement that illustrative figures are never presented as real performance.

## 6. Refresh

- For a pilot connected to real MS Lists: **Scheduled refresh** (Power BI Service), e.g. every 30–60 minutes — MS Lists do not support real-time push into Power BI without Power Automate/Dataflows.
- Confirm gateway/licensing requirements with GIZ IT before scheduling refresh in production (Section 14, Section 17 — IT/KIM support role).
