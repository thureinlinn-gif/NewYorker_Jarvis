# Backend — Agent Instructions

This is the FastAPI orchestration service for **New Yorker Jarvis**. Read [../AGENTS.md](../AGENTS.md) first — universal repo rules and dependencies live there. This file defines backend-specific conventions and architecture rules.

---

## Stack

- **Runtime:** Python 3.12+
- **Framework:** FastAPI + `uvicorn`
- **Settings & Validation:** Pydantic v2 + `pydantic-settings`
- **HTTP Client:** `httpx` (async client for Nessie, Photon, ElevenLabs and Open-Meteo)
- **Vision & Multimodal AI:** `google-genai` official SDK (model set by `GEMINI_MODEL`)
- **Database:** Supabase (`livelens_transit_alerts` and `livelens_sessions` tables). `mongo.py` is an optional fallback and is not wired into the pipeline.
- **Voice Synthesis:** ElevenLabs text-to-speech REST API (`eleven_flash_v2_5`)
- **Financial Infrastructure:** Capital One Nessie API
- **iMessage Delivery:** Photon Spectrum SDK
- **Weather:** Open-Meteo geocoding + forecast APIs
- **Package & Env Manager:** `uv`

---

## Layout

```text
backend/
├── app/
│   ├── main.py          # FastAPI entrypoint, REST routes (/health, /api/tts) & WebSocket route (/ws/stream/{session_id})
│   ├── config.py        # Pydantic Settings — single source of truth for env
│   ├── agents/
│   │   ├── orchestrator.py # Rolling visual memory (deque(maxlen=10)), spatial state machine, tool loop
│   │   └── tools.py        # The 12 Gemini function-calling tools + execute_tool dispatcher
│   ├── services/
│   │   ├── gemini.py           # Gemini multimodal prompt (frame + question + context) & tool calls
│   │   ├── elevenlabs.py       # ElevenLabs text-to-speech
│   │   ├── nessie.py           # Capital One Nessie client (balance, deposit, fare payment)
│   │   ├── photon.py           # Photon iMessage dispatcher
│   │   ├── weather.py          # Open-Meteo hourly forecasts
│   │   ├── supabase_service.py # Supabase client (transit alerts, session checkpoints)
│   │   └── mongo.py            # Optional MongoDB client (unused by default)
│   └── models/
│       └── schemas.py   # Pydantic schemas (frames, events, session state, payloads)
├── pyproject.toml
├── requirements.txt
└── Dockerfile
```

---

## Request Flow

1. The client sends a JSON message over `/ws/stream/{session_id}` with a base64 JPEG frame, optional spoken text, and a language code (`en` or `ru`).
2. **Passive frames** (sent at 1.5 FPS with no text) do **not** call Gemini and produce no spoken reply.
3. **Questions** (with text) call Gemini with the frame, the question, rolling context and the 12 tools. Tool calls run through `execute_tool`.
4. The backend sends `copilot_response` (text + action cards) first, then `audio_chunk` (base64 ElevenLabs MP3).

---

## Dependency Policy

See universal policy in [../AGENTS.md](../AGENTS.md). Backend-specific:

- **Prefer stdlib:** `asyncio`, `pathlib`, `datetime`, `uuid`, `enum`, `collections` (`deque` for the visual memory buffer), `json`, `typing`.
- **Not OK without justification:** Heavy agent abstractions (e.g. LangChain, CrewAI, AutoGen). Direct Gemini tool declarations are faster and more deterministic.

---

## Configuration

- `app.config.settings` is the **single source of truth**. Import `settings` where needed; never call `os.getenv` in application logic, never call `load_dotenv`.
- Nessie and Photon return simulated responses when their API keys are missing, so the demo runs without every key.
- Default sensible non-secret values for local development (e.g., `PORT=8000`, `ENVIRONMENT="development"`).

---

## Concurrency Rules

1. **Async-First:** Every route handler and service I/O method must be `async def`. Never run blocking network I/O on the main event loop.
2. **Answer before audio:** Send the text response and action cards before waiting on ElevenLabs.
3. **Always answer a spoken question:** If Gemini stalls or errors, send a short fallback reply so the HUD never gets stuck. Gemini calls are bounded by `GEMINI_TIMEOUT_SECONDS`.
4. **Rolling Visual Memory:** Keep context in the in-memory `deque(maxlen=10)` in `orchestrator.py`. Never write raw frames to disk.

---

## Anti-Patterns (Rejected)

- Calling `os.getenv` or `load_dotenv` inside service modules.
- Introducing generic agent frameworks that add overhead.
- Catching bare `Exception` to print and re-raise without context.
- Storing raw camera frames permanently to disk.
- Silent fallbacks when third-party APIs fail; log explicitly.
