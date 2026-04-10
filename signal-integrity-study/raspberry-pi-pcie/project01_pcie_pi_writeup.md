# PCIe Signal Characterization — Raspberry Pi 5
**PROJECT-1 Reference Document**
**GitHub:** `signal-integrity-study/raspberry-pi-pcie/`

---

## Project Overview

This project characterizes the PCIe interface on a Raspberry Pi 5 using a Pimoroni NVMe Base (PIM699) and a Hantek 6022BE USB oscilloscope. The goal was to demonstrate practical understanding of PCIe signal integrity, link training, and oscilloscope measurement technique — including honest documentation of equipment limitations.

**Hardware:**
- Raspberry Pi 5 (BCM2712 SoC)
- Pimoroni NVMe Base (PIM699) — M.2 M-key, PCIe x1
- SK Hynix BC501A 128GB NVMe SSD (M.2 2230)
- Hantek 6022BE USB oscilloscope (20MHz bandwidth, 2 channel)

**Software:**
- Raspberry Pi OS (Linux kernel 6.x)
- OpenHantek6022 v3.4.0 on Linux Mint (Ubuntu Noble base)
- Tools: `lspci`, `dmesg`, `nvme-cli`, `RPi.GPIO`

---

## PCIe Link Configuration

The Pi 5 exposes a single PCIe lane via a 16-pin 0.5mm pitch FFC connector. The following was added to `/boot/firmware/config.txt` to enable it:

```
dtparam=pciex1
dtparam=pciex1_gen=2
```

### Link Topology (from `lspci -vv`)

| Component | Details |
|-----------|---------|
| Root complex | Broadcom BCM2712 PCIe Bridge (domain 0001) |
| Endpoint | SK Hynix BC501A NVMe SSD (domain 0001, bus 01) |
| Internal bus | RP1 South Bridge (domain 0002) — handles USB/GPIO/Ethernet, separate from external FFC connector |

### Negotiated Link State

| Parameter | Value | Notes |
|-----------|-------|-------|
| Speed | 5 GT/s (PCIe Gen 2) | Drive capable of 8 GT/s (Gen 3) — downgraded by Pi hardware limit |
| Width | x1 | Drive capable of x4 — downgraded by Pi hardware limit |
| De-emphasis | -6dB | Active transmitter equalization compensating for FFC cable |
| Error counters | All zero | DLP, TLP, ECRC all clear — healthy link |
| ASPM state | L1 enabled | Link drops to low-power state when idle |
| Equalization | Not completed | Expected — full equalization only runs at Gen 3 |

The "downgraded" flags on speed and width are expected and correct — the Pi 5 FFC connector is a hardware-limited x1 Gen 2 interface. The drive negotiates down to match the host capability.

**Confirmed via sysfs:**
```bash
cat /sys/bus/pci/devices/0001:01:00.0/current_link_speed
# 5.0 GT/s PCIe

cat /sys/bus/pci/devices/0001:01:00.0/current_link_width
# 1
```

### Kernel Enumeration (from `dmesg`)

```
[0.409849] nvme nvme0: pci function 0001:01:00.0
[0.409865] nvme 0001:01:00.0: enabling device (0000 -> 0002)
[0.420092] nvme nvme0: 4/0/0 default/read/poll queues
[0.424132]  nvme0n1: p1 p2
```

Clean first-time enumeration — no errors, no retries. Four I/O queues established, two partitions found.

### Device Identity (from `nvme id-ctrl`)

| Field | Value |
|-------|-------|
| Model | BC501A NVMe SK Hynix 128GB |
| Firmware | 80001101 |
| Vendor ID | 0x1c5c |
| Max transfer size | 256KB (mdts=6) |

### Sequential Read Throughput

```bash
sudo dd if=/dev/nvme0n1 of=/dev/null bs=1M count=512 status=progress
# 406 MB/s
```

Theoretical maximum for PCIe Gen 2 x1 after 8b/10b encoding overhead: 500 MB/s (4 Gbit/s usable). Measured throughput of 406 MB/s = **81% of theoretical maximum** — a well-performing link with realistic overhead.

---

## Bandwidth Analysis — Why the Scope Cannot Resolve PCIe Signals

A key finding of this project is understanding the relationship between PCIe signal frequencies and oscilloscope bandwidth.

### 8b/10b Encoding and Wire Frequency

PCIe Gen 1/2 uses 8b/10b encoding — for every 8 bits of data, 10 bits are transmitted on the wire. This means:

- The wire bit rate = the stated GT/s figure (5 GT/s for Gen 2)
- Usable data rate = 5 GT/s × (8/10) = **4 Gbit/s**
- The 20% overhead is used for DC balance and error detection

### Fundamental Clock Frequencies

| Generation | Wire rate | Fundamental frequency | 3rd harmonic (min BW needed) |
|------------|-----------|----------------------|------------------------------|
| Gen 1 | 2.5 GT/s | 1.25 GHz | 3.75 GHz |
| Gen 2 | 5.0 GT/s | 2.50 GHz | 7.50 GHz |
| Gen 3 | 8.0 GT/s | 4.00 GHz | 12.0 GHz |

The fundamental frequency = wire rate ÷ 2 (two bit periods per cycle: HIGH + LOW).

**The Hantek 6022BE at 20 MHz is 375× below the minimum bandwidth needed to resolve Gen 2 bit transitions.** This was confirmed experimentally — probing the PCIe TX pins and REFCLK pin on the FFC connector produced a DC-like averaged reading rather than a resolvable waveform.

The 100MHz REFCLK signal on the FFC connector (pin 8) showed a ~560mV DC-like reading — the scope's 20MHz low-pass response averaging the 100MHz clock into what appears as a DC level. Signal presence was confirmed by the amplitude reading, but individual cycles could not be resolved.

---

## Oscilloscope Measurements — GPIO Signals

Since PCIe data signals exceed the scope's bandwidth, measurements were performed on Pi 5 GPIO signals to demonstrate oscilloscope technique and signal integrity concepts. These measurements validate the scope setup and provide baseline signal integrity observations.

**Signal generation:**
```python
import RPi.GPIO as GPIO
import time
GPIO.setmode(GPIO.BCM)
GPIO.setup(17, GPIO.OUT)
p = GPIO.PWM(17, frequency)
p.start(50)
time.sleep(30)
GPIO.cleanup()
```

**Probe setup:** Hantek 6022BE, 10x probe (matched in OpenHantek software), ground clip on GPIO pin 6.

### Measurement 1 — 100Hz Square Wave

| Parameter | Value |
|-----------|-------|
| Frequency | 100 Hz |
| Vpp | ~5.12V (includes overshoot) |
| Vrms | ~2.6V |
| Overshoot | Present, recovers quickly |

*Screenshot: `gpio_100hz.png`*

### Measurement 2 — 1kHz Square Wave

| Parameter | Value |
|-----------|-------|
| Frequency | 1.00 kHz |
| Vpp | ~5.23V (includes overshoot) |
| Vrms | ~2.7V |
| Rise time | ~80ns (cursor measurement, 10% to 90%) |
| Overshoot | Present on both edges, gradual recovery |

*Screenshot: `gpio_1khz_risetime.png`*

**Rise time bandwidth check:**
```
Minimum BW = 0.35 / rise time = 0.35 / 80ns = 4.375 MHz
Hantek bandwidth = 20 MHz — sufficient to capture this signal accurately
```

### Measurement 3 — 10kHz Square Wave

| Parameter | Value |
|-----------|-------|
| Frequency | 10.00 kHz |
| Vpp | ~4.93V |
| Vrms | ~2.9V |
| Waveform shape | Visibly degraded — sloped edges, triangular appearance |

*Screenshot: `gpio_10khz.png`*

At 10kHz the signal period is 100µs. The overshoot recovery time becomes a significant fraction of the half-period, meaning the signal never fully settles before the next edge arrives. This produces the observed sloped, triangular waveform — a direct demonstration of rise time limiting.

The higher Vrms (2.9V vs 2.6V at 100Hz) reflects the waveform spending more time at intermediate voltages rather than snapping cleanly between 0V and 3.3V.

---

## Signal Integrity Observations

### Overshoot and Impedance Mismatch

Overshoot was observed on all rising and falling edges of the GPIO square wave. This is consistent with transmission line theory — the GPIO pin drives into an unterminated load (scope probe), creating an impedance mismatch that causes the voltage to overshoot before settling. This is the same mechanism that makes PCIe signal integrity challenging at high speeds.

### Vpp vs Vrms

Vpp measurements on real signals are inflated by overshoot on both edges — the peak-to-peak measurement captures the overshoot spike on the rising edge down to the undershoot on the falling edge. Vrms is a more representative measure of the actual signal energy. On a 3.3V 50% duty cycle square wave: theoretical Vrms = 3.3V × 0.707 = 2.33V. Measured Vrms of 2.6–2.9V reflects additional energy from the overshoot.

### De-emphasis on the PCIe Link

The `-6dB de-emphasis` seen in the `lspci` output is active transmitter equalization. The transmitter reduces the amplitude of repeated bits (the "flat" portions of the signal) relative to transitions, compensating for the low-pass filtering effect of the FFC cable. This was observable in software alone — before any oscilloscope work — demonstrating that signal integrity inferences can be made from link configuration registers.

---

## Physical Probing Notes

The M.2 M-key connector on the Pimoroni NVMe Base presented physical probing challenges:

- The NVMe drive, once installed, covers the top-edge signal pins of the M.2 connector
- Access was obtained via the rear solder pads on the Pimoroni PCB (odd-numbered pins: 1, 3, 5... 75)
- The M-key notch (pins 59–66) was used as a reference for counting to target pins (55, 49, 47)
- A vampire probe tip extension was used to contact the small solder pads
- Ground was taken from Pi GPIO pin 6 (easier to access than the M.2 GND pads)

The FFC cable connecting Pi to Pimoroni board was also probed. Pins 1 and 2 confirmed as 5V DC rails. Pins 6 (PETp0) and 8 (REFCLKP) showed signals present but unresolvable due to scope bandwidth limitation.

---

## Key Learnings

1. **GT/s ≠ MHz** — the wire frequency is GT/s ÷ 2. At 5 GT/s the fundamental is 2.5 GHz, requiring ~7.5 GHz scope bandwidth for faithful capture.

2. **8b/10b encoding adds 25% overhead** — 5 GT/s wire rate yields only 4 Gbit/s of usable data. The "downgraded" flags in lspci are not errors — they indicate correct negotiation to the host's hardware limits.

3. **Software tools reveal signal integrity** — de-emphasis level, equalization state, and error counters are all readable from `lspci` without any oscilloscope, providing signal integrity inferences before hardware probing begins.

4. **Scope bandwidth determines what you can see** — the 20MHz Hantek could not resolve PCIe or REFCLK signals, but cleanly captured GPIO signals up to ~10kHz before rise time limiting became visible.

5. **Vpp is misleading on real signals** — overshoot inflates peak-to-peak readings. Always examine the waveform shape alongside numerical measurements.

6. **Rise time limits signal quality** — at 10kHz, GPIO edges could not settle within the half-period, producing a triangular waveform. At 5 GT/s PCIe, the bit period is 200ps — any settling behaviour must resolve within that window.

---

## Equipment Limitations and Future Work

The Hantek 6022BE (20MHz) is insufficient for direct PCIe signal characterisation. A minimum of 4–8 GHz bandwidth would be required for Gen 2. Future work with higher bandwidth equipment could capture:

- Eye diagrams on the PCIe TX differential pair
- Rise time measurements on PCIe edges
- REFCLK waveform quality (100MHz, within reach of a 500MHz scope)
- PERST# reset pulse capture during link training
- Two-channel differential measurement of TX+/TX− pairs

OpenHantek features to explore in future sessions: spectrum analysis mode, two-channel differential measurement, and eye diagram accumulation.

---

**Document version:** 1.0
**Project:** PROJECT-1 PCIe Signal Characterization
**Date:** April 2026
**Portfolio:** `signal-integrity-study/raspberry-pi-pcie/`
