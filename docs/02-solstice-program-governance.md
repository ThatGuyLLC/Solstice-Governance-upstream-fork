# 2. Solstice Program Governance

> Part of the [Solstice Governance Repository](../README.md). Protocol rules are fixed by [**FIP-0118** at `23b42964890590a49c9cbc6e19a7e9e361d1f8a0`](https://github.com/filecoin-project/FIPs/blob/23b42964890590a49c9cbc6e19a7e9e361d1f8a0/FIPS/fip-0118.md) and referenced throughout. This section describes *how* the program is governed and operated. Latest text, not used for quotations: [FIP-0118 on `master`](https://github.com/filecoin-project/FIPs/blob/master/FIPS/fip-0118.md).
>
> Quotations of FIP-0118 in this repository are copied from commit [`23b42964890590a49c9cbc6e19a7e9e361d1f8a0`](https://github.com/filecoin-project/FIPs/blob/23b42964890590a49c9cbc6e19a7e9e361d1f8a0/FIPS/fip-0118.md). 

This section consolidates the operating manual for the two governance tiers: the standards behind each Safe, the tasks and actions each tier performs, the registry of admitted Orchestrators, the program parameters, and the safety and rotation playbook.

**Contents**

- [2.1 Governance Tiers and Safes](#21-governance-tiers-and-safes)
- [2.2 SWA Governance Tier — Tasks and Actions](#22-swa-governance-tier--tasks-and-actions)
- [2.3 SRA Governance Tier — Tasks and Actions](#23-sra-governance-tier--tasks-and-actions) (including the Orchestrator Registry at 2.3.12)
- [2.4 Parameters](#24-parameters)
- [2.5 Safety and rotation playbook (both tiers)](#25-safety-and-rotation-playbook-both-tiers)

---

## 2.1 Governance Tiers and Safes

Solstice governance is split across two contracts, each operated by a tier composed of two independent organization multisigs (Safes): **Tier 1, the SWA Governance Tier (§2.2)**, and **Tier 2, the SRA Governance Tier (§2.3)**. FIP-0118 fixes the separation and the approval structure:

> "No address in the registry may participate in either contract governance tier, as a governance multisig or as a key holder within one: a party that set weights could raise its own share, and one that ran the registry could admit itself or block competitors."

Governance is a bespoke UnanimousGovernance contract, not a nested Safe. Each entity's Safe handles co-signing. Where the FIP holds a change before it binds (every discretionary SWA write, and a code upgrade at either tier), the hold and the unilateral cancel are the contract's own logic. Nesting Safes solves co-signing, not objection. An SRA registry change has no post-approval hold and no cancel (§2.1.2). For SWA writes the hold is additionally enforced at f02 (`SWA_TIMELOCK`), so it survives a compromised or maliciously upgraded governance contract. How each owner is implemented internally (incl. a nested multisig) is the entity's operational discretion.

A note on wording: FIP-0118 says "governance multisig"; this repository says "Safe". Both terms refer to the same thing. The FIP identifies each governance multisig by its address and requires no particular wallet product — "no specific multisig implementation (e.g., Safe{Wallet}) is required" (FIP-0118 §4.2). This repository uses Safe because that is what the two tiers deploy, not because the FIP mandates it. Quotations of the FIP keep its wording.

### 2.1.1 Standards for a governance Safe multisig wallet (both tiers)

- Repository disclosure, not a change to that rule: each organization states, in this repository, the entity that holds its Safe and whether that entity or a signer operates, invests in, or is otherwise affiliated with an Orchestrator. Both Safes of the tier must still approve on chain. This repository does not waive the two-Safe approval. `TODO(owners):` publish the entities in the table in [§2.4.1](#241-safe-addresses).
- The organization has a track record in Filecoin governance or engineering, and is independent of the other organization in the same tier.
- Required by this repository, though not enforced by the protocol: an internal threshold of at least 2, and hardware key custody. Signers clear-sign on a hardware key and check the transaction in a tool other than the signing front-end. `TODO(owners):` the transaction simulator, the signing stack, and the indexer release owner. No product is named here. Indexer releases, once published, carry a tag, a checksum, and a reproducible build.
- No registry address may participate in either governance tier, as a governance multisig or as a key holder within one (FIP-0118 §4.2). The FIP's rule is address-level.
- Internal member rotation is org-internal and immediate; it requires no governance action.

Proposed repository rule, not in force. `TODO(owners): approve`. An affiliated signer stands out of that Safe's internal approval of the affiliated Orchestrator's admission, correction, removal, or wallet change. At the signer roster in [§2.4.1](#241-safe-addresses), each governance Safe was 2-of-4, so one signer standing out would leave the threshold reachable. This repository does not waive the two-Safe approval. Until owners approve, no recusal rule is in force.

### 2.1.2 Approval and cancellation rules (both tiers)

These rules are fixed by the FIP and restated here for operators.

> FIP-0118, on the change lifecycle (§4.3). Discretionary changes follow two paths, not one:
>
> > "Every discretionary SWA write is published and held, and either multisig can cancel during the hold; there is no exemption by category at that tier, so reducing a weight to zero is held exactly like removing a stream. SRA registry writes bind at once: their protection is the published rationale plus the replaceability of the tier (Sections 3.2 and 4.1), and the one held SRA action is the code upgrade, held for `SRA_UPGRADE_HOLD` (Section 3.2)."

**Mechanism-executed updates** take no approval and are not cancellable. The quarterly gate check and the `SetShares` recompute run permissionlessly through entry points that can emit only the value the mechanism computes; their integrity guard sits upstream in the FPV verification window. The tiers exercise no judgment over these updates because there is no judgment to exercise.

> FIP-0118 on why mechanism writes cannot be canceled (§4.3):
>
> > "Mechanism-executed updates cannot be canceled: the gate's quarterly w2 write queues in f02 under `SWA_TIMELOCK` for visibility, but it arrives through its own method, `StepWeightRecords`, the one operation `CancelPending` refuses (Section 2.4.7), while the SRA's quarterly share map is never queued and binds directly via `SetShares` (Section 3.2)."

**Discretionary changes** are as follows:

- **Approval:** both of the tier's Safes. SWA writes additionally require a published and accepted FIP. Registry changes deliberately carry no per-change FIP requirement; the carve-out covers data within the rules, never code. An upgrade of either contract's code requires a FIP.
- **Cancellation:** SWA discretionary writes are published and held for SWA_TIMELOCK; either Safe alone can cancel during the hold. SRA registry changes bind at once: both Safes approve, the change emits an on-chain event and carries a mandatory published rationale in this repository. The one exception is an SRA code upgrade, which is held before it binds.

### 2.1.3 Summary of changes

Informative summary; the lifecycle table in the FIP is normative.

| Action | Hold | Enforced by | Cancellable by |
| :-- | :-- | :-- | :-- |
| Discretionary SWA write to f02 (alter, add, or remove a stream; re-point a Distribution) | `SWA_TIMELOCK`, 7 days (FIP-fixed) | f02 | Either SWA Safe |
| SWA internal change (gate parameters, code upgrade) | Internal timelock, equal to the f02 window (FIP-fixed) | SWA | Either SWA Safe via `veto(taskId)` |
| Cooperative Safe replacement (`ReplaceOwner`) | None; binds at once | SWA or SRA | Not cancellable |
| Quarterly gate step | 7-day queue, visibility only | f02 | No one (mechanism-executed) |
| Registry change (add/remove orchestrator, replace wallet, set admitted lists, set pricing, replace owner) | None; binds at once | SRA | Not cancellable |
| Registry / code upgrade | Requires an approved FIP; held for `SRA_UPGRADE_HOLD` (20,160 epochs / 7 days) before it binds | SRA | Either SRA Safe via `veto(taskId)` during the hold |
| CorrectVolume | None; bounded by the verification window | SRA | Not cancellable |
| SetShares, PostVolume, RegisterPairs | None; bounded by the window, the posting period, and the uniqueness check | SRA | Not cancellable |

> FIP-0118 fixes the `SWA_TIMELOCK` hold in L1:
>
> > "f02 itself holds every SWA write for `SWA_TIMELOCK` (7 days, Section 2.4) before applying it, so even a compromised multisig or a maliciously upgraded SWA cannot make a change bind early."

The full compromise and rotation procedures for both tiers are in §2.5, Safety and rotation playbook.

### 2.1.4 Signer conduct (placeholder)

`TODO(owners):` adopt a signer code of conduct and an annual no-payment attestation, published per organization. This repository does not set a rule on payments, employment, or gifts from an Orchestrator to a signer.

---

## 2.2 SWA Governance Tier — Tasks and Actions

> **How to read this subsection:** every individual action below is a collapsible dropdown. Click an action's title to expand only the one you care about.

SWA Governance is **Tier 1**: it governs the Stream Weights Actor (SWA) — the list of streams and the split among them. FIP-0118 fixes its discretionary powers. Cooperative `ReplaceOwner` is the one that needs no FIP. The others require a published and accepted FIP.

SWA Governance discretionary powers are fixed by FIP-0118. These four spans are copied from lines 2490–2500 at the pin:

```
  - add a stream (`RegisterStream`) or remove one (`RemoveStream`); a
    removed stream's weight reverts to the burn residual, so removal
    creates no free value;

  - alter a stream: rewrite its Weight record (`SetWeightRecords`) or
    re-route its Distribution; a w2 record change carries the pairing
    and sequencing duty of Section 3.1.1;

  - tune the gate parameters (`SetGateParams`);

  - upgrade the SWA's code;
```

Lines 2502–2503, the power that does not write to f02 and does not need a FIP:

```
  - cooperatively rotate an SWA multisig (`ReplaceOwner`), binding at
    once; the one power here needing no FIP.
```

SWA Governance approves nothing routine: the w1 ramp and the volume gate are mechanism-executed. Writes that alter, add, or remove a stream, or re-point a Distribution, go to f02, require a published and accepted FIP, and are held for `SWA_TIMELOCK` (7 days) before they bind. `SetGateParams` and an SWA code upgrade are held in the SWA. They do not write to f02, and they still require a published and accepted FIP. Cooperative `ReplaceOwner` binds at once and needs no FIP.

- **During every `SWA_TIMELOCK`:** monitor the f02 queue and the off-chain objection process, and act on a sustained objection within the expected response-time commitment of 7 days.
- **Per discretionary write:** confirm the backing FIP is published and accepted before approval, and verify the schedule envelope (the sum of weights at most 1 across the whole segment) before submission.
- **Continuous:** keep Safe rosters and internal-implementation disclosures in this repository current.

> **Standard SWA discretionary-write flow** (referenced by 2.2.1–2.2.4):
>
> 1. Draft the exact write and carry a FIP through to 'Accepted' (including Last Call). No write is submitted without an accepted FIP.
> 2. Pre-submission check: verify the schedule envelope (Σw ≤ 1 across the whole segment) and that the write matches the accepted FIP.
> 3. Both SWA Safes approve `ProposeWrite(write)`. The first Safe’s transaction only records its approval (`UnanimousGovernance`); the action body runs in the second Safe’s transaction. The second Safe signs only if a dry-run simulation of **its own** approval succeeds, and notes the simulation result on the task-register issue. On a simulated revert, either Safe `veto`s and the action is resubmitted (a failed second approval leaves the task half-approved and burns the signature round). Proposed, not in force. `TODO(owners): approve`. Before either Safe signs, that Safe decodes the calldata with its own tool, posts the decoded function and arguments and the `taskId` it computed, and signs only if both match the human-readable description and the issue's expected `taskId`. After the first approval, lock the issue.
> 4. The SWA relays to f02; f02 queues it with an `effective_epoch` and holds it for `SWA_TIMELOCK` (7 days).
> 5. Objection window: the queued write is public. Monitor objections filed under [§2.2.8](#228--objection-during-an-swa-hold). Either SWA Safe may `CancelPending` / `CancelPendingWeight` / `veto(taskId)` (as applies) on a mismatch with the accepted FIP, a missing FIP, or new information that surfaced after acceptance. Absent a cancellation it binds at `effective_epoch`. `TODO(owners): confirm` that a veto or a cancellation is recorded on the task-register issue with a written reason. The contract does not require the reason.
> 6. Record the outcome in this repository and in the [Program Change Log](06-changelog.md).

The L1 envelope check restates a FIP invariant:

> "the projected weights must satisfy `Σ_{i>=1} w_i(e) <= 1` at every epoch from the write's effective epoch onward, so the burn residual w0 stays non-negative."

<details>
<summary><strong>2.2.1 — Alter a stream: rewrite a Weight record (SetWeightRecords)</strong></summary>

> **Requires:** both SWA Safes + accepted FIP · **Hold:** `SWA_TIMELOCK`, 7 days · **Enforced by:** f02 · **Cancel:** either SWA Safe

Standard flow above; the call is `SetWeightRecords`. Reducing a weight to zero is held exactly like removing the stream. A discretionary write that changes the w2 Weight record must be paired with the matching `steps` adjustment via `SetGateParams` (see §2.2.5 for sequencing and recovery).

</details>

<details>
<summary><strong>2.2.2 — Add a stream (RegisterStream)</strong></summary>

> **Requires:** both SWA Safes + accepted FIP · **Hold:** `SWA_TIMELOCK`, 7 days · **Enforced by:** f02 · **Cancel:** either SWA Safe

Standard flow; the call is `RegisterStream(id, record, activation_epoch)`, or `RegisterStream(id, record, writer, shares, activation_epoch)` where the stream is registered with an initial distribution. Step 2 also confirms the stream and recipient caps and that `activation_epoch ≥ current_epoch + SWA_TIMELOCK`.

</details>

<details>
<summary><strong>2.2.3 — Remove a stream (RemoveStream)</strong></summary>

> **Requires:** both SWA Safes + accepted FIP · **Hold:** `SWA_TIMELOCK`, 7 days · **Enforced by:** f02 · **Cancel:** either SWA Safe

Standard flow; the call is `RemoveStream(id)`. The removed weight reverts to the burn residual; removal creates no free value.

</details>

<details>
<summary><strong>2.2.4 — Re-point a stream's Distribution (SetDistribution)</strong></summary>

> **Requires:** both SWA Safes + accepted FIP · **Hold:** `SWA_TIMELOCK`, 7 days · **Enforced by:** f02 · **Cancel:** either SWA Safe

Standard flow; the call is `SetDistribution(id, writer)`. Can be used, for instance, to re-point the service stream's designated writer to a redeployed SRA (such as the SRA-deadlock backstop). The current wallet-to-share map stays in force until the new writer overwrites it, so payments continue across the change.

</details>

<details>
<summary><strong>2.2.5 — Tune the gate parameters (SetGateParams)</strong></summary>

> **Requires:** both SWA Safes + accepted FIP · **Hold:** SWA-internal timelock, 7 days · **Enforced by:** SWA · **Cancel:** either SWA Safe

1. Publish and get an accepted FIP for the new gate parameters (`VOL_TARGET_ENTRY`, the gate entry target, initially 3,500 USD; `VOL_TARGET_RATIO`, the per-step escalation, initially 2.7; and `steps`, the gate-step counter). There is no free "step size": the 5pp step is a contract constant, and steps indexes the escalation map directly.
2. Both SWA Safes approve `SetGateParams(params)`. A discretionary write that changes the w2 Weight record must go in the same governance action as the matching `steps` adjustment via `SetGateParams`. Per FIP-0118 §3.1.1, sequence the pair so that no `QuarterlyGateCheck` applies between the two effective epochs; the SWA enforces this only when the two holds overlap. Before approval, the issue records both effective epochs and the next `QuarterlyGateCheck` epoch, and states that no gate check falls between them.
3. The change queues in the SWA's own state under the SWA-internal timelock (7 days). The gate rule itself is code and cannot change via a parameter write. Either Safe may cancel during the hold.
4. If one half of a paired write is cancelled after the other has applied, w2 and `steps` stay mismatched until a new paired write. That re-send is authorized only by an accepted FIP. This repository does not pre-authorize it.
5. Record in this repository.

</details>

<details>
<summary><strong>2.2.6 — Upgrade the SWA's code</strong></summary>

> **Requires:** both SWA Safes + accepted FIP · **Hold:** SWA-internal timelock, 7 days · **Enforced by:** SWA · **Cancel:** either SWA Safe

Same flow as 2.2.5; the FIP carries the upgrade. It executes through the pre-upgrade code, so the hold and checks still bind. The upgrade issue pins the [solstice upgrade runbook](https://github.com/filecoin-project/solstice/blob/b9a63a87b5adee272c16d7e507cc9c127fb34f0f/docs/UPGRADE.md) by commit (pin `b9a63a87b5adee272c16d7e507cc9c127fb34f0f`; [latest on `main`](https://github.com/filecoin-project/solstice/blob/main/docs/UPGRADE.md) is not the pin). FIP-0118 §4.2 requires that the pending upgrade and its execution be observable on chain. `TODO(owners): confirm` whether the issue must also record an implementation bytecode hash. 

</details>

<details>
<summary><strong>2.2.7 — Cancel a pending SWA write</strong></summary>

> **Requires:** either SWA Safe alone · **When:** during the hold/window

Either Safe can stop a bad SWA change on the matching path:

1. `CancelPending(id, op)` — queued stream operation on f02 (relayed to `f02.CancelPending`).
2. `CancelPendingWeight(op)` — queued weight write on f02.
3. `veto(taskId)` — a task still inside the SWA (half-approved, or in the SWA-internal hold).

A cancellation only preserves the status quo. Mechanism writes (the gate step via `StepWeightRecords`) cannot be cancelled: `CancelPending` refuses that operation (FIP-0118 §2.4.7 and §4.3). Record the cancellation on the task-register issue. `TODO(owners): confirm` whether that record includes a written reason. 

</details>

<details>
<summary><strong>2.2.8 — Objection during an SWA hold</strong></summary>

The objection channel is an issue in this repository, using the community issue template, with nature "Objection to a pending SWA write". It is open to anyone during the hold.

The issue states the chain id, the queued write (`taskId` or the f02 operation), the accepted FIP (link and commit), and which of the FIP's cancel grounds is claimed: the queued write does not match the accepted FIP, the FIP is missing, or new information surfaced after acceptance (FIP-0118 §4.3). 

A **sustained objection** is an objection that states one of those grounds, is filed here during the hold, and is not withdrawn. It does not itself cancel the write. Either SWA Safe may cancel. Whether to cancel is that Safe's decision. `TODO(owners): confirm` that a cancellation, and a decision not to cancel, is recorded on the issue with a written reason. The contract does not require the reason.

The response-time commitment is a maximum of 7 days, published in this section and in [§2.4.2](#242-governed-parameters). It is how quickly a Safe acts on an objection during the hold. It does not extend `SWA_TIMELOCK`.

SRA registry writes have no objection cancel. They bind when the second Safe approves (FIP-0118 §4.3). This section does not add a cancel path for them.

</details>

<details>
<summary><strong>2.2.9 — Monitor mechanism-executed updates</strong></summary>

The w1 ramp and the quarterly gate step (`QuarterlyGateCheck` → `StepWeightRecords`) run permissionlessly. Any party may crank `QuarterlyGateCheck`. Nothing on chain schedules it. A late crank delays the stepped w2 by the same amount, and f02 burns the difference between the ceiling and the standing w2 while the delay lasts (FIP-0118 §2.2). Each quarter:

1. Confirm the queued w2 write matches the gate rule and the bound `AggregatedFPV(Q)`.
2. Note it is a scheduled write: it queues 7 days for visibility only and is not cancellable.
3. Record the crank (chain id, quarter, transaction) on the quarter's issue. Repository deadline, not a FIP deadline: crank within 24 hours of the epoch the call becomes available. `TODO(owners):` name a primary and a backup cranker. This repository does not name them.
4. If a mismatch suggests compromise, escalate per §2.5, Safety and rotation playbook; the remedy will be a Safe/code replacement, not cancelling the mechanism write.

</details>

<details>
<summary><strong>2.2.10 — Continuous upkeep</strong></summary>

During every `SWA_TIMELOCK`, monitor the f02 queue and objections under [§2.2.8](#228--objection-during-an-swa-hold), and act within the 7-day response-time commitment published there and in [§2.4.2](#242-governed-parameters). Keep Safe rosters and internal-implementation disclosures current.

</details>

---

## 2.3 SRA Governance Tier — Tasks and Actions

> **How to read this subsection:** every individual action below is a collapsible dropdown. Click an action's title to expand only the one you care about. The [Orchestrator Registry](#2312-orchestrator-registry-admitted-orchestrators) sits at the bottom (2.3.12).

SRA Governance is **Tier 2**: it governs the Service Rewards Actor (SRA) — the orchestrator registry and the split within the service stream. FIP-0118 fixes its discretionary powers; registry changes require no per-change FIP except a code upgrade:

SRA Governance's discretionary powers are fixed by FIP-0118 §4.2. They are: admit or remove an Orchestrator; replace an Orchestrator's payout wallet on its public request; cooperatively rotate a Registry multisig (`ReplaceOwner`); reassign contested (payer, operator) bindings; correct or supply posted volumes during the verification window (`CorrectVolume`); audit Orchestrators on an exception basis; maintain the admitted lists that scope FPV; set the FIL pricing parameters and `REGISTRATION_CUTOFF` via `SetPricingParams` (§2.3.7); and upgrade the SRA's code, the one registry change that requires a FIP.

Registry changes need both Registry Safes but need no per-change FIP (the one exception is a code upgrade). The quarterly verification runs against a public record and is deterministic.

> **Standard registry-change flow** (referenced by 2.3.1–2.3.5 and by the Safe replacement in 2.5.3):
>
> 1. Trigger and diligence (application, dispute outcome, audit finding, rotation request, or list update).
> 2. Both Registry Safes approve the relevant call. The first Safe’s transaction only records its approval (`UnanimousGovernance`); the action body runs in the second Safe’s transaction (SRA header: the second Safe MUST dry-run the calldata before approving). The second Safe signs only if a dry-run simulation of **its own** approval succeeds, and notes the simulation result on the task-register issue. On a simulated revert (wrong wallet, closed window, pending shares, etc.), either Safe `veto`s and the action is resubmitted. `TODO(owners): confirm` that a veto is recorded on the issue with a written reason. The contract does not require the reason. Proposed, not in force. `TODO(owners): approve`. Before either Safe signs, that Safe decodes the calldata with its own tool, posts the decoded function and arguments and the `taskId` it computed, and signs only if both match the human-readable description and the issue's expected `taskId`. After the first approval, lock the issue. 
> 3. The change binds when the second SRA Safe approves (no pending queue, no cancellation path). Record it in the Change Log.
> 4. Both Registry Safes can see the submitted task. The `Submitted` event carries the `taskId`, not the full content; the issue carries the content.
> 5. Record the outcome in the issue/repository, and update the [Orchestrator Registry](#2312-orchestrator-registry-admitted-orchestrators) where the change affects an Orchestrator's status.

The recomputation is deterministic: any observer running the reference indexer over the same settlement events and registry state reaches the same figure.

### Admission (Phase 1: discretionary)

An Orchestrator's application is filed as an issue in this repository. If application is approved, SRA Governance executes the admit on-chain (action 2.3.1). **Every admitted Orchestrator is recorded in the [Orchestrator Registry (2.3.12)](#2312-orchestrator-registry-admitted-orchestrators).**

Phase 1 admission is discretionary (FIP-0118 §4.3). 
A uniqueness rule is fixed by the FIP:

> "Indeed, the SRA enforces that each (payer, operator) pair is bound to at most one Orchestrator, so a registration that duplicates an existing binding reverts."

For Phase 2 (subject to a future FIP): admission becomes permissionless, enabled by a standard attribution-metadata interface specified in a future FIP. The rubric and diligence apply to Phase 1 only.

<details>
<summary><strong>2.3.1 — Admit an Orchestrator (AddOrchestrator)</strong></summary>

> **Requires:** both SRA Safes · **Hold:** none · **Enforced by:** SRA · **Cancel:** not cancellable (either Safe may `veto(taskId)` only while half-approved) · **No FIP**

1. Application filed as an issue (identity/team, funding plan, declared (payer, operator) pairs and measurement rules). The issue states the chain id. The application asks whether the applicant, or a person who controls it, has operated as a Fil+ allocator, notary, or client, and whether any public finding about Fil+ deal patterns applies. That question is diligence. It is not a bar. The application also discloses common control with any other admitted Orchestrator. `TODO(owners):` whether a registration by a commonly controlled Orchestrator of pairs released by a removed sibling is treated as wrongful and corrected by `ReassignBinding` by default. This repository does not set that default. Beneficial ownership and KYC: `TODO(owners):` the standard, pending counsel. The application has a slot for it. This repository does not define a standard and does not treat the slot as satisfied.
2. Resolve the payout wallet to an actor ID. It must exist, and it must not be a payment channel. f02 rejects payment-channel recipients on `SetShares` and `ReplaceAddress` (FIP-0118 §2.4.4). Record the actor code on the issue. A payment-channel wallet can still break `SubmitShares` for every Orchestrator; see §2.3.3.
3. Both Registry Safes approve `AddOrchestrator(orch, wallet)` using the task-register issue’s canonical calldata; the uniqueness rule reverts any pair already bound elsewhere.
4. Either Safe may `veto(taskId)` while the task is half-approved. Once both have approved, the admission is not cancellable.
5. Binds and applies at once: the Orchestrator may register pairs and post from the admission epoch (FIP-0118 §3.2). Record it in the issue **and add the Orchestrator to the [Orchestrator Registry (2.3.12)](#2312-orchestrator-registry-admitted-orchestrators)**.
6. A zero-volume quarter does not install a new share map. FIP-0118 lines 2299–2303 at the pin:

```
        If eligible FPV for that quarter sums to zero, `SubmitShares` is
        a benign no-op: `SplitRule` is not evaluated, the existing share
        map stands, including the burn share an earlier removal created,
        and the quarter still counts as submitted (so it cannot be
        resubmitted, and the `RemoveOrchestrator` guard lifts).
```

A newly admitted Orchestrator is not paid until a quarter with eligible volume binds. Through a zero-volume quarter the share map already in place, including the launch seat's map, remains. That lag is FIP behavior. This repository does not change it.

</details>

<details>
<summary><strong>2.3.2 — Remove an Orchestrator (RemoveOrchestrator)</strong></summary>

> **Requires:** both SRA Safes · **Hold:** none · **Enforced by:** SRA · **Cancel:** not cancellable (either Safe may `veto(taskId)` only while half-approved) · **No FIP**

`RemoveOrchestrator` is the single sanction (FIP-0118 §3.2). It is not reversed in place. From the epoch it binds, the Orchestrator's future service-stream income is re-pointed to `f099` and burns until the next `SubmitShares`. Amounts already accrued stay claimable. The removal releases the Orchestrator's (payer, operator) bindings, and those pairs can be registered again. A new registration's back-cover reaches the release epoch only, not the span the removed Orchestrator served. Update the Orchestrator Registry to mark the entry *Removed*.

The call reverts while an ended quarter's share map is still pending. Crank `SubmitShares` first and call `RemoveOrchestrator` in the next message (FIP-0118 §3.2). An Orchestrator that expects a removal can `Claim` first. Honest Orchestrators limit that exposure by claiming often (FIP-0118, Security Considerations).

Expect other Orchestrators to race for the released pairs. A registration the dispute process finds wrongful is corrected by [§2.3.11](#2311--reassign-a-binding-reassignbinding).

Repository explanation of FIP timing, not a new protocol rule. A removal excludes that quarter's FPV from the share map and from `AggregatedFPV` only when it binds before `Start(Q+1)` (FIP-0118 §2.2, lines 746–750 at the pin). `RemoveOrchestrator` reverts from `Start(Q+1)` until that quarter's `SubmitShares` has run (FIP-0118 §3.2, lines 2319–2324 at the pin). Wash or self-dealt volume that is still in place at `Start(Q+1)` is inside the bound figure. During the last `REGISTRATION_CUTOFF` of each quarter, SRA Governance records new `RegisterPairs` on the quarter's issue so a late registration is visible before the quarter ends. Whether `CorrectVolume` may exclude volume a dispute has called circular is a question for the FIP, not a rule of this repository.

Proposed, not in force. `TODO(owners): approve`. Ask the outgoing Orchestrator to notify affected payers, and post that notice on the issue. `TODO(owners):` launch-seat removal criteria, and whether to aim for a second Orchestrator before the end of Q2. This repository does not set those criteria. 

A later return, where one is allowed, is a fresh `AddOrchestrator` with a fresh application. Failure to file the quarterly report forfeits re-admission for the following quarter (FIP-0118 §4.2), so a fresh application is not available in that quarter. 

</details>

<details>
<summary><strong>2.3.3 — Replace / rotate an Orchestrator address (ReplaceWallet)</strong></summary>

> **Requires:** both SRA Safes · **Hold:** none · **Enforced by:** SRA · **Cancel:** not cancellable (either Safe may `veto(taskId)` only while half-approved) · **No FIP**

`ReplaceWallet` is available only on a public request from the Orchestrator recorded in this repository (FIP-0118 §3.2). No approval without that request. A replacement with no matching request is the capture the FIP calls hostile on sight.

1. The Orchestrator files an issue identifying `orch`, the new payout wallet, and the chain id. Proposed, not in force. `TODO(owners): confirm` that the request also carries a message signed by `orch` (EIP-1271 when `orch` is a contract, including a Safe) naming the old wallet, the new wallet, and the chain id. Until that is confirmed, the Safes do not treat an unsigned GitHub issue as the public request.
2. Resolve the new wallet to an actor ID. It must exist and must not be a payment channel (same check as §2.3.1). Record the actor code on the issue.
3. Both Registry Safes approve `ReplaceWallet(old, new)` on the issue's canonical calldata. Per FIP-0118 §3.2, the call swaps the payout wallet through an immediate `ReplaceAddress`, so f02 pays the new wallet at once. The Orchestrator identity does not change, and accrued balances are not moved. Update the payout-wallet field in the Orchestrator Registry.
4. If the `orch` identity cannot file the request, this procedure does not run. There is no `ReplaceWallet` without the public request. `TODO(owners):` any out-of-band identity verification for a lost `orch` key, used only toward a fresh admission or a removal review. This repository does not define one.

**Payment-channel wallet recovery.** A payout wallet that is a payment channel can pass an actor-exists check but breaks `SubmitShares` / `SetShares` for **every** Orchestrator (Rod’s devnet). `RemoveOrchestrator` also reverts while a quarter’s share map is still pending (`PendingShares`). Exit order:

1. `ReplaceWallet` to a non-payment-channel wallet, on the public request above (and re-check actor code).
2. `SubmitShares` for the pending quarter (now that every wallet in the map is admissible).
3. `RemoveOrchestrator` if the Orchestrator is being exited (only after shares are no longer pending).

</details>

<details>
<summary><strong>2.3.4 — Quarterly FPV verification & correction (CorrectVolume)</strong></summary>

> **Requires:** both SRA Safes (jointly) · **Hold:** none; bounded by the verification window · **Enforced by:** SRA · **Cancel:** not cancellable

1. Recompute every admitted Orchestrator's `FPV_i(Q)`, posted or not, from public settlement events and registry state. Use the versioned reference indexer when it is published. Until then, the normative algorithm is FIP-0118 §2.2 at the commit pinned in the [README](../README.md). This recomputation is an SRA duty every quarter. It is not triggered only by a dispute (FIP-0118 §2.2 and Security Considerations).
2. Publish the recomputation on the task-register issue before either Safe approves: chain id, indexer version or the pinned FIP commit, the settlement and registry events, and the blocks covered. Proposed repository rule, not a FIP requirement and not in force. `TODO(owners): approve` a minimum of 48 hours between that publication and the first approval. `TODO(owners):` who watches for a correction that changes whether the gate passes. 
3. During the verification window, both Safes jointly call `CorrectVolume(orch, Q, value)` to replace a misreport or supply a figure for a non-poster. A value neither posted nor supplied binds at zero (conservative under-count). The issue carries the canonical calldata and the expected `taskId` before the first approval. An upward correction, or a figure supplied for an Orchestrator that did not post, uses the same publication and states why the figure is the recomputed value. The FIP allows both directions. 
4. **One Safe can block a correction.** `CorrectVolume` needs both Registry Safes. The window check runs in the function body after approvals are stored (`ServiceRewardsActor.correctVolume`), so a second approval after the window closes reverts and the first approval stays. Aim for both approvals by day 5 of the 7-day mainnet window (scale the same fraction on calibnet). That aim is not a protocol deadline. The calendar dates in [§2.4.2](#242-governed-parameters) are estimates from epochs. A chain halt moves the wall-clock dates, and a second approval included near the window close can still be reorganized out. Signatures should land by day 5 of the window so the second approval is not left at the edge. `TODO(owners): confirm` a hard internal deadline of day 5, with a public notice on the issue if the second signature has not landed by then. Until confirmed, day 5 stays an aim. 
5. **If the second approval misses the window,** the posted figure, or zero if nothing was posted, binds (FIP-0118 §2.2). The first approver `veto`s the stale task. The Safes publish a "correction blocked" note on the same issue: the Orchestrator, the recomputed figure, and that it did not bind. `TODO(owners): confirm` that the note is a verification trigger under [§4.4.4](04-quarterly-review-and-runbook.md#444-what-triggers-a-verification) without a further owner confirmation. Until confirmed, the note is published and is not by itself a removal ground. If the bound figure passed the gate, reversing the standing w2 step takes a new FIP-backed SWA write paired with the matching `steps` adjustment (FIP-0118, Security Considerations, and §2.2.5). This repository does not pre-authorize that FIP. `TODO(owners):` a reversal template, if one is wanted before it is needed. Proposed, not in force. `TODO(owners): approve` publishing `MIN_LOT(2)` and its inputs (`BoundVolume` of Q1, `N` of Q1, `MIN_LOT_FLOOR`, `MIN_LOT_ALPHA`) on the Q1 issue when Q1 closes.
6. Values bind when the window closes. There is no separate cancellation of a completed correction.
7. A contested correction is appealed via the [dispute process](04-quarterly-review-and-runbook.md#442-dispute-resolution-for-contested-bindings). The outcome affects only later quarters.

FIP-0118 fixes the binding behavior:

> "Binding: whatever each value is when the window closes binds, and bound values are final for the quarter."
>
> "A value neither posted nor supplied binds as 0, so a non-poster cannot block the quarter."

</details>

<details>
<summary><strong>2.3.5 — Dispute resolution for contested bindings</strong></summary>

1. The contesting party opens an issue identifying the pair and its claim, and the chain id. A dispute that asks for a `CorrectVolume` in the open verification window is filed by day 4 of that window. The written reason is recorded with the outcome. `TODO(owners):` an independent appeal reviewer. This repository does not name one.
2. SRA Governance requests evidence from both parties (increasing strength: business records; client confirmation via a verifiable channel; a signed payer-wallet attestation naming the orchestrator).
3. Decide the outcome and record a written reason on the issue. A reassignment executes via [§2.3.11](#2311--reassign-a-binding-reassignbinding). A removal executes via §2.3.2. Neither is a `CorrectVolume` (§2.3.4).
4. Record the resolution in the issue.

Full procedure and evidence standards: [§4.4.2](04-quarterly-review-and-runbook.md#442-dispute-resolution-for-contested-bindings).

</details>

<details>
<summary><strong>2.3.6 — Exception-based verification (removal)</strong></summary>

1. Trigger: an anomaly in public settlement data, a community report, or a contested binding. This exception audit is not the quarterly recomputation in §2.3.4, and it is not pre-clearance of deals (FIP-0118, Security Considerations).
2. Investigate: recompute FPV and examine the (payer, operator) relationships. The terms are defined in [§4.4.3](04-quarterly-review-and-runbook.md#443-verification-and-removal).
3. Decide grounds. Grounds are listed in [§4.4.3](04-quarterly-review-and-runbook.md#443-verification-and-removal). They include misreported FPV, a pattern of wrongful claims (FIP-0118, Security Considerations), and, as repository policy, wash trading and self-dealing. Inactivity is a ground only once `TODO(owners)` in [Policy 7](03-orchestrator-operational-guidelines.md#31-policies) sets the number of consecutive quarters. `TODO(owners):` whether "binding fraud" is a ground distinct from misreported FPV and wrongful claims. This repository does not define it. 
4. Act: `RemoveOrchestrator` (§2.3.2) via the standard registry-change flow. Either Safe may `veto(taskId)` only while the task is half-approved. Once both Safes have approved, removal is not cancellable.
5. Record the finding and rationale.

</details>

<a id="237--maintain-admitted-lists--fee-auction-parameters-setadmittedlists-setpricingparams"></a>

<details>
<summary><strong>2.3.7 — Maintain admitted lists & fee-auction parameters (SetAdmittedLists, SetPricingParams)</strong></summary>

> **Requires:** both SRA Safes · **Hold:** none · **Enforced by:** SRA · **Cancel:** not cancellable (either Safe may `veto(taskId)` only while half-approved) · **No FIP**

Standard registry-change flow for the admitted-stablecoin whitelist, the admitted Filecoin Pay contract addresses, and the fee-auction pricing parameters, set by `SetPricingParams`(`min_lot_floor`, `min_lot_alpha`, `price_band`, `registration_cutoff`). All four arguments are supplied on every call, so a change to one restates the other three — the task-register issue must carry the full tuple, not the delta. These are gate-consequential. The calls bind at once when the second Safe approves: each emits an event and stores nothing, and neither is held or cancellable after that approval. Either Safe can `veto(taskId)` on a half-approved task before the second approval. Never a silent edit.

Both the admitted lists and the pricing parameters, including `REGISTRATION_CUTOFF`, apply from the first epoch of the next quarter, not from the execution epoch (FIP-0118 §2.2 and §3.2). The latest event before a quarter's first epoch is the set in force for that quarter. An Orchestrator does not apply a list or parameter change mid-quarter. For `SetAdmittedLists`, sort each address array ascending before encoding so both Safes share one `taskId` (see task-register canonical calldata). The issue states the chain id.

Repository rule for `SetPricingParams`. Publish the full tuple and a written rationale on the issue at least 14 days before the first approval. `TODO(owners):` the allowed ranges for `MIN_LOT_FLOOR`, `MIN_LOT_ALPHA`, `PRICE_BAND`, and `REGISTRATION_CUTOFF`. This repository does not set numeric ranges.

</details>

<details>
<summary><strong>2.3.8 — Upgrade the SRA's code</strong></summary>

> **Requires:** both SRA Safes + accepted FIP · **Hold:** `SRA_UPGRADE_HOLD`, 7 days · **Enforced by:** SRA · **Cancel:** either SRA Safe via `veto(taskId)` during the hold

The one registry change that requires a FIP (e.g., the Phase 2 permissionless-admission transition). Otherwise follows the standard flow. The upgrade issue pins the [solstice upgrade runbook](https://github.com/filecoin-project/solstice/blob/b9a63a87b5adee272c16d7e507cc9c127fb34f0f/docs/UPGRADE.md) by commit (pin `b9a63a87b5adee272c16d7e507cc9c127fb34f0f`; [latest on `main`](https://github.com/filecoin-project/solstice/blob/main/docs/UPGRADE.md) is not the pin). FIP-0118 §4.2 requires that the pending upgrade and its execution be observable on chain. `TODO(owners): confirm` whether the issue must also record an implementation bytecode hash. 

</details>

<details>
<summary><strong>2.3.9 — Monitor mechanism-executed updates (oversight, no approval)</strong></summary>

`SubmitShares()` runs permissionlessly and is not held (FPV is posted already USD-denominated; there is no separate on-chain conversion pass). Any party may crank it. Nothing on chain schedules it. A late crank delays the new share map; during the delay the previous map stands, and after a removal the removed share keeps burning until the new map is installed (FIP-0118 §2.2 and §3.2). The `RemoveOrchestrator` guard stays up until `SubmitShares` has run.

Duty: confirm `SubmitShares` has run and wrote the correct wallet-to-share map (integrity rests on the upstream verification window). It is not cancellable. Record the crank (chain id, quarter, transaction) on the quarter's issue. Repository deadline, not a FIP deadline: crank within 24 hours of the epoch the call becomes available. `TODO(owners):` name a primary and a backup cranker. This repository does not name them. 

</details>

<details>
<summary><strong>2.3.10 — Continuous upkeep</strong></summary>

Maintain and version the reference indexer; monitor anomaly reports, contested bindings, and the cancellation queue; keep Safe rosters, disclosures, and the Orchestrator Registry current. The reference indexer is not yet published. Until it is, recomputation follows the FIP text pinned in the [README](../README.md). Indexer releases, once published, carry a tag, a checksum, and a reproducible build. `TODO(owners):` the indexer release owner, in [§1.4 of the README](../README.md).

Proposed, not in force. `TODO(owners): approve` a quarterly snapshot of `getOwners` and `getThreshold` for every governance Safe and every Orchestrator Safe, published on the quarter's issue, with a diff against the previous snapshot.

Proposed, not in force. `TODO(owners): approve` a quarterly on-chain read of the deployed parameters (`MIN_LOT_FLOOR`, `MIN_LOT_ALPHA`, `PRICE_BAND`, `REGISTRATION_CUTOFF`, and the admitted-list events), recorded on the quarter's issue and compared with [§2.4.2](#242-governed-parameters). The table there is a mirror. It is not a live read. 

</details>

<details>
<summary><strong>2.3.11 — Reassign a binding (ReassignBinding)</strong></summary>

> **Requires:** both SRA Safes · **Hold:** none · **Enforced by:** SRA · **Cancel:** not cancellable once both have approved (either Safe may `veto(taskId)` only while half-approved) · **No FIP**

`ReassignBinding(pair, orch, inherit)` moves a (payer, operator) pair. `inherit` states how far the new binding reaches, and the event carries it so every verifier applies the same scope (FIP-0118 §3.2). `CancelBinding` is not this call. The bound Orchestrator releases its own pair with `CancelBinding`; see [§3.2.8](03-orchestrator-operational-guidelines.md#328--release-a-binding-cancelbinding).

Which disputes use `inherit = true` is this repository's guidance, as the FIP asks. The rule follows the FIP's own defaults. It does not add cases the FIP does not describe.

| Situation | What runs | `inherit` |
| :-- | :-- | :-- |
| A client changes Orchestrator, and the change is not a wrongful claim | The departing Orchestrator calls `CancelBinding`. The new Orchestrator calls `RegisterPairs`. SRA Governance calls `ReassignBinding` only when the pair must move without that cooperative release. | `false`. The previous holder keeps the span it served. The new binding applies from the execution epoch. |
| A claim adjudicated wrongful, and the displaced binding's application epoch is in the current quarter or later | `ReassignBinding` to the Orchestrator that serves the client. | `true`. The new binding adopts the displaced registration's execution epoch, including back-cover and `REGISTRATION_CUTOFF` computed from that epoch. The wrongful claimant keeps nothing of that span. |
| The displaced binding already applied in an earlier quarter | `ReassignBinding` | The contract applies the move from the execution epoch, whatever argument is passed. An established Orchestrator does not lose a span it already served. |
| The pair is unbound at execution | `ReassignBinding` | Either argument. The contract applies the binding from the execution epoch. Nothing is displaced. |
| The quarter is already closed | No call reaches it | Out of reach (FIP-0118 §2.2). The resolution, like an appeal, applies prospectively. |

A closed quarter is out of reach either way. `inherit = true` is honored only when the displaced binding's application epoch falls in the current quarter or later (FIP-0118 §3.2).

The task-register issue states the chain id, the pair, the new Orchestrator, the `inherit` value, why that row of the table applies, the canonical calldata, and the expected `taskId`, before the first approval. Record the written reason on the issue.

Proposed, not in force. `TODO(owners): approve`. Ask the outgoing Orchestrator and the incoming Orchestrator to notify affected payers, and post that notice on the issue.

</details>

### 2.3.12 Orchestrator Registry (admitted Orchestrators)

This is the canonical, human-readable list of Orchestrators admitted to the Solstice program. It is maintained by SRA Governance and mirrors on-chain SRA registry state — **the on-chain registry is the source of truth**; this list is a convenience view and audit trail. Entries are added and updated only through the SRA registry actions above: an Orchestrator appears on Admit (2.3.1), has its wallet updated on Replace (2.3.3), and is marked *Removed* on Remove (2.3.2).

**Initial Orchestrator at activation.** FIP-0118 seats an Orchestrator at activation and requires it to “publish a disclosure on activation, following the requirements in the governance repository set for any other Orchestrator.” That disclosure is on activation, not within 7 days of an `AddOrchestrator` admission ([§3.1, Policy 6](03-orchestrator-operational-guidelines.md#31-policies)). That Orchestrator is not admitted through an Admit action (2.3.1). On activation it publishes the same declaration file required of any other Orchestrator, and SRA Governance sets its registry row below to **Active** with admission = quarter 1 (identity and payout wallet `0x97A90f5696be5E3C8d3752C92Adac287c2b4484e`, matching `initialOrchestrator` / `initialOrchestratorWallet` in [`deployments.json` at `b9a63a87b5adee272c16d7e507cc9c127fb34f0f`](https://github.com/filecoin-project/solstice/blob/b9a63a87b5adee272c16d7e507cc9c127fb34f0f/deployments.json); [latest on `main`](https://github.com/filecoin-project/solstice/blob/main/deployments.json) is not the pin). The disclosure stub is [activation-seat.md](disclosures/activation-seat.md). `TODO(owners):` complete it before 2026-10-19 13:00 UTC, including the f1 claim keeper. The activation checklist is [activation-day-2026-10-19.md](activation-day-2026-10-19.md).

The row below is chain id **314** (Filecoin mainnet). As of solstice `deployments.json` commit `b9a63a87b5adee272c16d7e507cc9c127fb34f0f`, the same address is `initialOrchestrator` and `initialOrchestratorWallet` on calibnet (chain id 314159). A mainnet action names chain id 314. The same address string is not a check that the signer is on mainnet.

Removing the sole admitted Orchestrator burns the whole service stream from the epoch the removal binds until a later `SubmitShares` with eligible volume (FIP-0118, Security Considerations). That is the launch configuration.

**Status legend**

| Status | Meaning |
| :-- | :-- |
| **Active** | Admitted and operating; can `RegisterPairs`/`PostVolume`; FPV counts toward `SplitRule` and `AggregatedFPV`. |
| **Removed** | Permanently removed; (payer, operator) bindings released for re-registration (2.3.2). |

**Admitted Orchestrators**

> **Program status:** the activation seat is recorded below; further Orchestrators appear when SRA Governance executes an `Admit` action (2.3.1). The table shows the columns each entry must carry.

| # | Orchestrator | Identity (`orch`) | Payout wallet | Status | Admitted (quarter / epoch) | Declaration | Registered (payer, operator) pairs |
| :-- | :-- | :-- | :-- | :-- | :-- | :-- | :-- |
| 1 | Initial (activation seat) | 0x97A90f5696be5E3C8d3752C92Adac287c2b4484e | 0x97A90f5696be5E3C8d3752C92Adac287c2b4484e | Active | quarter 1 | TBD ([disclosure stub](disclosures/activation-seat.md)) | — |

**Per-Orchestrator entry template**

When admitting an Orchestrator, add a row to the table above and a detailed entry using this template:

```markdown
### <Orchestrator name>

- **Identity (`orch`):** <f4… / 0x… address>
- **Payout wallet:** <f4… / 0x… address> (may differ from identity)
- **Status:** Active | Removed
- **Admitted:** <quarter, epoch, tx hash>
- **Declaration issue:** #<issue number> (link)
- **Contact:** <public contact per the §3.1 responsiveness rule>
- **Recovery contact:** <a second contact, distinct from the identity key>
- **Registered (payer, operator) pairs:** <pointer to on-chain registry / list>
- **Per-pair affiliation (proposed):** `TODO(owners): approve` a flag on each pair for whether the Orchestrator or an affiliate controls the payer or the operator. Not required until approved.
- **Common control with another admitted Orchestrator:** <disclose, or "none">
- **Measurement rules:** <short description or link to service-contract metadata>
- **History:** <admit / replace / remove events, each with date + tx>
```

> Field definitions follow the Orchestrator guidelines: identity and payout wallet in [§3.2](03-orchestrator-operational-guidelines.md#32-orchestrators-tasks-and-actions), the declaration file in [§3.1, Policy 6](03-orchestrator-operational-guidelines.md#31-policies), and the (payer, operator) binding rules in [§3.1, Policy 3](03-orchestrator-operational-guidelines.md#31-policies) (FIP-fixed uniqueness).

---

## 2.4 Parameters

### 2.4.1 Safe Addresses

Each tier consists of two organization multisigs (Safes) registered in the contract it governs. The protocol sees only the two addresses.

| Tier | Contract governed | Organization 1 Safe (address) | Organisation 2 Safe (address) | Rule (fixed by the FIP) |
| :-- | :-- | :-- | :-- | :-- |
| SWA Governance (§2.2) | Stream Weights Actor (SWA) | 0x4d6db5600c908b3C9b9888cFef43f27F38b203e9 | 0x024a3c8CCA435db64D2dfa0f903E4823A5eBdf63 | Both approve; either alone cancels |
| SRA Governance (§2.3) | Service Rewards Actor (SRA) | 0x6c724FF811f51945d95872CF99FBd3fF63f7cFa2 | 0xFb1B58925947E52B3f75BAc3D9fB5325cfb36371 | Both approve; binds at once (not cancellable; SRA code upgrades held, either Safe may veto during the hold) |

Each organization runs a separate Safe per tier — four accounts in total — so approvals cannot be replayed across surfaces. That sentence describes four accounts. It does not say whether the same two legal entities sit in both tiers. FIP-0118 §4.2 requires one multisig per entity per tier. It permits an entity to hold one multisig in each tier, and it does not name the entities. This repository does not assert that the four Safes are or are not held by the same two organizations.

**Governance organizations.** Entity cells are blank on purpose. `TODO(owners):` name each entity and its affiliations.

| Safe | Address | Entity | Affiliation disclosure |
| :-- | :-- | :-- | :-- |
| SWA Organization 1 | 0x4d6db5600c908b3C9b9888cFef43f27F38b203e9 | `TODO(owners):` | `TODO(owners):` |
| SWA Organisation 2 | 0x024a3c8CCA435db64D2dfa0f903E4823A5eBdf63 | `TODO(owners):` | `TODO(owners):` |
| SRA Organization 1 | 0x6c724FF811f51945d95872CF99FBd3fF63f7cFa2 | `TODO(owners):` | `TODO(owners):` |
| SRA Organisation 2 | 0xFb1B58925947E52B3f75BAc3D9fB5325cfb36371 | `TODO(owners):` | `TODO(owners):` |

**Signer roster.** Read from chain at mainnet block 6,458,454 (`getOwners` / `getThreshold`). Threshold was 2 on each Safe. This table does not assert that a repeated address is the same legal entity. `TODO(owners):` Organization 1 splits its signers per tier, or discloses the overlap. A rule that the two tiers must be different entities would be a FIP change. This repository does not make that change.

| Safe | Signers at block 6,458,454 |
| :-- | :-- |
| SWA Organization 1 | `0x7d7969fe7d54c538fafa4347c7fa3871ac664ae7`, `0xd73811c4589194278232e24a89bbcf6bfc23717b`, `0xc998dc48e880c5608fd9aed4d05f8ea96b9b8a61`, `0xe499994b84b5793eb4fe3e66699debd1819762db` |
| SWA Organisation 2 | `0x8a63434e12048ec0a64b63fb35d26e9fe5819a4c`, `0x615ed6180e6a1b2054ca1a747c47d26d8a11d244`, `0x98bfc9628656afa091ae45f54d7e656064ace534`, `0xcd7731927a96a09aa41c5e57bf1629e9797c7981` |
| SRA Organization 1 | `0x7d7969fe7d54c538fafa4347c7fa3871ac664ae7`, `0xe07d859ca26e695339a2dc86a2612a58102523cd`, `0xe499994b84b5793eb4fe3e66699debd1819762db`, `0x0930251729602b294bce0418dfbe547654286af7` |
| SRA Organisation 2 | `0x6a33dac999d2f7b716cfb7c44ef5df78cefea43e`, `0x00878e3616f3bef2b5005786d87a4eaa5760773b`, `0x124136cdfd04a912f8db4b0b1e55b89d2dca1af4`, `0x5f50a20e6089d9622f158121f1b76e7663ea476f` |

Two addresses, `0x7d7969fe7d54c538fafa4347c7fa3871ac664ae7` and `0xe499994b84b5793eb4fe3e66699debd1819762db`, were owners of both Organization 1 Safes at that block. Together they meet the threshold of 2 on each of those Safes. The Organisation 2 Safes shared no owners at that block.

Addresses match [`filecoin-project/solstice` `deployments.json` at `b9a63a87b5adee272c16d7e507cc9c127fb34f0f`](https://github.com/filecoin-project/solstice/blob/b9a63a87b5adee272c16d7e507cc9c127fb34f0f/deployments.json) for Filecoin mainnet (`314`): Organization 1 = `*Owner2`, Organisation 2 = `*Owner1`. [Latest `deployments.json` on `main`](https://github.com/filecoin-project/solstice/blob/main/deployments.json) is not the pin. The SRA row’s cancel rule follows §2.1.2 (registry changes bind at once; only an SRA code upgrade is held and cancellable).

As of solstice `deployments.json` commit `b9a63a87b5adee272c16d7e507cc9c127fb34f0f`, the SRA proxy `0x0339f205314C8210AF7Cb075d1A96D012e7896a9` and the SWA proxy `0x66C11A9F6dfEC3c1557958cF9f575a023EB01421` are the same on mainnet (chain id 314) and calibnet (chain id 314159). The Safe owner addresses in the table above are the mainnet owners. The calibnet owners are different. Every task-register issue states the chain id. The same proxy address is not a check of which network a signer is on. 


> The approval rules are fixed by FIP-0118 §4.2, and they differ by tier:
>
> > "Both multisigs approve every discretionary action at both tiers. At the SWA tier the change is additionally held before it binds and either multisig alone can cancel it during the hold."
>
> > "At the SRA tier registry actions bind at once, each with an on-chain event and a published rationale; there is no pending queue and no cancellation path (Section 3.2)."

### 2.4.2 Governed Parameters

| Parameter | Where set | Mainnet (314) | Calibnet (314159) |
| :-- | :-- | :-- | :-- |
| `ACTIVATION_EPOCH` (quarter 1 begins) | Activation migration | 6,470,280 | 4,109,134 |
| `EPOCHS_PER_QUARTER` | FIP-fixed at SRA deployment (code upgrade to change) | 262,974 epochs (≈ 91.3 days) | 2,880 epochs (24 hours) |
| `POST_PERIOD` (FPV posting) | FIP-fixed at SRA deployment (code upgrade to change) | 8,640 epochs (3 days) | 240 epochs (2 hours) |
| `VERIFICATION_WINDOW` (FPV verification) | FIP-fixed at SRA deployment (code upgrade to change) | 20,160 epochs (7 days) | 480 epochs (4 hours) |
| `SRA_UPGRADE_HOLD` (the one held SRA action) | FIP-fixed at SRA deployment (code upgrade to change) | 20,160 epochs (7 days) | 720 epochs (6 hours) |
| `SWA_TIMELOCK` (hold on SWA writes to f02) | FIP-fixed, enforced in f02 as `swa_timelock_epochs`, migration-set | 20,160 epochs (7 days) | Set by the migration; not carried in `deployments.json`. |
| SWA-internal timelock (own state and code) | FIP-fixed, equal to the f02 window | 20,160 epochs (7 days) | Follows `SWA_TIMELOCK` |
| Quarter boundaries | `Start(Q) = ACTIVATION_EPOCH + (Q-1) * EPOCHS_PER_QUARTER` | Not calendar quarters | Not calendar quarters |
| Volume gate: `W2_BASE` / `W2_STEP` / gate count | FIP-fixed in SWA code | 10% / 5pp / 8 gates to `W2_CAP` | Same |
| `VOL_TARGET_ENTRY` (gate entry target) | SWA state, `SetGateParams` | 3,500 USD | Same unless set otherwise at deployment |
| `VOL_TARGET_RATIO` (per-step escalation) | SWA state, `SetGateParams` | 2.7 | Same unless set otherwise at deployment |
| `MIN_LOT_FLOOR` | SRA parameter event (`SetPricingParams`); applies from the first epoch of the next quarter | $0.50 (5 × 10^17 atto-USD) | Same |
| `MIN_LOT_ALPHA` | SRA parameter event (`SetPricingParams`); applies from the first epoch of the next quarter | 1/400 | Same |
| `PRICE_BAND` | SRA parameter event (`SetPricingParams`); applies from the first epoch of the next quarter | 30% | Same |
| `REGISTRATION_CUTOFF` | SRA parameter event (`SetPricingParams`); applies from the first epoch of the next quarter | 20,160 epochs (7 days) | Scale to the calibnet quarter before use |
| Admitted-stablecoin whitelist and admitted Filecoin Pay contract addresses | Latest `SetAdmittedLists` event, which is the authoritative record. Applies from the first epoch of the next quarter. This repository only mirrors the event. | `TODO(owners):` the mainnet transaction hash. Not evidenced in this table. | `TODO(owners):` the calibnet transaction hash. |
| `SetPricingParams` pre-activation event | Latest `SetPricingParams` event before activation. | `TODO(owners):` the mainnet transaction hash. Not evidenced in this table. | `TODO(owners):` the calibnet transaction hash. |
| Dispute resolution target ([§4.4.2](04-quarterly-review-and-runbook.md#442-dispute-resolution-for-contested-bindings)) | This repository | 7 days | 7 days |
| SWA objection response-time commitment ([§2.2.8](#228--objection-during-an-swa-hold)) | This repository | 7 days | 7 days |
| Orchestrator response window ([§4.4.5](04-quarterly-review-and-runbook.md#445-escalation)) | This repository | 7 days | 7 days |
| Quarterly report filing deadline ([§4.1](04-quarterly-review-and-runbook.md#41-community-reporting-and-disclosure)) | This repository | Within 7 days after the verification window closes | Same |
| Admission rubric | This repository | Not set. `TODO(owners):` publish the rubric and a decision deadline before the first application is decided. | — |


Mainnet and calibnet values are the deployed values in [`deployments.json` at `b9a63a87b5adee272c16d7e507cc9c127fb34f0f`](https://github.com/filecoin-project/solstice/blob/b9a63a87b5adee272c16d7e507cc9c127fb34f0f/deployments.json) (keys `314` and `314159`). They are candidate values recorded here for operators; they do not amend the FIP. A third key, `3141592` (local devnet), is not operational and is not mirrored here. [Latest `deployments.json` on `main`](https://github.com/filecoin-project/solstice/blob/main/deployments.json) is not the pin.

### 2.4.3 Quarter 1 calendar

Wall-clock times are estimates from epoch 6,470,280, 30-second epochs, and genesis 2020-08-24 22:00:00 UTC. A chain halt moves them. PT is PDT on 2026-10-19 and PST on the 2027 dates. UTC+8 is a fixed offset, not a named zone. 2027-01-18 is a US holiday (Martin Luther King Jr. Day). `TODO(owners):` an announcement channel and whether any translation is published. This repository is English-only until that is decided.

| Event | Epoch | UTC | PT | UTC+8 |
| :-- | --: | :-- | :-- | :-- |
| Q1 starts (activation) | 6,470,280 | 2026-10-19 13:00 | 2026-10-19 06:00 PDT | 2026-10-19 21:00 |
| `REGISTRATION_CUTOFF` zone opens | 6,713,094 | 2027-01-11 20:27 | 2027-01-11 12:27 PST | 2027-01-12 04:27 |
| Q1 ends / posting opens | 6,733,254 | 2027-01-18 20:27 | 2027-01-18 12:27 PST | 2027-01-19 04:27 |
| Posting closes | 6,741,894 | 2027-01-21 20:27 | 2027-01-21 12:27 PST | 2027-01-22 04:27 |
| Verification closes | 6,762,054 | 2027-01-28 20:27 | 2027-01-28 12:27 PST | 2027-01-29 04:27 |
| Quarterly report due | 6,782,214 | 2027-02-04 20:27 | 2027-02-04 12:27 PST | 2027-02-05 04:27 |

The day-5 aim for a `CorrectVolume` second signature falls inside the verification window (opens 2027-01-21 20:27 UTC, closes 2027-01-28 20:27 UTC). It is an aim until `TODO(owners): confirm` in [§2.3.4](#234--quarterly-fpv-verification--correction-correctvolume). Signatures should land by that day so the second approval is not left at the window edge. A second approval included near the close can still be reorganized out.

Two cautions when reading this table:

- **`SRA_UPGRADE_HOLD` and `VERIFICATION_WINDOW` share the value 20,160 on mainnet and are different parameters.** They diverge on calibnet (720 vs 480). Never infer one from the other.
- **Calibnet is not a scaled copy of mainnet.** A calibnet quarter is 24 hours against mainnet's ~91 days, and the posting and verification windows compress by different factors. A rehearsal planned against the mainnet column will run against the wrong clock. Every task-register issue states the chain id for this reason (§2.4.1).

> `SWA_TIMELOCK` is FIP-fixed and enforced in L1:
>
> > "f02 itself holds every SWA write for `SWA_TIMELOCK` (7 days, Section 2.4) before applying it, so even a compromised multisig or a maliciously upgraded SWA cannot make a change bind early."

---

## 2.5 Safety and rotation playbook (both tiers)

The threat model rests on the two-Safes rule: no single Safe can make a discretionary change bind. Either SWA Safe can cancel a pending SWA change during its hold. An SRA registry change has no cancel path once both Safes have approved. Either SRA Safe can `veto(taskId)` only while the task is half-approved or, for a code upgrade, during `SRA_UPGRADE_HOLD`. FIP-0118 fixes the L1 backstop that makes even a fully compromised SWA multisig unable to bind a write early:

> "f02 itself holds every SWA write for `SWA_TIMELOCK` (7 days, Section 2.4) before applying it, so even a compromised multisig or a maliciously upgraded SWA cannot make a change bind early."

### 2.5.1 Key compromise inside a Safe (below the internal threshold)

1. The organization reports publicly in this repository.
2. Any change the Safe approved while the key was compromised is reviewed. If it is still pending, either Safe cancels it: an SWA write during its hold, or a half-approved task via `veto(taskId)`. An SRA registry change that has already bound cannot be cancelled. The review, and any cancel, is recorded on the issue. 
3. The organization rotates its internal membership (org-internal, immediate, no governance action) and announces the rotation here.

### 2.5.2 A whole Safe compromised (internal threshold reached by an attacker)

1. Nothing binds from one Safe alone. A hold (where one exists) starts only after both Safes approve. One Safe can only submit a task; the `Submitted` event carries the `taskId` (a hash), not the full content. The other Safe’s remedy is `veto(taskId)` on that half-approved or held task. Repeated resubmission is publicly visible; the attacker cannot bind a change silently. On the SWA side, a malicious discretionary write also lacks its required published FIP, making the objection case unambiguous.
2. **Task register (off-chain).** Pending tasks never expire on-chain (`UnanimousGovernance`), and `Submitted` names only the `taskId` hash. Before the first on-chain approval of any governance action, open an issue in this repository that records the chain id, the exact calldata, and the expected `taskId` (= `keccak256(msg.data)`). Mainnet is chain id 314. Calibnet is chain id 314159. The same proxy address on both networks is not a check that the signer is on the intended network (§2.4.1). That issue is the human-readable register entry the hash alone cannot provide. **Canonical calldata:** both Safes copy the calldata from that issue verbatim. Address lists in the payload (e.g. `SetAdmittedLists`) are sorted ascending by address before encoding — the same list in a different order is a different `msg.data` and a different task that never reaches unanimity. The second Safe also records its pre-approval dry-run result on that issue (see the standard flows in §2.2 and §2.3). Proposed, not in force. `TODO(owners): approve`. Each Safe decodes that calldata with its own tool and signs only on a match, and the issue is locked after the first approval. Proposed, not in force. `TODO(owners): approve` vetoing a half-approved task older than 14 days, with a written reason for every veto. The contract does not expire the task. 
3. The tier is treated as stood down, and the consequences are bounded: a stood-down registry tier means payments and `SetShares` continue; a stood-down SWA tier means discretionary changes stop while the ramp and the gate continue through the permissionless crank. This is an operational posture for the tier, not a status on an Orchestrator — there is no Orchestrator freeze in this program, and `RemoveOrchestrator` is the single sanction (§2.3.2).
4. Exit: replace the compromised Safe address. This requires both Safes, so if the compromised Safe obstructs, the FIP backstop applies (2.5.3).

### 2.5.3 Replacing a registered Safe

Applies to either tier; the mechanics are the same on the SRA and the SWA.

**Cooperative case** (requires both Safes of that contract · hold: none · not cancellable):

1. Veto every open task on that contract first. `removeOwner` does not clear pending approvals; a freed owner bit can recycle to a future owner still carrying old approvals (`Owners.sol`). Every unanimous action shares one pending-task mapping, so this covers code upgrades, registry or gate-parameter changes and owner replacements alike; the task register above lists them. The issue records a sign-off that every open task was vetoed before the first `ReplaceOwner` approval. 
2. Both Safes approve the replacement (`ReplaceOwner`), with the action's chain id, calldata, and `taskId` recorded in the issue before the first approval (see task register above). It binds at once when the second Safe approves.
3. Record the new Safe in [2.4.1 Safe Addresses](#241-safe-addresses) and the [Program Change Log](06-changelog.md), with the reason for the replacement; a compromise also gets a post-mortem.
4. Do the engineering items in the [owner change FAQ](https://github.com/filecoin-project/solstice/blob/b9a63a87b5adee272c16d7e507cc9c127fb34f0f/docs/UPGRADE.md#how-do-i-handle-an-owner-change) (pin `b9a63a87b5adee272c16d7e507cc9c127fb34f0f`). The issue pins that runbook by commit. [Latest on `main`](https://github.com/filecoin-project/solstice/blob/main/docs/UPGRADE.md#how-do-i-handle-an-owner-change) is not the pin.

**Hostile or deadlocked case**, always under a published FIP. The path depends on which tier has failed (FIP-0118 §4.3). A network upgrade is not the path for both.

- **Rogue or deadlocked SRA.** SWA Governance re-points the service stream's designated writer to a redeployed SRA (`SetDistribution`, §2.2.4). Registry state is reconstructible from public data. Until that write binds, the incumbent SRA keeps the service-stream flow. The gate keeps reading the previous SRA until an SWA code upgrade changes the wired SRA address (FIP-0118, Security Considerations). That code upgrade is a FIP and a contract upgrade. It is not a network upgrade (FIP-0118 §4.1).
- **Rogue or deadlocked SWA.** A network upgrade migrates the SWA address in f02 (FIP-0118 §4.3 and §3.1).

`TODO(owners):` whether proxy addresses hardwired in Lotus, Forest, and Venus, the question PR #17 raised, change the SRA path in FIP-0118 §4.3. This repository does not answer that question.

### 2.5.4 Both SWA Safes captured

When both SWA Safes are captured, a hostile valid write stays queued for the full SWA_TIMELOCK. The backstop is an emergency network upgrade within that hold (FIP-0118, Security Considerations). FIP-0118 does not provide a fast path short of a FIP. This repository does not shorten that clock.
 
Simultaneous compromise of both tiers has no separate procedure in the FIP beyond that emergency upgrade and, if an honest SWA write is still possible, the SRA re-point in §2.5.3.

### 2.5.5 Emergency annex (skeleton)

`TODO(owners):` implementer contacts for Lotus, Forest, and Venus, a public alert channel, a pre-drafted emergency FIP, and a tabletop before activation. No contact, channel, or tabletop is named here. The backstop remains the emergency network upgrade in §2.5.4. This repository does not shorten that clock.

### 2.5.6 Decision records

Rationale, dispute outcomes, dry-run results, and objections are written on the task-register issue and snapshotted into the [Program Change Log](06-changelog.md) by pull request, using [the decision-record template](templates/decision-record.md). After the decision is recorded, lock the issue. Do not edit the rationale in place. A correction is a new comment or a new Change Log pull request. `TODO(owners):` an archive method beyond GitHub. This repository does not choose one.

---

← [Back to README](../README.md) · Next: [3. Orchestrator Operational Guidelines](03-orchestrator-operational-guidelines.md) →
