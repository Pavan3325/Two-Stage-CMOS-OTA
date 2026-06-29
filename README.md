# Precision PMIC — Two-Stage CMOS OTA + PMOS LDO
### GPDK090 90nm CMOS | Cadence Virtuoso / Spectre

[![Technology](https://img.shields.io/badge/Technology-GPDK090%2090nm-blue)]()
[![EDA](https://img.shields.io/badge/EDA-Cadence%20Virtuoso-red)]()
[![Simulator](https://img.shields.io/badge/Simulator-Spectre-orange)]()
[![Status](https://img.shields.io/badge/Status-Simulation%20Complete-green)]()
[![Phase](https://img.shields.io/badge/Next%20Phase-Bandgap%20Reference-purple)]()

---

## Overview

This project implements a **precision Power Management IC (PMIC)** platform in GPDK090 90nm CMOS, designed and verified in Cadence Virtuoso using the Spectre simulator.

The current phase delivers a **two-stage Miller-compensated CMOS OTA** integrated as the error amplifier inside a **PMOS Low-Dropout (LDO) Regulator**. The design has been verified against a comprehensive 13-point simulation matrix including Monte Carlo, noise, PSRR, and thermal analysis.

### Target Applications
- IoT sensor nodes
- Precision analog front-ends
- Battery-powered mixed-signal SoCs
- Reference voltage generation systems

---

## Architecture

```
VDD (1.8V)
    │
    ├─── PMOS Pass Device (M_pass)
    │         │
    │       VOUT (1.197V)
    │         │
    │    ┌────┴────┐
    │    Rfb1     Rfb2      ← Feedback divider
    │    └────┬────┘
    │         │ VFB
    │    ┌────▼────────────────────┐
    │    │   Two-Stage OTA         │
    │    │   (Error Amplifier)     │
    │    │                         │
    │    │  Folded Cascode Stage 1 │
    │    │  + Common Source Stage 2│
    │    │  Miller Cc=3pF, Rc=500Ω │
    │    └─────────────────────────┘
    │         │
    └─────────┘ (Gate drive to M_pass)

VREF ──► VIN- of OTA
```

---

## OTA Design Details

| Parameter | Value |
|-----------|-------|
| Topology | Two-stage Miller-compensated |
| Input Stage | Folded cascode PMOS differential pair |
| Technology Node | GPDK090 — 90 nm CMOS |
| Supply Voltage | 1.8 V |
| Miller Capacitor (Cc) | 3 pF |
| Zero-cancellation Resistor (Rc) | 500 Ω |
| Bias Current (I0) | 20 µA |

---

## Verified Performance

### AC / Frequency Response

| Metric | Simulated Value |
|--------|----------------|
| Open-Loop DC Gain | **70.89 dB** @ 10.96 kHz |
| Unity-Gain Bandwidth | **127.24 MHz** |
| Phase Margin | **67.39°** |
| Gain Margin | **45.52 dB** |

### LDO Closed-Loop

| Metric | Simulated Value |
|--------|----------------|
| Output Voltage (VOUT) | **1.197 V** |
| Supply Voltage (VDD) | 1.8 V |
| Dropout Voltage | 603 mV |
| PSRR @ Low Frequency | **−85 dB** |
| PSRR @ ~100 kHz | **−56 dB** |

### Noise

| Metric | Simulated Value |
|--------|----------------|
| Spectral Density @ Low Freq | 1.675 mV/√Hz |
| Spectral Density @ 104 MHz | 32.08 nV/√Hz |
| Integrated Output Noise (10 Hz–100 kHz) | **9.04 µVrms** |

### Transient Response

| Metric | Simulated Value |
|--------|----------------|
| Output Undershoot | **4.34 mV** |
| Output Overshoot | **4.06 mV** |
| Input Pulse (net5) | 700 mV → 1.1 V square wave |

### Statistical & Thermal

| Metric | Result |
|--------|--------|
| Monte Carlo Runs | 100 |
| Failures | **0** |
| Temperature Range | −40°C to +125°C |

---

## Simulation Results

### AC Response — Bode Plot
![AC Response](simulations/ac_response.png)
> Green: Gain (dB) | Magenta: Phase (deg)
> M1: 70.89 dB @ 10.96 kHz · M3: 67.39° @ 127.24 MHz · M2: −88.10 dB @ 127.06 MHz

### Noise Response
![Noise Response](simulations/noise_response.png)
> Output-referred noise spectral density (V/√Hz) vs frequency
> 1/f corner visible ~100–500 Hz, thermal floor ~32 nV/√Hz at 100 MHz

### Transient Response
![Transient Response](simulations/transient_response.png)
> Red (net5): Input square wave 700mV→1.1V | Green (net06): LDO output settling

### Schematic
![Op-Amp Schematic](schematics/op-amp/op_amp_schematic.png)
> Full transistor-level two-stage OTA in GPDK090

---

## Verification Matrix

| # | Analysis | Status |
|---|----------|--------|
| 1 | DC Operating Point | ✅ |
| 2 | AC / Bode Plot | ✅ |
| 3 | Open-Loop Gain | ✅ |
| 4 | Unity-Gain Stability | ✅ |
| 5 | Phase & Gain Margin | ✅ |
| 6 | Noise Spectral Density | ✅ |
| 7 | PSRR | ✅ |
| 8 | Line Regulation | ✅ |
| 9 | Load Regulation | ✅ |
| 10 | Dropout Verification | ✅ |
| 11 | Load Transient | ✅ |
| 12 | Temperature Sweep −40°C→+125°C | ✅ |
| 13 | Monte Carlo (100 runs) | ✅ |

---

## Project Roadmap

```
Phase 1 — LTspice Prototype          ✅ Complete
Phase 2 — Cadence Virtuoso / Spectre  ✅ Complete (this repo)
Phase 3 — Bandgap Voltage Reference   🔄 In Progress
Phase 4 — Full PMIC Integration       ⏳ Planned
Phase 5 — Sky130 / Open PDK Migration ⏳ Planned
Phase 6 — IEEE Publication            ⏳ Targeted
```

---

## Design Notes

### Why Miller Compensation?
Two-stage OTAs introduce a second pole from the output stage. Without compensation, phase margin collapses below 45°. The Miller capacitor Cc=3pF splits the poles — moving the dominant pole to low frequency and pushing the non-dominant pole above the unity-gain bandwidth. The zero-cancellation resistor Rc=500Ω eliminates the RHP zero that Cc introduces via feedforward.

### Why Folded Cascode Input?
Rail-to-rail input common-mode range. The folded topology allows the PMOS differential pair to operate even when VCM approaches VDD, critical for LDO error amplifiers where the reference input can be near the supply rail.

### PSRR Interpretation
The −85 dB PSRR at low frequency reflects the OTA's open-loop gain rejecting supply noise. Degradation to −56 dB at ~100 kHz is caused by the Miller capacitor Cc providing a feedthrough path from VDD to the gate of the pass device — a known trade-off in single-Miller LDO architectures.

---

## Tools & Environment

| Tool | Version / Details |
|------|------------------|
| Cadence Virtuoso | Schematic Editor L |
| Simulator | Spectre |
| PDK | GPDK090 (90nm CMOS, academic) |
| OS | Linux (CentOS / RHEL) |

---

## Author

**Pavankumar**
M.Tech Power Electronics — CMR College of Engineering & Technology
Specialization: Analog IC & Precision Power Management Design
Member: RISC-V International

📌 *Actively seeking full-time roles in Analog IC Design · PMIC · AMS · Semiconductor R&D*

[![LinkedIn](https://img.shields.io/badge/LinkedIn-Connect-blue)](https://linkedin.com/in/YOUR_PROFILE)

---

## License

This project is shared for academic and portfolio purposes.
GPDK090 PDK is proprietary to Cadence Design Systems — netlists and PDK files are not included in this repository.

---

*If you find this useful or have technical feedback, open an issue or connect on LinkedIn.*
# Two-Stage-CMOS-OTA