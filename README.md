# Strategic AI in Finance: Building Smarter Assistants with Gemini

> Turn your Telegram bot into a smart personal finance assistant — no coding background needed.

---

## What You'll Build

Send a receipt photo to your Telegram bot → **Gemini Vision** reads it → data is logged to **Google Sheets** automatically. Ask questions about your spending and get a live **dashboard chart** back in Telegram.

```
📱 Send receipt photo
        ↓
🤖 Telegram Bot (BotFather)
        ↓
⚡ n8n Workflow
        ↓
🔍 Gemini Vision API → extracts JSON
        ↓
📊 Google Sheets → logs the row
        ↓
💬 Confirmation sent back to you
```

---

## What's in This Repo

| File | Description |
|------|-------------|
| `README.md` | This guide — setup steps, architecture, tips |
| `Personal_Finance_with_Dashboard.json` | n8n workflow — import this directly into n8n |

---

## Resources

| Resource | Link |
|----------|------|
| Google Sheet Template | [Make a copy here](https://docs.google.com/spreadsheets/d/1nTD4c1_DAuDJAC9oDvgdNkp3-Fl5hT6t25DkiR3-LsE/edit?usp=sharing) |
| Google Drive Receipts | [Make a copy here](https://drive.google.com/drive/folders/1Ide1xEytIBX5HoMl5I8laR4_rjseRXoY?usp=sharing) |
| Workshop Slides (Canva) | [View slides](https://www.canva.com/d/uxl2sODyhvlNKaJ) |
| n8n Cloud (free tier) | [n8n.io](https://n8n.io) |
| Google AI Studio | [aistudio.google.com](https://aistudio.google.com) |

---

## 🛠️ Setup Guide

### Step 1 — Get Your Telegram Bot Token

1. Open Telegram → search **@BotFather**
2. Tap **Start** or type `/start`
3. Type `/newbot`
4. Enter a display name e.g. `My Finance Tracker`
5. Enter a username ending in `bot` e.g. `YourNameFinanceTracker_bot`
6. Copy the token BotFather gives you → looks like `123456789:ABCdef...`

> This token is your bot's password — never share it publicly or commit it to GitHub.

---

### Step 2 — Get Your Free Gemini API Key

1. Go to [aistudio.google.com](https://aistudio.google.com) → sign in with Google
2. Click **Get API Key** (top left sidebar) → **Create API key**
3. Copy and store it safely

---

### Step 3 — Set Up Google Sheets

1. Open the [Google Sheet Template](https://docs.google.com/spreadsheets/d/1nTD4c1_DAuDJAC9oDvgdNkp3-Fl5hT6t25DkiR3-LsE/edit?usp=sharing)
2. Click **File → Make a copy** → save to your Google Drive
3. Copy your Sheet ID from the URL:
   ```
   https://docs.google.com/spreadsheets/d/YOUR_SHEET_ID_HERE/edit
   ```

Your sheet should have these columns (already set up in the template):

| Date | Amount in RM | Description | Category | Google Drive Image |
|------|-------------|-------------|----------|--------------------|

---

### Step 4 — Import the n8n Workflow

1. Sign up free at [n8n.io](https://n8n.io) (cloud) or self-host
2. Create a new workflow → click the **⋮ menu** → **Import from file**
3. Upload `Personal_Finance_with_Dashboard.json` from this repo
4. You'll see the full workflow with 3 branches:
   - **Receipt branch** — image/PDF → Gemini OCR → Sheets
   - **Text query branch** — AI Agent answers spending questions
   - **Dashboard branch** — generates donut chart → sends photo

---

### Step 5 — Connect Your Credentials in n8n

In n8n, open **Credentials** and add the following:

#### Telegram
- Node: `Telegram Trigger`
- Add credential → **Telegram API**
- Paste your Bot Token from BotFather
- Set webhook mode: `Webhook`

#### Google Gemini
- Open any Gemini node in the workflow
- Add credential → **Google Gemini API**
- Paste your API key from AI Studio

#### Google Sheets
- Open the `Append row in sheet` node
- Add credential → **Google Sheets OAuth2**
- Sign in with your Google account
- Update the **Spreadsheet ID** with your copied sheet's ID

#### Google Drive
- Open the `Upload file` node
- Add credential → **Google Drive OAuth2**
- Sign in with your Google account
- Update the **Folder ID** to your preferred Drive folder

---

### Step 6 — Activate & Test

1. Click the **Activate** toggle (top right in n8n)
2. Send your bot a test message in Telegram
3. Send a receipt photo as a **File** (tap paperclip → File → select image)
4. Watch the row appear in your Google Sheet 
5. Type `create dashboard for me` → get a donut chart back 

---

## 🧠 How It Works (Architecture)

### Receipt Flow
```
Telegram Trigger → Switch (image detected)
  → Get file from Telegram
  → Upload to Google Drive
  → Analyze with Gemini Vision
  → Extract Text for AI
  → AI Agent (parse JSON)
  → Code node (strip markdown fences)
  → Append row to Google Sheets
  → Send confirmation to Telegram
```

### Text Query Flow
```
Telegram Trigger → Switch (text detected)
  → AI Agent (Gemini + memory + Sheets tool + calculator)
  → Send text reply to Telegram
```

### Dashboard Flow
```
Telegram Trigger → Switch ("dashboard" detected)
  → Get all rows from Google Sheets
  → Code node (aggregate by category)
  → HTTP Request to QuickChart.io (generate donut chart)
  → Send photo to Telegram with caption breakdown
```

---

## What You Can Ask the Bot

| Message | What happens |
|---------|-------------|
| Send receipt photo as file | Logs to Sheets automatically |
| `how much did I spend on food?` | AI reads your sheet and replies |
| `what's my biggest expense?` | Breakdown with percentages |
| `give me 3 ways to cut my spending` | Personalised AI tips |
| `create dashboard for me` | Donut chart + category breakdown sent as photo |

---

## Customisation Tips

- **Add categories** — edit the AI Agent system prompt to add new category types
- **Change currency** — update `MYR` references in the Code node and AI Agent prompt
- **Adjust dashboard** — modify the Code node to add more chart types or metrics
- **Memory window** — increase the Simple Memory node's window size for longer conversation context

---

## Extension Ideas

| Idea | How |
|------|-----|
| Budget alerts | Add an IF node — if monthly total > threshold, send warning |
| Voice receipts | Add a Telegram voice message handler → Whisper API transcription |
| Multi-user tracker | Add a user ID column to Sheets, filter by `chat.id` |
| WhatsApp version | Swap Telegram nodes for WhatsApp via WATI or 360dialog |

---

## Common Issues & Fixes

| Error | Cause | Fix |
|-------|-------|-----|
| Bot doesn't respond | Webhook not registered | Activate the workflow in n8n |
| Row not appending to sheet | Wrong Spreadsheet ID | Double-check ID copied from sheet URL |
| `Amount in Naira` column missing | Old workflow version | Use the JSON from this repo |
| Telegram parse error 400 | Markdown formatting issue | Set Parse Mode to `None` in Send Message node |
| Dashboard shows 2 replies | Switch wired to both AI Agent and dashboard branch | Delete wire from Switch output 3 to AI Agent |

---
## About

Built by **Jacki Choo**
