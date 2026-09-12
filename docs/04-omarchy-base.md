<!--
Mirrored from Agent Store docs/graycart-linux/ — durable copy for this repo.
Cross-family links point at Graycart Project Agent Store paths.
No secrets / no binary blobs.
-->

# Omarchy as graycart-linux base — deep dive

**Audience:** Implementers building the derivative; Dave for strip/keep review.  
**Status:** Research after [DECISION-omarchy-base.md](./DECISION-omarchy-base.md) (2026-09-12).  
**Posture:** Omarchy-based **derivative appliance/host** — customize hard; not “Omarchy + wallpaper.”  
**Siblings:** [01](./01-scope-and-models.md) · [02](./02-technical-stack.md) · [03](./03-roadmap-and-risks.md)

---

## 0. What Omarchy is today (primary sources)

| Fact | Source |
|------|--------|
| Opinionated **Arch Linux** distribution by DHH; “omakase” desktop | [manual/01-welcome](https://github.com/basecamp/omarchy/blob/quattro/manual/01-welcome-to-omarchy.md), [hey.com launch](https://world.hey.com/dhh/omarchy-is-out-4666dd31) |
| Site / product | https://omarchy.org |
| Main repo | https://github.com/basecamp/omarchy (canonical org redirect: [omacom/omarchy](https://github.com/omacom/omarchy)); default branch **`quattro`**; MIT, © David Heinemeier Hansson ([LICENSE](https://github.com/basecamp/omarchy/blob/quattro/LICENSE)) |
| Desktop | **Hyprland** (tiling Wayland) + **Quickshell** desktop kit ([welcome](https://github.com/basecamp/omarchy/blob/quattro/manual/01-welcome-to-omarchy.md)) |
| Install | **ISO only** supported path; installs Arch + Omarchy packages from bundled mirror ([omarchy-iso README](https://github.com/omacom-io/omarchy-iso/blob/quattro/README.md), [getting started](https://github.com/basecamp/omarchy/blob/quattro/manual/02-getting-started.md)) |
| Packages | Pacman packages from [omacom-io/omarchy-pkgs](https://github.com/omacom-io/omarchy-pkgs); `omarchy` + `omarchy-settings` pair; system files via `/etc` (Quattro) |
| Arch mirror | [omacom-io/omarchy-mirror](https://github.com/omacom-io/omarchy-mirror) — stable ≈1 month behind; edge tracks Arch hourly |
| Updates | `_Update > Omarchy_` / `omarchy update`; ALPM guard discourages raw `pacman -Syu`; channels stable / RC / edge / dev ([updates](https://github.com/basecamp/omarchy/blob/quattro/manual/30-updates.md)) |
| Boot / rollback | **Limine** default since 2.0; snapshots on update; restore root not `/home` ([snapshots](https://github.com/basecamp/omarchy/blob/quattro/manual/47-system-snapshots.md)) |
| Display manager | SDDM (tied to Plymouth branding tooling) ([branding](https://github.com/basecamp/omarchy/blob/quattro/manual/41-branding.md)) |
| Current line | Quattro / v4 — package-based updates, dual-boot/free-space install, unified Quickshell ([v4.0.0 notes](https://github.com/basecamp/omarchy/releases/tag/v4.0.0)) |
| Secure Boot | Must **disable** Secure Boot/TPM to install ([getting started](https://github.com/basecamp/omarchy/blob/quattro/manual/02-getting-started.md)) |
| Mac | Intel Mac supported; **M-series not directly supported** ([mac support](https://github.com/basecamp/omarchy/blob/quattro/manual/44-mac-support.md)) |
| Gaming extras | Steam/RetroArch/etc. under *Install > Gaming* — optional, not core OS identity ([gaming](https://github.com/basecamp/omarchy/blob/quattro/manual/26-gaming.md)) |

**Kernel / userspace relationship:** Omarchy is **not** a custom kernel tree. It is Arch’s kernel (plus special cases like `linux-t2` on Intel T2 Macs) + systemd + glibc + Mesa/PipeWire stack + Omarchy’s configs, migrations, and pacman packages.

**Trademarks:** MIT covers **code**, not the Omarchy / Basecamp / DHH product identity. Graycart must ship under **Graycart** branding; attribute “based on Omarchy” nominatively; do not imply official Omarchy spin. Separately, product name **“Graycart Linux”** still needs [Linux Mark](https://www.linuxfoundation.org/legal/the-linux-mark) planning.

---

## 1. Product posture (Dave)

```text
Omarchy upstream (install, pacman spine, migrations, Arch)
        │  fork / overlay / rebrand
        ▼
graycart-linux  =  STRIP heavy DHH desktop  +  ADD Graycart host/session
        │  family API
        ▼
graycart-*-core (gb → gba → …)
```

- **Yes:** lean host OS that boots to a Graycart-capable session, inherits Omarchy’s reproducible ISO/update/snapshot machinery, stays mergeable.
- **No:** redistribution of stock Omarchy aesthetics as the product; keeping Obsidian/LibreOffice/Neovim-centric chrome “because Omarchy has it”; forking cores into the OS repo.

---

## 2. Strip / keep / replace matrix

This is the working contract for packaging and session design. Review when rebasing Quattro+.

### 2.1 KEEP (inherit or thin-wrap)

| Area | Why |
|------|-----|
| **Arch + pacman + Omarchy pkg/mirror pattern** | Proven update/CVE path; Graycart adds its own repo, does not invent a format |
| **ISO install spine** (`omarchy-iso` flow: configurator → Arch → packages → chroot setup → user setup) | Only supported install story upstream; unattended `cidata` useful for CI/fleet |
| **`omarchy update` analogue** (snapshot → migrate → pacman → channel) | Rolling without this = support hell; rename CLI to Graycart but keep mechanics |
| **Limine + update snapshots** | Rollback for bad upgrades; keep unless we deliberately switch bootloader (high cost) |
| **systemd + logind seat** | Wayland/DRM seat management; matches wgpu host needs |
| **Mesa / Vulkan / PipeWire / PipeWire-ALSA / libinput / udev HID** | Required for egui/wgpu/cpal/gilrs hosts |
| **NetworkManager (or equivalent connectivity)** | Install + updates need network; keep lean |
| **Full-disk encryption default (optional off for lab)** | Sensible for portable machines; CI may use no-encrypt |
| **Migration hooks pattern** | Lets Graycart ship schema upgrades alongside package bumps |
| **Channel idea (stable / edge)** | Pin Graycart users to tested Arch/Omarchy rebase windows |
| **GPL corresponding-source discipline for redistributed kernel/userspace** | Distributor duty regardless of MIT Omarchy layer |

### 2.2 STRIP (do not ship as product defaults)

| Area | Why strip |
|------|-----------|
| **DHH “everything I use” app set** — Obsidian, LibreOffice, Kdenlive, OBS, Winamp-style player, commercial web apps, etc. | Not emulator-host value; bloat ISO and attack surface ([welcome](https://github.com/basecamp/omarchy/blob/quattro/manual/01-welcome-to-omarchy.md)) |
| **Heavy theming / rice surface as product identity** — Omarchy themes, screensaver ASCII brand, About glint as *Graycart* identity | Replace with Graycart brand; optional theme tooling can stay for power users later |
| **Neovim-as-center, agentic desktop chrome, Basecamp plugin surfaces** | Developer-laptop product, not Graycart appliance/host |
| **Omarchy menu / launcher IA** oriented around Install>Service / Style / AI | Replace with Graycart session/launcher |
| **Preinstalled RetroArch-as-default retro stack** | Competes with Graycart cores; optional package OK, not default identity ([gaming](https://github.com/basecamp/omarchy/blob/quattro/manual/26-gaming.md)) |
| **Steam/Proton/Lutris/Heroic/Battle.net as first-boot defaults** | Fine as *optional* later; not required for Graycart host MVP |
| **Omarchy wordmarks, logos, Plymouth/SDDM art** | Trademark/confusion; use Graycart assets only |
| **Dev-channel git-checkout-in-home workflow as supported product mode** | Upstream contributor path; Graycart maintainers may use it internally |
| **Opinionated dotfiles that fight a kiosk/lean session** | Keep only what the host/session needs |

**Strip method:** prefer **not installing** packages (lean metapackage) over shipping then `Remove > Preinstalls`. First-boot image should already be lean.

### 2.3 REPLACE / ADD

| Piece | Action |
|-------|--------|
| **Session / first paint** | Graycart session: boot → (login) → lean compositor → Graycart host or host launcher |
| **Launcher / menu** | Graycart-branded launcher: run hosts, load fixtures (dev), settings for display/audio/controllers — not Omarchy’s productivity IA |
| **Compositor path** | Default: keep **Hyprland** only if config is stripped to lean gaming/host layout; evaluate **Gamescope** or minimal compositor for appliance/kiosk SKU (see [02](./02-technical-stack.md)) |
| **Display manager branding** | SDDM/Plymouth → Graycart artwork; or autologin kiosk where appropriate |
| **Metapackage** | `graycart-host-base` (deps for wgpu host) + `graycart-session` + optional `graycart-devtools` |
| **Family cores** | Packages or pinned binaries: `graycart-gb` / `graycart-gba` hosts consuming `*-core` + ABI — **gb path first**, gba when core API ready |
| **Repo** | `graycart` pacman repo (or overlay) beside Arch/Omarchy mirrors; pin versions for CI |
| **Docs / support matrix** | Graycart hardware + “unsupported” list; do not claim Omarchy Discord as support |
| **CLI** | `graycart-update` (or wrap) that still runs snapshot + migrations + pacman; do not teach users to bypass guards casually |

### 2.4 Matrix summary

| | KEEP | STRIP | REPLACE/ADD |
|---|------|-------|-------------|
| Install/ISO | ✓ spine | Omarchy splash/brand | Graycart ISO brand, lean package set |
| Updates | ✓ channels + snapshots | Upstream-only messaging | Graycart migrations + repo |
| Desktop apps | Mesa/PipeWire/input | Productivity/creative suite | Graycart host apps |
| Session | systemd/Wayland seats | DHH Quickshell chrome as identity | Graycart session ± lean Hyprland/Gamescope |
| Emulation | Vulkan/audio/HID | RetroArch-default story | graycart-* cores/hosts |
| Brand | MIT attribution to Omarchy code | Omarchy marks | Graycart + Linux Mark plan |

---

## 3. How to derivative (engineering)

### 3.1 Recommended shape

1. **Track upstream** `omacom/omarchy` (and iso/pkgs as needed) as git remotes; Graycart lives in `graycart/graycart-linux` (and possibly `graycart-os` / pkgs sibling).
2. **Thin fork or overlay:** prefer overlay packages that **replace** `omarchy` / `omarchy-settings` equivalents with Graycart packages providing `/etc` + session — keeps pacman flow.
3. **Metapackage first:** define `graycart-base` depends = Arch bits we need + stripped Omarchy runtime libs; conflicts/replaces bulky omarchy optional groups.
4. **ISO:** fork or patch [omarchy-iso](https://github.com/omacom-io/omarchy-iso) to pull Graycart packages, Graycart branding, lean manifest; keep `cidata` unattended for CI.
5. **CI:** build ISO or rootfs → QEMU boot → SDDM/autologin → launch Graycart host fixture (ROM-free).

### 3.2 Branding checklist

- [ ] All Plymouth/SDDM/screensaver/About strings Graycart  
- [ ] ISO filename / URLs under graycart  
- [ ] README: “Based on Omarchy (MIT); not affiliated with Basecamp/DHH/Omarchy”  
- [ ] Retain MIT notices for copied Omarchy code  
- [ ] Linux Mark path if marketing “Graycart Linux”

### 3.3 Packages & CI

| Artifact | Owner |
|----------|-------|
| Host crates | Existing graycart-* repos; publish versioned binaries/packages into Graycart repo |
| Image recipes | graycart-linux / iso repo |
| SBOM + source offer | Every public ISO/tag ([03](./03-roadmap-and-risks.md)) |
| Rebase bot / scheduled merge | Merge Omarchy `quattro`/tags; run migration + QEMU smoke |

### 3.4 Rebase policy (stay mergeable)

1. **Minimize edit surface** in vendored Omarchy files; prefer Graycart packages that override via pacman file ownership / drop-ins.
2. **Never** rewrite Omarchy history; merge or rebase onto release tags.
3. **Carry a `UPSTREAM.md`:** pinned Omarchy commit/tag, list of Graycart deltas, known conflict hotspots (shell menu, default packages, branding).
4. **Migrations:** Graycart migrations run *after* or *instead of* Omarchy ones we stripped — document order.
5. **Reject** one-way “we diverged forever” unless Dave explicitly abandons Omarchy (then revisit Model 4 atomic).

---

## 4. Graycart emulators — plug-in points

```text
Limine → LUKS (optional) → systemd → SDDM/autologin
    → compositor (lean Hyprland | Gamescope | future kiosk)
        → graycart-session / launcher
            → graycart-gb host  →  graycart-gb-core
            → graycart-gba host →  graycart-gba-core   (when ready)
            → later nes/snes/n64
```

| Integration | Notes |
|-------------|-------|
| **Wayland client** | Current hosts: winit + egui + wgpu — match Omarchy’s Wayland environment; verify Vulkan ICD |
| **Fullscreen / gamescope** | Appliance mode: Gamescope embeds host like a game; good controller-first path |
| **.desktop / launcher entries** | Ship Graycart entries only; no RetroArch-first |
| **Controllers** | Keep udev/HID; optional Xbox Bluetooth package à la Omarchy *if* tested |
| **Fixtures CI** | Headless or Weston/Hyprland nested; commercial ROMs forbidden |
| **Core ≠ OS** | OS packages depend on published cores; no core source trees inside image repo |

Family contract unchanged: [graycart-family](/cursor/stores/bc-57076e62-f812-47da-bf4c-f3bb0f3af797/docs/graycart-family/README.md).

---

## 5. What building on Omarchy means (short)

1. We inherit Arch rolling + Omarchy’s ISO/update/snapshot **machinery**, then **delete** most of the DHH desktop product.  
2. Differentiation is **Graycart session + cores**, not themes.  
3. Maintenance cost is **rebase + strip discipline**, not inventing pacman.  
4. Legal: MIT for Omarchy code we keep; **GPL** for kernel/userspace we redistribute; **marks** for Omarchy and Linux.  
5. Success = stranger boots Graycart image and plays via family host — not “feels like Omarchy.”

---

## 6. Open implementation choices (post-decision)

- [ ] Overlay packages vs hard git fork of `omarchy` + `omarchy-iso`  
- [ ] Default session: stripped Hyprland vs Gamescope-first appliance  
- [ ] Whether Quickshell remains at all after strip  
- [ ] Public brand: Graycart Linux vs Graycart OS  
- [ ] How much optional Steam/Proton to offer *after* host MVP  

---

## 7. Sources

| Topic | URL |
|-------|-----|
| Omarchy site | https://omarchy.org |
| Main repo | https://github.com/basecamp/omarchy / https://github.com/omacom/omarchy |
| LICENSE (MIT) | https://github.com/basecamp/omarchy/blob/quattro/LICENSE |
| Welcome / stack | https://github.com/basecamp/omarchy/blob/quattro/manual/01-welcome-to-omarchy.md |
| Install | https://github.com/basecamp/omarchy/blob/quattro/manual/02-getting-started.md |
| Updates / channels | https://github.com/basecamp/omarchy/blob/quattro/manual/30-updates.md |
| Snapshots / Limine | https://github.com/basecamp/omarchy/blob/quattro/manual/47-system-snapshots.md |
| Branding hooks | https://github.com/basecamp/omarchy/blob/quattro/manual/41-branding.md |
| Gaming extras | https://github.com/basecamp/omarchy/blob/quattro/manual/26-gaming.md |
| Mac / T2 | https://github.com/basecamp/omarchy/blob/quattro/manual/44-mac-support.md |
| ISO | https://github.com/omacom-io/omarchy-iso |
| Pkgs | https://github.com/omacom-io/omarchy-pkgs |
| Mirror | https://github.com/omacom-io/omarchy-mirror |
| DHH launch post | https://world.hey.com/dhh/omarchy-is-out-4666dd31 |
| Quattro / v4 | https://github.com/basecamp/omarchy/releases/tag/v4.0.0 |
| Linux Mark | https://www.linuxfoundation.org/legal/the-linux-mark |
| Decision | ./DECISION-omarchy-base.md |
