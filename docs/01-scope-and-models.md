<!--
Mirrored from Agent Store docs/graycart-linux/ — durable copy for this repo.
Cross-family links point at Graycart Project Agent Store paths.
No secrets / no binary blobs.
-->

# Graycart Linux — Scope and product models

**Audience:** Dave (product) + implementers.  
**Status:** Research amended after **Dave decision 2026-09-12** — Omarchy-based derivative; hard customize.  
**Decision:** [DECISION-omarchy-base.md](./DECISION-omarchy-base.md) · deep dive [04-omarchy-base.md](./04-omarchy-base.md).  
**Family context:** [`docs/graycart-family/`](/cursor/stores/bc-57076e62-f812-47da-bf4c-f3bb0f3af797/docs/graycart-family/README.md) — `graycart-linux` is a **host-only** consumer of shared core API (+ C ABI).

**Internal status:** [`internal/graycart-linux/status-omarchy-base.md`](/cursor/stores/bc-57076e62-f812-47da-bf4c-f3bb0f3af797/internal/graycart-linux/status-omarchy-base.md)

---

## 0. Two meanings of “graycart-linux”

| Meaning | What it is | Status |
|---------|------------|--------|
| **A. Linux host app** | Emulator shell (Wayland/DRM, audio, FS, consent UX) loading `graycart-*-core` via Rust trait / `gc_*` C ABI | **Still required** — family design |
| **B. Branded OS / distro** | Installable system image people call “Graycart Linux” / Graycart OS | **Chosen path:** Omarchy-based **hard-customized derivative** (not stock Omarchy) |

**Implication:** Meaning A is still the Graycart-specific payload. Meaning B is how we *deliver* a lean host OS. Shipping B without library-first cores wastes packaging effort.

---

## 1. Decision frame (post-Omarchy choice)

For the chosen model, ask:

1. Does each inherited Omarchy piece help **run emulators**, or only recreate DHH’s desktop?
2. Can we **rebase** Omarchy on a cadence without drowning in merge conflicts?
3. Hardware: x86_64 PC/laptop first (Omarchy’s home); Apple Silicon / custom kernel remain non-goals.
4. Naming: Omarchy marks vs Graycart brand; “Linux” → LF Mark if used in product trademark.

---

## 2. Model comparison (updated)

| # | Model | Role after 2026-09-12 |
|---|--------|----------------------|
| **O** | **Omarchy-based derivative (hard strip + Graycart host)** | **Chosen** for Meaning B |
| 1 | From-scratch hobby OS | **Non-goal** |
| 2 | LFS / full custom distro | **Non-goal** as product (training only) |
| 3 | Yocto / Buildroot appliance | **Alternative** only if a fixed hardware SKU appears later |
| 4 | Immutable/atomic (bootc / UB / Nix) | **Alternative / fallback** if Omarchy rebase becomes untenable — *not* the chosen path |
| 5 | ChromeOS / Asahi-scale kernel fork | **Non-goal** |

---

## 3. Chosen model — Omarchy-based derivative

### What it is

Fork/overlay **[Omarchy](https://omarchy.org)** (Arch + Hyprland/Quickshell + pacman packages + ISO/update/snapshot spine — see [04](./04-omarchy-base.md)), then:

- **KEEP** install/update/Arch machinery and graphics/audio/input stack needed by hosts  
- **STRIP** productivity apps, DHH-desktop chrome, RetroArch-as-identity, Omarchy branding  
- **REPLACE/ADD** Graycart session/launcher, `graycart-*` hosts/cores, lean compositor/kiosk path  

**Not:** Omarchy with a wallpaper. **Is:** emulator-first host/appliance OS that remains mergeable with upstream Omarchy.

### Effort / timeline (order of magnitude)

| Milestone | Rough shape |
|-----------|-------------|
| Pin Omarchy tag + document strip matrix | Days (this pack) |
| Lean metapackage + Graycart branding on ISO spine | Weeks |
| CI: build → boot → host fixture | Weeks–couple months |
| Public installable + update channel + compliance | Multi-quarter; permanent rebase tax |

CVE/security firefighting is **shared** with Arch/Omarchy mirrors — Graycart owns deltas, session, and redistributor duties for what we ship.

### License & marks

- Omarchy layer: **MIT** (© DHH) — keep notices for code we ship.  
- Kernel / much of Arch userspace: **GPL-2.0** etc. — corresponding source / written offer.  
- Do **not** ship as “Omarchy”; attribute “based on Omarchy.”  
- “Graycart Linux” product mark → [Linux Mark](https://www.linuxfoundation.org/legal/the-linux-mark) plan or drop “Linux” from trademark.

### Hardware

Primary: x86_64 (Omarchy’s target). Intel Mac incidental. **Not** Graycart-maintained Asahi/M-series.

### Family fit

```text
graycart-linux (OS session + host UI)
        → family API / C ABI
graycart-gb-core → graycart-gba-core → …
```

Repo remains **host + image recipes**; cores stay in machine repos.

### Verdict

**Accepted product model** for branded OS, with hard customize.

**Sources:** [04](./04-omarchy-base.md); Omarchy primary URLs indexed there.

---

## 4–8. Alternatives and non-goals (demoted)

### Model 1 — From-scratch hobby OS

**Non-goal.** Blocks GPU/drivers/emulators. Learning only.

### Model 2 — LFS / custom distro as product

**Non-goal.** Maintenance owns whole CVE surface. Training for maintainers OK.

### Model 3 — Yocto / Buildroot

**Contingent alternative** for a future fixed appliance SKU. Not the PC/laptop Graycart Linux path while Omarchy is chosen.

### Model 4 — Immutable/atomic (bootc / Universal Blue / Nix)

Previous research **default recommendation**; **superseded** by Dave’s Omarchy decision. Keep as **escape hatch** if Omarchy strip/rebase fails. Do not parallel-build unless explicitly reopened.

### Model 5 — ChromeOS / Asahi-scale downstream kernel

**Non-goal** without funded hardware platform team.

*(Detailed effort tables from earlier research remain conceptually valid for these alternatives but are not the active plan — see git history of this file if needed.)*

---

## 9. How this maps to the Graycart family

| Artifact | Role |
|----------|------|
| `graycart-gb` / `gba` / `nes`… | Machines (`*-core`) + desktop hosts |
| `graycart-abi` | Shared trait + C header |
| `graycart-linux` | **Host + Omarchy-derived OS delivery** — no machine logic in OS recipes |
| Strip/keep matrix | [04 §2](./04-omarchy-base.md#2-strip--keep--replace-matrix) |

Phased sequence:

1. Family API / library-first cores (ongoing).  
2. Prove host on stock Arch/Omarchy **or** lean Graycart image early.  
3. Ship Meaning B: stripped Omarchy derivative + Graycart session.  
4. Optional Model 3 only with hardware SKU.  
5. Never: Models 1, 2-as-product, 5.

---

## 10. Explicit non-goals

- Shipping **stock Omarchy** as Graycart Linux.  
- **Omarchy + wallpaper** / theme-only differentiation.  
- From-scratch kernel/OS; full independent distro; Asahi-scale forks.  
- Putting **core logic** in the OS image repo.  
- Vendoring BIOS / commercial ROMs.  
- Blocking GBA P0–P9 on OS cosmetics.  
- Using Omarchy/Basecamp marks as our product identity.  
- Claiming “Graycart Linux” without Linux Mark check.

---

## 11. Open choices (narrowed)

- [x] Base: **Omarchy** (Dave, 2026-09-12).  
- [x] Customize: **hard strip** (Dave clarification, same day).  
- [ ] Brand string: **Graycart Linux** vs **Graycart OS**.  
- [ ] Default session: stripped Hyprland vs Gamescope-first.  
- [ ] Overlay packages vs deeper git fork ([04 §6](./04-omarchy-base.md#6-open-implementation-choices-post-decision)).

---

## 12. Sources (URL index)

| Topic | URL |
|-------|-----|
| Decision | ./DECISION-omarchy-base.md |
| Omarchy deep dive | ./04-omarchy-base.md |
| Omarchy site | https://omarchy.org |
| Omarchy repo | https://github.com/basecamp/omarchy |
| Linux Mark | https://www.linuxfoundation.org/legal/the-linux-mark |
| Kernel license rules | https://docs.kernel.org/process/license-rules.html |
| Family API | ../graycart-family/README.md |
