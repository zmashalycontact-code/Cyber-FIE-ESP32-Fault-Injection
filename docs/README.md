# ⚡ Cyber-FIE: Software-Induced Memory Fault Injection Framework
> **Target:** AES-256 Resilience on Bare-Metal ESP32-D0WDQ6[cite: 7, 8]  
> **Threat Model:** Post-Exploitation Arbitrary Code Execution (ACE)

## 🧠 Core Architecture & Objective
Cyber-FIE is a C++ Software-Induced Fault Injection (SIFI) framework engineered to evaluate cryptographic execution security. Bypassing standard OS-level memory abstractions via volatile C++ pointer arithmetic, the framework directly corrupts the AES Key Schedule residing in Static RAM (SRAM) without physical hardware tampering (e.g., laser/voltage glitching)[cite: 7, 8].

The ultimate objective is to force precise functional failures, generating the mathematical artifacts (corrupted ciphertexts) strictly required to facilitate a **Differential Fault Analysis (DFA)** pipeline[cite: 7, 8].

---

## 📊 Empirical Hardware Validation (3,560 Live Trials @ 240MHz)[cite: 7, 8]

Unlike deterministic software simulations, Cyber-FIE was validated on live silicon, capturing stochastic behavior, bus arbitration latency, and Hardware Watchdog Timer (WDT) interference[cite: 10].

| Metric / Finding | Measured Value | Engineering Implication |
| :--- | :--- | :--- |
| **Critical Failure Rate** | **12.3%** (95% CI: 11.2% - 13.4%)[cite: 7] | Corrupts AES-256 key schedule, enabling DFA key recovery[cite: 7]. |
| **System Crash Rate** | **28.0%**[cite: 7] | Triggered Hardware Watchdog Timer (WDT) resets[cite: 7]. |
| **Legacy Cipher Failure** | **100%** (DES/3DES)[cite: 7] | Legacy primitives possess zero inherent SRAM corruption resilience[cite: 10]. |
| **Window of Vulnerability** | **14.7 µs (3,528 clock cycles)**[cite: 7, 8] | Strict timing constraint required to bypass WDT resets[cite: 10]. |

---

## 🛡️ "The Shield": Software-Based Fault Tolerance (SBFT)[cite: 7, 8]
To mitigate the identified vulnerabilities, we architected a dual-layer SBFT defense mechanism capable of real-time state restoration with minimal computational penalty[cite: 7, 11].

### Defense Performance Metrics[cite: 11]:
- **State Restoration Rate:** `94.2%` against single-byte faults (via Reverse-XOR recovery)[cite: 7].
- **Detection Latency:** `2.1 µs` (504 cycles) preemptive interception[cite: 11].
- **Throughput Drop:** `-1.5%` (Negligible for constrained IoT deployments)[cite: 11].
- **Memory Footprint:** `+4 Bytes` SRAM allocation[cite: 11].

### Shield Execution Pipeline
```mermaid
graph TD
    A[Boot: Generate Golden Hash] --> B[Runtime: Compute Current Hash]
    B --> C{Match?}
    C -->|Yes| D[Execute AddRoundKey & Output Safe Ciphertext]
    C -->|No| E[Layer 1: Intercept Fault via Interrupt]
    E --> F[Layer 2: Apply Reverse-XOR Mask]
    F -->|94.2% State Restored| D
    
    style E fill:#b71c1c,stroke:#333,stroke-width:2px,color:#fff
    style F fill:#1565c0,stroke:#333,stroke-width:2px,color:#fff