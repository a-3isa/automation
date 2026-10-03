# automation

n8n workflows I've designed and built: AI agents, API integrations and webhook-driven automations.

| Workflow | File | What it does |
|---|---|---|
| WhatsApp AI Assistant | [`workflows/whatsapp-ai-assistant.json`](workflows/whatsapp-ai-assistant.json) | Multi-agent personal assistant on WhatsApp: email, calendar, web search and X posts from text, voice or image messages |
| AI Marketing Agency (12 agents) | `workflows/ai-marketing-agency-12-agents.json` *(coming soon)* | 12-agent, 126-node system that turns a client onboarding form into researched marketing deliverables in Google Docs, Drive and Sheets |

---

## WhatsApp AI Assistant

One WhatsApp chat that takes **text, voice notes or photos** and actually does things, replying in the same format (voice in → voice out).

**Flow**

1. **WhatsApp Trigger** (webhook) → **Switch** on message type
2. **Normalise the input to text**
   - Text: used as-is
   - Voice: get media URL → HTTP download with Bearer auth → OpenAI Whisper transcription
   - Image: get media URL → HTTP download → GPT-4o-mini vision description + caption
3. **Orchestrator Agent** (GPT-4.1-mini) with a Think tool and Simple Memory keyed on the sender's phone number
4. **Sub-agents used as tools**, each with its own model and only the integrations it needs:
   - **E-mail Agent**: Gmail send, reply, list, mark read/unread, delete
   - **Calendar Agent**: Google Calendar create, get, list, update, delete
   - **Search Agent**: SerpAPI (Google), Wikipedia, Hacker News
   - **Social Media Agent**: writes X posts and logs them to Google Sheets
5. **If** the original message was audio → OpenAI TTS (Opus) → WhatsApp audio reply; otherwise → WhatsApp text reply

**Stack:** n8n · WhatsApp Cloud API · OpenAI (GPT-4.1-mini, GPT-4o-mini, Whisper, TTS) · Gmail · Google Calendar · Google Sheets · SerpAPI

---

## AI Marketing Agency (12 agents)

A 12-agent, 126-node n8n workflow: a Tally onboarding form triggers market research (Tavily), and a chain of specialist agents (Claude 3.5 Sonnet via OpenRouter) produces deliverables written to Google Docs, filed in per-client Google Drive folders, tracked in a 12-tab Google Sheet and delivered by Gmail.

---

## Importing a workflow

1. In n8n: **Workflows → Import from file** and pick the JSON file.
2. Re-select your own credentials on each node. Credential IDs are replaced with `REPLACE_ME`.
3. Replace the placeholders: `YOUR_PHONE_NUMBER_ID`, `YOUR_RECIPIENT_NUMBER`, `YOUR_SHEET_ID`, `you@example.com`.

No API keys, tokens or personal data are stored in this repository.
