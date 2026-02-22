# Fallout-Game-System: Embedded Handheld
**Documentation for a purpose-built Linux handheld for the Fallout Community Edition (CE) engine.**

## Project Status 🏗️ Phase 1 - Research & PoC

## 🛠 Tech Stack
- **Yocto / Buildroot Custom Distribution**
  - **Goal:** Minimal footprint OS (under 100MB) optimized for sub-10 second boot times.
  - **Pros:** Total control over kernel configuration and package overhead.
  - **Key Features:** Read-only rootfs to prevent SD card corruption; custom init scripts for instant-app-launch.

- **Hardware & Low-Level I/O**
  - **Compute:** Raspberry Pi 4.
  - **Inputs:** Native hardware integration using `gpio-keys` and `adc-joystick` drivers via **Device Tree Overlays**. 
  - **ADC:**
    - External SPI-based ADC (e.g., MCP3008) for polled sampling.
    - Alternatively I2C based ADC (ADS1015) for hardware-triggered interrupts. 
  - **Display:** MIPI DSI interface utilizing the **VC4 DRM/KMS** driver for GPU-accelerated output.

- **Software & Input Mapping**
  - **Engine:** [Fallout-CE](https://github.com/alexbatalov/fallout1-ce) (Community Edition) cross-compiled for AArch64.
  - **Mapping:** Headless input remapping via `evsieve` or SDL2 GameController API to translate gamepad events into mouse/keyboard sequences required for Fallout’s legacy UI.

- **Hybrid Architecture**:
  - Host Machine:
    - Core OS, Drivers, and Game Selection UI (multiple games not MVP), 
  - Docker Containers
    - Specific Games, game-specific input remapping, and any necessary emulators (e.g. DOSbox for Elder Scrolls Arena)
   
  - *Pros*
    *   **Modular Scalability:** Add new games (like *The Elder Scrolls: Arena*) by simply pulling a new image without needing to reflash or modify the core Buildroot filesystem.
    *   **Dependency Isolation:** Run games that require legacy libraries or specific versions of DOSBox/Python without "polluting" the slim host OS or creating version conflicts.
    *   **Atomic Updates:** Update game logic, input remapping scripts, or emulator versions independently of the underlying hardware drivers.
    *   **Development Speed:** Build and test game "sections" on a standard Linux PC (using cross-platform base images) and deploy to the Pi 4 instantly.
    *   **Enhanced Stability:** A crash or memory leak within a game container is less likely to freeze the Main UI or the host system.

  - *Cons*
    *   **Storage Overhead:** Container images (even Alpine-based) consume more SD card space than native binaries due to bundled libraries.
    *   **GPU Complexity:** Hardware acceleration requires mounting `/dev/dri` and ensuring the container's Mesa drivers match the host's kernel version.
    *   **Input Management:** Requires explicit passthrough of `/dev/input` devices and potentially `/dev/uinput` for virtual remapping logic.
    *   **Slightly Slower Cold Boots:** Launching a containerized game takes a few seconds longer than launching a native binary due to the container startup sequence.

## 🗺️ Development Roadmap

### Phase 1: Silicon & Kernel Proof-of-Concept
*Goal: Establish a hardware-to-kernel communication bridge.*
- [ ] **SPI & IIO Integration:** Connect ADC via SPI and verify raw voltage readings.
- [ ] **Device Tree Development:** Write custom `.dts` overlays to bind ADC channels to the `adc-joystick` driver.
- [ ] **Interrupt vs Polled sampling:** Polling via `adc-joystick` can sometimes feel jumpy, interrupt driven readings may be better. Use ADC with interrupt pin if interrupt route is used to avoid need for an external MCU. 
- [ ] **Input Verification:** Achieve native `/dev/input/js0` recognition without userspace polling scripts.
- [ ] **Digital Logic:** Implement `gpio-keys` with kernel-level debouncing for physical buttons.

### Phase 1.5: Containerized Graphics & Input Passthrough
*Goal: Validate the hybrid architecture by proving Docker-to-Hardware performance.*
- [ ] **Mesa/DRM Mapping:** Research and map host /dev/dri/card0 and /dev/dri/renderD128 into a test container to achieve GPU-accelerated rendering.
- [ ] **Library Alignment:** Ensure the containerized SDL2/Mesa version matches the host kernel’s VC4/V3D driver requirements to avoid software fallback.
- [ ] **Input Device Plumbing:** Verify /dev/input/ passthrough, ensuring evsieve or SDL2 can capture raw events from the host’s gpio-keys and adc-joystick.
- [ ] **Benchmark Validation:** Run glmark2-es2 or a simple SDL2 test app within a container to confirm 60FPS output with zero input lag.
- [ ] **OCI Runtime Optimization:** Evaluate Podman or raw runc as alternatives to the standard Docker daemon to minimize startup overhead and memory footprint.

### Phase 2: Engine & Graphics Optimization
*Goal: High-performance execution of the Fallout-CE engine.*
- [ ] **Toolchain Setup:** Configure a cross-compilation environment for `aarch64`.
- [ ] **Engine Port:** Compile Fallout-CE and link against SDL2 with DRM/KMS support.
- [ ] **Visual Pipeline:** Enable the `vc4-kms-v3d` driver for 60FPS output on the MIPI DSI display.
- [ ] **Input Translation:** Configure `evsieve` for analog-to-mouse mapping and button macro logic.

### Phase 3: Custom OS Synthesis (Buildroot)
*Goal: Strip the bloat and harden the system for handheld use.*
- [ ] **Minimal Filesystem:** Build a custom Linux distribution using Buildroot.
- [ ] **Fast-Boot Optimization:** Streamline `init` scripts to launch the game directly from the bootloader.
- [ ] **System Hardening:** Implement a **read-only rootfs** with OverlayFS for save-game persistence.
- [ ] **Power Management:** Integrate battery sensing and safe-shutdown signaling via GPIO.

### Phase 4: Industrial Design & Final Polish
*Goal: Transition from a breadboard to a handheld console.*
- [ ] **PCB Design:** Create a carrier board for the Pi 4, ADC, and button matrix.
- [ ] **UX/UI:** Implement a custom splash screen (using `fbi` or `psplash`) to hide boot logs.
- [ ] **Chassis:** 3D print an ergonomic enclosure housing the DSI screen and LiPo power system.

### Phase 5: Multi-Game Ecosystem & Custom UI (Future Considerations)
*Goal: Transition from a single-title appliance to a multi-game handheld platform.*
- [ ] **Unified Launcher:** Evaluate lightweight front-ends (e.g., EmulationStation, Pegasus, or a custom LVGL-based UI).
  - Potentially Pip Boy UI via LVGL
- [ ] **Dynamic Input Profiling:** Develop a middleware script to swap `evsieve` profiles based on the selected title.
- [ ] **DRM/KMS Multi-tenancy:** Research seamless transitions between the launcher context and the game engine context.
- [ ] Test adding a Fallout 2 container or Elder Scrolls Arena container to prove scalability




