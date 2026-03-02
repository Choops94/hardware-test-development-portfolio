# Hardware Test Engineer Development Plan

## Overview

**Current Position:** Electronic Engineering department, testing/proving hardware systems, motor development background  
**Target Field:** Hardware test engineering with focus on high-speed digital interfaces, test automation, and signal integrity  
**Primary Interest:** Test development roles that blend hardware and software (hands-on technical work)  
**Key Strength:** Proven ability to learn independently and build practical skills

**Development Philosophy:** Build demonstrable skills through portfolio projects and structured learning. Focus on practical understanding over theoretical mastery. Create reference documentation as proof of learning.

---

## Equipment Shopping List

### Must-Buy (Priority Order)
- **USB Oscilloscope - Hantek 6022BE:** £50-60 (20MHz, 2-channel, laptop-connected)
- **Logic Analyzer - 24MHz 8-channel clone:** £15-20
- **Soldering Iron - Pinecil or TS100:** £25-30 (if current iron inadequate)
- **Oscilloscope Probes:** £10-15 (if not included with scope)

### Nice-to-Have (Remaining Budget)
- **PCIe Riser Card/Extender:** £10-15
- **Ethernet PHY Breakout Board:** £15-20
- **Quality Breadboard Supplies:** £20-30 (jumpers, connectors)
- **Books (used) - "Signal Integrity Simplified" by Eric Bogatin:** £20-30

**Total Budget:** £200  
**Equipment Already Owned:** Raspberry Pi 5, laptop, Arduino, sensors, basic breadboard supplies

---

## Learning Methodology with Claude

### Interactive Learning Pattern
1. Request lesson on specific topic with context about current knowledge level
2. Engage in Q&A to clarify understanding
3. Test knowledge through examples and questions
4. Request creation of reference document in markdown format
5. Save reference document to GitHub portfolio under appropriate folder

### Reference Document Style
- Similar format to existing project reference docs (python-basics-reference.md, etc.)
- Structured with clear sections and examples
- Command reference sections where applicable
- Quick tips and common mistakes
- Builds permanent knowledge base

### Starting a Learning Session
Use lesson ID from this plan (e.g., "I want to start LESSON-SI-01 on Signal Integrity Fundamentals")

---

## Projects Portfolio

### PROJECT-1: PCIe Signal Characterization on Raspberry Pi 5
**Objectives:** Demonstrate understanding of PCIe protocol and signal integrity without expensive equipment  
**Key Deliverables:**
- Signal measurements using USB oscilloscope
- Documentation of PCIe signal characteristics (rise time, overshoot, ringing)
- Analysis write-up showing understanding of transmission line effects
- Reference document: `pcie-characterization-reference.md`

**Skills Developed:**
- PCIe protocol understanding
- Signal integrity measurement
- Oscilloscope proficiency
- Technical documentation

**GitHub Location:** `signal-integrity-study/raspberry-pi-pcie/`

---

### PROJECT-2: Ethernet BER Testing Framework
**Objectives:** Demonstrate bit error rate testing methodology and test automation  
**Key Deliverables:**
- Python automation scripts for Ethernet stress testing
- Test framework measuring packet loss and errors under various conditions
- Automated test reports with graphs
- Reference document: `ber-testing-methodology-reference.md`

**Skills Developed:**
- BER testing concepts
- Network protocol analysis
- Test automation
- Data analysis and reporting

**GitHub Location:** `ethernet-ber-testing/`

---

### PROJECT-3: Professional Test Automation GUI
**Objectives:** Create production-quality test control interface  
**Key Deliverables:**
- Python GUI application (tkinter or PyQt)
- Features: test configuration, live plotting, automated test sequences, PDF reports
- SQLite database for historical data
- User documentation
- Reference document: `test-automation-gui-reference.md`

**Skills Developed:**
- GUI development
- Test framework architecture
- Professional software practices
- Technical documentation

**GitHub Location:** `test-automation-gui/`

---

### PROJECT-4: Protocol Analysis - I2C to I3C Bridge
**Objectives:** Demonstrate protocol learning ability and automation  
**Key Deliverables:**
- I2C test automation using logic analyzer
- Comparative analysis document: I2C vs I3C
- Explanation of how test approach would adapt for I3C
- Reference document: `i2c-i3c-protocols-reference.md`

**Skills Developed:**
- Protocol analysis
- Logic analyzer usage
- Technical comparison skills
- Future-proofing knowledge

**GitHub Location:** `signal-integrity-study/protocol-analysis/`

---

### PROJECT-5: Laptop PCIe Device Analysis
**Objectives:** Software-based PCIe understanding without hardware probing  
**Key Deliverables:**
- Linux scripts for PCIe device enumeration and monitoring
- Analysis of PCIe link training and configuration
- Performance benchmarking of NVMe/other PCIe devices
- Reference document: `pcie-software-analysis-reference.md`

**Skills Developed:**
- PCIe protocol depth
- Linux system tools
- Bash scripting
- Performance analysis

**GitHub Location:** `signal-integrity-study/pcie-software-analysis/`

---

## Learning Sessions

### Signal Integrity Foundation

**LESSON-SI-01: Signal Integrity Fundamentals**  
**Prerequisites:** None  
**Learning Outcomes:**
- Understand transmission line effects
- Explain impedance, reflections, and termination
- Recognize signal integrity issues in oscilloscope captures

**Reference Document:** `signal-integrity-fundamentals-reference.md`

---

**LESSON-SI-02: Rise Time and Signal Quality**  
**Prerequisites:** LESSON-SI-01  
**Learning Outcomes:**
- Calculate required bandwidth from rise time
- Understand relationship between rise time and signal integrity
- Identify overshoot, undershoot, and ringing

**Reference Document:** `rise-time-analysis-reference.md`

---

**LESSON-SI-03: Eye Diagrams and Jitter**  
**Prerequisites:** LESSON-SI-01, LESSON-SI-02  
**Learning Outcomes:**
- Interpret eye diagrams
- Understand jitter sources and measurement
- Relate eye diagram quality to BER

**Reference Document:** `eye-diagrams-jitter-reference.md`

---

### High-Speed Protocols

**LESSON-PCIE-01: PCIe Protocol Overview**  
**Prerequisites:** None  
**Learning Outcomes:**
- Understand PCIe layered architecture
- Explain link training and enumeration
- Describe PCIe packet structure

**Reference Document:** `pcie-protocol-reference.md`

---

**LESSON-PCIE-02: PCIe Physical Layer**  
**Prerequisites:** LESSON-PCIE-01, LESSON-SI-01  
**Learning Outcomes:**
- Understand PCIe differential signaling
- Explain equalization and de-emphasis
- Describe spread spectrum clocking

**Reference Document:** `pcie-physical-layer-reference.md`

---

**LESSON-ETH-01: Ethernet Protocol Fundamentals**  
**Prerequisites:** None  
**Learning Outcomes:**
- Understand Ethernet frame structure
- Explain MAC and PHY layers
- Describe auto-negotiation process

**Reference Document:** `ethernet-protocols-reference.md`

---

**LESSON-ETH-02: High-Speed Ethernet (10G/25G/100G)**  
**Prerequisites:** LESSON-ETH-01, LESSON-SI-01  
**Learning Outcomes:**
- Understand PAM4 encoding
- Explain FEC (Forward Error Correction)
- Describe high-speed Ethernet PHY requirements

**Reference Document:** `high-speed-ethernet-reference.md`

---

### Test Methodology

**LESSON-TEST-01: BER Testing Fundamentals**  
**Prerequisites:** None  
**Learning Outcomes:**
- Understand bit error rate concepts
- Explain PRBS patterns
- Describe statistical confidence in BER measurements

**Reference Document:** `ber-testing-fundamentals-reference.md`

---

**LESSON-TEST-02: Test Equipment Overview**  
**Prerequisites:** None  
**Learning Outcomes:**
- Understand BERT (Bit Error Rate Tester) operation
- Explain VNA (Vector Network Analyzer) basics
- Describe high-bandwidth oscilloscope requirements

**Reference Document:** `test-equipment-overview-reference.md`

---

**LESSON-TEST-03: Compliance Testing and Standards**  
**Prerequisites:** LESSON-PCIE-01, LESSON-ETH-01  
**Learning Outcomes:**
- Understand IEEE and PCI-SIG test specifications
- Explain compliance test methodology
- Describe test coverage analysis

**Reference Document:** `compliance-testing-reference.md`

---

**LESSON-TEST-04: Test Automation Architecture**  
**Prerequisites:** None  
**Learning Outcomes:**
- Design scalable test frameworks
- Implement proper error handling in test code
- Create professional test reports

**Reference Document:** `test-automation-architecture-reference.md`

---

### Advanced Topics

**LESSON-ADV-01: Memory Interface Basics (LPDDR/HBM)**  
**Prerequisites:** LESSON-SI-01  
**Learning Outcomes:**
- Understand memory interface architectures
- Explain timing requirements and constraints
- Describe common test challenges

**Reference Document:** `memory-interfaces-reference.md`

---

**LESSON-ADV-02: Power Integrity Fundamentals**  
**Prerequisites:** LESSON-SI-01  
**Learning Outcomes:**
- Understand PDN (Power Distribution Network)
- Explain decoupling strategies
- Describe high-current transient requirements

**Reference Document:** `power-integrity-reference.md`

---

**LESSON-ADV-03: Production Test Methods (ICT/SLT)**  
**Prerequisites:** None  
**Learning Outcomes:**
- Understand In-Circuit Test (ICT) methodology
- Explain System Level Test (SLT) approach
- Describe test coverage optimization

**Reference Document:** `production-test-methods-reference.md`

---

## Development Milestones (Time-Ambiguous)

### Milestone 1: Foundation Established
**Definition of Complete:**
- PROJECT-3 (Test Automation GUI) complete and documented
- 3-4 reference documents created from learning sessions
- GitHub portfolio structure established and populated
- Basic understanding of signal integrity concepts

**What This Unlocks:**
- Ready to begin hardware-focused projects
- Can discuss test automation intelligently
- Portfolio demonstrates learning ability

---

### Milestone 2: Hardware Integration
**Definition of Complete:**
- USB oscilloscope acquired and proficient in basic use
- PROJECT-1 (PCIe Characterization) complete
- LESSON-SI-01, LESSON-SI-02, LESSON-PCIE-01 completed
- Can measure and interpret basic digital signals

**What This Unlocks:**
- Can discuss high-speed digital interfaces with understanding
- Demonstrated hands-on signal integrity work
- Ready for protocol-specific projects

---

### Milestone 3: Portfolio Ready
**Definition of Complete:**
- 3+ projects complete and well-documented
- 8+ reference documents covering key topics
- Learning journal showing clear progression
- GitHub portfolio professional and navigable

**What This Unlocks:**
- Competitive candidate for test engineering roles
- Strong portfolio to reference in applications
- Confidence to discuss technical topics in interviews

---

### Milestone 4: Expert Demonstrator
**Definition of Complete:**
- PROJECT-2 (BER Testing) complete with published analysis
- All foundation learning sessions complete
- Can explain complex concepts clearly with examples
- Portfolio shows depth in multiple areas

**What This Unlocks:**
- Strong differentiated candidate
- Deep enough knowledge to contribute from day one
- Credible self-taught expertise in relevant domains

---

## Weekly Task Templates

### 1 Hour Per Week Pattern
- **Week 1-2:** Complete one learning session, create reference doc
- **Week 3-4:** Work on active project (small increment)
- Alternate between learning and building

### 2 Hours Per Week Pattern
- **1 hour:** Learning session or studying
- **1 hour:** Active project work
- Maintains balance between theory and practice

### 3 Hours Per Week Pattern
- **1 hour:** Learning session
- **2 hours:** Active project development
- Or: 3 hours focused project push when needed

**Key Principle:** Consistent small progress beats sporadic intense effort. Schedule recurring blocks.

---

## GitHub Portfolio Structure

### Recommended Organization
```
hardware-test-development-portfolio/
├── README.md                          # Portfolio overview
├── learning-journal.md                # Ongoing progress log
│
├── reference-docs/                    # All learning reference documents
│   ├── signal-integrity-fundamentals-reference.md
│   ├── pcie-protocol-reference.md
│   ├── ber-testing-fundamentals-reference.md
│   └── ...
│
├── signal-integrity-study/
│   ├── raspberry-pi-pcie/            # PROJECT-1
│   │   ├── README.md
│   │   ├── measurements/
│   │   ├── analysis/
│   │   └── scripts/
│   ├── protocol-analysis/            # PROJECT-4
│   └── pcie-software-analysis/       # PROJECT-5
│
├── ethernet-ber-testing/             # PROJECT-2
│   ├── README.md
│   ├── automation-scripts/
│   ├── test-reports/
│   └── analysis/
│
├── test-automation-gui/              # PROJECT-3
│   ├── README.md
│   ├── src/
│   ├── docs/
│   ├── screenshots/
│   └── examples/
│
└── equipment-notes/
    ├── oscilloscope-setup.md
    ├── logic-analyzer-setup.md
    └── measurement-techniques.md
```

### Naming Conventions
- **Projects:** Descriptive folder names, hyphen-separated
- **Reference docs:** `topic-name-reference.md` format
- **README files:** Every project folder needs one
- **Scripts:** Clear names indicating purpose, `test_`, `analyze_`, etc.

### What to Commit
- All source code and scripts
- Documentation and reference materials
- Analysis and results (within reason for file size)
- Learning journal updates
- Equipment setup guides

### What NOT to Commit
- Large binary data files (>10MB)
- Temporary test outputs
- API keys or credentials
- Vendor documentation (link instead)

---

## Project Priority Recommendation

**Start Immediately:** PROJECT-3 (Test Automation GUI)  
**Reason:** No equipment needed, builds on existing work, quick wins

**Parallel Track:** Order equipment, begin signal integrity learning sessions

**Second Project:** PROJECT-1 (PCIe Characterization) when equipment arrives

**Third Project:** PROJECT-2 (BER Testing) or PROJECT-5 (software-based)

**Fourth Project:** PROJECT-4 (Protocol Analysis)

**Flexibility:** Adjust order based on interest and opportunities. All projects build valuable skills.

---

## Notes on Learning Sessions

- Sessions can be taken in any order within their category
- Prerequisites are suggestions, not hard requirements
- Some sessions complement specific projects (noted in session description)
- Create reference document immediately after each session while knowledge is fresh
- Review reference docs periodically to reinforce learning

---

## Development Plan Maintenance

This document is a living plan. Update as:
- Projects are completed
- New learning sessions are added
- Equipment is acquired
- Priorities shift based on opportunities
- Skills develop and gaps close

**Version:** 1.0  
**Created:** February 2026  
**Last Updated:** February 2026
