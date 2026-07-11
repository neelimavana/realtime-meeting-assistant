# Real-Time Meeting Intelligence Assistant

A live meeting assistant that transcribes speech in real time and generates a structured summary — key points and action items — while the meeting is still happening, instead of after it ends.

## Why I built this

Meeting notes are almost always an afterthought: someone scribbles bullet points, or nobody takes notes at all. This project chains real-time speech recognition with an LLM summarization step so that by the time a meeting ends, the summary is already done — turning a ~15-minute manual task into a ~30-second automated one.

## Features

- **Real-time transcription** of live audio using OpenAI Whisper, processed in small streaming chunks
- **Low-latency pipeline** — transcript text appears within seconds of speech, not after the call ends
- **Automatic structured summarization** — key points and action items extracted via prompt-engineered LLM calls
- **WebSocket audio streaming** — no need to upload a full recording after the fact
- **FastAPI backend** serving the entire pipeline through a single service

## Architecture

```
 Microphone / Meeting Audio
            │
            ▼
   WebSocket Audio Stream (2-3s chunks)
            │
            ▼
     ┌─────────────┐
     │   Whisper    │  → live transcript chunks
     │    (ASR)     │
     └──────┬──────┘
            │
            ▼
   Running Transcript Buffer
            │
            ▼
     ┌─────────────┐
     │   LLM API    │  → structured summary
     │ (prompt eng.)│     + action items
     └──────┬──────┘
            │
            ▼
     FastAPI Response / Dashboard
```

## Tech Stack

| Layer | Technology |
|---|---|
| Speech-to-text | OpenAI Whisper |
| Real-time audio transport | WebSockets |
| Summarization | LLM API + prompt engineering |
| Backend | Python, FastAPI |

## Getting Started

### Prerequisites
- Python 3.10+
- An LLM API key (OpenAI / Anthropic / etc.)
- `ffmpeg` installed (for audio chunk decoding)

### Run locally
```bash
git clone https://github.com/neelimavana/realtime-meeting-assistant.git
cd realtime-meeting-assistant
pip install -r requirements.txt
uvicorn app.main:app --reload
```

### Environment variables
```
LLM_API_KEY=your_key_here
WHISPER_MODEL=base   # or small/medium depending on latency vs. accuracy tradeoff
```

## API Overview

| Method | Endpoint | Description |
|---|---|---|
| `WS` | `/ws/audio/{session_id}` | Stream live audio chunks in |
| `GET` | `/sessions/{id}/transcript` | Fetch running transcript |
| `GET` | `/sessions/{id}/summary` | Fetch generated summary + action items |
| `POST` | `/sessions/{id}/end` | End session, finalize summary |

## Results (internal testing)

| Metric | Result |
|---|---|
| Transcription latency per chunk | < 3 seconds |
| Word-level transcription accuracy | ~90% |
| Audio chunk size | 2–3 seconds |
| Time to readable transcript text | < 5 seconds from speech |
| Longest meeting tested without degradation | 60+ minutes |
| Summary relevance (manual review, 20+ test meetings) | ~85% rated relevant |
| Manual note-taking time saved | ~15 min → <30 sec (~97% reduction) |

*Numbers above are from manual test sessions logged in `/tests/eval_notes.md`. Re-run your own meetings to reproduce/update.*

## Project Structure
```
realtime-meeting-assistant/
├── app/
│   ├── main.py             # FastAPI entrypoint
│   ├── audio_stream.py      # WebSocket audio ingestion
│   ├── transcriber.py       # Whisper streaming wrapper
│   ├── summarizer.py        # LLM prompt + summary generation
│   └── session_store.py     # In-memory/session state
├── tests/
│   └── eval_notes.md
├── requirements.txt
└── README.md
```

## Future Improvements
- Speaker diarization (who said what)
- Multi-language support
- Slack/email auto-delivery of summaries post-meeting

## License
MIT

