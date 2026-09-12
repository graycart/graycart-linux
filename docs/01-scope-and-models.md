<!--
Mirrored from Agent Store docs/graycart-linux/ — durable copy for this repo.
Cross-family links below point at the Graycart Project Agent Store paths.
No secrets / no binary blobs.
-->

# Graycart Linux — Scope and product models

**Audience:** Dave (product decision) + implementers.  
**Status:** Research / options — **no implementation in this pass**.  
**Family context:** [`docs/graycart-family/`](/cursor/stores/bc-57076e62-f812-47da-bf4c-f3bb0f3af797/docs/graycart-family/README.md) already names `graycart-linux` as a future **host-only** consumer of the shared core API (+ C ABI). This doc answers a different question: what “make our own Linux” could mean as a **product**, and which model fits Graycart.

**Internal status:** [`internal/graycart-linux/status-scope.md`](/cursor/stores/bc-57076e62-f812-47da-bf4c-f3bb0f3af797/internal/graycart-linux/status-scope.md)

---

## 0. Two meanings of “graycart-linux”

| Meaning | What it is | Already decided? |
|---------|------------|------------------|
| **A. Linux host app** | Emulator shell on Linux (DRM/KMS or Wayland/X, audio, FS, consent UX) that loads `graycart-*-core` via Rust trait / `gc_*` C ABI | **Yes in family design** — [family README](/cursor/stores/bc-57076e62-f812-47da-bf4c-f3bb0f3af797/docs/graycart-family/README.md), [01 core API](/cursor/stores/bc-57076e62-f812-47da-bf4c-f3bb0f3af797/docs/graycart-family/01-core-api.md), [02 repo layout](/cursor/stores/bc-57076e62-f812-47da-bf4c-f3bb0f3af797/docs/graycart-family/02-repo-layout.md) |
| **B. Branded OS / distro** | An installable system image people call “Graycart Linux” | **Open** — this doc |

**Implication:** Meaning A is a prerequisite for any useful Meaning B. Shipping B without library-first cores and a host wastes years on packaging before there is anything Graycart-specific to run.

**Product trajectory reminder (family):** desktop apps today (`graycart-gb` interim → `graycart-gba` supersede); NES/SNES/N64 placeholders; linux host later. Prefer reusing gb for 8-bit; force library-first cores so linux can call them.

---

## 1. Decision frame

Ask for each model:

1. Does it make **emulator cores better**, or only the **install story**?
2. Can a Graycart-sized team keep **security updates** for years?
3. What **hardware** do we actually target (PC/laptop? handheld appliance? Apple Silicon?)?
4. Does branding as `*-linux` create **trademark / compliance** work we are ready for?

Effort figures below assume a **small team (≈1–3 full-time equivalent)** unless noted. “Years-to-usable” means a **stranger can install and run Graycart emulators with acceptable graphics/audio**, not “kernel boots.”

---

## 2. Model comparison (summary)

| # | Model | Team (sustain) | Years to usable* | Effort | Risk | Fit for Graycart family |
|---|--------|----------------|------------------|--------|------|-------------------------|
| 1 | From-scratch hobby OS (not Linux) | 1–many forever | 5–15+ (desktop-class: never competitive) | Extreme | Extreme | **Poor** — blocks GPU/drivers/emulators |
| 2 | LFS / custom distro on mainline kernel | 2–5+ for a real distro | 0.1 first boot; **2–5+** as a maintained product | Very high | High | **Weak** as product; OK as learning |
| 3 | Yocto / Buildroot embedded image | 1–3 embedded eng | 0.25–1 first appliance; **1–3** production SKU | High | Medium–high | **Good only for a fixed appliance SKU** |
| 4 | Immutable/atomic branded image (OSTree / bootc / nix / sysext) | 1–2 on top of upstream | **0.25–1** branded ISO; continuous rebase | Medium | Medium | **Best default for “our Linux” brand** |
| 5 | ChromeOS / Asahi-style downstream kernel + userspace | 5–50+ (Asahi-scale); Google-scale for ChromeOS | 2–10+ | Extreme | Extreme | **Reject** unless hardware + funded kernel team |

\*Usable = playable Graycart host + cores on target hardware, with a path to security updates.

---

## 3. Model 1 — From-scratch hobby OS (not Linux)

### What it is

Write your own kernel (or microkernel), drivers, VFS, networking, graphics stack, and userspace. Precedents: [ToaruOS](https://github.com/klange/toaruos) (since 2011; first full release ~2017; 2.0 in 2021), countless osdev toys. These are **not** Linux.

### Effort / timeline

| Milestone | Rough cost |
|-----------|------------|
| Boots + shell in VM | Months–years (1 person) |
| Self-hosting toolchain | Multi-year |
| GPU acceleration + audio + gamepads good enough for emulators | Decade-class / rarely achieved |
| Competitive with a mid-tier Linux distro for “just play ROMs” | **Not a realistic product goal** |

### License landmines

- You own your license (BSD/MIT/GPL/etc.).
- You **lose** the Linux syscall exception / ecosystem: no glibc-on-Linux, no Mesa-on-DRM as upstream intends, no “drop in RetroArch.”
- Importing GPL components into a proprietary or differently licensed base needs careful boundary design (see hobby OS `contrib/` patterns).

### Hardware targets

Typically QEMU/x86_64 first; real hardware is a second career. Apple Silicon / handheld SoCs are out of scope without a driver army.

### Family fit

**Does not serve** gb/gba/nes hosts or a future linux host. Emulator cores assume a mature OS (threads, timers, GL/Vulkan or at least software blit + PCM). Model 1 **competes with** Graycart’s actual product (accurate consoles), not enables it.

### Verdict

**Learning / art project only.** Explicit non-goal for Graycart product.

**Sources:** [ToaruOS README](https://github.com/klange/toaruos); osdev community norms.

---

## 4. Model 2 — Linux From Scratch / custom distro on mainline kernel

### What it is

Build a userspace from source on an upstream (or lightly patched) Linux kernel: [LFS](https://www.linuxfromscratch.org/lfs/) + [BLFS](https://www.linuxfromscratch.org/blfs/), or a hand-rolled package set with your own installer and update story. Kernel stays “mainline-ish”; **you** become the distribution.

### Effort / timeline

| Milestone | Rough cost |
|-----------|------------|
| Follow LFS book → bootable minimal system | ~20–80 person-hours for a first build ([example write-up](https://moi.vonos.net/linux/linux-from-scratch/)) |
| Desktop + GPU + PipeWire + gamepads + Flatpak | Months |
| Package manager, reproducible builds, installer, signed updates | 1–3+ years |
| **Ongoing CVE response** across the whole tree | Permanent; small teams often burn out ([Linuxiac on small distros](https://linuxiac.com/when-passion-is-not-enough-small-linux-projects-big-problems/)) |

A first LFS system is a **tutorial**, not a product. Product cost is **maintenance**, not the weekend compile.

### License landmines

- Kernel: **GPL-2.0-only** (+ syscall note for UAPI) — [kernel license rules](https://docs.kernel.org/process/license-rules.html). Redistribute binaries → provide corresponding source or a valid written offer.
- Userspace mix: GPL/LGPL (glibc, GCC runtime exceptions), permissive libs — need an SBOM and source offer process.
- Name **“Graycart Linux”**: “Linux” as a **trademark element** in a product brand generally requires a **Linux Foundation sublicense** ([Linux Mark](https://www.linuxfoundation.org/legal/the-linux-mark)). Fair use for descriptive text ≠ branding an OS. Attribution rules still apply.

### Hardware targets

Anything mainline supports (x86_64 laptops/desktops first). You inherit mainline quality; you do **not** inherit Fedora/Debian’s QA matrix unless you rebuild it.

### Family fit

Gives a place to install the host app, but **duplicates** work Ubuntu/Fedora already do. Does not accelerate core accuracy. Conflicts with “small team, library-first emulators.”

### Verdict

**Useful as training** for whoever will maintain images later. **Poor default** as the shipping product model.

**Sources:** [LFS homepage](https://www.linuxfromscratch.org/lfs/); [kernel licensing](https://docs.kernel.org/process/license-rules.html); [Linux trademark sublicense](https://www.linuxfoundation.org/legal/the-linux-mark); [small-distro failure modes](https://linuxiac.com/when-passion-is-not-enough-small-linux-projects-big-problems/).

---

## 5. Model 3 — Yocto / Buildroot embedded distro

### What it is

Generate a **purpose-built image** for known boards: [Yocto Project](https://www.yoctoproject.org/) / OpenEmbedded (layers, recipes, sstate) or [Buildroot](https://buildroot.org/) (Kconfig + make, whole-image rebuilds). Common for appliances, kiosks, handhelds.

### Effort / timeline

| | Buildroot | Yocto |
|---|-----------|--------|
| First bootable image | Days–weeks | Weeks–months (learning + cold build) |
| Team fit | Solo / small, one SKU | 1–3+ when many boards / long life |
| Ongoing | Full-image rebuilds; simpler mental model | Layer discipline, sstate/CI (often 100+ GiB workdirs) |
| When it wins | Single appliance, image A/B updates | Multi-SKU platform, SDKs, compliance artifacts |

Industry comparisons: [Ezurio Yocto vs Buildroot](https://www.ezurio.com/resources/blog/yocto-vs-buildroot-comparing-the-two-platforms); [embedded-sbc guide](https://embedded-sbc.com/posts/yocto-vs-buildroot-production-embedded-linux/); cautionary “you probably don’t need Yocto” for Debian-class needs ([sigma-star](https://sigma-star.at/blog/2026/05/you-probably-dont-need-yocto-and-thats-fine/)).

### License landmines

Same kernel GPL-2.0 source-offer obligations; Yocto can emit license manifests (a reason orgs pick it). Vendor BSPs often ship **out-of-tree** kernel modules — track GPL compatibility and taint. Trademark: same “Linux” brand rules if the product name includes Linux.

### Hardware targets

**Fixed set:** e.g. one handheld SoC, one NUC-like kiosk, one RPi-class board. Bad fit for “any gamer PC.”

### Family fit

**Strong if** Graycart ships a **console-like appliance** (boot → Graycart UI, no general desktop). Cores still come from `*-core` + ABI; the image is just the delivery vehicle. **Weak if** the goal is “Graycart Linux for everyone’s laptop.”

### Verdict

**Contingent path** — activate when there is a real hardware SKU and update channel. Not the default for a software-first emulator family.

**Sources:** Yocto/Buildroot docs and comparisons linked above; [Yocto overview](https://docs.yoctoproject.org/).

---

## 6. Model 4 — Immutable / atomic branded “graycart-linux”

### What it is

Keep a **major upstream** (usually Fedora Atomic / CentOS bootc, or NixOS, or Debian+mkosi) and ship a **branded image**:

- **bootc / OSTree / rpm-ostree** — atomic updates + rollback; OCI container as the OS ([Fedora Magazine bootc desktop](https://fedoramagazine.org/building-your-own-atomic-bootc-desktop/); [rpm-ostree](https://coreos.github.io/rpm-ostree/); Fedora [OstreeNativeContainer / bootc](https://www.fedoraproject.org/wiki/Changes/OstreeNativeContainerStable))
- **Universal Blue / Bazzite-style** — custom Atomic image via Containerfile + GitHub Actions ([Bazzite](https://github.com/ublue-os/bazzite), [docs](https://docs.bazzite.gg/General/FAQ/)) — gaming-oriented precedent closest to “emulator OS”
- **NixOS flake** — declarative branded config / ISO; you maintain modules + `flake.lock`, not a full fork ([NixOS flakes](https://nixos.wiki/wiki/Flakes))
- **systemd-sysext** — thin overlays on a trusted base for Graycart tools without forking the world ([mkosi sysext](https://github.com/systemd/mkosi/blob/main/docs/sysext.md), [UAPI extension images](https://uapi-group.org/specifications/specs/extension_image/))

Graycart-specific content: host binary, branded first-boot, curated Flatpaks/containers for optional tools, maybe a kiosk session. **Kernel and 99% of userspace stay upstream.**

### Effort / timeline

| Milestone | Rough cost (1 engineer familiar with containers) |
|-----------|--------------------------------------------------|
| Rebase Universal Blue / fedora-bootc + install Graycart host | Weeks |
| Signed images, auto-update, rollback UX, CI | 2–6 months |
| “Product” polish (ISO, docs, support matrix) | ~0.5–1 year |
| Ongoing | Track upstream rebases + own packages; **do not** carry a large kernel fork |

Much lower than Models 1–2–5 because **CVE firefighting stays with Fedora/NixOS/Debian**.

### License landmines

- Still redistribute GPL kernel → source/offer compliance (often inherited via upstream image + your deltas documented).
- **“Graycart Linux” trademark:** apply for Linux Mark sublicense if “Linux” is part of the product trademark ([LF Linux Mark](https://www.linuxfoundation.org/legal/the-linux-mark)). Alternative brand: **Graycart OS** / **Graycart Atomic** (describes “based on Linux” in prose only) — lower trademark friction; confirm with counsel.
- Flatpak/firmware blobs: keep Graycart’s **no BIOS/ROM in git** posture; user-supplied firmware stays user-supplied.
- Mixing proprietary GPU drivers (NVIDIA) in the base image increases compliance and Secure Boot key complexity (Bazzite documents this class of problem).

### Hardware targets

- **Primary:** x86_64 PCs/laptops (and whatever Atomic/NixOS already support well).
- **Secondary:** handheld PCs if following Bazzite-like images.
- **Not automatic:** Apple Silicon (needs Asahi-class stack — Model 5).

### Family fit

**Best match** for a branded OS that still respects family architecture:

```text
graycart-linux (host app)  →  abi + gb-core + gba-core + …
        ↑
branded atomic image (optional product skin)
        ↑
Fedora Atomic / bootc / NixOS / Debian base
```

The **repo** `graycart-linux` in [02 repo layout](/cursor/stores/bc-57076e62-f812-47da-bf4c-f3bb0f3af797/docs/graycart-family/02-repo-layout.md) remains **host-only**; the OS image can live in a sibling repo (e.g. `graycart-os` / image Containerfile) so emulator CI is not yoked to image builds.

### Verdict

**Default path for Meaning B** (branded OS), **after** Meaning A (host) exists on stock distros.

**Sources:** bootc/OSTree/Bazzite/Nix/sysext links above.

---

## 7. Model 5 — ChromeOS / Asahi-style downstream kernel + userspace

### What it is

Carry a **large downstream kernel** and matching userspace forks (Mesa, bootloader, Flatpak runtimes, etc.), either for a locked hardware platform (ChromeOS) or unsupported SoCs (Asahi on Apple Silicon).

### Effort / timeline (evidence, not vibes)

**ChromeOS (Google):** yearly LTS pick, multiple active kernel branches, deliberate upstream-first policy so uprevs stay possible — still a full platform org ([ELC talk PDF](https://static.sched.com/hosted_files/ossna19/9c/ELC19_ChromeOSAndUpstream.pdf); [LWN summary](https://lwn.net/Articles/798147/)).

**Asahi Linux:** downstream tree historically **1000+ patches** / ~90k LOC not upstream; rebases described as **hours nearly weekly**; also downstream Mesa, virglrenderer, Flatpak extensions; hardware spend cited **>$10k/year** on Macs alone ([Passing the torch](https://asahilinux.org/2025/02/passing-the-torch/); [Progress report 6.14](https://asahilinux.org/2025/03/progress-report-6-14/)). Governance moved to a multi-maintainer model because **one person cannot sustain it**.

### License landmines

- Every downstream patch is still GPL-2.0 — you must ship sources for what you distribute.
- Long-lived out-of-tree GPU stacks force **userspace forks**; containers/Flatpaks break until UAPI is upstream (Asahi’s explicit pain).
- Trademark + branding as above.

### Hardware targets

Only makes sense for **hardware Linux does not yet support well** (or a ChromeOS-like locked device tree). For Graycart’s emulator mission on PCs, this is pure cost.

### Family fit

**Misaligned.** Graycart’s scarce skill should go to **console accuracy** (gb/gba/nes…), not DRM driver rebases. A linux **host** can run on Fedora Asahi Remix or stock distros without Graycart owning the fork.

### Verdict

**Explicit non-goal** unless Graycart becomes a hardware company with a funded platform team.

**Sources:** Asahi and ChromeOS links above.

---

## 8. Cross-cutting: licenses & naming

| Topic | Action if shipping any Linux-based image |
|-------|------------------------------------------|
| Kernel GPL-2.0 | Publish corresponding sources / offer; track patches |
| Mixed userspace | SBOM; respect LGPL linking; document proprietary blobs |
| “Linux” in product name | Budget LF [sublicense](https://www.linuxfoundation.org/legal/the-linux-mark) + attribution; or avoid “Linux” in the trademark |
| Firmware / ROMs | Unchanged Graycart policy: **never** vendor BIOS or commercial ROMs |
| Graycart core license | Keep cores’ licenses compatible with host distribution; C ABI does not launder GPL |

---

## 9. How this maps to the Graycart family

From [family README](/cursor/stores/bc-57076e62-f812-47da-bf4c-f3bb0f3af797/docs/graycart-family/README.md) / [02](/cursor/stores/bc-57076e62-f812-47da-bf4c-f3bb0f3af797/docs/graycart-family/02-repo-layout.md):

| Artifact | Role |
|----------|------|
| `graycart-gb` / `graycart-gba` / `nes`… | Machines (`*-core`) + desktop hosts |
| `graycart-abi` | Shared trait + C header |
| `graycart-linux` | **Host only** — window/DRM, devices, paths, chrome |
| Branded OS image (optional) | Ships the host + OS policy; **does not** contain machine logic |

Phased product sequence (recommended):

1. **Accept family API** and extract `*-core` (already gated on Dave review).
2. **Ship `graycart-linux` host** on stock Fedora/Ubuntu/Arch (Meaning A) — proves ABI, audio, input.
3. **Optionally** wrap Meaning A in Model 4 atomic image (Meaning B) for one-click “Graycart machine.”
4. **Only if hardware SKU:** Model 3 image for that board.
5. **Never (default):** Models 1, 2-as-product, 5.

---

## 10. Recommendation (default path)

**Default:** Treat “make our own Linux” as **Model 4 — an immutable/atomic branded image** (Fedora Atomic / bootc or Universal Blue-style; Nix flake is an acceptable alternate if the team already lives in Nix), **after** the family **host app** exists on stock distros.

**Why:** Lowest effort/risk that still yields a coherent “Graycart Linux” *product skin*; inherits upstream security and GPU stacks; keeps emulator work in cores; matches how successful niche OSes (e.g. Bazzite) actually ship.

**Near-term non-OS milestone (do first):** implement Meaning A per family docs — `graycart-linux` host depending on abi + cores — **without** building a distro.

---

## 11. Explicit non-goals

- Writing a **from-scratch kernel/OS** (Model 1) as a Graycart product.
- Operating a **full independent package distro** à la Debian/Fedora (Model 2 as end state).
- Carrying a **ChromeOS- or Asahi-scale downstream kernel** (Model 5) without a funded hardware mandate.
- Putting **machine/core logic** inside an OS image repo (violates core ≠ host).
- Making the **umbrella** a Cargo workspace or distro monorepo.
- Vendoring **BIOS / commercial ROMs** into any image.
- Blocking **GBA P0–P9** or gb extract on OS-image work.
- Targeting **Apple Silicon as first-class Graycart-maintained** platform via our own kernel fork.
- Claiming “Graycart Linux” trademark use without checking **Linux Mark** requirements.

---

## 12. Open choices for Dave

- [ ] Confirm **Model 4** as the OS end-state (vs “host app only, never ship an ISO”).
- [ ] Brand string: **Graycart Linux** (sublicense) vs **Graycart OS** / **Graycart Atomic** (Linux descriptive only).
- [ ] Base: **Fedora bootc / Universal Blue** vs **NixOS flake** vs **Debian + mkosi/sysext**.
- [ ] Whether a future **handheld appliance** should open Model 3 in parallel.
- [ ] Timeline: host-on-stock-distro gate before any image CI.

---

## 13. Sources (URL index)

| Topic | URL |
|-------|-----|
| LFS | https://www.linuxfromscratch.org/lfs/ |
| LFS effort anecdote | https://moi.vonos.net/linux/linux-from-scratch/ |
| Small distro sustainability | https://linuxiac.com/when-passion-is-not-enough-small-linux-projects-big-problems/ |
| Kernel license rules | https://docs.kernel.org/process/license-rules.html |
| Linux trademark / sublicense | https://www.linuxfoundation.org/legal/the-linux-mark |
| Yocto vs Buildroot (Ezurio) | https://www.ezurio.com/resources/blog/yocto-vs-buildroot-comparing-the-two-platforms |
| Yocto vs Buildroot (production) | https://embedded-sbc.com/posts/yocto-vs-buildroot-production-embedded-linux/ |
| “You probably don’t need Yocto” | https://sigma-star.at/blog/2026/05/you-probably-dont-need-yocto-and-thats-fine/ |
| rpm-ostree | https://coreos.github.io/rpm-ostree/ |
| Fedora OSTree→bootc change | https://www.fedoraproject.org/wiki/Changes/OstreeNativeContainerStable |
| DIY bootc desktop | https://fedoramagazine.org/building-your-own-atomic-bootc-desktop/ |
| Bazzite (Atomic gaming image) | https://github.com/ublue-os/bazzite |
| Bazzite FAQ | https://docs.bazzite.gg/General/FAQ/ |
| systemd mkosi sysext | https://github.com/systemd/mkosi/blob/main/docs/sysext.md |
| Extension image spec | https://uapi-group.org/specifications/specs/extension_image/ |
| NixOS flakes | https://nixos.wiki/wiki/Flakes |
| Asahi governance / patch stack | https://asahilinux.org/2025/02/passing-the-torch/ |
| Asahi maintenance cost | https://asahilinux.org/2025/03/progress-report-6-14/ |
| ChromeOS upstream kernel | https://lwn.net/Articles/798147/ |
| ChromeOS ELC slides | https://static.sched.com/hosted_files/ossna19/9c/ELC19_ChromeOSAndUpstream.pdf |
| ToaruOS (hobby OS precedent) | https://github.com/klange/toaruos |
| Graycart family API | ../graycart-family/README.md |
