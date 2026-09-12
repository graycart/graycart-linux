<!--
Mirrored from Agent Store docs/graycart-linux/ — durable copy for this repo.
Cross-family links below point at the Graycart Project Agent Store paths.
No secrets / no binary blobs.
-->

# Graycart Linux — Technical stack (boot → image)

**Audience:** Dave (stack decisions) + implementers.  
**Status:** Research / apparatus — **no implementation in this pass**.  
**Assumption:** Custom distro / LFS / Buildroot-class image on a **mainline (or lightly patched) Linux kernel** — not a from-scratch kernel.  
**Sibling research:** [01 — Scope & models](./01-scope-and-models.md) · [03 — Roadmap & risks](./03-roadmap-and-risks.md).  
**Family context:** [`graycart-linux` = host-only](/cursor/stores/bc-57076e62-f812-47da-bf4c-f3bb0f3af797/docs/graycart-family/README.md) over `*-core` + ABI; graphics today = **egui + wgpu + winit** (~15–22 MiB host weight — [binary-size investigation](/cursor/stores/bc-57076e62-f812-47da-bf4c-f3bb0f3af797/docs/binary-size-investigation.md)).

**Internal status:** [`internal/graycart-linux/status-tech-stack.md`](/cursor/stores/bc-57076e62-f812-47da-bf4c-f3bb0f3af797/internal/graycart-linux/status-tech-stack.md)

---

## 0. How to read this doc vs 01 / 03

| Doc | Question |
|-----|----------|
| **01** | *Should* we ship a branded OS? Default: Model 4 atomic image **after** host-on-stock-distro |
| **02 (this)** | *If* we build/ship an image (Buildroot/LFS-class **or** atomic overlay), what is the technical apparatus? |
| **03** | Phases, cost, legal traps, go/no-go checklist |

**Product tension (explicit):** 01 prefers inheriting Fedora/Nix security via atomic images; a pure LFS/Buildroot product owns the whole CVE surface. This doc still documents Buildroot/LFS-class stacks because they teach the boot→userspace chain and remain the right tool for a **fixed appliance SKU** (01 Model 3). For a PC “Graycart Linux,” prefer **reusing upstream composition** (bootc/OSTree/Nix) while keeping the same boot, graphics, and SBOM discipline below.

---

## 1. Boot chain — firmware → bootloader → kernel → initramfs → init

```text
Power-on
  → Firmware (UEFI or legacy BIOS / U-Boot on embedded)
  → Bootloader (systemd-boot, GRUB, U-Boot, or UKI as EFI application)
  → Kernel image (+ optional separate initramfs; or both packed in a UKI)
  → Early userspace (/init in initramfs)
  → switch_root / pivot to real root
  → PID 1 on real root (BusyBox init, OpenRC, systemd, …)
  → Graycart session / shell / host
```

### 1.1 Firmware

- **x86_64 PC:** UEFI is the default target. Firmware locates an EFI System Partition (ESP), loads a bootloader or Unified Kernel Image (UKI), and may enforce Secure Boot (see §6).
- **Embedded / appliance:** Often [U-Boot](https://docs.u-boot.org/) (or vendor boot ROM → U-Boot). Same logical handoff: load kernel + DTB + initramfs, jump to kernel entry.
- Firmware does **minimal** hardware bring-up; Linux and userspace own the rest.

### 1.2 Bootloader

Role: load kernel + initramfs (or UKI), pass command line (`root=`, `rdinit=`, console, …), optionally chainload.

| Choice | When |
|--------|------|
| **systemd-boot** | UEFI + simple BLS entries; pairs well with UKIs and atomic systems |
| **GRUB 2** | Legacy BIOS + UEFI, complex dual-boot; heavier |
| **U-Boot** | Boards / handhelds |
| **UKI as `BOOTX64.EFI`** | Smallest chain: firmware → signed PE (kernel+initrd+cmdline) — no separate bootloader stage |

systemd documents the overall sequence:

> Immediately after power-up, the system firmware will do minimal hardware initialization, and hand control over to a boot loader … This boot loader will then invoke an OS kernel from disk (or the network).

— [bootup(7)](https://www.freedesktop.org/software/systemd/man/latest/bootup.html) (systemd)

Arch’s overview of loaders + initramfs generation: [Arch boot process](https://wiki.archlinux.org/title/Arch_boot_process).

### 1.3 Kernel

Bootloader (or UEFI stub) jumps to the kernel with the initramfs already in memory (or built into the image). Graycart policy: **mainline first** (§2) — no ChromeOS/Asahi-scale fork.

### 1.4 Initramfs (early userspace)

The kernel extracts a **cpio** archive into rootfs and runs `/init` as PID 1 if present:

> All 2.6 Linux kernels contain a gzipped “cpio” format archive, which is extracted into rootfs when the kernel boots up. After extracting, the kernel checks to see if rootfs contains a file “init”, and if so it executes it as PID 1.

— Rob Landley, [Ramfs, rootfs and initramfs](https://docs.kernel.org/filesystems/ramfs-rootfs-initramfs.html) (kernel.org)

**Graycart MVP uses of initramfs:**

| Phase | Initramfs job |
|-------|----------------|
| Boots-to-shell | Optional: omit complex initramfs; root on `root=/dev/…` with drivers built-in, or tiny BusyBox `/init` |
| Runs graycart-gb | Modules for DRM, input, sound; mount real root; hand off to session |
| Installable image | Full early userspace: LUKS/LVM if used, `switch_root`, UKI rebuild hooks |

External initramfs (passed as “initrd”) is still a cpio.gz extracted into rootfs — preferred when packaging non-kernel bits separately from `vmlinuz`. Debug tip from the same kernel doc: `rdinit=/bin/sh`.

Generators: BusyBox scripted init, [dracut](https://github.com/dracutdevs/dracut), mkinitcpio, Buildroot’s built-in rootfs as initramfs, Yocto `INITRAMFS_IMAGE`.

### 1.5 Real init (PID 1 after switch_root)

After mounting the real root, early userspace `exec`s the real `/sbin/init` (or `systemd`). From there: mount `/proc` `/sys` `/dev`, start udev/mdev, bring up networking (optional), start the Graycart session or a login shell.

**MVP exit criterion (“boots to shell”):** getty or BusyBox `ash` on a serial/virtio console under QEMU — not a graphical target.

---

## 2. Kernel config strategy

### 2.1 Mainline vs vendor

| Source | Pros | Cons | Graycart stance |
|--------|------|------|-----------------|
| **kernel.org mainline / stable / longterm** | CVE backports, DRM/input maturity, reproducible config | May lack brand-new SoC bits | **Default** |
| **Distro kernel** (Fedora/Debian/Arch) | Already tested matrix; inherits update channel | Opaque config; hard to strip | Prefer when using Model 4 atomic base |
| **Vendor BSP fork** | Boots obscure boards quickly | Stale trees, out-of-tree modules, GPL source-offer pain | **Only for Model 3 appliance SKU**, with exit plan to mainline |

01 Model 5 (Asahi/ChromeOS-scale forks) remains a **non-goal**. If a handheld SKU needs vendor blobs, isolate them; do not make Graycart the long-term kernel maintainer.

### 2.2 LTS / stable policy

Upstream definitions ([Active kernel releases](https://www.kernel.org/releases.html); process overview [docs.kernel.org/process/2.Process.html](https://docs.kernel.org/process/2.Process.html)):

- **Stable:** short-lived `.y` backports after each mainline release.
- **Longterm:** selected releases with longer maintenance; projected EOL often starts ~2 years and may extend with industry interest — **not** a promise Graycart controls.

**Recommended Graycart policy:**

1. Pin one **longterm** (or the base distro’s supported kernel) per *released* image generation.
2. Track `.y` updates in CI; fail the farm if defconfig + Graycart fragment no longer builds.
3. Rebase to the next LTS **once per image generation** (not every mainline rc).
4. Document every out-of-tree patch in the SBOM / `PATCHES.md` with upstream status.

### 2.3 Config fragment strategy

Do not hand-maintain a full `.config` forever. Prefer:

```text
make defconfig          # or board defconfig / distro config
scripts/kconfig/merge_config.sh .config graycart.fragment
```

**`graycart.fragment` should enable (graphics/input path):**

- DRM/KMS (+ virtio-gpu for QEMU CI; Intel/AMD/Nouveau or SoC DRM for hardware)
- `CONFIG_INPUT`, evdev, HID, gamepad-relevant HID quirks
- Sound: ALSA (+ PipeWire/Pulse later in userspace)
- Filesystems needed for root (ext4, btrfs, erofs/squashfs for immutable images)
- UEFI stub / EFI vars if using UKI
- **Disable** unused bus drivers on appliance images to shrink attack surface and build time

Buildroot: `BR2_LINUX_KERNEL_CUSTOM_CONFIG_FILE` / fragment. Yocto: `SRC_URI` `.cfg` fragments. Atomic Model 4: inherit Fedora/Arch kernel; ship only Graycart userspace deltas.

---

## 3. Userspace: libc, coreutils, init

### 3.1 musl vs glibc

| | **musl** | **glibc** |
|---|----------|-----------|
| Size / static story | Smaller; clean static linking | Larger; dynamic NSS complexity |
| Ecosystem | Alpine, many Buildroot embeds | Desktop GPU stacks, Steam/Proton, most Rust `*-linux-gnu` targets |
| Rust host today | Possible via `x86_64-unknown-linux-musl` | **Matches shipped graycart-gb Linux artifact** (`x86_64-unknown-linux-gnu`) |

**Recommendation:**

- **PC branded image / gaming session:** **glibc** — Mesa, Vulkan ICDs, controller stacks, and the existing Graycart Linux release target assume it.
- **Tiny appliance / recovery initramfs:** **musl** (+ BusyBox) is fine for the *shell* layer; still prefer glibc if the same rootfs must run the full wgpu host without a second userspace.

Do not dual-libc one rootfs casually — pick one primary.

### 3.2 BusyBox vs coreutils (+ friends)

| | **BusyBox** | **coreutils + util-linux + …** |
|---|-------------|-------------------------------|
| Footprint | One multicall binary | Many packages |
| POSIX coverage | “Good enough” for embeds | Full desktop/admin |
| Debug UX | Limited | Familiar |

**MVP:** BusyBox ash + applet set through “boots to shell.”  
**Product image:** either keep BusyBox for recovery and add selective GNU tools, or switch to a real coreutils set once the image is installable. Avoid maintaining both full trees without need.

### 3.3 systemd vs other init

| Init | Fit |
|------|-----|
| **BusyBox `/sbin/init` + inittab** | Fastest path to shell; fine for P1 CI |
| **OpenRC** | Alpine-class; light; good with musl |
| **systemd** | Desktop/atomic default; user sessions, logind DRM master, socket activation; **required practical choice** if following Fedora bootc / many Wayland compositors’ assumptions |

**Recommendation:** BusyBox or OpenRC for early “boots to shell” experiments; **systemd** for any image that runs a Wayland/Gamescope session and expects logind seat management ([bootup(7)](https://www.freedesktop.org/software/systemd/man/latest/bootup.html)). Mixing “no systemd” with a full gaming desktop multiplies glue work.

---

## 4. Package / update story

Shipping an image without an update story is a CVE liability. Options:

| Model | Mechanism | Update UX | Graycart fit |
|-------|-----------|-----------|--------------|
| **None (monolithic reflash)** | Rebuild whole rootfs; `dd` / installer rewrite | Simple; no partial upgrades | **OK for MVP P1**; document as policy |
| **apk** (Alpine) | [apk-tools](https://wiki.alpinelinux.org/wiki/Alpine_Package_Keeper) | Fast, musl-native | Good for musl appliance; weak Steam/Proton story |
| **deb** (Debian/Ubuntu base) | apt | Huge archive | Heavy to brand; use only if basing on Debian/mkosi |
| **rpm / dnf** | Fedora/RHEL family | Same | Natural under bootc |
| **nix** | Declarative flake + `flake.lock` | Rebuild generations | Strong reproducibility; team must already Nix |
| **OSTree / rpm-ostree / bootc** | Atomic tree deploy + rollback | Best “appliance OS” UX | **Best match for 01 Model 4** |
| **Buildroot / Yocto image A/B** | Two partitions; bootloader flips slot | Common embedded pattern | **Best for Model 3 SKU** |

**Phased recommendation:**

1. **P1 boots-to-shell:** *none* — versioned image artifact in CI is the “package.”
2. **P2 runs graycart-gb:** still image-scoped; Graycart host is a pinned binary/crate build inside the image recipe (or Flatpak/sysext overlay).
3. **P3 installable:** pick **one** of: OSTree/bootc, Nix generations, or A/B image slots. Do **not** invent a new package format.

Buildroot has no first-class package feed for field upgrades — plan full-image updates ([Buildroot manual](https://buildroot.org/downloads/manual/manual.html)). Yocto can emit package feeds **or** image-based flows; prefer image/OSTree for a locked gaming appliance.

---

## 5. Graphics / input for Graycart emulators (wgpu / egui)

### 5.1 What the host needs

Today’s shipping app stack (graycart-gb): **winit + egui + wgpu** (Vulkan/GLES), **cpal** audio, **gilrs** gamepads — see [binary-size investigation](/cursor/stores/bc-57076e62-f812-47da-bf4c-f3bb0f3af797/docs/binary-size-investigation.md). Family design says the **linux host** must not put winit/wgpu/egui types in `*-core` APIs ([family README](/cursor/stores/bc-57076e62-f812-47da-bf4c-f3bb0f3af797/docs/graycart-family/README.md)).

On a branded OS, three presentation paths (increasing integration cost):

```text
A. Wayland compositor (labwc / sway / Gamescope) + winit Wayland backend + wgpu Vulkan
B. Xwayland fallback (compatibility only)
C. Direct DRM/KMS (kiosk) — compositor owns KMS, or experimental wgpu DRM surface
```

### 5.2 DRM / KMS (kernel display)

Kernel Mode Setting models framebuffers → planes → CRTC → connector ([DRM KMS](https://docs.kernel.org/gpu/drm-kms.html)). Atomic modesetting is the modern userspace contract.

Wayland’s architecture puts **KMS + input in the compositor**:

> In Wayland the compositor is the display server. We transfer the control of KMS and evdev to the compositor. The Wayland protocol lets the compositor send the input events directly to the clients and lets the client send the damage event directly to the compositor.

— [Wayland Architecture](https://wayland.freedesktop.org/docs/book/Architecture.html) (freedesktop)

**MVP recommendation:** Path **A** — small compositor (or Gamescope for gaming-focused images per 03’s product lens) + Graycart as a Wayland client. Reuses the existing winit path; Mesa provides Vulkan.

Path **C** (direct KMS) is research-grade for Graycart: wgpu gained unsafe Vulkan DRM surface helpers (`create_surface_from_drm`, `VK_EXT_acquire_drm_display` — see [wgpu#7212](https://github.com/gfx-rs/wgpu/pull/7212)), but egui/winit do not give a turnkey DRM-only app. Defer until a true kiosk appliance needs “no compositor.”

### 5.3 Buffer exchange

For compositor ↔ GPU clients, prefer **dma-buf** with negotiated format+modifier ([Exchanging pixel buffers](https://docs.kernel.org/userspace-api/dma-buf-alloc-exchange.html)). Graycart’s host already renders via wgpu; the OS must ship:

- Mesa + Vulkan ICD for the target GPU (incl. `virtio-gpu` / Venus or llvmpipe for CI)
- `libwayland`, xkbcommon, libxkbcommon
- libinput / udev for seats
- ALSA or PipeWire for cpal backends
- HID/gamepad udev rules (gilrs)

### 5.4 CI graphics reality

QEMU CI should not require a physical GPU:

| Tier | Stack | Proves |
|------|-------|--------|
| **Smoke** | headless core + PCM dump | ABI / frame advance (no wgpu) |
| **Host soft** | llvmpipe or SwiftShader + Wayland in Weston/headless | Window + egui path |
| **Host virtio** | virtio-gpu + Venus (when available) | Closer to real Vulkan |

Never block P1 (“boots to shell”) on Vulkan.

### 5.5 Input

- **Kernel:** evdev, HID, optionally `CONFIG_JOYSTICK_*` for legacy nodes.
- **Userspace:** libinput under Wayland; raw `/dev/input/event*` acceptable for kiosk.
- Product principle from 03: controller-first ≠ keyboard-hostile — both paths must work by P2.

---

## 6. Secure Boot, reproducible builds, SBOM

### 6.1 Secure Boot

Goal: firmware verifies a signature before running the bootloader/UKI.

**Practical Graycart approach:**

1. Build UKI (kernel + initramfs + cmdline) via dracut/`ukify`/efi-mkuki.
2. Sign with a Graycart **Machine Owner Key (MOK)** or vendor key enrolled in firmware (lab/dev: disable Secure Boot).
3. Prefer short chain: firmware → signed UKI (avoids signing GRUB modules).

Document enrollment for users; do not ship private keys in git. NVIDIA proprietary stacks complicate Secure Boot — same class of problem Bazzite documents; treat as P3 hardware-matrix debt.

### 6.2 Reproducible builds

Definition (Yocto): same inputs → same binaries across time, path, and host tools ([Yocto reproducible builds](https://docs.yoctoproject.org/test-manual/reproducible-builds.html)).

**Minimum Graycart bar:**

| Practice | Why |
|----------|-----|
| Pin Buildroot/Yocto/Nix/bootc base digests | Bit-for-bit recipe identity |
| `SOURCE_DATE_EPOCH` + toolchain path mapping | Kill timestamp/path noise |
| Vendor or mirror all tarballs (`make source` / sstate / Nix store) | Offline + audit |
| Record `defconfig` + fragment + package list | Rebuild without folklore |
| Compare two CI rebuilds (diffoscope) on release tags | Catch drift |

Buildroot: pin version + offline sources; weaker native provenance than Yocto. Yocto/Nix: stronger story. Model 4: reproduce *your Containerfile/flake*; trust upstream’s reproducibility for the base.

### 6.3 SBOM

Ship a machine-readable inventory with every public image:

- **SPDX** or **CycloneDX** listing kernel version, libc, Mesa, compositor, Graycart host, licenses.
- Yocto: license manifests / SPDX generation in-tree.
- Buildroot: parse legal-info / package lists; external tools can emit CycloneDX from manifests.
- Atomic/bootc: combine base image SBOM + Graycart delta packages.

SBOM is also how you honor GPL-2.0 **corresponding source** obligations for the kernel and GPL userspace ([kernel license rules](https://docs.kernel.org/process/license-rules.html)) — pair SBOM with a source tarball or written offer process (03 expands legal).

---

## 7. CI build farm shape

Target: prove the MVP ladder under [03’s P0–P3](./03-roadmap-and-risks.md) without a human babysitting QEMU.

```text
┌─────────────┐   recipes + lockfiles   ┌──────────────────┐
│ GitHub PR   │ ───────────────────────►│ Image builder    │
│ / main      │                         │ (Buildroot/Yocto │
└─────────────┘                         │  mkosi/bootc/Nix)│
                                        └────────┬─────────┘
                                                 │ artifacts:
                                                 │  - vmlinuz + initramfs / UKI
                                                 │  - rootfs / disk.img
                                                 │  - SBOM + sources stub
                                                 ▼
                                        ┌──────────────────┐
                                        │ QEMU smoke       │
                                        │  serial console  │
                                        │  → login / ash   │
                                        └────────┬─────────┘
                                                 │ (P2+)
                                                 ▼
                                        ┌──────────────────┐
                                        │ Host smoke       │
                                        │  graycart-gb     │
                                        │  fixture ROM     │
                                        │  N frames + PCM  │
                                        └──────────────────┘
```

### 7.1 Runner sizing

| Stage | Resources (order of magnitude) |
|-------|--------------------------------|
| BusyBox rootfs + mainline defconfig (x86_64) | ~2–8 vCPU, ~8–16 GiB RAM, tens of minutes cold |
| Full Mesa + Wayland + Rust host | Larger cache; expect multi-GB artifacts |
| Yocto sstate / Nix cache | **Persistent cache volume** mandatory or CI burns hours |
| QEMU smoke | Nested virt or TCG; serial-only is enough for P1 |

### 7.2 Pipeline jobs (suggested)

1. **`kernel-fragment`** — merge config, `make -j`, publish `vmlinuz` + `System.map` hash.
2. **`rootfs`** — Buildroot/mkosi/Nix/bootc build; upload `disk.img` + SBOM.
3. **`qemu-boot`** — expect string on serial (`login:` or custom banner) within timeout.
4. **`host-fixture`** (P2) — boot image, run Graycart host headless or Weston-headless against fixture; assert frame hash / PCM energy.
5. **`repro-spotcheck`** (release) — second rebuild, diffoscope critical paths.

### 7.3 Caching & secrets

- Cache: toolchain tarballs, cargo registry, ccache/sstate/Nix.
- Secrets: Secure Boot signing keys in CI only for *release* jobs; PR CI uses unsigned UKIs.
- Artifacts: retain last N successful images; tag releases with SBOM + source offer URL.

### 7.4 Split repos

Per family layout: **emulator CI ≠ image CI**. Host crate builds on ordinary Rust runners; image farm consumes versioned host binaries. Prevents Mesa rebuilds from blocking gb accuracy work.

---

## 8. Default stack picks (decision table)

| Layer | MVP (P1) | Product path (P2→P3) |
|-------|----------|----------------------|
| Firmware target | UEFI (QEMU OVMF) | UEFI; U-Boot only if appliance SKU |
| Bootloader | UKI or systemd-boot | UKI + Secure Boot (P3) |
| Kernel | Mainline longterm + `graycart.fragment` | Distro kernel if Model 4; else pinned LTS |
| Initramfs | Minimal / none | dracut or image-integrated |
| libc | glibc (or musl only if Alpine-appliance) | glibc for wgpu host |
| Base utils | BusyBox | BusyBox recovery + selective full tools |
| Init | BusyBox init | systemd (desktop/atomic) |
| Packages/updates | None (reimage) | OSTree/bootc **or** Nix **or** A/B images |
| Graphics | serial only | Wayland + Mesa Vulkan + winit/wgpu |
| Host | — | Family ABI host; egui/wgpu out of cores |
| SBOM / repro | Package list + hashes | SPDX/CycloneDX + dual-build check |
| CI | build + QEMU serial | + fixture host smoke + signing |

Cross-check: if Dave picks **01 Model 4**, swap “Buildroot rootfs” for “Containerfile/bootc or NixOS flake” but **keep** the boot chain, graphics, SBOM, and CI ladder.

---

## 9. Phased MVP

Aligned with user milestones and [03 P1–P3](./03-roadmap-and-risks.md).

### M1 — Boots to shell

**Exit:** QEMU boots Graycart-branded artifact to an interactive shell (BusyBox ash or getty) on serial/virtio-console; CI green.

| Work | Notes |
|------|-------|
| Pin kernel LTS + fragment | virtio, ext4, serial |
| Rootfs with BusyBox + `/sbin/init` | No graphics stack required |
| Disk image or UKI | Document exact QEMU command line |
| Artifact hashes + file list | Proto-SBOM |

### M2 — Runs graycart-gb

**Exit:** Same image (or thin successor) launches a Graycart host that loads a **fixture** GB ROM via family API / interim crate, renders frames (Wayland or headless GPU), plays PCM, accepts keyboard **and** a virtual/gamepad path.

| Work | Notes |
|------|-------|
| Mesa + Vulkan ICD (llvmpipe OK in CI) | Real GPU on one reference machine |
| Tiny Wayland compositor **or** Weston | Prefer winit Wayland path |
| Graycart host package pinned | No commercial ROMs in CI |
| ALSA/PipeWire + libinput/HID | Match desktop host deps |
| Library-first core gate | Blocked if GUI still fused into core |

### M3 — Installable image

**Exit:** Stranger-installable image with documented install, update/rollback (or explicit reimage policy), SBOM + source offer, Secure Boot story (even if “unsupported / MOK”), hardware matrix.

| Work | Notes |
|------|-------|
| Installer or first-boot wizard | ESP + root layout |
| Update channel | OSTree/bootc, Nix, or A/B |
| Branding + Linux Mark check | See 01 / 03 |
| Recovery | Second slot, UKI rollback, or documented reflash |

**Do not reorder** M3 before M2 — an installable wallpaper distro without a Graycart host wastes the brand.

---

## 10. Open choices for Dave (stack-specific)

- [ ] Image engine: **Buildroot** (simple appliance) vs **Yocto** (compliance/multi-SKU) vs **bootc/Nix/mkosi** (01 Model 4 default).
- [ ] libc: confirm **glibc** for the shipping host image.
- [ ] Session: **Gamescope** (gaming OS) vs **labwc/sway** (minimal) vs defer compositor until after host-on-stock-distro.
- [ ] Updates: **atomic/OSTree** vs **A/B reflash** vs **Nix generations**.
- [ ] Secure Boot: ship MOK enrollment docs at M3, or declare “unsigned / user-enrolled only.”

---

## 11. Sources (URL index)

| Topic | URL |
|-------|-----|
| systemd bootup(7) | https://www.freedesktop.org/software/systemd/man/latest/bootup.html |
| Kernel initramfs | https://docs.kernel.org/filesystems/ramfs-rootfs-initramfs.html |
| Arch boot process | https://wiki.archlinux.org/title/Arch_boot_process |
| Active kernel releases / LTS | https://www.kernel.org/releases.html |
| Kernel development process | https://docs.kernel.org/process/2.Process.html |
| Kernel license rules | https://docs.kernel.org/process/license-rules.html |
| DRM KMS | https://docs.kernel.org/gpu/drm-kms.html |
| dma-buf exchange | https://docs.kernel.org/userspace-api/dma-buf-alloc-exchange.html |
| Wayland architecture | https://wayland.freedesktop.org/docs/book/Architecture.html |
| Buildroot manual | https://buildroot.org/downloads/manual/manual.html |
| Yocto reproducible builds | https://docs.yoctoproject.org/test-manual/reproducible-builds.html |
| Yocto overview | https://docs.yoctoproject.org/ |
| LFS | https://www.linuxfromscratch.org/lfs/ |
| Alpine apk | https://wiki.alpinelinux.org/wiki/Alpine_Package_Keeper |
| OSTree project | https://ostreedev.github.io/ostree/ |
| rpm-ostree | https://coreos.github.io/rpm-ostree/ |
| U-Boot docs | https://docs.u-boot.org/ |
| wgpu DRM surface PR | https://github.com/gfx-rs/wgpu/pull/7212 |
| Graycart family API | ../graycart-family/README.md |
| Host binary size (egui/wgpu) | ../binary-size-investigation.md |
| Scope models (01) | ./01-scope-and-models.md |
| Roadmap / legal (03) | ./03-roadmap-and-risks.md |
