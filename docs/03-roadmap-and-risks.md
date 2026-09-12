<!--
Mirrored from Agent Store docs/graycart-linux/ — durable copy for this repo.
Cross-family links point at Graycart Project Agent Store paths.
No secrets / no binary blobs.
-->

# graycart-linux — Roadmap, cost, legal, and traps

**Audience:** Dave (go / no-go) + implementers.  
**Repo:** [graycart/graycart-linux](https://github.com/graycart/graycart-linux).  
**Decision:** [DECISION-omarchy-base.md](./DECISION-omarchy-base.md) — Omarchy base, **hard customize**.  
**Siblings:** [01](./01-scope-and-models.md) · [02](./02-technical-stack.md) · [04](./04-omarchy-base.md).  
**Family:** [`docs/graycart-family/`](/cursor/stores/bc-57076e62-f812-47da-bf4c-f3bb0f3af797/docs/graycart-family/README.md).

**Disclaimer:** Legal sections summarize public GPL-2.0 / Linux Mark / MIT guidance for planning. Not legal advice.

---

## 0. What this project is

| Lens | Meaning |
|------|---------|
| **Product** | **Omarchy-derived** lean **host/appliance OS** — emulator-first; Graycart session + cores |
| **Family API** | Host for Graycart cores via trait / C ABI — no machine logic in OS recipes |
| **Not** | Stock Omarchy; Omarchy + wallpaper; from-scratch kernel |

**Roadmap spine:** fork/brand/overlay on Omarchy → **strip** per [04 matrix](./04-omarchy-base.md#2-strip--keep--replace-matrix) → add Graycart host → public image while **staying rebaseable**.

---

## 1. Phased roadmap

### P0 — Docs / bootstrap

**Exit:** Decision recorded; 01–04 accepted as working plan; strip/keep matrix reviewed.

| Work | Done when |
|------|-----------|
| Omarchy research + decision note | [DECISION](./DECISION-omarchy-base.md) + [04](./04-omarchy-base.md) |
| Scope/stack/roadmap amended | 01–03 match Omarchy-hard-customize |
| Mirror into `graycart-linux` repo | Scaffold / docs PR |
| Family prerequisites named | abi + gb-core path |

### P1 — Bootable lean CI image

**Exit:** Reproducible Omarchy-derived artifact boots under QEMU; **stripped** package set; Graycart branding present; not a product ISO claim.

| Work | Done when |
|------|-----------|
| Pin Omarchy tag + list KEEP packages | `UPSTREAM.md` |
| ISO/rootfs recipe with lean manifest | Rebuild from empty runner |
| CI boot smoke (systemd target) | Green on `main` |
| Source/SBOM stub | Compliance checklist started |
| Snapshot/rollback still works | Limine path verified or deliberate alternate documented |

### P2 — Graycart host on the image

**Exit:** Session launches Graycart host; family API; fixture ROM; input + audio.

| Work | Done when |
|------|-----------|
| Graycart session/launcher replaces Omarchy desktop IA | First paint is Graycart |
| `graycart-gb` (then gba) packaged | No cores vendored into OS git |
| Compositor lean (Hyprland strip or Gamescope) | Host fullscreen OK |
| ROM-free CI | Fixtures only |

### P3 — Installable product

**Exit:** Public image/channel; update + recovery; hardware matrix; trademark + GPL process.

| Work | Done when |
|------|-----------|
| Installer path documented | Stranger can install |
| `graycart-update` (or wrap) + channels | Bad update recoverable |
| LF Linux Mark if needed; Omarchy attribution | No mark confusion |
| Rebase runbook proven on ≥1 Omarchy upgrade | Mergeable in practice |

Do not reorder to “pretty ISO first, host later.”

---

## 2. Rough effort

| Phase | Shape | Dominant risk |
|-------|-------|----------------|
| **P0** | Days | Scope thrash (done: Omarchy + hard strip) |
| **P1** | ~2–6 weeks to reliable lean boot | Strip too little (bloat) or too much (break update spine) |
| **P2** | ~1–3 months after API/core extract | Family ABI / GUI entanglement |
| **P3** | Quarters + permanent | **Upstream sync**, CVE, marks |

Agents accelerate packaging; humans own rebase ship/no-ship and security.

---

## 3. Primary risks (accepted with Omarchy choice)

### 3.1 Upstream sync / rebaseability

Omarchy moves fast (Quattro package re-arch, Quickshell, migrations). Graycart must:

- Minimize patched file surface; prefer overlay packages.  
- Pin tags; merge regularly; keep `UPSTREAM.md`.  
- Never “fork and forget.”  

Failure mode: unmaintainable diff → forced jump to Model 4 atomic fallback ([01](./01-scope-and-models.md)).

### 3.2 GPL / copyleft (kernel & modules)

Shipping Arch kernel + GPL userspace triggers GPL-2.0 distributor duties ([kernel license rules](https://docs.kernel.org/process/license-rules.html)): accompany corresponding source **or** ≥3-year written offer, etc. MIT on Omarchy configs does **not** waive kernel obligations.

Checklist: image + recipe tag; offer URL; SBOM; no mystery blobs; CI retention aligned with offer.

### 3.3 Trademark — “Linux” and Omarchy

| Mark | Action |
|------|--------|
| **Linux®** | Sublicense via LF if “Graycart Linux” is product brand ([Linux Mark](https://www.linuxfoundation.org/legal/the-linux-mark)); or brand **Graycart OS** and say “Linux-based” in prose |
| **Omarchy / Basecamp / DHH identity** | Nominative “based on Omarchy” OK; do not ship Omarchy logos or imply official spin |
| Steam/etc. | Separate marks if optional packages added later |

### 3.4 Strip discipline

Keeping DHH desktop chrome “for now” recreates Omarchy and misses the product. Treat STRIP list as release-blocking for P1 defaults.

### 3.5 Security maintenance

Rolling Arch via Omarchy channels = fast CVEs **and** breakage. Keep snapshot rollback; prefer stable-like channel for users; human gates gameplay-breaking updates.

---

## 4. GPL detail (unchanged substance)

Under GPL-2.0 §3, object forms need corresponding source or valid written offer. Prefer shipping exact build recipes (ISO configs, PKGBUILDs, package lists). Do not point only at kernel.org/Arch and call distributor duty done.

Graycart **userspace** cores (MIT/Apache-style) talking via syscalls are not GPL’d by the kernel syscall note; still respect any GPL libs you link.

---

## 5. “Don’t do this” traps

1. **Omarchy + wallpaper** as the product.  
2. **Ship full Omarchy preinstall set** then hope users remove it.  
3. **Fork Omarchy wholesale** without overlay discipline → unmergeable.  
4. **Custom kernel for fun.**  
5. **Vendor `*-core` into the OS repo.**  
6. **Kernel/modules without source/offer.**  
7. **Market Graycart Linux without Linux Mark plan.**  
8. **Use Omarchy marks as ours.**  
9. **RetroArch-default** instead of Graycart hosts.  
10. **Block GBA greenfield** on linux cosmetics.  
11. **Bypass update guards** in docs so users skip snapshots/migrations.  
12. **Commercial ROMs/BIOS** in CI or images.  
13. **Promise SteamOS parity.**  
14. **Relicense upstream** or strip license texts.

---

## 6. Decision checklist for Dave

### Settled

- [x] Base on **Omarchy** (2026-09-12).  
- [x] **Hard customize** — strip/keep/replace ([04](./04-omarchy-base.md)).  
- [x] Synthesis: OS host + gaming/emulator defaults, not second emulator tree.

### Still open

- [ ] Public brand: Graycart Linux vs Graycart OS.  
- [ ] Session: lean Hyprland vs Gamescope-first.  
- [ ] Overlay vs deeper fork.  
- [ ] Who owns rebase + security after first public image.  
- [ ] LICENSE for *Graycart* recipes/scripts.  
- [ ] P1 CI budget.

**Suggested go path:** Accept P0 → P1 lean boot CI → P2 host → P3 marketing only with trademark + update/rollback + rebase runbook.

---

## 7. Interaction with graycart-gba superseding graycart-gb

Canonical: [`product-decision-supersede-gb.md`](/cursor/stores/bc-57076e62-f812-47da-bf4c-f3bb0f3af797/internal/graycart-gba/product-decision-supersede-gb.md).

Linux consumes **gb-core** first, then **gba-core**; never reimplements machines. gba app supersede does not remove gb-core for the linux host.

```text
graycart-linux (stripped Omarchy + Graycart session)
        → family API
 graycart-gb-core / graycart-gba-core / …
```

---

## 8. Go / no-go summary

**Go (P0/P1 design):** Omarchy-derived lean image experiments.  

**No-go (public installable) until:** brand/trademark path; update/rollback; GPL offer process; family host path scheduled; rebase runbook exists.  

**No-go forever:** cores in OS repo; commercial ROMs in CI; Omarchy-as-our-brand; claiming LF endorsement; blocking GBA on linux cosmetics.

---

## 9. Cross-links

| Doc | Role |
|-----|------|
| [DECISION-omarchy-base.md](./DECISION-omarchy-base.md) | Decision record |
| [04-omarchy-base.md](./04-omarchy-base.md) | Strip/keep + derivative how-to |
| [01](./01-scope-and-models.md) / [02](./02-technical-stack.md) | Scope / stack |
| [Linux Mark](https://www.linuxfoundation.org/legal/the-linux-mark) | Trademark |
| [Kernel license rules](https://docs.kernel.org/process/license-rules.html) | GPL |
| [Omarchy](https://omarchy.org/) | Upstream |
| Internal | [`status-omarchy-base.md`](/cursor/stores/bc-57076e62-f812-47da-bf4c-f3bb0f3af797/internal/graycart-linux/status-omarchy-base.md) |
