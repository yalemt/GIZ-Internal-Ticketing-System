# Changelog

## v2 — August 2026 (this package)

Revised in response to sponsor review by Janosch Stammberger (20–21 August 2026). See `docs/GIZ_Internal_Ticketing_System_Concept_Note_v2.docx`, Appendix D, for the full point-by-point response. Summary of changes:

- **Scope narrowed** to internal procurement and contract-related requests plus directly-related general support; vendor-facing requests moved out of the first prototype.
- **Category vs. request type separated** — category now determines the responsible service area only; request type/sub-category describes the specific work; a new comparison table (Section 5) clarifies the distinction, including clarification requests vs. substantive requests.
- **Routing and prioritization logic made explicit** — semi-automated routing with a coordinator control point for the pilot (Section 7); priority separated into a transparent impact/deadline/dependency scoring model (Section 8, Appendix B).
- **Ticket lifecycle tied to concrete database/automation behavior** — status-by-status update mechanism and automation opportunity (Section 9); current-state vs. audit-history separation (Section 10).
- **KIM component reframed as an assistive, confirmable layer** rather than a prerequisite, with candidate functions and human-control points spelled out (Section 12) and a companion discussion brief added (`docs/KIM_Assistant_Concept_Brief.md`).
- **Technical architecture moved to GIZ's approved Microsoft 365 environment** (MS Forms/Lists/Power Automate/Power BI) as the primary production path; the Python/Colab notebook repositioned explicitly as a design/testing sandbox, not a production system (Section 11, Appendix C).
- **New supporting technical package** added: `MS_List_Schema.xlsx`, `Power_Automate_Flow_Specification.md`, `Power_BI_Dashboard_Guide.md` (Appendix E).

## v1 — Initial concept note

Original concept covering background, objectives, a broader initial category set (including vendor-related requests), ticket lifecycle, dashboard/KPI framework, and a Python/Colab prototype.
