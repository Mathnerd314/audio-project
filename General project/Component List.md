# 7.1.4 Home Theater System – Complete Component List

## Source Devices and Input Stage
- Computer - Media PC - single input source, simplifies input switching and audio extraction 
- Secondary Control App (smartphone/tablet remote control)
- **Analog-to-Digital Converter (ADC) Chips** (optional, for analog inputs)
  - Converts analog audio sources to digital

## HDMI and Video Processing
- **HDMI 2.1 Cables** (4-7 total, connecting sources to receiver and receiver to display)
- **Video Display Output**
  - 4K/8K TV or Projector
  - Projector Screen (if applicable)
- Direct HDMI input to display via graphics card

## Audio Decoding and DSP Processing
- Custom PC-based solution based on Cavern + software DSP
- **I2S Audio Interface** (via sound card or external USB to I2S converter)
  - **I2S Interface Controller:** Manages I2S bus for communication between DSP, DACs, and amplifiers
  - **Master Clock Generator / Crystal Oscillator Circuit** (27 MHz or 48 MHz reference)
  - **PLL (Phase-Locked Loop):** Generates precise I2S clock signals at various sampling rates
- **Digital-to-Analog Converter (DAC) Chips** (e.g., PCM5252, AK series)
  - Multiple / multi-channel DACs for converting digital audio to analog
  - RCA or XLR outputs for each channel
- **GPIO Expander** (e.g., TCA6408A)
  - Provides additional digital I/O lines for amplifier reset and control signals
- **I2C Multiplexer** (e.g., TCA9548A)
  - Allows single MCU I2C line to address multiple DACs or configuration chips
- **Volume Control**
  - Stepped volume control using analog or digital potentiometer connected to microcontroller
  - Per-channel gain adjustment in addition to master volume and DSP-based EQ
- **Operational Amplifier (Op-Amp) Chips** (e.g., NE5532)
  - Differential-to-single-ended conversion
  - Signal buffering and preamp functions

## Amplification Stage
- **Multiple amplifier modules** - parallel, cascaded, external, internal
  – Choice of Topology depending on power requirements
  - Delivers 1-250W per channel depending on topology
- I2S or analog audio inputs
- Gain staging and low-impedance drive for power stage
- Filtering and protection circuitry as needed -  Block DC, bad frequencies, etc.
- Requires substantial heat sinking and power supply

## Speakers
- Front: Left, Right, Center
- Surround: Left, Right, Left Rear, Right Rear
- Height/Atmos: Front Left, Front Right, Rear Left, Rear Right
- Subwoofer(s): 2 units
- Speaker Construction TBD
- Cabling/Connectivity:
  - Speaker wire (14–16 gauge) for passive speakers
  - XLR or RCA cables for active speakers
  - Banana plugs or spade connectors

## Power Supply and Protection
- **Switching Power Supplies /Power Transformers** (for digital supply rails and amplifier power)
- **Step-Down DC/DC Converter Chips** (e.g., TPS54340)
- **Low Dropout Regulator (LDO) Chips** (e.g., TLV1117, LM2940C)
  - Generates regulated 3.3V, 5V, 12V, 15V, and other supply rails from mains
- **Power Cables and Mains Connections**
- **Surge Protectors or Power Conditioners**
- **Protection Circuits**: Overcurrent detection, thermal shutdown, DC offset detection (if not built in)
- **Heat Sinks** (significantly larger for Class AB and linear amplifier topologies)

## Installation and Mounting Hardware
- Speaker Stands (for bookshelf/active speakers)
- In-Ceiling Mounting Brackets and Grilles
- Wall Mounts (for surround speakers)
- Speaker Isolation Pads or Subwoofer Risers
- Cable Management Accessories (raceways, conduit, velcro ties, labels)
- AV Equipment Rack or Cabinet (with enhanced ventilation for higher-power amplifier topologies)

## Calibration and Room Tuning
- Room Calibration Microphone (e.g., miniDSP UMIK-1, USB or analog)
- Microphone Stand
- Acoustic Treatment Panels or Bass Traps
- Calibration Software (Dirac Live, Audyssey, etc.)