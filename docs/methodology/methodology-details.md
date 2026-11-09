# Media Synchronization Methodology

## 1. Multi-Sensor Edge Acquisition
Capturing continuous audio and video on ultra-low-power microcontrollers introduces significant resource challenges:
*   **Variable Frame Latency:** Hardware bus constraints and environmental lighting variations cause dynamic frame-rate fluctuations during visual capture.
*   **Continuous Auditory Sampling:** Auditory acquisition requires strictly uninterrupted sampling to prevent perceptual clipping or artifacts.
*   **Resource Preservation:** Traditional container encoding (such as MP4 muxing) exceeds microcontroller memory and computational capacity. SAGE isolates these acquisition streams during the recording window, capturing raw temporal buffers without on-device encoding bottlenecks.

---

## 2. Temporal Metadata Binding
Rather than performing computationally expensive container muxing on the wearable unit:
*   The firmware logs temporal alignment metadata alongside raw sensor buffers during acquisition.
*   Visual and auditory streams are packaged with structural framing metadata to establish precise temporal correspondence.
*   Media streams are preserved in lightweight intermediate structures ready for rapid peer-to-peer transmission.

---

## 3. Client-Side Reconstruction & Playback
Once offloaded to the companion mobile application, the processing layer reconciles the decoupled sensory streams:
1.  **Temporal Reconciliation:** The client parses temporal markers from the media headers to determine real-world duration and effective frame intervals.
2.  **Display Time Decoupling:** The playback interface decouples rendering refresh rates from the sensor capture frequency, dynamically interpolating frames to present smooth, continuous playback.
3.  **Proportional Multi-Stream Seeking:** Timeline navigation synchronizes visual indices and auditory positions proportionally, maintaining coherent audio-visual alignment throughout user interaction.
