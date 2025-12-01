# Ultra-Low Jitter 30-Channel Audio Distribution System
## Technical Design Report

---

## Executive Summary

This report describes a practical, cost-effective architecture for distributing 30 channels of ultra-low jitter audio (96 kHz, 24-bit) from a host PC to amplifiers via USB and SPI. The system leverages commodity components (Raspberry Pi 4, Raspberry Pi Pico 2) paired with a high-grade NDK crystal oscillator to achieve **femtosecond-level jitter performance** while maintaining DSP processing capability and easy programmability.

**Key Innovation:** A "Jitter Firewall" architecture that completely isolates the sensitive audio clocks from the noisy digital processing domain, using the RP2350's Programmable I/O (PIO) to decouple timing domains.

**Performance vs. XMOS:** 3x greater processing power, native High-Bandwidth USB (3x transactions), and the ability to apply room correction and DSP to all 30 channels simultaneously.

---

## Section 1: Bill of Materials

### 1.1 Primary Components

| Component | Part Number | Qty | Unit Price | Total | Notes |
|-----------|------------|-----|-----------|-------|-------|
| Raspberry Pi 4B (8GB) | RPi4-8GB | 1 | [$48.20](https://www.digikey.com/en/products/detail/raspberry-pi/SC0193-9/10258782) | $48.20 | USB audio host, SPI master, DSP engine |
| Raspberry Pi Pico 2 | RP2350 | 3 | [$5.00](https://www.digikey.com/en/products/detail/raspberry-pi/SC1631/24627136) | $15.00 | Audio re-clockers (10ch each) |
| NDK Crystal Oscillator | NSC5083D-49.152M | 1 | [$2.83](https://www.digikey.com/en/products/detail/ndk-nihon-dempa-kogyo-co-ltd/NSC5083D-49-152M/13982350) | $2.83 | Ultra-low jitter reference (43 fs) |
| **Subtotal (Primary)** | | | | **$66.03** | |

### 1.2 Crystal Oscillator Support Components

The NDK NSC5083D is a bare crystal resonator. To create a functional 49.152 MHz clock source with low-jitter fanout capability:

| Component | Value/Type | Qty | Unit Price | Total | Purpose |
|-----------|-----------|-----|-----------|-------|---------|
| **Pierce Oscillator** | | | | | |
| Capacitor (Load) | 10 pF | 2 | $0.05 | $0.10 | Crystal load capacitance |
| Capacitor (Bypass) | 100 nF | 1 | $0.02 | $0.02 | Power supply decoupling |
| Resistor (Feedback) | 2.2 MΩ | 1 | $0.08 | $0.08 | Pierce oscillator feedback |
| Capacitor (Supply) | 100 nF | 4 | $0.02 | $0.08 | Buffer power decoupling |
| Clock Buffer IC | TI LMK1C1108APWR | 1 | [$3.11](https://www.digikey.com/en/products/detail/texas-instruments/LMK1C1108APWR/26812430) | $3.11 | 1-to-8 LVCMOS fanout (<50 fs jitter) |
| Single-Layer PCB | 50mm × 50mm | 1 | $6.00 ([introductory rate](https://www.allpcb.com/)) | $6.00 | Crystal PCB |
| **Subtotal (Crystal)** | | | | **$9.57** | |

### 1.3 Power Supply Components

| Component | Part Number | Qty | Unit Price | Total | Notes |
|-----------|------------|-----|-----------|-------|-------|
| 5V Logic Supply | Mean Well RS-15-5 | 1 | $25.00 | $25.00 | 5A, SMPS, Pi 4 & Pico main power |
| 3.3V Ultra-Low Noise LDO | LT3045 Module | 1 | $12.00 | $12.00 | Analog audio rail (< 1 µVrms) |
| **Subtotal (Power)** | | | | **$37.00** | |

### 1.4 Total Bill of Materials

| Category | Total |
|----------|-------|
| **Primary Electronics** | $66.03 |
| **Crystal Oscillator Circuit** | $9.57 |
| **Subtotal (vs. XMOS)** | $75.60 |
| **Power Supplies & Regulators** | $37.00 |
| **GRAND TOTAL** | **$112.60** |

---

## Section 2: RP2350 (Pico 2) Hardware & Firmware Architecture

### 2.1 Dual-Clock Strategy (The "Jitter Firewall")

Each Pico 2 implements two completely independent clock domains to isolate audio timing from digital processing noise.

**Domain A: System Clock (150 MHz, Noisy)**
- **Source:** Internal PLL locked to onboard 12 MHz crystal, or alternatively PLL locked to the external clock (not sure it would do anything).
- **Purpose:** Drives CPU, DMA, and SPI Slave logic.
- **Jitter:** ~150 ps (irrelevant for digital data).
- **Benefit:** Massive bandwidth to handle 18+ Mbps SPI input streams.

**Domain B: Audio Clock (49.152 MHz, Ultra-Pure)**
- **Source:** External NDK Crystal (fed into GPIO 20).
- **Purpose:** Clocks the PIO Audio State Machines directly.
- **Jitter:** ~43 fs (Preserved from crystal).
- **Mechanism:** PIO is configured to use GPIO 20 as its clock input, bypassing the system clock entirely.
- **Result:** The DACs see only the crystal's jitter; they are immune to USB/SPI activity.

### 2.2 Hardware Pinout Configuration

| Pin # | GPIO | Function | Direction | Connection |
|-------|------|----------|-----------|------------|
| **SPI Interface** | | | | |
| 1 | GP0 | SPI RX (MOSI) | In | From Pi 4 SPI Bus |
| 2 | GP1 | SPI CS | In | From Pi 4 Chip Select |
| 3 | GP2 | SPI SCK | In | From Pi 4 SCLK (25+ MHz) |
| 4 | GP3 | SPI TX (MISO) | Out | To Pi 4 (Feedback) |
| **Audio Clock** | | | | |
| 26 | GP20 | **AUDIO_REF_CLK** | In | From NDK Crystal (49.152 MHz) |
| **Synchronization** | | | | |
| 27 | GP21 | **GLOBAL_SYNC** | In | From Pi 4 GPIO (Start trigger) |
| **DAC Shared Clocks** | | | | |
| 9 | GP6 | **DAC_BCLK** | Out | To all DACs (6.144 MHz) |
| 10 | GP7 | **DAC_LRCK** | Out | To all DACs (96 kHz) |
| **DAC Audio Data** | | | | |
| 11 | GP8 | I2S_DATA_0 | Out | To DAC 1 (Ch 1-2) |
| 12 | GP9 | I2S_DATA_1 | Out | To DAC 2 (Ch 3-4) |
| 13 | GP10 | I2S_DATA_2 | Out | To DAC 3 (Ch 5-6) |
| 14 | GP11 | I2S_DATA_3 | Out | To DAC 4 (Ch 7-8) |
| 15 | GP12 | I2S_DATA_4 | Out | To DAC 5 (Ch 9-10) |
| **Flow Control** | | | | |
| 6 | GP4 | **BUF_LEVEL** | Out | To Pi 4 GPIO (Interrupt) |

This uses 14 GPIO pins total, leaving plenty of room for future expansion, additional features, or moving pins around to avoid noise.

### 2.3 Programmable I/O (PIO) Implementation

Each Pico 2 runs 3 PIO state machines (out of 12 available across 3 PIO blocks):

**PIO Block 0: SPI Slave (Input Path)**
- **Clock Source:** System Clock (150 MHz).
- **Function:** Receives data from Pi 4 at ~20 MHz.
- **Buffering:** Pushes 32-bit words to a FIFO. **DMA Channel A** drains the FIFO into a large circular buffer in SRAM (64 KB ring).
- **Why PIO?** Standard hardware SPI buffers are too shallow (8 words). PIO + DMA creates a virtually infinite buffer.

**PIO Block 1: I2S Master (Output Path)**
- **Clock Source:** External NDK Pin (GP20 / 49.152 MHz).
- **Resources:** 6 State Machines:
  - **SM0 (Clock Gen):** Divides 49.152 MHz by 8 (BCLK: 6.144 MHz) and by 512 (LRCK: 96 kHz). Outputs to GP6/GP7.
  - **SM1-SM5 (Data Feeders):** Each feeds one I2S Data line (GP8-GP12). **DMA Channel B** supplies the audio data from the SRAM ring buffer.
- **Synchronization:** All 6 State Machines are started simultaneously using `pio_enable_sm_mask_in_sync()`, triggered by GLOBAL_SYNC.
- **Timing:** Every instruction executes in *exactly* one cycle of the 49.152 MHz clock. The output jitter is identical to the crystal's jitter plus negligible gate delay.

### 2.4 Buffer Management

The SRAM Ring Buffer acts as an elastic connector between two asynchronous domains:
- **Input (Bursty SPI):** Data arrives at ~20 Mbps in bursts.
- **Output (Constant I2S):** Data drains at exactly 9.216 Mbps (constant, paced by NDK crystal).

**Flow Control:**
1. The main CPU monitors the Ring Buffer's read/write pointers.
2. **If Level > 75%:** Toggle GPIO 4 HIGH. The Pi 4 sees this interrupt and commands the PC to slow down (via USB Explicit Feedback).
3. **If Level < 25%:** Toggle GPIO 4 LOW. The Pi 4 commands the PC to speed up.
4. **Result:** Buffers stay ~50% full. No sample loss, and the entire system locks to the NDK crystal frequency.

---

## Section 3: Raspberry Pi 4 Host Architecture

### 3.1 USB High-Bandwidth Isochronous Interface

The Pi 4 acts as a USB Audio Device (Gadget Mode) to the host PC/Phone.

**Data Load:**
- 30 Channels × 24-bit × 96 kHz = **8.64 MB/s** (69.12 Mbps).
- Bytes per microframe (125 µs): **1080 bytes**.

**Bandwidth Requirement:**
Standard High-Speed Isochronous endpoints limit 1024 bytes/microframe. Since 1080 > 1024, we must use **High-Bandwidth Mode** (multiple transactions per microframe).

**Configuration:**
- Set endpoint descriptor `wMaxPacketSize` to support **2 transactions per microframe** (up to 2048 bytes).
- This provides **47% headroom** for reliable packet transmission.
- **Bus Utilization:** 1080 / 3072 (max 3x capacity) = **35%**.

**Critical Implementation Detail:** The standard Linux `f_uac2` gadget driver often defaults to standard isochronous (1024 bytes). You must patch or configure it to enable High-Bandwidth mode via ConfigFS endpoints.

### 3.2 Multi-Bus SPI Distribution

To prevent contention, we use three separate SPI controllers on the BCM2711 SoC, one for each Pico 2:

| Pico Unit | SPI Controller | Pi 4 GPIO Pins | Throughput | Headroom |
|-----------|----------------|----------------|-----------|----------|
| **A** | SPI0 | GPIO 9/10/11 (CS: GPIO 8) | ~23 Mbps | ~8% |
| **B** | SPI4 | GPIO 19/20/21 (CS: GPIO 18) | ~23 Mbps | ~8% |
| **C** | SPI5 | GPIO 13/14/15 (CS: GPIO 12) | ~23 Mbps | ~8% |

**Clock Speed:** 25 MHz target. Can be overclocked to 30-35 MHz safely, as the Pico 2 (150 MHz system clock) samples reliably up to ~37 MHz.

### 3.3 CPU Utilization & DSP Headroom

**I/O Overhead:**
- USB Interrupts (8000/sec): ~5-8% of one core.
- SPI DMA: ~2-3% of one core (mostly hardware-driven).

**DSP Processing Capacity:**
The Pi 4 can run complex audio filters (30 channels of 10-biquad IIR filters + parametric EQ). Projected load: **1.5–2 full cores** (out of 4).

**Total System Load:**
- **Core 0:** I/O + Housekeeping (~25%).
- **Cores 1–2:** DSP (Channels 1-30) (~80% each).
- **Core 3:** Free (User Interface / Control / Headroom).

**Conclusion:** The Pi 4 can easily handle this workload while maintaining significant processing headroom for advanced features like convolution-based room correction.

### 3.4 Synchronization & Explicit Feedback Loop

Since the PC and local NDK crystals are asynchronous, they drift relative to each other (~50 ppm typical).

**Mechanism:**
1. Each Pico monitors its Ring Buffer level.
2. It signals the Pi 4 via GPIO interrupt when the buffer drifts from target (50%).
3. A Linux user-space daemon aggregates these signals and calculates a composite "drift value."
4. The daemon writes to the UAC2 gadget driver's feedback endpoint.
5. The PC receives feedback and adjusts its transmission rate (e.g., "Send 48.002 kHz instead of 48.000 kHz").
6. The system locks to the average speed of the NDK crystals without ever dropping a sample.

---

### Section 3: Crystal Oscillator Design

**Topology:**
- LT3045 Ultra-Low Noise LDO Power Supply
- Pierce Oscillator with NDK NSC5083D Crystal
- LMK1C110x Clock Buffer

**Why This Approach:**
- The NDK NSC5083D is a high-Q AT-cut crystal with excellent phase noise (~–150 dBc/Hz @ 10kHz offset).
- A Pierce oscillator is the lowest-noise topology for this frequency range.
- The LMK1C110x provides jitter-free fanout to the Picos and DACs.
- On-board regulation with LT3045 ensures ultra-clean power, minimizing phase noise degradation.

**Pinout Description:**
The LT3045 LDO takes 5V input and provides a clean 3.3V rail to the Pierce oscillator and LMK1C110x. It also provides external power for the 3.3V DACs.
The LMK1C110x clock buffer drives:
- Pico 1 (GPIO 20)
- Pico 2 (GPIO 20)
- Pico 3 (GPIO 20)
- DAC SCK Pins (for direct connection)

---

## Section 4: Interconnect Signal Integrity

### 4.1 Layout Strategy: Compact Integration (Recommended)

**Physical Layout:** All boards (Pi 4, 3x Pico 2, Crystal PCB) housed in a single enclosure (~15 cm × 15 cm).

**Interconnect:** Short jumper wires (dupont cables) or custom "Motherboard" PCB that seats the modules.

**Signal Integrity:**

**SPI Bus (High-Speed Digital):**
- Target Speed: 25–30 MHz.
- Critical Length (before reflections): ~15–20 cm.
- **Verdict:** With 10 cm jumpers, you are safely below transmission-line limits.
- **Recommendation:** Use **twisted-pair wiring** (Signal + GND) derived from old Ethernet cable. Keep length < 15 cm.

**Audio Clock (NDK Crystal Signal):**
- **Sensitivity:** Ultra-High (43 fs jitter reference).
- **Constraint:** Total capacitive loading < 30 pF to maintain fast edges.
- **Wire Limit:** Standard ribbon cable is ~50 pF/meter; 10 cm adds ~5 pF.
- **Verdict:** Excellent. Connecting via 10 cm wires preserves femtosecond performance.
- **Recommendation:** Use **coaxial cable** or tightly twisted pairs (Clock + GND). The ground wire must connect to Pico GND immediately adjacent to GPIO 20 to minimize loop inductance.

### 4.2 Jitter Budget (Compact Layout)

| Signal | Source Jitter | Interconnect Adder | Total at DAC |
|--------|---------------|-------------------|--------------|
| **Audio Clock** | 43 fs | +10–20 fs (inductive pickup) | ~60 fs |

**Conclusion:** The compact layout preserves the crystal's femtosecond performance, degrading only negligibly.

### 4.3 Why NOT to Use Long Cables

**Distributed Layout (NOT Recommended):** Pi 4 central, Picos/DACs remote (1–2 meters).

**SPI Bus Problems:**
- 1–2 meter cables are transmission lines. Without impedance matching, 30 MHz signals will reflect, causing double-clocking and corrupted data.
- Inductance creates "Ground Bounce," where the Pico's 0V reference jumps relative to the Pi 4's 0V.
- **Verdict:** Do NOT run raw SPI over long cables without LVDS transceivers (which add latency and complexity).

**Audio Clock Problems:**
- 2-meter cable adds ~100 pF capacitance.
- Rise time slows dramatically (1 ns → 10+ ns).
- Slower edges are more susceptible to voltage noise → AM-to-PM conversion.
- **Result:** 43 fs jitter degrades to picoseconds or worse due to EMI and ground loops.
- **Verdict:** Do NOT run NDK clock over long cables.

**Design Rule:** Keep the "Digital Core" (Pi 4 + Picos + Crystal) physically compact (< 15 cm box). If long-distance distribution is needed, run the *analog* audio outputs (RCA/XLR) to amps/speakers using proper shielded cable and balanced connections.

---

## Section 5: Power Distribution & Signal Integrity

### 5.1 Voltage Domains

| Domain | Voltage | Current | Noise Sensitivity | Source |
|--------|---------|---------|-------------------|--------|
| Logic Core | 5.0 V | 2.5 A | Low | Mean Well RS-15-5 (SMPS) |
| **Audio Clock** | **3.3 V** | **50 mA** | **Ultra-High** | **LT3042 LDO** |
| **DAC** | **3.3 V** | **30 mA** | **High** | **Separate 3.3V (Star routed)** |
| Amplifiers | 24.0 V | 5–10 A | Medium | Separate 16-24V SMPS |

### 5.2 The "Clean" 3.3V Rail Strategy

**Problem:** Do NOT use the 3.3V pin from the Raspberry Pi 4 or Pico 2's internal regulator for the NDK Crystal. These rails have significant ripple (often >50 mV) from CPU load transients.

**Solution: The LT3042 Ultra-Low Noise LDO**
- **Component:** LT3042 Regulator Module (or ADM7150).
- **Performance:** < 1 µVrms noise (vs. ~1000 µVrms for standard regulators).
- **Configuration:**
  - Input: 5.0V from the main Logic Supply.
  - Output: 3.3V dedicated to Crystal and Clock Buffer.
- **Result:** The clock source operates in a "silence bubble," immune to Pi 4's USB/SPI activity.

What about using the Pi 4 / Pico 2's 3.3V pin for the DACs? Also not a good idea, due to the ripple. Using an LT3045 or similar with sufficient capacity for crystal + DACs offers cost/simplicity benefits. For best performance, isolating the domains by a ferrite bead + 100 uF capacitor for the crystal and 0.1 ohm resistor for the DACs will ensure minimal cross-talk.

### 5.3 Grounding Topology: "Star-of-Stars"

Ground loops are the primary cause of hum and jitter. We employ a hierarchical Star Ground:

**Dirty Ground Hub (Digital):**
- The (–) terminal of the 5V PSU.
- Connects: Pi 4 GND, Pico Digital GND.
- *Path:* High-current USB/SPI noise flows directly back to PSU.

**Clean Ground Hub (Analog):**
- The (–) output pin of the LT3042 LDO.
- Connects: NDK Crystal GND, Buffer GND, DAC GNDs.
- *Path:* Quiet, constant reference currents.

**The Bridge:**
- Connect Clean Hub to Dirty Hub at **exactly one point** (at the LT3042 input).
- **Why:** Prevents "Dirty" return currents from flowing through the "Clean" ground path.

### 5.4 Wiring & Parasitic Mitigation

**Power Entry:**
- Place a **Bulk Capacitor (470 µF)** + **Decoupling Capacitor (100 nF)** at the power input of each board (Pi 4, Pico, Crystal).
- This mitigates wire inductance and provides instant current for transients.

**Crosstalk Prevention:**
- Never bundle DC power cables parallel to Analog Audio or Clock cables.
- Run them perpendicularly or spaced > 5 cm apart.

**Mains Isolation:**
- Use **Class II (Floating) Power Supplies** (2-prong plugs) for 5V and 24V bricks to prevent ground loops through the mains Earth pin.
- If chassis must be earthed for safety, use a "Ground Breaker" circuit (10Ω resistor || 100nF capacitor || Bridge Rectifier) to clamp faults while blocking hum.

---

## Section 6: Software Architecture & Comparative Analysis

### 6.1 Why This Design Beats XMOS

The DIYINHK XMOS XUF216 was the previous consideration for high-channel-count USB interfaces. However, this architecture is superior:

| Feature | XMOS XUF216 | RPi 4 + 3× Pico 2 | Advantage |
|---------|------------|------------------|-----------|
| **Processing Power** | 16 Cores @ 500 MHz | 4× Cortex-A72 @ 1.5 GHz + 6× Cortex-M33 @ 150 MHz | **RPi 4 (3x)** |
| **RAM** | ~512 KB | 2–8 GB | **RPi 4** |
| **USB Bandwidth** | 1× Transaction (standard) | 3× Transactions (native) | **RPi 4** |
| **Max Output Rate** | 32-bit @ 384 kHz | 32-bit @ 768+ kHz | **RPi 4** |
| **DSP Capability** | Basic EQ | Advanced Room Correction + FIR/IIR + Convolution | **RPi 4** |

**Key Differentiator:**
XMOS is a "dumb pipe" (moves audio). The Pi 4 is a "smart engine" (processes audio). You can apply complex room correction to all 30 channels *before* sending to DACs, eliminating external DSP hardware.

### 6.2 Theoretical Hardware Limits

Our 30-channel @ 96 kHz design uses ~**35% of available USB bandwidth**. How far could this hardware be pushed?

**USB 2.0 High-Bandwidth Limit:** ~24.576 MB/s (3 transactions/microframe).

**Max Channels @ 96 kHz (24-bit):**
$$\frac{24,576,000}{96,000 \times 3} \approx 85 \text{ Channels}$$

**Max Sample Rate @ 30 Channels:**
$$\frac{24,576,000}{30 \times 3} \approx 273 \text{ kHz}$$

*(Note: The PCM5102A DAC supports up to 384 kHz).*

**Conclusion:** Significant headroom. This design could upgrade to 192 kHz without hardware changes (only doubling the data rate to ~17.2 MB/s).

### 6.3 Software Implementation

**Raspberry Pi 4 (Host):**
- **OS:** Linux with `PREEMPT_RT` patch (Real-Time Kernel) to minimize USB interrupt latency (8000 Hz cycle).
- **USB Gadget:** ConfigFS USB Gadget API (`g_audio` / `libcomposite`).
  - *Critical:* Standard Linux UAC2 may default to `wMaxPacketSize = 1024`. You must explicitly enable **High-Bandwidth Mode** to support >1080 byte payloads.
- **DSP:** Software like `CamillaDSP` or custom C++/Rust application can bind to UAC2 input and output to SPI bus.

**Pico 2 (PHY):**
- **Firmware:** Bare-metal C SDK (No OS).
- **Complexity:** Low.
  1. DMA setup (circular buffer).
  2. PIO initialization (load SPI Slave and I2S Master programs).
  3. Main loop (housekeeping: check buffer levels, toggle GPIO).
- **No Real-Time OS Needed:** PIO + DMA is inherently hard-real-time. CPU doesn't touch audio data; it only manages flow control.

### 6.4 Development Ease

- **Raspberry Pi:** Extremely high. Develop DSP in Python (prototyping) or C++ (production) directly on device.
- **Pico 2:** High. UF2 bootloader flashes instantly. PIO state machines can be simulated/debugged with `pioasm` tools.

---

## Summary & Recommendations

### Key Strengths of This Design

1. **Ultra-Low Jitter:** 43 fs from crystal, degrading only ~10–20 fs over interconnects (compact layout).
2. **High DSP Capacity:** The Pi 4 can run room correction and advanced filtering on all 30 channels simultaneously.
3. **Significant Headroom:** Could theoretically handle 85 channels @ 96 kHz or 273 kHz @ 30 channels without hardware changes.
4. **Easy to Develop:** Standard Linux and C SDK tools; no proprietary XMOS toolchain required.
5. **Cost-Effective:** ~$180 BOM (including power supplies).

### Implementation Roadmap

1. **PCB Design:** Create the 50mm × 50mm crystal oscillator board with Pierce oscillator and LMK1C110x buffer.
2. **Pi 4 Setup:** Install `PREEMPT_RT` kernel and patch the USB gadget driver for High-Bandwidth endpoints.
3. **Pico Firmware:**
   - Implement SPI Slave (PIO Block 0 + DMA).
   - Implement I2S Master clocked from external pin (PIO Block 1 + DMA).
   - Buffer management and GPIO feedback logic.
4. **Integration Testing:** Verify jitter performance, USB stability, and DSP throughput.

### Future Scalability

- **More Channels:** Up to 85 @ 96 kHz with this hardware.
- **Higher Sample Rates:** Up to 192 kHz @ 30 channels requires only software optimization.
- **More Picos:** Easily add more units; the USB bus and SPI are the only constraints.
