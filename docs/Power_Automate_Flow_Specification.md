# Power Automate Flow Specification

**Companion to:** `GIZ_Internal_Ticketing_System_Concept_Note_v2.docx` — Sections 7, 9, 10, 11
**Status:** Design specification for a Power Automate maker to build against the real GIZ tenant. No flow has been exported or deployed — this document exists because the author does not have build access to GIZ's Microsoft 365 environment.
**Depends on:** `MS_List_Schema.xlsx` (Tickets, Ticket History, Routing Matrix lists)

This document lists each proposed flow as trigger → conditions → actions, at the level of detail needed to build it directly in Power Automate, without prescribing a specific low-level implementation where more than one reasonable option exists.

---

## Flow 1 — Ticket Intake and ID Generation

**Trigger:** New item created in **Tickets** list (via MS Forms submission mapped to the list, or direct MS List form).

**Actions:**
1. Generate `TicketID` = `"TCK-" + formatDateTime(utcNow(),'yyyyMMdd') + "-" + right(concat('0000', string(ID)), 4)` (using the list's built-in auto-incrementing `ID` column as the sequence — avoids collision without a counter list).
2. Update the item: set `TicketID`, `Status = "NEW"`, `RequestReceivedDate = utcNow()`.
3. Look up `ServiceQueue` from the **Routing Matrix** list where `Category` matches the new item's `Category` (first matching active row) → update item's `ServiceQueue`.
4. Set `Status = "RECEIVED"`.
5. Add a row to **Ticket History**: `TicketID`, `ActionType = "Status Change"`, `OldValue = "NEW"`, `NewValue = "RECEIVED"`, `ActorPerson = "System (Power Automate)"`.
6. Send an email/Teams message to the requester: ticket ID, category, service queue, and a link to view status.
7. Post an adaptive card / email to the **Ticketing Coordinator** group with a summary and a link to the item, for the manual triage step described in Section 7.

**Notes:** Step 3 implements the routing-matrix lookup from Appendix A without hardcoding rules inside the flow, so the Ticketing Owner can update routing by editing the Routing Matrix list rather than editing the flow.

---

## Flow 2 — Status Change Timestamps and History Logging

**Trigger:** Item modified in **Tickets** list, condition: `Status` field has changed (`triggerOutputs()?['body/Status/Value']` differs from the item's previous value — Power Automate's "Get changes" or a stored previous-value column can be used to detect this reliably).

**Actions (branch on the new `Status` value):**

| New Status | Actions |
|---|---|
| UNDER REVIEW | Log history row. No timestamp field. |
| CLARIFICATION REQUIRED | Increment `ClarificationCycles` by 1. Log history row. Notify requester (email/Teams) with the clarification question from `Comment`. |
| CLARIFICATION RECEIVED | Log history row. Notify assigned processor (or coordinator if unassigned). |
| ASSIGNED | Log history row with `NewValue = AssignedProcessor`. Notify the assigned processor. |
| IN PROCESS | If `FirstResponseDate` is empty, set `FirstResponseDate = utcNow()`. Log history row. |
| PENDING APPROVAL | Log history row. Notify the designated approver (see Section 17, Roles and Responsibilities). |
| RESOLVED | Set `ResolutionDate = utcNow()`. Log history row. Notify requester with `ResolutionSummary`. |
| CLOSED | Set `ClosureDate = utcNow()`. Log history row. |
| ON HOLD / REJECTED / CANCELLED / ESCALATED | Log history row with `Comment` capturing the reason (make `Comment` a required field in the update form for these four statuses). |

**Also, on every branch:** add a row to **Ticket History** with `OldValue` = previous status, `NewValue` = new status, `ActorPerson` = the person who made the change (`triggerOutputs()?['body/Editor/Email']`).

This flow is what keeps the "processor should not manually maintain every performance field" principle from Section 10 — the processor only ever changes `Status` (and fills required fields like `Comment` or `ResolutionSummary`); every timestamp and history row is written automatically.

---

## Flow 3 — SLA Target and SLA Status

**Trigger:** Item modified in **Tickets** list, condition: `PriorityConfirmed` is set or changed.

**Actions:**
1. Set `SLATargetHours` by lookup: Urgent → 24, High → 72, Normal → 168 (Section 8). A small **SLA Targets** reference list (Priority, TargetHours) is cleaner than hardcoding these in the flow, and lets the Service Unit adjust targets after the pilot without a flow change.
2. Recompute `SLAStatus`:
   - If `Status` is RESOLVED or CLOSED: compare `(ResolutionDate − RequestReceivedDate)` in hours to `SLATargetHours` → `"On Track"` or `"Breached"`.
   - If still open: compare `(utcNow() − RequestReceivedDate)` to `SLATargetHours` → `"On Track"` or `"At Risk"` (e.g. within 80% of target) or `"Breached"`.

**Recommended companion:** a scheduled (nightly) flow that re-runs step 2 for all open tickets, since elapsed time changes even when nobody edits the item.

---

## Flow 4 — Reminders and Escalation (Scheduled)

**Trigger:** Recurrence, e.g. daily at 08:00.

**Actions:**
1. Get all items from **Tickets** where `Status` not in (RESOLVED, CLOSED, REJECTED, CANCELLED).
2. For each: if `SLAStatus = "At Risk"`, notify the `AssignedProcessor` (or coordinator if unassigned).
3. For each: if `SLAStatus = "Breached"` and `Status` has not changed in the configured escalation window (e.g. 48 hours since `ChangeTimestamp` of the most recent Ticket History row), notify the coordinator and the processor's manager, and set `Status = "ESCALATED"` (which itself triggers Flow 2's history logging).

---

## Flow 5 — Clarification Loop Convenience (Optional, Phase 2+)

**Trigger:** Reply received on the clarification email/Teams message (via a "Forms response" or a dedicated reply-to-comment mechanism, e.g. a Power Apps-based reply form linked to the TicketID).

**Actions:** Append the requester's reply to `DependencyNotes` or a new `Comment`, set `Status = "CLARIFICATION RECEIVED"` (triggers Flow 2).

This flow is optional for the pilot — manually changing status after checking email is an acceptable fallback (Section 7's "human control point while routing rules are tested").

---

## Build Notes for the Power Automate Maker

- Prefer **one flow per lifecycle concern** (as above) over one large flow with many branches — easier to test, easier to hand over, and matches how the concept note phases automation incrementally (Section 16, Phase 3).
- Use a **service account or flow owner group**, not a named individual's Power Automate license, so the flows survive staff changes — confirm the licensing model with GIZ IT.
- All "notify" actions should degrade gracefully (e.g. log to Ticket History even if the Teams/email action fails) so a notification failure never silently drops a status change.
- None of these flows have been built or tested against a live GIZ tenant. Trigger names, list-change detection method, and connector availability (Teams vs. Outlook vs. both) depend on what GIZ IT confirms is available (see concept note Section 14 and Appendix C).
