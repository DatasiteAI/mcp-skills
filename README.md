# Datasite MCP server — data room skills

Eight coordinated skills for sell-side M&A deal teams using Datasite as their virtual data room platform.

## Skills

| # | Skill | What it does | MCP core operations (no AI document search required) | Requires AI document search access |
|---|-------|-------------|:----:|:---------:|
| 1 | VDR index setup | Propose and create a deal-specific VDR folder index (after user confirmation) | ✅ | — |
| 2 | Smart file renaming | Standardize document names across the data room | ✅ | ✅ |
| 3 | Data room gap analysis | Identify missing, sparse, or incomplete sections before go-live | ✅ | ✅ |
| 4 | Document quality check | Flag document quality issues (9 metadata checks + 3 content review flags) | ✅ | ✅ |
| 5 | IRL tracker | Suggest IRL matches from filenames, verify them against document content, and track delivery | ✅ | ✅ |
| 6 | Risk analysis audit | Show risk-section coverage; surface potential risk signals for diligence review | ✅ | ✅ |
| 7 | Bulk Q&A answers | Draft sell-side answers to due diligence questions from VDR content, for human review | ✅ | ✅ |
| 8 | Launch readiness orchestrator | Single pre-go-live readiness view (structural checks, plus content review flags) | ✅ | ✅ |

## What each capability needs

| Capability type | What it covers |
|------|-------------|
| **MCP core operations (no AI document search required)** | Uses project metadata — folder names and structure, filenames, file types, dates, counts, processing status and Q&A status: folder setup, metadata quality flags, structural gap analysis, tentative filename-based IRL matching, file renaming from filename patterns, Q&A counts and status |
| **Requires AI document search access** | Reads document content through `searchDocuments`: potential-PII and redaction flags, IRL content verification, potential risk signals, Q&A draft answers, contract coverage checks |

AI document search may be available through Blueflame, Connect, or approved partner entitlements.

> **Important:** `searchDocuments` is the only permitted source of document content. Skills will never answer content-level questions using Claude's training knowledge — if AI document search access is not available, skills either fall back to metadata-only mode (Smart File Renaming, Document Quality Check) or stop and present the activation link.

Each skill states which results come from project metadata (MCP core operations) and which require reading document content (AI document search access). Generated reports mark content checks that were not run as **NOT RUN**.

## Requirements

- Datasite MCP server installed and authenticated (Blueflame AI)
- Claude desktop app (Cowork mode) or Claude Code with MCP support
- For content checks: AI document search access on the project (available through Blueflame, Connect, or approved partner entitlements)

## Installation

Install each `.skill` file via the Claude desktop app → Skills → Install from file.

## Skills

### 1. VDR index setup
Reads deal context (sector, size, geography, transaction type) directly from the Datasite project, proposes a numbered folder hierarchy, iterates with the user, then creates the confirmed structure in Datasite in a single call (requires the user's Datasite permissions). Supports reference structures — if the user provides an existing index, asks whether to use it as-is, suggest additions, or fully adapt it.

### 2. Smart file renaming
Crawls the data room, identifies files with uninformative or inconsistent names, and proposes standardized names following deal conventions (e.g. `[Company] - Audited Accounts - FY2025.pdf`). Falls back to filename-pattern mode with placeholders if AI document search access is not available. Reading document content resolves more names, but some ambiguous filenames may still need manual review.

### 3. Data room gap analysis
Audits folder structure for missing sections, empty folders, sparse time-series coverage, and potential contract coverage gaps (customer, employee, vendor cross-referencing; requires AI document search access). Structural findings are not proof that all required documents are present. Uses last closed financial year (today's year − 1) as the base for year coverage checks. Offers HTML dashboard or plain text summary on completion.

### 4. Document quality check
Runs 12 checks across the data room: 9 metadata checks (failed or still-processing files, blank or near-blank files, possibly incomplete scans, placeholders, duplicates, version conflicts, wrong formats, stale documents, uninformative filenames) and 3 content review flags (potential PII, possible redaction issues, broken references) that require AI document search access. Metadata flags cannot confirm a document is readable or complete; content flags are not a validated PII scan or proof of secure redaction. Falls back to 9-check metadata-only mode if AI document search access is not available.

### 5. IRL tracker
Maps each item in an information request list against data room content: filename and folder matching first (no AI document search required; tentative matches only), then keyword and semantic search to verify content (AI document search access). Assigns available / partially complete / open status with source citations; requests are never marked complete from filenames alone. Offers interactive HTML tracking dashboard with .csv and .pdf export.

### 6. Risk analysis audit
Reports which risk-related sections are present, empty or sparse (no AI document search required). With AI document search access, scans six workstreams (finance, tax, legal, HR, commercial, IP/ESG) for potential risk signals using targeted search queries, with cross-document data consistency checks (FTE, revenue, board roster). Includes an external news intelligence pass on top customers and vendors (web search; no AI document search required). Flags potential budget misses, customer concentration, material customer loss, employee lawsuits, and M&A activity in the customer/vendor base — as leads for diligence review, not confirmed conclusions.

### 7. Bulk Q&A answers
Reads a spreadsheet of due diligence questions, runs semantic and keyword searches per question, and drafts professional sell-side responses grounded in VDR content with full document and page-level citations. Drafts are for human review and are never submitted to Datasite Q&A automatically. Offers Excel tracker and interactive Q&A management dashboard on completion.

### 8. Launch readiness orchestrator
Runs gap analysis, document quality check, and risk audit in a single pass and produces a consolidated readiness view with a structured sign-off checklist. Content checks that were not run are marked NOT RUN, and a metadata-only report never shows the room as ready to go live. Read-only — does not modify the data room.

## Repository Structure

```
skills/
├── vdr-index-setup/SKILL.md
├── smart-file-renaming/SKILL.md
├── gap-analysis/SKILL.md
├── document-quality-check/SKILL.md
├── irl-tracker/SKILL.md
├── risk-analysis-audit/SKILL.md
├── bulk-qa-answers/SKILL.md
└── launch-readiness-orchestrator/SKILL.md
```

## Author

Blueflame AI — [blueflame-ai.com](https://blueflame-ai.com)
