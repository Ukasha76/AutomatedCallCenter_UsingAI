# Automated AI Call Center — Healthcare

An intelligent call-center style system for healthcare: **voice-first interaction**, **multi-agent routing**, and **structured actions** against appointments, prescriptions, and medical knowledge. The stack pairs a **React (Vite)** operator console with a **Python** backend built on **LangChain**, **Groq**, **Supabase**, and optional **Neo4j** graph RAG.

---

## Overview

This project simulates and supports a hospital or clinic contact center where callers are routed to specialized AI agents:

- **SQL agent** — Book, reschedule, or cancel appointments; look up doctors; prescription refill flows; available slots (backed by Supabase).
- **RAG agent** — Document-grounded answers from your knowledge base (Supabase + embeddings).
- **Graph RAG agent** — Structured medical relationships via Neo4j, with CSV fallback for resilience.
- **Router** — Classifies intent (e.g. emergency vs. appointment vs. general) and directs traffic to the right agent, with observability hooks (RAGAS-related logging and traces).

The **web client** provides voice capture, live transcripts, agent highlighting, and a dashboard for monitoring-style views.

---

## Features

### Backend

- Intent routing with configurable priority and keyword/pattern matching (`server/text-text.py`).
- LangChain tool agents for appointment lifecycle, doctor info, prescriptions, and related flows (`server/agents/sql/`).
- RAG retrieval and graph-enhanced Q&A (`server/agents/rag/`, `server/agents/graph/`).
- Text-to-speech pipeline for spoken responses (`server/tts/`).
- Optional **LiveKit** voice worker entrypoint (`server/agents/voice/input.py`) for real-time voice stacks.

### Frontend

- **React 19** + **Vite 6** + **Tailwind CSS 4**.
- Microphone recording, waveform visualization, and transcript display.
- LiveKit client components for future or parallel voice sessions.
- Dashboard tabs (sessions, agent performance, logs, settings) — demo-oriented UI scaffolding.

---

## Tech Stack

| Layer | Technologies |
|--------|----------------|
| UI | React, Vite, Tailwind CSS, Chart.js, Heroicons |
| Realtime / voice | LiveKit (`livekit-client`, `@livekit/components-react`) |
| Backend | Python 3.x, LangChain, LangChain-Groq, Supabase, Neo4j, RAGAS, sentence-transformers |
| LLM / APIs | Groq (Llama family), Cohere (embeddings where configured) |
| Data | Supabase (PostgreSQL + vector/RAG storage patterns) |

---

## Repository layout

```
├── client/                 # React SPA (Vite)
│   └── src/
│       ├── components/     # Call simulation, dashboard, knowledge base, etc.
│       └── ...
├── server/
│   ├── agents/
│   │   ├── sql/            # LangChain SQL / appointment tools
│   │   ├── rag/            # Retrieval-augmented generation
│   │   ├── graph/          # Neo4j graph + medical RAG
│   │   └── voice/          # LiveKit voice worker
│   ├── router/             # Traces, RAGAS logs, relevance checks
│   ├── tts/                # Text-to-speech utilities
│   ├── text-text.py        # Hospital router CLI / orchestration
│   └── requirements.txt
└── README.md
```

---

## Prerequisites

- **Node.js** 18+ (or current LTS) and **npm** (or compatible package manager).
- **Python** 3.10+ recommended.
- Accounts and keys as needed:
  - **Supabase** project (URL + service role or anon key).
  - **Groq** API key for the SQL agent LLM.
  - **Cohere** API key if you use the RAG embedding pipeline as configured.
  - **Neo4j** instance if you use the graph agent (URI, user, password, database name).

---

## Environment variables

Create a `.env` file in `server/` (or project root, depending on how you run Python) with at least:

| Variable | Purpose |
|----------|---------|
| `SUPABASE_URL` | Supabase project URL |
| `SUPABASE_KEY` | Supabase API key |
| `GROQ_API_KEY` | Groq API access for LangChain-Groq |
| `COHERE_API_KEY` | Cohere embeddings (RAG agents) |
| `NEO4J_URI` | Neo4j Bolt URI |
| `NEO4J_USERNAME` | Neo4j user |
| `NEO4J_PASSWORD` | Neo4j password |
| `NEO4J_DATABASE` | Optional; defaults to `medicalrag` in graph code |

Voice and LiveKit workers typically require additional LiveKit and OpenAI-related keys in `.env` per your deployment; see `server/agents/voice/input.py` and LiveKit documentation.

---

## Installation

### Backend

```bash
cd server
python -m venv .venv
# Windows: .venv\Scripts\activate
# macOS/Linux: source .venv/bin/activate
pip install -r requirements.txt
```

Heavy optional pieces (GPU TTS, local models) are documented in `server/tts/installation manual.txt` for a Conda-based setup.

### Frontend

```bash
cd client
npm install
```

---

## Running the application

### Web client (development)

```bash
cd client
npm run dev
```

The dev server prints a local URL (typically `http://localhost:5173`).

### Backend entry points

- **Hospital router (CLI / async orchestration):** from `server/`, run:

  ```bash
  python text-text.py
  ```

  This drives the conversational router and integrates TTS for responses in interactive mode.

- **LiveKit voice agent** (when configured):

  ```bash
  cd server/agents/voice
  python input.py
  ```

  Ensure LiveKit credentials and plugin dependencies match your environment.

### Audio API and the web UI

The React client posts recorded audio to:

`http://localhost:5000/api/process-audio`

If you use the full voice UI, run or deploy an HTTP service on that host and path that accepts multipart audio and returns JSON with fields such as `transcription`, `response`, `session_id`, and `agent_type`. Wire this endpoint to your STT pipeline and `HospitalRouter` (or equivalent) as needed for your deployment.

---

## Security and compliance

- This is a **research/demo-oriented** codebase. Do **not** use production PHI without proper BAA, encryption, access controls, and audit.
- Rotate API keys regularly and keep `.env` out of version control.
- Review Supabase Row Level Security (RLS) and Neo4j auth for any real deployment.

---

## License

Add your preferred license file if this repository is distributed publicly. If the upstream project specified a license, retain and honor it.

---

## Acknowledgments

Built with [LangChain](https://github.com/langchain-ai/langchain), [Groq](https://groq.com/), [Supabase](https://supabase.com/), [Neo4j](https://neo4j.com/), [Vite](https://vitejs.dev/), and [LiveKit](https://livekit.io/).
