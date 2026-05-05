# AI-Powered Client Onboarding & Proposal Generation Pipeline

> Built by [Isaac](https://github.com/yourusername) · Issachar Solutions  
> Stack: n8n · Claude API (Haiku + Opus + Sonnet) · Tally · Notion · Google Docs · Gmail

---

## Overview

This workflow automates the full client onboarding journey — from the moment a lead submits a form to the moment they receive a tailored proposal document and notification email. What would normally take hours of manual research, writing, and document creation is reduced to a fully automated pipeline that runs in minutes.

A prospective client fills out a Tally form. The workflow enriches their data using Claude, researches their industry and competitors in parallel, audits relevant tools for their budget, synthesizes all findings into a structured proposal using Claude Opus and Sonnet, creates records and documents in Notion and Google Docs, and sends a notification email — all without human intervention.

---

## Problem Solved

Sales and consulting teams spend significant time manually researching new leads, writing proposals, and setting up client folders. This pipeline eliminates that toil entirely, allowing teams to focus on closing rather than administrative work. Estimated time saved: **3–5 hours per new client inquiry.**

---

## Workflow Architecture

```
Tally Form Submission
        │
        ▼
Parse Tally Fields
        │
        ▼
Data Enrichment (Claude Haiku)
        │
        ▼
Parse Enrichment → Budget Router
        │
        ├──────────────────────────────────────────┐
        ▼                                          ▼                        ▼
Research Industry (HTTP)          Research Competitors (HTTP)       Tool Audit (Claude Haiku)
  ├── Industry Error?               ├── Competitor Error?             ├── Tool Fallback
  └── Industry Fallback             └── Competitor Fallback           └── Skip Research
        │                                          │                        │
        └──────────────────────────────────────────┘
                                   │
                                   ▼
                               Merge
                                   │
                                   ▼
                          Assemble Context
                                   │
                                   ▼
                      Opus Synthesis (Claude Opus)
                                   │
                                   ▼
                         Parse Opus Output
                                   │
                                   ▼
                   Sonnet Client Proposal (Claude Sonnet)
                                   │
                                   ▼
                           Parse Proposal
                                   │
                                   ▼
                        Build Report Blocks
                                   │
        ┌──────────────────────────┼───────────────────────────┐
        ▼                          ▼                           ▼
Create Client Record       Create Suggestion            Create Proposal Doc
   (Notion)                  Report Doc                 (Google Docs)
        │                          │                           │
        └──────────────────────────┴───────────────────────────┘
                                   │
                                   ▼
                        Create Client Intake Box
                                   │
                                   ▼
                       Send Notification Email (Gmail)
```

---

## Key Features

- **Multi-model AI strategy** — Claude Haiku for fast data enrichment, Claude Opus for deep synthesis, Claude Sonnet for polished proposal writing. Each model is chosen for the right cost/quality tradeoff.
- **Parallel research branches** — Industry research and competitor research run simultaneously to minimize total execution time.
- **Graceful error handling** — Every HTTP research branch has an error catch node and a fallback path, so the workflow never breaks on a failed API call.
- **Budget-aware routing** — A budget router node adjusts the depth of research and tooling recommendations based on the client's indicated budget range.
- **End-to-end document generation** — Proposal documents and suggestion reports are created automatically in Google Docs with structured content.
- **Notion CRM integration** — A client record is created in Notion automatically with all enriched data, ready for the sales team.
- **Automated notification** — Gmail sends a notification email when the proposal is ready.

---

## Tech Stack

| Layer | Tool |
|---|---|
| Trigger | Tally (form submission webhook) |
| AI Enrichment | Claude Haiku (Anthropic) |
| AI Synthesis | Claude Opus (Anthropic) |
| AI Proposal Writing | Claude Sonnet (Anthropic) |
| Web Research | HTTP Request nodes |
| Workflow Orchestration | n8n |
| CRM / Records | Notion |
| Document Generation | Google Docs |
| Notification | Gmail |

---

## How to Set This Up

### Prerequisites
- n8n instance (cloud or self-hosted)
- Anthropic API key (Claude access)
- Tally account with a configured form
- Notion account and integration token
- Google account with Docs and Gmail API access

### Steps

1. **Clone this repo** and locate `workflow.json`
2. **Import into n8n** — go to Workflows → Import from File → select `workflow.json`
3. **Configure credentials** in n8n for:
   - Anthropic (Claude API key)
   - Tally webhook (copy the webhook URL from n8n into your Tally form settings)
   - Notion (integration token + database ID)
   - Google OAuth (Docs + Gmail)
4. **Update the Notion database ID** in the Create Client Record node to point to your database
5. **Test with a sample Tally submission** and verify each node output

> ⚠️ Never commit your `.env` file or API keys to GitHub. Use n8n's built-in credential manager.

---

## Sample Input / Output

**Input (Tally form submission):**
```json
{
  "company": "Bright Salon Co.",
  "industry": "Beauty & Wellness",
  "budget": "$2,000/month",
  "pain_point": "Manual client booking and follow-up",
  "contact_email": "owner@brightsalon.com"
}
```

**Output:**
- ✅ Enriched lead profile with industry context
- ✅ Competitor analysis summary
- ✅ Tool recommendations matched to budget
- ✅ Structured proposal document in Google Docs
- ✅ Client record created in Notion
- ✅ Notification email sent to team

---

## What I Would Add Next

- [ ] Slack notification to sales channel in addition to email
- [ ] Human-in-the-loop approval step before sending proposal to client
- [ ] Webhook to trigger a follow-up sequence after 48 hours if no response
- [ ] Scoring model to prioritize high-value leads automatically

---

## About

Built by Isaac through [Issachar Solutions](https://issacharsolutions.com) as part of an AI automation portfolio.  
Focused on practical, production-grade automation that solves real business problems.

**Connect:** [LinkedIn](https://linkedin.com/in/yourprofile) · [GitHub](https://github.com/yourusername)
