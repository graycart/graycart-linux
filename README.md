# Graycart Linux

> **Status:** placeholder / research stage

Graycart Linux is the future gaming-focused Linux distribution in the Graycart family.

The current direction is to explore an **Omarchy-inspired, Arch-based foundation** with a much stronger emphasis on gaming, emulation, controller-first use, sane recovery, and a cohesive Graycart experience from boot to play.

This repository is intentionally small today. It exists so the project has a public home while we research the base system and decide what Graycart Linux should become.

## Vision

Graycart Linux should feel like a purpose-built gaming machine, not a generic Linux install with a launcher dropped on top.

The goals are:

- **Boot to play quickly.** Minimize ceremony between power-on and a running game.
- **Controller-first without being controller-only.** The desktop should remain excellent with keyboard and mouse.
- **Great native gaming.** Steam, Proton, Gamescope, MangoHud, and modern Linux graphics should feel like first-class citizens.
- **First-class emulation.** Graycart emulators and other high-quality emulators should integrate cleanly without turning the OS into a closed appliance.
- **Opinionated defaults, escape hatches everywhere.** Make the common path excellent while keeping the underlying Linux system accessible.
- **Reliable updates and recovery.** A gaming OS should not become a weekend repair project because one package upgrade went sideways.
- **Hardware-aware.** Desktop GPUs, handhelds, controllers, VRR/HDR, Bluetooth, audio, suspend/resume, and game-mode behavior matter.
- **A cohesive identity.** Graycart should look and feel intentional across the boot experience, desktop, launcher, emulator tooling, and diagnostics.

## Likely foundation

We are currently leaning toward an approach inspired by [Omarchy](https://omarchy.org/): a curated, opinionated Arch Linux experience rather than inventing a distribution stack from scratch.

That direction is **not final**. Before implementation we need to answer questions around update strategy, image creation, recovery, hardware enablement, desktop/session choice, packaging, and how much of the base should remain upstream-compatible.

Graycart Linux should borrow good ideas where they make sense without simply becoming "Omarchy with different wallpaper."

## Early roadmap

### Phase 0 — Research

- Evaluate Omarchy and alternative Arch-based foundations.
- Define supported hardware classes.
- Decide desktop/session architecture.
- Define update, rollback, and recovery strategy.
- Establish branding and UX principles.

### Phase 1 — Bootstrap

- Reproducible base installation/image.
- Graycart package repository or overlay strategy.
- First-boot configuration.
- Hardware detection and sane defaults.

### Phase 2 — Gaming platform

- Steam/Proton integration.
- Gamescope and performance-session policy.
- Controller and Bluetooth experience.
- Audio-device behavior suitable for TVs, monitors, headsets, and desktop setups.
- GPU/VRR/HDR validation.

### Phase 3 — Graycart experience

- Graycart GB integration.
- Future Graycart GBA integration.
- Unified game-library concepts where useful.
- Save-data, screenshot, controller-profile, and diagnostic conventions.
- Cohesive visual identity and system UX.

### Phase 4 — Recovery and release

- Rollback/recovery flow.
- Update channels.
- Hardware acceptance matrix.
- Installable public images.
- Documentation and support tooling.

## Principles

1. **Gaming performance comes before desktop novelty.**
2. **Do not fork upstream components without a compelling reason.**
3. **Configuration should be reproducible and reviewable.**
4. **The system should remain understandable by a competent Linux user.**
5. **No hidden magic that makes recovery harder.**
6. **Do not sacrifice the desktop just to imitate a console.**
7. **Measure performance claims instead of assuming them.**
8. **Controller focus, audio routing, suspend/resume, and display behavior are product features.**

## Graycart family

- [`graycart`](https://github.com/graycart/graycart) — project umbrella
- [`graycart-gb`](https://github.com/graycart/graycart-gb) — DMG / Game Boy Color emulator
- [`graycart-gba`](https://github.com/graycart/graycart-gba) — future Game Boy Advance emulator
- **`graycart-linux`** — future gaming-focused Linux distribution

## Contributing

There is nothing substantial to build yet. Early contributions should focus on research, experiments, hardware findings, and architecture proposals rather than large implementation PRs.

When development begins, this repository will gain a formal contribution guide, build instructions, and project-specific agent guidance.

## License

The eventual source/configuration in this repository will be licensed explicitly as implementation begins. Individual upstream components will retain their own licenses.
