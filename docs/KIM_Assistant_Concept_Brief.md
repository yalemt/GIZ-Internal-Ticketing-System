# KIM Procurement/Contract Assistant — Concept Brief

**Companion to:** `GIZ_Internal_Ticketing_System_Concept_Note_v2.docx` — Section 12 (KIM Integration)
**Prompted by:** sponsor feedback point 4 — "could we create a dedicated bot/assistant in KIM that we feed with our relevant procurement knowledge, templates, contract types, guidance, etc.?"
**Status:** Discussion sketch only. This is not a technical specification, a build, or a claim about what KIM can currently do — it exists so the idea can be discussed concretely with GIZ IT/KIM owners rather than only in the abstract. Every item below needs their confirmation before anything is built.

---

## 1. Purpose

Help a requester (a project colleague) arrive at a **complete, well-classified** procurement or contract request *before* they submit the ticket form — reducing the clarification cycles named as a core problem in the concept note (Section 2). The assistant helps the requester think and phrase their request; it does not decide, approve, or replace the Service Unit's judgement (Section 12's "human control" column applies throughout).

## 2. Candidate knowledge sources (to confirm with the Procurement/Contract Service Unit)

- The four category/sub-category definitions from Section 4–5 of the concept note (so the assistant's classification vocabulary matches the ticket form exactly).
- Standard GIZ procurement thresholds and required documentation per threshold (if a knowledge document already exists internally).
- Standard contract types and what "required action" options are relevant per type (Section 6, Contract-specific fields).
- A short glossary distinguishing "clarification" from "new request" (Section 5's distinction), since this was explicitly flagged as unclear in the original concept.
- **Excluded from scope:** actual contract terms, vendor-specific data, pricing, or anything requiring access to confidential procurement records — the assistant should only need public/internal process knowledge, not case-specific confidential data, to do the intake job described here.

## 3. Illustrative system prompt (for discussion, not deployment)

```
You are an intake assistant for GIZ Social Transformation Cluster's internal
Procurement/Contract ticketing process. Your job is to help a colleague turn
a request into a complete, well-classified ticket — not to approve, decide,
or give binding procurement/contract advice.

Ask about: what they need, the project/cost centre, required-by date, and
(if procurement) estimated value and budget availability, or (if contract)
contract reference and required action.

Classify the request into exactly one of: Procurement Request, Contract
Request, Procurement/Contract Clarification, General Procurement/Contract
Support — using [category definitions from Section 4]. If the request
doesn't clearly fit, ask one clarifying question rather than guessing.

Always let the colleague correct your classification. Never state a
procurement/contract decision, deadline commitment, or approval — that is
the Service Unit's role. If asked something requiring specialist judgement,
say so and suggest they note it in the ticket for the assigned processor.

End by summarizing the structured fields you have gathered, so the colleague
can review before submitting.
```

## 4. Human control points (mirrors Section 12's table)

| Step | Assistant does | Human retains |
|---|---|---|
| Understand request | Suggests a category | Requester confirms or corrects |
| Ask follow-ups | Prompts for missing fields | Requester supplies/edits answers |
| Completeness check | Flags missing mandatory fields | Requester resolves before submission |
| Priority support | Surfaces stated deadline/impact | Receiver confirms `PriorityConfirmed` (Section 8) |
| Ticket preparation | Prepares structured field values | Requester submits; system generates `TicketID` |

## 5. Open questions for GIZ IT / KIM owners

1. Can a KIM assistant read a small, controlled knowledge source (e.g. the category/sub-category table) without that being treated as sensitive data upload?
2. Is there an existing interface/API pattern for a KIM assistant to hand structured output to an MS Forms/MS List submission, or would this stay a "copy the summary into the form" step for the pilot?
3. What authentication model would apply if this assistant is scoped to STC procurement/contract requesters only?
4. What is the review/approval process for deploying a new KIM assistant, and what lead time should the pilot plan for?

## 6. Relationship to the current prototype

The rule-based keyword classifier in the Colab notebook and the standalone HTML chatbot (see `prototype/`) demonstrate the **shape** of this interaction (ask → classify → confirm → structure) without using KIM at all. If KIM integration is approved, that same interaction shape is what would move from "keyword rules" to "KIM-generated," while every human-control point in Section 4 above stays unchanged.
