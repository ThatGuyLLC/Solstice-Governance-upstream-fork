# Solstice Governance Repository
## 1.1 Purpose

This repository is the operational layer of the governance defined in the Solstice FIP. The FIP fixes what is enforced: the two governance tiers and their powers, the change lifecycle, and the invariants the threat model relies on. This repository holds how those tiers operate: addresses for each governance Safe, the procedures for verification, rotation, disputes, and the parameters designed to iterate without a FIP.

## 1.2 What lives here
### How this document maps to the repository

Each section below is a separate page, ordered by its number, so a reader interested in only one topic can open that page alone. File names carry the section number so they sort in reading order.

| Section | Repository file | Covers |
| :-- | :-- | :-- |
| 1. README | [`README.md`](README.md) | Purpose, references, section map |
| 2. Solstice Program Governance | [`docs/02-solstice-program-governance.md`](docs/02-solstice-program-governance.md) | Governance tiers & Safes · SWA tier · SRA tier (+ Orchestrator Registry) · Parameters · Safety and rotation playbook |
| 3. Orchestrator Operational Guidelines | [`docs/03-orchestrator-operational-guidelines.md`](docs/03-orchestrator-operational-guidelines.md) | Orchestrator guidelines · policies · tasks · monitoring |
| 4. Quarterly Review and Runbook | [`docs/04-quarterly-review-and-runbook.md`](docs/04-quarterly-review-and-runbook.md) | Quarterly community reporting duty · report template · example · declaration, verification & escalation |
| 5. Reports | [`docs/05-quarterly-reports.md`](docs/05-quarterly-reports.md) | Archive/history of filed quarterly reports (one page per report) |
| 6. Program Change Log | [`docs/06-changelog.md`](docs/06-changelog.md) | Rule-change history with community issues & FIP links |
| Open decisions | [`docs/07-open-decisions.md`](docs/07-open-decisions.md) | Every `TODO(owners)` still waiting on the owners |
| Storage provider FAQ | [`docs/storage-provider-faq.md`](docs/storage-provider-faq.md) | What this repository does and does not tell storage providers |
| Activation day | [`docs/activation-day-2026-10-19.md`](docs/activation-day-2026-10-19.md) | Checklist for 2026-10-19 13:00 UTC |

### Contains key documentation including:
- Tasks and actions for each ecosystem actor, inclusive of the operational guidelines for each role, and admission and removal criteria for Orchestrators.
- Measurement-rule declarations for each orchestrator.
- The registry of admitted Orchestrators and the change log of program-rule updates.
- The dispute procedure for contested bindings, and the verification playbook.
- The versioned reference indexer that recomputes FPV from public settlement events will be maintained here. It is not published yet. FIP-0118 §2.2, lines 708–710 at the pin below, reads:

```
        paragraph is the normative algorithm; a versioned reference
        implementation of the indexer is maintained in the governance
        repository.
```

- Pointers to the FIP-fixed durations of `POST_PERIOD`, `VERIFICATION_WINDOW`, `SRA_UPGRADE_HOLD` and `EPOCHS_PER_QUARTER` (not edited here; changed only by an SRA code upgrade).
- [The contract upgrade runbook](https://github.com/filecoin-project/solstice/blob/b9a63a87b5adee272c16d7e507cc9c127fb34f0f/docs/UPGRADE.md) in the [solstice](https://github.com/filecoin-project/solstice) repository, pinned at `b9a63a87b5adee272c16d7e507cc9c127fb34f0f`. Upgrades are driven by engineers at the request of governance (an accepted FIP, approved by both tiers' Safes), and the runbook stays next to the scripts and workflows it depends on so they are reviewed together. Latest text, not the pin: [solstice `docs/UPGRADE.md` on `main`](https://github.com/filecoin-project/solstice/blob/main/docs/UPGRADE.md).
- The wallet addresses for each tier's Safes.
- Issue templates, application forms, and workflow automation.

## 1.3 Changing this repository

Changes happen by pull request with public review. Every change to the program rules is recorded in the [Program Change Log](docs/06-changelog.md), which links the originating community issue and, where applicable, the resulting FIP.

Who may merge, the review window for the procedures in docs 02–04 and the policies, whether those sections need sign-off from both tiers, and whether commits must be signed are not set. `TODO(owners):` name them in [MAINTAINERS.md](MAINTAINERS.md). Branch protection on `main` is a GitHub setting. This file does not turn it on.

## 1.4 Reference Links

- FIP text pinned for quotations in this repository: [FIP-0118 at `23b42964890590a49c9cbc6e19a7e9e361d1f8a0`](https://github.com/filecoin-project/FIPs/blob/23b42964890590a49c9cbc6e19a7e9e361d1f8a0/FIPS/fip-0118.md). A quotation is updated in the same pull request that follows a change in that text. Until the reference indexer is published, this pinned text is the normative FPV algorithm. Latest text, not used for quotations: [FIP-0118 on `master`](https://github.com/filecoin-project/FIPs/blob/master/FIPS/fip-0118.md).
- A check compares that pin with `master` and fails if they differ. It does not open an issue or a pull request. `TODO(owners):` name an owner for the check (`.github/workflows/fip-pin-check.yml`).
- Contract upgrade runbook, pinned: [solstice `docs/UPGRADE.md` at `b9a63a87b5adee272c16d7e507cc9c127fb34f0f`](https://github.com/filecoin-project/solstice/blob/b9a63a87b5adee272c16d7e507cc9c127fb34f0f/docs/UPGRADE.md). Latest, not the pin: [solstice `docs/UPGRADE.md` on `main`](https://github.com/filecoin-project/solstice/blob/main/docs/UPGRADE.md). An upgrade issue pins this file by commit. The pending upgrade and its execution must be observable on chain (FIP-0118 §4.2). `TODO(owners): confirm` whether the issue must also record an implementation bytecode hash.
- Reference indexer: TBD. `TODO(owners):` an owner, a backup maintainer, and a ship date.
- Settlement data and dashboards: TBD. `TODO(owners):` the publication decision.

## 1.5 Repository Maintainers

Maintainers keep this repository current and administer pull requests. Who they are is not stated. `TODO(owners):` name them in [MAINTAINERS.md](MAINTAINERS.md). Authority over the Solstice program rests with two governance tiers (the SWA and SRA Safes) as fixed by FIP-0118.

## 1.6 Open decisions

Every `TODO(owners)` in this repository is collected in [Open decisions](docs/07-open-decisions.md). Text marked `TODO(owners): approve` or `TODO(owners): confirm` is proposed. It is not a rule in force.

---

*Solstice Governance Repository — operational layer of [FIP-0118 at `23b42964890590a49c9cbc6e19a7e9e361d1f8a0`](https://github.com/filecoin-project/FIPs/blob/23b42964890590a49c9cbc6e19a7e9e361d1f8a0/FIPS/fip-0118.md). Latest, not used for quotations: [FIP-0118 on `master`](https://github.com/filecoin-project/FIPs/blob/master/FIPS/fip-0118.md).*
