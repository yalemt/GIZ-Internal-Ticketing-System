# Prototype Folder — Read This First

**None of the files in this folder are the production system.** They are sandboxes for testing the data model, category/routing logic, priority scoring, lifecycle rules and dashboard concepts described in the concept note — before anything is built in GIZ's approved Microsoft 365 environment. Do not enter real/confidential GIZ data into any file here (see concept note, Appendix C — Prototype Limitations).

## Files

| File | What it is | How to run it |
|---|---|---|
| `GIZ_Ticketing_System_Colab_Humanized_Revised.ipynb` | Python notebook demonstrating the revised category/sub-category model, routing suggestion, priority scoring, lifecycle transitions, SLA calculations, audit-trail simulation, mock KIM intake, and the four dashboard pages (via Plotly). | Upload to [Google Colab](https://colab.research.google.com) (or open in Jupyter), also upload `tickets_dataset_sample.csv` to the same session, then Runtime → Run all. |
| `tickets_dataset_sample.csv` | Synthetic sample tickets used by the notebook so the dashboards can be previewed before real data exists. | Used automatically by the notebook if present in the working directory. |
| `GIZ_Ticketing_Requester.html` | Standalone browser app: chatbot-style intake + "My Tickets" lookup. | Open directly in Chrome or Edge (double-click the file). No install needed. |
| `GIZ_Ticketing_Admin.html` | Standalone browser app: dashboard + full ticket queue with status editing. PIN-protected (default `2026`, see the `ADMIN_PIN` constant near the top of the file's `<script>`). | Open directly in Chrome or Edge. |
| `GIZ_Ticketing_SingleFile_Demo.html` | One file combining both roles behind a role-select screen — useful for a quick single-machine demo. | Open directly in any modern browser. |

## Syncing the Requester and Admin apps

`GIZ_Ticketing_Requester.html` and `GIZ_Ticketing_Admin.html` are designed to run on two different machines and stay in sync via **one shared JSON file placed in a folder that syncs through Google Drive or OneDrive** (desktop sync client required on both machines). In each app's top bar:

1. On the first machine, click **"Create shared file (first time only)"** and save it inside the synced folder.
2. Once that file has synced to the second machine, open the other app there and click **"Connect to shared file"**, selecting the same file.
3. Both apps auto-sync roughly every 8 seconds while open, and immediately after any ticket is created or a status is changed.

This requires Chrome or Edge (the File System Access API is not available in Firefox/Safari); both apps also include manual JSON Export/Import as a fallback. **This is a convenience sync for prototyping, not a real backend** — see the concept note's Section 11 for why the production path is MS Lists, not this mechanism.

## Why this isn't the production system

GIZ's approved environment does not include Google Colab or unmanaged static HTML files as a production platform (concept note, sponsor feedback point 5, Appendix D). These prototypes exist to let the category definitions, routing logic, priority model and dashboard KPIs be tested and argued about cheaply, before committing to a Microsoft 365 build. Once the `docs/` architecture is implemented, this folder can be archived.
