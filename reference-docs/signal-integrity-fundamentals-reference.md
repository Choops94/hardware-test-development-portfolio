# Signal Integrity Fundamentals - Quick Reference

## What is Signal Integrity?

### Core Concept
**Signal integrity** is the study of what happens to electrical signals as they travel through physical conductors, and how to prevent distortions from causing errors.

At low speeds, wires are "just wires." At high speeds, wires become **transmission lines** with their own electrical behavior that can corrupt signals.

### The Critical Question: When Does Speed Matter?

**It's not about frequency - it's about rise time!**

- A 10 MHz square wave with 1ns rise time → Signal integrity issues
- A 100 MHz sine wave with slow transitions → Minimal issues

**Rule of Thumb:** If signal rise time becomes comparable to propagation delay along the wire, you're in "high-speed" territory.

---

## Transmission Lines

### What Makes a Wire Become a Transmission Line?

When **rise time ≈ propagation delay**, the entire wire becomes part of the transition. The wire is no longer "just a connection" - it's a **distributed system** where voltage varies along its length at any given instant.

### Key Parameters

**Propagation Delay:**
- How long for signal to travel from one end to other
- Speed ≈ 20 cm/ns in typical PCB traces
- Example: 10cm wire = 0.5ns propagation delay

**Rise Time:**
- How long for voltage to transition from low to high
- Determined by transmitter drive strength
- Fast signal: 1ns rise time
- Slow signal: 1000ns rise time

**Critical Insight:**
Propagation delay is **independent** of rise time. The "leading edge" travels at speed of light, but voltage is still changing as it travels.

---

## Characteristic Impedance (Z₀)

### What is Characteristic Impedance?

The inherent "resistance" a transmission line presents to a signal, determined by **geometry**, not frequency.

**Formula:** Z₀ = √(L/C)
- L = inductance per unit length
- C = capacitance per unit length

**Common Values:**
- PCB traces: 50Ω or 75Ω
- Coaxial cables: 50Ω (RF) or 75Ω (video)
- Twisted pair (Ethernet Cat5): 100Ω
- CAN bus twisted pair: 120Ω

### Physical Properties

Characteristic impedance is determined by:
- Trace width
- Trace thickness
- Distance to ground plane
- PCB dielectric material

**Key Point:** Z₀ is a **physical property** - it's always there regardless of signal speed.

---

## The Three Major Signal Integrity Issues

### 1. Reflections (Impedance Mismatch)

**Cause:** Load impedance ≠ Transmission line characteristic impedance

**Results in:**
- Overshoot (voltage exceeds target)
- Undershoot (voltage goes below target)
- Ringing (oscillations around target)

### 2. Crosstalk (Signal Coupling)

**Cause:** Electromagnetic coupling between adjacent traces

**Mechanisms:**
- Capacitive coupling (electric field)
- Inductive coupling (magnetic field)

**Results in:**
- Noise on "victim" traces from "aggressor" traces
- False triggering
- Reduced noise margins

### 3. Attenuation & Dispersion (Signal Loss)

**Attenuation:** Signal gets weaker over distance
**Dispersion:** High frequencies attenuated more than low frequencies

**Results in:**
- Reduced signal amplitude
- Slow, rounded edges (loss of high-frequency content)
- Inter-symbol interference (ISI)

---

## Reflections Deep Dive

### The Reflection Coefficient Formula

**ρ = (Z_L - Z₀) / (Z_L + Z₀)**

Where:
- ρ = Reflection coefficient (-1 to +1)
- Z_L = Load impedance (receiver)
- Z₀ = Characteristic impedance (transmission line)

### What the Numbers Mean

| Scenario | Z_L | ρ | Result |
|----------|-----|---|--------|
| **Matched** | Z_L = Z₀ | 0 | No reflection ✓ Perfect! |
| **Open circuit** | Z_L = ∞ | +1 | Voltage doubles (overshoot) |
| **Short circuit** | Z_L = 0 | -1 | Voltage goes to zero |
| **Partial mismatch** | Z_L = 75Ω, Z₀ = 50Ω | +0.2 | 20% overshoot |

**Positive ρ:** Reflection adds to signal (overshoot)
**Negative ρ:** Reflection subtracts from signal (undershoot)

### Why Fast Signals Show Problems But Slow Signals Don't

**The physics is the same - reflections happen at all speeds!**

**Fast signal (1ns rise time, 10cm wire):**
- Reflection causes 2-3ns of ringing
- This is 30-50% of the signal period
- **Highly visible distortion**

**Slow signal (1000ns rise time, same wire):**
- Same 2-3ns of ringing occurs
- This is <0.3% of the signal period
- **Invisible on practical timescales**

**Key Insight:** Overshoot and ringing are always present, but only matter when their duration is significant relative to the signal period.

---

## Termination Strategies

### Purpose
Match impedances to eliminate reflections (make ρ = 0)

### Series Termination (at Transmitter)

```
Transmitter --[R_series]-- Transmission Line -- Receiver
```

- Add resistor so: R_series + Z_source = Z₀
- Signal launches at half voltage, reflects at far end, adds up to full voltage
- **Pros:** No DC power consumption, simple
- **Cons:** Only works point-to-point (not multi-drop)

### Parallel Termination (at Receiver)

```
Transmitter -- Transmission Line -- Receiver
                                        |
                                       [R = Z₀]
                                        |
                                       GND
```

- Resistor to ground = Z₀
- Creates matched impedance
- **Pros:** Works with multi-drop, clean signals
- **Cons:** Continuous power consumption (P = V²/R)

### AC Termination (at Receiver)

```
                                       [R = Z₀]
                                        |
                                       ---  C
                                       ---
                                        |
                                       GND
```

- Resistor + capacitor to ground
- **Pros:** No DC power waste, terminates AC edges
- **Cons:** Slightly more complex, two components

### Real-World Example: CAN Bus

CAN uses **120Ω parallel termination** at each end of the bus:
- Twisted pair has 120Ω characteristic impedance
- Multi-drop topology (many nodes)
- Differential signaling (CAN_H and CAN_L)
- Termination between H and L lines (not to ground)

**Without termination:** Reflections cause communication errors, timeouts, bus failures

---

## Crosstalk Deep Dive

### The Two Coupling Mechanisms

**Capacitive Coupling (Electric Field):**
- Parallel traces form a capacitor
- Voltage change on aggressor couples to victim
- Proportional to dV/dt (rate of voltage change)
- Stronger when traces are closer, longer, faster edges

**Inductive Coupling (Magnetic Field):**
- Current in aggressor creates magnetic field
- Induces current in victim
- Proportional to dI/dt (rate of current change)
- Stronger with large current loops, shared return paths

### Forward vs Backward Crosstalk

**Both occur simultaneously when aggressor switches!**

- **Forward crosstalk:** Travels with the aggressor signal
- **Backward crosstalk:** Travels opposite direction
- On PCB traces, backward crosstalk typically dominates

### Mitigation Strategies

**PCB Design:**
- **3W Rule:** Trace spacing ≥ 3× trace width (reduces crosstalk to <1%)
- **Ground planes:** Shield between layers, controlled return paths
- **Perpendicular routing:** Cross sensitive traces at 90° to aggressors
- **Differential pairs:** Route as matched pairs, maintain constant spacing

**Cabling:**
- **Twisted pairs:** Equal coupling to both wires → common-mode noise
- **Differential sensing:** Rejects common-mode (V+ - V-), cancels crosstalk
- **Shielded cables:** Ground shield at one point, blocks radiation

**Real-World Application - Inverters/Motor Drives:**

Crosstalk hierarchy (worst to best):
1. Gate drive ↔ Gate drive (can cause shoot-through, destruction!)
2. High-current switching → Current sense (false readings, control instability)
3. PWM switching → Voltage/analog feedback (noisy sensors)
4. High dI/dt → Communication lines (CAN, RS-485 errors)

**Most effective mitigation:** Differential sensing with twisted pair
- Works on PCB traces AND cabling
- Provides common-mode rejection
- Industry standard for robust sensing

---

## Attenuation and Dispersion

### Attenuation (Signal Loss)

Signal gets **weaker** over distance due to:

**Resistive losses:** I²R heating in conductor
- Worse at DC and low frequencies
- Thinner traces = more resistance

**Dielectric losses:** Energy absorbed by PCB material
- Worse at high frequencies
- Cheap FR4 has higher loss than expensive materials (Rogers, Megtron)

**Radiation losses:** Energy radiated as EM waves
- Reduced by proper impedance control

**Result:** 5V signal arrives as 4V or 3V after long trace/cable

### Dispersion (Frequency-Dependent Loss)

**High frequencies attenuated more than low frequencies**

A square wave edge is the sum of many frequencies:
- Fundamental + 3rd harmonic + 5th harmonic + 7th harmonic...
- High harmonics give the edge its "sharpness"

**After traveling through lossy channel:**
- High harmonics reduced/removed (acts like low-pass filter)
- Edge becomes slow and rounded
- Loss of high-frequency content

**Example:**
- Transmitter: 1ns rise time (sharp edge)
- After 50cm trace: 3ns rise time (rounded, slow edge)

### Equalization (The Fix)

**Problem:** High frequencies lost → slow, weak edges

**Solution:** Boost high frequencies to compensate

**Pre-Emphasis (at transmitter):**
- Intentionally over-drive the edges
- After lossy channel, edges arrive at correct amplitude
- Used in PCIe, USB 3.x, HDMI

**De-Emphasis (at transmitter):**
- Reduce flat portions, keep edges full amplitude
- Lower DC power consumption
- Common in modern interfaces

**Receive Equalization (at receiver):**
- Apply high-pass filter
- Boost attenuated high frequencies
- Adaptive: measures channel, adjusts coefficients
- PCIe, Ethernet 10G+ use sophisticated adaptive equalization

**Why Equalization is #1 Solution:**
- Built into PHY chips (free to enable)
- Software configurable
- Adapts to specific channel
- More practical than redesigning board or using expensive materials

---

## High-Speed Interface Examples

### PCIe (Peripheral Component Interconnect Express)

**Speeds:**
- Gen 1: 2.5 GT/s
- Gen 2: 5 GT/s (2× faster edges, more dispersion problems)
- Gen 3: 8 GT/s
- Gen 4: 16 GT/s

**Key Techniques:**
- Differential signaling (all lanes are differential pairs)
- Adaptive equalization (link training phase)
- De-emphasis at transmitter
- Continuous-time linear equalization (CTLE) at receiver

**Common Issue:** Works at Gen 1, fails at Gen 2
- **Cause:** Dispersion/attenuation at higher frequencies
- **Solution:** Enable/tune equalization settings

### Ethernet

**1000BASE-T (Gigabit over Cat5e, 100m max):**
- PAM-5 encoding (5 voltage levels instead of 2)
- Reduces baud rate: 125 MBaud for 1 Gbps
- Forward error correction (FEC)
- Echo cancellation (full-duplex on same wires)
- Adaptive equalization

**10GBASE-T (10 Gigabit):**
- Even more sophisticated encoding (PAM-16)
- Advanced DSP at receiver
- Requires Cat6a cable (lower loss)

**Why 100m specifically?**
- Attenuation budget: Can tolerate -20 to -30 dB loss
- Timing budget for collision detection
- SNR requirements

---

## What to Look For With Your Oscilloscope

### Measuring Signal Integrity Issues

**Reflections:**
- **Overshoot:** Voltage spikes above target (e.g., 10V instead of 5V)
- **Undershoot:** Voltage dips below target (negative spikes)
- **Ringing:** Multiple oscillations settling to final value
- **Settling time:** Duration until signal stabilizes

**Crosstalk:**
- Measure victim line while aggressor switches
- Look for voltage spikes correlated with aggressor edges
- Can be positive or negative polarity

**Attenuation/Dispersion:**
- Compare rise time at transmitter vs receiver
- Measure signal amplitude reduction
- Look for rounded, slow edges (high-frequency loss)

### Oscilloscope Setup Tips

**For digital signals:**
- Use **edge trigger** on signal of interest
- Set timebase to see 2-3 rising/falling edges
- For reflections: Zoom to see overshoot/ringing detail
- Capture single-shot to see worst-case behavior

**Differential measurements:**
- Use **math function** (CH1 - CH2) for differential signals
- Look at both common-mode and differential-mode

**Bandwidth requirements:**
- Need 3-5× signal bandwidth for accurate rise time measurement
- 1ns rise time → need 350 MHz+ oscilloscope bandwidth

---

## Quick Reference Tables

### Transmission Line Rules of Thumb

| Parameter | Typical Value | Notes |
|-----------|--------------|-------|
| **Propagation speed** | 20 cm/ns | PCB traces (FR4) |
| **PCB trace impedance** | 50Ω or 75Ω | Design controlled |
| **Coax cable** | 50Ω (RF), 75Ω (video) | Standard types |
| **Twisted pair** | 100Ω (Ethernet), 120Ω (CAN) | Differential |
| **Becomes transmission line when** | Rise time ≈ Propagation delay | Speed-critical threshold |

### PCB Design Rules

| Rule | Value | Purpose |
|------|-------|---------|
| **Trace spacing (3W)** | ≥ 3× trace width | Crosstalk <1% |
| **Critical spacing (5W)** | ≥ 5× trace width | Sensitive analog/RF |
| **Differential pair spacing** | Constant, matched lengths | Maintain differential impedance |
| **Via count** | Minimize on high-speed traces | Each via adds discontinuity |

### Reflection Coefficient Quick Reference

| ρ Value | Meaning | Overshoot |
|---------|---------|-----------|
| **0** | Perfect match | None |
| **+0.2** | Slight mismatch | 20% |
| **+0.5** | Moderate mismatch | 50% |
| **+1.0** | Open circuit | 100% (voltage doubles) |
| **-1.0** | Short circuit | Signal forced to 0V |

---

## Common Patterns and Troubleshooting

### Symptom: Overshoot and Ringing

**Likely cause:** Reflections (impedance mismatch)

**Check:**
1. Is load impedance matched to line? (Calculate ρ)
2. Is trace properly terminated?
3. Are there impedance discontinuities? (vias, connectors, trace width changes)

**Solutions:**
- Add parallel termination (R = Z₀ to ground)
- Add series termination at source
- Use AC termination for power savings
- Improve PCB layout (constant impedance)

### Symptom: Works at Low Speed, Fails at High Speed

**Likely cause:** Dispersion/attenuation

**Check:**
1. Measure rise time at transmitter vs receiver
2. Measure signal amplitude loss
3. Check PCB material quality (FR4 vs low-loss)

**Solutions:**
- Enable transmitter pre-emphasis/de-emphasis
- Enable receiver equalization
- Shorten trace length
- Upgrade to low-loss PCB material (expensive)
- Reduce data rate (last resort)

### Symptom: Noise on Quiet Signals

**Likely cause:** Crosstalk

**Check:**
1. Are there high dI/dt traces nearby?
2. Is spacing sufficient? (3W rule)
3. Are grounds properly connected?

**Solutions:**
- Increase trace spacing (3W minimum, 5W for critical)
- Route perpendicular to aggressors
- Use ground plane for shielding
- Use twisted pair + differential sensing (for cabling)
- Add shielding to cables

### Symptom: Communication Errors, Bus Failures

**Examples:** CAN bus, RS-485, Ethernet

**Likely causes:** 
- Missing termination (reflections)
- Cable too long (attenuation)
- Improper grounding (noise/crosstalk)

**Check:**
1. Are termination resistors installed and correct value?
2. Is cable length within spec?
3. Cable quality adequate for data rate?
4. Grounds connected properly (single point for shields)?

---

## Real-World Applications

### Motor Inverters / Power Electronics

**Critical SI issues:**
1. **Gate drive crosstalk** → Can cause shoot-through (destruction!)
2. **Current sense crosstalk** → False readings, control instability
3. **PWM noise** → Corrupts analog feedback

**Best practices:**
- Twisted pair for gate drives
- Differential current sensing (reject common-mode)
- Separate analog and digital grounds
- Shield motor phase wires

### High-Speed Digital (PCIe, USB, HDMI)

**Critical SI issues:**
1. **Dispersion** → Slow edges at high data rates
2. **Reflections** → Eye diagram closure
3. **Crosstalk** → Inter-lane interference

**Best practices:**
- Controlled impedance throughout
- Matched differential pairs
- Enable equalization
- Short trace lengths
- Quality connectors (impedance matched)

### Networking (Ethernet, CAN, RS-485)

**Critical SI issues:**
1. **Long cables** → Attenuation
2. **Multi-drop** → Multiple reflections
3. **EMI susceptibility** → Industrial environments

**Best practices:**
- Proper termination (CAN: 120Ω at each end)
- Twisted pair cabling
- Differential signaling
- Respect cable length limits
- Quality cables for data rate

---

## Key Takeaways

1. **Speed is about rise time, not frequency** - Fast edges create high-frequency content
2. **Transmission lines emerge when rise time ≈ propagation delay** - The wire is no longer "just a wire"
3. **Characteristic impedance is a physical property** - Determined by geometry, constant across frequencies
4. **Reflections always happen** - Only matter when significant relative to signal period
5. **Impedance matching eliminates reflections** - Make ρ = 0 through termination
6. **Crosstalk requires electromagnetic coupling** - Minimize with spacing, shielding, differential sensing
7. **Dispersion is frequency-dependent loss** - High frequencies attenuated more (rounds edges)
8. **Equalization compensates for channel loss** - Boost high frequencies at TX or RX
9. **Differential sensing rejects common-mode noise** - Most effective crosstalk mitigation
10. **Real-world designs use multiple techniques** - Termination + spacing + equalization + differential

---

## Next Steps for Learning

### When Your Oscilloscope Arrives

**First measurements:**
1. Measure a known good digital signal (Arduino, Raspberry Pi GPIO)
2. Observe rise time, overshoot, ringing
3. Try different probe positions (near TX vs near RX)
4. Add/remove termination, see the effect

**PROJECT-1 Goals (PCIe on Raspberry Pi 5):**
- Measure PCIe differential signals
- Observe equalization effects at different Gen speeds
- Document signal quality (rise time, amplitude, overshoot)
- Compare Gen 1 vs Gen 2 signal characteristics

### Topics for Future Lessons

- **Eye diagrams and jitter** (LESSON-SI-03)
- **PCIe protocol and physical layer** (LESSON-PCIE-01, LESSON-PCIE-02)
- **BER testing fundamentals** (LESSON-TEST-01)
- **Power integrity** (LESSON-ADV-02)

---

## Resources

### Books
- "Signal Integrity Simplified" by Eric Bogatin (industry standard)
- "High-Speed Digital Design: A Handbook of Black Magic" by Howard Johnson

### Online
- PCIe specification (PCI-SIG website)
- Ethernet standards (IEEE 802.3)
- Application notes from chip vendors (TI, Analog Devices, Maxim)

### Tools
- Online impedance calculators
- Smith chart tools
- Eye diagram analysis
- SPICE simulation for SI analysis

---

**Document Version:** 1.0  
**Lesson:** LESSON-SI-01  
**Created:** February 2026  
**Portfolio:** Hardware Test Engineer Development Plan
