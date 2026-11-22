# New Yorker Jarvis — Product Brief & Hackathon Demo Guide

## Executive Summary

**New Yorker Jarvis** is a hands-free voice-and-camera AI transit copilot that cuts "app fatigue" and language barriers for non-English speakers, tourists, and visually impaired people navigating New York City.

Instead of taking single photos and typing to a chatbot, the user points their phone camera and asks a question out loud in English or Russian. Jarvis answers by voice, keeps a rolling memory of where the user is in the station, and can act through 12 Gemini function-calling tools: paying fares, planning routes, checking transit alerts and weather, and texting the itinerary to iMessage.

---

## 👥 Target Users & Pain Points

1. **Non-English Speakers & Immigrants:**
   - *Pain Point:* Confusing paper MTA detour notices and transit signs that translation apps don't explain in context.
   - *Jarvis Solution:* Point the camera, ask in English or Russian, and hear the answer in the same language.

2. **Commuters & Tourists with "App Fatigue":**
   - *Pain Point:* Juggling maps, transit, weather, and payment apps while holding luggage or children.
   - *Jarvis Solution:* One hands-free copilot for routes, alerts, weather, fare payment, and iMessage itineraries.

3. **Visually Impaired Riders:**
   - *Pain Point:* Difficulty reading platform signs, train designations, or service notices.
   - *Jarvis Solution:* Spoken descriptions of what the camera sees and spoken transit guidance.

---

## 🎬 3-Minute Live Hackathon Demo Script (For Judges)

### Act 1: Voice + Vision (0:00 - 1:00)
- **Presenter Action:** Points the phone at a New York subway entrance sign.
- **Presenter Voice:** *"How do I get to Columbia? Which train do I take?"*
- **System Action:** The question and the current camera frame go to Gemini, which identifies the entrance and calls `set_navigation_route`.
- **Output:** ElevenLabs reads the answer aloud and the route appears on the map.
- **Language Switch:** The presenter says *"switch to Russian"* and asks a follow-up in Russian; Jarvis answers in Russian.

### Act 2: Alerts & Weather (1:00 - 2:00)
- **Presenter Voice:** *"Are there any delays on the 1 train? And will it rain later?"*
- **System Action:** Gemini calls `lookup_transit_alerts` (Supabase) and `get_weather_forecast` (Open-Meteo).
- **Output:** Jarvis speaks the alerts and forecast and shows action cards.

### Act 3: Pay & Send (2:00 - 3:00)
- **Presenter Voice:** *"What's my balance? Pay my subway fare."*
- **System Action (Nessie):** Gemini calls `check_bank_balance` and `pay_with_card` ($2.90, MTA turnstile) through the **Capital One Nessie API**.
- **Presenter Voice:** *"Text me the transfer guide so I have it underground."*
- **System Action (Photon):** Gemini calls `dispatch_imessage_itinerary`, and the backend sends it through the **Photon Spectrum SDK**.
- **Finish:** An iMessage with the step-by-step route arrives on the presenter's phone.

---

## 🏅 Judging Criteria Alignment

| Criterion | New Yorker Jarvis Delivery |
| :--- | :--- |
| **Technical Complexity** | WebSocket pipeline, rolling visual memory with a spatial state machine, multimodal Gemini calls, and 12 function-calling tools. |
| **Stage Wow Factor** | Live phone camera, spoken questions in two languages, spoken answers, and a real iMessage arriving on stage. |
| **Community Impact** | Helps transit riders with navigation, fare payment, and language barriers. |
| **Sponsor Integration** | Gemini (vision & tools), ElevenLabs (voice), Nessie (payments), Photon (iMessage), and DigitalOcean (hosting). |
