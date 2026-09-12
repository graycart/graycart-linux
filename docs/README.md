# graycart-linux research docs

Research pack for a gaming-focused Linux **host / optional branded image** in the [Graycart](https://github.com/graycart/graycart) family. **Review gate before building images or a distro** ([03](./03-roadmap-and-risks.md)).

**Agent norms + credits:** [`AGENTS.md`](../AGENTS.md) · [`ATTRIBUTION.md`](../ATTRIBUTION.md) — file-header attribution is **mandatory** before any in-repo sources that cite external docs or code.

| | |
|--|--|
| **Target repo** | [graycart/graycart-linux](https://github.com/graycart/graycart-linux) — placeholder + vision; this docs pack |
| **Peers** | [graycart-gb](https://github.com/graycart/graycart-gb) (interim app) · [graycart-gba](https://github.com/graycart/graycart-gba) (long-term app) |
| **Umbrella** | [graycart/graycart](https://github.com/graycart/graycart) — submodule pins only; not a Cargo workspace |
| **Agent Store partition** | `/cursor/stores/bc-57076e62-f812-47da-bf4c-f3bb0f3af797/docs/graycart-linux/` |
| **Family API** | Agent Store [`docs/graycart-family/`](/cursor/stores/bc-57076e62-f812-47da-bf4c-f3bb0f3af797/docs/graycart-family/README.md) — linux is **host-only** over `*-core` |

### Product goal (two meanings)

1. **Meaning A — Linux host app:** shell that loads Graycart cores via family trait / C ABI (already in family design).
2. **Meaning B — branded OS / image (open):** installable “Graycart Linux”; research default is **Model 4** immutable/atomic image *after* host-on-stock-distros ([01](./01-scope-and-models.md)). Omarchy/Arch remains UX inspiration only — not the recommended update architecture.

Semver / packaging policy: TBD once implementation begins; match family host posture (lib-first cores, no machine logic in this repo).

---

## Reading order

0. **[AGENTS.md](../AGENTS.md)** + **[ATTRIBUTION.md](../ATTRIBUTION.md)** — norms and mandatory file-header credits.
1. **[vision.md](./vision.md)** — product vision already in-tree (boot-to-play, controller-first, recovery).
2. **[01 — Scope & models](./01-scope-and-models.md)** — host vs branded OS; five models; default path + non-goals.
3. **[02 — Technical stack](./02-technical-stack.md)** — boot → kernel → userspace → graphics → SBOM/CI.
4. **[03 — Roadmap & risks](./03-roadmap-and-risks.md)** — phases, legal, traps, go/no-go.

Family-standard core API (sibling-owned): Agent Store [`docs/graycart-family/`](/cursor/stores/bc-57076e62-f812-47da-bf4c-f3bb0f3af797/docs/graycart-family/README.md).

---

## Document index

| Doc | Topic | Status |
|-----|--------|--------|
| [AGENTS.md](../AGENTS.md) | Agent norms (public repo) | Written |
| [ATTRIBUTION.md](../ATTRIBUTION.md) | File-header credit policy | Written |
| [vision.md](./vision.md) | Product vision / experience goals | Written |
| [01-scope-and-models.md](./01-scope-and-models.md) | Scope + competing product models | Mirrored from Agent Store |
| [02-technical-stack.md](./02-technical-stack.md) | Boot / kernel / userspace / packaging | Mirrored from Agent Store |
| [03-roadmap-and-risks.md](./03-roadmap-and-risks.md) | Phases, cost, legal, go/no-go | Mirrored from Agent Store |

Cross-family and internal status links in mirrored pages point at Agent Store absolute paths so Cursor agents can follow them. GitHub web readers: use the [store index](/cursor/stores/bc-57076e62-f812-47da-bf4c-f3bb0f3af797/docs/graycart-linux/README.md) when available in-agent.

---

## Still stubbed / open

- Implementation code, `Cargo.toml`, image CI, and installable ISOs — **not started** (research gate).
- Umbrella submodule pin for `graycart-linux` — **recommended, not yet proposed** (see scaffold status).
- Dave acceptance of 01–03 (Model 4 vs Arch/Omarchy lean; host-first sequencing).

---

## Internal status (Agent Store)

Agents drop short completion notes under `internal/graycart-linux/status-*.md`:

- [`status-scope.md`](/cursor/stores/bc-57076e62-f812-47da-bf4c-f3bb0f3af797/internal/graycart-linux/status-scope.md)
- [`status-tech-stack.md`](/cursor/stores/bc-57076e62-f812-47da-bf4c-f3bb0f3af797/internal/graycart-linux/status-tech-stack.md)
- [`status-roadmap.md`](/cursor/stores/bc-57076e62-f812-47da-bf4c-f3bb0f3af797/internal/graycart-linux/status-roadmap.md)
- [`status-scaffold.md`](/cursor/stores/bc-57076e62-f812-47da-bf4c-f3bb0f3af797/internal/graycart-linux/status-scaffold.md)
