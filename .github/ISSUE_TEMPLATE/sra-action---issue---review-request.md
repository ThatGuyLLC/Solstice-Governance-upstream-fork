---
name: SRA Action - Issue / Review Request
about: Related to SRA actions - for community review
title: ''
labels: ''
assignees: ''

---

### YYYY-MM-DD — <short title>

- **Type:** SRA action issue
- **Chain id:** 314 (mainnet) | 314159 (calibnet)
- **Action Related to issue:** AddOrchestrator | RemoveOrchestrator | ReplaceWallet | ReassignBinding | CorrectVolume | SetAdmittedLists | SetPricingParams | ReplaceOwner | Upgrade SRA code
- **Summary:** <what is being asked and why>
- **Requested by:** <Orchestrator | Registry Safes | community issue #<issue>>
- **Repository reference:** (optional)
- **FIP reference (if any):** <FIP-0118 §x.x and commit, or "n/a". A code upgrade requires an accepted FIP.>
- **inherit (ReassignBinding only):** true | false | n/a — <which row of §2.3.11>
- **Recomputation (CorrectVolume only):** <indexer version or pinned FIP commit, events, and blocks. Published before the first approval.>
- **Calldata:** <canonical calldata both Safes copy>
- **Expected taskId:** <keccak256(msg.data)>
- **Independent decode (proposed):** `TODO(owners): approve`. Each Safe posts the decoded function and arguments and the `taskId` it computed, and signs only if both match. Lock the issue after the first approval.
- **Dry-run result:** pending | success | revert — <reason>
- **Bytecode hash (upgrades only):** `TODO(owners): confirm` whether to record one
- **Payout-wallet actor code (AddOrchestrator / ReplaceWallet):** <actor id and code; must not be a payment channel>
- **ReplaceWallet signature (proposed):** `TODO(owners): confirm` a message signed by `orch` (EIP-1271 if `orch` is a contract) naming the old wallet, the new wallet, and the chain id.

- **Rationale for issue:** <the reason recorded on execution — mandatory>
- **Change Log entry:** <link, or "pending">
