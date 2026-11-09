# System Architecture Details

## 1. Embedded Wearable Tier
*   **Runtime Environment:** Real-time operating system (FreeRTOS) with prioritized task scheduling.
*   **Asynchronous Execution Model:**
    - **Capture Task:** Dedicated execution thread managing sensor acquisition and direct memory transfers into partitioned memory buffers.
    - **Communication Task:** Independent execution thread managing authenticated local wireless communication, command handling, and high-throughput buffer offloading.

---

## 2. Companion Mobile Application Tier
*   **Framework:** Cross-platform Flutter architecture supporting responsive mobile execution.
*   **State Management:** Reactive state container managing hardware connectivity, media offload pipelines, and interactive timeline rendering.
*   **Media Processing Engine:** Client-side media subsystem that reconciles raw sensory buffers into standard container formats for timeline preview and cloud archiving.
*   **Secure Local Storage:** Encrypted application cache storing timeline metadata and recent capture sessions.

---

## 3. Cognitive AI & Retrieval Tier
*   **Multimodal AI Engine:** Orchestrates multimodal foundation models for scene understanding and event indexing.
*   **Cognitive Services:**
    - **Multimodal Synthesis:** Generates concise titles, contextual summaries, and visual descriptions from joint audio-visual data.
    - **Semantic Representation:** Constructs vector embeddings and structured timeline metadata.
    - **Conversational Retrieval Engine:** Resolves natural language user queries by matching intent against personal timeline records.

---

## 4. Personal Cloud & Archival Tier
*   **User-Centric Storage:** Direct integration with the user's private cloud storage infrastructure (e.g., Google Drive), ensuring third-party zero-knowledge data privacy.
*   **Transactional Offload Pipeline:** Resilient background queue ensuring verifiable data transfer and zero-corruption synchronization across varying network conditions.
