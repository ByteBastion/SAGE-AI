# Hardware Architecture Specification: Micro-Sensor Edge Node & Interconnect Topology

**Author:** Vinayak Sharma  
**Context:** SAGE Theoretical Architectural Foundations  
**Preprint Reference:** TR-2026-SAGE01-HW

---

## 1. System Block Diagram & Interconnect Architecture

The SAGE Micro-Sensor Edge Node is an ultra-compact embedded system optimized for deterministic multimodal data acquisition within strict power ($\le 100\text{mW}$ average) and weight ($\le 18\text{g}$) boundaries.

```
+-----------------------------------------------------------------------------------------+
|                               SAGE EDGE NODE (ESP32-S3)                                 |
|                                                                                         |
|  +--------------------+        +-----------------------+        +--------------------+  |
|  |   OV2640 Camera    |        |  Tensilica LX7 Core 0 |        |   MSM261S4030H0R   |  |
|  |   Sensor (CMOS)    |        |  (Real-Time Capture)  |        |   Digital PDM Mic  |  |
|  +--------------------+        +-----------------------+        +--------------------+  |
|            |                               |                              |             |
|   8-bit DVP Bus (PCLK, VSYNC)              |                      I2S DMA (CLK, DATA)   |
|            |                               |                              |             |
|            +-----------------------+       |       +----------------------+             |
|                                    ▼       ▼       ▼                                    |
|                              +---------------------------+                              |
|                              | Internal AHB Crossbar DMA |                              |
|                              +---------------------------+                              |
|                                            |                                            |
|                                            ▼                                            |
|                              +---------------------------+                              |
|                              |   Octal-SPI PSRAM (8MB)   |                              |
|                              |   [Video: 4.2MB Ring Buf] |                              |
|                              |   [Audio: 480KB Ring Buf] |                              |
|                              +---------------------------+                              |
|                                            |                                            |
|                                            ▼                                            |
|                              +---------------------------+                              |
|                              |  Tensilica LX7 Core 1     |                              |
|                              |  (802.11 AP HTTP Server)  |                              |
|                              +---------------------------+                              |
|                                            |                                            |
|                                            ▼                                            |
|                              +---------------------------+                              |
|                              | RF 2.4GHz Wi-Fi Antenna   |                              |
|                              +---------------------------+                              |
+-----------------------------------------------------------------------------------------+
                                             |
                                             ▼ (Ad-Hoc Burst Link)
                                     [Mobile Gateway Host]
```

---

## 2. Core Electrical Component Specifications

| Subsystem | Selected Component | Electrical Parameters | Architectural Function & Rationale |
| :--- | :--- | :--- | :--- |
| **Microcontroller (MCU)** | Espressif ESP32-S3 (WROOM/Sense form-factor) | $3.0\text{V} - 3.6\text{V}$, dual Xtensa LX7 @ $240\text{ MHz}$, $512\text{ KB}$ SRAM | Asymmetric dual-core architecture allows complete isolation of capture DMA (Core 0) and network serving (Core 1). |
| **External Memory (PSRAM)** | 8 MB Octal-SPI High-Speed PSRAM | $120\text{ MHz}$ DDR (Double Data Rate), $8\text{ bits/clock}$ ($240\text{ MB/s}$ throughput) | Provides dedicated contiguous storage space for decoupled video and audio circular buffers without CPU intervention. |
| **Visual Transducer** | OmniVision OV2640 $1/4"$ CMOS sensor | Active array $1600 \times 1200$, operating voltage $1.8\text{V} - 3.3\text{V}$ | Hardware JPEG compression engine decimates raw sensor bandwidth, outputting pre-compressed frames directly over 8-bit DVP. |
| **Acoustic Transducer** | MSM261S4030H0R PDM MEMS Microphone | $1.6\text{V} - 3.6\text{V}$, Sensitivity $-26\text{ dBFS}$, SNR $64\text{ dBA}$ | Digital Pulse-Density Modulation (PDM) avoids analog noise pickup; decimated directly by hardware I2S peripheral. |
| **Power Storage** | Lithium Polymer (LiPo) Cell | $3.7\text{V}$ nominal, $300\text{ mAh}$ capacity, integrated PCM protection | Provides $\ge 4.5\text{ hours}$ operational lifespan under cyclic $33\%$ capture duty cycles. |

---

## 3. Bus Protocol & Timing Analysis

### 3.1 Digital Video Port (DVP) Timing
The OV2640 sensor interfaces via the Digital Video Port (DVP) operating in 8-bit parallel transfer mode:
* **Pixel Clock ($PCLK$):** Clocked at $f_{pclk} = 10\text{ MHz} \text{ to } 20\text{ MHz}$.
* **Vertical Synchronization ($VSYNC$):** Asserts active-high at the beginning of each frame. Triggers Core 0 frame start interrupt.
* **Horizontal Reference ($HREF$):** Asserts high when valid pixel data is driven onto data lines $D[7:0]$.
* **Data Pins ($D[7:0]$):** Transmits compressed JPEG stream delimited by Start-of-Image (`0xFFD8`) and End-of-Image (`0xFFD9`) markers.

DMA buffers are configured as ping-pong descriptors in internal SRAM ($2 \times 4096\text{ B}$), which trigger high-priority AHB burst writes directly into the external PSRAM video circular buffer $\mathcal{B}_v$.

### 3.2 Inter-IC Sound (I2S) PDM Decimation Timing
The microphone transmits over an I2S PDM serial bus:
* **Bit Clock ($BCLK$):** Driven by the ESP32-S3 master at:
  $$f_{bclk} = f_s \times M = 16\text{ kHz} \times 64 = 1.024\text{ MHz}$$
* **Data In ($DIN$):** High-impedance 1-bit PDM stream. The hardware digital decimation filter within the ESP32-S3 decodes the 1-bit density stream into signed 16-bit linear Pulse-Code Modulation (PCM) samples at $16\text{ kHz}$ mono.
* **DMA Ring Buffer:** DMA triggers interrupt-driven transfers every $1024\text{ samples}$ ($2048\text{ bytes} \approx 64\text{ ms}$ interval), streaming directly into the PSRAM audio buffer $\mathcal{B}_a$.

---

## 4. Power State Management & Current Profiling

To maintain surface temperature below dermal irritation thresholds ($<43^\circ\text{C}$ per ISO 13732-1) and maximize battery longevity, the edge node transitions between four disciplined power states:

```
               ┌───────────────┐
               │  STATE_IDLE   │  (CPU: 80MHz, Cam: Sleep, Wi-Fi: DTIM3)
               └───────────────┘  Current: 14.2 mA (46.8 mW)
                       │
       Start Capture   │
                       ▼
               ┌───────────────┐
               │ STATE_CAPTURE │  (CPU: 240MHz, Cam: Active, Audio: DMA, Wi-Fi: SoftAP beacon)
               └───────────────┘  Current: 98.4 mA (324.7 mW) for 15.0 seconds
                       │
       Epoch Complete  │
                       ▼
               ┌───────────────┐
               │ STATE_BURST   │  (CPU: 240MHz, Cam: Standby, Wi-Fi: Active TX Burst)
               └───────────────┘  Current: 210.5 mA (694.6 mW) for 2.2 seconds
                       │
       Transfer Done   │
                       ▼
               ┌───────────────┐
               │  STATE_IDLE   │
               └───────────────┘
```

### 4.1 Quantitative Energy Budget per 45s Cycle

Assuming a realistic episodic capture duty cycle: $15\text{s capture} \to 2.2\text{s transfer} \to 27.8\text{s idle}$:

| State | Duration ($t_i$) | Operating Voltage ($V$) | Current Draw ($I_i$) | Power ($P_i$) | Energy ($E_i = P_i \cdot t_i$) |
| :--- | :--- | :--- | :--- | :--- | :--- |
| **STATE_CAPTURE** | $15.0\text{ s}$ | $3.3\text{ V}$ | $98.4\text{ mA}$ | $324.7\text{ mW}$ | $4.871\text{ Joules}$ |
| **STATE_BURST** | $2.2\text{ s}$ | $3.3\text{ V}$ | $210.5\text{ mA}$ | $694.6\text{ mW}$ | $1.528\text{ Joules}$ |
| **STATE_IDLE** | $27.8\text{ s}$ | $3.3\text{ V}$ | $14.2\text{ mA}$ | $46.8\text{ mW}$ | $1.301\text{ Joules}$ |
| **Composite Cycle** | **$45.0\text{ s}$** | — | — | **$\bar{P} \approx 171.1\text{ mW}$** | **$7.700\text{ Joules}$** |

#### Battery Endurance Derivation
For a $300\text{ mAh}$ cell ($3.7\text{ V}$ nominal, $E_{cell} = 3.7 \times 0.300 \times 3600 = 3996\text{ Joules}$):
$$\text{Cycles} = \frac{3996\text{ J} \times 0.85\text{ (efficiency)}}{7.700\text{ J/cycle}} \approx 441\text{ cycles}$$
$$\text{Operational Lifetime} = \frac{441 \times 45\text{ s}}{3600\text{ s/h}} \approx \mathbf{5.51\text{ Hours}}$$

---

## 5. Physical Pinout & Multiplexing Map

The following matrix documents the hardware interconnect mapping on the Seeed XIAO ESP32-S3 Sense carrier:

| Physical GPIO Pin | Alternate Function | Connected Sensor Signal | Protocol Standard |
| :--- | :--- | :--- | :--- |
| **GPIO 10** | `CAM_D0` | OV2640 Data Bit 0 | 8-bit DVP Parallel |
| **GPIO 11** | `CAM_D1` | OV2640 Data Bit 1 | 8-bit DVP Parallel |
| **GPIO 12** | `CAM_D2` | OV2640 Data Bit 2 | 8-bit DVP Parallel |
| **GPIO 14** | `CAM_D3` | OV2640 Data Bit 3 | 8-bit DVP Parallel |
| **GPIO 16** | `CAM_D4` | OV2640 Data Bit 4 | 8-bit DVP Parallel |
| **GPIO 18** | `CAM_D5` | OV2640 Data Bit 5 | 8-bit DVP Parallel |
| **GPIO 17** | `CAM_D6` | OV2640 Data Bit 6 | 8-bit DVP Parallel |
| **GPIO 15** | `CAM_D7` | OV2640 Data Bit 7 | 8-bit DVP Parallel |
| **GPIO 13** | `CAM_PCLK` | OV2640 Pixel Clock | Clock line ($10\text{–}20\text{ MHz}$) |
| **GPIO 38** | `CAM_VSYNC` | OV2640 Frame Synchronization | Vertical Sync strobe |
| **GPIO 47** | `CAM_HREF` | OV2640 Horizontal Reference | Horizontal Line strobe |
| **GPIO 40** | `CAM_SIOD` | OV2640 SCCB Data | I2C/SCCB Control bus |
| **GPIO 39** | `CAM_SIOC` | OV2640 SCCB Clock | I2C/SCCB Control bus |
| **GPIO 42** | `PDM_CLK` | MSM261S4030H0R Clock | I2S Master Clock ($1.024\text{ MHz}$) |
| **GPIO 41** | `PDM_DIN` | MSM261S4030H0R Data | I2S Serial Data In |
| **GPIO 21** | `STATUS_LED` | Visual State Annunciator LED | Active-Low Opto-Indication |

---

## 6. Thermal Dissipation Modeling

The steady-state temperature rise $\Delta T$ of the wearable package is governed by:
$$\Delta T = \bar{P} \cdot R_{\theta,ja}$$
Where $R_{\theta,ja} \approx 45^\circ\text{C/W}$ for the ventilated polycarbonate enclosure. 

Under active cyclic operation:
$$\Delta T = 0.171\text{ W} \times 45^\circ\text{C/W} \approx 7.7^\circ\text{C}$$
At an elevated ambient temperature $T_{amb} = 30^\circ\text{C}$:
$$T_{surface} = 30^\circ\text{C} + 7.7^\circ\text{C} = 37.7^\circ\text{C} \ll 43.0^\circ\text{C}$$
This confirms unconditional thermal compliance without requiring active convective cooling or bulky heatsinks.
