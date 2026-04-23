# Hardware Test Engineer — Learning Journal

**Portfolio:** Hardware Test Engineer Development Plan  
**Started:** February 2026  
**Last Updated:** April 2026

---

## Progress Summary

| Item | Status | Date | Reference Doc |
|---|---|---|---|
| LESSON-SI-01 | ✅ Complete | Feb 2026 | `signal-integrity-fundamentals-reference.md` |
| LESSON-PCIE-01 | ✅ Complete | Mar 2026 | `pcie-protocol-reference.md` |
| LESSON-PCIE-02 | ✅ Complete | Mar 2026 | `pcie-physical-layer-reference.md` |
| PROJECT-1 | ✅ Complete | Apr 2026 | `pcie-characterization-reference.md` |
| LESSON-SI-02 | ✅ Complete | Apr 2026 | `rise-time-analysis-reference.md` |

**Milestone 2 status:** Nearly complete — SI-01, SI-02, PCIE-01 done, PROJECT-1 done. Remaining: LESSON-SI-03 (Eye Diagrams and Jitter).

---

## Equipment

| Item | Status | Notes |
|---|---|---|
| Hantek 6022BE oscilloscope | ✅ Acquired | 20MHz bandwidth, 2-channel USB. OpenHantek6022 v3.4.0 on Linux Mint |
| Pimoroni NVMe Base (PIM699) | ✅ Acquired | PCIe x1, M.2 M-key |
| SK Hynix BC501A 128GB NVMe SSD | ✅ Acquired | M.2 2230 |
| Logic analyser (24MHz 8-channel) | ✅ Acquired | |

---

## Lesson Log

---

### LESSON-SI-01 — Signal Integrity Fundamentals
**Date:** February 2026  
**Prerequisites:** None  
**Reference doc:** `signal-integrity-fundamentals-reference.md`

**Learning outcomes achieved:**
- Transmission line effects — when a wire becomes a transmission line (rise time ≈ propagation delay)
- Characteristic impedance, reflection coefficient, and termination strategies
- Recognising overshoot, undershoot, ringing, crosstalk, and dispersion on scope captures

**Key insights from this session:**
- Signal integrity is governed by rise time, not signal frequency — a key conceptual shift
- Reflections always occur at impedance mismatches; rise time determines whether they're visible
- Connected concepts to existing CAN bus and motor inverter experience (crosstalk, termination)
- Three termination strategies: series (at TX), parallel (at RX), AC termination

---

### LESSON-PCIE-01 — PCIe Protocol Overview
**Date:** March 2026  
**Prerequisites:** None  
**Reference doc:** `pcie-protocol-reference.md`

**Learning outcomes achieved:**
- PCIe layered architecture — Transaction, Data Link, and Physical layers and their roles
- Link training and LTSSM state machine — how PCIe devices negotiate and establish a link
- TLP and DLLP packet structure — headers, payloads, sequence numbers, and ACK/NAK

**Key insights from this session:**
- PCIe is point-to-point (not a shared bus) — each device gets its own dedicated link
- Link width (x1, x4, x16) and generation (Gen 1–5) are independently negotiable
- LTSSM transitions happen on ms/µs timescales — measurable with a 20MHz scope unlike the data signals themselves
- Reason for multi-lane designs: bandwidth scaling without increasing per-lane speed

---

### LESSON-PCIE-02 — PCIe Physical Layer
**Date:** March 2026  
**Prerequisites:** LESSON-PCIE-01, LESSON-SI-01  
**Reference doc:** `pcie-physical-layer-reference.md`

**Learning outcomes achieved:**
- Differential signalling — D+/D− pairs, common-mode noise rejection via subtraction
- De-emphasis (TX) and CTLE equalization (RX) as a system for combating dispersion
- Spread spectrum clocking — frequency dithering to spread EMI energy across a band

**Key insights from this session:**
- Noise coupling onto both conductors equally cancels at the differential receiver — the core value of differential signalling
- De-emphasis reduces repeated-bit amplitude at the transmitter to make transitions more identifiable after channel loss
- CTLE is adaptive (responds to channel insertion loss profile); de-emphasis is configurable but fixed per-session
- SSC reduces peak EMI by spreading clock energy — does not reduce total energy

---

### PROJECT-1 — PCIe Signal Characterization on Raspberry Pi 5
**Date:** April 2026  
**Hardware:** Pi 5, Pimoroni NVMe Base (PIM699), SK Hynix BC501A SSD, Hantek 6022BE  
**Reference doc:** `pcie-characterization-reference.md`  
**GitHub:** `signal-integrity-study/raspberry-pi-pcie/`

**Deliverables completed:**
- OpenHantek6022 v3.4.0 installed on Linux Mint (`.deb` package, udev rules, plugdev group)
- PCIe link characterisation via software: `lspci`, `dmesg`, `nvme-cli`, sysfs
- GPIO oscilloscope measurements: 100Hz, 1kHz, and 10kHz square waves on GPIO17 captured and saved
- Project write-up: `pcie-characterization-reference.md`

**Key findings and insights:**
- Hantek 6022BE (20MHz) cannot resolve PCIe signals at any generation — Gen 2 minimum bandwidth requirement is ~7.5GHz
- Physical probing of M.2 connector solder pads and FFC cable attempted; PCIe data and REFCLK signals unresolvable due to bandwidth limit
- Vpp readings (~5.23V) were inflated by overshoot on both edges — self-derived insight
- 10kHz square wave appeared triangular — correctly identified as rise time limiting caused by scope bandwidth
- 1kHz rise time measured at ~80ns using cursor method (10%–90%)

**Deferred to future sessions:**
- Two-channel differential measurements
- Spectrum analysis
- Eye diagrams

---

### LESSON-SI-02 — Rise Time and Signal Quality
**Date:** April 2026  
**Prerequisites:** LESSON-SI-01  
**Reference doc:** `rise-time-analysis-reference.md`

**Learning outcomes achieved:**
- Bandwidth–rise time relationship: `BW = 0.35 / t_r` — the 0.35 constant is empirical, not a hard cutoff
- RSS rule for combining scope and signal rise times: `t_r(measured)² = t_r(signal)² + t_r(scope)²`
- De-embedding true signal rise time from a bandwidth-limited measurement
- Identifying and quantifying overshoot, undershoot, and ringing; measurement formulas for each

**Key insights from this session:**
- Rise time is a transmitter property — it does not change with signal frequency
- Overshoot amplitude is set by ρ (impedance mismatch) — not by rise time
- Rise time determines *visibility* of overshoot, not its magnitude
- A bandwidth-limited scope does not just slow down edges — it actively hides the signal integrity problems that live on those edges
- Ringing duration is fixed; it becomes more significant at higher frequencies because it occupies a larger fraction of the period
- Undershoot below −0.3V is destructive to CMOS inputs
- Scope bandwidth rule of thumb: t_r(signal) ≥ 3× t_r(scope) for <5% measurement error

**Notable moment:** Self-diagnosed the conceptual confusion between signal frequency and frequency content (harmonics) — an important distinction that is easy to conflate.

---

## Next Steps

**Immediate:**
- Commit all reference docs and this journal to GitHub portfolio under `/reference-docs/`
- LESSON-SI-03: Eye Diagrams and Jitter (direct continuation from SI-02)

**On the horizon:**
- Two-channel oscilloscope measurements (deferred from PROJECT-1)
- LESSON-ETH-01 / PROJECT-2 (Ethernet BER Testing) when ready to branch out
- Logic analyser acquisition to unlock PROJECT-4 (Protocol Analysis)

---

*This journal is updated after each completed lesson or project milestone.*
