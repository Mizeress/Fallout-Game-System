# Fallout-Game-System: Embedded Handheld
**Documentation for a purpose-built Linux handheld for the Fallout Community Edition (CE) engine.**

## 🛠 Tech Stack
- **Yocto / Buildroot Custom Distribution**
  - **Goal:** Minimal footprint OS (under 100MB) optimized for sub-10 second boot times.
  - **Pros:** Total control over kernel configuration and package overhead.
  - **Key Features:** Read-only rootfs to prevent SD card corruption; custom init scripts for instant-app-launch.

- **Hardware & Low-Level I/O**
  - **Compute:** Raspberry Pi 4.
  - **Inputs:** Native hardware integration using `gpio-keys` and `adc-joystick` drivers via **Device Tree Overlays**. 
  - **ADC:** External SPI-based ADC (e.g., MCP3008) to bridge analog stick data to the IIO (Industrial I/O) subsystem.
  - **Display:** MIPI DSI interface utilizing the **VC4 DRM/KMS** driver for GPU-accelerated output.

- **Software & Input Mapping**
  - **Engine:** [Fallout-CE](https://github.com/alexbatalov/fallout1-ce) (Community Edition) cross-compiled for AArch64.
  - **Mapping:** Headless input remapping via `evsieve` or SDL2 GameController API to translate gamepad events into mouse/keyboard sequences required for Fallout’s legacy UI.

## 🗺️ Development Roadmap

### Phase 1: Silicon & Kernel Proof-of-Concept
*Goal: Establish a hardware-to-kernel communication bridge.*
- [ ] **SPI & IIO Integration:** Connect ADC via SPI and verify raw voltage readings.
- [ ] **Device Tree Development:** Write custom `.dts` overlays to bind ADC channels to the `adc-joystick` driver.
- [ ] **Input Verification:** Achieve native `/dev/input/js0` recognition without userspace polling scripts.
- [ ] **Digital Logic:** Implement `gpio-keys` with kernel-level debouncing for physical buttons.

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
- [ ] **Library Expansion:** Cross-compile additional engine recreations (e.g., OpenMW, DevilutionX) to test platform extensibility.

