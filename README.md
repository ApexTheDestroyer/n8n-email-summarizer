cat << 'EOF' > README.md
# 📧 Automated Email Summarizer & MIS Alert Pipeline

An automated, self-hosted email filtering, summarization, and emergency escalation pipeline built with **n8n**, **Google Gemini AI**, **PostgreSQL**, and **Docker Desktop**.

The workflow monitors incoming Gmail messages, filters routine operational reports to prevent token waste, checks for critical outage keywords, summarizes actionable emails via Gemini 1.5 Flash, and routes priority alerts to key team members.

---

## 🏗️ System Architecture

```text
               ┌────────────────────────────────────────────────────────┐
               │                01. Gmail Trigger (Unread)              │
               └───────────────────────────┬────────────────────────────┘
                                           │
                                           ▼
                       ┌──────────────────────────────────────┐
                       │       02. Emergency Pre-Check        │
                       │   (Checks subject AND body text)     │
                       └───────────────────┬──────────────────┘
                                           │
                   ┌───────────────────────┴───────────────────────┐
                   │ (TRUE: Emergency Match)                       │ (FALSE: Non-Emergency)
                   ▼                                               ▼
                   │                               ┌───────────────────────────────┐
                   │                               │   03. Standard Stream Filter  │
                   │                               └───────────────┬───────────────┘
                   │                                               │
                   │                       ┌───────────────────────┴───────────────────────┐
                   │                       │ (TRUE: Approved Stream)                       │ (FALSE: Ignored Stream)
                   ▼                       ▼                                               ▼
     ┌──────────────────────────────────────────┐                             ┌───────────────────────────┐
     │         04. Gemini AI Processing         │                             │  08. Mark Email as Read   │
     │         (Summarization & Action)         │                             │       (Filtered Out)      │
     └─────────────────────┬────────────────────┘                             └───────────────────────────┘
                           │
                           ▼
     ┌──────────────────────────────────────────┐
     │      05. Data Formatting & Priority      │
     └─────────────────────┬────────────────────┘
                           │
                           ▼
     ┌──────────────────────────────────────────┐
     │          06. Escalation Router           │
     └─────────────────────┬────────────────────┘
                           │
         ┌─────────────────┴─────────────────┐
         │ (HIGH_PRIORITY)                   │ (NORMAL)
         ▼                                   ▼
┌───────────────────────────┐       ┌───────────────────────────┐
│ 07a. High Priority        │       │ 07b. Normal Priority      │
│      Emergency Alert      │       │      Summary Email        │
└───────────────────────────┘       └───────────────────────────┘
```
##⚡ Key Features
### Zero-Token Routine Filtering: Automatically logs routine mail (HR ANNOUNCEMENT, Unclosed WOs Report) as FILTERED_OUT without sending payload to Gemini API, conserving token quota.

### Emergency Keyword Override: Intercepts critical body keywords (urgent, outage, down, critical, breach, failure) and forces instant Gemini priority analysis regardless of sender routines.

### AI-Powered Summarization: Uses gemini-1.5-flash to generate concise, 1-line key issue descriptions and action requirements.

### Targeted Escalations: Directs high-priority alerts to dedicated emergency team lists while sending standard summaries to routine stakeholders.

### PostgreSQL Engine: Runs on a dedicated PostgreSQL 16 database backend (n8n_postgres) for execution history and operational persistence.

## 📁 Repository Structure & Documentation
### Plaintext
### n8n-email-summarizer/
### │
### ├── README.md               # Main project overview & architecture
### ├── WORKFLOW_GUIDE.md       # Detailed node breakdown & pipeline logic
### ├── POSTGRES_SETUP.md       # PostgreSQL configuration & database commands
### ├── CLOUDFLARE_TUNNEL.md    # Remote webhook setup via Cloudflare Tunnel
### ├── N8N_ARCHITECTURE.md    # Core architecture & design principles
### ├── INSTALLATION.md         # Full Windows, WSL 2, and Docker installation guide
### └── docker-compose.yml      # Docker compose definition (n8n + postgres)
### 📖 Complete Documentation Links
### ⚡ Workflow Architecture Guide: Node-by-node logic, filters, and prompt specifications.

### 🐘 PostgreSQL Setup Guide: Database credentials, schema specs, and backup/restore commands.

### 🌐 Cloudflare Tunnel Guide: External webhook exposure and security setup.

### 🏗️ System Architecture: High-level infrastructure breakdown.

### 🛠️ Complete Installation Guide: WSL 2, Docker Desktop, and environment variables.

### 🐳 Quick Start (Docker Compose)
### The environment runs two coupled services defined in docker-compose.yml:

### n8n: Automation server accessible at http://localhost:5678

### PostgreSQL: System database running postgres:16-alpine

Start Services
Bash
docker compose up -d
Stop Services
Bash
docker compose down
View Real-Time Logs
Bash
docker compose logs -f n8n postgres
## 🔐 Security
### Never commit raw API keys, OAuth client secrets, or database passwords to public repositories. All sensitive configurations should be managed within n8n Credentials or passed via external .env environment files.
### EOF

### git add README.md && git commit -m "docs: update README.md with complete architecture, feature set, and documentation links" && git push origin main
