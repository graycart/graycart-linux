# AGENTS.md — Graycart Linux

Graycart Linux is currently in the **research / placeholder stage**. Do not treat this repository as an invitation to prematurely build a distribution.

## Current objective

Research and define a gaming-focused Linux system in the Graycart family, likely using an Omarchy-inspired Arch Linux foundation.

Before implementation, establish the architecture, update/recovery model, supported hardware classes, desktop/session strategy, packaging approach, and acceptance criteria.

## Working rules

- Research before implementation. Prefer primary upstream documentation for Arch Linux, systemd, kernel, Mesa, PipeWire, Steam/Proton, Gamescope, Hyprland/Wayland, and whatever base project is selected.
- Do not copy another distribution wholesale. Understand ownership, update boundaries, and why each component exists.
- Keep upstream compatibility wherever practical. Fork only when there is a concrete product requirement.
- Prefer declarative/reproducible configuration over opaque mutation scripts.
- Keep components modular and narrowly owned. Avoid giant bootstrap scripts and miscellaneous helper files.
- Do not add placeholder UI, fake integrations, or speculative abstractions.
- Controller-first must not mean keyboard/mouse-hostile.
- Host services such as audio, Bluetooth, input, display configuration, updates, and diagnostics must fail loudly and observably rather than silently degrading.
- Treat rollback and recovery as first-class architecture, not post-release polish.
- Performance work must be evidence-driven. Benchmark before and after changes.
- Never commit credentials, machine-specific secrets, proprietary firmware, copyrighted game content, or user data.

## Graycart integration

Graycart Linux is a sibling project to `graycart-gb` and `graycart-gba`. Emulator cores should not become Linux-distribution-specific merely to integrate with this project. Prefer stable interfaces between the OS shell and Graycart applications.

## Changes during the research stage

Good changes:

- architecture notes
- experiments with clear conclusions
- hardware compatibility findings
- reproducible prototypes
- benchmark results
- upstream capability research
- threat/recovery/update analysis

Avoid large implementation PRs until the relevant design has been agreed and documented.

## Validation

Every implementation change, once implementation begins, should define its own acceptance criteria. System-level changes should be tested on clean installs or reproducible images rather than relying only on a developer workstation that has accumulated state.
