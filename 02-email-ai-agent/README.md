<div align="center">

# 📧 Project 02 — Email AI Agent

[![n8n Cloud](https://img.shields.io/badge/Platform-n8n%20Cloud-EA4B71?style=for-the-badge&logo=n8n&logoColor=white)](https://app.n8n.cloud)
[![Gemini Flash](https://img.shields.io/badge/LLM-Gemini%20Flash-4285F4?style=for-the-badge&logo=google&logoColor=white)](https://deepmind.google/technologies/gemini/)
[![Gmail](https://img.shields.io/badge/Gmail-API-EA4335?style=for-the-badge&logo=gmail&logoColor=white)](https://developers.google.com/gmail/api)
[![Google Calendar](https://img.shields.io/badge/Google%20Calendar-API-4285F4?style=for-the-badge&logo=googlecalendar&logoColor=white)](https://developers.google.com/calendar)
[![Google Sheets](https://img.shields.io/badge/Google%20Sheets-Contacts%20DB-34A853?style=for-the-badge&logo=googlesheets&logoColor=white)](https://sheets.google.com)

</div>

---

## 📌 What I Built

A **chat-based workspace assistant** that can send emails and create calendar events — but with an important constraint built in: it must always look up a person's email address from a Google Sheets contacts database before taking any action. It can never guess, assume, or hallucinate an email address.

This was my first time building an AI agent with real **safety rails** — designing the system prompt so the agent behaves reliably and refuses to do the wrong thing, not just the right thing.

The agent also has **conversational memory** — it remembers what was said earlier in the chat session, so you can say "Send that to John" after previously mentioning John, and it knows who you mean.

---

## 💡 What I Learned Building This

**Safety constraints in system prompts:** The most important thing I learned here is that the system prompt is where you control the agent's behaviour and enforce rules. I wrote explicit instructions:
- *"You must always look in the contacts database before doing something like creating an event or sending an email."*
- *"Never make up someone's email address. You must look in the contacts database tool."*

Without these rules, the LLM might guess `john@company.com` and send an email to the wrong person. The constraint forces a tool lookup first.

**Tool-use ordering:** This is a chain-of-thought pattern enforced through the system prompt. The agent has to: (1) query contacts → (2) get the verified email → (3) then proceed with Gmail or Calendar. You can't skip step 1.

**Memory Buffer Window:** n8n's `memoryBufferWindow` node gives the agent short-term memory for the current conversation. Without it, every message is treated as a fresh start and the agent has no context about what was just discussed.

**Google Sheets as a live data source:** Rather than hardcoding contact data, I connected a live Google Sheet as a tool. The agent can query it at runtime. This means the contacts database can be updated by anyone without changing the workflow.

---

## 🗺️ How It Works — Node by Node

```
User types a message in the Chat interface
        │
        ▼
Chat Trigger (n8n built-in chat UI)
        │
        ▼
AI Agent (Gemini Flash via OpenRouter)
        │
        │  The agent has 4 things connected to it:
        │
        ├── 🧠 Simple Memory (Buffer Window)
        │       Stores recent conversation turns so the agent
        │       remembers context within a session
        │
        ├── 📋 Contacts Database Tool (Google Sheets)
        │       The agent MUST call this first to get a
        │       verified email address for any person
        │
        ├── 📧 Send Email Tool (Gmail)
        │       Sends a plain text email once the agent has
        │       a verified address from the contacts sheet
        │
        └── 📅 Create Event Tool (Google Calendar)
                Creates a calendar event with an attendee,
                using the verified email from the contacts sheet
```

---

## 🔐 The Safety Rail — How It Works

The key to this workflow is the system prompt. Here is the exact safety instruction I wrote:

```
You must always look in the contacts database before doing something
like creating an event or sending an email. You need the person's
email address in order to do one of those actions.

Never make up someone's email address. You must look in the
contacts database tool.
```

**Why this matters:** LLMs are trained on enormous amounts of text and have seen millions of email address patterns. Without this constraint, a model might confidently output `uday@techcompany.com` — which could be a real address belonging to someone else. The safety rail prevents this by making the Contacts Database lookup a mandatory first step.

**What happens if the contact isn't found:** The agent will tell the user it couldn't find the person in the contacts database and ask for the correct email address. It won't proceed.

---

## 🔧 Nodes Used

| Node | Purpose |
|---|---|
| `Chat Trigger` | Receives messages from the chat UI |
| `AI Agent` | Core reasoning — Gemini Flash via OpenRouter |
| `Simple Memory` (Buffer Window) | Maintains conversation context within the session |
| `Contacts Database` (Google Sheets Tool) | Live contacts lookup — must be queried first |
| `Send Email` (Gmail Tool) | Sends plain text emails via Gmail OAuth2 |
| `Create Event` (Google Calendar Tool) | Creates calendar events with attendees |

---

## 🏗️ Architecture Diagram

```
  ┌─────────────────────────────────────────────────────┐
  │                  EMAIL AI AGENT                      │
  │                                                      │
  │   Chat Input                                         │
  │       │                                              │
  │       ▼                                              │
  │   ┌───────────────────────────────────────────┐     │
  │   │         AI Agent (Gemini Flash)            │     │
  │   │                                            │     │
  │   │  Memory ◄──  Buffer Window                 │     │
  │   │  (remembers conversation context)          │     │
  │   │                                            │     │
  │   │  STEP 1: Always call Contacts Database    │     │
  │   │          ──────────────────────────────   │     │
  │   │          Google Sheets → verified email   │     │
  │   │                                            │     │
  │   │  STEP 2: Use verified email for action    │     │
  │   │          ──────────────────────────────   │     │
  │   │          Gmail (send email)               │     │
  │   │          OR                               │     │
  │   │          Google Calendar (create event)   │     │
  │   └───────────────────────────────────────────┘     │
  │                                                      │
  │   Response returned to chat UI                       │
  └─────────────────────────────────────────────────────┘
```

---

## ⚙️ Setup Instructions

### Step 1: Set Up the Contacts Google Sheet

1. Create a new Google Sheet
2. Name Sheet1's columns: `name`, `email`, `phone` (or similar)
3. Add a few test contacts with real or dummy email addresses
4. Copy the Spreadsheet ID from the URL: `docs.google.com/spreadsheets/d/YOUR_ID/edit`
5. Update the `Contacts Database` node in n8n with this Spreadsheet ID

### Step 2: Add Credentials in n8n Cloud

| Credential | Type | Where to Get |
|---|---|---|
| Google Sheets | OAuth2 | [Google Cloud Console](https://console.cloud.google.com) — enable Sheets API |
| Gmail | OAuth2 | Same Google Cloud project — enable Gmail API |
| Google Calendar | OAuth2 | Same Google Cloud project — enable Calendar API |
| OpenRouter | API Key | [openrouter.ai/keys](https://openrouter.ai/keys) |

### Step 3: Import & Activate

1. Import `EMAIL_AI_AGENT.json` into n8n Cloud
2. Connect all credentials
3. Update the Google Sheet ID in the Contacts Database node
4. Update the calendar email in the Create Event node to your Gmail address
5. Activate and open the Chat interface

---

## 🧪 Testing It

**Test 1 — Contact lookup + email:**
```
You: Send an email to John saying the meeting is rescheduled to Thursday
Agent: [looks up John in Contacts Database] → [sends email to john@example.com]
```

**Test 2 — Safety rail in action:**
```
You: Send an email to sarah@fakeemail.com saying hello
Agent: [bypasses the contacts check since email was explicitly given]
       NOTE: If you want to test the safety rail, say "Send an email to Sarah"
             without providing an address — the agent must look it up.
```

**Test 3 — Calendar with memory:**
```
You: Schedule a meeting with John tomorrow at 2pm about the project
Agent: [looks up John in contacts] → [creates calendar event with attendee]
You: Make it 3pm instead
Agent: [remembers the context] → [updates accordingly]
```

---

## 📸 Screenshots

> Add your n8n canvas screenshot to `screenshots/` and update this section.

| View | File |
|---|---|
| Workflow canvas | `screenshots/workflow-canvas.png` |


---

## 🔮 What I Would Add Next

- **Read emails** — add a Gmail `getAll` tool so the agent can also fetch and summarise inbox messages
- **Draft mode** — before sending, create a draft and show it to the user for approval
- **Persistent memory** — replace Buffer Window with vector-based memory so it remembers contacts and preferences across sessions
- **Multiple calendars** — support scheduling across different calendar accounts

---

[← Back to Portfolio](../README.md)

