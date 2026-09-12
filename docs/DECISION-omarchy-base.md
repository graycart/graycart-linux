<!--
Mirrored from Agent Store docs/graycart-linux/ — durable copy for this repo.
Cross-family links point at Graycart Project Agent Store paths.
No secrets / no binary blobs.
-->

# Decision — Omarchy as graycart-linux base

**Date:** 2026-09-12  
**Decider:** Dave  
**Status:** Accepted

## Decision

**graycart-linux will be based on Omarchy** (DHH / Basecamp–line project: [omarchy.org](https://omarchy.org), [basecamp/omarchy](https://github.com/basecamp/omarchy) → org [omacom/omarchy](https://github.com/omacom/omarchy)).

This is **not** “ship Omarchy with a Graycart wallpaper.” It is a **hard-customized derivative**: keep Omarchy’s install/update/Arch spine, **strip** DHH-desktop chrome and apps irrelevant to an emulator host, **replace/add** Graycart session + family cores (`graycart-*`), and stay **rebaseable** on Omarchy upstream.

## What this settles

| Question | Answer |
|----------|--------|
| Product model for Meaning B (branded OS) | **Omarchy-based derivative** (see [01](./01-scope-and-models.md)) |
| Prior “default Model 4 atomic/bootc” recommendation | **Superseded** as the chosen path; remains an *alternative / non-goal* unless Omarchy becomes untenable |
| Host vs OS | Still two layers: family **host** on cores + **OS image** that delivers that host |
| Customization intensity | **Hard strip/tune** — appliance/host posture, not ricing Omarchy |

## Clarification (same day)

Dave: base on Omarchy, but **customize hard** — lean desktop, graycart-* cores, minimal inherited chrome we don’t need. Explicit **KEEP / STRIP / REPLACE·ADD** matrix lives in [04-omarchy-base.md](./04-omarchy-base.md).

## Implications (accepted, not argued)

1. Stack starts from Omarchy’s real components (Arch + pacman, Limine, Hyprland/Quickshell today, Omarchy ISO/pkgs/mirror channels) — then **subtract**.
2. Roadmap = fork/brand/overlay → strip → Graycart host/session → public image; primary risks = upstream sync, GPL redistributor duties, “Linux” trademark, keeping merges rebaseable.
3. Branding must not imply official Omarchy/Basecamp affiliation; “Linux” in product name still needs LF Mark planning.

## Related docs

- [01-scope-and-models.md](./01-scope-and-models.md)  
- [02-technical-stack.md](./02-technical-stack.md)  
- [03-roadmap-and-risks.md](./03-roadmap-and-risks.md)  
- [04-omarchy-base.md](./04-omarchy-base.md)  
- Internal: [`internal/graycart-linux/status-omarchy-base.md`](/cursor/stores/bc-57076e62-f812-47da-bf4c-f3bb0f3af797/internal/graycart-linux/status-omarchy-base.md)
