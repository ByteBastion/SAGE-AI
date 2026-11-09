# Gap Analysis: Resource Constraints & Synchronization in Wearables

## 1. Introduction
While multimodal memory indexing is conceptually straightforward on high-performance desktop or mobile hardware, bringing continuous audio-visual capture to low-power, wearable microcontroller platforms introduces unique engineering bottlenecks.

---

## 2. Identified Technical Gaps

### Gap A: Memory Exhaustion in Single-Buffer Architectures
On-chip memory (SRAM/PSRAM) on ultra-low-power microcontrollers is heavily restricted (typically 8MB or less of external PSRAM). 
*   **The Problem:** Storing a combined audio-video stream in a single unified buffer causes immediate heap fragmentation and memory exhaustion within seconds of continuous recording.
*   **The Impact:** Most existing commercial wearable solutions rely on streaming raw frames to the smartphone continuously. This quickly drains device battery and causes packet drops over wireless links.

### Gap B: Asynchronous Drift and Frame Dropping
When trying to capture high-speed camera frames and digital audio samples concurrently on compact edge chips, bus contention arises:
*   **The Problem:** Camera frame capture takes variable time depending on ambient lighting and compression complexity. Auditory transactions are time-critical and must run without interruption.
*   **The Impact:** Monolithic single-threaded architectures either drop audio samples (causing acoustic artifacts) or drop camera frames (causing visual stuttering), leading to significant drift over time.

---

## 3. The SAGE Solution: Decoupled Multi-Channel Architecture
SAGE addresses these gaps by implementing a **Decoupled Multi-Channel Pipeline**:
1.  **Partitioned Buffering:** Audio and video streams are acquired into isolated, non-contiguous memory blocks during capture.
2.  **Asynchronous Multi-Task Execution:** Visual frame acquisition and audio collection operate as prioritized, independent real-time tasks.
3.  **Downstream Synchronization:** Rather than muxing on the resource-constrained microcontroller, sensory streams are transferred independently and synchronized natively on the companion mobile client.
