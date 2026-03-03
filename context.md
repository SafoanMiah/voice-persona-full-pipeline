# PROJECT: Fully Local Real-Time Voice Clone Conversational AI

## OBJECTIVE

Build a fully local, low-latency, interruptible, voice-to-voice AI system that:

1. Learns a specific person’s voice (1–2 hours audio).
2. Learns how they speak (style, phrasing, fillers, rhythm).
3. Can hold a real-time, interruptible spoken conversation.
4. Runs primarily locally on RTX 4070 Ti (12GB VRAM).
5. Minimizes API usage (only optional if needed).

System must:

* Support streaming
* Support interruption (full duplex feel)
* Maintain <2s latency if possible
* Be modular

---

# HARDWARE CONSTRAINTS

GPU: RTX 4070 Ti (12GB VRAM)
CPU: Assume modern desktop CPU
RAM: Assume ≥32GB
OS: Assume Windows or Linux (write cross-platform where possible)

---

# HIGH-LEVEL ARCHITECTURE

PIPELINE:

Mic Input
→ Voice Activity Detection (interrupt detection)
→ Streaming Speech-to-Text
→ Local LLM (style-imitation model)
→ Streaming Text-to-Speech
→ (Optional Voice Conversion layer)
→ Audio Output

All components modular.

---

# TRAINING PIPELINE

## PART 1: DATA INGESTION

Input:

* 1–2 hours clean mono WAV
* 16kHz or 24kHz
* No music
* Minimal background noise

Steps:

1. Auto-split into 5–15 second clips
2. Remove silence >500ms
3. Normalize audio
4. Filter distorted segments

Output:

* Clean dataset folder: /dataset/clean_clips/

---

## PART 2: TRANSCRIPTION (STYLE DATASET)

Tool:

* Faster-Whisper (local)

Steps:

1. Batch transcribe all clips
2. Store:

   * Raw transcript
   * Timestamped transcript
3. Merge into:
   /dataset/style_corpus.txt

Then:

* Extract filler words frequency
* Extract average sentence length
* Extract phrase frequency
* Extract speech rhythm markers (pauses)

Generate:

* style_profile.json

---

## PART 3: VOICE MODEL TRAINING

Primary Model:

* RVC WebUI (Retrieval-based Voice Conversion)

Training settings optimized for 12GB VRAM:

* 40k or 48k model
* f0 method: rmvpe
* Batch size: 6–8
* 200–300 epochs

Output:

* trained_voice_model.pth
* index file

Store under:

* /models/voice/

---

## PART 4: STYLE MODEL

Preferred method (avoid full fine-tune initially):

Base model:

* Llama 3 8B (quantized Q4_K_M)

Load locally via:

* Ollama or llama.cpp backend

Inject:

* style_profile.json
* Common phrases
* Filler words
* Tone instructions

System prompt auto-generated from transcript stats.

Optional advanced:

* QLoRA fine-tune on transcript reformatted as dialogue pairs.

Store under:

* /models/style/

---

# REAL-TIME INFERENCE PIPELINE

## 1. VAD

Use:

* Silero VAD

Function:

* Detect user speaking start
* Detect interruption
* Stop TTS immediately if user speaks

---

## 2. Streaming ASR

Use:

* Faster-Whisper streaming mode
* medium.en or small.en

Send partial transcripts to LLM as they are generated.

---

## 3. LLM

Local:

* Llama 3 8B quantized

Requirements:

* Streaming token output
* Short response chunking
* Response length cap
* Slight randomness for realism

---

## 4. TTS

Primary:

* Coqui XTTS-v2 (local)

Mode:

* Streaming audio generation

Optional:

* Generate neutral voice
* Pass through RVC model for stronger voice identity

---

## 5. AUDIO OUTPUT

* Low-latency playback
* Optional VB-Cable routing
* Support Discord or system mic injection

---

# INTERRUPTION LOGIC

If VAD detects speech while AI speaking:

1. Immediately stop TTS playback
2. Clear generation buffer
3. Resume ASR processing
4. Send new partial input to LLM

Maintain conversational state memory.

---

# LATENCY TARGETS

ASR: <600ms
LLM first token: <800ms
TTS first audio chunk: <500ms
Total perceived delay: ~1.5–2.0s

---

# DEVELOPMENT PRIORITY ORDER

1. Implement real-time ASR + LLM + TTS (no voice clone)
2. Add interruption logic
3. Train RVC model
4. Integrate voice conversion
5. Optimize latency
6. Add personality realism tuning

---

# RESEARCH TASKS FOR CLAUDE CODE

1. Identify best current RVC fork with active maintenance.
2. Identify best streaming Whisper implementation.
3. Benchmark XTTS-v2 vs alternatives for latency.
4. Optimize llama.cpp vs Ollama performance for 4070 Ti.
5. Research GPU memory optimizations for simultaneous ASR + LLM + TTS.

---

# SUCCESS CRITERIA

System can:

* Hold 5+ minute spoken conversation
* Be interrupted mid-sentence
* Resume naturally
* Sound >90% similar to training voice
* Use similar phrasing patterns

---

END OF CONTEXT
