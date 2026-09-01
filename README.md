# 📧 Automated Email Summarizer Pipeline — n8n + Google Gemini

An automated, self-hosted email summarization pipeline built with **n8n**, **Gmail**, **Google Gemini AI**, **Docker Desktop**, and **WSL 2**.

The workflow monitors Gmail for unread emails, sends the email body to Gemini, generates a concise summary, converts the result into a `.txt` file, saves it to the Windows host, and sends a summary notification.

---

## 🏗️ Pipeline Architecture

```text
Gmail
  │
  ▼
Gmail Trigger
  │
  ▼
Google Gemini AI
  │
  ▼
Edit Fields
  │
  ▼
Convert to File
  │
  ▼
Read/Write Files
  │
  ▼
C:\n8n
  │
  └──────────────► Send Summary Email
```

---

## ⚡ Quick Overview

The project runs locally using:

```text
Windows
   ↓
WSL 2
   ↓
Docker Desktop
   ↓
n8n
   ├── Gmail
   ├── Google Gemini
   └── C:\n8n file storage
```

Generated summary files are written to:

```text
C:\n8n
```

n8n is available at:

```text
http://localhost:5678
```

---

## 📖 Installation

The complete Windows setup is documented separately so the main README stays easy to read.

👉 **[Open the complete INSTALLATION.md setup guide](./INSTALLATION.md)**

The installation guide covers:

- WSL 2 installation
- Ubuntu setup
- Docker Desktop installation
- Docker Desktop WSL integration
- Docker verification
- Windows `C:\n8n` storage setup
- Persistent `n8n_data` volume
- `docker-compose.yml` creation
- n8n startup
- File-mount testing
- Gmail configuration
- Google Gemini configuration
- Workflow configuration
- End-to-end testing
- Troubleshooting
- Docker maintenance commands

---

## 🐳 Docker

The project uses:

```text
n8nio/n8n
```

The Compose configuration provides:

| Component | Configuration |
|---|---|
| Web UI | `localhost:5678` |
| Container | `n8n` |
| Persistent data | `n8n_data` |
| Container files | `/files` |
| Windows files | `C:\n8n` |

Start the project:

```powershell
docker compose up -d
```

Stop the project:

```powershell
docker compose down
```

View logs:

```powershell
docker compose logs -f n8n
```

---

## 📁 Repository Structure

```text
n8n-email-summarizer/
│
├── README.md
├── INSTALLATION.md
└── docker-compose.yml
```

### `README.md`

Project overview and architecture.

### `INSTALLATION.md`

Complete installation and configuration instructions.

### `docker-compose.yml`

Docker configuration used to run n8n.

---

## 🔐 Security

Never commit API keys, OAuth secrets, passwords, access tokens, or other credentials to GitHub.

Use n8n Credentials or an appropriate secret-management mechanism.

---

## 🎯 Result

Once configured:

```text
Unread Gmail
     ↓
Gmail Trigger
     ↓
Gemini AI
     ↓
Summary
     ↓
TXT File
     ↓
C:\n8n
     ↓
Summary Email
```

---

## 📚 Documentation

- [Complete Installation Guide](./INSTALLATION.md)
- [Docker Compose](./docker-compose.yml)
- [WSL Documentation](https://learn.microsoft.com/en-us/windows/wsl/install)
- [Docker Desktop for Windows](https://docs.docker.com/desktop/setup/install/windows-install/)
- [n8n Documentation](https://docs.n8n.io/)
