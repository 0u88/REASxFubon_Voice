# REASxFubon Voice

A Flask web tool that turns raw customer-service call recordings into structured, reviewable records: transcription, hallucination cleanup, speaker separation, intent classification, and a summary — all through a browser upload.

Built during the **REAS x Fubon** project as a proof of concept for processing Fubon Insurance's customer-service call recordings.

> **Note on data**: the recordings, transcripts, and case summaries used during development were real Fubon customer-service calls and have been removed from this repository and its git history. `recordings/` and the `class_*/voice_emo/` folders are kept as empty placeholders (see [Project structure](#project-structure)) so the expected layout is still visible.

## What it does

1. **Upload** a call recording (`.wav`) through the web UI.
2. **Transcribe** it with the OpenAI Whisper API, splitting long audio into ~280-second chunks.
3. **Clean up the transcript**:
   - Rule-based hallucination detection (repeated phrases, nonsense segments typical of Whisper on silence/noise).
   - Silence-trimmed re-transcription of flagged segments, retried with a few temperature/prompt variations until the output passes a quality check.
   - Deduplication before and after speaker separation.
4. **Separate speakers and correct wording** with a GPT pass (fixes homophone-style ASR errors against a domain vocabulary list, e.g. insurance terms that Whisper tends to mis-hear).
5. **Classify and summarize** the call by insurance category (health, travel, auto, other) and produce a short intent summary.
6. **Review results** in the browser: view/download transcripts, emotion notes, and an aggregated Excel export; basic usage analytics (uploads per day/hour, button clicks).

## Project structure

```
.
├── src/
│   ├── app.py              # Flask web app: upload, processing status, review/download routes
│   ├── pipeline.py         # Core pipeline: transcription, hallucination detection,
│   │                       #   speaker separation, classification/summarization
│   └── templates/
│       └── index.html
├── class_car/               \
├── class_disease/            } one folder per insurance category app.py classifies into;
├── class_other/               } each holds a voice_emo/ subfolder for per-call emotion notes
├── class_travel/            /   (transcripts land in a class_*/voice_text/ folder created at runtime)
├── recordings/              # uploaded audio lands here (git-ignored, folder kept as placeholder)
├── dockerfile
├── requirements.txt
└── .env.example
```

## Setup

Requires Python 3.11+, `ffmpeg` (used by `pydub` for audio conversion), and an OpenAI API key.

```bash
pip install -r requirements.txt
cp .env.example .env   # fill in OPENAI_API_KEY
python src/app.py      # serves on http://localhost:5003
```

### Docker

```bash
docker build -t reasxfubon-voice .
docker run -p 5003:5003 --env-file .env reasxfubon-voice
```

## Prompt-tuning notes

Part of the development work was iterating on the GPT prompt/temperature used when re-transcribing hallucinated segments (see `retranscribe_hallucination_segments` in `src/pipeline.py`) — testing different system prompts and temperatures (0.4–0.6) and picking whichever pass scored best on the quality checks. An early standalone experiment script comparing these variants existed during development but has been removed as a near-duplicate of `pipeline.py`; the surviving strategy list in `retranscribe_hallucination_segments` reflects that tuning.

## Contributors

Built with a teammate as part of the REAS x Fubon project; see the commit history for individual contributions.
