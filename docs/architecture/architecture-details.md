# Multi-Tier System Architecture Specification: Formal Protocol & Boundary Definitions

**Author:** Vinayak Sharma  
**Context:** SAGE Theoretical Architectural Foundations  
**Preprint Reference:** TR-2026-SAGE01-ARCH

---

## 1. Architectural Taxonomy

The SAGE platform implements an asymmetric four-tier topology designed to isolate physical real-time constraints from high-latency cognitive processing:

```
[Tier 1: Micro-Sensor Edge Node]
        │
        │ 802.11 b/g/n Ad-Hoc Binary Stream (HTTP REST)
        ▼
[Tier 2: Personal Edge Gateway Host]
        │
        │ gRPC / REST (HTTPS TLS 1.3)
        ▼
[Tier 3: Multimodal Cognitive AI Core]
        │
        │ Asynchronous Zero-Knowledge Tokenized Sync
        ▼
[Tier 4: Sovereign Personal Cloud Vault & Vector Store]
```

---

## 2. Tier 1: Micro-Sensor Edge Node Specification

### 2.1 Hardware Operating Abstraction
* **Execution Paradigm:** FreeRTOS dual-core preemptive multitasking with static priority scheduling.
* **Core Pinning Topology:**
  - **Core 0 (Capture Task, Priority 24 - Critical):** Exclusively drives the OV2640 DVP parallel bus and the I2S PDM decimation engine. Memory transactions utilize dedicated Direct Memory Access (DMA) channels writing into pre-allocated external PSRAM partitions.
  - **Core 1 (Protocol Server Task, Priority 5 - Normal):** Manages the 802.11 SoftAP networking stack, servicing HTTP/1.1 binary chunk streaming requests from the mobile gateway host.

### 2.2 Physical Memory Boundaries
* **Total Addressable PSRAM:** $8192\text{ KB}$ ($8\text{ MB}$).
* **Video Partition $\mathcal{B}_v$:** Fixed contiguous block of $4300\text{ KB}$ formatted as in-memory AVI RIFF structures with embedded microsecond timing descriptors.
* **Audio Partition $\mathcal{B}_a$:** Fixed contiguous block of $491\text{ KB}$ formatted as in-memory WAV RIFF structures holding $16\text{ kHz}$ 16-bit mono PCM samples.
* **Runtime Dynamic Heap Footprint:** Strictly $0\text{ bytes}$ during active acquisition epochs ($\mathcal{O}(1)$ static allocation).

---

## 3. Tier 2: Personal Edge Gateway Host Specification

### 3.1 Network Handshake & P2P Burst Protocol
* **Physical Link:** IEEE 802.11 b/g/n point-to-point ad-hoc Wi-Fi connection.
* **Transfer Engine:** Concurrent dual-channel HTTP streaming with non-blocking TCP socket buffers.
* **Integrity Verification:** End-to-end MD5 checksum validation computed over binary payloads prior to buffer release on the edge node.

### 3.2 Temporal Phase Alignment & Containerization
* **Header Parsing:** Decodes elapsed microsecond duration $T_{\mu s}$ and total visual frame count $N_v$ from the AVI header.
* **Timestamp Normalization:** Interpolates presentation timestamps ($PTS_v$) to match the acoustic duration $T_a$, guaranteeing sub-$12\text{ ms}$ phase coherence.
* **Hardware-Accelerated Transcoding:** Encodes synchronized audiovisual streams into standardized ISO base media file format (`.mp4`) utilizing client-side hardware codecs.

---

## 4. Tier 3: Multimodal Cognitive AI Core Specification

### 4.1 Feature Extraction Pipeline
* **Visual Subsampling:** Extracts $m = 4$ uniform temporal keyframes across the 15-second episode to capture dynamic scene progression while constraining token complexity.
* **Acoustic Transcription:** Automatic Speech Recognition (ASR) engine transcribes speech channels, filtering ambient background noise.
* **Multimodal Cross-Attention Synthesis:** A joint vision-language foundation model ingests keyframes and transcript context to synthesize:
  1. **Canonical Title:** Short, hierarchical namespace identifier (e.g., `Meeting_Lab_Discussion`).
  2. **Episodic Summary:** Concise natural-language synopsis of active events.
  3. **Visual Scene Descriptor ($\mathbf{p}_i$):** Dense text prompt characterizing actors, physical objects, environmental lighting, and relative spatial layout.

### 4.2 Dense Semantic Vector Space Mapping
* **Embedding Model:** Deep dense metric projection $g_\phi: \mathcal{T} \to \mathbb{R}^d$ ($d = 768$ or $1536$).
* **Normalization:** All episodic vectors are mapped onto the unit hypersphere: $\mathbf{e}_i = \frac{g_\phi(E_i)}{\|g_\phi(E_i)\|_2}$ to support efficient cosine similarity calculations.

---

## 5. Tier 4: Sovereign Storage & Temporal Vector Indexing

### 5.1 Storage Hierarchy & Folder Taxonomy
Episodes are archived within the user's private cloud vault under a deterministic, human-auditable directory structure:

```
SAGE_STORAGE_VAULT/
└── [User_Cryptographic_UUID]/
    └── [Session_Date_YYYYMMDD]/
        └── Episode_[Timestamp_Epoch]_[ClipID]/
            ├── episode_synced.mp4        # Synchronized audiovisual container
            ├── raw_video.avi             # Unmodified sensor buffer archive
            ├── raw_audio.wav             # Unmodified acoustic PCM buffer
            ├── metadata.json             # Timing, GPS (optional), sensor telemetry
            └── semantic_descriptor.json  # Title, summary, dense prompt, vector embedding
```

### 5.2 Retrieval-Augmented Generation (RAG) Topology
* **Vector Index:** Approximate Nearest Neighbor (ANN) index structured over the episodic embedding graph.
* **Temporal-Decay Scoring:** Ranks episodes using combined cosine distance and exponential half-life recency decay.
* **Conversational Generation:** Constructs grounded answers referencing direct evidence timestamps, precluding generative hallucination.
