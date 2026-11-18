# Hardware Architecture: Wearable Capture Unit

## 1. Core Component Specifications
The current SAGE wearable prototype is built on the **Seeed Studio XIAO ESP32-S3 Sense**, selected for its compact footprint, integrated camera/PDM microphone shield, and external PSRAM:

| Subsystem | Component Specification | Role in System Architecture | Status |
| :--- | :--- | :--- | :--- |
| **Microcontroller** | ESP32-S3R8 (Xtensa LX7 dual-core @ 240 MHz, 8 MB PSRAM, 8 MB Flash, 2.4 GHz Wi-Fi) | Coordinates sensor acquisition, local SoftAP hosting, and dual-port streaming | **Implemented** |
| **Image Sensor** | OV2640 CMOS camera (`640×480` VGA JPEG, double-buffered in PSRAM) | Captures live viewfinder & session visual frames | **Implemented** |
| **Microphone** | Integrated digital PDM MEMS microphone (`I2S_NUM_0`, `16 kHz`, 16-bit mono, pins GPIO 41/42) | Captures ambient & speech audio with 8x digital gain + clipping guard | **Implemented** |
| **Enclosure & Power** | Custom compact battery enclosure & dynamic peripheral sleep-state power optimization | Extended all-day wearable operation | **Planned** |

---

## 2. Sensor Interfacing & Dual-Core Concurrency (Firmware v3.0.0)
To eliminate wireless and capture bus contention on a constrained microcontroller:

* **Core 1 — HTTP Control & Visual Pipeline (Port `80`):**
  * Hosts Wi-Fi SoftAP (`SAGE-XIAO`, channel 6, up to 4 clients).
  * Serves `/health` (6-second phone heartbeat monitor), `/status` (free heap & PSRAM diagnostics), and `/live_frame` (directly streams the OV2640 JPEG DMA buffer from PSRAM).
* **Core 0 — FreeRTOS Audio Streaming Task (Port `81`):**
  * Dedicated FreeRTOS task pinned to Core 0 reads I2S PDM audio continuously and writes raw 16-bit PCM frames to the connected TCP audio socket with `TCP_NODELAY` enabled.
* **Memory Budget:**
  * Double-buffered VGA JPEG DMA frames consume `~200 KB` out of `8 MB` PSRAM, leaving `~7.8 MB` headroom for TCP/HTTP stacks and FreeRTOS tasks.

---

## 3. Design Trade-Offs & Limitations
* **No On-Device Muxing or ML:** Audio-visual container muxing (RIFF/AVI) and multimodal AI inference are deliberately deferred to the companion smartphone and cloud layers to preserve wearable simplicity and avoid PSRAM fragmentation.
* **Constant 8x Gain Limitation:** Firmware currently applies a fixed 8x digital gain with hard clipping protection (`[-32768, 32767]`); adaptive noise suppression is scoped for future revisions.
* **Continuous AP Mode:** While powered, the prototype runs in active `WIFI_AP` mode; deep-sleep duty cycling and physical enclosure integration are planned hardware milestones.
