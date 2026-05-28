<div align="center">

# 🤖 n8n AI Automation Portfolio
### by Uday Simhadri

[![n8n Cloud](https://img.shields.io/badge/Platform-n8n%20Cloud-EA4B71?style=for-the-badge&logo=n8n&logoColor=white)](https://app.n8n.cloud)
[![OpenAI](https://img.shields.io/badge/OpenAI-Embeddings%20%7C%20Whisper-412991?style=for-the-badge&logo=openai&logoColor=white)](https://openai.com)
[![Google Gemini](https://img.shields.io/badge/Gemini-Flash-4285F4?style=for-the-badge&logo=google&logoColor=white)](https://deepmind.google/technologies/gemini/)
[![DeepSeek](https://img.shields.io/badge/DeepSeek-v3.2%20%7C%20v4%20Flash-0066FF?style=for-the-badge)](https://deepseek.com)
[![Pinecone](https://img.shields.io/badge/Pinecone-Vector%20DB-00C4A7?style=for-the-badge)](https://pinecone.io)
[![Telegram](https://img.shields.io/badge/Telegram-Bot%20API-26A5E4?style=for-the-badge&logo=telegram&logoColor=white)](https://core.telegram.org/bots)

<br/>

> **A hands-on learning journey through n8n — from a simple chat trigger to a full multi-agent AI system with RAG, voice input, email automation, and customer support.**

</div>

---

## 👋 About This Portfolio

I built this portfolio to master n8n and demonstrate real-world AI workflow automation skills. Starting with zero n8n experience, I engineered my way through progressively complex projects—each one designed to solve a distinct operational challenge. Through this hands-on process, I mastered core engineering concepts including advanced event triggers, LangChain AI agents, vector database indexing, multi-model routing, and multi-system orchestration.

Every workflow here is functional, built on **n8n Cloud**, and connected to real APIs. The JSON blueprints are included in each folder so anyone can import and run them.

**What I learned building this:**
- How n8n's node-based architecture works (triggers, actions, AI nodes)
- How to connect and configure LLMs from multiple providers (OpenAI, Google Gemini, DeepSeek, Gemma via OpenRouter)
- How RAG (Retrieval-Augmented Generation) works in practice — ingestion pipeline + vector search
- How to build AI agents with tool use, memory, and safety constraints
- How to design multi-agent systems where a central orchestrator delegates to specialist sub-agents
- How to handle real-world automation: email, calendar, contacts, voice transcription, content generation

---

## 🗂️ Project Directory

| # | Project | Key Concept Learned | Complexity |
|---|---|---|---|
| 01 | [RAG Pipeline & Chatbot](#01---rag-pipeline--chatbot) | Vector embeddings, Pinecone, RAG | ⭐⭐⭐ |
| 02 | [Email AI Agent](#02---email-ai-agent) | AI agents, safety rails, memory, tool chaining | ⭐⭐⭐ |
| 03 | [Customer Support System](#03---customer-support-system) | Email triggers, text classification, automated replies | ⭐⭐⭐⭐ |
| 04 | [Multi-Agent Architecture](#04---multi-agent-architecture) | Orchestrator pattern, sub-agents, voice, Telegram | ⭐⭐⭐⭐⭐ |

---

## 01 — RAG Pipeline & Chatbot

📁 [`/01-rag-pipeline-chatbot`](./01-rag-pipeline-chatbot/)

**What it does:** Watches a Google Drive folder for new PDFs. When a file is uploaded, it automatically extracts the text, converts it into vector embeddings using OpenAI, and stores them in a Pinecone vector database. A separate chat interface lets users ask questions — the AI agent retrieves the most relevant chunks from Pinecone and answers using DeepSeek v3.2.

**Key concepts I learned:**
- The two-pipeline structure of RAG (ingestion vs. retrieval)
- How vector embeddings represent meaning numerically
- Using Pinecone namespaces to organise document collections
- Connecting an AI agent to a vector store as a retrieval tool

**Tech used:** Google Drive Trigger → OpenAI Embeddings → Pinecone | Chat Trigger → DeepSeek v3.2 (OpenRouter) → Pinecone retrieval

---

## 02 — Email AI Agent

📁 [`/02-email-ai-agent`](./02-email-ai-agent/)

**What it does:** A chat-triggered workspace assistant powered by Gemini Flash. It can send emails and create Google Calendar events — but it has a strict rule: it must always look up the person's email address from a Google Sheets contacts database before taking any action. It can never guess or make up an email address.

**Key concepts I learned:**
- How to give an AI agent safety constraints via the system prompt
- How tool-use ordering works (must query contacts before emailing or scheduling)
- Using Memory Buffer Window to maintain conversation context across turns
- Connecting Google Sheets as a live data source tool

**Tech used:** Chat Trigger → Gemini Flash (OpenRouter) → Memory Buffer + Google Sheets (Contacts) + Gmail + Google Calendar

---

## 03 — Customer Support System

📁 [`/03-customer-support-system`](./03-customer-support-system/)

**What it does:** A fully automated customer support pipeline. When an email arrives in Gmail, a Text Classifier powered by DeepSeek v4 Flash decides if it's a real customer support query or unrelated noise. Genuine support emails are routed to an AI agent (Gemma 3 12B) named "Mr. Tech" that searches a Pinecone FAQ knowledge base and sends a friendly, emoji-filled reply directly via Gmail.

**Key concepts I learned:**
- Using n8n's Text Classifier node for intelligent email triage
- Building event-driven workflows triggered by incoming email
- How to route workflow paths based on AI classification output
- Grounding AI replies in a knowledge base to prevent hallucination

**Tech used:** Gmail Trigger → DeepSeek v4 Flash Classifier → Gemma 3 12B IT (OpenRouter) + Pinecone FAQ → Gmail Send

---

## 04 — Multi-Agent Architecture

📁 [`/04-multi-agent-architecture`](./04-multi-agent-architecture/)

**What it does:** The most complex project — a Telegram bot that acts as a central personal assistant. It handles both text and voice messages (voice is transcribed via OpenAI Whisper), then routes each request to the correct specialist sub-agent: Email Agent, Calendar Agent, Contact Agent, or Content Creator Agent. Each sub-agent is a separate n8n workflow with its own LLM and tools.

**Key concepts I learned:**
- The orchestrator/sub-agent pattern and why it scales better than a single large agent
- How to use `Execute Workflow` nodes to call other workflows as tools
- Voice message handling: downloading Telegram audio and transcribing with Whisper
- How to use the `Think` tool to give the orchestrator a reasoning step
- Managing conversational memory at the orchestrator level

**Tech used:** Telegram Bot → Switch (text/voice) → OpenAI Whisper → GPT-4o-mini Orchestrator → [Email Agent | Calendar Agent | Contact Agent | Content Creator Agent]

---

## 🏗️ Overall Architecture

```
TELEGRAM (text or voice)
         │
         ▼
  ┌─────────────┐
  │   Switch    │ ── voice ──► Download Audio ──► OpenAI Whisper Transcribe
  └──────┬──────┘
         │ text / transcribed text
         ▼
  ┌──────────────────────┐
  │  GPT-4o-mini         │  ◄── Simple Memory (Buffer Window)
  │  Orchestrator Agent  │  ◄── Think Tool (chain-of-thought)
  │  "Ultimate Assistant"│  ◄── Tavily Web Search
  │                      │  ◄── Calculator
  └──────────┬───────────┘
             │
    ┌────────┼──────────────────┐
    ▼        ▼        ▼         ▼
 EMAIL    CALENDAR  CONTACT  CONTENT
 AGENT    AGENT     AGENT    CREATOR
 GPT-4o   Gemini    Gemini   Gemini
 -mini    2.5Flash  2.0Flash 2.0Flash
    │        │        │         │
  Gmail  G.Calendar Airtable  Tavily
```

---

## 📁 Repository Structure

```
n8n-ai-portfolio/
│
├── README.md                          ← You are here
│
├── 01-rag-pipeline-chatbot/
│   ├── README.md
│   ├── RAG_PIPELINE_CHATBOT.json      ← Import into n8n Cloud
│   └── screenshots/
│
├── 02-email-ai-agent/
│   ├── README.md
│   ├── EMAIL_AI_AGENT.json
│   └── screenshots/
│
├── 03-customer-support-system/
│   ├── README.md
│   ├── CUSTOMER_SUPPORT_SYSTEM.json
│   └── screenshots/
│
└── 04-multi-agent-architecture/
    ├── README.md
    ├── MULTI_AGENT_ARCHITECTURE.json
    ├── sub-agents/
    │   ├── EMAIL_AGENT.json
    │   ├── CALENDAR_AGENT.json
    │   ├── CONTACT_AGENT.json
    │   └── CONTENT_CREATOR_AGENT.json
    └── screenshots/
```

---

## 🛠️ How to Run Any of These

All workflows run on **n8n Cloud** — no local setup needed.

1. Sign up at [app.n8n.cloud](https://app.n8n.cloud)
2. Go to **Workflows → Import from File** and upload the `.json` file
3. Add your API credentials under **Settings → Credentials**
4. Activate the workflow

Each project folder has a detailed setup guide with all the credentials you need.

---

## 👤 Author

**Uday Simhadri**

[![GitHub](https://img.shields.io/badge/GitHub-udaysimhadrii-181717?style=for-the-badge&logo=github)](https://github.com/udaysimhadrii)

---

## 📄 License

MIT — feel free to use these workflows as a learning reference or starting point for your own projects.

