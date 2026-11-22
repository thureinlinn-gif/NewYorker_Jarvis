# New Yorker Jarvis 👁️🎙️

> **Hands-free, voice-and-camera AI transit copilot for NYC that answers spoken questions about what you're looking at, in English or Russian, and takes action through 12 Gemini function-calling tools.**

New Yorker Jarvis turns multimodal AI into a hands-free urban navigator. The phone camera samples the scene at 1.5 FPS, and every spoken question is sent to Gemini together with a live camera frame over WebSockets. Gemini answers out loud and can call tools to check a bank balance, pay a fare, plan a route, look up transit alerts, check the weather, or text an itinerary to iMessage. It's built for immigrants, tourists, and visually impaired riders who struggle with app fatigue and language barriers.

---

## 🏆 Hackathon Tracks & Sponsor Alignment

| Track / Sponsor | Prize Category | How New Yorker Jarvis Integrates |
| :--- | :--- | :--- |
| **Capital One** | **Best Use of Nessie** | **Agentic transit payments:** checks account balances, makes deposits, and pays turnstile fares through the Nessie API via Gemini tool calls. |
| **Google Cloud / MLH** | **Best Use of Gemini API** | Multimodal understanding of a live camera frame plus the user's spoken question, with 12 function-calling tools for actions. |
| **ElevenLabs [MLH]** | **Best Use of ElevenLabs** | Text-to-speech (`eleven_flash_v2_5`) that reads Jarvis's answers aloud in the user's language. |
| **Photon** | **Agents in iMessage using Photon** | Sends formatted route guides and transit warnings to the user's native Messages app using the **Photon Spectrum SDK**. |
| **DigitalOcean [MLH]** | **Best Use of DigitalOcean** | Hosting of the FastAPI backend and WebSocket orchestrator. |
| **.Tech Domains [MLH]** | **Best .Tech Domain Name** | `livelens.tech` domain for the web app and API. |
| **Cursor** | **Best Use of Cursor** | Built end to end in Cursor: the WebSocket pipeline, Gemini tool definitions, and the React HUD, with on-device debugging of iPad audio routing. |
| **SpaceX** | **Best Real-Time Systems Engineering** | Continuous camera sampling, a rolling visual state machine, and graceful degradation (timeouts and fallbacks) when an upstream service stalls. |

---

## 🎯 The Problem & Value Proposition

- **App Fatigue in the Wild:** Transit apps require constant typing, reading, and switching between mapping, translation, and transit agency apps.
- **Multimodal Gap:** Standard LLM wrappers handle single photo uploads with no memory. They don't keep a **rolling visual context** across questions.
- **Accessibility & Language Barriers:** Non-English speakers and visually impaired travelers face confusing printed detour notices and complex transit hubs every day.

**New Yorker Jarvis bridges this gap:** point your camera, ask a question in English or Russian, hear the answer, and get the itinerary texted to your iMessage.

---

## ⚡ Demo Workflow & System Execution

| Stage | User Action | System Perception & Tool Execution | Output & User Experience |
| :--- | :--- | :--- | :--- |
| **1. Ask by Voice** | User points the phone at a subway entrance and asks, in English or Russian, how to get to Columbia University. | The question and the current camera frame go over a WebSocket to FastAPI, which calls **Gemini** to identify the entrance and the user's intent. | **ElevenLabs** reads the answer aloud in the user's language, and `set_navigation_route` shows the route on the map. |
| **2. Check Conditions** | User asks about delays on the 1 train or the weather. | Gemini calls `lookup_transit_alerts` (stored in **Supabase**) or `get_weather_forecast` (Open-Meteo). | Jarvis speaks the alerts or forecast and shows an action card. |
| **3. Pay & Send** | User asks to pay the fare, then: *"Text me the full transfer guide."* | Gemini calls `pay_with_card` (**Nessie**), then `dispatch_imessage_itinerary` (**Photon**). | The fare payment is confirmed, and a formatted **iMessage** arrives on the user's phone. |

---

## 🧰 Gemini Function-Calling Tools (12)

Defined in [`backend/app/agents/tools.py`](backend/app/agents/tools.py) and passed to Gemini as `TOOL_FUNCTIONS`.

| Category | Tools |
| :--- | :--- |
| **Payments (Capital One Nessie)** | `check_bank_balance`, `deposit_money`, `pay_with_card` |
| **Routing & Map** | `set_navigation_route`, `close_navigation_map` |
| **Transit Alerts** | `lookup_transit_alerts` |
| **Weather (Open-Meteo)** | `get_weather_forecast` |
| **Messaging (Photon iMessage)** | `dispatch_imessage_itinerary` |
| **Trip Plan HUD** | `update_mission_plan`, `clear_mission_plan`, `hide_mission_plan`, `show_mission_plan` |

> Without API keys, the Nessie and Photon services return simulated responses, so the demo still runs.

---

## 🏗️ Architecture

```text
                         ┌──────────────────────────────────────────────┐
                         │        React + Vite + TypeScript Frontend    │
                         │   • Camera frame sampler (Canvas, 1.5 FPS)   │
                         │   • Speech recognition (English / Russian)   │
                         │   • Audio player, map & action cards         │
                         └──────────────────────┬───────────────────────┘
                                                │
                                       WebSockets / REST
                                                │
                                                ▼
                         ┌──────────────────────────────────────────────┐
                         │          FastAPI Orchestrator                │
                         │   • Rolling visual context & state machine   │
                         │   • Gemini prompt + 12-tool dispatch         │
                         └──────┬──────────────┬──────────────┬─────────┘
                                │              │              │
    ┌───────────────────────────┴───┐    ┌─────┴──────────┐   └───────────────────────────┐
    ▼                               ▼    ▼                ▼                               ▼
Gemini API                     ElevenLabs  Supabase         Photon SDK              Nessie API /
Vision + Function Calling      TTS Voice   Transit Alerts   iMessage Dispatch       Open-Meteo
                                           & Sessions                               Weather
```

---

## 🛠️ Stack

| Layer | Choice | Purpose |
| :--- | :--- | :--- |
| **Frontend** | React (Vite) + TypeScript + Tailwind CSS | Mobile camera view, speech input, audio playback, map, and WebSocket client |
| **Backend** | Python 3.12+ & FastAPI | Async orchestrator handling frame ingestion, Gemini calls, and tool dispatch |
| **Vision & Reasoning** | Google Gemini API (model set by `GEMINI_MODEL`) | Multimodal understanding of camera frames and function calling |
| **Speech Input** | Web Speech API (browser) | Voice recognition in English (`en-US`) and Russian (`ru-RU`), switched by voice |
| **Voice Output** | ElevenLabs | Text-to-speech for spoken answers |
| **Financial Agent** | Capital One Nessie API | Balance checks, deposits, and turnstile fare payments |
| **Messaging Agent** | Photon Spectrum SDK | Route guides sent to native iOS iMessage |
| **Database** | Supabase | Transit alerts and session checkpoints |
| **Weather** | Open-Meteo API | Hourly forecasts by location |
| **Maps & Routing** | Leaflet + OpenStreetMap, Transitous, OSRM | Map display, transit routes, and driving routes |
| **Cloud Hosting** | DigitalOcean | Deployment of the FastAPI container and React build |
| **Domain** | MLH .Tech | `livelens.tech` |

---

## 📂 Repo Layout

```text
NewYorker_Jarvis/
├── README.md               # Project documentation & hackathon submission brief
├── docker-compose.yml      # Local orchestration for dev/testing
├── backend/                # FastAPI orchestrator & agent service
│   ├── app/
│   │   ├── main.py         # FastAPI entry & WebSocket route (/ws/stream/{session_id})
│   │   ├── config.py       # Pydantic environment settings
│   │   ├── agents/
│   │   │   ├── orchestrator.py # Rolling visual context, state machine & tool loop
│   │   │   └── tools.py        # The 12 Gemini function-calling tools
│   │   ├── services/
│   │   │   ├── gemini.py           # Gemini API client (vision + function calling)
│   │   │   ├── elevenlabs.py       # ElevenLabs text-to-speech
│   │   │   ├── nessie.py           # Capital One Nessie client (balance, deposit, fare)
│   │   │   ├── photon.py           # Photon iMessage dispatcher
│   │   │   ├── weather.py          # Open-Meteo weather forecasts
│   │   │   ├── supabase_service.py # Supabase client (transit alerts, sessions)
│   │   │   └── mongo.py            # Optional MongoDB client (not used by default)
│   │   └── models/
│   │       └── schemas.py      # Pydantic schemas (frames, session, messages)
│   ├── pyproject.toml      # Python dependencies (managed via uv)
│   └── Dockerfile
└── frontend/               # React + Vite mobile-first web app
    ├── src/
    │   ├── components/
    │   │   ├── CameraView.tsx    # Live video capture
    │   │   ├── MapView.tsx       # Navigation map
    │   │   ├── PlanDashboard.tsx # Trip plan HUD
    │   │   ├── ActionCard.tsx    # Tool result cards (payments, iMessage, weather)
    │   │   ├── Transcript.tsx    # Live subtitle overlay
    │   │   └── JarvisOrb.tsx     # Speaking/listening indicator
    │   ├── hooks/
    │   │   ├── useLiveLens.ts    # WebSocket connection & 1.5 FPS frame sampling
    │   │   ├── useCamera.ts      # Camera access
    │   │   └── useAudioPlayer.ts # Audio playback
    │   ├── config/env.ts         # Frontend configuration
    │   ├── App.tsx               # Main layout, speech input & controls
    │   └── main.tsx
    ├── package.json
    ├── vite.config.ts
    └── tailwind.config.js
```

---

## 📋 Prerequisites

Install the following tools before running locally:

| Tool | Recommended Version | Used For | Installation |
| :--- | :--- | :--- | :--- |
| **Python** | `3.12+` | Backend runtime | [python.org](https://www.python.org/) or OS package manager |
| **uv** | Latest | Fast Python package management | `curl -LsSf https://astral.sh/uv/install.sh \| sh` |
| **Node.js** | `20+` (LTS) | Frontend runtime | [nodejs.org](https://nodejs.org/) or `nvm` |
| **pnpm** or **npm** | Latest | Frontend package manager | `corepack enable` or `npm install -g pnpm` |

### Required API Keys

Create a `.env` file in `backend/` containing:

```env
# Gemini API (Google AI Studio)
GEMINI_API_KEY=your_gemini_api_key
GEMINI_MODEL=gemini-2.0-flash

# ElevenLabs
ELEVENLABS_API_KEY=your_elevenlabs_api_key
ELEVENLABS_VOICE_ID=21m00Tcm4TlvDq8ikWAM # or preferred multilingual voice

# Capital One Nessie API
NESSIE_API_KEY=your_capital_one_nessie_api_key
NESSIE_CUSTOMER_ID=your_nessie_customer_id

# Photon Spectrum (iMessage Agent)
PHOTON_API_KEY=your_photon_api_key

# Supabase
SUPABASE_URL=your_supabase_project_url
SUPABASE_SERVICE_ROLE_KEY=your_supabase_service_role_key

# Environment
ENVIRONMENT=development
PORT=8000
```

---

## 🚀 Running Locally

Detailed setup guides:
- [Backend Setup Guide](docs/guides/backend-setup.md) — Python 3.12, `uv`, WebSockets, and Docker
- [Frontend Setup Guide](docs/guides/frontend-setup.md) — React, Vite, camera streaming, and mobile HTTPS testing
- [Sponsor APIs & Credentials Guide](docs/guides/sponsor-integrations-setup.md) — Gemini, ElevenLabs, Nessie, Photon, and DigitalOcean
- [Full Technical Architecture](docs/architecture.md) — System design and sequence diagram
- [Product Brief & Demo Script](docs/project-brief.md) — 3-minute live judging script

### 1. Backend Setup (FastAPI)

```bash
cd backend

# Create virtual environment and install dependencies with uv
uv venv
source .venv/bin/activate  # On Windows: .venv\Scripts\activate
uv pip install -r requirements.txt  # Or: uv sync

# Start development server
uvicorn app.main:app --host 0.0.0.0 --port 8000 --reload
```

FastAPI server runs at `http://localhost:8000` with interactive Swagger docs at `http://localhost:8000/docs`.

### 2. Frontend Setup (React + Vite)

```bash
cd frontend

# Install dependencies
pnpm install # or npm install

# Start Vite dev server with HTTPS/tunnel for mobile camera testing
pnpm dev
```

Frontend runs at `http://localhost:5173`.

> **Tip for Mobile Testing on Stage:** Use [ngrok](https://ngrok.com/) or Cloudflare Tunnels (`cloudflared tunnel`) to test camera permissions directly from an iPhone or Android phone on the same network!

---

## 🌟 Judge Rubric Highlights

- **Live Stage "Wow" Factor:** No static slides or mockups. A working live demo with a phone camera feed, spoken questions in English or Russian, and spoken answers.
- **Real-World Accessibility Impact:** Immediate utility for immigrants, tourists, and visually impaired people navigating unfamiliar transit networks with confusing paper signage.
- **Deep Sponsor Cohesion:** Every sponsor technology has a role in the pipeline (Gemini = eyes & brain, ElevenLabs = voice, Nessie = payments, Photon = iMessage actions, DigitalOcean = hosting).

---

## 👥 Team

Built by **Thu Rein Linn** and **Abdumalik** using pair programming: one person wrote the code (driver) while the other reviewed it and planned the next steps (navigator), and we swapped roles every hour.
