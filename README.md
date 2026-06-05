# Datasite MCP server — data room skills

Eight coordinated skills for sell-side M&A deal teams using Datasite as their virtual data room platform.

## Skills

| # | Skill | What it does | Free | Blueflame |
|---|-------|-------------|:----:|:---------:|
| 1 | VDR index setup | Propose and push a deal-specific folder index to Datasite | ✅ | — |
| 2 | Smart file renaming | Standardize document names across the data room | ✅ | ✅ |
| 3 | Data room gap analysis | Identify missing, sparse, or incomplete sections before go-live | ✅ | ✅ |
| 4 | Document quality check | Audit documents for quality issues (7 free checks + 3 content checks) | ✅ | ✅ |
| 5 | IRL tracker | Match an information request list against VDR content; track delivery | ✅ | ✅ |
| 6 | Risk analysis audit | Surface risk signals across finance, tax, legal, HR, IP, commercial | ✅ | ✅ |
| 7 | Bulk Q&A answers | Draft sell-side answers to due diligence questions from VDR content | ✅ | ✅ |
| 8 | Launch readiness orchestrator | Single pre-go-live check → Go / no-go recommendation | ✅ | ✅ |

## Feature Tiers

| Tier | What you get |
|------|-------------|
| **Free (T1 Core)** | Structural tools: folder setup, metadata quality checks, filename-based gap analysis, file renaming from filename patterns |
| **Blueflame (T2+)** | AI content search: PII detection, IRL content matching, risk signal extraction, Q&A answer drafting, contract cross-referencing |

> **Important:** `searchDocuments` is the only permitted source of document content. Skills will never answer content-level questions using Claude's training knowledge — if Blueflame is not enabled, skills either fall back to metadata-only mode (Smart File Renaming, Document Quality Check) or stop and present the activation link.

## Requirements

- Datasite MCP server installed and authenticated (Blueflame AI)
- Claude desktop app (Cowork mode) or Claude Code with MCP support
- For Blueflame checks: Datasite AI search feature activated on the project

## Installation

Install each `.skill` file via the Claude desktop app → Skills → Install from file.

## Skills

### 1. VDR index setup
Reads deal context (sector, size, geography, transaction type) directly from the Datasite project, proposes a numbered folder hierarchy, iterates with the user, then pushes the confirmed structure to Datasite in a single call. Supports reference structures — if the user provides an existing index, asks whether to use it as-is, suggest additions, or fully adapt it.

### 2. Smart file renaming
Crawls the data room, identifies files with uninformative or inconsistent names, and proposes standardized names following deal conventions (e.g. `[Company] - Audited Accounts - FY2025.pdf`). Falls back to filename-pattern mode with placeholders if Blueflame is not enabled.

### 3. Data room gap analysis
Audits folder structure for missing sections, empty folders, sparse time-series coverage, and contract completeness (customer, employee, vendor cross-referencing). Uses last closed financial year (today's year − 1) as the base for year coverage checks. Offers HTML dashboard or plain text summary on completion.

### 4. Document quality check
Runs 10 checks across the data room: 7 free metadata checks (failed files, blanks, wrong formats, bad filenames, duplicates, version conflicts, stale docs) and 3 Blueflame content checks (PII exposure, redaction quality, broken references). Falls back to 7-check metadata-only mode if Blueflame is not enabled.

### 5. IRL tracker
Maps each item in an information request list against data room content using a three-pass approach: filename matching (free), keyword search, then semantic search. Assigns available / partially complete / open status with source citations. Offers interactive HTML tracking dashboard with .csv and .pdf export.

### 6. Risk analysis audit
Scans six workstreams (finance, tax, legal, HR, commercial, IP/ESG) for risk signals using targeted search queries. Includes cross-document data consistency checks (FTE, revenue, board roster) and an external news intelligence pass on top customers and vendors (free, web search). Flags budget misses, customer concentration, material customer loss, employee lawsuits, and M&A activity in the customer/vendor base.

### 7. Bulk Q&A answers
Reads a spreadsheet of due diligence questions, runs semantic and keyword searches per question, and drafts professional sell-side responses grounded in VDR content with full document and page-level citations. Offers Excel tracker and interactive Q&A management dashboard on completion.

### 8. Launch readiness orchestrator
Runs gap analysis, document quality check, and risk audit in a single pass and produces a consolidated go / no-go recommendation with a structured sign-off view. Read-only — does not modify the data room.

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
