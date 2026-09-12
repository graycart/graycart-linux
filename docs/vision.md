# Graycart Linux — Vision

Graycart Linux should become a **gaming operating system that happens to be Linux**, not a Linux desktop with a folder full of emulators.

The experience should be coherent from boot through game launch, play, suspend, updates, diagnostics, and recovery.

## Product direction

The likely starting point is an **Omarchy-inspired Arch Linux base**: curated, opinionated, fast-moving, and transparent enough that advanced users can still understand the system underneath.

The project should learn from Omarchy rather than merely reskin it. Graycart's priorities are different: gaming latency, controller behavior, display/audio switching, emulator integration, Steam/Proton, handheld/TV use, recovery, and reproducibility are central product concerns.

## Experience goals

### Boot to play

A user should be able to power on, pick a game, and start playing with minimal ceremony. Startup services and desktop behavior should be measured against that goal.

### Controller-first, desktop-capable

A controller should be sufficient for the common gaming path, but Graycart Linux should still be a real desktop environment with excellent keyboard and mouse behavior.

### Native games and emulation are peers

Steam/Proton and emulation should live in the same product rather than feeling like unrelated subsystems. Graycart's own emulators should integrate naturally, while users remain free to install and use other emulators.

### Hardware behavior is part of UX

The distro should care about things Linux gaming systems often leave to the user:

- correct default audio endpoint
- hot-plugged controllers
- Bluetooth reconnects
- TV/monitor switching
- VRR and HDR
- suspend/resume
- GPU driver state
- game performance mode
- handheld power behavior

### Recovery should be boring

Updates must not turn a gaming machine into a repair project. Rollback, logs, and recovery should be designed before broad release.

## Non-goals

Graycart Linux should not become:

- a locked console appliance
- an arbitrary collection of themed dotfiles
- a replacement for upstream projects that already work well
- an excuse to fork the kernel, Mesa, Proton, or desktop components without evidence
- a giant shell script that mutates an Arch installation into an undocumented state

## Technical questions to resolve

Before choosing an implementation architecture, research:

1. Omarchy's actual ownership/update model and how safely it can serve as a base.
2. Image-based vs package-based installation/update strategies.
3. Filesystem snapshot and rollback options.
4. Desktop/session choice and Gamescope integration.
5. Wayland/Hyprland implications for TV, handheld, desktop, HDR, and VRR use.
6. Audio endpoint discovery and policy across PipeWire/WirePlumber.
7. Controller/input architecture and Bluetooth recovery.
8. NVIDIA/AMD/Intel graphics support boundaries.
9. Secure Boot and installation expectations.
10. How Graycart applications integrate without becoming distro-bound.

## Aesthetic direction

The Graycart family already has a retro/pixel identity, but the OS should not look like a novelty 8-bit theme. Use that identity selectively: branding, boot art, icons, diagnostic surfaces, and small details. The daily desktop should remain crisp, dense, modern, and readable.

Think **retro computing DNA inside a serious modern gaming workstation**.
