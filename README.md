# Graycart Linux

> **Status:** placeholder / research stage

Graycart Linux is the future gaming-focused Linux **host** (and optional branded distribution) in the Graycart family.

**Research default ([01](./docs/01-scope-and-models.md)):** ship the **host app on stock distros first** (Meaning A), then optionally a **Model 4 immutable/atomic branded image** (Fedora bootc / Universal Blue–style; Nix flake acceptable alternate). Emphasize gaming, emulation, controller-first use, sane recovery, and a cohesive Graycart experience from boot to play. **Dave review of docs 01–03 is the gate before image work.**

This repository is intentionally small today. It exists so the project has a public home while we research the base system and decide what Graycart Linux should become.

## Research docs

Start here: **[`docs/README.md`](./docs/README.md)**

| Doc | Topic |
|-----|--------|
| [vision.md](./docs/vision.md) | Product vision |
| [01 — Scope & models](./docs/01-scope-and-models.md) | Host vs branded OS; competing models |
| [02 — Technical stack](./docs/02-technical-stack.md) | Boot → kernel → userspace → graphics → CI |
| [03 — Roadmap & risks](./docs/03-roadmap-and-risks.md) | Phases, legal, go/no-go |

Agent norms: [`AGENTS.md`](./AGENTS.md) · credits: [`ATTRIBUTION.md`](./ATTRIBUTION.md)

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

**Default path (research):** **Model 4 — immutable/atomic branded image** on an upstream atomic stack ([Fedora bootc](https://fedoramagazine.org/building-your-own-atomic-bootc-desktop/) / [Universal Blue](https://github.com/ublue-os/bazzite)–style; [Nix flake](https://nixos.wiki/wiki/Flakes) if the team already lives in Nix) — **after** a Graycart Linux **host app** runs on stock Fedora/Ubuntu/Arch.

Why: lowest effort/risk that still yields a coherent product skin; inherits upstream security and GPU stacks; keeps emulator accuracy in `*-core`; matches how niche gaming OSes (e.g. Bazzite) actually ship. See [01 §10](./docs/01-scope-and-models.md).

**Not the default:** from-scratch OS, LFS-as-product, or ChromeOS/Asahi-scale downstream kernels. An [Omarchy](https://omarchy.org/)-inspired Arch UX remains a useful **ideas source** for session polish — not the recommended image/update architecture.

Before implementation we need Dave acceptance of base (bootc/UB vs Nix vs host-only), update/rollback, hardware classes, session choice, branding (Linux Mark), and packaging boundaries.

## Early roadmap

Aligned with [`docs/03-roadmap-and-risks.md`](./docs/03-roadmap-and-risks.md):

### Phase 0 — Research (now)

- Confirm Model 4 (atomic image) vs host-app-only; choose bootc/UB vs Nix vs Debian+mkosi.
- Define supported hardware classes.
- Decide desktop/session architecture (Gamescope / Wayland).
- Define update, rollback, and recovery strategy.
- Establish branding and UX principles (incl. Linux Mark).
- Accept docs 01–03 before any public “download Graycart Linux” claim.

### Phase 1 — Bootstrap

- Reproducible base installation/image (only after research gate).
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

- Graycart GB integration (host over cores — no forked machine logic).
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
9. **Emulator accuracy stays in `*-core` crates** — this repo is host / OS shell only.

## Graycart family

- [`graycart`](https://github.com/graycart/graycart) — project umbrella
- [`graycart-gb`](https://github.com/graycart/graycart-gb) — DMG / Game Boy Color emulator
- [`graycart-gba`](https://github.com/graycart/graycart-gba) — future Game Boy Advance emulator
- **`graycart-linux`** — future gaming-focused Linux host / distribution

## Contributing

Early contributions should focus on research, experiments, hardware findings, and architecture proposals rather than large implementation PRs. Read [`AGENTS.md`](./AGENTS.md) and [`ATTRIBUTION.md`](./ATTRIBUTION.md) before editing docs or sources.

When development begins, this repository will gain a formal contribution guide and build instructions.

## License

The eventual source/configuration in this repository will be licensed explicitly as implementation begins. Individual upstream components will retain their own licenses.
