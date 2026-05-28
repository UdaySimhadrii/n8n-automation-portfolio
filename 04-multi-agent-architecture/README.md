<div align="center">

# 🤖 Project 04 — Multi-Agent Architecture

[![n8n Cloud](https://img.shields.io/badge/Platform-n8n%20Cloud-EA4B71?style=for-the-badge&logo=n8n&logoColor=white)](https://app.n8n.cloud)
[![GPT-4o-mini](https://img.shields.io/badge/Orchestrator-GPT--4o--mini-412991?style=for-the-badge&logo=openai&logoColor=white)](https://openai.com)
[![Telegram](https://img.shields.io/badge/Interface-Telegram%20Bot-26A5E4?style=for-the-badge&logo=telegram&logoColor=white)](https://core.telegram.org/bots)
[![OpenAI Whisper](https://img.shields.io/badge/Voice-OpenAI%20Whisper-412991?style=for-the-badge&logo=openai&logoColor=white)](https://openai.com/research/whisper)
[![Multi-Agent](https://img.shields.io/badge/Pattern-Multi--Agent%20Orchestration-FF6B35?style=for-the-badge)]()

</div>

---

## 📌 What I Built

This is the most complex project in the portfolio — a **Telegram-based personal assistant** that uses a multi-agent architecture. Instead of one large AI agent trying to do everything, there is a central **orchestrator** and five **specialist sub-agents**, each in their own separate n8n workflow.

The orchestrator receives your message on Telegram (text or voice), figures out what you're asking for, and delegates to the right sub-agent. It never does the actual work itself — it's purely a router and coordinator.

**Sub-agents it can delegate to:**
- 📧 **Email Agent** — send, reply, draft, fetch, label emails via Gmail
- 📅 **Calendar Agent** — create, update, delete, retrieve Google Calendar events
- 👥 **Contact Agent** — look up, add, or update contacts in Airtable
- ✍️ **Content Creator Agent** — research and write SEO blog posts using Tavily web search
- 🌐 **Web Search** — direct Tavily search for quick internet lookups
- 🔢 **Calculator** — built-in maths tool for calculations
- 🤔 **Think Tool** — lets the orchestrator reason step-by-step before deciding

---

## 💡 What I Learned Building This

**Why use multiple agents instead of one?** This was the biggest architectural lesson. A single agent with 20 tools becomes unreliable — the LLM gets confused about which tool to use when, and errors in one tool can cascade. The orchestrator pattern keeps each agent small and focused. Each sub-agent only knows about its own domain and tools.

**The `Execute Workflow` node as a tool:** This was the key n8n concept that makes the whole thing work. In n8n, you can wrap an entire sub-workflow as a tool that the orchestrator's AI agent can call. From the orchestrator's perspective, `emailAgent`, `calendarAgent`, and `contentCreator` are just tool calls — it has no idea they're entire workflows with their own LLMs, nodes, and API connections.

**Voice message handling:** Telegram sends voice messages as audio files, not text. The workflow handles this with a Switch node that detects the message type. Voice messages go through: download audio file → OpenAI Whisper transcription → same text pipeline as typed messages. This was the first time I worked with binary audio data in n8n.

**The Think tool:** This is a simple but powerful addition. It gives the orchestrator a "scratchpad" — a tool it can call to reason through a complex request before deciding which sub-agent to use. Without it, the LLM might jump straight to a tool call on ambiguous requests.

**Memory at the orchestrator level:** The `Simple Memory` (Buffer Window) is attached to the orchestrator only. Sub-agents are stateless — they receive a single query, do their job, and return a result. Conversation context is managed centrally, not scattered across sub-agents.

**Model selection per agent:** Different agents use different LLMs based on what they need:

| Agent | Model | Why |
|---|---|---|
| Orchestrator | GPT-4o-mini | Reliable tool-use routing |
| Email Agent | GPT-4o-mini | HTML formatting, precise instructions |
| Calendar Agent | Gemini 2.5 Flash | Fast, good at date/time reasoning |
| Contact Agent | Gemini 2.0 Flash | Simple lookups, low latency |
| Content Creator | Gemini 2.0 Flash | Creative writing quality |

---

## 🗺️ How It Works — Full Flow

### Text Message Flow
```
User sends text message on Telegram
        │
        ▼
Telegram Trigger
        │
        ▼
Switch node → detects message type = "text"
        │
        ▼
Set 'Text' node (extracts the message text)
        │
        ▼
Ultimate Assistant Agent (GPT-4o-mini)
        │  ◄── Simple Memory (Buffer Window)
        │  ◄── Think Tool
        │  ◄── Calculator Tool
        │  ◄── Tavily Web Search Tool
        │
        ├── emailAgent Tool → EMAIL AGENT workflow
        ├── calendarAgent Tool → CALENDAR AGENT workflow
        ├── contactAgent Tool → CONTACT AGENT workflow
        └── contentCreator Tool → CONTENT CREATOR AGENT workflow
        │
        ▼
Response node → sends answer back to user on Telegram
```

### Voice Message Flow
```
User sends voice note on Telegram
        │
        ▼
Telegram Trigger
        │
        ▼
Switch node → detects message type = "voice"
        │
        ▼
Download Voice File (Telegram node — fetches the audio binary)
        │
        ▼
Transcribe Audio (OpenAI Whisper — audio → text)
        │
        ▼
[Joins the same pipeline as text messages above]
        │
        ▼
Ultimate Assistant Agent → sub-agents → Telegram Response
```

---

## 🔧 Nodes Used — Orchestrator Workflow

| Node | Purpose |
|---|---|
| `Telegram Trigger` | Receives all incoming Telegram messages |
| `Switch` | Routes to text path or voice path |
| `Set 'Text'` | Extracts the plain text from a text message |
| `Download Voice File` | Downloads audio binary from Telegram servers |
| `Transcribe Audio` (OpenAI) | Converts voice note to text via Whisper |
| `Ultimate Assistant` (AI Agent) | Orchestrator — GPT-4o-mini |
| `Simple Memory` | Buffer Window for conversation context |
| `Think Tool` | Chain-of-thought reasoning before tool selection |
| `Calculator Tool` | Built-in maths calculations |
| `Tavily` (HTTP Tool) | Direct web search |
| `Email Agent` (Workflow Tool) | Calls EMAIL AGENT sub-workflow |
| `Calendar Agent` (Workflow Tool) | Calls CALENDAR AGENT sub-workflow |
| `Contact Agent` (Workflow Tool) | Calls CONTACT AGENT sub-workflow |
| `contentCreator` (Workflow Tool) | Calls CONTENT CREATOR AGENT sub-workflow |
| `Response` (Telegram) | Sends the final answer back to the user |

---

## 🏗️ Full Architecture Diagram

```
  TELEGRAM (text or voice message)
           │
           ▼
  ┌────────────────┐
  │ Telegram Trigger│
  └───────┬────────┘
          │
          ▼
  ┌───────────────┐
  │  Switch Node  │
  └───┬───────────┘
      │
      ├── TEXT ──────────────────────────────────┐
      │                                          │
      └── VOICE ──► Download Audio               │
                         │                       │
                         ▼                       │
                   OpenAI Whisper                │
                   (transcribe)                  │
                         │                       │
                         └───────────────────────┤
                                                 │
                                                 ▼
                                  ┌──────────────────────────┐
                                  │  GPT-4o-mini             │
                                  │  "Ultimate Assistant"    │
                                  │                          │
                                  │  ◄── Simple Memory       │
                                  │  ◄── Think Tool          │
                                  │  ◄── Calculator          │
                                  │  ◄── Tavily Search       │
                                  └──────────┬───────────────┘
                                             │
                    ┌────────────────────────┼──────────────────────┐
                    ▼            ▼           ▼           ▼          ▼
              EMAIL AGENT  CALENDAR    CONTACT      CONTENT    (direct
              (GPT-4o-mini) AGENT      AGENT        CREATOR     tools:
                           (Gemini    (Gemini       (Gemini    Tavily,
                           2.5Flash)  2.0Flash)     2.0Flash)  Calc)
                    │            │           │           │
                  Gmail     G.Calendar    Airtable    Tavily
                                                    Web Search
                                             │
                                             ▼
                                  Telegram Response
                                  (sent back to user)
```

---

## 📁 Sub-Agent Reference

Each sub-agent is a separate n8n workflow, included in the `/sub-agents/` folder.

| File | Workflow Name | LLM | Tools |
|---|---|---|---|
| `EMAIL_AGENT.json` | EMAIL AGENT | GPT-4o-mini | Send, Reply, Draft, Fetch, Label, Mark Unread |
| `CALENDAR_AGENT.json` | CALENDAR AGENT | Gemini 2.5 Flash | Create, Create with Attendee, Get, Update, Delete |
| `CONTACT_AGENT.json` | CONTACT AGENT | Gemini 2.0 Flash | Get Contacts, Add/Update Contact (Airtable) |
| `CONTENT_CREATOR_AGENT.json` | CONTENT CREATOR AGENT | Gemini 2.0 Flash | Tavily web search → HTML blog post |

---

## ⚙️ Setup Instructions

### Step 1: Import Sub-agents First

The orchestrator references sub-agents by their n8n workflow ID. Import sub-agents first, note their IDs, then update the orchestrator.

Import in this order:
```
1. CONTENT_CREATOR_AGENT.json  → note the workflow ID
2. CONTACT_AGENT.json          → note the workflow ID
3. CALENDAR_AGENT.json         → note the workflow ID
4. EMAIL_AGENT.json            → note the workflow ID
5. MULTI_AGENT_ARCHITECTURE.json  ← import last
```

After importing the orchestrator, open each `toolWorkflow` node (Email Agent, Calendar Agent, etc.) and update the workflow ID to match the IDs assigned to your newly imported sub-agents.

### Step 2: Set Up Your Telegram Bot

1. Open Telegram and message [@BotFather](https://t.me/BotFather)
2. Send `/newbot`, follow prompts, and copy your Bot Token
3. Add the token as a **Telegram API** credential in n8n Cloud
4. Update the Telegram Trigger node with this credential

### Step 3: Add All Credentials in n8n Cloud

| Credential | Used By |
|---|---|
| OpenAI API | Orchestrator (GPT-4o-mini), Whisper, Embeddings |
| OpenRouter API | Calendar Agent (Gemini 2.5), Content Creator (Gemini 2.0) |
| Google Gemini API | Contact Agent (Gemini 2.0 Flash) |
| Gmail OAuth2 | Email Agent |
| Google Calendar OAuth2 | Calendar Agent |
| Airtable Token | Contact Agent |
| Telegram API | Orchestrator trigger and response |
| Pinecone API | Content Creator (if using FAQ knowledge base) |

### Step 4: Activate in Order

1. Activate all sub-agents first
2. Activate the orchestrator last
3. Open Telegram, message your bot, and test

---

## 🧪 Testing It

```
You (Telegram): "Send an email to John about tomorrow's meeting"
Bot: [routes to Email Agent] → looks up John → sends email → confirms

You (Telegram): "Schedule a call with Sarah on Friday at 3pm"
Bot: [routes to Calendar Agent] → creates event with Sarah as attendee → confirms

You (Telegram): "Add Mike to my contacts, his email is mike@example.com"
Bot: [routes to Contact Agent] → upserts in Airtable → confirms

You (Telegram): "Write a blog post about the future of AI in healthcare"
Bot: [routes to Content Creator] → searches web → writes HTML article → returns

You (Telegram): [voice note: "What's on my calendar tomorrow?"]
Bot: [Whisper transcribes] → [routes to Calendar Agent] → returns events
```

---

## 📸 Screenshots


| View | File |
|---|---|
| Orchestrator workflow canvas | `screenshots/orchestrator-canvas.png` |
| Voice message flow | `screenshots/voice-flow.png` |
| Telegram conversation demo | `screenshots/telegram-demo.png` |
| Sub-agent: Email Agent canvas | `screenshots/email-agent.png` |
| Sub-agent: Calendar Agent canvas | `screenshots/calendar-agent.png` |

---

## 🔮 What I Would Add Next

- **Persistent long-term memory** — replace Buffer Window with Pinecone-based memory so the assistant remembers preferences and past conversations across sessions
- **WhatsApp interface** — add a second trigger via Twilio to support WhatsApp in addition to Telegram
- **Error handling** — add retry logic and user-friendly error messages when a sub-agent fails
- **Proactive notifications** — schedule-triggered workflows that send Telegram reminders for calendar events
- **Additional sub-agents** — Task manager (Notion/Todoist), document reader (Google Docs), or a news briefing agent

---

[← Back to Portfolio](../README.md)

