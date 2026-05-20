This n8n workflow turns WhatsApp into a fully automated real estate lead generation system for Indore properties. When a prospect messages you, a GPT-powered AI agent takes over — it detects intent (buy, sell, rent, or rent out), qualifies the lead step-by-step using natural conversation in English or Hinglish, and collects name, location, budget, property type, timeline, and preferred call timing without asking multiple questions at once.
Every qualified lead is automatically saved to the right tab in a Google Sheets spreadsheet (Buy Flat / Sell Flat / Rent out / Rent) with a lead score and stage. If a prospect stops responding, the agent waits and then sends a personalised follow-up nudge per their intent. Ready leads can also trigger a Google Calendar booking for a site visit.

# 🏠 Lead Generation & Follow-Up Agent
### Real Estate WhatsApp AI Bot — Indore, Madhya Pradesh

---

## 📋 Overview

An automated n8n workflow that acts as a human-like WhatsApp broker for real estate leads in Indore. When a user messages on WhatsApp, an AI agent qualifies them step-by-step, scores the lead, saves data to Google Sheets, schedules calendar reminders, and automatically follows up if the user goes silent.

---

## 🏗️ Project Structure

```
Lead_generation_Follow_up_agent (n8n Workflow)
│
├── 📥 ENTRY POINT
│   └── WhatsApp Trigger              ← Listens for incoming WhatsApp messages
│
├── 🔧 DATA EXTRACTION
│   └── Edit Fields                   ← Extracts: Number, Chat Input, Session ID,
│                                        Client Name, lastUserMessageTime
│
├── 🤖 AI BRAIN
│   ├── AI Agent                      ← Core LLM agent with real estate system prompt
│   │   ├── OpenAI Chat Model         ← GPT-based language model (gpt-5-mini)
│   │   ├── Simple Memory             ← Session-based conversation memory (Buffer Window)
│   │   ├── Structured Output Parser  ← Enforces strict JSON output schema
│   │   └── Google Calendar Tool      ← Books "Client Visit" events on Google Calendar
│   │
│   └── If4 (Unknown Intent Check)    ← Routes unknown intent → Send message9 (clarify)
│                                        Known intent → Switch node
│
├── 🔀 INTENT ROUTING
│   └── Switch                        ← Routes by intent:
│       ├── buy      → Send message1  → Date & Time3 → Buy Sheet  → Wait  → ...follow-up
│       ├── sell     → Send message2  → Date & Time2 → Sell Sheet → Wait1 → ...follow-up
│       ├── rent_out → Send message3  → Date & Time  → Rent_out   → Wait2 → ...follow-up
│       ├── rent     → Send message4  → Date & Time1 → Rent       → Wait3 → ...follow-up
│       └── follow_up→ (handled via inactive logic)
│
├── 🗃️ GOOGLE SHEETS (LEADS Spreadsheet)
│   ├── Buy Sheet    → Tab: "Buy Flat"   (appendOrUpdate)
│   ├── Sell Sheet   → Tab: "Sell Flat"  (appendOrUpdate)
│   ├── Rent_out     → Tab: "Rent out"   (appendOrUpdate)
│   └── Rent         → Tab: "Rent"       (append)
│
├── ⏳ WAIT + FOLLOW-UP ENGINE
│   ├── Wait / Wait1 / Wait2 / Wait3    ← 2-minute delay after saving lead
│   │
│   ├── Get row(s) buy / sell / rent_out / rent
│   │   └── Re-reads the saved row to check lastUserMessageTime
│   │
│   ├── If / If1 / If2 / If3           ← Checks: has it been > 60 seconds since last message?
│   │   ├── TRUE  → Send follow-up WhatsApp message
│   │   └── FALSE → No action (user is still active)
│   │
│   └── Follow-up Messages (per intent):
│       ├── Send message  → "still looking to buy?"
│       ├── Send message6 → "still looking to sell?"
│       ├── Send message7 → "still looking to rent out?"
│       └── Send message8 → "still looking for a property?"
│
└── 📤 WHATSAPP REPLIES
    ├── Send message1–4   ← AI-generated reply (per intent from Switch)
    ├── Send message9     ← Clarification reply (unknown intent)
    └── Send message6–8   ← Follow-up nudge messages
```

---

## 🔄 Workflow Flow (Step-by-Step)

```
User sends WhatsApp message
        ↓
WhatsApp Trigger fires
        ↓
Edit Fields → extracts Number, Chat Input, Session ID, Client Name, timestamp
        ↓
AI Agent processes the message
  ├── Memory: recalls prior conversation turns
  ├── LLM: GPT-5-mini generates JSON response
  └── Output Parser: validates and structures JSON
        ↓
If4: Is intent = "unknown"?
  ├── YES → Send clarification reply (Send message9) → END
  └── NO  → Continue to Switch
        ↓
Switch routes by intent (buy / sell / rent_out / rent / follow_up)
        ↓
Send AI reply to WhatsApp (Send message1/2/3/4)
        ↓
Date & Time formats timestamp → Save to Google Sheets tab
        ↓
Wait 2 minutes
        ↓
Get row from Google Sheets (check lastUserMessageTime)
        ↓
If: Has user been silent for > 60 seconds?
  ├── YES → Send follow-up WhatsApp nudge
  └── NO  → Do nothing (conversation still active)
```

---

## 🧠 AI Agent — JSON Output Schema

Every response from the AI Agent is a strict JSON object:

| Field           | Type   | Values / Example                        |
|-----------------|--------|-----------------------------------------|
| `intent`        | string | `buy`, `sell`, `rent`, `rent_out`, `follow_up`, `unknown` |
| `stage`         | string | `new`, `qualifying`, `ready`, `inactive` |
| `reply`         | string | WhatsApp-style message to user          |
| `Number`        | string | Phone number (if provided)              |
| `Name`          | string | Client name (if known)                  |
| `Location`      | string | Area in Indore                          |
| `budget`        | string | e.g. `"80 lakh"`, `"₹15,000/month"`    |
| `property_type` | string | e.g. `"2BHK Flat"`, `"Plot"`           |
| `timeline`      | string | e.g. `"Within 1 month"`                |
| `call_timing`   | string | e.g. `"After 6 PM"`                    |
| `lead_score`    | string | `hot`, `warm`, `cold`                   |

---

## 📊 Google Sheets — LEADS Spreadsheet

**Spreadsheet ID:** `1vyO8xcovoR1I6UvOPBvl1MV3mlQf91P51mStCAmL-pU`

### Tabs & Column Structure

All four tabs share the same 11 columns:

| Column               | Source                          |
|----------------------|---------------------------------|
| `Date`               | Formatted timestamp (dd/MM/yyyy - hh:mm a) |
| `Name`               | WhatsApp contact profile name   |
| `Phone Number`       | WhatsApp sender number          |
| `Intent`             | AI output: buy/sell/rent/rent_out |
| `Lead Score`         | AI output: hot / warm / cold    |
| `Property Type`      | AI output: e.g. 2BHK Flat       |
| `Location`           | AI output: area in Indore       |
| `Budget`             | AI output: price range          |
| `Call Timing`        | AI output: preferred call time  |
| `Status`             | AI output: stage (qualifying/ready) |
| `lastUserMessageTime`| Timestamp of last message       |

**Tabs:**
- `Buy Flat` — leads looking to buy property
- `Sell Flat` — leads looking to sell property
- `Rent out` — leads looking to rent out their property
- `Rent` — leads looking to rent a property

---

## 🔌 Integrations & Credentials Required

| Service           | Credential Type              | Usage                              |
|-------------------|------------------------------|------------------------------------|
| WhatsApp Business | `whatsAppTriggerApi` (OAuth) | Receive incoming messages          |
| WhatsApp Business | `whatsAppApi`                | Send outgoing messages             |
| OpenAI            | `openAiApi` (AI Gateway)     | GPT-5-mini language model          |
| Google Sheets     | `googleSheetsOAuth2Api`      | Read & write lead data             |
| Google Calendar   | `googleCalendarOAuth2Api`    | Book client site visit events      |

**WhatsApp Phone Number ID:** `1113243401872354`
**Google Calendar:** `Client Visit` calendar

---

## ⚙️ Configuration Notes

- **Follow-up timer:** Currently set to `> 60,000ms` (1 minute) for testing. Change to `> 3600000` (1 hour) or `> 86400000` (24 hours) for production use in the `If` / `If1` / `If2` / `If3` nodes.
- **Wait duration:** Currently 2 minutes. Adjust in `Wait`, `Wait1`, `Wait2`, `Wait3` nodes.
- **Language:** Agent replies in English or Hinglish based on user's message language.
- **Scope:** Agent is focused exclusively on Indore, MP properties.
  
---

## 🚀 Setup Checklist

- [ ] Import `Lead_generation_Follow_up_agent.json` into n8n 
- [ ] Connect **WhatsApp Business** credentials (Trigger + Send nodes)
- [ ] Connect **OpenAI** credentials (or configure AI Gateway)
- [ ] Connect **Google Sheets** OAuth account
- [ ] Connect **Google Calendar** OAuth account
- [ ] Create **LEADS** Google Spreadsheet with tabs: `Buy Flat`, `Sell Flat`, `Rent out`, `Rent`
- [ ] Add all 11 columns (see table above) as header row in each tab
- [ ] Update WhatsApp Phone Number ID if different
- [ ] Adjust follow-up timer thresholds for production
- [ ] Activate the workflow
