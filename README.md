# ZentroMart VoiceOps Capstone

Public source repository: https://github.com/elontusker/zentromart-voiceops-capstone

The complete runnable source tree is currently provided in the downloadable `outputs/zentromart_voiceops_submission_package.zip` archive. Extract it, then follow the setup and run commands below. The public GitHub upload contains this README, `.env.example`, `requirements.txt`, the source archive, and the report PDF.

A four-stage customer-support assistant built against the supplied synthetic ZentroMart dataset:

1. Prompt-only LLM baseline.
2. Company-document retrieval (RAG) with evidence IDs.
3. Multi-turn support agent with verified order lookup and ticket creation.
4. Voice input and spoken replies using Gnani STT and TTS.

**Retrieval implementation note:** the current version uses BM25-style lexical ranking. The brief asks for embeddings stored in a vector index, so this part still needs to be upgraded before treating the project as final.

The LLM provider is Gemini Flash-Lite. The brief allows any LLM; Gnani is used for the required speech-to-text and text-to-speech stages.

## Setup

1. Install Python 3.10 or newer.
2. Download and extract the capstone dataset to `work/dataset/Capstone_Project_Dataset_FTTT_2026`. Keep the provided `kb/`, `data/`, `practice/` and `tools/` folders in place. The dataset contains synthetic records and is deliberately excluded from this code repository and submission ZIP.
3. Copy `.env.example` to `.env`. Add your Google AI Studio key to `LLM_API_KEY`. Voice features additionally need a Gnani API key in `GNANI_API_KEY`. Never commit `.env`.
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
- Retrieval: all gold evidence chunks were found for 47/47 answerable practice questions at top-k 5.
- RAG: 52/52 passed after review of multilingual paraphrases and deterministic business-day date arithmetic. Decision accuracy and gold-source recall were 100%.
- Multi-turn: the first complete run passed 82/96 turns. Follow-up fixes were made for deterministic escalation, delivery attempts, refusal decisions, and cancellation refund timing. Targeted live replays confirmed several corrected scenarios. A full post-fix replay was blocked by the Gemini free-tier HTTP 429 rate limit, so 82/96 is the pre-fix complete-run result, not a final post-fix score.
- Voice: using Gnani's official Python SDK, the synthetic single-turn path and one-file batch path passed end to end (STT, Gemini response, TTS WAV, and saved records). Real microphone audio, voice order verification, and the required two-minute demo video remain pending.

Evaluation JSON files are written under `work/results/`. The fixed dataset reference date is 20 October 2026. Business days are Monday through Saturday except the supplied holidays; return windows use calendar days.

## Privacy and submission

Order/customer records are synthetic and accessed only through the supplied local mock tools. Order details remain gated until verification succeeds. The mock tools are educational and are not production authentication. API keys belong only in `.env`; the dataset, virtual environment, real microphone recordings, generated ticket CSV and evaluation scratch files are excluded from the submission ZIP. The archive contains only the clearly labeled synthetic voice test sample.
