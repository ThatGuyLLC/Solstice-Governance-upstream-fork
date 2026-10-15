# Open decisions

Every `TODO(owners)` in this repository is a decision the owners have not made. A marker is not a rule in force.

- `TODO(owners): approve` — proposed text. It does not apply until the owners approve it.
- `TODO(owners): confirm` — proposed text that restores a confirmation flag from pull request #18. It does not apply until the owners confirm it.
- `TODO(owners):` — a blank the owners have to fill. No name, threshold, list, or counsel conclusion is supplied here.

No person is named in this list.

| ID | Marker | Decision |
| :-- | :-- | :-- |
| F01 | `TODO(owners):` | Organization 1 splits its signers per tier, or discloses the overlap in [§2.4.1](02-solstice-program-governance.md#241-safe-addresses). A different-entities rule would be a FIP change. |
| F02 | `TODO(owners):` | Name the entity and the affiliation disclosure for each governance Safe. |
| F03 | `TODO(owners): approve` | Internal recusal of an affiliated signer. Not in force. |
| F04 | `TODO(owners):` | Indexer owner, backup maintainer, and ship date. Publication decision for settlement data and dashboards. |
| F05 | `TODO(owners): approve` | 48 hours between publication of a recomputation and the first `CorrectVolume` approval. Not in force. Not a FIP requirement. |
| F06 | `TODO(owners): confirm` | A "correction blocked" note is a verification trigger without a further confirmation, and day 5 of the verification window is a hard internal deadline with a public notice if the second signature has not landed. Until confirmed, day 5 is an aim and the note is not by itself a removal ground. |
| F09 | `TODO(owners):` | An independent appeal reviewer. The day-4 filing deadline for an in-window `CorrectVolume` dispute is repository procedure. The reviewer is not named. |
| F10 | `TODO(owners):` | Publish the admission rubric and a decision deadline before the first application is decided. |
| F11 | `TODO(owners):` | The number of consecutive quarters in Policy 7, or delete that policy. Inactivity is not a removal ground until then. Whether "binding fraud" is a ground distinct from misreported FPV and wrongful claims. It is not defined. |
| F14 | `TODO(owners):` | Name a primary and a backup cranker for `SubmitShares` and `QuarterlyGateCheck`. The 24-hour text is a repository deadline. The names are blank. |
| F16 | `TODO(owners):` | Whether proxy addresses hardwired in Lotus, Forest, and Venus change the SRA path in FIP-0118 §4.3. |
| F17 | `TODO(owners):` | Emergency-annex contacts, alert channel, pre-drafted emergency FIP, and a tabletop. None are named. |
| F18 | `TODO(owners): confirm` | A cancellation, and a decision not to cancel, is recorded with a written reason. The contract does not require the reason. |
| F19 | `TODO(owners): approve` | Veto a half-approved task older than 14 days, and record a written reason for every veto. Not in force. |
| F23 | `TODO(owners):` | Maintainer names, who may merge, the review window, both-tier sign-off, and signed commits. See [MAINTAINERS.md](../MAINTAINERS.md). Branch protection is a GitHub setting and is not set here. |
| F24 | `TODO(owners):` | Complete the [activation-seat disclosure](disclosures/activation-seat.md) before 2026-10-19 13:00 UTC, including who operates the seat and the f1 claim keeper. |
| F28 | `TODO(owners):` | Sanctions lists, who screens, and the cadence. None are named. |
| F32 | `TODO(owners): confirm` | Whether an upgrade issue records an implementation bytecode hash. |
| F35 | `TODO(owners):` | Out-of-band identity verification for a lost `orch` key. Not defined. |
| F36 | `TODO(owners):` | Indicative numeric values for anomaly phrases, used only as triggers. None are set. |
| F41 | `TODO(owners):` | Launch-seat removal criteria, and whether to aim for a second Orchestrator before the end of Q2. Criteria are not set. |
| F42 | `TODO(owners):` | Transaction hashes for the pre-activation `SetAdmittedLists` and `SetPricingParams` events, mainnet and calibnet. |
| F45 | `TODO(owners):` | Allowed ranges for `MIN_LOT_FLOOR`, `MIN_LOT_ALPHA`, `PRICE_BAND`, and `REGISTRATION_CUTOFF`. No range is set. The 14-day notice and written rationale are repository procedure. |
| F46 | `TODO(owners): approve` | Quarterly `getOwners` / `getThreshold` snapshot. Not in force. |
| F47 | `TODO(owners): confirm` | A `ReplaceWallet` request carries a message signed by `orch` (EIP-1271 when `orch` is a contract) naming the old wallet, the new wallet, and the chain id. |
| F48 | `TODO(owners):` | Hashing tool and pinning provider for a report CID. Who opens the reporting issue and who merges the report pull request. |
| F51 | `TODO(owners): approve` | Per-pair affiliation flag in the declaration. Not in force. |
| F52 | `TODO(owners): approve` | Publish `MIN_LOT(2)` and its inputs at Q1 close. Not in force. |
| F53 | `TODO(owners): approve` | Each Safe decodes calldata with its own tool and signs only on a match. Lock the issue after the first approval. Not in force. |
| F54 | `TODO(owners):` | The transaction simulator, the signing stack, and the indexer release owner. No product is named. |
| F56 | `TODO(owners):` | Whether a sibling registration of a removed Orchestrator's pairs is treated as wrongful and corrected by `ReassignBinding` by default. The default is not set. Common-control disclosure is asked at admission. |
| F57 | `TODO(owners): approve` | "Materially false or misleading report" as a removal ground, and the attestation line in the report template. Not in force. Not a FIP requirement. |
| F58 | `TODO(owners): confirm` | Disputes filed in the last 14 days of a quarter are decided, and any `ReassignBinding` sent, before `Start(Q+1)`. |
| F59 | `TODO(owners):` | A signer code of conduct and an annual no-payment attestation. Not set. |
| F61 | `TODO(owners):` | Announcement channel and whether any translation is published. |
| F62 | `TODO(owners):` | Legal review of data protection, pending counsel, and whether a private evidence submission to both Safes is adopted. No private channel is opened. |
| F63 | `TODO(owners): approve` | Outgoing and incoming Orchestrators notify affected payers, and the notice is posted on the issue. Not in force. |
| F64 | `TODO(owners):` | Legal review of roles, liability, and regulatory framing, pending counsel. No conclusion is stated. |
| F65 | `TODO(owners):` | KYC and beneficial-ownership standard, pending counsel. The admission checklist has a slot. The standard is not defined. |
| F66 | `TODO(owners):` | Archive method for decision records beyond GitHub. |
| F67 | `TODO(owners):` | Names on the [activation-day checklist](activation-day-2026-10-19.md). |
| F69 | `TODO(owners): approve` | Quarterly on-chain read of the deployed parameters. Not in force. |
| F70 | `TODO(owners):` | Owner of the FIP-pin check. The check fails on a difference and does not open an issue or a pull request. |
| F71 | `TODO(owners):` | Raise the calibnet launch Safe threshold from 1 to 2. |
| — | `TODO(owners):` | Who watches for a correction that changes whether the gate passes. A reversal template, if one is wanted before it is needed. |

Inline markers repeat some of these rows (both tiers, both templates). They are the same decision.

---

← [Back to README](../README.md)
