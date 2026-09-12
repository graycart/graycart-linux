# Graycart Linux

> **Status:** placeholder / research stage

Graycart Linux is the gaming-focused Linux system in the Graycart family.

**Product decision (Dave, 2026-09-12):** Graycart Linux is **based on [Omarchy](https://omarchy.org/)** (DHH) — an opinionated Arch Linux setup — as the technical foundation to derive from. Emphasize gaming, emulation, controller-first use, sane recovery, and a cohesive Graycart experience from boot to play. Sibling research will document the derivative plan in [`docs/04-omarchy-base.md`](./docs/04-omarchy-base.md) (pending) and revise 01–03 accordingly.

This repository is intentionally small today. It exists so the project has a public home while we map Omarchy → Graycart Linux and decide packaging, update, and host integration boundaries.

## Research docs

Start here: **[`docs/README.md`](./docs/README.md)**

| Doc | Topic |
|-----|--------|
| [vision.md](./docs/vision.md) | Product vision |
| [01 — Scope & models](./docs/01-scope-and-models.md) | Host vs branded OS; competing models *(being revised for Omarchy base)* |
| [02 — Technical stack](./docs/02-technical-stack.md) | Boot → kernel → userspace → graphics → CI *(being revised)* |
| [03 — Roadmap & risks](./docs/03-roadmap-and-risks.md) | Phases, legal, go/no-go *(being revised)* |
| [04 — Omarchy base](./docs/04-omarchy-base.md) | Derivative plan from Omarchy → Graycart Linux *(stub until research lands)* |

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

## Foundation — Omarchy derivative

**Decided:** Graycart Linux is based on **Omarchy** ([omarchy.org](https://omarchy.org/)). We will document ownership boundaries, update surfaces, what we keep vs replace, and how Graycart host/emulator integration sits on top — see [`docs/04-omarchy-base.md`](./docs/04-omarchy-base.md) when research lands.

Technical posture:

1. **Derive, do not cargo-cult.** Understand Omarchy’s package/config/update model before forking or overlaying.
2. **Keep upstream Arch + Omarchy compatibility where practical.** Fork only for concrete Graycart product requirements (gaming session, controller, recovery, branding).
3. **Host vs cores.** Emulator accuracy stays in `*-core` crates; this repo owns OS/session/host integration.
4. **Earlier Model 4 (bootc/UB) research** remains useful comparative context in 01–03, but is **not** the chosen primary path.

Open technical work (for 04 + revised 01–03): update/rollback story on an Omarchy/Arch base; hardware matrix; Gamescope/session policy; Steam/Proton; Graycart host install path; branding/Linux Mark; SBOM and GPL source offer when shipping images.

## Early roadmap

Aligned with [`docs/03-roadmap-and-risks.md`](./docs/03-roadmap-and-risks.md); will re-phase once Omarchy base research lands:

### Phase 0 — Research (now)

- Map Omarchy → Graycart Linux derivative plan ([04](./docs/04-omarchy-base.md)).
- Revise scope/stack/roadmap (01–03) against the Omarchy decision.
- Define supported hardware classes.
- Decide desktop/session architecture (Gamescope / Wayland) on the Omarchy base.
- Define update, rollback, and recovery strategy for the derivative.
- Establish branding and UX principles (incl. Linux Mark).
- Accept research pack before any public “download Graycart Linux” claim.

### Phase 1 — Bootstrap

- Reproducible base installation/image derived from Omarchy (only after research gate).
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
10. **Omarchy is the base** — document the derivative; do not silently diverge.

## Graycart family

- [`graycart`](https://github.com/graycart/graycart) — project umbrella
- [`graycart-gb`](https://github.com/graycart/graycart-gb) — DMG / Game Boy Color emulator
- [`graycart-gba`](https://github.com/graycart/graycart-gba) — future Game Boy Advance emulator
- **`graycart-linux`** — Omarchy-based gaming Linux host / distribution

## Contributing

Early contributions should focus on Omarchy derivative research, experiments, hardware findings, and architecture proposals rather than large implementation PRs. Read [`AGENTS.md`](./AGENTS.md) and [`ATTRIBUTION.md`](./ATTRIBUTION.md) before editing docs or sources.

When development begins, this repository will gain a formal contribution guide and build instructions.

## License

The eventual source/configuration in this repository will be licensed explicitly as implementation begins. Individual upstream components (including Omarchy and Arch packages) will retain their own licenses.
