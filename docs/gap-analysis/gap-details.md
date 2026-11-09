# Deep Gap Analysis: Wearable Systems Constraints

## 1. Single-Core Processing Bottleneck
Most affordable IoT development boards run visual processing and networking on a single core:
*   **Contention:** Camera driver interrupts disrupt TCP network task scheduling. This leads to dropped frames during streaming or packet timeout failures.
*   **SAGE Solution:** SAGE decouples high-bandwidth camera acquisition from network server execution through asynchronous task scheduling, preventing core locks and ensuring stable streaming.

---

## 2. Combined Muxing Overhead
Microcontrollers do not have the processing power to mux MP4 container files:
*   **Overhead:** Performing H.264 compression or AAC encoding requires heavy floating-point math and dedicated codecs.
*   **SAGE Solution:** SAGE uses a decoupled dual-buffer capture pipeline, deferring the processing overhead. Muxing and transcoding are handled downstream on the companion mobile client.
