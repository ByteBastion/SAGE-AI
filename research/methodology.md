# Methodology: Asynchronous Capture and Semantic Recall

## 1. Asynchronous Capture Lifecycle
SAGE operates on an event-driven state architecture designed to minimize wearable energy consumption while providing comprehensive sensory documentation:

```
[Standby / Viewfinder]
        │
        ▼ (Session Trigger)
[Isolated Sensor Capture] ── Concurrent audio/visual acquisition into bounded memory
        │
        ▼ (Capture Interval Complete)
[Peer-to-Peer Offload] ──── Low-latency media transfer to companion mobile client
        │
        ▼ (Client-Side Ingestion)
[Media Synthesis & Cache] ─ Client-side timeline reconciliation and local indexing
        │
        ▼ (Continuous Monitoring)
[Next Capture Decision] ── Evaluates context to trigger subsequent capture cycle
```

---

## 2. Multi-Channel Stream Reconciliation
To resolve the processing limitations of low-power wearables:
*   **Independent Buffer Acquisition:** Sensor streams are acquired into isolated memory buffers, avoiding on-device container overhead and frame-dropping.
*   **Metadata Synchronization:** Temporal markers captured during the acquisition window are transmitted alongside media buffers.
*   **Client-Side Ingestion:** The companion client ingests the parallel streams, using temporal metadata to generate synchronized media archives without taxing the wearable processor.

---

## 3. Cognitive Synthesis & Semantic Structuring
1.  **Multimodal Analysis:** Synchronized media segments are submitted to the cognitive AI pipeline for contextual parsing.
2.  **Semantic Structuring:** The model extracts:
    -   A descriptive event title for chronological indexing.
    -   A concise contextual narrative of activities, objects, and participants.
    -   Dense semantic descriptors enabling fine-grained vector similarity search.
3.  **Local Timeline Database:** Extracted metadata is indexed within a localized timeline database for immediate on-device querying.
4.  **Private Cloud Archival:** Media segments and associated metadata are encrypted and backed up to user-owned private cloud storage.

---

## 4. Conversational Retrieval-Augmented Generation (RAG)
SAGE implements an intelligent retrieval pipeline for natural language query resolution:
1.  **Query Parsing:** Translates natural language questions into semantic search vectors.
2.  **Contextual Matching:** Identifies top relevant timeline events using semantic and temporal filtering.
3.  **Grounded Synthesis:** Supplies retrieved event context to the language model to generate accurate, hallucination-free answers.
4.  **Interactive Evidence:** Couples conversational responses with direct navigation to the corresponding temporal media segment.
