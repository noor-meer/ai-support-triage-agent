# 🤖 AI Support Ticket Triage Agent (with RAG-Powered Knowledge Base)

An AI-powered customer support automation system built in **n8n** that processes incoming support emails, classifies them using **Google Gemini**, retrieves relevant information from a company knowledge base using **RAG (Retrieval-Augmented Generation)**, and either sends a knowledge-grounded response or escalates complex and urgent tickets to a human support team via Slack.

## 🎯 Problem This Solves

Support teams often receive a mix of simple questions, general requests, and urgent or sensitive issues in the same inbox. Manually reviewing and routing every ticket can be repetitive, while generic AI-generated responses may lack accurate company-specific knowledge.

This project demonstrates how AI-powered ticket triage and RAG can automate classification and routing while grounding responses in company FAQ and policy content.

## ⚙️ How It Works

1. **Trigger:** Gmail monitors incoming support emails.
2. **AI Classification:** Google Gemini analyzes each email by category, urgency, sentiment, and confidence score.
3. **Routing:** The workflow determines whether the ticket can be handled automatically or requires human review.
4. **Human Escalation:** Complex or urgent tickets are sent to a dedicated Slack channel with their classification details and context.
5. **RAG Auto-Response:**
   - The customer's question is matched against a Pinecone vector database containing company FAQ and policy content.
   - Relevant knowledge is retrieved and provided to Gemini as context.
   - Gemini generates a knowledge-grounded response.
   - The response is automatically sent to the customer through Gmail.
6. **Logging:** Every ticket is logged in Google Sheets for tracking and auditability.

## 🧠 RAG Architecture

The workflow uses a RAG-based knowledge layer to ground automated responses in company-specific information.

- TaskFlow FAQ and policy content is embedded using **Google Gemini embeddings**.
- Embeddings are stored in a **Pinecone vector index**.
- Customer questions are matched against the stored knowledge using semantic search.
- Relevant knowledge is retrieved and supplied to Gemini as context.
- The model generates a response based on the retrieved information.

## 🛠️ Tech Stack

- **n8n** — Workflow automation and orchestration
- **Google Gemini API** — Ticket classification and response generation
- **Pinecone** — Vector database and semantic retrieval
- **Gmail API** — Incoming emails and automated responses
- **Slack API** — Human escalation notifications
- **Google Sheets API** — Ticket logging and tracking

## 📋 Setup Instructions

1. Import the workflow JSON into n8n.
2. Create and configure the Pinecone index.
3. Connect Gmail, Google Gemini, Pinecone, Slack, and Google Sheets credentials.
4. Upload the knowledge base content to the vector store.
5. Configure the Slack channel and Google Sheet.
6. Activate and test the workflow.

> API keys, credentials, and other sensitive information are not included in this repository.

## 🎥 Demo

Watch the complete workflow walkthrough:

[View Demo](https://drive.google.com/file/d/1XG_W14WNfk_f8VdP45LEcsOgCD1_Am4w/view?usp=sharing)

The workflow was tested across both the **RAG-powered automated response path** and the **human escalation path**.

## 👤 Built By

**Noor ul Ain**  
AI Automation Developer | n8n • AI Agents • RAG • API Integrations • Python
