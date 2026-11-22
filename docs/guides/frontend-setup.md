# Frontend Setup Guide

This guide details setting up and running the **New Yorker Jarvis** React + Vite mobile-first web application, including camera capture, speech input, audio playback, and testing on mobile devices for hackathon live judging.

---

## 📋 Prerequisites

- **Node.js**: `20.x` or higher (LTS recommended)
- **Package Manager**: `pnpm` (preferred) or `npm`:
  ```bash
  # Enable pnpm via Corepack
  corepack enable
  corepack prepare pnpm@latest --activate
  ```

---

## ⚙️ Environment Configuration

Create a `.env.local` file inside `frontend/`:

Both URLs are optional. If unset, they default to the page's own origin, so a single ngrok tunnel serves the app, API and WebSocket through the Vite proxy.

```env
# API / WebSocket target (local FastAPI or production DigitalOcean URL)
VITE_API_URL=http://localhost:8000
VITE_WS_URL=ws://localhost:8000/ws/stream

# App Settings
VITE_FRAME_RATE_FPS=1.5
VITE_IMAGE_QUALITY=0.8
```

---

## 🚀 Running Locally

1. Navigate to the frontend directory:
   ```bash
   cd frontend
   ```

2. Install dependencies:
   ```bash
   pnpm install  # or npm install
   ```

3. Launch the development server:
   ```bash
   pnpm dev
   ```

The application will be accessible at: `http://localhost:5173`

---

## 📱 Crucial: Mobile Device & Live Stage Testing

Modern mobile browsers (iOS Safari and Android Chrome) **strictly require HTTPS** (or `localhost`) to access camera (`getUserMedia`) and microphone permissions.

To run and demo New Yorker Jarvis on your physical phone during the hackathon:

### Method 1: Vite HTTPS with `@vitejs/plugin-basic-ssl`
Run Vite with HTTPS enabled:
```bash
pnpm dev --host
```
Open your mobile browser and enter the LAN IP shown in your terminal (e.g., `https://192.168.1.50:5173`) and accept the self-signed certificate warning.

### Method 2: Public Tunnel with ngrok (Recommended for Stage Demos)
Expose both backend and frontend securely:

1. Expose Vite frontend:
   ```bash
   ngrok http 5173
   ```
2. Update `VITE_WS_URL` to point to the backend tunnel if testing remotely.
3. Open the resulting HTTPS link (`https://your-domain.ngrok-free.app`) on your iPhone or Android. You can now use your back camera (`facingMode: "environment"`) and ask Jarvis about what it sees.

> Speech recognition uses the browser's Web Speech API (best in Chrome). It supports English and Russian; say "switch to Russian" or "switch to English" to change.

---

## 🧩 Key Frontend Modules

- `src/hooks/useLiveLens.ts`: WebSocket connection with automatic 2-second reconnect, frame capture (off-screen `<canvas>`, 640px wide JPEG), and the 1.5 FPS live sampling loop.
- `src/App.tsx`: Speech recognition (English/Russian), language switching, and main layout.
- `src/hooks/useAudioPlayer.ts`: Plays the base64 MP3 audio from ElevenLabs via the Web Audio API.
- `src/components/CameraView.tsx`: Live camera viewfinder.
- `src/components/MapView.tsx`: Leaflet map with routes from Transitous (transit) and OSRM (driving).
- `src/components/ActionCard.tsx`: Cards showing tool results (e.g., Nessie balance, iMessage confirmation, weather).
