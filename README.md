# 🤖 N8N Workflows

A collection of AI-powered automation workflows built with [n8n](https://n8n.io/) — covering chatbots, RAG pipelines, weather agents, and e-commerce AI integrations.

---

## 📋 Workflows Overview

### 1. � Panda-Bot

The AI-powered customer support chatbot for the **Panda Shop** multivendor e-commerce platform. It answers product and general queries using a Retrieval-Augmented Generation (RAG) pipeline over a Pinecone vector database.

| Detail | Value |
|---|---|
| **Trigger** | HTTP Webhook (`POST /chat-message`) |
| **LLM** | Google Gemini |
| **Vector DB** | Pinecone (`panda-shop` index, `chatbot` namespace) |
| **Embeddings** | Google Gemini Embeddings (`gemini-embedding-001`) |

**How it works:** Incoming chat messages from the frontend hit the webhook endpoint. The message is passed through a QA Chain that retrieves relevant context from Pinecone (containing product data, reviews, etc.) and generates a grounded response, which is sent back as a plain-text HTTP response.

**🛒 Part of the Panda Shop E-Commerce Platform:**

| Repository | Description |
|---|---|
| [panda-shop (Frontend)](https://github.com/TanvirCou/panda-shop) | React-based multivendor e-commerce frontend |
| [panda-shop-server (Backend)](https://github.com/TanvirCou/panda-shop-server) | Node.js/Express REST API & MongoDB backend |

---

### 2. 📱 Messenger Chatbot

An AI chatbot integrated directly into **Facebook Messenger** — acts as an e-commerce assistant.

| Detail | Value |
|---|---|
| **Trigger** | Facebook Messenger Webhook |
| **LLM** | Google Gemini |
| **Memory** | Buffer Window Memory (per sender ID) |
| **Integration** | Facebook Graph API v25.0 |

**How it works:** Handles the Messenger webhook verification handshake (GET request) automatically. On incoming messages (POST), the AI Agent processes the text and replies back to the user directly in Messenger via the Graph API. Responses are kept plain-text, concise (max 200 words), and no markdown.

---

### 3. �️ Basic Chatbot (Perplexity)

A conversational AI chatbot that answers user queries using **real-time web search**.

| Detail | Value |
|---|---|
| **Trigger** | n8n Chat UI |
| **LLM** | Groq (`openai/gpt-oss-safeguard-20b`) |
| **Tools** | SerpAPI (real-time web search) |
| **Memory** | Buffer Window Memory |

**How it works:** When a message is received via the n8n chat interface, the AI Agent fetches up-to-date information using SerpAPI before formulating a response. Conversation history is maintained within the session.

---

### 4. 🧠 RAG Pipeline Chatbot

A two-phase workflow: **ingestion** (loading documents into a vector store) and **retrieval** (answering questions from stored knowledge).

| Detail | Value |
|---|---|
| **Ingestion Trigger** | Manual Execute |
| **Data Source** | Google Drive (PDF file) |
| **Vector DB** | Pinecone (`panda-shop` index, `products` namespace) |
| **Chat Trigger** | n8n Chat UI |
| **LLM** | Google Gemini |
| **Embeddings** | Google Gemini Embeddings (`gemini-embedding-001`) |

**How it works:**
- **Phase 1 (Ingestion):** Manually triggered — downloads a PDF from Google Drive, splits and embeds the document content, then upserts chunks into Pinecone.
- **Phase 2 (Chat):** A chat-triggered QA chain retrieves relevant chunks from Pinecone and answers user questions using Gemini.

---

### 5. 🌤️ Weather Bot (Parent + Child)

A two-workflow agentic system where an AI agent delegates weather lookups to a specialized child workflow tool.

#### Parent: `Weather`

| Detail | Value |
|---|---|
| **Trigger** | n8n Chat UI |
| **LLM** | Google Gemini |
| **Memory** | Buffer Window Memory |
| **Tool** | Calls `Child Weather` sub-workflow |

**How it works:** The parent agent receives a natural language weather query (e.g. "What's the weather in Tokyo?"), decides to use the `Child Weather` tool, and presents the result in a conversational response.

#### Child: `Child Weather`

| Detail | Value |
|---|---|
| **Trigger** | Called by parent workflow |
| **API** | OpenWeatherMap |
| **LLM** | Google Gemini 2.5 Flash |

**How it works:** Receives a city name from the parent, fetches current weather data from OpenWeatherMap, then uses Gemini to format the raw data (temp, feels like, description) into a readable plain-text summary before returning to the parent.

---

## ⚙️ Local Setup — n8n + ngrok with Docker

Run n8n locally using Docker and expose it to the internet with ngrok (required for external webhooks like Messenger).

### Prerequisites

- [Docker Desktop](https://www.docker.com/products/docker-desktop/) installed and running
- [ngrok](https://ngrok.com/download) account and CLI installed
- API keys ready (Google Gemini, Pinecone, SerpAPI, OpenWeatherMap, etc.)

---

### Step 1: Start n8n with Docker

```bash
docker run -it --rm \
  --name n8n \
  -p 5678:5678 \
  -v n8n_data:/home/node/.n8n \
  n8nio/n8n
```

> **Windows (PowerShell):**
> ```powershell
> docker run -it --rm `
>   --name n8n `
>   -p 5678:5678 `
>   -v n8n_data:/home/node/.n8n `
>   n8nio/n8n
> ```

n8n will be accessible at **http://localhost:5678**

---

### Step 2: Expose n8n with ngrok

Open a new terminal and run:

```bash
ngrok http 5678
```

Copy the **Forwarding URL** (e.g. `https://abc123.ngrok-free.app`). This is your public webhook base URL.

---

### Step 3: Configure n8n Webhook URL

Set the `WEBHOOK_URL` environment variable so n8n uses the ngrok URL for all webhooks:

```bash
docker run -it --rm \
  --name n8n \
  -p 5678:5678 \
  -e WEBHOOK_URL=https://abc123.ngrok-free.app \
  -v n8n_data:/home/node/.n8n \
  n8nio/n8n
```

> Replace `https://abc123.ngrok-free.app` with your actual ngrok URL each time you start a new ngrok session.

---

### Step 4: Import Workflows

1. Open **http://localhost:5678** in your browser
2. Go to **Workflows → Add Workflow → Import from File**
3. Import the `.json` files from this repository
4. Add your credentials under **Settings → Credentials**

---

### Step 5: Activate Workflows with Webhooks

For webhook-based workflows (Panda-Bot, Messenger Chatbot):

1. Open the workflow in n8n
2. Click **Activate** (toggle top-right)
3. Copy the webhook URL shown in the Webhook node (it will use your ngrok base URL)
4. Use this URL in your external service (e.g., Facebook App Dashboard for Messenger, or your frontend `.env`)

---

### Required Credentials

| Credential | Used In |
|---|---|
| Google Gemini (PaLM) API | Panda-Bot, Messenger Chatbot, RAG Pipeline, Weather Bot |
| Pinecone API | Panda-Bot, RAG Pipeline Chatbot |
| SerpAPI | Basic Chatbot (Perplexity) |
| OpenWeatherMap API | Child Weather |
| Google Drive OAuth2 | RAG Pipeline Chatbot |
| Facebook Page Access Token | Messenger Chatbot |
| Groq API | Basic Chatbot (Perplexity) |

---

> **Note on ngrok URLs:** Free ngrok sessions generate a new URL each restart. Update the `WEBHOOK_URL` environment variable and re-register any external webhooks (Facebook, etc.) whenever the ngrok URL changes.
