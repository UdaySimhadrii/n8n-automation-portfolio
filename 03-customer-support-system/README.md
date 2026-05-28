<div align="center">

# 🎧 Project 03 — Customer Support System

[![n8n Cloud](https://img.shields.io/badge/Platform-n8n%20Cloud-EA4B71?style=for-the-badge&logo=n8n&logoColor=white)](https://app.n8n.cloud)
[![DeepSeek](https://img.shields.io/badge/Classifier-DeepSeek%20v4%20Flash-0066FF?style=for-the-badge)](https://deepseek.com)
[![Gemma](https://img.shields.io/badge/Agent-Gemma%203%2012B%20IT-FF6F00?style=for-the-badge)](https://ai.google.dev/gemma)
[![Pinecone](https://img.shields.io/badge/Knowledge%20Base-Pinecone-00C4A7?style=for-the-badge)](https://pinecone.io)
[![Gmail](https://img.shields.io/badge/Gmail-Trigger%20%26%20Send-EA4335?style=for-the-badge&logo=gmail&logoColor=white)](https://developers.google.com/gmail/api)

</div>

---

## 📌 What I Built

A **fully automated customer support email pipeline** for a fictional company called "Tech Haven". When a new email arrives in Gmail, the workflow:

1. Classifies it — is it a genuine customer support query, or something unrelated?
2. Routes it — real support emails go to the AI agent; everything else is discarded
3. Generates a reply — the agent (named "Mr. Tech") searches a Pinecone FAQ knowledge base and writes a friendly, emoji-filled response
4. Sends it — the reply goes back to the customer automatically via Gmail

This was the first workflow I built that is **fully event-driven** — it runs on its own without any human trigger, end to end.

---

## 💡 What I Learned Building This

**Event-driven automation:** The Gmail Trigger is what made this feel "real". Unlike a Chat Trigger that waits for me to type something, this workflow fires automatically whenever a new email arrives. That's the core of production automation — things that happen on their own.

**Text Classification as a routing mechanism:** n8n's Text Classifier node was new to me. You give it categories and descriptions, connect an LLM, and it returns which category the input belongs to. I used it to separate real support emails from noise (newsletters, spam, unrelated messages). The two categories I defined:
- `customer support` — emails asking about policies, products, or services
- `other` — everything else

**Why classification matters before the AI agent:** If I sent every email straight to the AI agent, it would try to reply to newsletters, spam, and automated notifications. The classifier acts as a filter, so the expensive LLM call (the full AI agent) only happens for real support queries. This is good practice for cost and accuracy.

**Grounding replies in a knowledge base:** The AI agent (Gemma 3 12B) doesn't just answer from general LLM knowledge — it searches the Pinecone FAQ index first. This means replies are based on *Tech Haven's actual FAQ content*, not hallucinated information. This is the RAG pattern applied to customer support.

**Persona and tone through system prompts:** I set the agent's persona entirely via the system prompt:
```
you are a customer support agent for tech haven.
your job is respond to incoming emails with relevant information using your knowledge tool
- you output should be friendly and use emojis
- signing off with mr.tech
- output only the body content of the email
```
The last instruction — *output only the body content* — is important. Without it, the LLM might add its own "Subject:" line or preamble, which would break the Gmail send node.

---

## 🗺️ How It Works — Node by Node

```
New email arrives in Gmail inbox
        │
        ▼
Gmail Trigger (fires on every new email)
        │
        ▼
Text Classifier (DeepSeek v4 Flash via OpenRouter)
        │
        │   Classifies the email into one of two categories:
        ├── "customer support" ──────────────────────┐
        │                                            ▼
        └── "other" ──► NoOp (do nothing, stop)   AI Agent (Gemma 3 12B IT)
                                                     │
                                            ┌────────┤
                                            │        │
                                    [Tool Call]      │
                                            ▼        │
                                    Pinecone Search  │
                                    (FAQ namespace)  │
                                    + OpenAI         │
                                    Embeddings       │
                                            │        │
                                    Relevant FAQ     │
                                    chunks returned  │
                                            └────────┘
                                                     │
                                             Friendly reply
                                             written as Mr. Tech
                                                     │
                                                     ▼
                                             Gmail — Send Message
                                             (reply to original sender)
```

---

## 🔧 Nodes Used

| Node | Purpose |
|---|---|
| `Gmail Trigger` | Fires when a new email arrives in the inbox |
| `Text Classifier` | Routes the email — "customer support" or "other" |
| `OpenRouter Chat Model` (DeepSeek v4 Flash) | Powers the Text Classifier |
| `No Operation` | Silently discards non-support emails |
| `AI Agent` | Generates the customer reply as "Mr. Tech" |
| `OpenRouter Chat Model` (Gemma 3 12B IT) | Powers the AI Agent |
| `Pinecone Vector Store` (retrieve-as-tool) | Searches the FAQ knowledge base |
| `OpenAI Embeddings` | Embeds the query for Pinecone similarity search |
| `Gmail (Send Message)` | Sends the generated reply back to the customer |

---

## 🏗️ Architecture Diagram

```
  ┌─────────────────────────────────────────────────────────┐
  │              CUSTOMER SUPPORT SYSTEM                     │
  │                                                          │
  │   Incoming Email                                         │
  │         │                                                │
  │         ▼                                                │
  │   Gmail Trigger                                          │
  │         │                                                │
  │         ▼                                                │
  │   ┌─────────────────────────────────────────┐           │
  │   │   Text Classifier (DeepSeek v4 Flash)   │           │
  │   │                                         │           │
  │   │   Input: email subject + body           │           │
  │   │   Categories:                           │           │
  │   │     ✅ "customer support" → Agent       │           │
  │   │     ❌ "other"            → NoOp        │           │
  │   └─────────────────────────────────────────┘           │
  │         │                          │                     │
  │         ▼                          ▼                     │
  │   AI Agent                      No Operation             │
  │   (Gemma 3 12B IT)              (workflow ends)          │
  │         │                                                │
  │         ├── Pinecone FAQ Search                          │
  │         │   (OpenAI Embeddings + "FAQ" namespace)        │
  │         │                                                │
  │         ▼                                                │
  │   Friendly reply as "Mr. Tech" 😊                        │
  │         │                                                │
  │         ▼                                                │
  │   Gmail — Send Reply to customer                         │
  └─────────────────────────────────────────────────────────┘
```

---

## 🤖 The AI Agent's Persona

The support agent "Mr. Tech" is defined entirely in the system prompt:

```
You are a customer support agent for Tech Haven.
Your job is to respond to incoming emails with relevant information
using your knowledge tool.

Instructions:
- Your output should be friendly and use emojis
- Sign off with "Mr. Tech"
- Output only the body content of the email
```

**Why "output only the body content"?** The Gmail Send node expects just the email body as input. If the LLM adds formatting like "Subject: Re: Your Query" or "Here is my reply:", the email would look broken to the recipient. This instruction keeps the output clean and directly usable.

---

## ⚙️ Setup Instructions

### Step 1: Populate the Pinecone FAQ Index

This workflow uses the same Pinecone FAQ namespace as Project 01. If you've already set up the RAG Pipeline, the knowledge base is ready. If not:

1. Create a Pinecone index (dimensions: 1536)
2. Use Project 01's ingestion pipeline to upload FAQ documents
3. Both should use namespace: `FAQ`

### Step 2: Configure the Gmail Trigger

1. The Gmail Trigger monitors your inbox for new emails
2. Make sure the Gmail credential has `https://mail.google.com/` scope
3. You can add filters (e.g., only watch a specific label) to avoid processing every single email during testing

### Step 3: Add Credentials in n8n Cloud

| Credential | Type | Where to Get |
|---|---|---|
| Gmail (Trigger) | OAuth2 | [Google Cloud Console](https://console.cloud.google.com) |
| Gmail (Send) | OAuth2 | Same credential as above |
| OpenRouter | API Key | [openrouter.ai/keys](https://openrouter.ai/keys) |
| OpenAI | API Key | [platform.openai.com/api-keys](https://platform.openai.com/api-keys) |
| Pinecone | API Key | Your Pinecone console |

### Step 4: Import & Activate

1. Import `CUSTOMER_SUPPORT_SYSTEM.json` into n8n Cloud
2. Connect all credentials
3. Update the Pinecone index name in both Pinecone nodes
4. Activate the workflow
5. Send a test email to your Gmail inbox — the workflow will fire automatically

---

## 🧪 Testing It

**Test 1 — Genuine support query:**
Send yourself an email with subject *"Question about return policy"* and body *"Hi, I bought a product last week. What is your return policy?"*

Expected result: Gmail Trigger fires → Classifier routes to "customer support" → Mr. Tech replies with FAQ-based answer.

**Test 2 — Non-support email (classifier filter):**
Send yourself a newsletter or a random email unrelated to support.

Expected result: Classifier routes to "other" → NoOp → workflow ends, no reply sent.

---




---

## 🔮 What I Would Add Next

- **Reply threading** — ensure replies are sent in the same Gmail thread, not as new emails
- **Ticket logging** — log each support interaction to a Google Sheet or Airtable for tracking
- **Escalation path** — if Pinecone returns no relevant results, flag the email for human review instead of sending a potentially unhelpful reply
- **Multi-language support** — detect the customer's language and reply in the same language

---

[← Back to Portfolio](../README.md)

