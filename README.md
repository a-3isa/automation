# automation

n8n workflows I've designed and built: AI agents, API integrations and webhook-driven automations.

| Workflow | File | What it does |
|---|---|---|
| WhatsApp AI Assistant | [`workflows/whatsapp-ai-assistant.json`](workflows/whatsapp-ai-assistant.json) | Multi-agent personal assistant on WhatsApp: email, calendar, web search and X posts from text, voice or image messages |
| AI Marketing Agency (12 agents) | [`workflows/ai-marketing-agency-12-agents.json`](workflows/ai-marketing-agency-12-agents.json) | Turns a client onboarding form into 12 researched marketing deliverables in Google Docs, filed in Drive, tracked in Sheets and emailed to the client |

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

A client fills in a **Tally** onboarding form. Twelve specialist AI agents then research the market and produce a complete marketing package. Each deliverable becomes a Google Doc in the client's own Drive folder, is logged in a Google Sheet, and the client gets an email with links.

**Flow** (98 nodes)

1. **Intake:** Tally webhook → instant acknowledgement → normalise the answers (Code) → create the client's Drive folder → register the client in the `Clients` tab
2. **Research:** 3 Tavily web searches (market, competitors, audience) → compiled into one research pack
3. **12 agents in sequence**, each building on the previous agents' output (Claude Sonnet via OpenRouter, retries on failure):

   | # | Agent | Deliverable | Sheet tab |
   |---|---|---|---|
   | 1 | Intake Analyst | Client brief with SMART goals & assumptions | `Briefs` |
   | 2 | Market Researcher | Market trends & opportunities (cited) | `Market Research` |
   | 3 | Competitor Analyst | Competitor profiles & gap analysis | `Competitors` |
   | 4 | Audience Strategist | 3 buyer personas | `Personas` |
   | 5 | Brand Positioning | Positioning, UVP, messaging pillars, tone rules | `Positioning` |
   | 6 | Marketing Strategist | 90-day funnel plan, channel mix & budget split | `Strategy` |
   | 7 | Content Strategist | Content pillars + 30-day calendar | `Content Calendar` |
   | 8 | Social Copywriter | 12 ready-to-post social posts & video scripts | `Social Posts` |
   | 9 | Paid Ads Specialist | Meta / Google / TikTok structure, ad copy, test plan | `Ad Copy` |
   | 10 | Email Marketer | 5-email welcome & nurture sequence | `Email Sequences` |
   | 11 | SEO & Landing Page | Keyword clusters, blog topics, landing-page copy | `SEO & Landing` |
   | 12 | QA Director | Consistency check + executive summary | `Clients` |

4. **Per deliverable:** create Google Doc → write content (separate node) → move into the client folder → log the row to the agent's tab
5. **Delivery:** email the client (summary + folder links) → notify the agency → mark the client `Delivered`

**Stack:** n8n · Tally · Tavily · OpenRouter (Claude) · Google Docs · Google Drive · Google Sheets · Gmail

**Setup**

- **Google Sheet** with 12 tabs:
  - `Clients`, with columns `client_id, business_name, contact_name, contact_email, website, industry, budget, status, folder_url, summary_doc_url, submitted_at, delivered_at`
  - the 11 agent tabs from the table above, each with columns `client_id, business_name, deliverable, doc_url, preview, created_at`
- **Config node:** set `spreadsheetId`, `parentFolderId` (Drive folder that holds client folders), `agencyName` and `agencyEmail`.
- **Credentials:**
  - Google Docs, Drive, Sheets and Gmail (OAuth2)
  - OpenRouter
  - Tavily as *Header Auth*, with name `Authorization` and value `Bearer tvly-…`
- **Tally:** point the form's webhook at the `/webhook/marketing-intake` URL. Fields are matched by label (business, email, website, industry, country, product/service, audience, goal, budget, competitor, tone).

---

## Importing a workflow

1. In n8n: **Workflows → Import from file** and pick the JSON file.
2. Re-select your own credentials on each node. Credential IDs are replaced with `REPLACE_ME`.
3. Replace the placeholders, such as `YOUR_PHONE_NUMBER_ID`, `YOUR_RECIPIENT_NUMBER`, `YOUR_SHEET_ID`, `YOUR_GOOGLE_SHEET_ID`, `YOUR_CLIENTS_PARENT_FOLDER_ID` and `you@example.com`.

No API keys, tokens or personal data are stored in this repository.
