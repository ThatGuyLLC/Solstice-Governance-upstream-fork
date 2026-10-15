# Storage provider FAQ

This page points storage providers at the FIP. It does not restate pledge or quality-adjusted-power rules as rules of this repository. Those effects are specified in FIP-0118 and in the node implementations.

Quotations below are copied from FIP-0118 at [`23b42964890590a49c9cbc6e19a7e9e361d1f8a0`](https://github.com/filecoin-project/FIPs/blob/23b42964890590a49c9cbc6e19a7e9e361d1f8a0/FIPS/fip-0118.md). Latest text, not used for quotations: [FIP-0118 on `master`](https://github.com/filecoin-project/FIPs/blob/master/FIPS/fip-0118.md).

## What changes for sector quality

FIP-0118, lines 2922–2924:

```
New sectors receive 10× QAP regardless of content, changing the QAP and
pledge inputs expected by sealing pipelines, pledge estimators and SP
operations software.
```

Datacap deprecation is FIP-0118 §1, lines 293–317 of the same pin: [section 1](https://github.com/filecoin-project/FIPs/blob/23b42964890590a49c9cbc6e19a7e9e361d1f8a0/FIPS/fip-0118.md#1-datacap-deprecation).

## What this repository does not review

Orchestrators choose their clients. The program does not review Orchestrator pricing, client selection, or operations, including which storage providers an Orchestrator routes business to. See [§3.2.1](03-orchestrator-operational-guidelines.md#321--deal-making).

---

← [Back to README](../README.md)
