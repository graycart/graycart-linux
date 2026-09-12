# Attribution — graycart-linux

Mandatory for in-repo Markdown (and future Rust/scripts) that depend on external code or documentation.

Also stated in [`AGENTS.md`](./AGENTS.md). Provenance notes and README link lists alone are **not** enough for in-tree sources that use external materials.

## Rule

If a file references someone else's **code** or **internet documentation** (kernel.org, Arch Wiki, Fedora/bootc, systemd, Mesa, Steam/Proton, Omarchy, Buildroot, Yocto, blogs, another distro, research excerpts, etc.), put **credit at the top of that file**:

| Field | Required |
|-------|----------|
| What | Document / project / section used |
| Where | URL and/or canonical name |
| How | Brief note: *cited* / *inspired by* / *ported from* / *cross-checked against* |

Applies to:

- Future Rust / shell / Containerfile sources (`//!` / header comments)
- Markdown in this repo when the page quotes, ports, or structurally follows an external source

Does **not** replace:

- Minimal excerpts + license/terms when pasting upstream text
- LICENSE / NOTICE tables for vendored configs (when they exist)

Never commit credentials, machine-specific secrets, proprietary firmware blobs, commercial game content, full manuals, or user data.

## Example — Markdown (`.md`)

HTML comment (keeps the visible title clean):

```markdown
<!--
Cited: Fedora Magazine — Building your own atomic bootc desktop
URL: https://fedoramagazine.org/building-your-own-atomic-bootc-desktop/
Note: update/rollback framing for Model 4; not a full reprint.
-->
# Atomic image notes
```

Or a visible header block under the title:

```markdown
# Boot chain notes

title: systemd-boot documentation
URL: https://www.freedesktop.org/software/systemd/man/latest/systemd-boot.html
retrieved: 2026-09-12
license/terms: upstream docs — minimal excerpt only
why cited: ESP layout / UKI entry points
```

## Example — future Rust / script

```rust
//! Gamescope session wrapper stubs.
//!
//! Cited: Gamescope README — nested compositor usage
//!   https://github.com/ValveSoftware/gamescope
//! Cross-check: Bazzite session packaging (secondary; behavior TBD).
```

## Checklist before coding a behavior change

1. Name the primary reference and the acceptance test.
2. Add or update the **file-top credit** in every touched source that relied on that material.
3. If you paste or closely paraphrase an excerpt into research docs, keep the citation block and prefer linking over reprinting.

## Related

- [`AGENTS.md`](./AGENTS.md) — full agent norms
- [`docs/README.md`](./docs/README.md) — research pack index
