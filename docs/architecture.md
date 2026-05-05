# Architecture Deep Dive

## AI-Powered Client Onboarding & Proposal Generation Pipeline  
**Issachar Solutions · Built with n8n + Claude API**

---

## System Overview

This document explains the technical design decisions behind each stage of the pipeline. It is intended for developers, hiring managers, or collaborators who want to understand how the workflow is structured and why.

---

## Stage 1: Trigger & Data Parsing

### Tally — New Submission
The workflow is triggered by a Tally webhook. Tally was chosen as the form layer because it supports structured multi-field forms with a clean webhook payload, making downstream parsing straightforward.

### Parse Tally Fields
A JSON parsing node extracts and normalizes key fields from the raw Tally payload:
- Company name
- Industry
- Monthly budget range
- Primary pain point
- Contact email

Normalization at this stage (trimming whitespace, lowercasing industry strings) reduces noise before the data hits the AI enrichment layer.

---

## Stage 2: AI Data Enrichment

### Claude Haiku — Enrichment
**Model choice rationale:** Claude Haiku is used here because this is a high-frequency, low-complexity task. Haiku's speed and cost efficiency make it ideal for structured extraction and light reasoning.

**What it does:**
- Infers company size and maturity from the form data
- Tags the submission with relevant industry verticals
- Flags any ambiguous or incomplete fields for downstream handling
- Produces a structured JSON enrichment object

### Parse Enrichment
The Haiku output is parsed from JSON and mapped into a clean enrichment object that subsequent nodes can reference via n8n expressions.

---

## Stage 3: Budget Routing

### Budget Router
A Switch node routes the workflow into different research depth tiers based on the client's stated budget:

| Budget Range | Research Depth | Tool Recommendations |
|---|---|---|
| Under $500/mo | Light (skip competitor research) | Free / low-cost tools only |
| $500–$2,000/mo | Standard (all research branches) | Mid-tier SaaS stack |
| $2,000+/mo | Deep (full research + tool audit) | Enterprise-grade tools |

This routing ensures that compute and API costs scale proportionally with lead value, and that proposals feel appropriately scoped to what the client can realistically afford.

---

## Stage 4: Parallel Research Branches

All three research branches execute simultaneously, reducing total pipeline latency significantly compared to sequential execution.

### Research Industry (HTTP Request)
Makes an HTTP request to retrieve industry-specific data — market size, growth trends, common pain points, and regulatory context relevant to the client's vertical.

**Error handling:** An If node checks whether the HTTP response is valid. On failure, an Industry Fallback node injects a pre-built static industry summary so the workflow continues without breaking.

### Research Competitors (HTTP Request)
Makes a separate HTTP request to pull competitor intelligence for the client's industry and geography.

**Error handling:** Same pattern — Competitor Error? → Competitor Fallback. Fallback content is generic but structured to keep downstream nodes functional.

### Tool Audit (Claude Haiku)
Given the budget tier and enriched company profile, Claude Haiku evaluates which automation and AI tools are the best fit for this client. Output is a ranked list of tool recommendations with rationale.

**Error handling:** Tool Fallback injects a default tool recommendation set. A Skip Research path bypasses this branch entirely for the lowest budget tier.

---

## Stage 5: Merging & Context Assembly

### Merge
n8n's Merge node waits for all three parallel branches to complete (or fall back) before proceeding. This ensures the Assemble Context node always has a complete dataset regardless of which branches succeeded or fell back.

### Assemble Context
A Code node assembles all upstream data into a single structured context object:
```json
{
  "company": "...",
  "enrichment": { ... },
  "industry_research": { ... },
  "competitor_research": { ... },
  "tool_recommendations": [ ... ],
  "budget_tier": "standard"
}
```
This context object is passed to the synthesis stage as a single clean input.

---

## Stage 6: Multi-Model AI Synthesis

### Opus Synthesis (Claude Opus)
**Model choice rationale:** Claude Opus is reserved for the highest-complexity reasoning task in the pipeline — synthesizing all research findings into a coherent strategic narrative. The cost is justified because this output directly determines proposal quality.

**What it produces:**
- Executive summary of the client's situation
- Key pain points validated against research
- Strategic recommendations ranked by impact
- Risk factors and implementation considerations

### Parse Opus Output
The Opus response is parsed from structured output format into discrete blocks (summary, recommendations, risks) that the Sonnet node can reference individually.

### Sonnet Client Proposal (Claude Sonnet)
**Model choice rationale:** Claude Sonnet strikes the right balance between quality and cost for long-form writing. It takes the Opus strategic synthesis and writes the actual client-facing proposal — polished, professional, and appropriately scoped.

**What it produces:**
- Formal proposal with executive summary
- Recommended solution stack with justification
- Implementation timeline
- Investment breakdown aligned to budget tier
- Next steps and call to action

### Parse Proposal
The Sonnet output is parsed and mapped into document-ready blocks for the output stage.

---

## Stage 7: Output Generation

### Build Report Blocks
A transformation node structures the parsed proposal content into the specific format expected by Google Docs and Notion APIs — headings, body paragraphs, bullet lists, and table data.

### Create Client Record (Notion)
A new page is created in a Notion database with all enriched client data and a link to the generated proposal. This serves as the CRM entry for the sales team.

**Fields written:**
- Company name, industry, budget tier
- Contact email
- Enrichment tags
- Proposal document link
- Date created

### Create Suggestion Report Doc (Google Docs)
A Google Doc is created containing the tool recommendation report — a separate deliverable from the main proposal that gives the client a standalone reference for the suggested stack.

### Create Proposal Doc (Google Docs)
The full client proposal is written to a new Google Doc using the structured blocks from the Build Report Blocks node. The document is formatted with proper headings and sections.

### Create Client Intake Box
A folder or workspace is created to house all client documents, keeping deliverables organized from day one.

### Send Notification Email (Gmail)
A Gmail node sends a notification to the internal team (and optionally the client) with links to the created documents and a summary of key findings.

---

## Error Handling Philosophy

Every external HTTP call in this workflow has a corresponding error catch and fallback path. This was a deliberate design decision based on a core principle: **the workflow should never fail silently or crash mid-execution.**

Fallback nodes inject structured placeholder data that maintains the shape of a successful response, allowing downstream nodes to execute normally. The final output may be slightly less rich when a fallback fires, but it is always complete and coherent.

---

## Cost & Performance Considerations

| Task | Model | Rationale |
|---|---|---|
| Data enrichment | Claude Haiku | Fast, cheap, structured extraction |
| Tool audit | Claude Haiku | Moderate reasoning, high volume |
| Strategic synthesis | Claude Opus | Highest reasoning quality needed |
| Proposal writing | Claude Sonnet | Quality long-form, cost-efficient |

Using three different Claude models rather than a single model for everything reduces API costs by approximately 40–60% compared to running everything through Opus, while maintaining output quality where it matters most.

---

## What I Would Add in a Production Version

- **LangSmith or n8n execution logs** piped to a monitoring dashboard for observability
- **Human-in-the-loop approval step** using n8n's Wait node before the proposal is sent to the client
- **Retry logic** on HTTP research nodes with exponential backoff instead of immediate fallback
- **Vector store** (Pinecone or Supabase pgvector) to retrieve past successful proposals as RAG context for Sonnet
- **Slack integration** to notify the sales channel in real time with a preview of the proposal
- **Lead scoring node** using Claude Haiku to rank incoming leads by conversion likelihood before the pipeline runs

---

*Architecture documented by Isaac · Issachar Solutions · May 2026*
