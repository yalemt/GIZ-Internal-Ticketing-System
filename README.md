# GIZ Internal Request & Ticketing System — Concept, Design Package and Prototype

Internal concept for standardizing how GIZ Social Transformation Cluster (STC) projects submit and track procurement- and contract-related requests to the Procurement/Contract Service Unit, instead of relying on scattered emails, Teams messages and informal follow-ups.

Prepared by **Yalem Tilahun** (Procurement and Contract Specialist, STC). Reviewed by **Janosch Stammberger** (Head of Service Unit, STC) — see `docs/GIZ_Internal_Ticketing_System_Concept_Note_v2.docx`, Appendix D, for how his feedback shaped this revision.

## What this is — and isn't

This repository is a **concept and design package**, not a deployed system. Nothing here is connected to any live GIZ system, MS List, Power Automate flow or KIM instance. The intended production path is GIZ's approved Microsoft 365 environment (MS Forms/Lists, Power Automate, Power BI), subject to confirmation by GIZ IT — see Section 11 of the concept note. The Python/Colab notebook and the HTML prototypes in `prototype/` are sandboxes for testing the data model and logic on the author's own machine; they are explicitly **not** part of the proposed production architecture (concept note, Appendix C).

## Repository structure

```
GIZ_Ticketing_System/
├── README.md                                  ← you are here
├── CHANGELOG.md                                ← v1 → v2 revision history
├── NOTICE.md                                   ← internal-use notice
├── docs/
│   ├── GIZ_Internal_Ticketing_System_Concept_Note_v2.docx   ← the concept note (read this first)
│   ├── MS_List_Schema.xlsx                     ← column-by-column schema for the 3 proposed MS Lists
│   ├── Power_Automate_Flow_Specification.md    ← trigger/condition/action spec for the workflow layer
│   ├── Power_BI_Dashboard_Guide.md             ← data model, DAX measures, page-by-page dashboard layout
│   └── KIM_Assistant_Concept_Brief.md          ← discussion sketch for a KIM-based intake assistant
└── prototype/
    ├── README.md                               ← disclaimer + how to run each prototype file
    ├── GIZ_Ticketing_System_Colab_Humanized_Revised.ipynb   ← Python/Colab data-model & dashboard sandbox
    ├── tickets_dataset_sample.csv               ← synthetic sample data used by the notebook
    ├── GIZ_Ticketing_Requester.html              ← standalone browser demo: requester-side intake + tracking
    ├── GIZ_Ticketing_Admin.html                  ← standalone browser demo: dashboard + ticket queue
    └── GIZ_Ticketing_SingleFile_Demo.html         ← single-file demo combining both roles behind a PIN/role switch
```

## Suggested reading order

1. `docs/GIZ_Internal_Ticketing_System_Concept_Note_v2.docx` — background, scope, categories, routing, priority, lifecycle, architecture, KPIs, roles, risks.
2. `docs/MS_List_Schema.xlsx` — what to actually create in MS Lists.
3. `docs/Power_Automate_Flow_Specification.md` — what to build in Power Automate once the lists exist.
4. `docs/Power_BI_Dashboard_Guide.md` — how to build the four dashboard pages against that data.
5. `docs/KIM_Assistant_Concept_Brief.md` — optional, for the KIM/AI conversation with IT.
6. `prototype/` — optional, for testing the logic hands-on before anything is built in the GIZ tenant.

## Status

Design/concept stage. Next step per the concept note's implementation plan (Section 16) is current-state validation and pilot design with the Procurement/Contract Service Unit, followed by building the Microsoft 365 pilot described in `docs/`.
