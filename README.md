# ZentroMart VoiceOps Capstone

Public source repository: https://github.com/elontusker/zentromart-voiceops-capstone

The ready-to-submit archive is `zentromart_voiceops_submission_package.zip` at the repository root. If you are reading this from inside the extracted archive, you already have the complete source, setup guide, report, sample outputs, and demo. Follow the setup steps below.

A four-stage customer-support assistant built against the supplied synthetic ZentroMart dataset:

1. Prompt-only LLM baseline.
2. Company-document retrieval (RAG) with evidence IDs.
3. Multi-turn support agent with verified order lookup and ticket creation.
4. Voice input and spoken replies using Gnani STT and TTS.

The retrieval system creates multilingual semantic embeddings locally with `nomic-embed-text` and stores them in a persistent vector cache (`work/results/vector_index_cache.json`). Search fuses cosine similarity with BM25 lexical ranking, preserves exact SKU matches, and adds policy evidence routes for safety-critical topics. Evaluation found every gold source for 47/47 answerable questions (100% at top-k 5).

The text agent uses local Qwen3 4B through Ollama, so it does not need a Gemini account, API key, or hosted LLM quota. The brief allows any hosted or open-source LLM. Gnani is used for speech-to-text and text-to-speech.

## Setup

1. Install Python 3.10 or newer and [Ollama for Windows](https://ollama.com/download/windows). Open Ollama once, then download `qwen3:4b` and `nomic-embed-text` with `ollama pull qwen3:4b` and `ollama pull nomic-embed-text`.
2. Download and extract the capstone dataset to `work/dataset/Capstone_Project_Dataset_FTTT_2026`. Keep its `kb/`, `data/`, `practice/` and `tools/` folders in place. The dataset contains synthetic records and is excluded from this code repository and submission ZIP.
3. Copy `.env.example` to `.env`. No LLM key is needed. Voice features need your Gnani API key in `GNANI_API_KEY`. Never commit `.env`.
4. Install packages with `python -m pip install -r requirements.txt`.

## Run the project

- Prompt-only baseline: `python run_baseline.py "Can I return an opened product?"`
- Evaluate baseline: `python evaluate_baseline.py`
- Check document retrieval: `python evaluate_retrieval.py`
- Ask the RAG agent: `python run_rag.py "Can I return an opened product?"`
- Evaluate RAG: `python evaluate_rag.py`
- Start a multi-turn text conversation: `python run_tool_agent.py`
- Evaluate multi-turn tools: `python evaluate_conversations.py`
- Process all WAV files in a folder: `python run_voice_batch.py path\to\audio_folder --language en-IN --voice-language auto`

Voice batch input must be mono, 16 kHz, 16-bit PCM WAV. Each recording saves a transcript, assistant reply, synthesized WAV, tool-call record, and retrieved evidence chunks (IDs and text) under `work/results/voice_batch/<recording-name>/`. If Gnani returns an empty transcript, the assistant speaks a request to say it again. Record your own short test audio; the supplied dataset contains no recordings. Add `--conversation` to retain conversation state across sequential files. See `outputs/demo_script.md` for the under-two-minute demonstration outline.

## Evaluation snapshot

- Prompt-only baseline: 8/52 passed the required-fact checks; it escalated 49/52 without access to company documents.
- Retrieval: all gold evidence chunks were found for 47/47 answerable practice questions at top-k 5 with local embeddings and BM25.
- RAG: 52/52 passed; the final result combines a 48/52 full run and a 4/4 focused replay after edge-case fixes.
- Multi-turn tools: 96/96 supplied turns passed; this combines a 92/96 full run and a 17/17 focused replay across four corrected scenarios.
- Voice: three user-recorded WAV clips were processed in sequence through Gnani STT, local Qwen support flow, and Gnani TTS. The 1:51 demo shows a document-grounded question, a follow-up, and a human handoff. This is batch processing; continuous live microphone streaming and spoken order verification are not demonstrated.

Evaluation JSON files are under `work/results/`. The fixed dataset reference date is 20 October 2026. Business days are Monday through Saturday except listed holidays; return windows use calendar days.

## Privacy and submission

Order/customer records are synthetic and accessed only through supplied local mock tools. Order details remain gated until verification succeeds. The mock tools are educational, not production authentication. Keep API keys in `.env`; the dataset, virtual environment, raw voice recordings, generated ticket CSV, and evaluation scratch files are excluded from the submission ZIP. The demo includes playback of the recorded sample questions and spoken replies; the archive also contains clearly labeled synthetic voice test files.


