# AGENTS.md — Graycart Linux

How to change this project. Product name **Graycart Linux**; repo **`graycart-linux`**. Family: [graycart](https://github.com/graycart/graycart). Peers: [graycart-gb](https://github.com/graycart/graycart-gb), [graycart-gba](https://github.com/graycart/graycart-gba).

Graycart Linux is currently in the **research / placeholder stage**. Do not treat this repository as an invitation to prematurely build a distribution.

## Current objective

**Product decision:** **Omarchy-derived; Graycart-stripped host OS.** Research and document the aggressive strip/customize plan (see [`docs/04-omarchy-base.md`](./docs/04-omarchy-base.md)), then establish update/recovery, hardware classes, session strategy, packaging, and Graycart host integration — without shipping stock Omarchy branding.

Before implementation, Dave review of [`docs/`](./docs/README.md) (01–04, after Omarchy rewrite) is the research gate.

### How agents should navigate

1. Read **this file** + [`ATTRIBUTION.md`](./ATTRIBUTION.md) before editing sources.
2. Research pack: [`DECISION`](./docs/DECISION-omarchy-base.md) → [`04`](./docs/04-omarchy-base.md) → [`01`](./docs/01-scope-and-models.md) → [`02`](./docs/02-technical-stack.md) → [`03`](./docs/03-roadmap-and-risks.md); index [`docs/README.md`](./docs/README.md); vision [`docs/vision.md`](./docs/vision.md).
3. Family-standard core API (Agent Store): `/cursor/stores/bc-57076e62-f812-47da-bf4c-f3bb0f3af797/docs/graycart-family/01-core-api.md` — linux is **host-only**; do not put emulator machine logic here.
4. Own only your partition paths; keep PRs docs-first until the research gate clears.

## Working rules

- Research before implementation. Prefer primary upstream documentation for **Omarchy**, Arch Linux, systemd, kernel, Mesa, PipeWire, Steam/Proton, Gamescope, Hyprland/Wayland, and related packaging/update surfaces.
- Derive from Omarchy, then strip hard. Understand ownership and update boundaries before aggressive removal and Graycart customization; do not ship stock Omarchy branding.
- Keep upstream compatibility wherever practical. Fork only when there is a concrete product requirement.
- Prefer declarative/reproducible configuration over opaque mutation scripts.
- Keep components modular and narrowly owned. Avoid giant bootstrap scripts and miscellaneous helper files.
- Do not add placeholder UI, fake integrations, or speculative abstractions.
- Controller-first must not mean keyboard/mouse-hostile.
- Host services such as audio, Bluetooth, input, display configuration, updates, and diagnostics must fail loudly and observably rather than silently degrading.
- Treat rollback and recovery as first-class architecture, not post-release polish.
- Performance work must be evidence-driven. Benchmark before and after changes.
- Never commit credentials, machine-specific secrets, proprietary firmware, copyrighted game content, or user data.

## Attribution (file headers — mandatory)

Full policy + examples: [`ATTRIBUTION.md`](./ATTRIBUTION.md).

If a source file (future Rust/scripts **or** Markdown in this repo) references someone else's **code** or **internet documentation**, put **credit at the top of that file**:

- what was used
- URL and/or name
- brief note (inspired by / ported from / cited)

Provenance folders and README link lists alone are **not** enough for in-tree sources that depend on those materials.

Example (Markdown):

```markdown
<!--
Cited: Linux Foundation — The Linux Mark
URL: https://www.linuxfoundation.org/legal/the-linux-mark
Note: trademark planning only; not legal advice.
-->
```

Do not commit entire manuals, proprietary firmware blobs, or commercial game content. Keep excerpts minimal and attributed.

## Graycart integration

Graycart Linux is a sibling to `graycart-gb` and `graycart-gba`. Emulator cores must not become Linux-distribution-specific merely to integrate with this project. Prefer stable interfaces between the OS shell and Graycart applications (family host/core seams).

- The **host** owns session, display, audio, input, packaging, and recovery.
- **Cores** (`*-core`) own accuracy; never fork cores into this repo for cosmetics.

## Changes during the research stage

Good changes:

- architecture notes
- experiments with clear conclusions
- hardware compatibility findings
- reproducible prototypes
- benchmark results
- upstream capability research
- threat/recovery/update analysis
- mirrors of accepted Agent Store research into `docs/`

Avoid large implementation PRs until the relevant design has been agreed and documented (see [03 go/no-go](./docs/03-roadmap-and-risks.md)).

## Validation

Every implementation change, once implementation begins, should define its own acceptance criteria. System-level changes should be tested on clean installs or reproducible images rather than relying only on a developer workstation that has accumulated state.

Docs-only PRs: no secrets, no binary blobs, links resolve or are intentionally Agent Store absolute paths.
