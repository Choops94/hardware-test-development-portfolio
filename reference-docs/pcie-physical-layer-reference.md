# PCIe Physical Layer - Quick Reference

## Overview

The PCIe physical layer is responsible for transmitting raw bits across the physical medium between two devices.
It handles signal encoding, equalization, and clocking — all of which are essential for reliable high-speed communication across PCB traces.

**This reference covers:**
- Differential signaling and noise rejection
- De-emphasis and equalization (combating dispersion)
- Spread spectrum clocking (EMC compliance)

**Prerequisites:** LESSON-PCIE-01 (PCIe Protocol Overview), LESSON-SI-01 (Signal Integrity Fundamentals)

---

## Differential Signaling

### Core Concept

PCIe uses **differential pairs** — two conductors carrying opposite-polarity versions of the same signal (D+ and D−). The receiver measures the *difference* between them:

```
V_diff = V(D+) − V(D−)
```

Each PCIe lane contains:
- One **TX** differential pair (transmit)
- One **RX** differential pair (receive)

### Key Parameters

| Parameter | Value |
|-----------|-------|
| Nominal differential swing | ~800mV peak-to-peak (±400mV) |
| Differential impedance | 100Ω (50Ω per conductor to ground) |
| Coupling | AC coupled (capacitors in series) |

### Noise Rejection — How It Works

Common-mode noise couples equally onto both conductors. Because the receiver subtracts D− from D+, equal noise cancels out:

```
Without noise:   D+ = +400mV,  D− = −400mV  →  V_diff = +800mV  ✓
With noise:      D+ = +600mV,  D− = −200mV  →  V_diff = +800mV  ✓ (noise cancels)
```

**This is why differential signaling is so effective in noisy environments** — crosstalk and EMI typically couple equally onto both conductors of a tightly routed pair.

### AC Coupling

PCIe differential pairs are AC coupled — capacitors are placed in series on the signal path. A capacitor blocks DC and passes high frequencies (high-pass behaviour).

**Practical benefit:** Each device can operate at its own DC bias voltage. No DC voltage reference needs to be agreed between transmitter and receiver. This enables interoperability between devices from different vendors operating at different internal voltage levels, and protects sensitive PHY circuitry from DC current flow caused by ground potential differences.

### Healthy vs Faulty Differential Pair

| Condition | D+ | D− | V_diff |
|-----------|----|----|--------|
| Healthy | +400mV | −400mV | +800mV |
| Short between D+ and D− | +400mV | +400mV | 0V |
| D− shorted to ground | +400mV | 0V | +400mV (corrupted) |
| Differential-mode noise on one conductor | +400mV + noise | −400mV | V_diff corrupted |

**Key distinction:**
- **Common-mode noise** — equal noise on both conductors → cancels → no effect
- **Differential-mode noise** — noise on one conductor only → does not cancel → corrupts signal

---

## De-emphasis and Equalization

### The Problem: Dispersion

At high data rates, PCB traces behave like low-pass filters — they attenuate high-frequency components more than low-frequency ones. This is **dispersion**. The result: sharp edges at the transmitter arrive rounded and weak at the receiver.

The problem worsens with Gen speed:

| PCIe Gen | Data Rate | Dispersion Severity |
|----------|-----------|---------------------|
| Gen 1 | 2.5 GT/s | Mild |
| Gen 2 | 5 GT/s | Significant |
| Gen 3 | 8 GT/s | Severe — equalization essential |
| Gen 4 | 16 GT/s | Critical |

PCIe addresses this with two complementary techniques: **de-emphasis** at the transmitter and **CTLE equalization** at the receiver.

---

### De-emphasis (Transmitter Side)

**What it does:** Reduces the amplitude of *repeated bits* — bits that are the same value as the previous bit. Transition bits (where the signal changes) are transmitted at full amplitude.

```
Data stream:      1    1    1    0    0    1    0
Full amplitude:   ▄    ▄    ▄    _    _    ▄    _
With de-emphasis: ▄    ▄▄   ▄▄   _    __   ▄    _
                  ^    ^--- these repeated bits are reduced ~3.5dB
               transition
```

**Why this works:**
- Transition bits carry high-frequency energy that dispersion attenuates most
- Repeated bits are lower-frequency content that survives the channel better
- Reducing repeated bits costs little in information content but creates a clearer relative contrast at the receiver

**Worked example — PCIe Gen 2:**
- Transition bit (full amplitude): 800mV differential
- Repeated bit (de-emphasised): ~450mV differential (approx. −3.5dB)
- After lossy 20cm trace: receiver sees relatively consistent, detectable amplitude across all bits

**Important for oscilloscope measurements:** Repeated bits appearing smaller than transition bits on a PCIe signal is **normal expected behaviour** — not a fault.

**De-emphasis is a fixed setting** — transition and repeated bits are always distinguishable in the same way, so no adaptation is needed.

---

### CTLE — Continuous Time Linear Equalisation (Receiver Side)

**What it does:** Applies a high-pass filter at the receiver that boosts high-frequency components — the same frequencies that dispersion attenuated. This partially restores the original signal shape.

**CTLE is adaptive** — it configures itself based on the insertion loss profile of the specific channel it is connected to, because different board designs, trace lengths, and connector configurations all produce different insertion loss profiles.

#### Insertion Loss

**Insertion loss** is the frequency-dependent attenuation of a channel, measured in dB:

```
Insertion Loss (dB) = 20 × log10(V_received / V_transmitted)
```

**Example:**
```
Transmit: 800mV at 2.5GHz
Receive:  400mV at 2.5GHz

IL = 20 × log10(400/800)
   = 20 × log10(0.5)
   = −6dB
```

This calculation is repeated across a sweep of frequencies to build the **insertion loss profile** of the channel. The CTLE then applies approximately the inverse — +6dB gain at frequencies showing −6dB loss.

#### How CTLE Adapts — Link Training

During PCIe link training (before data flows), both devices exchange **known training sequences**. The receiver compares what arrives against what was expected, characterises the channel's insertion loss profile, and configures CTLE coefficients accordingly. Every time a PCIe link comes up, it performs this channel survey.

---

### De-emphasis and CTLE Working Together

| Technique | Where | What It Solves | Fixed or Adaptive |
|-----------|-------|----------------|-------------------|
| De-emphasis | Transmitter | Makes transition bits stand out relative to repeated bits | Fixed |
| CTLE | Receiver | Boosts attenuated high-frequency components | Adaptive |

**Why use both?**
- De-emphasis alone: receiver may still struggle to interpret any bits if overall amplitude is too low
- CTLE alone: without de-emphasis, repeated bits and transition bits may be hard to distinguish after dispersion
- Together: de-emphasis curates the signal so the most important bits stand out; CTLE then boosts what the channel attenuated

---

## Spread Spectrum Clocking (SSC)

### The Problem: EMC Compliance

PCIe uses a reference clock shared between devices. A perfectly stable single-frequency clock (e.g. exactly 100MHz) concentrates all its energy at one frequency and its harmonics (200MHz, 300MHz, 400MHz...), creating sharp narrow spikes in the electromagnetic emissions spectrum.

These spikes can breach regulatory limits (e.g. FCC, CE, Def Stan 59-411) — causing EMC compliance failures that could affect other electronic devices.

### The Solution: Spread the Energy

SSC deliberately drifts the clock frequency over a small range (typically ±0.5% of nominal) at a slow modulation rate (~30–33kHz).

**Effect on emissions:**

```
Without SSC:              With SSC:
     |                        |
     |  spike                 | lower, broader hump
     |   |                    |  ___
     |   |                    | /   \
_____|___|_____          ______|     |______
   100MHz                   99.5  100.5MHz
```

The total energy is **conserved** — it is redistributed across the frequency range, reducing the peak below the regulatory threshold. This is what achieves compliance.

### Why SSC Doesn't Break PCIe Timing

SSC introduces slow, predictable frequency drift. Two reasons this is tolerable:

1. **The CDR tracks it easily** — PCIe receivers use Clock/Data Recovery (CDR) circuits that extract timing from the incoming data stream continuously. At GB/s data rates, a 33kHz modulation is imperceptibly slow — the receiver tracks it effortlessly.

2. **The drift is slow relative to data rate** — SSC varies at ~33kHz; data transfers at billions of bits per second. The receiver's timing reference updates far faster than the clock drifts.

### One-Sided Spreading (Downward Only)

PCIe SSC spreads **downward only** from nominal frequency — e.g. 100MHz to 99.5MHz, never above 100MHz.

**Why:**
- Every PCIe specification defines a *maximum* data rate (e.g. Gen 2 = 5 GT/s exactly)
- Spreading upward would cause the clock to occasionally exceed nominal frequency, driving the data rate above the rated maximum
- Receivers designed to the spec may fail if data arrives faster than the rated maximum
- Downward-only spreading guarantees the clock never exceeds nominal — the data rate never exceeds spec — ensuring interoperability between all compliant devices

| Spreading Direction | Effect on Data Rate | Compliance Risk |
|--------------------|---------------------|-----------------|
| Downward only | Never exceeds maximum | Safe — all compliant receivers handle it |
| Two-sided | Occasionally exceeds maximum | May cause errors in receivers at timing limit |

---

## Key Takeaways

1. **Differential signaling rejects common-mode noise** — noise on both conductors cancels at the receiver via subtraction
2. **AC coupling enables vendor interoperability** — no shared DC voltage reference needed
3. **Dispersion worsens with Gen speed** — equalization is optional at Gen 1, essential at Gen 3+
4. **De-emphasis is a transmitter technique** — reduces repeated bits to make transition bits stand out
5. **CTLE is a receiver technique** — boosts high frequencies attenuated by the channel
6. **Insertion loss is the key channel characteristic** — IL (dB) = 20 × log10(V_rx / V_tx)
7. **Link training characterises the channel** — CTLE coefficients are set based on measured insertion loss
8. **SSC reduces EMC emissions** — by spreading clock energy across a frequency range rather than concentrating it
9. **SSC is one-sided downward** — ensures data rate never exceeds the rated maximum
10. **Repeated bits appearing smaller than transition bits is normal** — this is de-emphasis working as intended

---

## Common Mistakes and Gotchas

| Mistake | Reality |
|---------|---------|
| "Repeated bits looking smaller is a fault" | This is normal de-emphasis behaviour |
| "SSC loses energy" | Total energy is conserved — it is redistributed, not lost |
| "CTLE boosts the whole signal uniformly" | CTLE specifically boosts high frequencies only |
| "AC coupling means you need a capacitor to ground" | AC coupling here means a series capacitor on the signal path — blocks DC, passes transitions |
| "Both wires of a differential pair should look identical" | They should be identical but *inverted* — if they look the same, suspect a short or fault |

---

## Quick Reference: PCIe Physical Layer Parameters

| Parameter | Value |
|-----------|-------|
| Differential impedance | 100Ω |
| Per-conductor impedance | 50Ω |
| Nominal differential swing | ~800mV peak-to-peak |
| De-emphasis level (typical) | ~−3.5dB on repeated bits |
| SSC modulation rate | ~30–33kHz |
| SSC spread (typical) | −0.5% to 0% of nominal frequency |
| Gen 1 data rate | 2.5 GT/s |
| Gen 2 data rate | 5 GT/s |
| Gen 3 data rate | 8 GT/s |
| Gen 4 data rate | 16 GT/s |

---

## What You Can Measure with a USB Oscilloscope (PROJECT-1)

| Measurement | Feasible with 20MHz scope? | Notes |
|-------------|---------------------------|-------|
| Differential signal presence | Yes | Confirm pair is active |
| Approximate amplitude | Yes | Check against ~800mV spec |
| Rise time (Gen 1) | Marginal | Gen 1 edges ~400ps — at the limit of 20MHz BW |
| Rise time (Gen 2+) | No | Edges too fast for 20MHz bandwidth |
| De-emphasis effect | Unlikely | Requires sufficient bandwidth to resolve individual bits |
| SSC frequency drift | Possible | Clock modulation is slow enough to observe with FFT |

**Practical tip:** A 20MHz oscilloscope has a bandwidth limit that will round off fast edges. Measured rise times will appear slower than reality. Document this limitation explicitly in your PROJECT-1 write-up — understanding equipment limits is itself a test engineering skill.

---

## Connections to Other Lessons

- **LESSON-SI-01** — Dispersion, impedance, differential signaling fundamentals
- **LESSON-PCIE-01** — PCIe layered architecture, link training context
- **LESSON-SI-03** — Jitter (spread spectrum jitter introduced by SSC)
- **PROJECT-1** — PCIe signal characterisation on Raspberry Pi 5

---

**Document Version:** 1.0
**Lesson:** LESSON-PCIE-02
**Created:** March 2026
**Portfolio:** Hardware Test Engineer Development Plan
**GitHub Location:** `reference-docs/pcie-physical-layer-reference.md`
