<div align="center">

# 🧠 Project 01 — RAG Pipeline & Chatbot

[![n8n Cloud](https://img.shields.io/badge/Platform-n8n%20Cloud-EA4B71?style=for-the-badge&logo=n8n&logoColor=white)](https://app.n8n.cloud)
[![DeepSeek](https://img.shields.io/badge/LLM-DeepSeek%20v3.2-0066FF?style=for-the-badge)](https://deepseek.com)
[![OpenAI](https://img.shields.io/badge/Embeddings-OpenAI-412991?style=for-the-badge&logo=openai&logoColor=white)](https://openai.com)
[![Pinecone](https://img.shields.io/badge/Vector%20DB-Pinecone-00C4A7?style=for-the-badge)](https://pinecone.io)
[![Google Drive](https://img.shields.io/badge/Source-Google%20Drive-4285F4?style=for-the-badge&logo=googledrive&logoColor=white)](https://drive.google.com)

</div>

---

## 📌 What I Built

This is a **Retrieval-Augmented Generation (RAG)** system — one of the most important patterns in modern AI development. It has two distinct pipelines running in the same workflow:

**Pipeline 1 — Document Ingestion (automated):** Watches a Google Drive folder called "FAQ". Any time a PDF is added to that folder, the workflow automatically downloads it, extracts the text, converts it into vector embeddings using OpenAI, and stores those embeddings in a Pinecone vector database under the "FAQ" namespace.

**Pipeline 2 — Chat & Retrieval (on demand):** A chat interface where users can ask questions in plain English. An AI agent powered by DeepSeek v3.2 receives the question, searches the Pinecone index for the most semantically relevant content, and uses it to answer accurately.

---

## 💡 What I Learned Building This

This was my first time working with vector databases and embeddings. Here's what the process taught me:

**RAG vs. just asking an LLM:** An LLM on its own only knows what it was trained on. RAG lets you give it access to *your own documents* at query time — it doesn't need to be retrained. The LLM uses your content as context to answer questions it would otherwise get wrong or make up.

**What embeddings actually are:** When you send text to OpenAI's embedding model, it returns a list of ~1500 numbers. These numbers represent the *meaning* of the text in mathematical form. Two pieces of text with similar meanings will have similar numbers. Pinecone stores these and finds the closest matches to any new query — that's the "retrieval" in RAG.

**The two-pipeline pattern:** The ingestion pipeline (Google Drive → Pinecone) runs once per document. The retrieval pipeline (Chat → Agent → Pinecone search → Answer) runs every time a user asks a question. Keeping them separate is cleaner and more efficient.

**Namespaces in Pinecone:** I used the `FAQ` namespace to keep this document collection isolated from any other data that might be stored in the same Pinecone index. This is important for multi-tenant or multi-topic use cases.

---

## 🗺️ How It Works — Node by Node

### Pipeline 1: Document Ingestion

```
Google Drive Trigger (polls every minute, watches "FAQ" folder)
        │
        │  fires when a new PDF is uploaded
        ▼
Download File (Google Drive node — fetches the binary PDF)
        │
        ▼
Extract from File (PDF extractor — pulls raw text from the PDF)
        │
        ▼
Pinecone Vector Store [INSERT mode]
        │  ▲
        │  └── Default Data Loader (loads binary PDF content for chunking)
        │  └── OpenAI Embeddings (converts text chunks → vectors)
        │
        ▼
  Vectors upserted into Pinecone under namespace: "FAQ"
```

### Pipeline 2: Chat & Retrieval

```
Chat Trigger (n8n built-in chat interface)
        │
        ▼
AI Agent (DeepSeek v3.2 via OpenRouter)
        │  ◄── Pinecone Vector Store [RETRIEVE-AS-TOOL mode]
        │             └── OpenAI Embeddings (embeds the user's question
        │                  to find matching content in Pinecone)
        ▼
Answer returned to chat UI
```

---

## 🔧 Nodes Used

| Node | Purpose |
|---|---|
| `Google Drive Trigger` | Polls the "FAQ" folder every minute for new files |
| `Google Drive (Download)` | Downloads the new PDF as binary data |
| `Extract from File` | Extracts raw text from the PDF binary |
| `Default Data Loader` | Loads binary PDF into LangChain document format for chunking |
| `Pinecone Vector Store (insert)` | Writes embedded document chunks to Pinecone |
| `OpenAI Embeddings` (x2) | Converts text to vectors for both ingestion and retrieval |
| `Chat Trigger` | Opens the n8n chat interface for user questions |
| `AI Agent` | The reasoning brain — uses DeepSeek v3.2 via OpenRouter |
| `Pinecone Vector Store (retrieve-as-tool)` | Lets the agent search Pinecone for relevant content |

---

## 🏗️ Architecture Diagram

```
  ┌──────────────────────────────────────────────┐
  │           INGESTION PIPELINE                 │
  │                                              │
  │  Google Drive  →  Download  →  Extract PDF  │
  │       Trigger       File         Text        │
  │                                    │         │
  │                              OpenAI          │
  │                             Embeddings       │
  │                                    │         │
  │                              Pinecone        │
  │                            Vector Store      │
  │                          (namespace: FAQ)    │
  └──────────────────────────────────────────────┘

  ┌──────────────────────────────────────────────┐
  │           RETRIEVAL PIPELINE                 │
  │                                              │
  │  Chat Trigger  →  AI Agent (DeepSeek v3.2)  │
  │                        │                    │
  │                  [Tool Call]                 │
  │                        ▼                    │
  │               Pinecone Search               │
  │             (OpenAI Embeddings              │
  │              + namespace: FAQ)              │
  │                        │                    │
  │               Relevant chunks               │
  │               returned to agent             │
  │                        │                    │
  │               Final answer → User           │
  └──────────────────────────────────────────────┘
```

---

## ⚙️ Setup Instructions

### Step 1: Set up Pinecone

1. Create a free account at [pinecone.io](https://pinecone.io)
2. Create a new Index with **dimensions: 1536** (matching OpenAI's `text-embedding-ada-002` output)
3. Note your index name — update it in both Pinecone nodes
4. Both nodes should use namespace: `FAQ`

### Step 2: Prepare Google Drive

1. Create a folder in Google Drive named `FAQ`
2. Note the folder ID from the URL: `drive.google.com/drive/folders/YOUR_FOLDER_ID`
3. Update the Google Drive Trigger node with this folder ID

### Step 3: Add Credentials in n8n Cloud

| Credential | Type | Where to Get |
|---|---|---|
| Google Drive | OAuth2 | [Google Cloud Console](https://console.cloud.google.com) — enable Drive API |
| OpenAI | API Key | [platform.openai.com/api-keys](https://platform.openai.com/api-keys) |
| Pinecone | API Key | Your Pinecone console |
| OpenRouter | API Key | [openrouter.ai/keys](https://openrouter.ai/keys) |

### Step 4: Import & Activate

1. Import `RAG_PIPELINE_CHATBOT.json` into n8n Cloud
2. Connect all credentials
3. Activate the workflow
4. Upload a PDF to your Google Drive FAQ folder — wait ~1 minute for ingestion
5. Open the Chat interface and ask a question about the PDF content

---

## 🧪 Testing It

1. Upload any PDF (product manual, FAQ document, policy doc) to your Google Drive FAQ folder
2. Wait 1–2 minutes for the ingestion pipeline to run
3. Click "Open Chat" on the workflow canvas
4. Ask a question about the document's content
5. The agent should answer accurately based on your document, not from general LLM knowledge

**Good test question format:** *"According to the document, what is the return policy?"* or *"What does the FAQ say about [specific topic]?"*

---

## 📸 Screenshots

> Add your n8n canvas screenshot to `screenshots/` and update this section.

| View | File |
|---|---|
| Full workflow canvas | `screenshots/workflow-canvas.png` |


---

## 🔮 What I Would Add Next

- **Recursive text splitter** — better chunking strategy for large PDFs
- **Metadata filtering** — tag each chunk with its source filename so the agent can cite which document it used
- **Multiple namespaces** — separate FAQ, Policies, and Products into different namespaces
- **Delete-on-update** — detect when a PDF is replaced and remove old vectors

---

[← Back to Portfolio](../README.md)

