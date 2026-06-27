# 🎙️ ElevenLabs Conversational AI Voice Agent

A real-time AI voice assistant powered by **ElevenLabs Conversational AI API** and **OpenAI**, enabling natural spoken dialogue through your microphone and speakers. Speak to your agent — it listens, understands, and responds with lifelike voice synthesis.
{originally used as conversational health assistant with eleven labs for a hackathon}
---

## 📋 Table of Contents

- [Overview](#overview)
- [Architecture](#architecture)
- [APIs & Integrations](#apis--integrations)
- [Project Structure](#project-structure)
- [Prerequisites](#prerequisites)
- [Installation](#installation)
- [Configuration](#configuration)
- [Usage](#usage)
- [How It Works](#how-it-works)
- [Dependencies](#dependencies)

---

## Overview

This project sets up a fully functional **real-time conversational voice agent** using ElevenLabs' Conversational AI SDK. You speak into your microphone, the agent transcribes your speech, processes it, and responds with a synthesized voice — all in near real-time.

Key capabilities:
- 🎤 Real-time speech-to-text transcription
- 🤖 LLM-powered intelligent responses
- 🔊 High-quality text-to-speech synthesis via ElevenLabs
- 📡 Live conversation session management
- 🖨️ Console transcript logging for both user and agent

---

## Architecture

```
Your Microphone
      │
      ▼
DefaultAudioInterface (PyAudio)
      │  captures audio input
      ▼
ElevenLabs Conversational AI SDK
      │  streams audio to ElevenLabs cloud
      ▼
┌─────────────────────────────────┐
│     ElevenLabs Cloud API        │
│  ┌──────────┐  ┌─────────────┐  │
│  │  STT     │→ │  LLM Agent  │  │
│  │(Transcribe)  │ (OpenAI)   │  │
│  └──────────┘  └──────┬──────┘  │
│                        │        │
│               ┌────────▼──────┐ │
│               │  TTS Engine   │ │
│               │ (ElevenLabs)  │ │
│               └───────────────┘ │
└─────────────────────────────────┘
      │  streams synthesized audio back
      ▼
DefaultAudioInterface (PyAudio)
      │
      ▼
Your Speakers
```

---

## APIs & Integrations

### 🟣 ElevenLabs Conversational AI API

The core of this project. ElevenLabs provides a full-duplex conversational AI pipeline that handles:

| Feature | Details |
|---|---|
| **SDK** | `elevenlabs` Python SDK |
| **Endpoint** | ElevenLabs Conversational AI (WebSocket-based) |
| **Authentication** | API Key via `ELEVENLABS_API_KEY` env variable |
| **Agent** | Pre-configured agent identified by `AGENT_ID` |
| **Session management** | `conversation.start_session()` / `conversation.end_session()` |
| **Audio I/O** | `DefaultAudioInterface` handles mic input + speaker output |
| **Callbacks** | Real-time hooks for agent responses, corrections, and user transcripts |

The `Conversation` object from `elevenlabs.conversational_ai.conversation` wraps the entire lifecycle — authentication, audio streaming, event handling, and session termination.

**Key SDK classes used:**
- `ElevenLabs` — authenticated API client
- `Conversation` — manages the full conversation session
- `DefaultAudioInterface` — cross-platform audio I/O using PyAudio

---

### 🟢 OpenAI API

OpenAI's LLM powers the agent's reasoning and response generation. This is configured within the ElevenLabs Agent dashboard (not directly in code), where you select the underlying model (e.g., `gpt-4o`) for your conversational agent. The ElevenLabs platform handles routing your transcribed speech to OpenAI and returning the result for synthesis.

---

### 🔵 LangChain Community (Extensible)

`langchain_community` is included in `requirements.txt` as a foundation for extending the agent with:
- Custom tool use (web search, calculators, APIs)
- Document retrieval (RAG pipelines)
- Memory and context management
- Integration with vector databases

---

## Project Structure

```
├── src/
│   ├── __init__.py          # Package initializer
│   ├── helper.py            # Utility/helper functions
│   └── prompt.py            # System prompt definitions for the agent
├── research/
│   └── trials.ipynb         # Jupyter notebook for experimentation
├── app.py                   # (Reserved) Web app or UI entrypoint
├── main.py                  # Main entrypoint — starts the voice conversation
├── template.py              # Project scaffolding script
├── test.py                  # Unit/integration tests
├── setup.py                 # Package setup configuration
├── requirements.txt         # Python dependencies
└── .env                     # API keys and environment config (not committed)
```

---

## Prerequisites

- Python 3.9+
- A working microphone and speakers
- [ElevenLabs account](https://elevenlabs.io) with:
  - An API key
  - A configured Conversational AI Agent (with an Agent ID)
- [OpenAI account](https://platform.openai.com) (configured within ElevenLabs Agent settings)

---

## Installation

**1. Clone the repository**

```bash
git clone <your-repo-url>
cd <project-folder>
```

**2. Run the project scaffolding script** (creates all necessary files and folders)

```bash
python template.py
```

**3. Create and activate a virtual environment**

```bash
python -m venv venv
source venv/bin/activate        # macOS/Linux
venv\Scripts\activate           # Windows
```

**4. Install dependencies**

```bash
pip install -r requirements.txt
```

> **Note:** PyAudio may require system-level audio libraries. On Ubuntu/Debian: `sudo apt-get install portaudio19-dev`. On macOS: `brew install portaudio`.

---

## Configuration

Create a `.env` file in the root directory:

```env
ELEVENLABS_API_KEY=your_elevenlabs_api_key_here
AGENT_ID=your_elevenlabs_agent_id_here
```

| Variable | Description | Where to find it |
|---|---|---|
| `ELEVENLABS_API_KEY` | Your ElevenLabs API key | [ElevenLabs Dashboard → API Keys](https://elevenlabs.io/app/settings/api-keys) |
| `AGENT_ID` | Your Conversational AI Agent ID | [ElevenLabs Dashboard → Conversational AI](https://elevenlabs.io/app/conversational-ai) |

> ⚠️ Never commit your `.env` file to version control. Add it to `.gitignore`.

---

## Usage

Start a voice conversation session:

```bash
python main.py
```

- **Speak** into your microphone when the session starts.
- Watch the console for live transcripts:
  ```
  User: What's the weather like today?
  Agent: I don't have real-time weather data, but I can help you with...
  ```
- Press **Ctrl+C** to end the session gracefully.
- The conversation ID is printed on exit for reference/logging.

---

## How It Works

1. `main.py` loads credentials from `.env` via `python-dotenv`.
2. An authenticated `ElevenLabs` client is created using your API key.
3. A `Conversation` session is initialized with your `AGENT_ID`, attaching:
   - `DefaultAudioInterface` for mic/speaker audio I/O
   - Callback functions that print real-time transcripts to the console
4. `conversation.start_session()` opens a WebSocket connection to ElevenLabs and begins streaming audio.
5. Your voice is transcribed → sent to the LLM → response is synthesized → played back through your speakers.
6. `SIGINT` (Ctrl+C) triggers `conversation.end_session()`, cleanly closing the connection.
7. The session's unique `conversation_id` is returned and printed for audit/logging purposes.

---

## Dependencies

| Package | Purpose |
|---|---|
| `elevenlabs` | ElevenLabs SDK — conversational AI, TTS, and audio streaming |
| `pyaudio` | Cross-platform audio I/O (microphone capture + speaker playback) |
| `openai` | OpenAI client (for direct API calls or extensions) |
| `python-dotenv` | Loads environment variables from `.env` |
| `pillow` | Image processing (for future multimodal features) |
| `langchain_community` | LangChain integrations for extending agent capabilities |

---

## 📄 License

This project is for educational and personal use. Refer to [ElevenLabs Terms of Service](https://elevenlabs.io/terms) and [OpenAI Usage Policies](https://openai.com/policies/usage-policies) when deploying.
