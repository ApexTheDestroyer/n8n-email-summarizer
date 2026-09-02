cat << 'EOF' > WORKFLOW_GUIDE.md
# ⚡ n8n Email & MIS Automated Summarizer Guide

This document details the complete architecture, routing rules, and operational workflow for the automated email summarization pipeline in `ApexTheDestroyer/n8n-email-summarizer`.

---

## 1. Pipeline Overview

The workflow automatically monitors unread emails, applies cost-saving keyword pre-checks, summarizes relevant messages via Google Gemini AI, routes emergency alerts to team members, and archives processed items.

## [Gmail Trigger]
│
▼
## [Sender & Subject Filter] ──(Filtered Out)──► [Log Ignored] ──► [Mark Read]
│
▼
## [Emergency Keyword Pre-Check]
├── (Contains Emergency Keyword) ──► Bypass Routine Filter ──┐
└── (Standard Relevant Email) ──────────────────────────────┤
│
▼
## [Gemini AI Summarizer]
│
▼
## [Data & Subject Cleanup]
│
▼
## [Priority Check Branch]
├── (HIGH) ──► [Urgent Email Alert] ──┐
└── (NORM) ──► [Standard Summary]  ──┤
│
▼
## [Mark Read]


---

## 2. Node Specifications & Logic

### Node 01. Gmail Trigger (`Gmail Trigger`)
* **Poll Interval:** Every minute
* **Criteria:** `readStatus: "unread"`

### Node 02. Filter: Relevant Senders & Subjects (`IF Node`)
Filters incoming messages to ensure only authorized or relevant streams are processed.
* **Approved Senders:**
  * `carlocc2518@gmail.com`
  * `resellerlegit18@gmail.com`
  * `@inari-amertron.com.ph`
  * `midmannandez@gmail.com`
  * `MIS Helpdesk`
* **Approved Subjects:** `MIS`, `REPORT`, `RIL COMPARISON`, `IPD Test`, `Support Ticketing`

### Node 03. Emergency Pre-Check (`IF Node`)
Inspects email content (`text` or `snippet`) for critical keywords to bypass routine archiving:
* **Emergency Keywords:** `urgent`, `outage`, `down`, `critical`, `breach`, `failure`
* **Behavior:** Emails containing any keyword skip routine filtering and force immediate Gemini evaluation.

### Node 04. AI: Gemini Summarizer (`Google Gemini Node`)
* **Model:** `models/gemini-1.5-flash`
* **Tokens Max:** 150
* **Output Format:**
  * `PRIORITY: [HIGH_PRIORITY or NORMAL]`
  * `KEY ISSUE: [1 line summary]`
  * `ACTION: [1 line action required, or None]`

### Node 05–07. Escalation & Distribution
* **High Priority Recipients:** `midmannandez@gmail.com`, `carl.cortez@inari-amertron.com.ph`, `carlocc2518@gmail.com`, `carlcortez957@gmail.com`
* **Subject Prefix:** `[URGENT] <Clean Subject>`
* **Standard Recipients:** `midmannandez@gmail.com`, `carlocc2518@gmail.com`, `resellerlegit18@gmail.com`
* **Subject Prefix:** `[SUMMARY] <Clean Subject>`

### Node 09–10. Error Handling
* **Trigger:** `Error Trigger Node`
* **Action:** Sends an immediate failure report to `carlocc2518@gmail.com` with node execution details and error messages.

---

## 3. Maintenance & Testing Commands

* **Export Active Workflow JSON:**
  ```bash
  docker exec -it n8n n8n export:workflow --all --output=/files/workflows.json
