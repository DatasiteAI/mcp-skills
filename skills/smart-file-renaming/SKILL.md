---
name: smart-file-renaming
description: >
  Smart File Renaming skill for Datasite deal rooms. Use this skill whenever a deal
  team wants to standardise document names, clean up scanned file names, normalise
  naming across similar document types, or improve the professionalism of the data
  room before going live. Triggers include: "rename the files", "clean up the file
  names", "standardise naming", "the file names are a mess", "fix the document
  names", "rename scanned documents", "make the naming consistent", "tidy up the
  data room", or any request to improve, clean, or normalise document naming across
  a Datasite project. Never apply any rename without explicit user confirmation.
---

# Smart File Renaming

You are helping a deal team standardise document names across their Datasite data room. Buyers judge preparation quality from the first thing they see — a folder full of `Scan001.pdf`, `Agreement_FINAL_v3.docx`, and `Copy of Financial Model (2).xlsx` signals a poorly run process.

**The single most important rule: never rename anything without showing the user a full before/after table first and receiving explicit confirmation.**

---

## Terminology — fileroom vs. folder

Use these terms precisely when communicating with the user:

- **Fileroom** — the single top-level container inside a Datasite project. A project typically has one buyer-facing fileroom. It is not a subject area — it is the container that holds all subject areas.
- **Folder** — everything inside the fileroom: the subject areas (Financial, Legal, HR, Tax, IP, etc.) and all sub-levels beneath them. Always call these folders, never filerooms.

When in doubt: if it is not the single top-level container for the whole project, it is a folder.


## Feature Requirements

| Capability | Free | Requires Blueflame |
|---|:---:|:---:|
| Rename from folder context and filename | ✅ | — |
| Apply naming conventions across all document types | ✅ | — |
| Read inside documents to infer year, counterparty, or jurisdiction | — | ✅ |

**Without Blueflame:** Renames are based on folder context and filename patterns only. Where document content is needed to determine the year or counterparty (e.g. generic scan names), the proposed name will include a `[YYYY]` or `[Counterparty]` placeholder rather than guessing.

**With Blueflame:** `searchDocuments` reads document content to extract dates, counterparty names, and jurisdictions — producing fully resolved names with no placeholders.



> ⚠️ **Blueflame fallback — explicit choice required**
> `searchDocuments` is the only permitted source of document content.
> - Do **not** infer dates, counterparty names, or document types from Claude's training knowledge.
> - If `searchDocuments` returns an **activation link** instead of results, **do not silently continue**. Stop and present the user with an explicit choice:
>
>   > "For documents where the filename doesn't contain the counterparty name, year, or jurisdiction, I need to read inside the file to propose an accurate name — this requires Blueflame AI search to be activated on this project.
>   > 🔗 **Activate Blueflame:** [activation link]
>   > **With Blueflame:** I'll read the opening clauses of contracts (exact party names), year-end dates in financial statements, and jurisdiction from tax filings — fully resolved names with no placeholders.
>   > **Without Blueflame:** I'll complete all renames I can from filename patterns and folder context, and use `[Counterparty]`, `[YYYY]`, `[Jurisdiction]` placeholders where I'd need to read the document.
>   > Would you like to activate now, or shall I proceed with placeholder-based names?"
>
>   Wait for the user's response before continuing.

> **`listFolderContents` — efficient traversal**
> - `depth: 1` (default) — immediate children only. Use for targeted lookups.
> - `depth: 5, foldersOnly: true` (default when depth > 1) — full folder tree in one call, no documents. Use for structural checks.
> - `depth: 5, foldersOnly: false` — full folder tree including all document metadata in one call. Use when building a document inventory.
> - When `depth > 1`, the response is a **flat list** with `depth` and `path` columns — not a nested tree.

## Step 1 — Orient yourself

Call `getProjectOverview` to understand the project: company name, sector, and fileroom structure. The company name will be used in naming conventions (e.g. `[Company] - Audited Accounts - FY2025.pdf`).

---

## Step 2 — Crawl and identify files needing attention

Call `listFolderContents` with `depth: 5, foldersOnly: false` to retrieve the complete document inventory in a single call. The response is a flat list including all folders and documents with metadata (name, fileType, status, pageCount, path). For each document, record:
- Current filename (including extension)
- Metadata ID (needed for `updateContent` later)
- Folder path and VDR index
- File size and page count (to help infer document type)

Identify files that need renaming using these signals:

**Never rename — flag for immediate removal from data room:**
These files should not exist in a buyer-facing data room. Flag them as critical issues and do not include them in any rename proposals:
- Internal system or index files: `DOCUMENT_MANIFEST`, `FILE_INDEX`, `FILE_SUMMARY`, `TAX_FILINGS_SUMMARY`, `FOLDER_STRUCTURE`, `INDEX`, `MANIFEST`
- Any file whose name suggests it is a processing artefact, upload log, or internal tool output
- Present these to the user as: “**[N] internal system files found** — these should be deleted before go-live: [list with folder paths]. These have been excluded from the rename proposals.”

**Definitely rename:**
- Sequential scan names: `Scan001`, `Scan_001`, `IMG_0234`, `Document (3)`, `Untitled`
- Generic upload names: `File`, `New Document`, `Copy of`, `Attachment`
- Chaotic versioning: `FINAL_FINAL`, `USE THIS ONE`, `DO NOT USE`, `v2_revised_final`
- Double extensions: `Contract.pdf.pdf`, `Accounts.docx.pdf`
- Truncated or corrupted names from bulk upload tools

**Review for standardisation** (may be acceptable but inconsistent with siblings):
- Version suffixes: `v1`, `v2`, `draft`, `revised`, `updated`
- Inconsistent date formats: some files use `2024`, others `FY24`, others `April 2024`
- Inconsistent party naming: `Acme Corp Contract.pdf` next to `Agreement - Acme Corporation.pdf` — same counterparty, different name
- Missing year when year is expected (e.g. `Tax Return.pdf` in a tax folder with multiple years)

---

## Step 3 — Infer document type and content from context

Before proposing a name, understand what the document actually is. Use two signals:

**1. Folder context (primary):** A file in `3.1 Audited Financial Statements` is an annual accounts document. A file in `7.2 Employment Agreements` is an employment contract. The folder tells you the document type — use it.

**2. Document content (when needed):** If the folder context isn't enough to determine the year, counterparty name, or document subtype, use `searchDocuments` on the document to extract:
- The financial year (look for "year ended", "for the year", "FY", "as at 31 December")
- The counterparty name (look for "between [Company] and [X]", "agreement with", "entered into by")
- The jurisdiction (for tax returns: "Federal", "State of California", "HMRC", "Companies House")
- The employee name (for employment agreements: opening clause "This agreement is between [Company] and [Name]")

Only use content search when the filename alone is genuinely ambiguous. Don't read every document — use judgment.

---

## Step 4 — Apply naming conventions by document category

Use the conventions below to propose new names. Conventions are designed to be:
- **Scannable** — buyers can understand what a file is without opening it
- **Sortable** — date formats ensure chronological sort order
- **Consistent** — all files of the same type follow the same pattern

### Financial documents
| Document type | Convention | Example |
|---|---|---|
| Audited annual accounts | `[Company] - Audited Accounts - FY[YYYY].pdf` | `Apex Ltd - Audited Accounts - FY2024.pdf` |
| Management accounts (monthly) | `[Company] - Management Accounts - [Mon YYYY].pdf` | `Apex Ltd - Management Accounts - Mar 2025.pdf` |
| Management accounts (quarterly) | `[Company] - Management Accounts - Q[N] FY[YYYY].pdf` | `Apex Ltd - Management Accounts - Q3 FY2025.pdf` |
| Financial model / projections | `[Company] - Financial Model - [YYYY-MM-DD].xlsx` | `Apex Ltd - Financial Model - 2025-03-01.xlsx` |
| Board pack / management presentation | `[Company] - Board Pack - [Mon YYYY].pdf` | `Apex Ltd - Board Pack - Jan 2025.pdf` |

### Tax documents
| Document type | Convention | Example |
|---|---|---|
| Federal / national tax return | `[Company] - [Jurisdiction] Tax Return - FY[YYYY].pdf` | `Apex Ltd - Federal Tax Return - FY2024.pdf` |
| State / local tax return | `[Company] - [State] Tax Return - FY[YYYY].pdf` | `Apex Ltd - California Tax Return - FY2024.pdf` |
| VAT return | `[Company] - VAT Return - [Q/Period] [YYYY].pdf` | `Apex Ltd - VAT Return - Q4 2024.pdf` |
| Tax correspondence | `[Company] - [Authority] Correspondence - [YYYY-MM-DD].pdf` | `Apex Ltd - HMRC Correspondence - 2024-11-15.pdf` |

### Corporate documents
| Document type | Convention | Example |
|---|---|---|
| Certificate of incorporation | `[Company] - Certificate of Incorporation.pdf` | |
| Articles of association / by-laws | `[Company] - Articles of Association.pdf` | |
| Board minutes | `[Company] - Board Minutes - [YYYY-MM-DD].pdf` | `Apex Ltd - Board Minutes - 2024-09-12.pdf` |
| Shareholder agreement | `[Company] - Shareholder Agreement - [YYYY].pdf` | |
| Cap table | `[Company] - Cap Table - [YYYY-MM-DD].xlsx` | |

### Contracts — use counterparty name as the primary identifier
| Document type | Convention | Example |
|---|---|---|
| Customer / client contract | `[Client Name] - [Contract Type] - [YYYY].pdf` | `Tesco PLC - Master Services Agreement - 2023.pdf` |
| Vendor / supplier contract | `[Supplier Name] - [Contract Type] - [YYYY].pdf` | `AWS - Cloud Services Agreement - 2024.pdf` |
| Employment agreement | `[Last Name, First Name] - Employment Agreement - [YYYY].pdf` | `Smith, James - Employment Agreement - 2022.pdf` |
| Consulting / contractor agreement | `[Name] - Consulting Agreement - [YYYY].pdf` | |
| Lease agreement | `[Property Address or Name] - Lease Agreement - [YYYY].pdf` | `123 High Street London - Lease Agreement - 2021.pdf` |
| NDA | `[Counterparty] - NDA - [YYYY-MM-DD].pdf` | |

### IP and regulatory documents
| Document type | Convention | Example |
|---|---|---|
| Patent | `Patent - [Title or Ref No] - [YYYY].pdf` | |
| Trademark registration | `Trademark - [Mark Name] - [Jurisdiction] - [YYYY].pdf` | |
| Regulatory licence | `[Licence Type] - [Issuing Body] - Expires [YYYY-MM-DD].pdf` | `Customs Broker Licence - CBP - Expires 2026-04-30.pdf` |
| Insurance policy | `[Insurer] - [Policy Type] - [YYYY-YYYY].pdf` | `AXA - D&O Insurance - 2024-2025.pdf` |

### When date is unknown
If you cannot determine the year from folder context or content search, use `[YYYY]` as a placeholder rather than guessing — this is a signal for the deal team to complete.

---

## Step 5 — Group proposals by naming pattern

Before presenting to the user, group the proposed renames by document category. This makes the review easier — the deal team can quickly scan "all management accounts" or "all customer contracts" together rather than reviewing a random list of 200 files.

Prepare the proposal in this structure per group:

```
GROUP: Management Accounts (8 files)
Naming convention: [Company] - Management Accounts - [Mon YYYY].pdf

Current name                    →  Proposed name
Scan001.pdf                     →  Apex Ltd - Management Accounts - Jan 2025.pdf
Scan002.pdf                     →  Apex Ltd - Management Accounts - Feb 2025.pdf
mgmt accounts march.pdf         →  Apex Ltd - Management Accounts - Mar 2025.pdf
MA_April2025_FINAL.pdf          →  Apex Ltd - Management Accounts - Apr 2025.pdf
...
```

---

## Step 6 — Present to the user for confirmation

**Before showing the table — mandatory pre-flight extension check:**
For every proposed rename, verify the extension in the proposed name exactly matches the extension in the original filename (case-insensitive). This check must pass 100% before the table is shown.

- Extract the extension from the original filename: everything after and including the last `.`
- Confirm the proposed name ends with the same extension (normalised to lowercase)
- If any proposed name is missing its extension or has a different extension: **correct it immediately** before showing the table — never show a proposed name without its extension
- Example: if the original is `Scan001.pdf`, the proposed name must end in `.pdf`. If you wrote `Apex Ltd - Audited Accounts - FY2024` without `.pdf`, add it now.

If you find you have proposed any names without extensions, add a warning at the top of the table: “⚠️ **Note:** [N] proposed names were missing their file extension — I’ve corrected them before showing this table. Please verify the extensions below are correct.”

Show the full grouped before/after table. Clearly state the total number of renames proposed.

End with:
> "I've proposed **[N] renames** across **[M] document categories**. Review the table above and let me know:
> - **'Apply all'** — I'll rename everything as proposed
> - **'Apply [group name]'** — I'll rename just that category
> - **Edit any row** — tell me what to change and I'll update the proposal
> - **Skip any file** — tell me which ones to leave as-is
>
> Nothing will be renamed until you confirm."

**Do not call `updateContent` until the user explicitly confirms.** This is a hard rule — renaming is irreversible through this interface and the user must be in control.

---

## Step 7 — Apply confirmed renames

Once the user confirms (all or a subset), apply renames using `updateContent`:

```
updateContent(projectId, metadataId, name="[proposed name with extension]")
```

**Hard rules — check each name immediately before calling `updateContent`:**
- **Extension must be present.** Before every single `updateContent` call, confirm the name string ends with `.pdf`, `.xlsx`, `.docx`, `.pptx`, or whatever the original extension was. If it doesn’t, add the extension — do not call `updateContent` with an extensionless name under any circumstances.
- **Extension must match the original.** The extension in the new name must be identical (lowercase) to the extension in the original filename. Never change `.pdf` to `.docx` or any other type.
- **Extension must be lowercase.** Normalise `.PDF` → `.pdf`, `.XLSX` → `.xlsx` before calling.
- Apply renames one at a time and track success/failure for each.
- If a rename fails, note it and continue with the rest.

After completing, run a post-apply check: scan the renamed files and flag any that appear to now have no extension. Report:
> “Done — **[N] files renamed** successfully. [If any failed:] **[X] renames failed** — [list them]. [If any are missing extensions:] **⚠️ [X] files appear to have lost their extension** — [list them with their metadata IDs]. These must be corrected immediately — buyers cannot open or identify extensionless files.”

---

## Step 8 — Flag for manual attention

Some files cannot be confidently renamed without human judgment. Flag these separately rather than guessing:
- Documents where the counterparty name is ambiguous or abbreviated in a way you can't resolve (e.g. `JD Contract 2022.pdf` — is "JD" a person or company?)
- Documents where the year is truly unclear after content search
- Documents in folders where the naming convention isn't obvious from context

Present these as: "**[N] files flagged for manual review** — I couldn't confidently determine the correct name: [list with current name and folder path]"

---

## Operating principles

**Batch by pattern, not by folder.** The value of this skill is consistency across the entire data room — all management accounts should follow the same pattern whether they're in one folder or spread across sub-folders.

**Counterparty name consistency is critical.** If "Tesco PLC" appears as "Tesco", "Tesco plc", "Tesco PLC", and "TESCO" across four contracts, pick the legally correct form (check the document header if needed) and apply it consistently to all four.

**Preserve all extensions.** A `.pdf` stays a `.pdf`. Never change the file type.

**Never guess a year.** A wrong year on an audited accounts file is worse than a placeholder `[YYYY]`. If the year isn't clear, mark it.

**Respect intentional names.** If a file already has a clear, professional, and consistent name (e.g. `Apex Ltd - Audited Accounts - FY2024.pdf`), don't rename it just because you can. Only rename files that genuinely need it.
