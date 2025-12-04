# Aegis Sky Core: The Aura Perception Engine

![Build Status](https://img.shields.io/badge/build-passing-brightgreen)
![Standard](https://img.shields.io/badge/std-c%2B%2B20-blue)
![Platform](https://img.shields.io/badge/platform-NVIDIA%20Jetson%20Orin-green)
![CUDA](https://img.shields.io/badge/CUDA-12.x-76B900)
![License](https://img.shields.io/badge/license-Proprietary-red)

> **"In the era of asymmetric warfare, latency is the only metric that matters."**

**Aegis Sky Core** is the embedded software stack powering the **Aegis Sentry** autonomous defense pod. It is a hyper-optimized, real-time C++20/CUDA system designed to detect, track, and engage Class 1 & 2 UAS threats (drones) with superhuman speed and precision.

Unlike legacy air defense systems, Aegis Sky utilizes **Early Sensor Fusion** and the proprietary **xInfer** engine to achieve an end-to-end loop time of **< 50 milliseconds**.

---

## 🚀 Key Technical Features

### ⚡ Zero-Copy Architecture
We do not move data; we move pointers.
*   **Direct DMA Access:** Sensor drivers (GigE Vision, 4D Radar) write directly to pinned GPU memory.
*   **Ring Buffers:** A lock-free shared memory transport moves data between services without CPU intervention.

### 🧠 The "Aura" Fusion Engine
*   **Raw Data Fusion:** We fuse Radar Point Clouds and RGB pixels *before* the Neural Network input layer.
*   **Custom Kernels:** Proprietary CUDA kernels project 3D Doppler data onto 2D visual feature maps in real-time.
*   **xInfer Integration:** Powered by our internal `xInfer` runtime, utilizing fused TensorRT engines for max throughput on Jetson Orin.

### 🧩 Massive Modularity
*   **Plugin System:** Drivers and Algorithms are compiled as dynamic shared libraries (`.so`).
*   **Hot-Swappable:** Switch from a simulation driver to a hardware driver via config, without recompiling the core.
*   **Strict Contracts:** Public Interfaces (`include/`) are strictly separated from Private Implementation (`src/`).

---

## 🏗️ System Architecture

The system follows a strict **Service-Oriented Architecture (SOA)** tailored for embedded Linux.

```mermaid
graph TD
    HW[Hardware Layer (Sensors)] --> HAL[HAL Plugins (Drivers)]
    HAL -->|Zero-Copy DMA| BUS[Middleware Bus (Shared Mem)]

    BUS --> FUSION[Fusion Service]
    FUSION -->|Fused Tensor| PERC[Perception Service (xInfer)]
    PERC -->|Detections| TRACK[Tracking Service (Kalman)]
    TRACK -->|Tracks| FIRE[Fire Control Service]

    FIRE -->|Commands| EFF[Effector HAL]
```

---

## 📂 Project Structure

We enforce a strict separation between **Contracts** (Public) and **Logic** (Private).

```text
aegis-sky-core/
├── cmake/                  # Build modules (FindxInfer.cmake, Toolchains)
├── configs/                # Runtime YAML configurations
│
├── include/aegis/          # [PUBLIC] The Contracts (Headers Only)
│   ├── core/               # Primitive types (UUID, Timestamp)
│   ├── hal/                # Abstract Hardware Interfaces (ICamera.h, IRadar.h)
│   └── messages/           # Data Transfer Objects (PointClouds, Tracks)
│
├── src/                    # [PRIVATE] The Implementation (Source Code)
│   ├── platform/           # OS Wrappers (Memory, Threading, Time)
│   ├── middleware/         # IPC & Shared Memory Transport
│   ├── drivers/            # Hardware Plugins (GigE, CAN-FD, Serial)
│   └── services/           # The Core Logic (Fusion, Perception, Tracking)
│
├── tests/                  # GoogleTest Suites
└── tools/                  # Calibration & Replay Utilities
```

---

## 🛠️ Dependencies & Prerequisites

This project relies on the **Ignition AI Ecosystem**. Ensure these are installed in your system path or provided via CMake.

1.  **Hardware:** NVIDIA Jetson AGX Orin (JetPack 6.0+) or x86_64 w/ RTX GPU (Sim Mode).
2.  **Core Libraries (Proprietary):**
    *   **[xInfer](https://github.com/ignition-ai/xinfer)** (Runtime v2.1+) - *Required*
    *   **[xTorch](https://github.com/ignition-ai/xtorch)** (Optional, for online learning)
3.  **Third Party:**
    *   CUDA Toolkit 12.2+
    *   TensorRT 8.6+
    *   OpenCV 4.8 (with CUDA support)
    *   Protobuf & gRPC

---

## 🔨 Build Instructions

We use **CMake** and **Ninja** for high-speed parallel builds.

```bash
# 1. Clone the repository
git clone https://github.com/ignition-ai/aegis-sky-core.git
cd aegis-sky-core

# 2. Configure the project (Release Mode)
cmake -B build -G Ninja \
    -DCMAKE_BUILD_TYPE=Release \
    -DAEGIS_PLATFORM=JETSON_ORIN

# 3. Compile
cmake --build build --parallel $(nproc)

# 4. Install (Optional)
sudo cmake --install build
```

---

## 🖥️ Running the Engine

The system is controlled via the `aegis-supervisor` daemon.

**1. Simulation Mode (No Hardware):**
```bash
./build/bin/aegis_core --config configs/simulation/desert_scenario.yaml
```

**2. Deployment Mode:**
```bash
# Requires root for SCHED_FIFO priority and GPIO access
sudo ./build/bin/aegis_core --config configs/deployment/live_fire_v1.yaml
```

---

## 📜 License & Confidentiality

**Copyright © 2025 Aegis Sky, Inc. All Rights Reserved.**

*NOTICE: This software contains proprietary trade secrets and is protected by international copyright laws. Unauthorized reproduction, distribution, or reverse engineering of the `src/` directory is strictly prohibited.*

*This software may be subject to export control regulations (ITAR/EAR).*

---
**Maintained by the Aegis Sky Skunkworks Team.**
