# 🤖 AI Support Ticket Triage Agent

An intelligent customer support automation system built in n8n that reads incoming support emails, classifies them using Google Gemini, and automatically routes them — either drafting and sending a safe auto-reply for simple queries, or escalating complex/urgent tickets to a human support team via Slack.

## 🎯 Problem This Solves

Support teams receive a mix of simple questions and urgent/sensitive complaints in the same inbox, with no automatic way to tell them apart. This leads to slow response times on urgent issues and wasted human time on simple, repetitive questions. This agent solves that by triaging every ticket automatically — like a triage nurse for your support inbox.

## ⚙️ How It Works

1. **Trigger:** Gmail inbox is monitored for new incoming support emails
2. **AI Classification:** Google Gemini (`gemini-2.5-flash`) reads the email and returns a structured classification — category, urgency, sentiment, confidence score, and reasoning
3. **Routing Decision:** An IF node checks whether the ticket is simple and safe enough to auto-handle (category = general question AND confidence ≥ 70%)
4. **Escalation Path:** If not safe to auto-handle, a detailed alert (with the AI's full reasoning) is sent to a dedicated Slack channel for a human agent to review
5. **Auto-Handle Path:** If safe, Gemini drafts a reply, a heads-up notification is posted to Slack, a 2-minute safety delay runs, then the reply is automatically emailed to the customer
6. **Logging:** Every ticket — regardless of path — is logged to a Google Sheet with full classification data, for auditability and performance tracking over time

## 🧠 AI & Safety Design

- **Structured output:** Gemini is prompted to return strict JSON, allowing the workflow to make reliable, rule-based routing decisions rather than parsing free text
- **Conservative routing:** Only simple, high-confidence, non-sensitive queries are auto-handled. Anything urgent, billing-related, or low-confidence is escalated to a human
- **Hallucination guardrails:** The reply-drafting prompt explicitly instructs the AI not to invent specific facts (hours, policies, prices) it wasn't given — verified through testing that this significantly reduced fabricated details in responses
- **Human-in-the-loop safety window:** Auto-replies are not sent instantly — a Slack notification and 2-minute delay give a human visibility before the email goes out

## 🛠️ Tech Stack

- **Automation:** n8n (self-hosted)
- **AI/LLM:** Google Gemini API (gemini-2.5-flash)
- **Email:** Gmail API (OAuth2)
- **Notifications:** Slack API
- **Logging:** Google Sheets API

## 🔑 Key Learnings

- Field names from trigger nodes (e.g., Gmail's `Subject` vs `subject`, `snippet` vs `text`) must be verified against actual output data, not assumed
- When referencing data from earlier nodes in a multi-branch workflow, `$('Node Name').item.json.field` is essential once you're past nodes that don't carry the original data forward
- AI models can still fabricate specific facts even when told not to in general terms — being explicit and specific in guardrail instructions (naming exact categories to avoid guessing) measurably improved output honesty
- New/unfamiliar sender addresses can trigger Gmail's spam filter on a fresh inbox, which will silently prevent a trigger from firing — always check Spam during testing before assuming a workflow is broken

## 📋 Setup Instructions

1. Import the workflow JSON into your n8n instance
2. Connect your own Gmail, Google Gemini, Slack, and Google Sheets credentials
3. Update the Slack channel name and Google Sheet reference to your own
4. Activate the workflow

## 📌 Status

Fully functional — tested end-to-end for both auto-handle and escalation paths.
