# 🛠️ Windows Installation & Setup Guide
## n8n + Gmail + Google Gemini + Docker Desktop + WSL 2

This document contains the complete installation and setup procedure for the **Automated Email Summarizer Pipeline**.

Use this file when setting up the project on a new Windows computer.

> **Recommended GitHub structure**
>
> ```text
> n8n-email-summarizer/
> ├── README.md
> ├── INSTALLATION.md
> └── docker-compose.yml
> ```
>
> `README.md` = project overview  
> `INSTALLATION.md` = complete setup guide  
> `docker-compose.yml` = Docker deployment configuration

---

# 1. 🧩 What This Project Uses

| Component | Purpose |
|---|---|
| Windows | Host operating system |
| WSL 2 | Linux environment used by Docker Desktop |
| Ubuntu | WSL distribution |
| Docker Desktop | Runs the n8n container |
| Docker Compose | Defines the n8n container |
| n8n | Automation platform |
| Gmail | Receives and sends email |
| Google Gemini | Generates email summaries |
| `C:\n8n` | Windows folder for generated `.txt` files |
| `n8n_data` | Persistent Docker volume for n8n data |

---

# 2. ✅ Prerequisites

Before starting, make sure you have:

- Windows 10/11
- Administrator access
- Internet access
- Hardware virtualization enabled
- A Google account
- Gmail access
- Access to Google Gemini / Google AI
- Docker Desktop for Windows

---

# 3. 🐧 Install WSL 2

Open **PowerShell as Administrator**.

Run:

```powershell
wsl --install
```

Restart Windows if prompted.

Then run:

```powershell
wsl --set-default-version 2
```

Update WSL:

```powershell
wsl --update
```

Check WSL:

```powershell
wsl --status
```

Check installed distributions:

```powershell
wsl --list --verbose
```

You should see Ubuntu using version `2`, for example:

```text
NAME      STATE           VERSION
Ubuntu    Running         2
```

If Ubuntu is not installed:

```powershell
wsl --list --online
```

Then:

```powershell
wsl --install -d Ubuntu
```

Check again:

```powershell
wsl --list --verbose
```

---

# 4. 🔄 Restart WSL

To completely stop WSL:

```powershell
wsl --shutdown
```

Start Ubuntu:

```powershell
wsl
```

The first time Ubuntu starts, it may ask you to create a Linux username and password.

When finished:

```bash
exit
```

---

# 5. 🐳 Install Docker Desktop

Install **Docker Desktop for Windows**.

After installation, open Docker Desktop.

Go to:

```text
Settings
→ General
```

Enable:

```text
Use the WSL 2 based engine
```

Then go to:

```text
Settings
→ Resources
→ WSL Integration
```

Enable your Ubuntu distribution.

For example:

```text
✓ Enable integration with my default WSL distro
✓ Ubuntu
```

Click:

```text
Apply & Restart
```

---

# 6. ✅ Verify Docker

Open PowerShell.

Run:

```powershell
docker --version
```

Then:

```powershell
docker compose version
```

Test Docker:

```powershell
docker run hello-world
```

If `hello-world` completes successfully, Docker is working.

---

# 7. 📁 Create the Windows Output Folder

The workflow saves generated text files to Windows.

Create:

```powershell
New-Item -ItemType Directory -Force -Path "C:\n8n"
```

Verify:

```powershell
Test-Path "C:\n8n"
```

Expected:

```text
True
```

List the directory:

```powershell
Get-ChildItem "C:\n8n"
```

---

# 8. 💾 Create the Persistent n8n Docker Volume

Create:

```powershell
docker volume create n8n_data
```

Verify:

```powershell
docker volume ls
```

Inspect:

```powershell
docker volume inspect n8n_data
```

The volume stores persistent n8n data.

---

# 9. 📂 Create the Project Directory

Create the project directory:

```powershell
New-Item -ItemType Directory -Force -Path "$HOME\n8n-email-summarizer"
```

Enter it:

```powershell
Set-Location "$HOME\n8n-email-summarizer"
```

Verify:

```powershell
Get-Location
```

---

# 10. 🐳 Create `docker-compose.yml`

Run this entire PowerShell block:

```powershell
@'
services:
  n8n:
    image: n8nio/n8n:latest
    container_name: n8n
    restart: unless-stopped
    ports:
      - "5678:5678"
    environment:
      N8N_RESTRICT_FILE_ACCESS_TO: /files/
    volumes:
      - n8n_data:/home/node/.n8n
      - 'C:\n8n:/files'

volumes:
  n8n_data:
    external: true
'@ | Set-Content -Encoding UTF8 ".\docker-compose.yml"
```

Check the file:

```powershell
Get-Content ".\docker-compose.yml"
```

---

# 11. 🔍 What the Docker Configuration Does

## Web Port

```yaml
ports:
  - "5678:5678"
```

This makes n8n available at:

```text
http://localhost:5678
```

## Persistent Data

```yaml
- n8n_data:/home/node/.n8n
```

This stores n8n's persistent application data in the Docker volume:

```text
n8n_data
```

## Windows File Storage

```yaml
- 'C:\n8n:/files'
```

This maps:

```text
Windows:
C:\n8n
```

to:

```text
Container:
/files
```

Therefore:

```text
/files/email-summary.txt
```

inside the container becomes:

```text
C:\n8n\email-summary.txt
```

on Windows.

## File Access Restriction

```yaml
N8N_RESTRICT_FILE_ACCESS_TO: /files/
```

The workflow should use file paths under:

```text
/files/
```

---

# 12. ▶️ Start n8n

Make sure you are in:

```text
$HOME\n8n-email-summarizer
```

Then run:

```powershell
docker compose up -d
```

Check:

```powershell
docker ps
```

Also:

```powershell
docker compose ps
```

---

# 13. 🌐 Open n8n

Run:

```powershell
Start-Process "http://localhost:5678"
```

Or manually open:

```text
http://localhost:5678
```

Complete the initial n8n account/setup screen.

---

# 14. 📝 Check n8n Logs

Show logs:

```powershell
docker compose logs n8n
```

Follow logs:

```powershell
docker compose logs -f n8n
```

Press:

```text
CTRL + C
```

to stop following logs.

---

# 15. 🧪 Test the Windows File Mount

Check `/files` from inside the container:

```powershell
docker exec n8n sh -c "ls -la /files"
```

Create a test file:

```powershell
docker exec n8n sh -c "echo 'n8n file mount test' > /files/test.txt"
```

Read the file from Windows:

```powershell
Get-Content "C:\n8n\test.txt"
```

Expected:

```text
n8n file mount test
```

Remove the test file:

```powershell
Remove-Item "C:\n8n\test.txt"
```

---

# 16. 📧 Configure Gmail in n8n

Open:

```text
http://localhost:5678
```

Inside n8n:

```text
Credentials
→ Add Credential
→ Gmail
```

Authenticate your Google account using the supported Gmail authentication method in your n8n version.

This credential will be used by:

```text
Gmail Trigger
```

and:

```text
Send a Message
```

---

# 17. 🤖 Configure Google Gemini

Inside n8n, create the required Google Gemini credential.

The Gemini credential is used by the AI model node to process email content.

Make sure the selected Gemini model is available to your Google account/project.

---

# 18. 🔗 Build the Workflow

The intended workflow is:

```text
Gmail Trigger
      ↓
Message a Model (Gemini)
      ↓
Edit Fields
      ↓
Convert to File
      ↓
Read/Write Files from Disk
      ↓
Send a Message (Gmail)
```

---

# 19. 📥 Configure Gmail Trigger

Create:

```text
Gmail Trigger
```

Select your Gmail credential.

Configure it to monitor the required Gmail mailbox/messages.

The trigger should provide the email body/content needed by Gemini.

---

# 20. 🧠 Configure Gemini

Add:

```text
Message a Model
```

Select your Google Gemini credential.

Example prompt:

```text
Summarize the following email.

Provide:

1. A concise summary
2. Key points
3. Action items
4. Important dates or deadlines

Keep the output clear, concise, and easy to understand.

Email content:

{{ $json.text }}
```

> The exact Gmail output field may differ. Inspect the Gmail Trigger output in n8n and use the actual field containing the email body.

---

# 21. 📝 Configure Edit Fields

Create standardized fields such as:

```text
subject
summary
filename
recipient
```

Example:

```text
filename = email-summary.txt
```

Use the actual Gemini output field from the previous node.

---

# 22. 📄 Configure Convert to File

Add:

```text
Convert to File
```

Configure it to convert the summary text to a `.txt` file.

Example filename:

```text
email-summary.txt
```

The output should be binary data for the file-writing node.

---

# 23. 💾 Configure Read/Write Files from Disk

Add:

```text
Read/Write Files from Disk
```

Configure it to write the binary file.

Use:

```text
/files/email-summary.txt
```

Do **not** use:

```text
C:\n8n\email-summary.txt
```

inside the container.

The mapping is:

```text
Container:
/files/email-summary.txt

Windows:
C:\n8n\email-summary.txt
```

---

# 24. 📤 Configure Send a Message

Add:

```text
Send a Message
```

Use the Gmail credential.

Configure:

```text
To:
recipient@example.com
```

Example subject:

```text
AI Email Summary
```

Example body:

```text
{{ $json.summary }}
```

Use the actual field created by your Edit Fields node.

---

# 25. 🧪 Test the Complete Workflow

Send or receive an email in Gmail.

Leave it unread if your Gmail Trigger is configured for unread messages.

Test the workflow.

Expected flow:

```text
Unread Gmail
     ↓
Gmail Trigger
     ↓
Gemini AI
     ↓
Summary
     ↓
Edit Fields
     ↓
Convert to File
     ↓
Read/Write Files
     ↓
C:\n8n\email-summary.txt
     ↓
Gmail Summary Email
```

---

# 26. 🔍 Verify Generated Files

Check:

```powershell
Get-ChildItem "C:\n8n"
```

Read a summary file:

```powershell
Get-Content "C:\n8n\email-summary.txt"
```

---

# 27. 🔄 Docker Commands You Will Use

## Start

```powershell
Set-Location "$HOME\n8n-email-summarizer"
docker compose up -d
```

## Stop

```powershell
Set-Location "$HOME\n8n-email-summarizer"
docker compose down
```

## Restart

```powershell
Set-Location "$HOME\n8n-email-summarizer"
docker compose restart
```

## Status

```powershell
docker ps
```

## Logs

```powershell
docker compose logs -f n8n
```

---

# 28. ⬆️ Update n8n

Pull the latest image:

```powershell
Set-Location "$HOME\n8n-email-summarizer"
docker compose pull
```

Recreate:

```powershell
docker compose up -d
```

Check:

```powershell
docker ps
```

Check logs:

```powershell
docker compose logs --tail=50 n8n
```

---

# 29. 🛑 Stop and Restart Everything

Stop:

```powershell
docker compose down
```

Start:

```powershell
docker compose up -d
```

The external `n8n_data` volume remains unless you explicitly remove it.

---

# 30. ⚠️ Do Not Delete the n8n Volume

Avoid running:

```powershell
docker volume rm n8n_data
```

unless you intentionally want to remove the persistent n8n data.

---

# 31. 🔐 Security

Never commit credentials or secrets to GitHub.

Do not put the following into `README.md` or `docker-compose.yml`:

```text
Google API keys
OAuth client secrets
Access tokens
Passwords
Private keys
Gmail credentials
Gemini API keys
```

Use n8n Credentials or an appropriate secrets-management solution.

---

# 32. 🛠️ Troubleshooting

## Docker is not running

```powershell
docker version
```

Start Docker Desktop and try again.

---

## WSL is not version 2

Check:

```powershell
wsl --list --verbose
```

Convert Ubuntu:

```powershell
wsl --set-version Ubuntu 2
```

Verify:

```powershell
wsl --list --verbose
```

---

## WSL needs updating

```powershell
wsl --update
wsl --shutdown
```

Then start WSL:

```powershell
wsl
```

---

## n8n container is not running

```powershell
docker ps -a
```

Then:

```powershell
docker compose logs n8n
```

---

## `C:\n8n` does not exist

```powershell
New-Item -ItemType Directory -Force -Path "C:\n8n"
```

Verify:

```powershell
Test-Path "C:\n8n"
```

---

## File mount does not work

Check:

```powershell
docker exec n8n sh -c "ls -la /files"
```

Test:

```powershell
docker exec n8n sh -c "echo 'test' > /files/test.txt"
```

Then:

```powershell
Get-Content "C:\n8n\test.txt"
```

---

## n8n cannot write a file

Use:

```text
/files/
```

inside the n8n container.

Correct:

```text
/files/email-summary.txt
```

Incorrect:

```text
C:\n8n\email-summary.txt
```

---

## Gmail authentication fails

Open the Gmail credential in n8n and re-authenticate.

Then test the Gmail Trigger by itself.

---

## Gemini authentication fails

Check:

- Gemini credential
- Google project/account configuration
- Selected Gemini model
- Model availability
- Input field being passed to the AI node

---

# 33. ✅ Full PowerShell Setup Block

After WSL 2 and Docker Desktop are installed and configured, this block creates the Windows folder, Docker volume, project directory, Compose file, and starts n8n.

```powershell
# ==========================================
# n8n Email Summarizer - Windows Setup
# ==========================================

# Create Windows file storage directory
New-Item -ItemType Directory -Force -Path "C:\n8n"

# Create persistent n8n Docker volume
docker volume create n8n_data

# Create project directory
New-Item -ItemType Directory -Force -Path "$HOME\n8n-email-summarizer"

# Enter project directory
Set-Location "$HOME\n8n-email-summarizer"

# Create docker-compose.yml
@'
services:
  n8n:
    image: n8nio/n8n:latest
    container_name: n8n
    restart: unless-stopped
    ports:
      - "5678:5678"
    environment:
      N8N_RESTRICT_FILE_ACCESS_TO: /files/
    volumes:
      - n8n_data:/home/node/.n8n
      - 'C:\n8n:/files'

volumes:
  n8n_data:
    external: true
'@ | Set-Content -Encoding UTF8 ".\docker-compose.yml"

# Start n8n
docker compose up -d

# Show container status
docker ps

# Show recent logs
docker compose logs --tail=50 n8n

# Open n8n
Start-Process "http://localhost:5678"
```

---

# 34. ✅ Installation Verification Checklist

Run:

```powershell
wsl --status
```

```powershell
wsl --list --verbose
```

```powershell
docker --version
```

```powershell
docker compose version
```

```powershell
docker ps
```

```powershell
docker volume ls
```

```powershell
docker volume inspect n8n_data
```

```powershell
Test-Path "C:\n8n"
```

```powershell
docker exec n8n sh -c "ls -la /files"
```

Finally:

```powershell
Start-Process "http://localhost:5678"
```

---

# 35. 📌 Quick Reference

| Item | Value |
|---|---|
| n8n URL | `http://localhost:5678` |
| Windows output | `C:\n8n` |
| Container output | `/files` |
| n8n data | `/home/node/.n8n` |
| Docker volume | `n8n_data` |
| Container | `n8n` |
| Compose file | `docker-compose.yml` |
| Start | `docker compose up -d` |
| Stop | `docker compose down` |
| Restart | `docker compose restart` |
| Logs | `docker compose logs -f n8n` |

---

# 36. 🎯 Final Result

After setup, the complete system works like this:

```text
Windows
   │
   ├── WSL 2
   │
   └── Docker Desktop
           │
           ▼
        n8n
           │
           ├── Gmail Trigger
           │
           ├── Gemini AI
           │
           ├── Edit Fields
           │
           ├── Convert to File
           │
           ├── Read/Write Files
           │       │
           │       ▼
           │    /files
           │       │
           │       ▼
           │    C:\n8n
           │
           └── Gmail Send Message
```

The automated process is:

```text
Unread Gmail
     ↓
n8n
     ↓
Google Gemini
     ↓
AI Summary
     ↓
TXT File
     ↓
C:\n8n
     ↓
Summary Email
```

---

# 📚 Official Documentation

- WSL:
  https://learn.microsoft.com/en-us/windows/wsl/install

- Docker Desktop for Windows:
  https://docs.docker.com/desktop/setup/install/windows-install/

- Docker Desktop + WSL:
  https://docs.docker.com/desktop/features/wsl/use-wsl/

- n8n:
  https://docs.n8n.io/
