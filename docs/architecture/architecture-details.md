# System Architecture Details

This document outlines the four-tier research architecture of **SAGE (Smriti Adaptive Guardian Engine)**, distinguishing **currently implemented** components from **planned** enhancements.

---

## 1. Embedded Wearable Tier *(Implemented: Firmware v3.0.0)*
* **Runtime Environment:** FreeRTOS on Seeed Studio XIAO ESP32-S3 Sense (Xtensa LX7 dual-core @ 240 MHz, 8 MB PSRAM).
* **Dual-Core Execution Model:**
  * **Core 1 (Arduino `loop()`):** Hosts the SoftAP (`SAGE-XIAO`, channel 6) and HTTP control/preview server on port `80` (`/health`, `/status`, `/live_frame`), piping VGA (`640×480`) OV2640 JPEG frames directly from PSRAM DMA buffers.
  * **Core 0 (Pinned FreeRTOS `audioTask`):** Reads 16-bit mono PDM microphone samples via I2S (`16 kHz`, 8x digital gain with clipping guard) and streams PCM packets over TCP port `81` with `TCP_NODELAY`.
* **Session Watchdog:** 45-second firmware heartbeat timeout resets device state cleanly if the mobile link drops.

---

## 2. Companion Mobile Application Tier *(Implemented: Flutter + Kotlin Native)*
* **Framework:** Flutter / Dart application with Provider and reactive `ValueNotifier` progress streams.
* **Android Native Network Binding (`MainActivity.kt`):** Uses `WifiNetworkSpecifier`, `ConnectivityManager.bindProcessToNetwork()`, and native `Network.openConnection()` HTTP bridging so traffic to `192.168.4.1` never leaks to cellular interfaces.
* **Client-Side AVI Muxing Engine (`MjpegRecorder`):** Interleaves variable-rate MJPEG video chunks (`00dc`) and 16 kHz PCM audio chunks (`01wb`) into a standard RIFF/AVI stream, appends an `idx1` seek index, and patches the 1024-byte RIFF header post-recording (`videoRate = frameCount * 16000`, `videoScale = audioSamples`) to eliminate audio-video clock drift.
* **Local Storage:** Stores pending `.avi` clips in `/storage/emulated/0/Download/SAGE` and maintains per-user isolated JSON memory vaults (`vault.json`) and analysis records (`SharedPreferences`). *(Planned: migration to SQLite + on-device encryption).*

---

## 3. Cognitive AI & Retrieval Tier *(Implemented + Planned Extensions)*
* **Multimodal Video Analysis (`GeminiService`):** Uploads `.avi` segments to the Gemini File API, polls until `ACTIVE`, and invokes JSON-mode generation under a 7-pillar multimodal system instruction with multi-model fallback and HTTP 429/503 exponential backoff.
* **Dual-Representation Output:** Produces both `human_memory` (calm chronological narrative) and `reconstruction_blueprint` (timestamped people, objects, spatial relations, audio transcripts, OCR, and evidence claims).
* **Conversational Recall & Hybrid RAG (`MemoryStoreService`):**
  * *Implemented:* Multi-source context retrieval combining token-overlap keyword scoring across persistent profile facts, active projects, session summaries, and chunked document attachments (PDF, DOCX, code, images) within a 12,000-character budget, alongside recorded video summaries.
  * *Planned:* Local/cloud dense vector embeddings for semantic similarity retrieval at larger archive scales.

---

## 4. Personal Cloud & Archival Tier *(Implemented)*
* **User-Owned Storage (`GoogleDriveService`):** Authenticates via OAuth 2.0 using least-privilege `drive.file` and `drive.appdata` scopes. Video clips and structured JSON descriptions are archived in `/SAGE/YYYY-MM-DD/` inside the user's own Google Drive.
* **Cross-Device Vault Sync:** Synchronizes `sage_vault_<userId>.json` in Drive `appDataFolder` using tombstone-aware conflict resolution so deleted facts/sessions are never resurrected.
* **Transactional Safety Invariant:** Local temporary `.avi` files are deleted **only after** Google Drive confirms upload success.
