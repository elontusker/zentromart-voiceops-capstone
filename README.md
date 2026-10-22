# ZentroMart VoiceOps Capstone

Public source repository: https://github.com/elontusker/zentromart-voiceops-capstone

The complete runnable source tree, setup guide, scripts, sample outputs, and report are in the downloadable `zentromart_voiceops_submission_package.zip` archive at the repository root. Download and extract it, then follow the README inside the archive. The root also contains `.env.example`, `requirements.txt`, and the report PDF for quick access.

A four-stage customer-support assistant built against the supplied synthetic ZentroMart dataset:

1. Prompt-only LLM baseline.
2. Company-document retrieval (RAG) with evidence IDs.
3. Multi-turn support agent with verified order lookup and ticket creation.
4. Voice input and spoken replies using Gnani STT and TTS.

The retrieval system now creates multilingual semantic embeddings locally with `nomic-embed-text` and stores them in a persistent vector cache (`work/results/vector_index_cache.json`). Search fuses cosine similarity with BM25 lexical ranking, preserves exact SKU matches, and adds policy evidence routes for safety-critical topics. The dataset evaluation found every gold source for 47/47 answerable questions (100% at top-k 5).

The text agent uses the local open-source Qwen3 4B model through Ollama, so it does not need a Gemini account, API key, or hosted LLM quota. The brief allows any hosted or open-source LLM. Gnani remains required for speech-to-text and text-to-speech.

## Setup

1. Install Python 3.10 or newer and install [Ollama for Windows](https://ollama.com/download/windows). Open Ollama once so its local service is running, then download both models with `ollama pull qwen3:4b` (about 2.5 GB) and `ollama pull nomic-embed-text` (about 274 MB). With `OLLAMA_MODELS` configured, they are stored in that directory.
2. Download and extract the capstone dataset to `work/dataset/Capstone_Project_Dataset_FTTT_2026`. Keep the provided `kb/`, `data/`, `practice/` and `tools/` folders in place. The dataset contains synthetic records and is deliberately excluded from this code repository and submission ZIP.
3. Copy `.env.example` to `.env`. No LLM key is needed. Voice features need your Gnani API key in `GNANI_API_KEY`. Never commit `.env`.
4. Install Gnani's official Python SDK used by the voice adapters:

   `python -m pip install -r requirements.txt`

## Run the project

Ask the prompt-only baseline a question:

`python run_baseline.py "Can I return an opened product?"`

Evaluate the baseline on the 52 practice questions:

`python evaluate_baseline.py`

Check document retrieval against the dataset's expected evidence:

`python evaluate_retrieval.py`

Ask the RAG agent a question:

`python run_rag.py "Can I return an opened product?"`

Evaluate RAG answers, safety and evidence:

`python evaluate_rag.py`

Start a multi-turn, text-based support conversation:

`python run_tool_agent.py`

Evaluate multi-turn decisions, required facts and tool calls:

`python evaluate_conversations.py`

Process all short WAV recordings in a folder with Gnani STT, the support agent, and Gnani TTS:

`python run_voice_batch.py path\to\audio_folder --language en-IN --voice-language auto`

The voice batch expects mono, 16 kHz, 16-bit PCM WAV inputs. Each recording produces a transcript, assistant reply, synthesized WAV, tool-call record and retrieved evidence IDs under `work/results/voice_batch/<recording-name>/`. Record your own short test audio; the supplied dataset contains no recordings. To retain verification and conversation state across sequential recordings, name turns in order (for example `01.wav`, `02.wav`) and add `--conversation`. See `outputs/demo_script.md` for the required two-minute demonstration outline.

## Evaluation snapshot

- Prompt-only baseline: 8/52 passed the simple required-fact checks; it escalated 49/52 because it had no access to company documents.
- Retrieval: all gold evidence chunks were found for 47/47 answerable practice questions at top-k 5 with the local embedding + BM25 hybrid index.
- RAG: 52/52 passed on the local Qwen3 4B + embedding vector retrieval flow; decision accuracy and gold-source recall were 100%. The final result combines a 48/52 full run and a 4/4 focused replay after edge-case fixes.
- Multi-turn: 96/96 supplied practice turns passed. This final result combines a 92/96 full run and a 17/17 focused replay of four corrected scenarios after edge-case fixes. These are synthetic practice results, not a guarantee on unseen evaluator prompts.
- Voice: the official Gnani Python SDK passed synthetic single-turn and one-file batch checks using local Qwen for the support reply. The provided test audio is synthetic; recording a real microphone interaction, spoken order verification, and the required two-minute demo video remain pending.

Evaluation JSON files are written under `work/results/`. The fixed dataset reference date is 20 October 2026. Business days are Monday through Saturday except the supplied holidays; return windows use calendar days.

## Privacy and submission

Order/customer records are synthetic and accessed only through the supplied local mock tools. Order details remain gated until verification succeeds. The mock tools are educational and are not production authentication. API keys belong only in `.env`; the dataset, virtual environment, real microphone recordings, generated ticket CSV and evaluation scratch files are excluded from the submission ZIP. The archive contains only the clearly labeled synthetic voice test sample.
