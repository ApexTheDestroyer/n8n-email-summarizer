An automated, self-hosted integration built with **n8n** and **Google Gemini AI**. The pipeline listens for unread Gmail messages, generates concise AI summaries, converts the output into formatted text files saved locally to the Windows host drive, and dispatches summary notifications to a list of recipients.

---

## 🛠️ Pipeline Architecture & Workflow

1. **Gmail Trigger:** Polls the connected inbox for unread email payloads.
2. **Message a Model (Gemini AI):** Passes body text to Google Gemini to extract key takeaways.
3. **Edit Fields (Set):** Maps and formats LLM output into standardized fields.
4. **Convert to File:** Converts textual summary content into a binary file payload (`.txt`).
5. **Read/Write Files from Disk:** Writes binary payload directly to the host storage mount (`/files/`).
6. **Send a Message (Gmail):** Delivers the final summary email to target recipients.

---

## 🐳 Docker Deployment & Environment Setup

### 1. Host Directory Preparation
Ensure local host folder exists on your Windows host machine:
```powershell
mkdir C:\n8n
