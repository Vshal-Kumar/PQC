# SIMD-Accelerated Post-Quantum Cryptography (PQC) Framework
## Clean & Minimal Internship Presentation Deck

---

## 📌 Slide 1: Title Page
- **Main Header**: **SIMD-Accelerated Post-Quantum Cryptography Framework**
- **Subtitle**: Heterogeneous x86_64 Server ↔ ARM Client Networking with Formal Verification & Vector Acceleration
- **Presenter**: Cybersecurity & Systems Engineering Intern
- **Organization**: Systems & Post-Quantum Cryptography Research Lab
- **Key Stack**: C11, Linux `io_uring`, OpenSSL 3.0, liboqs, ProVerif, AVX2, ARM NEON
- **Primary Achievement**: Sub-millisecond 4-way PQC Handshake (0.85 ms) & 20 Formally Verified Security Proofs

---

## 📌 Slide 2: Introduction & Project Context
- **About the Project & Domain**:
  - **Internship Focus**: Design, optimization, and formal analysis of post-quantum network protocols.
  - **Core Domain**: Post-Quantum Cryptography (PQC), Microarchitectural Vectorization (SIMD), and Formal Verification (ProVerif).
  - **Deployment Targets**: High-performance x86_64 servers communicating with embedded ARM edge nodes (Raspberry Pi 3B / 4B / 5).
- **Primary Internship Scope**:
  1. **Protocol Engineering**: Build lightweight C11 UDP 4-way handshake using NIST algorithms.
  2. **Hardware Acceleration**: Implement AVX2/BMI2 vectorization for x86 and NEON 128-bit SIMD for ARM.
  3. **Formal Cybersecurity**: Prove secrecy, mutual authentication, and PFS in ProVerif.
  4. **Empirical Benchmarking**: Profile latency, throughput, energy, and memory consumption.

---

## 📌 Slide 3: Problem Statement & Motivation
- **The Quantum Threat**:
  - **Vulnerability of Classical Crypto**: Shor's algorithm efficiently breaks RSA and ECC public-key schemes.
  - **"Harvest Now, Decrypt Later"**: Adversaries store today's encrypted network traffic to decrypt once quantum hardware matures.
  - **NIST Standardization**: Mandates migration to ML-KEM (FIPS 203) for key exchange and ML-DSA (FIPS 204) for digital signatures.
- **Embedded Performance Bottlenecks**:
  - **Large Key & Ciphertext Overhead**: ML-KEM-768 PK = 1,184 B; ML-DSA-65 signature = 3,293 B (vs classical ECC 64 B).
  - **Heavy Lattice Mathematics**: Polynomial vector multiplication and Number Theoretic Transforms (NTT) strain low-power CPUs.
  - **RAM & Latency Limits**: Unoptimized PQC algorithms degrade network connection times on ARM edge hardware.

---

## 📌 Slide 4: Project Scope & Core Engineering Contributions
1. **High-Performance C11 Protocol Engine**:
   - Lightweight 4-way UDP PQC handshake protocol in C11.
   - Sub-millisecond connection time (0.85 ms RTT) over LAN.
   - Decoupled packet headers, nonces, and session key state management.
2. **Dual SIMD Acceleration Pipeline**:
   - Integrated 256-bit AVX2 & BMI2 instructions for x86_64 server hosts.
   - Harnesses 128-bit NEON vectorization and VFPv4 for ARM Cortex nodes.
   - Reduced lattice polynomial multiplication latency by up to 3.2x.
3. **Decoupled POSIX IPC Server Engine**:
   - Utilized Linux `io_uring` asynchronous I/O engine for high socket concurrency.
   - Decoupled UDP core daemon from UI logging via UNIX domain sockets.
   - Integrated `server_rx` display console and `server_tx` command console.
4. **Formally Verified Security Model**:
   - Modeled 4-way handshake in Applied Pi-Calculus using ProVerif.
   - Mathematically verified 20 security properties under Dolev-Yao model.
   - Proven Perfect Forward Secrecy (PFS), Mutual Authentication, and Replay Resistance.

---

## 📌 Slide 5: Methodology & Implementation Pipeline
- **Phase 1: Primitive Selection**: NIST FIPS 203 ML-KEM-768 & FIPS 204 ML-DSA-65 standardized parameters.
- **Phase 2: Formal Verification**: ProVerif Applied Pi-Calculus protocol modeling for Secrecy & Perfect Forward Secrecy.
- **Phase 3: C11 Protocol Engine**: Custom 4-way UDP handshake state machine & ChaCha20-Poly1305 AEAD payload streaming.
- **Phase 4: SIMD Optimization**: AVX2 (x86) and NEON (ARM) vector acceleration for lattice polynomial math.
- **Phase 5: Empirical Benchmarking**: Profiling via `bench_pqc` (microbenchmarks), `bench_nclient` (scale), and `bench_aead` (AEAD).

---

## 📌 Slide 6: System Architecture & Handshake Flow
- **4-Way PQC Handshake Flow**:
  1. `PKT_CLIENT_HELLO`: Client ──► Server: (`KEM_PK_c`, `DSA_PK_c`, `Nonce_c`)
  2. `PKT_SERVER_HELLO`: Server ──► Client: (`DSA_PK_s`, `KEM_CT`, `SessionID`, `Nonce_s`, `Sig_s`)
  3. `PKT_CLIENT_AUTH`: Client ──► Server: (`Sig_c` over full transcript)
  4. `PKT_ACK & Active`: Server ──► Client: Key derived via SHAKE256 KDF. Symmetric AEAD active (`ChaCha20-Poly1305`).
- **Decoupled POSIX IPC Server Engine**:
  - `server` Core Daemon: Listens on UDP port 9877 using Linux `io_uring` async I/O.
  - `server_rx` Console: Connects via `/tmp/pqc_rx.sock` UNIX domain socket to display decrypted text & metrics.
  - `server_tx` Console: Connects via `/tmp/pqc_tx.sock` UNIX domain socket for sending operator commands.
  - **Zero Display Latency**: UI logging completely decoupled from packet routing.

---

## 📌 Slide 7: Cryptographic Specifications

| Primitive | Standard / Algorithm | Security Category | Primary Function | Key / Artifact Sizes |
| :--- | :--- | :--- | :--- | :--- |
| **KEM** | ML-KEM-768 (Kyber768) | NIST Cat 3 (~AES-192) | Ephemeral Key Exchange | PK: 1,184 B \| CT: 1,088 B |
| **DSA** | ML-DSA-65 (Dilithium3) | NIST Cat 3 (~AES-192) | Mutual Auth & Signing | PK: 1,952 B \| Sig: 3,293 B |
| **AEAD** | ChaCha20-Poly1305 | 256-bit Key | Authenticated Encryption | Key: 32 B \| Nonce: 12 B \| Tag: 16 B |
| **Hash & KDF** | SHAKE256 | Extendable Output (XOF) | Transcript & Key Derivation | Variable Output Length |

- **Hybrid Operational Security Guarantee**:
  Combines forward-secret ephemeral ML-KEM-768 key exchange with ML-DSA-65 digital signature authentication. Once established via SHAKE256 KDF, communications switch to ChaCha20-Poly1305 AEAD payload streaming.

---

## 📌 Slide 8: Formal Security Verification (ProVerif)

| Security Property | Result | ProVerif Formal Query |
| :--- | :--- | :--- |
| **Session Key Secrecy** | `VERIFIED` | `query attacker(session_key_secret)` |
| **Client Payload Secrecy** | `VERIFIED` | `query attacker(client_data)` |
| **Server Payload Secrecy** | `VERIFIED` | `query attacker(server_data)` |
| **Injective Mutual Auth** | `VERIFIED` | `inj-event(server_accepts) ==> client_initiates` |
| **Perfect Forward Secrecy** | `VERIFIED` | Phase 1 long-term key leak query |

- **Verification Highlights**:
  - **Dolev-Yao Attacker Model**: Active adversary controls physical network packets.
  - **Phase-Based PFS Proof**: Phase 0 honest sessions; Phase 1 long-term signature key leak. Phase 0 session keys remain secret.
  - **Attack Immunity**: Proven resistant to KEM substitution, UKS attacks, and reflection.

---

## 📌 Slide 9: SIMD Hardware Acceleration & Toolchain
- **SIMD Acceleration Pipeline**:
  - **x86_64 Server**: AVX2 256-bit + BMI2 instruction set. **3.2x Speedup**.
  - **ARM64 Client**: 128-bit NEON SIMD vector registers. **2.8x Speedup**.
  - **ARM32 Client**: NEON + VFPv4 hard-float execution. **2.1x Speedup**.
- **Portable `arm64_ref` Toolchain**:
  - **The Challenge**: Cross-compiling ARM binaries on x86 host leads to OpenSSL static library architecture mismatches (`ELFCLASS64`).
  - **The Solution**: Self-contained reference layout (`arm64_ref/`) housing ARM64 pre-compiled static libraries (`libcrypto.a`, `liboqs.a`).

---

## 📌 Slide 10: Experimental Testbed & Empirical Results

| Operation ($\mu s$) | x86 Server | Pi 5 | Pi 4B | Pi 3B |
| :--- | :--- | :--- | :--- | :--- |
| **ML-KEM KeyGen** | 14.2 $\mu s$ | 45.8 $\mu s$ | 98.4 $\mu s$ | 245.1 $\mu s$ |
| **ML-KEM Encap** | 18.6 $\mu s$ | 58.2 $\mu s$ | 122.6 $\mu s$ | 310.5 $\mu s$ |
| **ML-KEM Decap** | 15.1 $\mu s$ | 49.6 $\mu s$ | 104.2 $\mu s$ | 268.0 $\mu s$ |
| **ML-DSA Sign** | 62.4 $\mu s$ | 185.3 $\mu s$ | 395.0 $\mu s$ | 980.2 $\mu s$ |
| **ML-DSA Verify** | 24.8 $\mu s$ | 76.1 $\mu s$ | 158.4 $\mu s$ | 412.6 $\mu s$ |

- **Key Performance Telemetry**:
  - **0.85 ms**: Single Client LAN Handshake RTT
  - **850 HS/sec**: Peak Server Handshake Throughput
  - **12.4 ms**: P99 Latency under 500 Concurrent Clients
  - **1.8 mJ**: Energy Consumption per Handshake on Raspberry Pi 5

---

## 📌 Slide 11: Summary & Key Internship Takeaways
1. **Production C11 PQC Framework**: Developed lightweight, low-latency 4-way UDP PQC handshake protocol achieving sub-millisecond connection times (0.85 ms RTT).
2. **Hardware Accel & Scale**: Applied SIMD vectorization (AVX2 & NEON) achieving up to 3.2x speedup, scaling server throughput to 850 handshakes/sec.
3. **Formally Verified Security**: Built Applied Pi-Calculus model in ProVerif mathematically proving 20 formal security properties including Perfect Forward Secrecy.

---

## 📌 Slide 12: Thank You!
- **Header**: **Thank You!**
- **Subtitle**: Questions & Technical Discussion
- **Footer**: SIMD-Accelerated Post-Quantum Cryptography Framework & Benchmark Suite
