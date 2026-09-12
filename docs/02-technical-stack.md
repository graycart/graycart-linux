<!--
Mirrored from Agent Store docs/graycart-linux/ — durable copy for this repo.
Cross-family links point at Graycart Project Agent Store paths.
No secrets / no binary blobs.
-->

# Graycart Linux — Technical stack (Omarchy-derived)

**Audience:** Dave + implementers.  
**Status:** Research amended after Omarchy decision — stack **starts from Omarchy’s real components**, then applies the [strip/keep matrix](./04-omarchy-base.md#2-strip--keep--replace-matrix).  
**Decision:** [DECISION-omarchy-base.md](./DECISION-omarchy-base.md).  
**Family:** host-only over `*-core` + ABI; graphics today = **egui + wgpu + winit** ([binary-size](/cursor/stores/bc-57076e62-f812-47da-bf4c-f3bb0f3af797/docs/binary-size-investigation.md)).

**Internal:** [`internal/graycart-linux/status-omarchy-base.md`](/cursor/stores/bc-57076e62-f812-47da-bf4c-f3bb0f3af797/internal/graycart-linux/status-omarchy-base.md)

---

## 0. How to read vs 01 / 03 / 04

| Doc | Question |
|-----|----------|
| **01** | Product model — **Omarchy derivative chosen** |
| **02 (this)** | Technical apparatus of that derivative |
| **03** | Phases, risks, legal, rebase |
| **04** | Omarchy facts + KEEP/STRIP/REPLACE matrix |

**Product tension (resolved direction):** Prefer Omarchy’s install/update spine over inventing bootc/LFS. Prefer **lean Graycart session** over keeping Hyprland/Quickshell chrome we don’t need.

---

## 1. Stack map (as discovered upstream → Graycart delta)

```text
Firmware (UEFI; Secure Boot OFF for Omarchy-class install today)
  → Limine (default Omarchy ≥2.0; snapshots)
  → Linux kernel (Arch linux; linux-t2 only on Intel T2 Macs — not our focus)
  → initramfs / LUKS (encryption default on Omarchy ISO)
  → systemd (PID 1)
  → SDDM (login; brand → Graycart)  [or autologin kiosk]
  → Wayland compositor
        KEEP candidate: Hyprland (stripped config)
        REPLACE candidate: Gamescope / minimal compositor for appliance
  → Quickshell / Omarchy desktop chrome  →  STRIP as product identity
  → Graycart session / launcher          →  ADD
  → graycart-* host (wgpu/egui)          →  ADD
```

| Layer | Omarchy today | Graycart posture |
|-------|---------------|------------------|
| Distro base | Arch Linux | KEEP |
| Packages | pacman + omarchy-pkgs + omarchy-mirror | KEEP spine; ADD graycart repo; STRIP fat metapackages |
| Installer | omarchy-iso (only supported path) | KEEP flow; rebrand; lean manifest |
| Updates | `omarchy update`, channels, ALPM guard, migrations | KEEP mechanics; rename/wrap; Graycart migrations |
| Rollback | Limine snapshots (root, not `/home`) | KEEP |
| libc / init | glibc, systemd | KEEP |
| Compositor | Hyprland | KEEP only if lean; else REPLACE |
| Shell UI | Quickshell + Omarchy menu | STRIP / REPLACE with Graycart UI |
| DRM/GPU | Mesa + Vulkan | KEEP |
| Audio | PipeWire stack (Arch defaults) | KEEP |
| Input | libinput / HID | KEEP |
| Host UI | — | ADD egui/wgpu host |
| Cores | — | ADD packages depending on published cores |

Primary sources: [04 §0](./04-omarchy-base.md#0-what-omarchy-is-today-primary-sources).

---

## 2. Boot chain

Same logical chain as any modern Arch desktop Omarchy ships:

```text
Power-on → UEFI → Limine → kernel + initramfs
  → unlock LUKS (if enabled) → switch_root → systemd
  → display manager → session → Graycart host
```

**MVP CI:** QEMU OVMF, often **no encryption** for unattended speed (Omarchy allows Ctrl+C path / cidata).  
**Product default:** follow Omarchy’s encrypted install unless lab policy says otherwise.  
**Secure Boot:** Omarchy requires it off today — document as Graycart limitation until we invest in signed UKI/MOK (P3+, optional).

References: [systemd bootup(7)](https://www.freedesktop.org/software/systemd/man/latest/bootup.html); [Omarchy snapshots](https://github.com/basecamp/omarchy/blob/quattro/manual/47-system-snapshots.md).

---

## 3. Kernel

| Policy | Choice |
|--------|--------|
| Tree | **Arch `linux`** (via Omarchy mirror channels) — not a Graycart fork |
| Config | Distro config; no Graycart fragment unless measured need |
| Special | Do not take on `linux-t2` / Asahi maintenance |
| Updates | Flow through Graycart update wrapper + stable/edge channel strategy |

01 Model 5 remains a non-goal.

---

## 4. Userspace

- **libc:** glibc (matches `x86_64-unknown-linux-gnu` Graycart releases).  
- **Init:** systemd (logind seats for Wayland).  
- **Core utils:** Arch defaults — do not BusyBox the product image.  
- **Omarchy packages:** replace `omarchy` / `omarchy-settings` style payloads with **lean Graycart equivalents** that still honor `/etc` + migration patterns ([Quattro package re-arch](https://github.com/basecamp/omarchy/releases/tag/v4.0.0)).

---

## 5. Package / update story

| Mechanism | Role |
|-----------|------|
| Arch core/extra/multilib via Omarchy mirror | Base OS |
| omarchy-pkgs pattern | Template for building/signing Graycart pkgs |
| graycart repo | Hosts, session, branding assets, optional tools |
| Update CLI | Snapshot → migrations → pacman sync (do not bypass casually) |
| Channels | Start users on **stable**-like pin; maintainers use edge |

**Strip:** default image must not install the full Omarchy “everything I use” set.  
**Add:** `graycart-host-base`, `graycart-session`, versioned `graycart-gb` (then gba).

---

## 6. Graphics / input for emulators

Hosts need: Mesa Vulkan ICD, Wayland (or Gamescope), xkbcommon, libinput, PipeWire/ALSA for cpal, HID for gilrs.

| Path | Use |
|------|-----|
| **A. Lean Hyprland + Graycart as client** | Default if strip succeeds |
| **B. Gamescope session** | Strong for controller-first / appliance |
| **C. Direct DRM/KMS** | Defer — research only |

**STRIP** Omarchy rice, bars, and productivity bindings that fight fullscreen host.  
**CI:** llvmpipe/Weston or nested Hyprland; never block boot smoke on Vulkan.

Wayland architecture reminder: compositor owns KMS + input ([Wayland Architecture](https://wayland.freedesktop.org/docs/book/Architecture.html)).

---

## 7. Secure Boot, SBOM, repro

| Topic | Graycart bar |
|-------|----------------|
| Secure Boot | Document “disabled for install” until optional MOK/UKI work |
| Repro | Pin Omarchy tag + package lists + Graycart lockfile; dual CI rebuild on releases |
| SBOM | SPDX/CycloneDX per ISO: kernel, Mesa, compositor, Graycart hosts, licenses |
| GPL | Source bundle / written offer with every public image ([03](./03-roadmap-and-risks.md)) |

---

## 8. CI farm shape

```text
PR/main → build graycart packages + ISO/rootfs (omarchy-iso-derived)
       → QEMU: Limine → systemd → session
       → launch graycart-gb host + fixture ROM
       → publish SBOM + source stub
```

Split **emulator crate CI** from **image CI**. Cache pacman and ISO mirrors. Unattended install via Omarchy-compatible `cidata` ([iso README](https://github.com/omacom-io/omarchy-iso/blob/quattro/README.md)).

---

## 9. Default stack picks (decision table)

| Layer | Graycart choice |
|-------|-----------------|
| Base | Omarchy / Arch (pinned tag) |
| Installer | Fork/patch omarchy-iso, lean manifest |
| Bootloader | Limine + snapshots |
| Kernel | Arch linux via mirror |
| Init | systemd |
| Session | Graycart session (REPLACE Omarchy desktop chrome) |
| Compositor | Lean Hyprland **or** Gamescope (open choice) |
| Packages | pacman + graycart repo |
| Updates | Wrapped Omarchy-style update + migrations |
| Host | Family ABI + egui/wgpu |
| Apps default | STRIP Omarchy preinstalls; optional Steam later |

---

## 10. Phased MVP (stack view)

### M1 — Boots lean Graycart image

QEMU boots artifact to shell **or** graphical target with **stripped** package set; CI green; Omarchy brand absent.

### M2 — Runs graycart-gb

Session launches host; fixture ROM; audio + keyboard/gamepad path; no commercial ROMs.

### M3 — Installable product

Public ISO, update/rollback, SBOM/source offer, hardware matrix, Linux Mark path if needed.

Do not ship M3 before M2.

---

## 11. Open stack choices

- [ ] Hyprland-lean vs Gamescope-first  
- [ ] Keep any Quickshell bits vs full replace  
- [ ] Encryption default for CI vs product  
- [ ] When/if to offer optional Steam/Proton packages  

---

## 12. Sources

| Topic | URL |
|-------|-----|
| Omarchy deep dive | ./04-omarchy-base.md |
| Omarchy updates | https://github.com/basecamp/omarchy/blob/quattro/manual/30-updates.md |
| Omarchy ISO | https://github.com/omacom-io/omarchy-iso |
| Omarchy pkgs | https://github.com/omacom-io/omarchy-pkgs |
| Omarchy mirror | https://github.com/omacom-io/omarchy-mirror |
| systemd bootup | https://www.freedesktop.org/software/systemd/man/latest/bootup.html |
| Wayland architecture | https://wayland.freedesktop.org/docs/book/Architecture.html |
| Kernel license | https://docs.kernel.org/process/license-rules.html |
| Family API | ../graycart-family/README.md |
| Scope | ./01-scope-and-models.md |
| Roadmap | ./03-roadmap-and-risks.md |
