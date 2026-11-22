# Frontend — Agent Instructions

This is the mobile-first React + Vite application for **New Yorker Jarvis**. Read [../AGENTS.md](../AGENTS.md) first — universal repo rules and conventions live there. This file defines frontend-specific patterns, audio/video pipelines, and UI architecture.

---

## Stack

- **Framework:** React 18+ (Vite SPA) + TypeScript
- **Styling:** Tailwind CSS (mobile-first, dark mode, high-contrast urban HUD)
- **Icons:** `lucide-react` (lightweight icon set)
- **Video Capture:** `MediaDevices.getUserMedia` + off-screen `<canvas>` frame sampler (1.5 FPS, 640px wide JPEG)
- **Speech Input:** Browser Web Speech API (`en-US` / `ru-RU`, switched by saying e.g. "switch to Russian")
- **Audio Playback:** Web Audio API `AudioContext` decoding base64 MP3 from ElevenLabs
- **Maps:** Leaflet + OpenStreetMap tiles, routes from Transitous (transit) and OSRM (driving)
- **Real-Time Transport:** Native browser `WebSocket` with automatic 2-second reconnection
- **Package Manager:** `pnpm` (or `npm`)

---

## Layout

```text
frontend/
├── src/
│   ├── components/
│   │   ├── CameraView.tsx      # Video viewfinder
│   │   ├── MapView.tsx         # Leaflet navigation map
│   │   ├── PlanDashboard.tsx   # Trip plan HUD (mission plan tools)
│   │   ├── ChatPanel.tsx       # Conversation history
│   │   ├── Transcript.tsx      # Live subtitle HUD overlay
│   │   ├── ActionCard.tsx      # Tool result cards (Nessie, iMessage, weather)
│   │   ├── JarvisOrb.tsx       # Speaking/listening indicator
│   │   ├── LandingPage.tsx     # Landing screen
│   │   ├── HealthPage.tsx      # Backend health check page
│   │   └── StatusBar.tsx       # Latency readout & connection badge
│   ├── hooks/
│   │   ├── useLiveLens.ts      # Core hook: WebSocket, frame capture, 1.5 FPS sampling loop
│   │   ├── useCamera.ts        # Camera permission, facingMode, and media stream lifecycle
│   │   └── useAudioPlayer.ts   # Web Audio API playback of ElevenLabs audio
│   ├── config/
│   │   └── env.ts              # Single source of truth for frontend environment variables
│   ├── types/
│   │   └── stream.ts           # WebSocket event schemas (Incoming/Outgoing message contracts)
│   ├── App.tsx                 # Root mobile view layout & HUD container
│   ├── main.tsx                # React root mount
│   └── index.css               # Tailwind directives & HUD animations
├── public/
├── package.json
├── tsconfig.json
├── tailwind.config.js
└── vite.config.ts
```

---

## Dependency Policy

See universal policy in [../AGENTS.md](../AGENTS.md). Frontend-specific:

- **Strictly prefer browser-native APIs:**
  - Use native `fetch()` instead of `axios`.
  - Use native `WebSocket` instead of Socket.io client.
  - Use native `navigator.mediaDevices.getUserMedia` for video/audio ingest.
  - Use native `AudioContext` for audio playback.
- **No heavy state management libraries:** React hooks (`useState`, `useRef`, `useCallback`) and `useContext` are sufficient. Avoid Redux, MobX, or Zustand unless explicit state complexity warrants it.
- **No heavy utility libraries:** Do not install `lodash`, `moment.js`, or `ramda`. Write clean native ES6+ TypeScript.

---

## Configuration & Environment Management

- `src/config/env.ts` is the **single source of truth** for client variables.
- Never scatter `import.meta.env` calls throughout individual UI components.
- API and WebSocket URLs default to the page's own origin, so one tunnel (e.g. ngrok) serves the app, API and WebSocket through the Vite proxy:
  ```typescript
  // src/config/env.ts
  export const env = {
    API_URL: import.meta.env.VITE_API_URL || origin,
    WS_URL: import.meta.env.VITE_WS_URL || `${origin.replace(/^http/, "ws")}/ws/stream`,
    FRAME_RATE_FPS: Number(import.meta.env.VITE_FRAME_RATE_FPS) || 1.5,
    IMAGE_QUALITY: Number(import.meta.env.VITE_IMAGE_QUALITY) || 0.8,
  };
  ```

---

## Real-Time Audio & Video Streaming Rules

1. **Frame Sampling:**
   - Never send continuous full-rate video over WebSockets.
   - Grab frames with an off-screen `<canvas>` at **1.5 FPS**, downscaled to 640px wide JPEG. Each spoken question is sent with the current frame.
2. **One Turn at a Time:**
   - Only one question is in flight at once, so replies and audio never overlap. A 15-second timeout releases a stuck turn.
3. **Audio Playback:**
   - Play the base64 MP3 from each `audio_chunk` message through `AudioContext`. Clean up audio nodes on unmount.
4. **Mobile Viewport & Camera Permissions:**
   - Default camera constraint: `{ video: { facingMode: { ideal: "environment" } } }` (back camera for pointing at signs).
   - Gracefully prompt users if camera or microphone permissions are blocked.

---

## UI / UX Design Principles (Spatial HUD)

- **High-Contrast, Sunlight-Readable:** Dark semi-transparent overlays (`bg-black/60 backdrop-blur-md`) with crisp white/yellow text for high visibility in dark subway stations and bright outdoor sunlight.
- **Minimalist Hands-Free Interface:** Keep viewports uncluttered. The live video feed is the hero background; subtitles, status indicators, and action cards float unobtrusively.
- **Live Latency & Action Feedback:**
  - Show the measured response latency and connection status in the `StatusBar`.
  - When a tool executes (e.g., iMessage sent or Nessie balance checked), render an animated toast card with instant visual confirmation.

---

## Mobile Stage Testing Checklist

During hackathon presentations, the frontend will be tested on real phones:
- [ ] Run Vite with `--host` or expose via `ngrok http 5173`.
- [ ] Ensure mobile connection uses **HTTPS** (or iOS/Android will block camera and mic).
- [ ] Verify audio un-mutes after user gesture (first tap initializes `AudioContext`).
