# Methodology: Asynchronous Capture and Semantic Recall

## 1. Asynchronous Capture Lifecycle
SAGE operates on a decoupled capture-and-enrichment lifecycle designed to keep the wearable microcontroller lean while preserving rich multimodal context on the companion smartphone:

```
[Wearable Standby (SAGE-XIAO AP Ready)]
        │
        ▼ (Phone Connects via WifiNetworkSpecifier + bindProcessToNetwork)
[Dual-Core Capture & Streaming] ── Core 1: JPEG (:80) | Core 0: I2S PCM (:81)
        │
        ▼ (Client-Side Interleaved Muxing)
[Streaming RIFF/AVI Assembly] ──── Interleaves 00dc + 01wb, rewrites header at stop
        │
        ▼ (Deferred Background Queue — Requires Connectivity)
[Multimodal AI Synthesis] ──────── Gemini File API -> 7-pillar dual JSON memory
        │
        ▼ (Transactional Cloud Archival)
[User-Owned Drive + Local Index] ─ Delete local .avi ONLY after Drive confirmation
```

---

## 2. Multi-Channel Stream Reconciliation (Implemented)
Because the OV2640 camera sensor and I2S PDM microphone operate on independent hardware clocks, fixed-FPS container encoding introduces perceptible audio-visual drift (~5% over short intervals). SAGE resolves this on the mobile client (`MjpegRecorder`):
1. **Placeholder Header Allocation:** Writes a 1024-byte RIFF/AVI placeholder header at recording start.
2. **Arrival-Order Interleaving:** Appends JPEG video frames (`00dc`) and 16 kHz PCM audio blocks (`01wb`) as they arrive over separate sockets, recording exact byte offsets for an `idx1` seek index.
3. **Post-Hoc Header Calibration:** Upon stopping, seeks back to byte `0` and patches `strh` video rate and scale from the actual measured audio sample count:
   - `videoRate = frameCount * audioSampleRate` (`frameCount * 16000`)
   - `videoScale = totalAudioSamples`
   This keeps measured A/V drift below `< 40 ms` on 15-second clips.

---

## 3. Cognitive Synthesis & Dual-Representation Memory (Implemented)
1. **Multimodal Ingestion:** Clips are uploaded via the Gemini File API and polled until `ACTIVE`.
2. **7-Pillar Grounded Prompting:** The model analyzes speech, non-speech audio, visual entities, spatial layout, off-camera sounds, and chronology under a strict anti-hallucination instruction (marking uncertain audio as `unclear`, preserving original spoken languages).
3. **Dual JSON Output:** Generates:
   - `human_memory`: A calm, accessible, chronological story tailored for human reading/listening.
   - `reconstruction_blueprint`: Exhaustive structured evidence (`people`, `objects`, `actions`, `audio.transcript`, `visible_text`, `evidence`, `uncertainties`).

---

## 4. Conversational Retrieval Pipeline (Current vs. Planned)
1. **Current Implementation (Hybrid Token RAG + Grounded Prompting):**
   - Retrieves relevant context from per-user Profile Facts, Project Memories, past Session Summaries, and chunked Document Attachments (`AttachmentService`) via token-overlap scoring within a 12,000-character budget.
   - Injects stored recorded video summaries and timelines alongside the user's voice/text query into `GeminiService.chatWithConversationIntelligence`.
   - Returns structured intent, conversational answer, and `matchedVideoId` so the user can open the original video clip and transcript as ground-truth evidence.
2. **Planned Phase 2 Upgrade:**
   - Migrate summary storage from `SharedPreferences` to SQLite and introduce dense vector embeddings for large-scale semantic retrieval and offline querying.
