<!--
Mirrored from Agent Store docs/graycart-linux/ — durable copy for this repo.
Cross-family links below point at the Graycart Project Agent Store paths.
No secrets / no binary blobs.
-->

# graycart-linux — Roadmap, cost, legal, and traps

**Audience:** Dave (go / no-go before code) + implementers.  
**Repo today:** [graycart/graycart-linux](https://github.com/graycart/graycart-linux) — placeholder + vision (`README`, `AGENTS.md`); not an invitation to build a distro yet.  
**Sibling research:** [01 — Scope & models](./01-scope-and-models.md) · [02 — Technical stack](./02-technical-stack.md) (cross-link when present).  
**Family contract:** [`docs/graycart-family/`](/cursor/stores/bc-57076e62-f812-47da-bf4c-f3bb0f3af797/docs/graycart-family/README.md) · [01-core-api](/cursor/stores/bc-57076e62-f812-47da-bf4c-f3bb0f3af797/docs/graycart-family/01-core-api.md) · [02-repo-layout](/cursor/stores/bc-57076e62-f812-47da-bf4c-f3bb0f3af797/docs/graycart-family/02-repo-layout.md).  
**Product trajectory:** [`product-decision-supersede-gb.md`](/cursor/stores/bc-57076e62-f812-47da-bf4c-f3bb0f3af797/internal/graycart-gba/product-decision-supersede-gb.md).

**Disclaimer:** Legal sections summarize public GPL-2.0 / Linux Mark guidance for planning. Not legal advice; confirm with counsel before shipping images or trademarked branding.

---

## 0. What this project is (and is not)

Two stories already exist; reconcile them before coding:

| Lens | Meaning |
|------|---------|
| **Product (repo README)** | Opinionated **gaming-focused Linux system** — boot-to-play, Steam/Proton/Gamescope, controller-first, recovery — currently **leans Omarchy-inspired Arch**, not a from-scratch kernel tree. |
| **Family API (store docs)** | **Host** for Graycart cores via the family trait / C ABI — no machine logic in `graycart-linux`; peers with desktop bins above the lib line. |
| **Scope research ([01](./01-scope-and-models.md))** | Prefer **Meaning A (host app) first**, then optional **Model 4** immutable/atomic branded image (Fedora bootc / Universal Blue–style; Nix alternate). Open choice vs README’s Arch lean. |

**Correct synthesis:** graycart-linux is the **OS + shell host**. Emulator accuracy stays in `*-core` crates. The distro (if any) is differentiated by session, packaging, recovery, and integration — not by forking cores. **Do not start image work until Dave picks base** (Arch/Omarchy-inspired vs bootc/UB vs host-only).

**Non-goals (until Dave accepts this pack + 01/02):** shipping ISOs, forking Omarchy/Arch wholesale, inventing a custom kernel, or blocking GBA P0–P9 on linux work.

---

## 1. Phased roadmap

Aligns with research → CI proof → Graycart host → installable. Maps loosely onto the repo README’s Phase 0–4; **this numbering is the implementation gate**.

### P0 — Docs / bootstrap

**Exit:** Dave-accepted research pack; repo remains docs-first; no public “download Graycart Linux” claim.

| Work | Done when |
|------|-----------|
| Scope models (appliance vs desktop vs overlay) | [01](./01-scope-and-models.md) accepted |
| Boot / kernel / userspace / session / packaging | [02-technical-stack.md](./02-technical-stack.md) accepted |
| This doc (cost, GPL, trademark, traps, checklist) | Accepted |
| Mirror research into `graycart-linux` | Scaffold agent / PR; LICENSE chosen for *repo* configs |
| Family prerequisites called out | `graycart-abi` + headless core API path named (even if deferred) |

**Solo+agents effort:** ~several days of parallel research + one review pass. Cheap relative to everything after.

### P1 — Bootable CI image

**Exit:** Reproducible artifact that boots under QEMU (and optionally one reference bare-metal class) in GitHub Actions; not a product ISO.

| Work | Done when |
|------|-----------|
| Image build recipe (mkosi / archiso / Packer / custom) pinned and documented | Clean rebuild from empty runner |
| Kernel + initramfs + rootfs composition recorded | SBOM or package list + versions published with artifact |
| CI: build → boot → smoke (`systemd` reaches multi-user or graphical target) | Green on `main`; flaky boots treated as bugs |
| Source accompanying binaries (or written offer process stub) | License compliance checklist for the artifact |
| Rollback story sketched | Even if “reflash whole image” only |

**Solo+agents effort:** ~2–6 weeks calendar-equivalent of agent loops + Dave triage once the **base image model** is chosen ([01](./01-scope-and-models.md) Model 4 or Arch overlay — not Yocto greenfield). First green boot is the cliff; polishing reproducibility and CI minutes dominates. If Dave chooses **host-on-stock-distro only**, skip full image CI and treat P1 as “packaged host + smoke on Ubuntu/Fedora runners.”

### P2 — Graycart host

**Exit:** On the CI image (or a thin sibling rootfs), a Graycart host process loads cores through the **family API**, renders framebuffer, plays PCM, accepts input — without egui-from-desktop dragged into the OS.

| Work | Done when |
|------|-----------|
| Depends on library-first cores | `graycart-gb-core` (or interim whole-crate) + `graycart-gba-core` callable headless |
| Host chrome (Wayland / DRM-KMS / Gamescope session — per 02) | Boot → launch Graycart → load homebrew/fixture → frame+audio |
| No core forks in this repo | Git deps / packages only ([family 02 §7](/cursor/stores/bc-57076e62-f812-47da-bf4c-f3bb0f3af797/docs/graycart-family/02-repo-layout.md#7-what-not-to-do)) |
| ROM-free default CI | Fixtures only; commercial ROMs never in CI |
| Controller + keyboard paths both work | Matches product principle: controller-first ≠ keyboard-hostile |

**Solo+agents effort:** ~1–3 months after P1 **and** after family API + gb extract decisions. Blocked hard if cores remain GUI-coupled.

**Parallelism:** Native GBA P0–P9 continues independently; linux P2 should not wait for GBA P12 supersede cutover — gb-core alone can prove the host seam earlier.

### P3 — Installable

**Exit:** Public installable image/channel with update + recovery story, hardware acceptance matrix, and documented support boundaries.

| Work | Done when |
|------|-----------|
| Installer or guided first-boot | Documented clean install path |
| Update channel + rollback | User can recover from a bad upgrade without reinstalling from scratch *or* documented reimage is explicit product policy |
| Hardware matrix (GPU / handheld / VRR / Bluetooth / audio) | Pass/fail published; “unsupported” is allowed |
| Trademark + GPL compliance for public distribution | See §§3–4 |
| Security maintenance process | CVE triage cadence; who owns kernel/Mesa/Steam stack |
| Graycart identity (boot → session → emulator tooling) | Cohesive without becoming a closed appliance |

**Solo+agents effort:** **multi-quarter** for a credible gaming OS; ongoing thereafter. Agents accelerate packaging and docs; they do not remove human ownership of security response, hardware triage, or trademark/GPL process.

### Phase map (repo README ↔ this gate)

| This doc | Repo README (approx.) |
|----------|------------------------|
| P0 | Phase 0 Research (+ early Bootstrap docs) |
| P1 | Phase 1 Bootstrap (reproducible image) |
| P2 | Phase 3 Graycart experience (host integration); gaming platform bits may start in P1/P2 |
| P3 | Phase 2 polish + Phase 4 Recovery and release |

Do not reorder to “pretty ISO first, host API later” — that produces a wallpaper distro with no family value.

---

## 2. Rough effort (solo + agents)

Assumptions: one human owner (Dave), many agents, no full-time distro engineer, **no Yocto-from-scratch**, base TBD between README (Arch/Omarchy lean) and [01](./01-scope-and-models.md) (atomic bootc/UB default), Graycart cores progressing on their own roadmap.

| Phase | Effort shape | Dominant risk |
|-------|--------------|---------------|
| **P0** | Days–2 weeks | Scope thrash (host vs full distro) |
| **P1** | ~2–6 weeks to first reliable CI boot; +ongoing CI cost | Image drift, flaky QEMU, license packaging |
| **P2** | ~1–3 months after API/core extract | Blocked on family ABI / GUI entanglement |
| **P3** | Quarters to v1; **permanent** maintainer tax | Security + hardware + updates |

**Order-of-magnitude total to “friends can install and play Graycart on it”:** think **half a year+** of wall-clock with agents helping, not a weekend ISO. A SteamOS/Bazzite-class gaming OS is years of product work; Graycart should **not** aim to out-compete SteamOS — aim for a coherent Graycart host OS with good gaming defaults.

**Cost beyond engineering time:** CI minutes/storage for images, occasional test hardware, potential counsel hours for trademark/GPL distribution, and **ongoing** CVE response time (often larger than feature work after P3).

---

## 3. GPL / copyleft obligations (kernel & modules)

Linux kernel is **GPL-2.0-only** with the syscall note exception ([kernel license rules](https://docs.kernel.org/process/license-rules.html)). Shipping a bootable image that includes kernel (and typically many userland GPL components) triggers distributor duties.

### When you distribute binaries

Under GPL-2.0 §3, for object/executable forms you must **either**:

1. **Accompany** with complete corresponding machine-readable source; **or**
2. Provide a **written offer**, valid **≥ 3 years**, to give **any third party** complete corresponding source for no more than physical distribution cost; **or**
3. (Narrow) pass along an offer you received — noncommercial only.

“Complete corresponding source” includes modules, interface files, and **scripts used to control compilation and installation** — prefer shipping the exact build recipes (mkosi/archiso configs, PKGBUILDs, kernel config) someone skilled can use to regenerate what you shipped.

**Do not** point recipients only at a third-party archive (e.g. “get it from kernel.org / Arch”) and call that done for *your* binary image — distributor responsibility remains yours ([Debian CD vendor guidance](https://www.debian.org/CD/vendors/legal.en.html) is the classic cautionary note).

### Modules

- In-tree / GPL-compatible modules: treat source like the rest of the kernel build.
- Out-of-tree modules: license them correctly; `MODULE_LICENSE()` is for the loader, **not** a substitute for SPDX/source license ([kernel docs](https://docs.kernel.org/process/license-rules.html)).
- Proprietary modules taint the kernel and must not claim GPL; if you ship any, understand export-symbol and compliance constraints — **default posture: avoid proprietary modules in Graycart images**.

### Graycart cores vs kernel

MIT/Apache-style emulator cores linked as **userspace** programs that talk to Linux via normal syscalls are **not** automatically GPL’d by the kernel’s syscall exception. Separate copyleft still applies to any GPL **userland** you ship (e.g. GPL tools in the image). Keep:

- Core crates → their own licenses (no accidental GPL infection via bad linking of GPL libs into MIT cores).
- Image recipes → SPDX / NOTICE / offer URL per release.

### Practical compliance checklist (P1+)

- [ ] Publish image **and** matching source bundle / git tag of recipes.
- [ ] Document written-offer URL and retention (≥ 3 years from last distribution of that binary).
- [ ] Generate SBOM / package list per release.
- [ ] No “mystery blobs” without license classification (firmware needs its own story).
- [ ] CI artifact retention policy matches offer policy.

---

## 4. Trademark (“Linux”) constraints

“Linux®” is a trademark owned by Linus Torvalds; sublicensing is administered by the Linux Foundation ([Linux Mark](https://www.linuxfoundation.org/legal/the-linux-mark)).

| Use | Likely posture |
|-----|----------------|
| Repo / project name **`graycart-linux`**, docs saying “Graycart Linux” as a **product brand** for a Linux-based OS | Treat as **trademark use** → apply for the **free** LF sublicense before marketing installable images ([request](https://www.linuxfoundation.org/legal/request-a-sublicense)). |
| Factual “runs on Linux”, “Linux-based”, “uses the Linux kernel” | Often **fair use** / descriptive; still attribute ownership; do not imply LF/Torvalds endorsement. |
| Claiming certification, official status, or confusing similarity to other marks | Forbidden under sublicense terms. |

**Required if sublicensed:** attribution / legend on goods and literature (see [sublicense agreement](https://www.linuxfoundation.org/legal/sublicense-agreement)); first standalone “Linux” with ® as specified.

**Omarchy / Arch / Steam / etc.:** their marks are separate. “Inspired by Omarchy” ≠ fork their branding. Do not ship something that looks like an official Omarchy/Arch spin without their rules.

**Action before P3 public marketing:** decide brand string (`Graycart Linux` vs `Graycart OS` vs host-only name), file LF sublicense if “Linux” stays in the mark, add attribution to ISO/docs.

---

## 5. Security maintenance burden

Shipping an OS means **owning** vulnerability response for everything you redistribute — kernel, Mesa, systemd, browsers if included, Steam/Proton stack adjacency, Bluetooth, etc.

| Strategy | Maintainer tax | Fit |
|----------|----------------|-----|
| **Track Arch rolling + curated overlay** (Omarchy-like) | High cadence; breakage risk; fast CVE flow if you keep updating | Matches current README lean; needs **rollback** |
| **Pin / snapshot + staged channels** | Medium; you backports or wait | Better for “gaming weekend not ruined” |
| **Yocto/Buildroot custom** | Heavy recipe expertise; better SBOM/CVE tooling long-term | Overkill unless appliance/hardware SKU |

**Ongoing obligations after P3:**

- Triage CVEs that affect *your* installed set (not the entire CVE universe).
- Rebuild and push images/packages; verify Gamescope/GPU path still works.
- Secure update transport (signatures); don’t curl|bash production updates.
- Decide support window (“we support last N releases / rolling forever”).
- Firmware and NVIDIA/AMD proprietary bits: policy for blobs and redistributability.

Agents can draft patch notes and bump pins; **a human must own “ship or not” on security updates that break gameplay.**

---

## 6. “Don’t do this” traps

1. **Build a distro before cores are hostable.** Without family API / lib extract, linux becomes a themed Arch with a .desktop file.
2. **Fork Omarchy/Arch configs wholesale.** Copying without ownership of update boundaries violates repo `AGENTS.md` and creates unmaintainable drift.
3. **Custom kernel “for fun.”** Only fork/patch with a measured need; prefer upstream + config.
4. **Vendor forks of `*-core` into this repo.** Family rule: hosts depend on cores; never fork machines into linux ([02 §7](/cursor/stores/bc-57076e62-f812-47da-bf4c-f3bb0f3af797/docs/graycart-family/02-repo-layout.md#7-what-not-to-do)).
5. **Ship kernel/modules without corresponding source/offer.** Classic GPL footgun.
6. **Market “Graycart Linux” publicly without trademark plan.** Name is already in the repo — resolve before ISO splash screens.
7. **Closed appliance that hides recovery.** Product principle: escape hatches; fail loudly.
8. **Controller-only UX.** Violates stated vision; breaks development and accessibility.
9. **Commercial ROMs / BIOS in CI or images.** Same posture as gb/gba.
10. **Promise SteamOS/Bazzite parity.** Scope trap; differentiate on Graycart host + sane gaming defaults.
11. **Block GBA greenfield on linux phases.** Parallel tracks; supersede cutover is later ([10 §5](../graycart-gba/10-core-api-and-gb-reuse.md#5-phased-reuse-vs-greenfield-gba)).
12. **Treat rolling updates as optional.** No rollback + rolling = support nightmare.
13. **Giant bootstrap scripts / speculative abstractions** during research (`AGENTS.md`).
14. **Relicense upstream components** or strip license texts from the image.

---

## 7. Decision checklist for Dave (before writing code)

Use as go / no-go. Prefer **no-go on implementation** until checked items are decided or explicitly deferred with an owner.

### Scope & product

- [ ] Confirm synthesis: **OS host + gaming defaults**, not a second emulator tree.
- [ ] Accept model from [01](./01-scope-and-models.md) — especially **host-first vs Model 4 ISO**, and **base** (bootc/UB vs Arch/Omarchy-inspired vs Nix).
- [ ] Resolve README Arch lean vs [01](./01-scope-and-models.md) §10 default (atomic Fedora-family) explicitly.
- [ ] Accept stack from [02](./02-technical-stack.md) (bootloader, session, image tool, update strategy) when present.
- [ ] Name public brand; **LF Linux sublicense** path if “Linux” stays in the mark.
- [ ] Success metric for v1 (e.g. “CI boots + Graycart GB loads fixture” vs “friends install ISO”).

### Family / cores dependency

- [ ] Accept family [01](/cursor/stores/bc-57076e62-f812-47da-bf4c-f3bb0f3af797/docs/graycart-family/01-core-api.md) + [02](/cursor/stores/bc-57076e62-f812-47da-bf4c-f3bb0f3af797/docs/graycart-family/02-repo-layout.md) enough to name linux as host consumer.
- [ ] Decide gb extract vs interim whole-crate ([GBA 10 §6](../graycart-gba/10-core-api-and-gb-reuse.md#6-open-choices-for-dave-review)).
- [ ] Decide whether linux is a **named SemVer consumer** before 1.0.
- [ ] Confirm linux **must not** block GBA P0–P9.

### Legal & ops

- [ ] LICENSE for *this* repo’s configs/scripts.
- [ ] GPL source accompaniment / written-offer process for images.
- [ ] Firmware / proprietary GPU policy.
- [ ] Who is on-call for security updates after first public image.
- [ ] Hardware classes in-scope for P3 (desktop only? handheld later?).

### Process

- [ ] P0 docs mirrored to GitHub; research stage still forbids large implementation PRs.
- [ ] P1 CI budget accepted (image build minutes).
- [ ] Explicit **no code** on installer/ISO until P0 accept + P1 design spike signed.

**Suggested default go path:** Accept P0 docs → spike P1 QEMU image only → gate P2 on family ABI availability → delay P3 marketing until trademark + update/rollback are real.

---

## 8. Interaction with graycart-gba superseding graycart-gb

Canonical policy: [`product-decision-supersede-gb.md`](/cursor/stores/bc-57076e62-f812-47da-bf4c-f3bb0f3af797/internal/graycart-gba/product-decision-supersede-gb.md).

| Concern | How linux fits |
|---------|----------------|
| **gba supersedes gb as desktop app** | Long-term users play 8-bit + GBA on **gba app** *or* on **linux host**; both sit above cores. |
| **Prefer reuse of graycart-gb for DMG/CGB** | Linux must consume **gb-core** (extracted or interim), not a linux-only SM83 rewrite. |
| **GBA-native greenfield** | Linux waits on `graycart-gba-core` for native GBA titles; does not reimplement ARM7. |
| **Library-first extract** | **Primary excuse** for the extract: desktop + linux (+ later nes/snes/n64) share one API ([family README](/cursor/stores/bc-57076e62-f812-47da-bf4c-f3bb0f3af797/docs/graycart-family/README.md)). |
| **Cutover (GBA P12)** | gb **app** demoted; **gb-core remains** for linux and gba compat. Linux never depended on the gb GUI binary. |
| **Sequencing** | Linux P2 can prove host seam on gb-core alone; full “Graycart family on linux” tracks gba + later systems. |

**Mental model:**

```text
graycart-linux (OS + session + host UI)
        │  family API / C ABI
        ▼
 graycart-gb-core ◄── reused by graycart-gba (compat) + linux
 graycart-gba-core ◄── linux + graycart-gba app
 (later nes/snes/n64 cores)
```

If gba never ships a lib boundary, linux cannot fulfill the family vision without forking — treat that as a **hard dependency**, not a nice-to-have.

---

## 9. Go / no-go summary

**Go (research / P0):** Continue docs, experiments with clear conclusions, QEMU spikes labeled non-product.

**No-go (implementation of installable OS) until:**

- 01 + 02 + this doc accepted  
- Brand/trademark path chosen  
- Update/rollback strategy chosen  
- Family host API path exists or is scheduled with owners  
- GPL source/offer process drafted for any binary image  

**No-go forever:** forking cores into linux, commercial ROMs in CI, proprietary-module-first images, claiming LF endorsement, blocking GBA native work on linux cosmetics.

---

## 10. Cross-links

| Doc | Role |
|-----|------|
| [01-scope-and-models.md](./01-scope-and-models.md) | Product shape options (sibling) |
| [02-technical-stack.md](./02-technical-stack.md) | Boot/kernel/userspace (sibling) |
| [graycart-family/README](/cursor/stores/bc-57076e62-f812-47da-bf4c-f3bb0f3af797/docs/graycart-family/README.md) | Host vs core |
| [graycart-gba/10](../graycart-gba/10-core-api-and-gb-reuse.md) | Lib extract + linux consumer |
| [project-context](../project-context.md) | Family map |
| [graycart-linux README](https://github.com/graycart/graycart-linux) | Public vision |
| Internal status | [`internal/graycart-linux/status-roadmap.md`](/cursor/stores/bc-57076e62-f812-47da-bf4c-f3bb0f3af797/internal/graycart-linux/status-roadmap.md) |

### External references (planning)

- [Linux Foundation — Linux Mark](https://www.linuxfoundation.org/legal/the-linux-mark)  
- [Linux Foundation — Sublicense agreement](https://www.linuxfoundation.org/legal/sublicense-agreement)  
- [Kernel license rules](https://docs.kernel.org/process/license-rules.html)  
- [GPL-2.0 text (kernel tree)](https://github.com/torvalds/linux/blob/master/LICENSES/preferred/GPL-2.0)  
- [Omarchy](https://omarchy.org/) (inspiration only; not a license to copy)  
