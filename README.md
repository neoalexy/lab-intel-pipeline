# lab-intel-pipeline

A scheduled pipeline that pulls biomedical papers from PubMed and bioRxiv, runs each one through an AI model for a plain-language summary, sends a daily digest by email, and writes the results back into HubSpot — tagging the right lab contact based on the paper's research category.

Built as a personal project to explore what a real automation stack looks like when HubSpot's native workflow engine isn't on the table.

---

## What it does

**Pulls from two sources**
PubMed E-utilities API for peer-reviewed clinical and biomedical research. bioRxiv API for preprints — newer, less filtered, often where the signal shows up first.

**Filters and categorizes**
A keyword filter removes anything outside the biomedical scope. Each paper that passes gets tagged with a lab type — Genomics, Pharma, Biotech, or Diagnostics — based on its category and abstract content.

**Summarizes with AI**
Each paper gets a 2–3 sentence summary in plain prose. Powered by Groq (gpt-oss-20b). The prompt explicitly avoids bullet points and jargon — just what the study found.

**Writes back into HubSpot**
The matching contact in HubSpot gets their lead status updated. A note is created for each paper — title, source, summary — attached to the relevant account. The CRM reflects that something happened in that research domain today without anyone doing it manually.

**Sends a digest**
Everything lands in your inbox as a clean daily email, one paper per entry, grouped by source.

---

## Architecture

```
PubMed API ──────┐
                 ├──► Filter + Lab Type Mapping ──► Groq AI Summary
bioRxiv API ─────┘                                        │
                                              ┌───────────┼───────────┐
                                              ▼           ▼           ▼
                                       HubSpot        Gmail        HubSpot
                                     Contact PATCH    Digest        Notes
```

---

## HubSpot CRM setup

The pipeline writes into a CRM structured to receive it.

**Custom contact properties**
- Lab Type (Genomics / Pharma / Biotech / Diagnostics / CRO/CDMO)
- Instrument Count
- Research Focus
- Paper Source
- AI Summary Status
- Last Digest Sent
- Research Relevance Score

**Lab Onboarding Pipeline**
Lead → Demo Booked → Trial Started → Onboarding → Active Lab

When research activity hits a contact's domain, their lead status moves to IN_PROGRESS automatically — a CRM signal that didn't require anyone to trigger it.

**Segments**
Genomics Labs, Pharma Labs, Biotech Labs — active contact lists that update automatically based on Lab Type.

**Intake form**
Demo requests come in through a published HubSpot form with lab-specific fields. New leads land directly in the pipeline.

**Dashboard**
Papers processed, notes created over time, contacts by lab type, pipeline stage distribution, automation coverage — all live in a single HubSpot dashboard.

---

## Numbers

- 27 papers processed per run (10 PubMed + 17 bioRxiv after filtering)
- 100+ notes written to HubSpot across two runs
- 3 contact records updated per run based on lab type matching
- Full run completes in under 2 minutes

---

## Why n8n instead of native HubSpot workflows

Native HubSpot automation requires a paid plan. More importantly, the logic here — calling external APIs, running an LLM, mapping categories to contacts — isn't something a drag-and-drop workflow builder handles cleanly regardless of plan tier.

n8n handles the orchestration. HubSpot handles the CRM and reporting. Keeping those two concerns separate makes the system easier to debug and extend.

---

## Stack

| Layer | Tool |
|---|---|
| Orchestration | n8n |
| Peer-reviewed literature | PubMed E-utilities API |
| Preprints | bioRxiv API |
| AI summarization | Groq (gpt-oss-20b) |
| Email | Gmail |
| CRM | HubSpot Free + Private App API |

---

## Setup

1. Import `workflow/lab-intel-pipeline.json` into n8n
2. Add credentials: Groq API key, Gmail OAuth2, HubSpot Private App token
3. Create the custom contact properties in HubSpot
4. Map your contact IDs to lab types in the Contact Update node
5. Set the Schedule trigger — default 08:00 daily

---

[github.com/neoalexy](https://github.com/neoalexy)
