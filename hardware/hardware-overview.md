# Hardware Architecture: Wearable Capture Unit

## 1. Core Component Specifications
The SAGE wearable unit is built upon an ultra-compact, energy-efficient system-on-chip optimized for concurrent visual and auditory acquisition:

| Subsystem | Component Specification | Role in System Architecture |
| :--- | :--- | :--- |
| **Microcontroller** | Dual-core 32-bit SoC with integrated Wi-Fi/BLE and external PSRAM | Coordinates sensor acquisition, media buffering, and secure offloading |
| **Image Sensor** | Low-power VGA CMOS Image Sensor (OV2640) | Captures contextual visual frames |
| **Microphone** | Digital PDM Microphone (Direct Memory Interfacing) | Captures synchronized ambient audio |
| **Expansion Storage** | High-speed SPI/SD Interface | Local buffer caching and persistent offline storage |

---

## 2. Sensor Interfacing & Concurrency
The wearable firmware decouples high-bandwidth perceptual data acquisition from external network communication:

*   **Visual Subsystem:** High-speed parallel interface communicating with the image sensor, configured for optimized frame capture while preserving device memory bounds.
*   **Auditory Subsystem:** Digital audio interface capturing ambient audio streams into isolated memory blocks to prevent bus contention with visual acquisition tasks.
*   **Peer-to-Peer Communication:** Establishes a local, authenticated wireless channel enabling the companion mobile application to discover the wearable, monitor device status, and offload media buffers.

---

## 3. Thermal & Power Optimization
Wearable form factors require stringent power and thermal management:
*   **Dynamic Power States:** Peripheral sensors and high-power radio channels enter low-power sleep states during idle capture intervals.
*   **Concurrent Task Separation:** Asynchronous multi-core task scheduling isolates time-critical sensor sampling from network transmission, preventing processing spikes and minimizing thermal dissipation.
