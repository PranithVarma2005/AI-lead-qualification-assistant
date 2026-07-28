# AI Lead Qualification Assistant

An AI-powered n8n automation that interacts with potential customers via Telegram, qualifies them as sales leads, and automatically triggers downstream business actions (Google Sheets logging, sales team notification via Gmail, and confirmation messaging).

Built for the **AI & Automation Engineer Intern** technical assignment — **Option 2: AI Lead Qualification Assistant**.

---

## Problem Statement

Build an AI assistant that interacts with potential customers and qualifies leads for a business. The solution should:

- Collect customer information through natural conversation
- Understand customer requirements
- Ask relevant follow-up questions
- Determine whether the lead is qualified
- Extract structured information: **Name, Contact Information, Business Requirement, Budget, and Timeline**
- Trigger business actions: saving the lead to Google Sheets, notifying the sales team, and sending a confirmation message

---

## Workflow Overview

The workflow is built entirely in **n8n** and is triggered by incoming Telegram messages. Here's how each node contributes to the pipeline:

| Node | Purpose |
|---|---|
| **Telegram Trigger** | Listens for incoming messages from users on Telegram |
| **Normalize Input** | Extracts and structures raw Telegram payload data (chat ID, user message, username) into a clean, consistent format for downstream nodes |
| **AI Sales Conversation Agent** | Powered by OpenAI (via LangChain agent node), this holds a natural back-and-forth conversation with the lead, asking follow-up questions until all required fields are gathered. Uses a **Conversation Memory** buffer (per-chat) so context persists across messages |
| **AI Extraction Agent** | A second AI agent that parses the conversation and extracts structured fields — Name, Contact, Requirement, Budget, Timeline — using a **Structured Output Parser** to guarantee consistent JSON output |
| **All Fields Collected?** | A conditional (IF) node that checks whether all 5 required fields have been captured. If not, loops back with a follow-up question |
| **Send Follow-up Question (Telegram)** | Sends the AI's next clarifying question back to the user if information is still missing |
| **Is Lead Qualified?** | Once all fields are collected, this conditional evaluates the lead against qualification logic (e.g. budget threshold, clear requirement) |
| **Save Lead to Google Sheets** | Appends qualified lead data as a new row in a Google Sheet |
| **Notify Sales Team (Gmail)** | Sends an automated email notification to the sales team with the new qualified lead's details |
| **Send Confirmation (Telegram)** | Sends a personalized thank-you/confirmation message back to the user, summarizing their submitted requirement |
| **Save Unqualified Lead to Sheets** | If the lead doesn't meet qualification criteria, logs it separately for tracking |
| **Send Polite Close (Telegram)** | Sends a polite closing message to unqualified leads |

### Flow Diagram (Logical)

```
Telegram Trigger
    → Normalize Input
    → AI Sales Conversation Agent (with Memory)
    → AI Extraction Agent (Structured Output Parser)
    → All Fields Collected?
        ├─ No  → Send Follow-up Question (Telegram) → [loop back via Telegram Trigger]
        └─ Yes → Is Lead Qualified?
                    ├─ Yes → Save Lead to Google Sheets → Notify Sales Team (Gmail) → Send Confirmation (Telegram)
                    └─ No  → Save Unqualified Lead to Sheets → Send Polite Close (Telegram)
```

---

## AI Model Used

- **Provider:** OpenAI
- **Model:** `gpt-4o-mini`
- **Sampling Temperature:** 0.4 (balances natural conversational tone with consistent, reliable extraction)
- **Framework:** n8n's LangChain-based AI Agent nodes, with a Structured Output Parser to enforce consistent JSON schema for extracted lead data

Two separate AI agent instances are used:
1. **Conversation Agent** — handles the natural back-and-forth dialogue with the lead, maintaining per-chat memory
2. **Extraction Agent** — parses the conversation and outputs structured, validated fields (Name, Contact, Requirement, Budget, Timeline)

---

## External Integrations

| Service | Purpose |
|---|---|
| **Telegram** | Primary chat interface for interacting with leads (trigger + message sending) |
| **Google Sheets** | Persistent storage for qualified and unqualified lead records |
| **Gmail** | Automated email notification to the sales team when a new qualified lead is captured |

---

## Setup Instructions

### Prerequisites
- An [n8n](https://n8n.io) instance (cloud or self-hosted)
- A Telegram bot (created via [@BotFather](https://t.me/BotFather))
- An OpenAI API key
- Google account with access to Sheets and Gmail APIs (via n8n's Google OAuth credentials)

### Steps

1. **Import the workflow**
   - In n8n, go to **Workflows → Import from File**
   - Select `workflow.json` from this repository

2. **Configure credentials**
   - **Telegram:** Add your bot token under Telegram credentials in n8n
   - **OpenAI:** Add your OpenAI API key under OpenAI credentials
   - **Google Sheets & Gmail:** Connect your Google account via OAuth2 credentials in n8n

3. **Set up your Google Sheet**
   - Create a Google Sheet with columns matching: `Name, Contact, Requirement, Budget, Timeline`
   - Update the "Save Lead to Google Sheets" and "Save Unqualified Lead to Sheets" nodes with your Sheet ID

4. **Update node references**
   - Ensure the Telegram Chat ID field in message-sending nodes references `{{ $json.message.chat.id }}` (or the appropriate upstream node) correctly

5. **Activate/Publish the workflow**
   - Click **Publish** in n8n to make the Telegram webhook live

6. **Test**
   - Message your Telegram bot to start a conversation and verify the full flow: conversation → extraction → qualification → Sheets/Gmail/confirmation

See `.env.example` for a list of required credentials/environment references.

---

## Screenshots

See the `screenshots/` folder for:
- Full workflow canvas in the n8n editor
- A successful Telegram conversation (start to confirmation)
- Google Sheets showing saved lead data
- Gmail notification received by the sales team

---

## Demo Video

[Watch the demo vedio here] — *(https://drive.google.com/file/d/18XZJU9nsaiow0h-i7rzLRSU2k0PShM2d/view?usp=sharing)*

---

## Future Improvements

- **Voice interaction:** Accept and respond to voice messages via Telegram's voice API + speech-to-text
- **Multi-language support:** Detect and respond in the user's language automatically
- **Human handoff:** Allow a live sales rep to take over the conversation seamlessly when needed
- **Error handling & retry logic:** Add retry mechanisms for failed API calls (e.g. Sheets/Gmail write failures) with fallback notifications
- **CRM integration:** Replace/augment Google Sheets with a proper CRM (e.g. HubSpot, Airtable) for richer lead management
- **Calendar integration:** Allow qualified leads to book a call directly via Google Calendar/Calendly
- **Analytics dashboard:** Track conversion rates, common drop-off points, and lead quality trends over time

---

## Tech Stack

- **Automation Platform:** n8n
- **AI Model:** OpenAI GPT-4o-mini
- **Messaging:** Telegram Bot API
- **Storage:** Google Sheets
- **Notifications:** Gmail
