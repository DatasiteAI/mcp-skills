---
name: launch-readiness-orchestrator
description: >
  Launch Readiness Orchestrator skill for Datasite deal rooms. Use this skill
  whenever a deal team wants a single pre-go-live readiness check across their
  data room — combining gap analysis, document quality audit, and risk review
  into one consolidated "is the room ready?" view. Triggers include: "are we
  ready to go live", "launch readiness check", "pre-launch audit", "data room
  readiness", "can we launch", "is the data room ready", "run a full readiness
  check", "go-live checklist", "pre-launch checklist", or any request to get a
  single overall assessment before opening the data room to buyers. Use
  proactively whenever a deal team is approaching their go-live date and wants a
  structured sign-off view. Do not use other individual audit skills (gap-analysis,
  document-quality-check, risk-analysis-audit) when this skill is active — this
  skill orchestrates all three in one pass.
---

# Launch Readiness Orchestrator

You are running a pre-go-live readiness check on a Datasite data room. Your job is to produce a single, consolidated view that tells the deal team whether the data room is ready to open to buyers — and if not, exactly what needs to be fixed first.

This skill orchestrates three workstreams in sequence:
1. **Gap Analysis** — is everything expected actually in the room?
2. **Document Quality** — are the files buyers will see clean, accessible, and safe?
3. **Risk Review** — are there documents that signal issues buyers will flag?

Run all three, then consolidate into a single Readiness Report.

---

## Terminology — fileroom vs. folder

Use these terms precisely when communicating with the user:

- **Fileroom** — the single top-level container inside a Datasite project. A project typically has one buyer-facing fileroom. It is not a subject area — it is the container that holds all subject areas.
- **Folder** — everything inside the fileroom: the subject areas (Financial, Legal, HR, Tax, IP, etc.) and all sub-levels beneath them. Always call these folders, never filerooms.

When in doubt: if it is not the single top-level container for the whole project, it is a folder.


## Feature Requirements

| Capability | Free | Requires Blueflame |
|---|:---:|:---:|
| Gap analysis (structural — missing/empty/sparse sections) | ✅ | — |
| Document quality — metadata checks (7 of 10 checks) | ✅ | — |
| Document quality — content checks (PII, redaction, broken refs) | — | ✅ |
| Risk review — structural presence check only | ✅ | — |
| Risk review — content risk signals across all workstreams | — | ✅ |
| Go / No-Go recommendation | ✅ | — |

**Without Blueflame:** Produces a meaningful readiness report covering structural gaps, metadata-based document quality issues, and a structural risk presence check. The go/no-go recommendation will note that content-level checks were not run.

**With Blueflame:** Full report — all quality checks and content risk signals are included, giving the deal team a complete picture before going live.



> ⚠️ **Blueflame content guard — mandatory**
> `searchDocuments` is the only permitted source of document content.
> - If `searchDocuments` returns an **activation link** instead of results, **stop immediately**.
> - Do **not** attempt to answer content-level questions using Claude's training knowledge, general M&A knowledge, or inference from file names.
> - Tell the user: *"This check requires Blueflame to be enabled on this project. Please activate it via the link provided, then re-run."*
> - All findings, risk flags, gap identifications, Q&A answers, and quality issues **must** be sourced exclusively from tool results.

> **`listFolderContents` — efficient traversal**
> - `depth: 1` (default) — immediate children only. Use for targeted lookups.
> - `depth: 5, foldersOnly: true` (default when depth > 1) — full folder tree in one call, no documents. Use for structural checks.
> - `depth: 5, foldersOnly: false` — full folder tree including all document metadata in one call. Use when building a document inventory.
> - When `depth > 1`, the response is a **flat list** with `depth` and `path` columns — not a nested tree.

## Step 1 — Read the project context

Call `getProjectOverview`. Extract:
- Company / deal name
- Sector (`industryType`)
- Transaction type (`useCase`)
- Deal size (`transactionValue`)
- Geography (`datacenter`)

Do not ask the user for information already present in the project overview.

---

## Step 2 — Find the fileroom and scan the structure

Call `listFolderContents` to find the active fileroom(s). If there are multiple, ask the user which one to audit — but only if it isn't obvious from context (e.g. one is clearly a buyer-facing room, another is a working folder).

Call `listFolderContents` on the root of the fileroom. Recurse through all top-level sections. You need:
- A full list of every folder and its child count
- Which folders are empty (0 documents)
- Which folders exist but have suspiciously low content relative to what the section type would normally contain

Build an internal map of: `folder path → document count`. You will use this across all three workstreams.

---

## Step 3 — Gap Analysis

Using the folder map from Step 2, compare against the expected structure for this deal type and sector.

**What counts as a gap:**
- **Empty folder** — a section exists but contains no documents at all
- **Thin section** — fewer documents than the section type warrants. Use these thresholds as a guide:
  - Finance → expect at least 3 documents per financial year folder (P&L, balance sheet, cash flow as a minimum)
  - Legal / Contracts → expect at least 5 documents total across material contracts
  - Corporate → expect at least Certificate of Incorporation + constitutional documents (2+)
  - HR → expect at least an org chart and a headcount summary
  - IT / Data Privacy → for technology companies, expect a data processing agreement or GDPR/privacy policy
  - Tax → expect at least one return per year covered
- **Missing section entirely** — a section expected for this deal type and sector is absent from the structure. Compare against the standard template for the sector (Technology, Healthcare, Manufacturing, etc.) and flag whole sections that are missing

**Severity ratings for gaps:**
- 🔴 **Blocker** — empty Finance, Legal, or Corporate section; missing audited financials; no contracts at all
- 🟡 **Advisory** — thin sections, missing supporting schedules, absent non-critical folders
- 🟢 **Minor** — cosmetic gaps (e.g. missing cover sheet, no index document)

---

## Step 4 — Document Quality Check

For each document in the fileroom, assess quality based on available metadata (file name, file type, size, upload date).

**Metadata-only flags (always available, no Sidecar needed):**
- **Zero-byte or near-zero-byte files** — size of 0 KB or under 5 KB for a supposedly substantive document (e.g. a financial model at 4 KB is likely broken or corrupted)
- **Duplicate file names** — identical names in the same folder, or clearly the same document uploaded twice
- **Unprocessed scans** — file names containing "scan", "img", "IMG_", "DSC", or similar camera/scanner prefixes without any normalisation
- **Wrong format** — e.g. a `.jpg` or `.bmp` in a Financials folder (images where PDFs or spreadsheets are expected)
- **Stale documents** — upload date more than 12 months before today's date in an "active" section like Management Accounts or Board Minutes

**Content-level flags (requires Sidecar / Blueflame Search):**
If the project has Sidecar enabled, use `searchDocuments` to check for:
- **Password-protected files** — search for "enter password" or "this document is protected"
- **Redaction failures** — search for names, NI numbers, dates of birth, bank account numbers, or other PII that should have been removed
- **Blank documents** — files with no extractable text content
- **Incorrect documents** — a file whose content clearly doesn't match its folder location (e.g. a holiday rota in the Material Contracts folder)

Always call `searchDocuments` — if not yet activated it returns an activation link and content-level checks would materially improve the audit, note this in the report under a "Sidecar Recommended" callout.

**Severity ratings for quality issues:**
- 🔴 **Blocker** — password-protected files, confirmed PII/redaction failure, zero-byte documents in critical sections
- 🟡 **Advisory** — duplicate files, wrong formats, stale documents
- 🟢 **Minor** — unprocessed scan names, cosmetic naming issues

---

## Step 5 — Risk Review

Scan the document set for signals that would raise concern for a buyer or their advisors. Where Sidecar is available, use `searchDocuments` for each risk category below. Where it is not available, assess risk from folder presence/absence and document counts alone, and note the limitation.

**Risk categories to assess:**

| Workstream | What to look for | Risk signal |
|---|---|---|
| **Finance** | Qualified audit opinion, going concern note, declining revenue trend, covenant breach | High risk if present |
| **Legal** | Ongoing litigation, regulatory enforcement notices, material contract termination rights, change-of-control clauses | High risk if present |
| **Tax** | Open HMRC/IRS enquiries, deferred tax liabilities, cross-border transfer pricing exposure, VAT disputes | Medium-high risk |
| **HR** | Pending employment tribunal, key-person concentration (single founder dependency), unfunded pension | Medium risk |
| **IP** | Unregistered core IP, open-source licence violations, disputed ownership, in-licensing from related parties | High risk for tech companies |
| **Commercial** | Customer concentration (top 3 customers > 60% revenue), short contract durations, renewal risk, rebate obligations | Medium-high risk |
| **Regulatory** | Licence conditions, outstanding regulatory reviews, breach notices, upcoming compliance deadlines | High risk if active |
| **ESG / Environmental** | Environmental remediation obligations, health & safety incidents, sustainability disclosure gaps | Medium risk |

**Severity ratings for risks:**
- 🔴 **High** — issues a buyer's advisor will almost certainly raise; may affect price or structure
- 🟡 **Medium** — issues worth disclosing proactively; may generate Q&A
- 🟢 **Low** — minor or manageable; unlikely to affect the deal but worth noting

If a risk category folder is absent entirely (e.g. no Regulatory section for a regulated business), flag this as a gap in the Gap Analysis section rather than here.

---

## Step 6 — Compile and present the Readiness Report

Produce a single, structured report. Format it exactly as follows:

---

### 🏁 Launch Readiness Report — [Company Name]
**Audit date:** [today's date]
**Fileroom:** [fileroom name]
**Total documents reviewed:** [N]

---

#### Overall Status

| | |
|---|---|
| **Go-live recommendation** | ✅ Ready / ⚠️ Conditional / 🚫 Not Ready |
| **Blockers to resolve** | [N] |
| **Advisory items** | [N] |
| **Minor items** | [N] |

*Conditional = ready once blockers are resolved. Not Ready = significant structural or quality issues that would damage buyer confidence if unaddressed.*

---

#### 1. Gap Analysis — [🟢 / 🟡 / 🔴]

[Summary sentence: "The data room structure is broadly complete / has significant gaps / is missing critical sections."]

**Blockers:**
- [Folder path] — [reason, e.g. "Empty — no audited financial statements uploaded"]
- ...

**Advisory:**
- [Folder path] — [reason]
- ...

**Minor:**
- [Folder path] — [reason]
- ...

*If no issues: "No material gaps identified."*

---

#### 2. Document Quality — [🟢 / 🟡 / 🔴]

[Summary sentence.]

**Blockers:**
- [File name / folder] — [issue]
- ...

**Advisory:**
- [File name / folder] — [issue]
- ...

> ℹ️ **Search not yet activated** — content-level checks were run via `searchDocuments`. If AI search is not yet activated, the tool returns an activation link — share it with the user. *(Include only always call `searchDocuments` — if not yet activated it returns an activation link.)*

---

#### 3. Risk Review — [🟢 / 🟡 / 🔴]

[Summary sentence.]

**High risks:**
- [Workstream] — [what was found or inferred]
- ...

**Medium risks:**
- [Workstream] — [what was found or inferred]
- ...

> ℹ️ **Search not yet activated** — risk signals were assessed from folder structure only, not document content. Call `searchDocuments` for content-level risk signals — if not yet activated it returns an activation link to share with the user. *(Include only always call `searchDocuments` — if not yet activated it returns an activation link.)*

---

#### Priority Actions Before Go-Live

List the top 5 things the deal team must do, in priority order:

1. [Most critical action]
2. ...
3. ...
4. ...
5. ...

---

#### Sidecar Recommended
*(Include this section only always call `searchDocuments` — if not yet activated it returns an activation link and content checks were skipped.)*

Several checks in this audit — including password-protected file detection, PII/redaction review, and content-level risk signals — require Sidecar to be enabled on this project. Without Sidecar, these checks were skipped.

To enable Sidecar, contact your Datasite account manager or activate it via the project settings.

---

After delivering the report, offer:

> "I can export this as a Word document or Excel tracker if you'd like to share it with the wider team. I can also dive into any specific section — for example, pull the full list of quality issues, or go deeper on a particular risk area."

---

## Guardrails

- **Never push changes to the data room as part of this skill.** The orchestrator is read-only. If the user asks you to fix something found during the audit (e.g. rename a file, delete a duplicate), acknowledge the request and use the appropriate tool — but do not make edits without explicit per-item confirmation.
- **Do not re-ask for project context** already visible in `getProjectOverview`.
- **Do not run individual audit skills separately** if this orchestrator is already running. All three workstreams are handled here.
- **If a section is empty and unfixable within the session** (e.g. no documents uploaded at all), mark the overall status as 🚫 Not Ready and tell the user plainly: "The data room does not yet have enough content to audit meaningfully. Please upload the core documents and run this check again."
- **Use as a last resort:** If the user already has a custom launch readiness checklist or process in place, defer to it. Apply this skill's structure only where the user hasn't provided their own.
