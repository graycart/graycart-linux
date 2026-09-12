# graycart-linux research docs

Research pack for a gaming-focused Linux host — **Omarchy-derived; Graycart-stripped host OS** — in the [Graycart](https://github.com/graycart/graycart) family. **Review gate before shipping public images** ([03](./03-roadmap-and-risks.md), [04](./04-omarchy-base.md)).

**Agent norms + credits:** [`AGENTS.md`](../AGENTS.md) · [`ATTRIBUTION.md`](../ATTRIBUTION.md) — file-header attribution is **mandatory** before any in-repo sources that cite external docs or code.

| | |
|--|--|
| **Target repo** | [graycart/graycart-linux](https://github.com/graycart/graycart-linux) |
| **Product decision** | **Omarchy-derived; Graycart-stripped host OS** — [DECISION](./DECISION-omarchy-base.md) · [04](./04-omarchy-base.md) |
| **Peers** | [graycart-gb](https://github.com/graycart/graycart-gb) · [graycart-gba](https://github.com/graycart/graycart-gba) |
| **Umbrella** | [graycart/graycart](https://github.com/graycart/graycart) — submodule pins only; not a Cargo workspace |
| **Agent Store partition** | `/cursor/stores/bc-57076e62-f812-47da-bf4c-f3bb0f3af797/docs/graycart-linux/` |
| **Family API** | Agent Store [`docs/graycart-family/`](/cursor/stores/bc-57076e62-f812-47da-bf4c-f3bb0f3af797/docs/graycart-family/README.md) — linux is **host-only** over `*-core` |

### Product goal

1. **Omarchy-derived; Graycart-stripped host OS:** start from Omarchy’s Arch stack, aggressively strip stock branding/chrome, customize for a lean Graycart gaming host ([04 strip/keep matrix](./04-omarchy-base.md)).
2. **Host for Graycart cores:** load family `*-core` via trait / C ABI; no machine logic in this repo.
3. **Stay rebaseable** on Omarchy/Arch update spine where practical.

---

## Reading order

0. **[AGENTS.md](../AGENTS.md)** + **[ATTRIBUTION.md](../ATTRIBUTION.md)** — norms and mandatory file-header credits.
1. **[DECISION-omarchy-base.md](./DECISION-omarchy-base.md)** — Dave decision + posture.
2. **[04 — Omarchy base](./04-omarchy-base.md)** — inventory + KEEP/STRIP/REPLACE matrix.
3. **[vision.md](./vision.md)** — product vision (boot-to-play, controller-first, recovery).
4. **[01 — Scope & models](./01-scope-and-models.md)** — scope after Omarchy decision.
5. **[02 — Technical stack](./02-technical-stack.md)** — stack from Omarchy components + deltas.
6. **[03 — Roadmap & risks](./03-roadmap-and-risks.md)** — phases, legal, go/no-go.

Family-standard core API (sibling-owned): Agent Store [`docs/graycart-family/`](/cursor/stores/bc-57076e62-f812-47da-bf4c-f3bb0f3af797/docs/graycart-family/README.md).

---

## Document index

| Doc | Topic | Status |
|-----|--------|--------|
| [AGENTS.md](../AGENTS.md) | Agent norms | Written |
| [ATTRIBUTION.md](../ATTRIBUTION.md) | File-header credit policy | Written |
| [DECISION-omarchy-base.md](./DECISION-omarchy-base.md) | Product decision record | Mirrored |
| [04-omarchy-base.md](./04-omarchy-base.md) | Omarchy facts + strip/keep/replace | Mirrored |
| [vision.md](./vision.md) | Product vision / experience goals | Written |
| [01-scope-and-models.md](./01-scope-and-models.md) | Scope + models (post-decision) | Mirrored |
| [02-technical-stack.md](./02-technical-stack.md) | Technical stack deltas | Mirrored |
| [03-roadmap-and-risks.md](./03-roadmap-and-risks.md) | Phases, cost, legal, go/no-go | Mirrored |

Cross-family and internal status links in mirrored pages point at Agent Store absolute paths.

---

## Still open

- Implementation code, image CI, and installable ISOs — not started (research gate).
- Umbrella submodule pin for `graycart-linux` — recommended after research acceptance.
- Dave acceptance of strip/keep matrix in [04](./04-omarchy-base.md).

---

## Internal status (Agent Store)

- [`status-omarchy-base.md`](/cursor/stores/bc-57076e62-f812-47da-bf4c-f3bb0f3af797/internal/graycart-linux/status-omarchy-base.md)
- [`status-scope.md`](/cursor/stores/bc-57076e62-f812-47da-bf4c-f3bb0f3af797/internal/graycart-linux/status-scope.md)
- [`status-tech-stack.md`](/cursor/stores/bc-57076e62-f812-47da-bf4c-f3bb0f3af797/internal/graycart-linux/status-tech-stack.md)
- [`status-roadmap.md`](/cursor/stores/bc-57076e62-f812-47da-bf4c-f3bb0f3af797/internal/graycart-linux/status-roadmap.md)
- [`status-scaffold.md`](/cursor/stores/bc-57076e62-f812-47da-bf4c-f3bb0f3af797/internal/graycart-linux/status-scaffold.md)
