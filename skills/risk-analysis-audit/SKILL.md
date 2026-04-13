---
name: risk-analysis-audit
description: >
  Risk Analysis Audit skill for Datasite deal rooms. Use this skill whenever a sell-side
  deal team wants to audit, review, or flag risks across a data room before going live.
  Triggers include: "run a risk audit", "flag risks in the data room", "risk review",
  "what are the risks in this deal", "audit the data room", "risk analysis", "flag issues
  before we go live", "what should we fix before launch", or any request to analyse deal
  risk by workstream (Tax, Finance, Legal, HR, IP, Commercial, Regulatory, ESG).
  Use this skill proactively whenever the user is preparing a data room for launch and
  wants a structured view of what might concern a buyer.
---

# Risk Analysis Audit

You are helping a sell-side deal team identify and understand risks across their Datasite data room before it goes live to buyers. Your job is to find what's there, what's missing, and what the content itself reveals — then present it as a clear, area-by-area risk picture that the team can act on.

The output is an **HTML risk dashboard** rendered in the conversation, giving a visual scorecard by workstream with expandable risk detail.

---

## Terminology — fileroom vs. folder

Use these terms precisely when communicating with the user:

- **Fileroom** — the single top-level container inside a Datasite project. A project typically has one buyer-facing fileroom. It is not a subject area — it is the container that holds all subject areas.
- **Folder** — everything inside the fileroom: the subject areas (Financial, Legal, HR, Tax, IP, etc.) and all sub-levels beneath them. Always call these folders, never filerooms.

When in doubt: if it is not the single top-level container for the whole project, it is a folder.


## Feature Requirements

| Capability | Free | Requires Blueflame |
|---|:---:|:---:|
| Structural presence check (which sections exist or are missing) | ✅ | — |
| Content risk signals (going concern, litigation, tax disputes, etc.) | — | ✅ |
| Per-workstream risk findings with source citations | — | ✅ |

**Without Blueflame:** The skill can confirm which risk workstream sections are present, sparse, or missing — but cannot find risk signals inside document text. The report will show structural observations only. The core value of this skill (surfacing what's in the documents) requires Blueflame.

**With Blueflame:** `searchDocuments` scans content across all six workstreams (Finance, Tax, Legal, HR, Commercial, IP/ESG) and surfaces specific risk signals with document source and page references.



> ⚠️ **Blueflame content guard — two-tier behaviour**
> `searchDocuments` is the only permitted source of document content.
> - Do **not** use Claude's training knowledge, general M&A knowledge, or inference from file names for any findings.
> - **Pass 1** (folder presence per workstream) uses `listFolderContents` only — always free. Complete Pass 1 across all workstreams first.
> - **Pass 2** (content search for risk signals) requires `searchDocuments`. Before starting Pass 2, attempt one call. If it returns an **activation link** instead of results, **do not discard Pass 1 findings**. Present them first, then say:
>
>   > "I've completed the structural review across all workstreams — you can see which sections are present, sparse, or missing above. To scan document content for actual risk signals (declining margins, open disputes, undisclosed litigation, IP ownership gaps), Blueflame AI search needs to be activated:
>   > 🔗 **Activate Blueflame:** [activation link]
>   > **With Blueflame:** I'll run targeted searches across Finance, Tax, Legal, HR, Commercial, and IP workstreams and surface specific flags with document source and page references — these are the signals that matter most to buyers in due diligence.
>   > Would you like to activate now to complete the content risk scan?"
>
> - All content findings **must** be sourced exclusively from tool results.

> **`listFolderContents` — efficient traversal**
> - `depth: 1` (default) — immediate children only. Use for targeted lookups.
> - `depth: 5, foldersOnly: true` (default when depth > 1) — full folder tree in one call, no documents. Use for structural checks.
> - `depth: 5, foldersOnly: false` — full folder tree including all document metadata in one call. Use when building a document inventory.
> - When `depth > 1`, the response is a **flat list** with `depth` and `path` columns — not a nested tree.

## Step 1 — Orient yourself in the project

Call `getProjectOverview` to understand the deal context: company name, sector, transaction type, deal size, and what filerooms exist. This shapes which risk areas matter most and how deeply to search.

Note the fileroom structure. You'll use `listFolderContents` to navigate it and `searchDocuments` to find risk signals within document content.

---

## Step 2 — Two-pass analysis per risk area

Work through each of the six risk workstreams below. For each one, run **two passes**:

**Pass 1 — Presence check (structural)**
Use `listFolderContents` to navigate the relevant section of the data room. For each expected document category, note:
- ✓ Present — folder exists and contains documents
- ⚠ Sparse — folder exists but appears empty or has fewer documents than expected
- ✗ Missing — folder or document category absent entirely

**Pass 2 — Content scan (substantive)**
Use `searchDocuments` with targeted queries (listed per workstream below) to surface risk signals from within document text. The tool returns snippets — read them for red flags. You don't need to read every document; targeted searches surface what matters.

Combine both passes to form your findings for that workstream.

---

## Workstream 1 — Financial & Accounting

**What to look for structurally:**
- Audited financial statements (last 3 years minimum — or 5 for large-cap)
- Management accounts (recent months)
- Financial model / projections
- Debt schedule and loan agreements
- Working capital analysis

**Search queries to run:**
- `"going concern"` — auditor qualification is a major red flag
- `"restatement"` or `"restated"` — prior period corrections signal accounting weakness
- `"breach of covenant"` or `"covenant default"` — debt covenant issues
- `"material weakness"` or `"significant deficiency"` — internal control failures
- `"related party"` — undisclosed or unusual related-party transactions
- `"deferred revenue"` or `"revenue recognition"` — quality of earnings signal
- `"contingent liability"` — off-balance sheet exposure
- `"capitalised"` or `"capitalization policy"` — aggressive capitalisation of costs (R&D, software dev, customer acquisition) that inflates EBITDA
- `"impairment"` — goodwill or asset write-downs signalling prior overpayment or deteriorating business units
- `"change in accounting policy"` or `"change in estimate"` — mid-stream policy shifts that flatter comparability
- `"earn-out"` or `"earnout"` — contingent consideration from prior acquisitions still on the balance sheet
- `"intercompany"` — intra-group transactions that may distort standalone financials
- `"normalisation"` or `"add-back"` — seller-proposed EBITDA adjustments that may not be defensible
- `"budget"` or `"forecast"` or `"variance"` — budget performance and miss analysis
- `"churn"` or `"attrition"` or `"lost customer"` or `"customer cancellation"` — customer loss signals
- `"concentration"` or `"top customer"` or `"largest customer"` — revenue concentration signals
- `"revenue decline"` or `"declining revenue"` or `"revenue shortfall"` — deteriorating top line
- `"margin pressure"` or `"margin decline"` or `"gross profit"` — margin trend signals

**Risk signals to flag:**
- Going concern note or audit qualification → **High**
- Restatements in last 3 years → **High**
- Covenant breaches or waivers → **High**
- Large or growing gap between reported EBITDA and operating cash flow → **High** (earnings quality concern — cash conversion below ~70% warrants scrutiny)
- Significant variance between management accounts and audited figures → **High**
- Aggressive cost capitalisation inflating reported EBITDA → **High**
- Revenue declining year-on-year for 2+ consecutive years → **High** (structural deterioration, not cyclical)
- Gross margin compression over 3 years with no explanation → **High** (signals pricing pressure, cost creep, or mix shift)
- Budget missed materially (>10% variance on revenue or EBITDA) in any of the last 3 years → **High** if recurring, **Medium** if isolated; cross-reference actual vs. budget in management accounts and board packs
- Material customer loss in last 12–24 months (named customer present in prior year revenue schedules but absent in current year) → **High** (revenue cliff risk)
- Top customer representing >20% of revenue → **High**; top 3 customers representing >50% → **High** (concentration risk — buyer will haircut valuation)
- Revenue concentration increasing year-on-year (fewer customers driving more revenue) → **Medium** to **High** depending on degree
- Significant related-party transactions without clear rationale → **Medium**
- Intercompany loans or management charges with no arm's-length basis → **Medium**
- Goodwill impairment charge in last 2 years → **Medium** (prior acquisition underperformance)
- Earn-out liabilities from prior acquisitions still outstanding → **Medium**
- Auditor change in last 3 years with no clear rationale → **Medium** (potential disagreement with prior auditor)
- Accounts receivable days (DSO) increasing materially year-on-year → **Medium** (collection risk or channel stuffing)
- Inventory build without corresponding revenue growth → **Medium** (demand signal or obsolescence risk)
- Missing or incomplete management accounts → **Medium**
- No financial model or projections → **Medium**
- Declining margins with no cost reduction plan → **Medium** to **High** depending on trajectory

---

## Workstream 2 — Tax

**What to look for structurally:**
- Filed federal/national tax returns (last 3 years)
- State/local returns (US) or VAT returns (UK/EU)
- Correspondence with tax authorities
- Tax disputes and assessments

**Search queries to run:**
- `"tax audit"` or `"tax examination"` — open audits
- `"assessment"` or `"notice of deficiency"` — tax authority demands
- `"penalty"` or `"interest"` — outstanding liabilities
- `"transfer pricing"` — cross-border risk, especially for multi-jurisdiction deals
- `"R&D credit"` or `"tax credit"` — potentially aggressive positions worth querying
- `"amended return"` — prior corrections
- `"uncertain tax position"` or `"FIN 48"` or `"ASC 740"` — disclosed tax risk reserves
- `"nexus"` or `"economic nexus"` — unregistered state/local tax obligations (US-specific, particularly post-Wayfair)
- `"withholding tax"` — cross-border payment obligations, especially on dividends or IP royalties
- `"permanent establishment"` — unintended taxable presence in foreign jurisdictions
- `"NOL"` or `"net operating loss"` or `"tax loss carryforward"` — value of deferred tax assets and any limitations on use post-acquisition (e.g. Section 382 in the US)
- `"BEAT"` or `"GILTI"` or `"Pillar Two"` — exposure to international tax reform regimes
- `"VAT"` or `"sales tax"` or `"indirect tax"` — unregistered or under-collected indirect tax obligations
- `"deferred tax"` — large deferred tax liabilities that reduce net asset value

**Risk signals to flag:**
- Open tax dispute or assessment → **High**
- Missing returns for any period in the last 3 years → **High**
- Correspondence with tax authority showing unresolved position → **High**
- Permanent establishment risk in jurisdictions where the company operates but is not registered → **High**
- Tax structure reliant on a specific ownership form that changes post-acquisition → **High** (e.g. S-Corp or partnership elections that terminate on sale)
- Unregistered sales tax / VAT obligations across multiple jurisdictions → **High** (particularly for e-commerce or SaaS businesses with broad customer bases)
- Aggressive positions (large R&D credits, transfer pricing arrangements) → **Medium**
- Amended returns in last 3 years → **Medium**
- No tax advice letters or counsel opinions on complex positions → **Medium**
- Section 382 limitation on NOL carryforwards post-change of control → **Medium** (reduces deferred tax asset value assumed in the model)
- Withholding tax exposure on intercompany royalties or dividends → **Medium**
- Large deferred tax liability not reflected in purchase price adjustments → **Medium**
- No documentation supporting R&D credit methodology → **Medium** (IRS/HMRC challenge risk)

---

## Workstream 3 — Legal, Litigation & Regulatory

**What to look for structurally:**
- Pending/threatened litigation schedule
- Material contracts (particularly change of control clauses)
- Regulatory licences and their expiry dates
- Regulatory correspondence and enforcement history
- Insurance schedule

**Search queries to run:**
- `"litigation"` or `"claim"` or `"proceedings"` — active disputes
- `"arbitration"` or `"mediation"` — alternative dispute proceedings
- `"employment tribunal"` or `"employment claim"` or `"wrongful dismissal"` or `"unfair dismissal"` — employee lawsuits and tribunal history
- `"discrimination"` or `"harassment claim"` or `"hostile work environment"` — workplace claim history
- `"redundancy"` or `"collective redundancy"` or `"WARN Act"` or `"layoff"` — recent redundancy programmes (check for TUPE/WARN obligations and risk of claims from affected employees)
- `"change of control"` — contracts that terminate or require consent on a sale
- `"termination for convenience"` — key contracts that can be exited easily by counterparty
- `"regulatory"` or `"enforcement"` or `"violation"` — compliance issues
- `"injunction"` or `"restraining order"` — court orders in force
- `"settlement"` — prior claims settled (pattern matters)
- `"material adverse"` — MAC clauses that could affect deal pricing
- `"consent order"` or `"cease and desist"` — prior regulatory actions
- `"class action"` or `"collective action"` — systemic litigation exposure
- `"antitrust"` or `"competition"` or `"cartel"` — competition law exposure
- `"FCPA"` or `"Bribery Act"` or `"anti-corruption"` — anti-bribery compliance
- `"product liability"` or `"recall"` — consumer/product risk
- `"data subject"` or `"subject access request"` — GDPR enforcement signals
- `"permit"` or `"licence renewal"` — operational permits beyond just regulatory licences
- `"indemnification"` or `"hold harmless"` — inherited liability from prior transactions

**Risk signals to flag:**
- Undisclosed or material litigation → **High**
- Active class action or collective claim → **High**
- Pattern of employee claims (multiple employment tribunal filings, discrimination or harassment settlements) → **High** (systemic culture/HR risk, not isolated incidents)
- Recent redundancy programme (last 12–24 months) with no severance agreement documentation or potential unfair dismissal claims → **High** if undocumented; **Medium** if properly documented
- Antitrust investigation or market inquiry involvement → **High**
- FCPA / Bribery Act exposure in international markets → **High**
- Consent orders or prior regulatory sanctions → **High**
- Operating permits (environmental, zoning, health) not transferable on change of ownership → **High**
- Change of control clauses in key customer or supplier contracts → **High**
- Regulatory licence expiring within 12 months → **High**
- Product liability claims or product recalls in last 5 years → **High** (if unresolved) / **Medium** (if resolved with no recurrence)
- Pattern of repeat settlements across similar claim types → **High** (systemic issue, not one-off)
- Prior employment claims settled without admission of liability but with NDA — multiple instances → **Medium** to **High** (culture signal)
- Indemnification obligations inherited from prior acquisitions → **Medium**
- Key contracts terminable for convenience with short notice → **Medium**
- Insurance coverage gaps or claims history → **Medium**
- D&O or E&O insurance with coverage gaps relative to deal size → **Medium**
- No legal entity organisational chart or unclear corporate structure → **Medium** (complicates title transfer and warranty scope)
- Missing standard contracts (e.g. no NDAs with key partners) → **Medium**

---

## Workstream 4 — HR & Employment

**What to look for structurally:**
- Employee list (especially senior/licensed staff)
- Key employment agreements
- Non-compete and non-solicitation agreements
- Benefits, pension, and incentive plans
- Any redundancy, grievance, or disciplinary records

**Search queries to run:**
- `"key person"` or `"key employee"` — dependency signals
- `"non-compete"` or `"non-solicitation"` — scope and enforceability
- `"redundancy"` or `"layoff"` or `"termination"` — recent restructuring activity
- `"grievance"` or `"disciplinary"` or `"misconduct"` — HR disputes
- `"equal employment"` or `"discrimination"` — employment claims
- `"pension deficit"` or `"unfunded liability"` — benefit plan exposure
- `"golden parachute"` or `"change in control payment"` — deal-triggered compensation costs
- `"worker classification"` or `"independent contractor"` or `"IR35"` — misclassification risk
- `"TUPE"` or `"WARN Act"` — transfer of undertakings or mass layoff notification obligations
- `"equity"` or `"option"` or `"LTIP"` or `"phantom"` — unvested equity that accelerates at close
- `"immigration"` or `"visa"` or `"work permit"` — key staff whose right to work depends on current employer sponsorship
- `"whistleblower"` or `"protected disclosure"` — active or prior whistleblower complaints
- `"harassment"` or `"hostile work environment"` — culture and liability signals
- `"union"` or `"collective bargaining"` or `"works council"` — organised labour obligations

**Risk signals to flag:**
- Key person concentration with no succession or retention plan → **High**
- Deal-triggered compensation (golden parachutes, accelerated vesting) → **High**
- Active whistleblower complaint or protected disclosure → **High**
- Union or works council with consultation rights triggered by the transaction → **High** (can delay or block close in certain jurisdictions)
- Contractor misclassification exposure (IR35 / US worker classification) → **High** (significant back-tax and employment rights liability)
- TUPE / WARN Act obligations not planned for → **High** (deal structure may trigger mandatory employee transfer or notification obligations)
- Large unvested equity pool accelerating at close → **High** (direct cash cost or dilution impact on deal economics)
- Pension or benefit plan deficit → **High**
- Active employment claims or tribunal proceedings → **Medium**
- Key staff on employer-sponsored visas with no portability → **Medium**
- Non-competes missing for senior employees or founders → **Medium**
- Disproportionately high attrition in 12 months pre-sale → **Medium** (seller managing headcount ahead of exit)
- Equity or bonus plans with no good leaver / bad leaver provisions → **Medium**
- No documented HR policies (anti-harassment, grievance procedure) → **Medium**
- Recent unexplained departures of senior staff → **Medium**
- No employment agreements for key staff → **Medium**

---

## Workstream 5 — Commercial & Contracts

**What to look for structurally:**
- Top customer contracts (especially top 5–10 by revenue)
- Supplier and vendor agreements
- Distribution and agency agreements
- Contract expiry/renewal schedule

**Search queries to run:**
- `"exclusive"` — exclusivity obligations restricting the buyer post-acquisition
- `"most favoured"` or `"MFN"` — pricing commitments that compress margins
- `"minimum purchase"` or `"minimum commitment"` — take-or-pay obligations
- `"assignment"` or `"novation"` — contracts that cannot be transferred on sale
- `"expir"` or `"renew"` — contracts expiring near-term
- `"price increase"` or `"escalation"` — cost commitments
- `"warranty"` or `"indemnity"` — contractual liability exposure
- `"liquidated damages"` or `"service level"` or `"SLA"` — penalty exposure for underperformance
- `"auto-renew"` or `"evergreen"` — contracts that roll without active renewal
- `"price cap"` or `"fixed price"` — contracts that limit ability to pass through cost inflation
- `"right of first refusal"` or `"ROFR"` or `"right of first offer"` — pre-emption rights that could complicate the sale
- `"non-disparagement"` or `"confidentiality"` — restrictions limiting what the company can say about prior disputes
- `"rebate"` or `"volume discount"` — margin-eroding commitments not visible in headline pricing

**Risk signals to flag:**
- Customer concentration (top 3 customers >50% revenue) → **High**
- Key customer contracts not assignable without consent → **High**
- Right of first refusal held by a customer or partner over the business or its assets → **High** (could block or complicate the transaction)
- Single-source supplier with no alternative and no long-term agreement → **High** (supply chain concentration)
- Distribution or agency agreements with termination payments on change of control → **High**
- SLA penalty clauses with material financial exposure → **High**
- Major contracts expiring within 6 months with no renewal evidence → **High**
- MFN or exclusivity clauses that limit buyer's commercial flexibility → **Medium**
- Fixed-price or price-capped contracts in an inflationary cost environment → **Medium** (margin compression risk)
- Material take-or-pay obligations → **Medium**
- Auto-renewing contracts with unfavourable terms that have rolled without renegotiation → **Medium**
- Rebate or volume discount commitments not reflected in management accounts → **Medium**
- No standard terms and conditions for customer sales → **Medium** (unlimited liability exposure)
- Missing contracts for known major customers or suppliers → **Medium**

---

## Workstream 6 — IP, Technology & ESG

### IP & Technology

**What to look for structurally:**
- IP ownership documentation (patents, trademarks, registered rights)
- IP assignments from founders and employees
- Open-source software inventory
- Data privacy and cybersecurity policies
- IT system and licence agreements

**Search queries to run:**
- `"open source"` or `"GPL"` or `"AGPL"` — copyleft licence exposure (could force disclosure of proprietary code)
- `"infringement"` or `"misappropriation"` — IP disputes
- `"assigned"` or `"assignment"` in IP context — confirm IP transferred to the company, not held personally by founders
- `"data breach"` or `"incident"` or `"breach notification"` — security events
- `"GDPR"` or `"data protection"` or `"personal data"` — compliance posture
- `"source code escrow"` — third-party obligations over proprietary software
- `"trade secret"` or `"confidential information"` — whether proprietary know-how is formally protected
- `"licence"` or `"royalty"` — third-party IP licences the company depends on
- `"end of life"` or `"end of support"` — legacy systems on unsupported software
- `"penetration test"` or `"vulnerability assessment"` — whether cybersecurity posture has been formally tested
- `"SOC 2"` or `"ISO 27001"` or `"Cyber Essentials"` — presence or absence of security certifications
- `"ransomware"` or `"phishing"` or `"malware"` — specific incident types

**IP & Technology risk signals:**
- IP not formally assigned from founders/key developers → **High**
- Copyleft open-source components (GPL/AGPL) in proprietary product → **High**
- Active IP infringement claim or dispute → **High**
- Data breach or security incident in last 3 years → **High**
- Inbound IP licences that terminate or require consent on change of control → **High**
- Key technology dependent on a single third-party licence with no alternative → **High**
- GDPR/data privacy programme absent or inadequate → **Medium**
- No formal trade secret protection programme (NDAs, access controls, documentation) → **Medium**
- No software licence inventory → **Medium**
- Legacy systems running end-of-life software with no upgrade roadmap → **Medium**
- No SOC 2 / ISO 27001 certification for a B2B SaaS or data-handling business → **Medium** (increasingly a customer contract requirement)
- Cybersecurity posture never formally tested (no pen test in last 2 years) → **Medium**
- Critical infrastructure on a single cloud provider with no DR plan → **Medium**
- No documented software development lifecycle or version control → **Medium**

### ESG

**What to look for structurally:**
- Environmental compliance certificates and violation history
- Health & safety incident records
- Diversity and inclusion policies
- Modern Slavery Act statement (required for UK businesses >£36M turnover)

**Search queries to run:**
- `"environmental violation"` or `"contamination"` or `"remediation"` — environmental liability
- `"health and safety"` or `"incident"` or `"fatality"` — H&S record
- `"sanctions"` or `"OFAC"` or `"AML"` — compliance flags
- `"carbon"` or `"net zero"` or `"emissions"` — climate commitments and Scope 1/2/3 exposure
- `"modern slavery"` or `"supply chain audit"` — Modern Slavery Act compliance
- `"child labour"` or `"forced labour"` — supply chain human rights risk
- `"DEI"` or `"pay gap"` or `"gender pay"` — regulatory reporting obligations

**ESG risk signals:**
- Supply chain with known exposure to high-risk jurisdictions (forced/child labour) → **High**
- Pending or historic environmental enforcement action not disclosed in the data room → **High**
- No Modern Slavery Act statement for a UK business above the turnover threshold → **High** (regulatory non-compliance)
- Environmental violation or remediation liability → **High** (if active) / **Medium** (if historic and resolved)
- Carbon reduction commitments made publicly but no internal roadmap to support them → **Medium** (greenwashing liability)
- Scope 1/2 emissions data absent for a business with LP ESG reporting requirements → **Medium**
- Gender pay gap reporting obligation missed or overdue → **Medium**
- No board-level ESG ownership or policy → **Medium**
- Missing H&S records → **Medium**

---

## Step 3 — Compile findings

After completing all six workstreams, compile your findings into a structured list:

```
findings = [
  {
    area: "Tax",
    severity: "High",
    title: "Open IRS audit for FY2023",
    detail: "Correspondence in folder 3.6 references an open IRS examination for tax year 2023. No resolution letter found.",
    source: "3.6 IRS Correspondence / Letter dated March 2024"
  },
  ...
]
```

Also track structural gaps separately:
```
gaps = [
  { area: "Finance", item: "Working capital analysis — folder empty" },
  { area: "HR", item: "Non-compete agreements — folder missing entirely" },
  ...
]
```

Count risks by severity per area — this drives the dashboard scorecard.

---


## Step 3b — Cross-document data consistency checks

Run the following consistency checks across the data room. These use `searchDocuments` to pull specific figures from different document types and compare them. Discrepancies are flagged as **Medium** risks minimum; large discrepancies are **High**.

### Headcount / FTE consistency
Find headcount figures in the following document types and compare them:
- P&L or financial statements (FTE cost line or employee note)
- HR employee list or org chart
- Board presentations or management accounts (FTE KPI)
- Any regulatory filings that reference employee numbers

Flag if the figures differ by more than 10% across sources, or if any source gives a materially different total. Note the specific sources and figures found.

### Top customer list consistency
If a "top 20 / top 50 customers" list exists, cross-check customer names and revenue figures against:
- Financial statements or revenue schedules
- CRM or sales data (if present)
- Any investor presentation or board pack referencing customer concentration

Flag customers who appear in one list but not another, or where revenue attributions differ materially.

### Top vendor / supplier spend consistency
If a vendor spend list exists, cross-check against:
- P&L cost line items (COGS, OpEx breakdown)
- Any procurement or spend analysis document

Flag if total vendor spend implied by the list is materially inconsistent with cost lines in the financials.

### Board and management roster consistency
Cross-check board member and senior management names across:
- Corporate documents (articles, board minutes, Companies House / registry filings)
- Org chart
- Employment contracts or service agreements
- Any investor or management presentation

Flag any person who appears in one source but not another (e.g. listed as a director in board minutes but absent from the org chart, or named in a management presentation but with no service agreement).

### Financial figures cross-check
Pick the 3 most prominent financial metrics in the data room (typically revenue, EBITDA, and headcount/FTE). Verify they are stated consistently across:
- Audited accounts
- Management accounts
- Board presentations / investor decks
- Any teaser or information memorandum

Flag any material discrepancy (>5% difference) as a **High** risk — buyers will spot these immediately and it will undermine confidence in the whole data room.

---

## Step 3c — External news intelligence on top customers and vendors

> This step uses web search, not `searchDocuments`. It is always free — no Blueflame credits required.

Extract the names of the top 5–10 customers and top 5–10 vendors from the data room (use the customer/vendor lists found in Step 3b, or from commercial documents identified in Workstream 5). Then run a targeted web news search for each name.

**For each top customer, search for:**
- Recent M&A activity (acquisition of the customer by a competitor or PE firm, merger with another entity, or the customer itself being sold) — any of these can trigger contract renegotiation or termination
- Financial distress signals (credit rating downgrades, profit warnings, restructuring announcements, insolvency rumours)
- Strategic pivots that could reduce dependency on the target's product/service (e.g. in-housing, switching to a competitor)
- Leadership changes (new CEO/CPO/CTO) — often precede vendor reviews
- Regulatory or legal issues that could disrupt the customer's own operations

**For each top vendor, search for:**
- M&A activity (vendor acquired by a competitor, merged, or restructuring) — may affect pricing, continuity, or exclusivity
- Financial distress or supply chain disruption signals
- Geopolitical exposure (sanctions, trade restrictions, country-of-origin risk)
- Price escalation announcements or force majeure notices

**How to run the search:**
Use web search with queries in the format: `"[Customer/Vendor Name]" news 2024 2025 acquisition OR merger OR restructuring OR insolvency OR "strategic review"`. Run a separate query for each name. If a name is generic (e.g. "Global Logistics Ltd"), add the sector or country to disambiguate.

**Risk signals to flag:**
- Top customer acquired by a known competitor of the target → **High** (high probability of contract review or termination post-close)
- Top customer in financial distress or undergoing restructuring → **High** (revenue at risk)
- Top customer announced vendor consolidation or platform shift → **High**
- Top vendor acquired by a company with conflicting interests → **High** (supply continuity risk)
- Top vendor subject to sanctions or trade restrictions → **High**
- M&A activity in the customer or vendor base with no change of control provision in the relevant contract → **Medium** (contract does not protect the target)
- New leadership at a key customer with no relationship established → **Medium**
- Any news (positive or negative) about a customer or vendor that is not reflected anywhere in the data room → **Medium** (disclosure gap — buyer will find it)

**Present findings as:** a table with columns: Name | Type (Customer/Vendor) | News Found | Risk Level | Source URL | Recommended Action.

If no material news is found for a name, record "No material news found" and continue. Do not skip this step — a clean result is itself a valuable finding.

---

## Step 4 — Offer outputs

Before generating the dashboard, ask:

> "I've completed the risk audit. Would you like me to generate the interactive HTML risk dashboard, or would a plain text risk summary in this conversation be enough? The dashboard uses additional credits to render — the plain text summary is free."

Only build the dashboard if the user confirms. If they decline, go to Step 5 and deliver a plain text summary.

### Dashboard specification (build only on user confirmation)

Generate a self-contained HTML page and write it as an artifact. The dashboard should include:

**Header:**
- Deal name, date of audit, total risk counts (High / Medium / Low)

**Risk Scorecard (top section):**
- Six area tiles, each showing: area name, risk counts (H/M/L), and a colour signal:
  - Any High → red tile border
  - Only Medium/Low → amber tile border
  - No findings → green tile border

**Detailed findings (below the scorecard):**
- Grouped by workstream
- Each finding shows: severity badge (colour-coded), title, detail text, and source reference
- A "Structural gaps" sub-section per area listing missing or sparse folders

**Style guidance:**
- Clean, professional — this will be shared in deal team meetings
- White background, dark headings, muted colour palette
- Severity badges: High = red (#DC2626), Medium = amber (#D97706), Low = grey (#6B7280)
- No external dependencies — fully self-contained HTML/CSS/JS

---

## Step 5 — Present to the user

After rendering the dashboard, give a brief verbal summary:

> "I've audited [N] sections of the data room and found [X] High, [Y] Medium, and [Z] Low risks. The areas with the most critical issues are [list]. Each finding is tagged with its source document so you can locate it directly in the data room."

Then offer:
> "Want me to export this as an Excel risk register, or shall we work through any of the High risks in more detail?"

---

## Operating principles

**Search intelligently, not exhaustively.** Run the targeted queries above. Don't attempt to read every document — the snippets are sufficient. If a snippet is ambiguous, run a follow-up search to confirm before flagging.

**Be specific about sources.** Every finding must reference the folder path or document name. Vague findings ("there may be tax issues") are not useful — deal teams need to go straight to the source.

**Calibrate to deal size.** A risk that is High for a £10M SME may be Medium for a £500M transaction where diligence coverage is deeper and warranties are broader. Use `transactionValue` from the project metadata to calibrate.

**Don't over-flag.** Not everything unusual is a risk. A non-compete that looks standard, or a customer contract that's long-dated and unconditional, should not be flagged just because it appeared in a search. Flag what a diligent buyer's counsel would genuinely raise.

**Sell-side framing.** This audit is for the team preparing the room, not buyers. Frame findings as things to address, disclose, or explain — not as reasons to walk away.
