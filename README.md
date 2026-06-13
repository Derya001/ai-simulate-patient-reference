# ai-simulate-patient

An AI-powered simulated psychiatric patient for medical education. Medical students and residents can practice psychiatric interview techniques, crisis intervention, and suicide risk assessment with an adaptive, responsive virtual patient.

Developer: Judit Bódis  
Clinical partner: Dr. Tamás Treuer, University of Pécs (PTE)

---

## What Is This?

The system implements a simulated patient built on a psychopathological state machine that responds to the learner's communication style. If the learner asks empathic, structured questions, the patient opens up. If the learner is impatient or avoids critical questions, the patient withdraws — exactly as in a real psychiatric interview.

This is not a simple chatbot. It is a guided educational system with clinician-validated case profiles, hidden information, and trigger-based state transitions.

---

## Stack

| Layer | Technology |
|---|---|
| LLM | Claude API (claude-sonnet-4-5) |
| Backend | Node.js + Express |
| Frontend | HTML/CSS/JS (split-screen) |
| Avatar | D-ID Streaming API (WebRTC) |
| Voice | Microsoft Neural TTS (hu-HU-NoemiNeural) |
| Hosting | Railway |
| Environment | WSL2 + VS Code |

---

## Architecture

```
learner message
       ↓
style evaluation AI call (empathic / directive / avoidant / confrontational)
       ↓
state machine update (trust, anxiety, suicidal accessibility, etc.)
       ↓
hidden information reveal check (separate AI call, parallel)
       ↓
patient response generation (Claude API, based on current state)
       ↓
text → D-ID Streaming Avatar (real-time WebRTC)
```

---

## Scenarios

| # | Case | Difficulty | Status |
|---|---|---|---|
| 1 | Ambivalent depressed patient in acute suicidal crisis | Beginner | 🔄 In progress |
| 2 | Young adult after impulsive self-harm | Intermediate | ⏳ Planned |
| 3 | Paranoid/psychotic patient — risk assessment | Intermediate | ⏳ Planned |
| 4 | Agitated, distrustful patient — verbal de-escalation | Advanced | ⏳ Planned |
| 5 | Post-crisis follow-up interview | Intermediate | ⏳ Planned |

---

## Project Status

| Component | Status |
|---|---|
| Project structure, Node.js backend | ✅ Done |
| System prompt — RP-CRISIS-001 (K., 32y female) | ✅ Done |
| Claude API integration, /chat endpoint | ✅ Done |
| Psychopathological state machine (8 variables) | ✅ Done |
| Communication style evaluator (4 categories) | ✅ Done |
| Hidden information reveal logic (3 triggers) | ✅ Done |
| Split-screen frontend (chat + avatar panel) | ✅ Done |
| D-ID Streaming Avatar integration (WebRTC) | ✅ Done |
| Hungarian voice (hu-HU-NoemiNeural) | ✅ Done |
| Patient greets on load (SYSTEM_INIT) | ✅ Done |
| Nonverbal cue filtering (strip asterisks before TTS) | ✅ Done |
| Password-protected access (Railway middleware) | ✅ Done |
| Loading state indicators (Hallgatom / Feldolgozás / A beteg válaszol) | ✅ Done |
| Microphone push-to-talk (Web Speech API, hu-HU) | ✅ Done |
| Railway deployment | ✅ Live |
| Clinical validation by Treuer — case 1 | ⏳ Planned |
| PTE demo | ⏳ June 2026 |
| Feedback module | ⏳ Autumn 2026 |
| Multi-scenario case bank | ⏳ Autumn 2026 |
| Conference presentation | ⏳ January 2027 |

---

## State Variables (0–10 scale)

The patient's internal state is described by 8 variables, updated after each exchange:

| Variable | Meaning |
|---|---|
| trust | How safe the patient feels in the conversation |
| anxiety | General tension level |
| shame | How difficult it is to reveal the problem |
| hopelessness | Sense of futility and no future |
| cooperation | How meaningfully the patient responds |
| suicidal_accessibility | How openly the patient discusses suicidal thoughts |
| irritability | Readiness for confrontation |
| crisis_intensity | Overall level of acute danger |

---

## Hidden Information

Three pieces of information the patient only reveals when the right question is asked:

| ID | Content | Trigger |
|---|---|---|
| hi_001 | Collected medication at home, checked quantities | Learner asks about plan, preparation, or access to means |
| hi_002 | Has not told her sister how bad things are | Learner asks about support system or people to involve |
| hi_003 | Afraid of being seen as crazy, ashamed of suicidal thoughts | Learner asks in a normalizing, non-stigmatizing way |

---

## Deployment (Railway)

The application is hosted on Railway at:

```
https://ai-simulate-patient-production.up.railway.app
```

### Environment Variables

Set the following in Railway → Project → Variables:

```
ANTHROPIC_API_KEY=your_key_here
DID_API_KEY=your_key_here
DEMO_PASSWORD=your_password_here
```

### Deploy

Railway deploys automatically on every `git push` to the connected GitHub repository. No manual steps required after initial setup.

The start command is defined in `package.json`:

```json
"scripts": {
  "start": "node server.js"
}
```

---

## Access Control

The application is in closed testing phase and protected by a server-side password middleware.

### How it works

- All routes are blocked by default
- `GET /login.html` and `POST /auth` are public
- On login, the password is sent to `/auth`; if correct, it is stored in `sessionStorage`
- Every subsequent request includes the password as an `x-demo-token` header
- The middleware validates the header on the server side — access cannot be bypassed client-side

### Granting access

Share the Railway URL and the demo password directly with authorized users (PTE faculty, test participants). Do not publish the password publicly.

To change the password, update the `DEMO_PASSWORD` environment variable in Railway. The change takes effect on the next deploy.

---

## zonitai.com Integration

The project is showcased at:

```
https://zonitai.com/front-page/ai-szimulalt-beteg/
```

The project page describes the system architecture, state variables, and stack. Access to the live demo is provided via a button linking to the Railway URL. The demo button is only shown to users who have received the access code directly.

---

## Running Locally

```bash
# Install dependencies
npm install

# Start backend
node server.js

# Open in browser
http://localhost:3000
```

Required in `.env`:

```
ANTHROPIC_API_KEY=your_key_here
DID_API_KEY=your_key_here
DEMO_PASSWORD=your_password_here
```

---

## Why Claude, Not OpenAI?

Claude maintains role consistency throughout long psychiatric simulations. GPT-4o tends to break character in extended conversations — a critical failure in an educational context.

---

## Avatar Technology Notes

### HeyGen → D-ID migration

Initial implementation used HeyGen's video generation API with a talking_photo avatar (Lori). The approach worked technically but video generation latency was 1-2 minutes per response — unacceptable for real-time simulation.

HeyGen's LiveAvatar (Streaming Avatar) API would solve this, but requires a $99/month Business plan. For a prototype and demo context, this cost is not justified.

**Decision:** Switch to D-ID Streaming API, which provides real-time WebRTC-based avatar at ~$5.90/month entry point. Quality is lower than HeyGen but sufficient for demo and prototype purposes.

If the project receives institutional funding post-conference, HeyGen LiveAvatar is the recommended upgrade path for production.

| Tool | Used for | Status |
|---|---|---|
| HeyGen Video API | Initial avatar video generation | ⛔ Too slow (1-2 min/response) |
| HeyGen LiveAvatar | Real-time streaming avatar | ⛔ $99/month — out of budget |
| D-ID Streaming API | Real-time WebRTC avatar | ✅ Current implementation |

---

## Clinical Background

The scenarios are built on the psychiatric education program of the University of Pécs. Case profiles, state variables, and trigger rules are based on a clinician-validated structure (Treuer et al., PTE).

The system is not a clinical decision-making tool — it is built exclusively for educational simulation purposes.
