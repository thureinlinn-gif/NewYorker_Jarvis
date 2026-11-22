# New Yorker Jarvis Implementation TODO & Milestone Plan

Based on [docs/architecture.md](../architecture.md), [docs/project-brief.md](../project-brief.md), and [AGENTS.md](../../AGENTS.md), this document tracks the roadmap to build, polish, and validate New Yorker Jarvis for live hackathon judging.

---

## 🎯 High-Level Objective
Deliver an end-to-end working demonstration on stage:
1. **Act 1:** Camera pointed at a subway entrance + spoken question (English or Russian) → Gemini identifies the entrance and plans the route → ElevenLabs speaks the answer.
2. **Act 2:** Questions about delays and weather → `lookup_transit_alerts` (Supabase) and `get_weather_forecast` (Open-Meteo).
3. **Act 3:** Balance check and fare payment through Capital One Nessie → Photon sends the itinerary as an iMessage.

---

## 📋 Phase 0: Scaffolding & Health Validation (Status: ✅ Complete)
- [x] Repository layout with root `AGENTS.md`, `backend/AGENTS.md`, and `frontend/AGENTS.md`
- [x] Backend FastAPI service with `uv` virtual environment and dependency setup
- [x] Frontend React + Vite + Tailwind setup with mobile viewport support
- [x] Single source of truth configs: `backend/app/config.py` & `frontend/src/config/env.ts`
- [x] Dockerfile & `docker-compose.yml`
- [x] Basic connectivity test: `/api/hello` + frontend `/health` page

---

## 📋 Phase 1: Camera & WebSocket Pipeline (Status: ✅ Complete)
- [x] **1.1 Camera Capture & Frame Sampler:**
  - [x] Back camera (`environment`) with fallback for laptops
  - [x] Off-screen `<canvas>` frame capture, 640px wide JPEG
  - [x] Live 1.5 FPS sampling mode vs. single frame per question
- [x] **1.2 Voice Input & Audio Playback:**
  - [x] Web Speech API recognition in English and Russian, switched by voice
  - [x] Web Audio playback of ElevenLabs MP3 (`audio_chunk`)
  - [x] One question in flight at a time, 15-second turn timeout
- [x] **1.3 WebSocket Route (`/ws/stream/{session_id}`):**
  - [x] Event payloads: `frame`, `copilot_response`, `audio_chunk`, `ping` / `pong`
  - [x] Automatic 2-second reconnect

---

## 📋 Phase 2: Multimodal Perception & Visual State (Status: ✅ Complete)
- [x] **2.1 Rolling Visual Memory:**
  - [x] `TransitLocationState` machine in `backend/app/agents/orchestrator.py` (`STREET_ENTRANCE` → `MEZZANINE` → `TURNSTILE` → `PLATFORM` → `TRAIN_BOARDED`)
  - [x] In-memory `deque(maxlen=10)` preserving recent context
  - [x] Spatial state badge in the frontend HUD
- [x] **2.2 Gemini Prompt & Tool System:**
  - [x] Frame + spoken question + rolling context per request; passive frames skip Gemini
  - [x] 12 Python function tools registered with Gemini (`TOOL_FUNCTIONS`)
  - [x] Timeout and fallback reply so every spoken question gets an answer

---

## 📋 Phase 3: Tool Integrations (Status: ✅ Complete)
- [x] **3.1 Capital One Nessie:** `check_bank_balance`, `deposit_money`, `pay_with_card` (simulated without key)
- [x] **3.2 Photon iMessage:** `dispatch_imessage_itinerary` (simulated without key)
- [x] **3.3 Supabase:** `lookup_transit_alerts` + session checkpoints
- [x] **3.4 Weather:** `get_weather_forecast` via Open-Meteo
- [x] **3.5 Routing & Map:** `set_navigation_route`, `close_navigation_map` with Leaflet, Transitous and OSRM
- [x] **3.6 Trip Plan HUD:** `update_mission_plan`, `clear_mission_plan`, `hide_mission_plan`, `show_mission_plan`
- [x] **3.7 ElevenLabs:** text-to-speech after each answer (`eleven_flash_v2_5`)

---

## 📋 Phase 4: Stage Demo Polish & Mobile Tuning
- [ ] **4.1 Mobile HTTPS & Network Setup:**
  - [ ] `ngrok` or Cloudflare tunnel for physical phone camera
  - [ ] First-gesture audio unlock for iOS Safari `AudioContext`
- [ ] **4.2 HUD Polish:**
  - [ ] Action cards for Nessie payments and Photon iMessage
  - [ ] Latency and connection readout in the status bar
- [ ] **4.3 Rehearse the 3-Minute Demo Script** ([project-brief.md](../project-brief.md))

---

## 📋 Phase 5: Deployment & MLH Prizes
- [ ] Deploy backend container to **DigitalOcean**
- [ ] Link domain **`livelens.tech`** (MLH Best .Tech Domain)
- [ ] Record a backup 60-second video demo in case of venue Wi-Fi failure
