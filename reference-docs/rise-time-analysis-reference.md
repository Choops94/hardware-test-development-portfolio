# Rise Time and Signal Quality - Quick Reference

## What is Rise Time?

Rise time is the time taken for a signal's voltage to transition from **10% to 90%** of its final value.

**Key properties:**
- Set purely by the **transmitter** — not by signal frequency
- A 100Hz and 10kHz square wave from the same GPIO pin have identical rise times
- Fast rise time → more high-frequency harmonic content required to represent the edge

---

## Bandwidth and Rise Time

### The Core Relationship

Signals are composed of a fundamental frequency plus higher-order harmonics. Fast edges require high-frequency harmonic content. An oscilloscope (or any system) with limited bandwidth attenuates these harmonics, rounding off edges.

**Formula:**

```
BW = 0.35 / t_r
```

- `BW` = bandwidth in Hz
- `t_r` = rise time in seconds
- `0.35` is empirical — derived from Gaussian filter rolloff. It is **not** a brick wall; frequencies near the limit are gradually attenuated, not cut off completely.

**Example — Hantek 6022BE (20MHz scope):**
```
t_r(scope) = 0.35 / 20×10⁶ = 17.5ns

Any edge faster than ~17.5ns will appear slower than it really is.
```

**Example — Raspberry Pi GPIO (~2ns rise time):**
```
BW required = 0.35 / 2×10⁻⁹ = 175MHz

A 20MHz scope cannot faithfully represent this edge.
```

### Scope Bandwidth is Not a Brick Wall

The bandwidth figure is the -3dB point — where signal amplitude is attenuated by ~30%. High-frequency harmonics near this limit are partially attenuated, not eliminated. This is why edges look rounded rather than chopped.

**Rule of thumb for trusted measurements:**
- t_r(signal) ≥ 3× t_r(scope) → error <5%
- t_r(signal) ≥ 5× t_r(scope) → error <2%

---

## The RSS Rule — Recovering True Rise Time

When a signal passes through a bandwidth-limited scope, both contribute to the measured rise time:

```
t_r(measured)² = t_r(signal)² + t_r(scope)²
```

**Rearranged to find the true signal rise time:**
```
t_r(signal) = √(t_r(measured)² − t_r(scope)²)
```

**Example:**
```
t_r(scope)    = 17.5ns
t_r(signal)   = 2ns (true)
t_r(measured) = √(2² + 17.5²) = √(4 + 306.25) = ~17.6ns

The scope is the dominant factor — almost no information about the true signal rise time.
```

**Practical implication:** If your measured rise time is close to your scope's rise time limit, you know very little about the signal's true rise time. Use de-embedding (rearranged RSS formula) if you need an estimate.

---

## Rise Time and Signal Integrity

Fast rise times make signal integrity problems visible. The physics (reflections, impedance mismatch) is always present — rise time determines whether you can see it.

### Why Fast Rise Times Reveal Problems

```
FAST rise time (1ns):

  V
  |        /‾‾\  /‾\  /‾\__________
  |       /    \/   \/
  |______/
  |_____________________________ t
           ↑
     reflection window (2ns) = significant fraction of transition

SLOW rise time (1µs):

  V
  |              /‾‾‾‾‾‾‾‾‾‾‾‾‾‾‾
  |             /
  |____________/
  |_____________________________ t
     ↑
     same 2ns reflection window = invisible within 1µs transition
```

**Key insight:** The reflection window duration is fixed. For a fast rise time it overlaps with the active transition and corrupts the signal. For a slow rise time it is swamped by the transition.

---

## Overshoot, Undershoot, and Ringing

### What They Look Like

```
  V
  |         *                        ← overshoot peak
  |        / \  /‾\  /‾\____________
3.3V ----/----\/---\/    ±5% settled band
  |      |    rising edge ringing
  |      |    |←— settling time —→|
  |
  |____________\____                 ← falling edge
0V ----------- \  /‾\  /‾\________
  |              \/   \/
  |               *                  ← undershoot trough
  |              |←— settling time →|
```

### Overshoot

Signal exceeds its target steady-state voltage on a rising edge.

**Formula:**
```
Overshoot (%) = (V_peak − V_steady) / V_steady × 100
```

**Example:** Peak = 4.1V, Steady state = 3.3V
```
Overshoot = (4.1 − 3.3) / 3.3 × 100 = 24%
```

**Cause:** Positive reflection coefficient (ρ > 0) — load impedance > line impedance.

### Undershoot

Signal goes below its target steady-state voltage on a falling edge.

**Formula:**
```
Undershoot (%) = (V_steady − V_trough) / V_steady × 100
```

**Cause:** Negative reflection coefficient (ρ < 0) — load impedance < line impedance.

> ⚠️ Undershoot below −0.3V on CMOS inputs can cause latch-up or permanent damage.

### Ringing

Oscillation around the steady-state voltage following overshoot or undershoot, caused by reflections bouncing back and forth along the transmission line.

**Two measurements needed to characterise ringing:**

| Measurement | What it tells you |
|---|---|
| **Peak amplitude** | Worst-case voltage violation |
| **Settling time** | How long until signal is within ±5% of steady state |

**What controls bounce count:** Properties of the transmission line itself — loss (skin effect, dielectric loss). A lossy line absorbs energy faster; bounces die out quickly. A low-loss line sustains ringing longer.

**What controls visibility:** Rise time relative to signal period. Ringing duration is fixed — but at higher frequencies it occupies a larger fraction of the period and becomes more significant.

---

## The Amplitude vs Visibility Distinction

This is a critical distinction for measurement:

| Property | Controlled by |
|---|---|
| Overshoot/undershoot **amplitude** | Reflection coefficient ρ (impedance mismatch) |
| Whether you can **see** it | Scope bandwidth relative to rise time |

**A bandwidth-limited scope doesn't just slow down edges — it hides the signal integrity problems that live on those edges.**

The overshoot is still happening electrically. The scope rounds off the edge and the spike disappears from the display. The device under test still sees the full overshoot voltage.

---

## Quick Reference: Reflection Coefficient Recap

```
ρ = (Z_load − Z_line) / (Z_load + Z_line)
```

| ρ | Condition | Result |
|---|---|---|
| ρ > 0 | Z_load > Z_line | Overshoot on rising edge |
| ρ < 0 | Z_load < Z_line | Undershoot on rising edge |
| ρ = 0 | Z_load = Z_line | Perfect match — no reflections |
| ρ = +1 | Open circuit | Voltage doubles |
| ρ = −1 | Short circuit | Voltage forced to zero |

---

## Oscilloscope Measurement Tips

- **Measure overshoot** at the peak of the first spike above steady state
- **Measure settling time** from the edge to where signal stays within ±5% of steady state
- **Check both edges** — rising and falling edge reflections are independent events; report worst case
- **Account for scope rise time** — if measured rise time is close to scope's limit, use RSS de-embedding
- **Don't trust clean-looking edges** on a bandwidth-limited scope — overshoot may be present but invisible

---

## Key Takeaways

1. **Rise time is a transmitter property** — it doesn't change with signal frequency
2. **BW = 0.35 / t_r** — the formula connecting bandwidth and rise time; 0.35 is empirical, not a hard cutoff
3. **RSS rule** — measured rise time combines signal and scope contributions in quadrature
4. **Overshoot amplitude is set by ρ** — the impedance mismatch, not the rise time
5. **Rise time determines visibility** — fast edges make signal integrity problems observable
6. **Ringing duration is fixed** — but becomes more significant at higher signal frequencies
7. **Bandwidth-limited scopes hide edge artefacts** — a clean scope trace does not mean a clean signal
8. **Undershoot below −0.3V is destructive** to CMOS inputs

---

**Document Version:** 1.0
**Lesson:** LESSON-SI-02
**Prerequisites:** LESSON-SI-01
**Created:** April 2026
**Portfolio:** Hardware Test Engineer Development Plan
