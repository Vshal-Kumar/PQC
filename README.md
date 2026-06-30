# SIMD-Accelerated Post-Quantum Cryptography (PQC) Framework & Benchmark Suite

[![Language: C11](https://img.shields.io/badge/Language-C11-blue.svg)](https://en.wikipedia.org/wiki/C11_(C_standard_revision))
[![Platforms: x86__64 | ARM64 | ARM32](https://img.shields.io/badge/Platforms-x86__64%20%7C%20ARM64%20%7C%20ARM32-green.svg)](https://arm.com)
[![PQC KEM: ML-KEM-768](https://img.shields.io/badge/KEM-ML--KEM--768%20%28FIPS%20203%20Draft%29-orange.svg)](https://csrc.nist.gov/)
[![PQC Signature: ML-DSA-65](https://img.shields.io/badge/Signature-ML--DSA--65%20%28FIPS%20204%20Draft%29-red.svg)](https://csrc.nist.gov/)
[![AEAD: ChaCha20-Poly1305](https://img.shields.io/badge/AEAD-ChaCha20--Poly1305-brightgreen.svg)](https://datatracker.ietf.org/doc/html/rfc7539)

A high-performance, production-grade C framework implementing quantum-resistant secure messaging, multi-client server relay infrastructure, and high-concurrency benchmarking. Designed to evaluate SIMD vector optimizations across multiple hardware targets (**AVX2/BMI2** on x86_64 servers, **NEON** on ARM64 / AArch64 devices like Raspberry Pi 4/5, and **NEON/VFPv4** on ARM32 / ARMv7-A embedded platforms).

---

## 📋 Table of Contents
- [Overview & Multi-Platform Architecture](#overview--multi-platform-architecture)
- [Cryptographic Specifications](#cryptographic-specifications)
- [Explanation of `arm64_ref` Directory](#explanation-of-arm64_ref-directory)
- [Repository Component Map](#repository-component-map)
- [Prerequisites & Cross-Toolchains](#prerequisites--cross-toolchains)
- [Comprehensive Multi-Platform Build Guide](#comprehensive-multi-platform-build-guide)
  - [1. Step-by-Step Dependency Build: static liboqs](#1-step-by-step-dependency-build-static-liboqs)
  - [2. Building for x86_64 (Host / Server Platform)](#2-building-for-x86_64-host--server-platform)
  - [3. Building for ARM64 / AArch64 (Raspberry Pi 64-bit Platform)](#3-building-for-arm64--aarch64-raspberry-pi-64-bit-platform)
  - [4. Building for ARM32 / ARMv7-A (Raspberry Pi 32-bit Embedded Platform)](#4-building-for-arm32--armv7-a-raspberry-pi-32-bit-embedded-platform)
- [Execution & Operational Deployment](#execution--operational-deployment)
  - [Multi-Client Server Infrastructure](#multi-client-server-infrastructure)
  - [Interactive Client Execution](#interactive-client-execution)
  - [Running Microbenchmarks & Stress Tests](#running-microbenchmarks--stress-tests)
- [Performance & Metric Telemetry](#performance--metric-telemetry)
- [License](#license)

---

## 🎯 Overview & Multi-Platform Architecture

As quantum computing advances, classical asymmetric algorithms like RSA and ECC are vulnerable to Shor's algorithm. This project provides a full-stack post-quantum cryptographic (PQC) networking implementation, achieving **sub-millisecond handshakes** and high-throughput authenticated encryption over real-world network interfaces.

### 💻 Target Hardware Compatibility Matrix

| Target Architecture | Platform Examples | Hardware Acceleration | Compiler / Toolchain | Binary Targets |
| :--- | :--- | :--- | :--- | :--- |
| **x86_64** | Intel Core / AMD Ryzen Servers | AVX2, BMI2, POPCNT | `gcc` | `server`, `server_rx`, `server_tx`, `client`, `bench_pqc`, `bench_nclient` |
| **ARM64 (AArch64)** | Raspberry Pi 4/5 (64-bit OS), Jetson | NEON (128-bit SIMD) | `aarch64-linux-gnu-gcc` / `gcc` | `client_arm`, `bench_pqc_arm`, `bench_nclient_arm` |
| **ARM32 (ARMv7-A)** | Raspberry Pi 3/4 (32-bit OS), BeagleBone | NEON + VFPv4 (armhf) | `arm-linux-gnueabihf-gcc` / `gcc` | `client_arm32`, `bench_pqc_arm32`, `bench_nclient_arm32` |

```
                      +------------------------------------------+
                      |          Interactive Client              |
                      |   (x86_64  |  ARM64  |  ARM32 Pi)        |
                      +--------------------+---------------------+
                                           |
                                UDP (PQC 4-Way Handshake)
                                           |
                                           v
                      +------------------------------------------+
                      |          Multi-Client Server             |
                      |               (server.c)                 |
                      +------+----------------------------+------+
                             |                            |
             Shared Memory / IPC                 Shared Memory / IPC
                             |                            |
                             v                            v
                      +--------------+            +--------------+
                      |  Server RX   |            |  Server TX   |
                      | (server_rx.c)|            | (server_tx.c)|
                      +--------------+            +--------------+
```

---

## 🔒 Cryptographic Specifications

| Component | Standard / Algorithm | Security Level | Function |
| :--- | :--- | :--- | :--- |
| **Key Encapsulation (KEM)** | **ML-KEM-768** (Kyber768) | NIST Category 3 (~AES-192 equivalent) | Ephemeral Session Key Exchange |
| **Digital Signature (DSA)** | **ML-DSA-65** (Dilithium3) | NIST Category 3 (~AES-192 equivalent) | Mutual Authentication & Transcript Signing |
| **Symmetric Encryption** | **ChaCha20-Poly1305** | 256-bit Key / 96-bit Nonce / 128-bit Tag | Authenticated Encryption with Associated Data (AEAD) |
| **Hash & KDF** | **SHAKE256** | Extendable-Output Function (XOF) | Transcript Binding and Session Key Derivation |

---

## ❓ Explanation of `arm64_ref` Directory & OpenSSL Role

### 1. What is `arm64_ref` and why is it included?
When cross-compiling C software for an ARM64 target (such as a Raspberry Pi 4/5 running 64-bit Linux) from an **x86_64 host PC**, the cross-compiler (`aarch64-linux-gnu-gcc`) converts C source code into ARM64 machine code. However, the toolchain on your PC does **not** contain ARM64 target libraries.

`arm64_ref` is a **standalone target reference directory / symlink** containing ARM64 pre-compiled static libraries (`libssl.a`, `libcrypto.a`, `liboqs.a`) and matching header files. It allows you to cross-compile ARM64 executables on an x86_64 host immediately without needing root privileges or installing multi-arch packages (`libssl-dev:arm64`) into PC system folders.

### 2. Why is OpenSSL (`openssl_arm64`) inside `arm64_ref`?
This project uses OpenSSL's `libcrypto` library for **ChaCha20-Poly1305 AEAD symmetric encryption** and **SHAKE256 transcript hashing** (defined in `crypto/aead.c` and `client.c`).

- The standard OpenSSL installed on your Linux host PC (`/usr/lib/x86_64-linux-gnu/libcrypto.a`) is compiled strictly for **x86_64** architectures.
- If you attempt to link an ARM64 binary against host x86_64 libraries, the compiler throws architecture mismatch errors (e.g., `incompatible ELF class: ELFCLASS64`).
- `arm64_ref/openssl_arm64/` contains **ARM64-compiled OpenSSL headers and static libraries** (`libcrypto.a` and `libssl.a`).

- **Makefile Integration**: Referenced in `Makefile.arm64` as follows:
  ```makefile
  CC          = aarch64-linux-gnu-gcc
  LIBOQS      = arm64_ref/liboqs/build_aarch64_simd
  OPENSSL_ARM = arm64_ref/openssl_arm64/usr/lib/aarch64-linux-gnu
  INC         = -I. -I$(LIBOQS)/include -Iarm64_ref/openssl_arm64/usr/include
  LIBS        = $(LIBOQS)/lib/liboqs.a $(OPENSSL_ARM)/libssl.a $(OPENSSL_ARM)/libcrypto.a -lpthread -lm
  ```

---

## 📁 Repository Component Map

```
├── crypto/
│   ├── aead.c               # ChaCha20-Poly1305 AEAD wrapper (OpenSSL libcrypto)
│   └── aead.h               # AEAD interface definitions
├── wrappers/
│   ├── kem_wrapper.c        # ML-KEM-768 API wrapper (liboqs abstraction)
│   ├── kem_wrapper.h        # KEM function declarations and constants
│   ├── dsa_wrapper.c        # ML-DSA-65 API wrapper (liboqs abstraction)
│   └── dsa_wrapper.h        # DSA function declarations and constants
├── bench_nclient.c          # High-concurrency multi-client handshake benchmark
├── bench_pqc.c              # Standalone cryptographic primitive microbenchmark
├── client.c                 # Interactive PQC secure chat client
├── server.c                 # Multi-client UDP PQC relay server core
├── server_rx.c              # Terminal interface: Server Receiver display
├── server_tx.c              # Terminal interface: Server Sender transmitter
├── client_registry.c/.h     # Thread-safe client session state management
├── transport.c/.h           # Custom UDP reliable packet transport & metrics protocol
├── ipc.h                    # Inter-process communications structures
├── Makefile                 # x86_64 build configuration file
├── Makefile.arm64           # ARM64 / AArch64 native and cross-compilation Makefile
├── Makefile.arm32           # ARM32 / ARMv7-A native and cross-compilation Makefile
├── build_code.txt           # Reference CMake build script snippet for x86_64 liboqs
├── run.sh                   # Convenience script launching server terminal splits
├── LICENSE                  # MIT License terms
└── README.md                # Project documentation
```

---

## 🛠️ Prerequisites & Cross-Toolchains

### Linux Host Dependencies
- **OS**: Ubuntu 22.04 LTS+, Debian 12+, or Raspberry Pi OS (32-bit / 64-bit).
- **Compiler**: `gcc` 11.0+ with C11 standard support.
- **Build Tools**: `cmake` (v3.20+), `make`, `git`, `pkg-config`.
- **System Libraries**: `libssl-dev` (OpenSSL 3.0+), `pthread`, `libm`.

### Installing Cross-Compilation Toolchains (on x86_64 host)
```bash
sudo apt-get update
# Toolchain for ARM64 (AArch64)
sudo apt-get install -y gcc-aarch64-linux-gnu g++-aarch64-linux-gnu

# Toolchain for ARM32 (ARMv7-A / armhf)
sudo apt-get install -y gcc-arm-linux-gnueabihf g++-arm-linux-gnueabihf
```

---

## 🏗️ Comprehensive Multi-Platform Build Guide

> [!WARNING]
> **Resource Management on Embedded ARM Platforms (Raspberry Pi)**:
> When building `liboqs` or compiling executables natively on embedded Raspberry Pi hardware, **DO NOT use high parallel job counts like `-j$(nproc)`**. High parallel C++ template and lattice-code compilation consumes excessive RAM per core, leading to Out-Of-Memory (OOM) kernel kills, swapping lag, or system freezes.
> - **On Raspberry Pi (ARM64 / ARM32)**: Use `-j2` or `-j1`.
> - **On High-Performance x86_64 Servers**: `-j$(nproc)` is recommended.

---

### 1. Step-by-Step Dependency Build: static `liboqs`

Before compiling project executables, static `liboqs` must be compiled for your specific target platform.

#### A. Building for x86_64 (AVX2 & BMI2 Hardware Acceleration)
*Matching `build_code.txt` reference configuration:*
```bash
git clone --depth 1 https://github.com/open-quantum-safe/liboqs.git liboqs

cmake -S liboqs -B liboqs/build \
      -DCMAKE_BUILD_TYPE=Release \
      -DOQS_BUILD_ONLY_LIB=ON \
      -DOQS_ENABLE_KEM_ml_kem_768=ON \
      -DOQS_ENABLE_SIG_ml_dsa_65=ON \
      -DOQS_USE_AVX2_INSTRUCTIONS=ON \
      -DOQS_USE_BMI2_INSTRUCTIONS=ON \
      -DOQS_USE_POPCNT_INSTRUCTIONS=ON \
      -DCMAKE_C_FLAGS="-march=native -O3" \
      -DBUILD_SHARED_LIBS=OFF

cmake --build liboqs/build -j$(nproc)
```

---

#### B. Building for ARM64 / AArch64 (NEON Vector Acceleration)

##### Native Build on Raspberry Pi (64-bit OS):
```bash
cmake -S liboqs -B liboqs/build \
      -DCMAKE_BUILD_TYPE=Release \
      -DOQS_BUILD_ONLY_LIB=ON \
      -DOQS_ENABLE_KEM_ml_kem_768=ON \
      -DOQS_ENABLE_SIG_ml_dsa_65=ON \
      -DCMAKE_C_FLAGS="-O3 -march=armv8-a+simd" \
      -DBUILD_SHARED_LIBS=OFF

# Note: Using -j2 to prevent memory exhaustion on Raspberry Pi
cmake --build liboqs/build -j2
```

---

#### C. Building for ARM32 / ARMv7-A (NEON / VFPv4 Acceleration)

##### Native or Cross-Compilation Build for ARM32:
```bash
cmake -S liboqs -B liboqs/build_arm32 \
      -DCMAKE_BUILD_TYPE=Release \
      -DOQS_BUILD_ONLY_LIB=ON \
      -DOQS_ENABLE_KEM_ml_kem_768=ON \
      -DOQS_ENABLE_SIG_ml_dsa_65=ON \
      -DCMAKE_C_FLAGS="-O3 -march=armv7-a -mfpu=neon-vfpv4 -mfloat-abi=hard" \
      -DBUILD_SHARED_LIBS=OFF

# Note: Using -j2 to prevent RAM exhaustion on ARM32 hardware
cmake --build liboqs/build_arm32 -j2
```

---

### 2. Building for x86_64 (Host / Server Platform)

Compiles high-throughput server and client binaries optimized with `-O3 -march=native -flto=auto`:

```bash
make all -j$(nproc)
```

**Output Binaries**: `server`, `server_rx`, `server_tx`, `client`, `bench_pqc`, `bench_nclient`.

---

### 3. Building for ARM64 / AArch64 (Raspberry Pi 64-bit Platform)

#### Native Build on Raspberry Pi (64-bit OS):
```bash
make -f Makefile.arm64 all -j2
```

#### Cross-Compilation on x86_64 Host:
Uses `aarch64-linux-gnu-gcc` and links via `arm64_ref`:
```bash
make -f Makefile.arm64 all -j$(nproc)
```

**Output Binaries**: `client_arm`, `bench_pqc_arm`, `bench_nclient_arm`.

---

### 4. Building for ARM32 / ARMv7-A (Raspberry Pi 32-bit Embedded Platform)

#### Native Build on Raspberry Pi (32-bit OS):
```bash
make -f Makefile.arm32 all -j2
```

#### Cross-Compilation on x86_64 Host:
Uses `arm-linux-gnueabihf-gcc` with hard-float NEON vector optimizations (`-march=armv7-a -mfpu=neon-vfpv4 -mfloat-abi=hard`):
```bash
make -f Makefile.arm32 all -j$(nproc)
```

**Output Binaries**: `client_arm32`, `bench_pqc_arm32`, `bench_nclient_arm32`.

---

## 🚀 Execution & Operational Deployment

### Multi-Client Server Infrastructure

The multi-client server deployment operates with three distinct terminal interfaces connected via local POSIX IPC to decouple high-speed UDP network routing from operator logging.

#### Option 1: Quick Launch (Automated Terminal Splitter)
```bash
chmod +x run.sh
./run.sh
```

#### Option 2: Manual Terminal Invocations (Three Separate Terminals)
1. **Terminal 1 (Main Server Relay Engine)**:
   ```bash
   ./server
   ```
2. **Terminal 2 (Receiver Terminal Display)**:
   ```bash
   ./server_rx
   ```
3. **Terminal 3 (Sender Console Interface)**:
   ```bash
   ./server_tx
   ```

---

### Interactive Client Execution

Execute the PQC client on your target client platform and specify the server IP address:

#executing server
PQC_DISABLE_RATE_LIMIT=1 ./server


```bash
# Executing on x86_64:
./client <SERVER_IP>

# Executing on ARM64 (Raspberry Pi 64-bit OS):
./client_arm <SERVER_IP>

# Executing on ARM32 (Raspberry Pi 32-bit OS):
./client_arm32 <SERVER_IP>
```

---

### Running Microbenchmarks & Stress Tests

#### 1. Standalone Cryptographic Microbenchmark
Measures exact microsecond latency for ML-KEM-768 and ML-DSA-65 operations:
```bash
./bench_pqc        # x86_64 Host
./bench_pqc_arm    # ARM64 Raspberry Pi
./bench_pqc_arm32  # ARM32 Raspberry Pi
```

#### 2. High-Concurrency Handshake Benchmark
Simulates N concurrent client sessions negotiating PQC handshakes simultaneously:
```bash
./bench_nclient        # x86_64 Host
./bench_nclient_arm    # ARM64 Raspberry Pi
./bench_nclient_arm32  # ARM32 Raspberry Pi
```

---

## 📊 Performance & Metric Telemetry

The framework incorporates high-resolution hardware timers (`clock_gettime(CLOCK_MONOTONIC)`) providing telemetry across all phases:

| Metric Category | Telemetry Tracked | Unit |
| :--- | :--- | :--- |
| **Handshake Micro-timings** | KEM keygen, DSA keygen, KEM encap/decap, DSA sign/verify | Milliseconds ($ms$) |
| **Total Handshake Latency** | Full 4-way round-trip time from `CLIENT_HELLO` to `ACTIVE` | Milliseconds ($ms$) |
| **Payload Overheads** | Symmetric encryption and decryption times | Microseconds ($\mu s$) |
| **Network Metrics** | Inter-arrival time, jitter, packet loss, throughput ($B/s$) | $ms$ / Bytes |

---

## 📄 License

This project is licensed under the **MIT License**. See the [LICENSE](LICENSE) file for full details.
