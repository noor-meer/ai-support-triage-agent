# 🤖 AI Support Ticket Triage Agent (with RAG-Powered Knowledge Base)

An intelligent customer support automation system built in n8n that reads incoming support emails, classifies them using Google Gemini, retrieves real answers from a company knowledge base using RAG (Retrieval-Augmented Generation), and either drafts and sends a fact-based auto-reply for simple queries, or escalates complex/urgent tickets to a human support team via Slack.

## 🎯 Problem This Solves

Support teams receive a mix of simple questions and urgent/sensitive complaints in the same inbox, with no automatic way to tell them apart and even AI-assisted replies are often unreliable because a generic AI has no real knowledge of the company's actual policies. This agent solves both problems: it triages every ticket automatically, and when it does auto-reply, the answer is grounded in real, retrievable facts instead of AI guesswork.

## ⚙️ How It Works

1. **Trigger:** Gmail inbox is monitored for new incoming support emails
2. **AI Classification:** Google Gemini (`gemini-2.5-flash`) reads the email and returns a structured classification category, urgency, sentiment, confidence score, and reasoning
3. **Routing Decision:** An IF node checks whether the ticket is simple and safe enough to auto-handle (category = general question AND confidence ≥ 70%)
4. **Escalation Path:** If not safe to auto-handle, a detailed alert (with the AI's full reasoning) is sent to a dedicated Slack channel for a human agent to review
5. **Auto-Handle Path:**
   - The customer's question is used to search a Pinecone vector database containing the company's real FAQ/policy knowledge base
   - The top matching knowledge chunks are retrieved and merged into a single context block
   - Gemini drafts a reply using only the retrieved factual context not general knowledge or guesses
   - A heads-up notification is posted to Slack, followed by a 2-minute safety delay, then the reply is automatically emailed to the customer
6. **Logging:** Every ticket regardless of path is logged to a Google Sheet with full classification data, for auditability and performance tracking over time

## 🧠 RAG Architecture (Knowledge Base Upgrade)

This project was upgraded from a general-purpose AI responder into a RAG-grounded support agent the key differentiator the market currently demands over generic "ChatGPT wrapper" support bots.

**How the knowledge base works:**

- A fictional SaaS company's ("TaskFlow") FAQ/policy content was written and embedded using Google's `gemini-embedding-001` embedding model
- Embeddings are stored in a Pinecone vector index (3072 dimensions, cosine similarity, serverless)
- At query time, the customer's question is embedded the same way and matched against the stored knowledge using semantic similarity search meaning it can find the right answer even if the customer doesn't use the exact wording from the FAQ
- Retrieved chunks are merged into one block of context and injected directly into the reply-drafting prompt, with an explicit instruction that the AI must answer only from that context, or honestly say it doesn't know rather than guess

## 🛠️ Tech Stack

- **Automation:** n8n (self-hosted)
- **AI/LLM:** Google Gemini API (`gemini-2.5-flash` for classification and reply drafting)
- **Embeddings:** Google Gemini API (`gemini-embedding-001`, 3072-dimension output)
- **Vector Database:** Pinecone (serverless, cosine similarity)
- **Email:** Gmail API (OAuth2)
- **Notifications:** Slack API
- **Logging:** Google Sheets API

## 🔑 Key Learnings

- Field names from trigger nodes must be verified against actual output data, not assumed (e.g., Gmail's `Subject` vs `subject`, `snippet` vs `text`)
- `$('Node Name').item.json.field` is essential once a workflow branches past nodes that don't carry the original trigger data forward automatically
- AI models can fabricate specific facts even when told not to in general terms being explicit and specific in guardrail instructions measurably improved output honesty, and RAG eliminates this risk entirely for anything covered in the knowledge base
- Embedding model output size must exactly match the vector database's configured dimensions. A default embedding model (`text-embedding-004`) turned out to be deprecated; the replacement (`gemini-embedding-001`) outputs 3072-dimension vectors, requiring the Pinecone index to be rebuilt to match
- Vector search results return as multiple separate items, not one bundled item. This caused a "paired item data unavailable" error several nodes downstream, because every node after the search was unexpectedly running once per retrieved chunk instead of once per ticket. The fix: merge all retrieved chunks into a single item immediately after the vector search step, before any node that expects one item per original ticket
- New/unfamiliar sender addresses can trigger Gmail's spam filter on a fresh inbox, which will silently prevent a trigger from firing always check Spam during testing before assuming a workflow is broken
- A workflow being "Active" does not mean every configuration is correct always verify with a fully hands-off, real-world test before considering a build production-ready

## 📋 Setup Instructions

1. Import the workflow JSON into your n8n instance
2. Create a Pinecone account and index (dimension must match your chosen embedding model's output size)
3. Connect your own Gmail, Google Gemini, Pinecone, Slack, and Google Sheets credentials
4. Upload your own knowledge base content using the "Add documents to vector store" flow (a one-time setup step, separate from the live workflow)
5. Update the Slack channel name and Google Sheet reference to your own
6. Activate the workflow

## Demo

Watch the full walkthrough (https://drive.google.com/file/d/1XG_W14WNfk_f8VdP45LEcsOgCD1_Am4w/view?usp=sharing)

## 📌 Status

Fully functional tested end-to-end for both the RAG-powered auto-handle path and the escalation path, including a full hands-off (no manual node execution) live test.

## 👤 Built By

Noor ul Ain — Agentic AI Developer specializing in Python, machine
learning, and AI-powered automation using n8n.
