# GenMDM – Technical Reference

**GenMDM** is a MIDI interface for the Sega Genesis / Mega Drive that exposes the **YM2612 (6 FM voices)** and **SN76489 (PSG)** directly to MIDI. It turns the console into a hardware synth: play from a keyboard, sequencer, or DAW with low latency and deep parameter control.

> **Note:** The CC map below reflects common GenMDM firmware (v102/v103 style). If your cart/firmware differs, adjust the CC numbers accordingly.

---

## Table of Contents
- [MIDI Channel Map](#midi-channel-map)
- [Preset (Instrument) Store/Recall](#preset-instrument-storerecall)
- [FM (YM2612) Controls](#fm-ym2612-controls)
  - [Per-Channel (Voice) Controls](#per-channel-voice-controls)
  - [Per-Operator Controls](#per-operator-controls)
  - [Global FM Controls](#global-fm-controls)
- [DAC / Custom Wave (YM2612 Ch.6)](#dac--custom-wave-ym2612-ch6)
- [PSG (SN76489) Mapping](#psg-sn76489-mapping)
- [Voice 3 – Special Mode](#voice-3--special-mode)
- [Notes & Caveats](#notes--caveats)
- [Quick CC Cheat-Sheet](#quick-cc-cheat-sheet-most-used)

---

## MIDI Channel Map
- **Ch. 1–6** → YM2612 FM voices 1–6  
- **Ch. 7–9** → PSG tone channels A/B/C  
- **Ch. 10** → PSG noise / percussion  
- **Ch. 11–13** → Operator frequency control when **Voice 3 Special Mode** is enabled (see below)

> Pitch Bend applies per channel; set your preferred bend range via CC.

---

## Preset (Instrument) Store/Recall
- **Store to RAM**: **CC 6**, value **0–15** (16 slots)  
- **Recall from RAM**: **CC 9**, value **0–15**

Store: On the FM channel you’re editing (MIDI ch 1–6), send CC 6 with a value N = 0–15 to write the current voice settings into RAM slot N.

Recall: Later, on whatever FM channel you want to load it to, send CC 9 with the same value N.

These are volatile RAM slots (cleared on power cycle)

---

## FM (YM2612) Controls

### Per-Channel (Voice) Controls
| Parameter | CC | Range / Notes |
|---|---:|---|
| Algorithm | 14 | 0–7 |
| Feedback | 15 | 0–7 |
| Stereo Pan | 77 | 0–3 (00=L/R off, 01=L, 02=R, 03=L+R) |
| AMS (Amp Mod Sens.) | 76 | 0–7 |
| PMS (Pitch Mod Sens.) | 75 | 0–7 |
| Pitch-Bend Range | 81 | 0–18 semitones |
| Transpose (per channel) | 85 | 0–127 (typically semitone offset) |

> Normal MIDI **Note On/Off**, **Velocity**, and **Pitch Bend** apply per channel.

### Per-Operator Controls
Controls are grouped OP1 → OP4.

| Parameter | CCs (OP1..OP4) | Range / Notes |
|---|---|---|
| Total Level (TL) | 16, 17, 18, 19 | 0–127 (operator loudness) |
| Multiple (MUL) | 20, 21, 22, 23 | 0–15 |
| Detune (DT) | 24, 25, 26, 27 | 0–7 |
| Key Scale (KS) | 39, 40, 41, 42 | 0–3 |
| Attack Rate (AR) | 43, 44, 45, 46 | 0–31 |
| Decay 1 (DR) | 47, 48, 49, 50 | 0–31 |
| Decay 2 (D2R) | 51, 52, 53, 54 | 0–15 |
| Sustain Level (SL) | 55, 56, 57, 58 | 0–15 |
| Release Rate (RR) | 59, 60, 61, 62 | 0–15 |
| AM Enable (per-op) | 70, 71, 72, 73 | 0/1 |

### Global FM Controls
| Parameter | CC | Range / Notes |
|---|---:|---|
| LFO Enable | 74 | 0/1 |
| LFO Speed | 1 | 0–7 |
| PAL/NTSC Base Rate | 83 | 0=NTSC, 1=PAL (affects tuning) |
| Octave Division | 84 | 0–127 (system scale/divider) |
| Voice 3 Special Mode (toggle) | 80 | 0/1 |
| Test / Debug Registers | 92–97 | Bitfields (advanced) |

> **Algorithm (0–7)** follows standard YM2612 operator routings (carriers/modulators).

---

## DAC / Custom Wave (YM2612 Ch.6)
These affect **FM Voice 6** when DAC is enabled.

| Parameter | CC | Range / Notes |
|---|---:|---|
| DAC Enable | 78 | 0/1 |
| DAC Direct Data | 79 | 0–127 (write data stream) |
| Sample Pitch / Speed | 86 | 0–127 |
| Oversample | 88 | 0–15 |
| Noise / Custom-Wave Mode | 89 | 0/1 |
| Custom Wave Table (14 bytes) | 100–113 | each 0–127 |

Use for one-shot/sample-style sounds or short custom wave tables on channel 6.

---

## PSG (SN76489) Mapping
- **Ch. 7–9**: PSG tone channels A/B/C (square waves)  
- **Ch. 10**: PSG noise channel (drums/FX)

**Noise note guide (Ch. 10):**
- **C / C#** → high-frequency periodic  
- **D / D#** → medium periodic  
- **E** → low periodic  
- **F** → high noise  
- **F#** → medium noise  
- **G / G#** → low noise  
- **A / A#** → “Ch. 9; Periodic” (alternate mapping)  
- **B** → “Ch. 9; Noise” (alternate mapping)

> **Dynamics:** PSG loudness follows **Note Velocity** (no dedicated PSG volume CC in stock mapping).

---

## Voice 3 – Special Mode
When **CC 80 = 1**, YM2612 **Voice 3** switches to a mode where **each operator has its own frequency**.
- Play **Ch. 3, 11, 12, 13** to drive OP1–OP4 frequencies respectively.  
- **Velocity** controls operator loudness.

Great for additive / formant-like timbres.

---

## Notes & Caveats
- **Firmware v103**
- **PSG gain staging:** Expect velocity-scaled loudness; if a patch feels quiet, layer a YM2612 carrier.  
- **Realtime MIDI:** GenMDM typically ignores most realtime clock (F8–FF); sequencing is note/CC driven.

---

## Quick CC Cheat-Sheet (most used)
- **Algorithm** 14 · **Feedback** 15 · **TL (OP1–4)** 16–19  
- **MUL (OP1–4)** 20–23 · **DT (OP1–4)** 24–27  
- **AR/DR/D2R/SL/RR** 43–62 (see per-op table)  
- **AMS** 76 · **PMS** 75 · **Pan** 77 · **PB Range** 81  
- **LFO On** 74 · **LFO Speed** 1  
- **DAC** 78/79/86/88/89/100–113 (Ch. 6)  
- **Preset Store / Recall** 6 / 9 (0–15)
