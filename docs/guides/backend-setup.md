# Backend Setup Guide

This guide walks through setting up and running the **New Yorker Jarvis** FastAPI orchestrator backend locally, with virtual environments, WebSockets, and environment configuration.

---

## 📋 Prerequisites

Ensure the following tools are installed:

- **Python 3.12+**
- **uv** (recommended for ultra-fast package management):
  ```bash
  curl -LsSf https://astral.sh/uv/install.sh | sh
  ```
- **Docker** (optional, for containerized execution)

---

## ⚙️ Environment Variables

Create a `.env` file inside `backend/`:

```bash
cp backend/.env.example backend/.env
```

Fill in your configuration:

```env
# Application
ENVIRONMENT=development
PORT=8000
HOST=0.0.0.0
CORS_ORIGINS=["http://localhost:5173","http://127.0.0.1:5173"]

# Google Gemini API
GEMINI_API_KEY=AIzaSy...
GEMINI_MODEL=gemini-2.0-flash

# ElevenLabs Text-to-Speech
ELEVENLABS_API_KEY=sk_...
ELEVENLABS_VOICE_ID=21m00Tcm4TlvDq8ikWAM # Multilingual voice

# Capital One Nessie API (simulated if missing)
NESSIE_API_KEY=your_capital_one_nessie_api_key
NESSIE_CUSTOMER_ID=your_nessie_customer_id

# Photon Spectrum SDK (iMessage, simulated if missing)
PHOTON_API_KEY=photon_...

# Supabase (transit alerts & session checkpoints)
SUPABASE_URL=your_supabase_project_url
SUPABASE_SERVICE_ROLE_KEY=your_supabase_service_role_key
```

Weather uses Open-Meteo, which needs no API key.

---

## 🚀 Installation & Local Execution

### Option A: Using `uv` (Recommended)

1. Navigate to the backend directory:
   ```bash
   cd backend
   ```

2. Create a virtual environment and sync dependencies:
   ```bash
   uv venv
   source .venv/bin/activate   # On Windows: .venv\Scripts\activate
   uv pip install -r requirements.txt
   ```

3. Run the development server with live reload:
   ```bash
   uvicorn app.main:app --host 0.0.0.0 --port 8000 --reload
   ```

### Option B: Using Standard Python Virtual Environment

```bash
cd backend
python3 -m venv .venv
source .venv/bin/activate
pip install --upgrade pip
pip install -r requirements.txt
uvicorn app.main:app --host 0.0.0.0 --port 8000 --reload
```

---

## 🔍 Verifying the Service

- **Health Check:** `http://localhost:8000/health`
- **Text-to-Speech Test:** `http://localhost:8000/api/tts?text=Hello` (returns MP3)
- **Interactive OpenAPI Documentation (Swagger UI):** `http://localhost:8000/docs`
- **ReDoc Documentation:** `http://localhost:8000/redoc`

---

## 🔌 WebSocket Endpoints

### `/ws/stream/{session_id}`
JSON messages in both directions:
- **Client to Server:** `frame` messages with a base64 JPEG, optional question text, and language (`en` / `ru`). Passive frames (no text) arrive at 1.5 FPS and don't call Gemini.
- **Server to Client:** `copilot_response` (answer text, action cards, spatial state), then `audio_chunk` (base64 ElevenLabs MP3).
- The server also answers `ping` messages with `pong`.

---

## 🐳 Docker Deployment

To build and run the backend locally with Docker:

```bash
docker build -t newyorker-jarvis-backend ./backend
docker run -p 8000:8000 --env-file backend/.env newyorker-jarvis-backend
```
