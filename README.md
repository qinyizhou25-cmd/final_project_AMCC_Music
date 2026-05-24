# Generative Sound Archive

### An AI-Driven Companion Music Player

> *"A system that turns everyday emotional traces into music, atmosphere, and memory."*

---

## Project Background

**Generative Sound Archive** is not a music player — it is a generative sound art piece that acts as a digital companion. The central question it asks is: *what if a machine could truly listen to how you feel, and respond with something it composed just for you?*

Everyday emotions are fragmented and hard to express. Conventional music apps offer selection, not response. This project takes a different position: each time a user speaks about their mood, the system translates that moment into a unique piece of music, a live visual atmosphere, and a permanent entry in a personal emotional archive — like a diary where every page plays back as sound.

The machine's role is not to replace the author. Instead, it participates in a chain of co-creation: from user voice, through emotional interpretation, into music parameters and visual scene — all driven by a single shared logic layer. As the archive grows, the system gradually infers the user's long-term emotional character and begins generating companion tracks autonomously, marking the shift from passive tool to active companion.

The response logic draws on the **iso principle** from music therapy: rather than always pushing toward a "positive" resolution, the system first matches the user's current emotional state before gently guiding it — meeting the emotion where it lives.

---

## System Pipeline & Highlights

```
┌─────────────────────────────────────────────────────────┐
│                      User Voice Input                   │
└───────────────────────────┬─────────────────────────────┘
                            │
               Web Speech API (browser, no key needed)
               └─ fallback: SenseVoiceSmall via backend
                            │
                       Transcript
                            │
              ┌─────────────▼──────────────┐
              │     LLM Emotion Analysis    │
              │  Qwen3-8B / SiliconFlow     │
              │                             │
              │  valence  ×  arousal        │
              │  need · context             │
              │  responseMode               │
              └─────────────┬──────────────┘
                            │
              ┌─────────────▼──────────────────────────┐
              │      Shared Interpretation Layer        │
              │  (one analysis drives both outputs)     │
              └────────┬───────────────┬───────────────┘
                       │               │
          ┌────────────▼───┐   ┌───────▼──────────────┐
          │  Music Params  │   │   Visual Params       │
          │  mood / genre  │   │   sceneType           │
          │  instruments   │   │   colorMood           │
          │  BPM / density │   │   motionLevel         │
          └────────┬───────┘   └───────┬───────────────┘
                   │                   │
            Mureka API           p5.js Canvas
            Generated Song       Live Atmosphere
                   │
              ┌────▼──────────────────────────────┐
              │           Archive                  │
              │  Emotion keywords · Past entries   │
              │  Persona inference                 │
              │  → Companion track generation      │
              └────────────────────────────────────┘
```

### Emotion × Arousal → Scene Mapping

The same `valence × arousal` pair simultaneously determines the music style and the p5.js visual scene. This is the core design decision that keeps the system's language unified — sound and visuals always carry the same emotional meaning.

| Valence | Arousal | Response Mode | Visual Scene | Music Direction |
|---|---|---|---|---|
| Positive | High | Resonate | Joy (fireworks clusters) | Bright, playful, BPM 115–130 |
| Positive | Medium / Low | Resonate | Mist (floating dots) | Warm, open, gentle energy |
| Negative | High | Regulate | Rain (downward streaks) | Ambient, spacious, lower BPM |
| Negative | Medium | Regulate | Water Drop (ripples) | Still, breathing space |
| Negative | Low | Regulate | Tide (wave clusters) | Slow, melancholic, sparse |
| Neutral | Low | Soft Iso | Ribbon (flowing sine band) | Warm pad, gentle activation |
| Neutral | Medium / High | Soft Iso | Cloud (soft particles) | Soft, reflective |

**Response modes** (based on iso principle):
- **Resonate** — mirror and affirm positive / energised states
- **Regulate** — calm anxious or overloaded states without amplifying tension
- **Soft Iso** — gently match a low / empty state first, then slowly warm toward something brighter

### Design Highlights

- **Single interpretation layer** — emotion analysis runs once and its output drives both the Mureka music prompt and the p5.js atmosphere parameters simultaneously. Sound and visuals speak the same language because they share the same source.
- **8-state ritual UX** — `IDLE → RECORDING → TRANSCRIBING → REVIEWING → LISTENING → INTERPRETING → COMPOSING → PLAYING`. Every state transition is animated so generating music feels like a ceremony, not a loading spinner.
- **Async music generation** — music jobs are decoupled from the HTTP connection. The backend returns a `jobId` instantly; the frontend polls every 4 s. This prevents proxy timeout failures on long generation calls.
- **Living archive & persona** — past transcripts are stored locally and surfaced as an emotion keyword cloud. Clicking a keyword triggers the companion flow: the LLM reads up to 15 past entries to infer the user's long-term emotional character and compose a personalised track.
- **Graceful fallback** — if no API keys are configured, the system runs in mock mode with pre-written transcripts and a sample audio file so the entire UX can be explored without any credentials.

---

## Project Structure

```
generative-sound-archive/               ← pnpm monorepo root
├── artifacts/
│   ├── api-server/                     ← Express 5 backend (Node.js / TypeScript)
│   │   └── src/
│   │       ├── services/
│   │       │   ├── analyze.ts          ← LLM emotion analysis + companion analysis
│   │       │   ├── transcribe.ts       ← Speech-to-text (SiliconFlow SenseVoice)
│   │       │   └── music.ts            ← Mureka music generation
│   │       └── routes/
│   │           └── audio.ts            ← API routes + async job store
│   └── generative-sound-archive/       ← React + Vite frontend
│       └── src/
│           ├── App.tsx                 ← 8-state machine + scene mapping
│           ├── hooks/
│           │   └── useRecorder.ts      ← MediaRecorder + Web Speech API
│           ├── components/
│           │   ├── AtmosphereCanvas.tsx ← p5.js visual scenes (8 types)
│           │   └── AudioPlayer.tsx      ← song card with waveform
│           └── services/
│               └── api.js              ← frontend API calls + polling logic
└── package.json
```

---

## How to Run

### Prerequisites

- **Node.js** 18 or later
- **pnpm** — install with `npm install -g pnpm`

### 1. Install dependencies

From the project root:

```bash
pnpm install
```

### 2. Configure API keys

Create the file **`artifacts/api-server/.env`** and fill in your keys:

```env
# ── Emotion analysis LLM ──────────────────────────────────────
# Provider: SiliconFlow  https://cloud.siliconflow.cn
LLM_KEY=sk-your-siliconflow-key
LLM_API_BASE_URL=https://api.siliconflow.cn/v1
LLM_MODEL=Qwen/Qwen3-8B

# ── Music generation ──────────────────────────────────────────
# Provider: Mureka  https://www.mureka.ai
MUSIC_API_KEY=your-mureka-api-key

# ── Speech-to-text (optional) ─────────────────────────────────
# The browser's built-in Web Speech API is used first (no key needed).
# This fallback is called only when browser STT returns nothing.
# Provider: SiliconFlow — same key as LLM_KEY works.
# Ensure FunAudioLLM/SenseVoiceSmall is enabled for your account.
STT_API_KEY=sk-your-siliconflow-key
STT_API_BASE_URL=https://api.siliconflow.cn/v1
```

> **Where the keys are read in the source code:**
> | Variable | File | Line |
> |---|---|---|
> | `LLM_KEY` | `artifacts/api-server/src/services/analyze.ts` | 5 |
> | `MUSIC_API_KEY` | `artifacts/api-server/src/services/music.ts` | 4 |
> | `STT_API_KEY` | `artifacts/api-server/src/services/transcribe.ts` | 6 |

### 3. Start the backend

```bash
pnpm --filter @workspace/api-server run dev
```

The API server starts on **port 8080**.

### 4. Start the frontend

Open a second terminal:

```bash
pnpm --filter @workspace/generative-sound-archive run dev
```

Open the URL printed in the terminal (default: `http://localhost:5173`).

---

### Running without API keys (Mock Mode)

Create **`artifacts/generative-sound-archive/.env`**:

```env
VITE_MOCK_MODE=true
```

In mock mode the app uses pre-written sample transcripts and a public sample audio file. Every stage of the UX state machine can be explored without making any real API calls.

---

## API Keys — Where to Get Them

| Key | Provider | Notes |
|---|---|---|
| `LLM_KEY` | [SiliconFlow](https://cloud.siliconflow.cn) | Register → API Keys → Create. Free tier available. Model used: `Qwen/Qwen3-8B`. |
| `MUSIC_API_KEY` | [Mureka](https://www.mureka.ai) | Register → Developer → API Keys. |
| `STT_API_KEY` | [SiliconFlow](https://cloud.siliconflow.cn) | Same key as `LLM_KEY`. Go to Model Square and enable `FunAudioLLM/SenseVoiceSmall` for your account. |

---

## Tech Stack

| Layer | Technology |
|---|---|
| Frontend | React 19, Vite, TypeScript |
| Visual scenes | p5.js — 8 generative scene types (rain, tide, ribbon, mist, cloud, water drop, joy, heart) |
| Backend | Express 5, Node.js 24, TypeScript |
| Monorepo | pnpm workspaces |
| Emotion LLM | SiliconFlow — Qwen/Qwen3-8B |
| Music generation | Mureka API |
| Speech-to-text | Browser Web Speech API + SiliconFlow FunAudioLLM/SenseVoiceSmall |
| Archive | localStorage — no database required |

---

*"The machine does not just generate sound. It listens, translates, remembers, and responds."*
