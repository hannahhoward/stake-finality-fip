---
fip: "XXXX"
title: Stake-Weighted F3 Finality
author: Hannah Howard (@hannahhoward)
discussions-to: https://github.com/filecoin-project/FIPs/discussions/1288
status: Draft
type: Technical
category: Core
created: 2026-07-21
requires: 86, 100, 105, 115
---

> **Draft for discussion.** *[re-check]* marks chain snapshots, citations, or modeling inputs to verify before submission; *[strawman]* marks provisional values; *[needs precise spec]* marks deliberately open specification work.

**How to read this.** Four questions, in order.

1. **Do I accept the premise?** The proposal takes as given that the storage base is declining and that FIL needs a durable security and holding incentive while deeper storage-economics changes are developed. If you reject that premise, cheaper instruments exist; *Change Motivation* describes them. Say why, and stop there.
2. **Is a stake-weighted finality committee the right first step?** It secures finality only; leader election, block production, and PoRep are unchanged. If a different first intervention would better address the premise, that is the feedback to give.
3. **Does the mechanism hold?** The design combines a staking actor, a stake-derived power table, rewards only for signatures included in canonical certificates, a reward rate that falls as stake arrives, correlation-scaled slashing, and an activation gate based on depth and dispersion. Where should the mechanism be tightened, and which strawman constants should change?
4. **Is the price worth paying, whatever the source?** The pool needs about 100M FIL of total funding over the program's life. This proposal leaves the source open. Is slashable, stake-secured finality worth a cost of that size?

Supporting analyses are linked in the comments below.

*The participation reward requires a funded pool, but this thread leaves the source open to test the mechanism first. **[Funding the finality reward: options and a recommendation](https://hannahhoward.github.io/stake-finality-fip/funding-options-and-recommendation.html)** covers the options and is not the subject here. If the mechanism has consensus, funding is the next question; §5 states the requirements, and activation requires an accepted funding source.*

# FIP-XXXX: Stake-Weighted F3 Finality

## Simple Summary

This proposal secures the token. The deeper fixes to Filecoin's storage economics will take years, and FIL needs a durable security and holding incentive in the meantime. The proposal revives Filecoin's fast-finality gadget (F3) by deriving committee voting weight from **staked FIL** rather than storage power and by paying participants whose signatures appear in finality certificates. The reward comes from a funded pool specified in §5. Leader election and Proof-of-Replication (PoRep) remain unchanged.

## Abstract

This proposal secures **finality** — irreversible chain history — with staked FIL. F3 (FIP-0086) uses a Group-based Practical Byzantine Fault Tolerance (GPBFT) committee weighted by quality-adjusted storage power; that resource also determines **leader election** through Expected Consensus and PoRep. F3 participation is unrewarded, and the committee has struggled to maintain quorum, leaving the network on the slower 900-epoch Expected-Consensus finality rule.

The mechanism weights the finality committee by staked FIL and rewards signing. Because GPBFT already consumes an abstract power table, it can derive that table from stake instead of storage power. A new staking actor records stake on chain, making GPBFT equivocation slashable against it.

A participant earns only when its signature is included in a certificate. The offered rate declines as stake arrives, letting the market determine equilibrium depth without a rate floor, and depends on the reward pool's released balance. The pool requires about 100M FIL over the program's life; §5 gives the four requirements for any source.

Because a new staking actor starts empty, staking opens at the upgrade epoch, but stake-weighted finality **becomes authoritative only after about 80M FIL is staked and sufficiently dispersed**, as specified in §6.

This proposal covers finality only. Leader election, block production, and PoRep are unchanged. Stake-weighting *leader election* — a full consensus pivot that would retire PoRep — is a separate and substantially larger question and is explicitly **out of scope**.

## Change Motivation

**No FIL is natively staked today.** Filecoin has no native staking mechanism. The FIL that secures the network is pledge collateral bonded to storage sectors, while liquid FIL earns no protocol-native yield. Consensus therefore depends on storage power, and the storage base is declining. This proposal takes that decline as given.

On that premise, the objective is to strengthen FIL's economic security while giving finality a base independent of future storage-power levels. A protocol-native reward gives holders a reason to stake, creating slashable collateral for finality.

**If the premise is rejected, this is the wrong instrument.** A committee weighted by a stable resource need not be re-based. Discussion #1106 restores the same two-thirds threshold near a storage provider's marginal signing cost using a slice of block rewards and no added funding (*Prior art*); vote escrow can create locking without changing consensus. If storage remains a sufficient security foundation, those approaches are cheaper.

Under the premise used here, however, a quorum patch would restore certificates on a security base that continues to shrink, backed by collateral that is returned after exit, and it would not address the proposal's token-demand objective. Re-weighting the committee from storage power to stake is what drives the additional funding requirement quantified in the companion analysis.

FIL holders have already allocated capital voluntarily to credit-exposed yield instruments (see the companion analysis), suggesting demand that a protocol-native instrument could serve. The posted rate opens below a 12% ceiling and declines as stake arrives (§3), settling around 4.0–5.0% while the modeled staked pool remains near the target security level (yield comparables: companion analysis).

Filecoin's lack of staking also leaves finality without an independent, slashable security base. FIP-0086 introduced F3 alongside Expected Consensus: participants sign GPBFT messages, and ≥⅔ of committee power produces a certificate that makes a tipset irreversible within tens of seconds, versus the ~900-epoch (~7.5h) Expected-Consensus rule.

FIP-0086 deliberately did **not** reward participation, reasoning that signing is cheap and participants benefit from faster finality. In practice, the committee lost quorum and F3 stopped producing certificates, leaving the network on Expected-Consensus fallback finality. This proposal gives finality a slashable stake base and fixes the minimum depth required for authority in protocol rather than accepting the market's storage-power level.

## Specification

The mechanism has six parts: a staking actor (§1); stake-weighted F3 power table (§2); certificate-gated reward (§3); on-chain certificate submission (§3a); equivocation slashing (§4); and an activation rule (§6). §5 defines the roughly 100M FIL funding requirement and required source properties; the source remains open.

Only the power-table formula (§2) is a trivial change. The certificate-submission path (§3a), fault-proof machinery (§4), and transition (§6) add genuine protocol surface. Everything outside the actor bundle must also be implemented separately in each node client.

### 1. Staking actor (new)

A new built-in actor is the on-chain source of truth for staked FIL. It records each participant's balance and BLS signing key; supports deposits, delayed withdrawals, and delegation; and maintains the activation state (§6), including the append-only `ActivationRecord` that determines which certificates bind (*Appendix B*).

**The unbonding delay has two lower bounds.** It must exceed the fault-proof window (§4), so an equivocator cannot withdraw before slashing, and be economically meaningful: exiting stake remains locked long enough that the pool cannot drain immediately and exit is priced against the posted rate rather than treated as an intraday trade (**strawman: 90 days**). It is a delay, not a fixed term; all figures here and in the companion analysis assume at-will stake.

*[needs precise spec — Appendix A]*

### 2. Stake-weighted F3 power table

GPBFT's ⅔ quorum, BLS12-381/BDN signature aggregation, certificate format and signer bitset, and modified Expected-Consensus fork-choice rule are all agnostic to the source of committee power. At the formula level, the change is therefore a single input substitution. Where FIP-0086 computes

```
ParticipantPower ← truncate(0xffff * MinerQAP / TotalQAP)
```

this proposal computes

```
ParticipantPower ← truncate(0xffff * Stake_i / TotalStake)
```

The staking actor supplies the new value at the same `PowerTableLookback = 10` lookback (ten GPBFT instances; the implementation names the constant `CommitteeLookback`). The 16-bit normalization and certificate machinery are unchanged.

The current committee's eligibility filters cannot be reused directly: the 10 TiB power floor, zero fee debt, and no unelapsed consensus fault are miner-actor conditions with no equivalent meaning for a staking actor. Stake-side eligibility therefore consists of a minimum stake plus the staking actor's own fault state (§4).

Two encoding limits bound minimum stake. Below 1/65535 of total stake, a participant normalizes to zero weight; CBOR also caps the power table and each certificate delta at 8192 entries, so the minimum must keep the committee within that limit. Rewards accrue outside the power-table balance (§3), avoiding per-epoch compounding that would place every participant in every certificate delta. This weighting becomes authoritative only after activation (§6).

### 3. Participation reward, gated on signed certificates

Staked FIL earns a reward **only** when its signature appears in a valid, canonical finality certificate, for the epochs that certificate newly finalizes. Repeating an already-final head earns nothing. Epochs advance every 30 seconds regardless of committee behavior, so faster certificate production cannot increase payout. Stalling, idle stake, and halted F3 earn nothing.

As in Discussion #1106 (*Prior art*), rewards split into a signer share and a smaller includer share (`α`, around 10%) paid to the on-chain submitter (§3a), in practice the block producer, since only the producer collecting `α` has reason to include a submission. One signer's reward does not reduce another's, so censoring a peer gains nothing and refusing submission forfeits `α`.

The offered rate is a published function of two on-chain values: total staked FIL, `S`, and the **released** balance of the reward pool, `P`:

```
r(S, P) = spend_rate × P / (S + S_offset)
```

`S_offset` keeps the rate finite at low stake and controls its decline. `P` is only the released ledger; funding enters unreleased and moves to released on a fixed schedule (§5).

`spend_rate` and `S_offset` are set together with the funding envelope, so their values depend on the adopted source. The table and curve below use the worked configuration from the companion analysis: `spend_rate = 0.22` and `S_offset = 55M`. The launch rate is capped at 12% across all options *[strawman]*.

| Staked FIL | Offered APR at launch, worked scenario |
|---|---|
| 0 | 12.00% |
| 25M | 8.25% |
| 50M | 6.29% |
| 80M | 4.89% |
| 100M | 4.26% |
| 130M | 3.57% |
| 200M | 2.59% |

![Posted rate against total staked FIL, worked scenario](https://hannahhoward.github.io/stake-finality-fip/charts/curve-depth.png)

*The posted rate declines as stake arrives, so the first 80M FIL is recruited along a declining curve rather than at a single fixed rate. The curve uses the same worked configuration.*

The schedule has two further properties:

- **Pool scaling.** The schedule moves with released balance: rewards lower it and scheduled releases raise it. Unspent funds persist, so a slow bootstrap does not starve later stakers; if releases outpace rewards, the rate rises until stake responds.
- **Annual payout remains below `spend_rate × P` at every stake depth.** In the worked scenario, year-1 payout is 6.60M FIL on a 30M released pool. There is no catch-up payment for epochs that pay nothing.

If the schedule cannot recruit enough stake to reach the activation threshold (§6), the year-3 non-activation review (§5) is triggered if no certificate has ever become binding. *Incentive Considerations* analyzes that case.

Lifetime payout is bounded by total pool funding, the payment rule above, and the sunset (§5).

For example, suppose a holder stakes 10,000 FIL in the tenth month of signing, when total network stake is 50M FIL. In the worked scenario, the offered rate is 6.29%, or roughly 630 FIL over the following year, accruing only for epochs in which the holder's signature appears in canonical finality certificates.

When total staked FIL reaches 100M — year 6 on that modeled path, with 34.2M FIL released and unpaid — the same 10,000 FIL earns 4.85%, or about 485 FIL per year. Claims are pull-based. The posted rates above are therefore gross of both the includer share `α` and the gas paid by a staker to claim.

Rewards begin accruing with the committee's first certificate, including during the pre-activation accumulation phase (§6).

*[needs precise spec — Appendix A]*

### 3a. Certificate submission and verification (new)

Today F3 certificates remain off chain, in node-local stores and peer-to-peer protocols. §3 requires the chain to receive and verify them, making submission the proposal's largest new protocol component. Discussion #1106 favored message submission over larger block headers; this proposal follows that direction.

A block producer submits each new certificate as a message to the reward-pool actor, which maintains a per-instance watermark. The first valid submission credits signers and pays `α`; duplicates fail cheaply. A conflicting certificate for the same instance is accepted as evidence but earns no reward, providing on-chain evidence of correlated equivocation (§4).

Submission is permissionless: any account may submit a certificate. Suppressing a valid certificate therefore requires every producer to forgo `α` against its own economic interest.

The actor verifies certificates directly. Aggregate verification requires a coefficient-weighted multi-scalar multiplication over signers plus one pairing; the required BLS12-381 arithmetic already ships under FIP-0105, so no new syscall is required.

Per-certificate gas cost is unmeasured and an early implementation deliverable. If in-actor verification is prohibitively expensive, the fallback is client-side validation feeding the actor a system-attested summary, which would add a block-validation rule to every client and depend on FIP-0107. This proposal assumes in-actor verification.

Resolving a certificate's signer bitset to staker identities requires the ordered committee table for that certificate's instance. Because the staking actor is the power source, it retains per-instance committee snapshots, and the retention window must cover the fault-proof submission window (§4).

Sections §3, §4, and §6 all depend on the same structures: per-instance committee snapshots; the instance-to-epoch mapping supplied by the certificate record; and the number of epochs each certificate newly finalizes.

A chain message is capped at 64 KiB, while the certificate format permits larger power-table deltas. Submitted deltas must therefore be chunked, or the table must read stake snapshots at a bounded cadence. *[needs precise spec — Appendix A]*

### 4. Slashing for equivocation

Double-signing (equivocation) in GPBFT — signing conflicting messages for the same instance — is slashable against the offender's staked FIL. This is the key security distinction from storage power: an attacker's staked capital can be *burned*, while acquired storage power remains recoverable.

Because GPBFT uses explicit signatures, conflicting signed messages attribute a fault to a key. Today F3 nodes drop conflicts and GPBFT messages are not verified on chain, so slashing needs a fault-proof subsystem: nodes retain and gossip conflicting pairs; anyone may submit one; the actor reconstructs the FIP-0086 payloads and verifies both signatures against the registered key.

Correlated equivocation can use a simpler proof. Two valid certificates for the same instance that decide different chains are themselves evidence that every signer appearing in both bitsets double-signed, and those certificates are already on chain (§3a).

The slash scales with correlation. Isolated equivocation receives a small base slash (**strawman: 1–5%**, plus committee ejection), because an operator accident must be survivable or the penalty will deter participation as well as attacks. Ejection takes effect at the power-table lookback, ten instances after the fault is recorded, because the tables for in-flight instances are already fixed.

The slash then rises with the fraction of stake that equivocates within the same window, reaching a **total burn of approximately 100% as the equivocating fraction approaches the ⅓ required to threaten finality**. Because the final scale depends on additional proofs that may arrive later, slash amounts remain in escrow until the window closes and then execute at the final scale.

Escrow matters especially in a mass event: equivocation by a third of the committee resembles both an attack and a partition or defective release. A successful finality attack therefore risks roughly the attack stake itself — about one-third of the pool, or tens of millions of FIL at activation. That amount must exceed the double-spend value protected by fast finality, and the activation band should be re-checked as settlement grows.

A valid fault-proof submitter receives a reporter share, capped by the amount burned, mirroring today's consensus-fault report and paying the gas cost of policing the committee. The remainder is burned; returning it to the pool would raise `P` and the offered rate, letting an attack subsidize other stakers.

Slashing is active from the committee's first certificate, before activation (§6). The unbonding delay (§1) must strictly exceed the fault-proof submission window, so an equivocator cannot withdraw before a slash can land. The window is defined over instances and anchored to epochs by the certificate record (§3a).

Under delegation, the signing key belongs to the operator. Which stake burns — the operator's bond, delegators' pro-rata stake, or both in some order — depends on the committee identity model (worker key, dedicated key, or delegated operator key). *Appendix A* records that choice as open.

*[needs precise spec — Appendix A]*

### 5. What the funding has to supply

**The reward pool requires about 100M FIL of total funding.** In the worked model, that supports roughly twelve years at the 80M FIL security level (§6) and remains a strawman. Less funding buys a shallower pool or fewer years at target depth; more raises total consensus expenditure.

Whatever the funding source, the mechanism requires four things from it:

- **A reward-pool actor.** The actor is built in and has no controller. It maintains two ledgers: **unreleased** and **released**. Only the released balance is spendable, and `r(S, P)` reads only that balance (§3).
- **A fixed release schedule.** Balance moves from the unreleased ledger to the released ledger according to a predetermined table with no chain-state input. This keeps the posted rate a published function of observable values. Release pacing limits the opening rate, and the rate changes as releases occur (§3).
- **Supply-accounting treatment.** The pool's entire unpaid balance is excluded from circulating supply rather than only a source-specific sub-ledger. Both new singletons — the reward pool and staking actor (§1) — must also be included in the circulating-supply traversal; otherwise `StateCirculatingSupply` fails.
- **Review and sunset hooks.** A year-25 funding review precedes a year-30 sunset, when payments stop and the remainder is disposed of. A **year-3 non-activation review** applies if §6 has never been met and repeats annually until activation, covering a pool that pays for advisory certificates but never binds and providing an amendment path if required return was misestimated.

**The funding source is open.** This proposal does not select one. The companion analysis compares candidate sources against the four requirements above and recommends one; every numerical illustration here uses that analysis's worked scenario. Adopting this mechanism requires an accepted funding source to be adopted with it.

### 6. Transition and activation

At upgrade, total staked FIL is approximately zero. F3 needs a non-empty initial power table, and a nearly empty committee is worse than none: one participant alone is a ⅔ quorum. *Implementation* gives the deployment sequence; this section defines its gates.

Activation therefore uses a stake-depth threshold plus distribution requirements. During accumulation, the protocol pays pre-activation rewards for advisory certificates and incurs the blockspace and verification gas required to carry them on chain (§3a; see also *Security Considerations*).

- **Staking opens at the upgrade epoch; the committee starts only after it can form.** Stake-weighted F3 begins as a fresh instance chain with a new F3 network name, bootstrap epoch, and pinned initial power table. Those values ship in a coordinated client release once registered stake exceeds a formation floor set well below the activation threshold *[needs precise spec]*.

  F3's original mainnet rollout similarly staged code deployment and activation and pinned the initial power table in a follow-up release; the formation-floor trigger is new here. A fresh chain also cleanly separates the QAP-era and stake-era certificate streams: verifiers, bridges, and snapshots built against the old stream reject stake-era certificates rather than mistaking them for authoritative.

  A handoff within the existing chain is representable — one certificate could replace the entire power table — but it would have to be signed by two-thirds of the old QAP-weighted committee, which is the quorum this proposal is intended to replace. From the first stake-era certificate, stakers earn the participation reward (§3). Rewarding the accumulation phase is necessary to recruit stake toward the activation threshold.
- **Certificates remain advisory until both depth and distribution conditions are met.** The authority condition is `total_staked ≥ stake_activation` **and** the distribution conditions below. Until then, the network continues to use today's 900-epoch Expected-Consensus rule, and stake-weighted certificates are advisory only. This avoids the *thin-but-trusted* failure mode in which a cheap-to-acquire quorum becomes authoritative.

  The staking actor maintains total stake, distribution statistics, and the activation latch. Every client reads the same state at finalization and suppresses advisory certificates from finality reporting. The depth latch resembles the power actor's minimum-miner count; distribution statistics differ because they require sorting the operator set. Once all conditions hold, stake-weighted fast finality becomes authoritative.
- **Activation requires dispersion, not just depth.** A deep pool only makes a one-third coalition expensive if stake is sufficiently dispersed. The gate therefore also requires the following strawman conditions *[needs precise spec]*: the smallest coalition of distinct participants controlling one-third of stake must contain more than `N_min` participants, and no single participant may control more than `max_share` of the pool. Both are measured over delegated *operators* — the entities that actually sign — rather than raw addresses.

  These tests are **necessary but not sufficient** because on-chain identities are cheap to multiply. They prevent accidental concentration from being activated and force a deliberate attacker to manufacture at least the appearance of distribution. The underlying economic backstop remains stake depth plus correlation-scaled slashing (§4).

**The latch uses asymmetric open and close conditions, making activation deliberately sticky.** It opens only when `total_staked ≥ stake_activation` and the distribution conditions hold continuously for a sustained window `W` (**strawman: three months**). A single deep epoch therefore cannot activate finality.

The latch closes only when `total_staked < stake_deactivation` continuously for a much longer `W′` (**strawmen: 50M FIL and twelve months**). **A distribution failure after activation never closes the latch.** Otherwise an adversary could create concentration and gain a shutdown switch without one-third of stake or equivocation, cheaper than stalling. Post-activation concentration is handled by correlation-scaled slashing (§4) and, if necessary, a later FIP.

The deactivation band is a safety catch, not a live control. It is wide enough that ordinary exit and refill should not trip it. A higher deactivation floor preserves a higher veto price but ends binding finality sooner; *Appendix B* sizes that tradeoff.

The latch history is the authority rule. The staking actor's `ActivationRecord` (§1) is an append-only list of open and close events. **A certificate for instance `i` is binding exactly when the record contains an activation opened at or before `i` that has not closed by `i`.** *Appendix B* defines the record layout and the certificate authority mark used to resolve against it.

*[needs precise spec — Appendix A]*

### Prior art

GitHub **Discussion #1106, "Incentivisation of F3" (Kubuxu, Jan 2025)** proposes allocating **up to ~10% of block rewards** to F3 participation, using evidence in on-chain finality certificates and splitting rewards between signers and the block producers that include them. It remains an open discussion and was never adopted as a FIP. Its text keeps total storage-provider rewards unchanged, so the proposal redistributes existing rewards among providers.

This proposal borrows #1106's inclusion and anti-gaming structure and its preference for message submission (§3a). *Change Motivation* explains the substantive difference between the two approaches and the additional cost introduced here.

## Design Rationale

The security argument depends on what stands behind the consensus threshold, not merely on where the threshold is set. Acquired storage power can transfer without a protocol fee, its pledge is recoverable collateral, and the penalty for manipulating chain selection through provable double-fork mining is a fixed slash of roughly 4 FIL. By contrast, staked FIL that equivocates is burned at a rate that scales with the equivocating fraction (§4).

At the activation threshold, the two security bases are of similar nominal size but differ in what an attacker can lose: roughly 21.5M FIL of recoverable pledge behind one-third of storage power *[re-check]* versus 26.7M FIL of burnable stake behind one-third of the pool. The required stake depth is enforced by the activation rule rather than left to the current level of storage power.

This comparison addresses safety, not liveness: a committee can stall without equivocating under either design. *Security Considerations* analyzes that case. The 26.7M FIL figure is computed from the base and correlation-slash assumptions in §4.

**Why finality, and only finality.** Finality and leader election can be weighted by different resources. Re-weighting finality is comparatively additive because GPBFT already consumes an abstract power table. Re-weighting leader election would require retiring or replacing PoRep, a much larger undertaking.

This proposal therefore captures the security benefit of slashable, stake-backed finality while leaving the storage market's proof machinery intact. If future PoRep research produces a better leader-election proof, changing that proof can remain a separate proposal while finality continues to use stake.

**What this does and does not build toward stake-based consensus.** Adopting this proposal is not a decision to move Filecoin to proof-of-stake consensus. It does, however, build four components that such a transition would otherwise need: an on-chain stake registry with deposits, delegation, and unbonding; equivocation slashing with fault proofs and correlation scaling; a pool of staked FIL with known depth and measured distribution; and operating experience with a BFT gadget whose committee power comes from stake.

It does **not** build stake-based leader election. Using stake to decide block production is the largest missing step in any proof-of-stake pivot and introduces defenses against grinding, nothing-at-stake, and long-range attacks. Those problems do not arise here in the same way because leader election remains PoRep/QAP-weighted and finality remains explicit-signature BFT, where the relevant failure is slashable equivocation rather than costless simulation.

If the network later chooses a broader proof-of-stake transition, this proposal would provide some of the substrate, but leader election would remain the largest unresolved component.

**Why a stake threshold rather than a QAP→stake ramp.** A blended power table, with storage power fading out as stake fades in, could restore storage-secured fast finality immediately. But fast finality is already offline, so there is no urgent live service to preserve during a transition. A blend would also reintroduce QAP into the finality table and require paying storage signers during the ramp. A threshold is simpler: use pure stake-weighting from the start, but make it authoritative only once the pool is deep and dispersed enough.

**Why rewards are gated on certificates.** Paying all registered stake regardless of participation would recreate F3's free-rider problem. Requiring inclusion in a valid canonical certificate pays for an actual service — a finality signature over the canonical chain — and is self-limiting: no certificates, no payout.

### Alternatives considered and set aside

Discussion #1106's block-reward slice is the cheaper alternative, and *Change Motivation* explains the choice between it and this proposal. Funding-side alternatives — different sources, envelopes, and designs that meet the same requirement — are covered in the companion analysis.

**Why the offered rate has no floor.** Ethereum's staking-yield curve remains near 1.5% even at high stake levels, so, in the words of EIP-8363, *Tapered Issuance Burn* (Draft, created 2026-07-14), there is "no point at which the incentive to stake switches off." Correcting that later can require adding a burn against issuance after products have already formed around the base rate.

Here, `r(S, P)` is strictly decreasing in staked FIL, has no floor, and approaches zero. The reward pool is depleted by its own payments, and the mechanism sunsets in year 30 (§5). The offer can therefore wind down by design rather than requiring a later governance decision to turn it off.

## Backwards Compatibility

- **Supply accounting.** The reward pool and staking actor are new singletons. Any funding source must satisfy the supply-accounting requirements in §5; changes specific to the funding source belong to the separate funding decision.
- **F3 certificate format, verification, and the ⅔ quorum rule are unchanged; the certificate chain restarts.** The stake-weighted committee begins a fresh instance chain under a new F3 network name (§6). Consumers of the QAP-era chain — bridges, verifying contracts, and F3-aware snapshots — must re-initialize against it. Only the power table's *input* changes from QAP to stake.
- **Finality committee membership changes.** Committee eligibility moves from storage providers with at least the QAP threshold to identities with at least the stake threshold. The switch has no predetermined date because it is data-triggered (§6): every client evaluates the same consensus rule against staking-actor state. Today's equivalent F3 switch is configured at build time.
- **Every client implements the non-actor changes independently.** The actor bundle is shared; the remaining changes land once per client and are held consistent through shared conformance vectors (*Implementation*).
- **Finality reporting changes once, at activation.** Because advisory certificates are suppressed (§6), the finalized/safe tipset reported by a node may jump forward by as much as ~900 epochs when certificates first become binding. Systems that derive confirmation counts from those tags will observe that one-time discontinuity.
- **Off-chain consumers must understand staked balances.** Staked FIL leaves the holder's account balance and resides in actor state, so wallets, custodians, and exchanges need a staked-and-accruing balance concept. The circulating-supply RPC must classify the two new actors, and existing storage-provider F3 participation software will no longer be eligible to participate.
- **Block-production liveness does not change.** Block production is unaffected, and until stake-weighted certificates become binding the finality floor remains today's 900-epoch Expected-Consensus rule (the *Halted* row in *Appendix B*).

## Test Cases

*[to be expanded]* Five cross-client test families, each tied to a specific invariant; the funding side has its own tests:

- **Actor lifecycle.** Deposits, delegated deposits, key registration, and withdrawals behind the unbonding delay preserve consistent staked balances and operator-table state.
- **Power-table derivation.** Every client derives the same normalized 16-bit table from `truncate(0xffff * Stake_i / TotalStake)` at the lookback, resolves a certificate's signer bitset to the same participants, accepts a certificate signed by ≥⅔ of stake, and rejects one below ⅔.
- **Certificate-gated reward.** A signature in a canonical certificate accrues at `r(S, P)` using the current stake depth and released balance for exactly the epochs that certificate newly finalizes, counted once across overlapping certificates. Non-signers, certificates that finalize no new epochs, replays, and halted periods accrue nothing. Annual payout never exceeds `spend_rate × P`.
- **Slashing and fault proofs.** A valid conflicting pair slashes the signer and pays the reporter within the cap. Invalid pairs are rejected before expensive signature verification; expired proofs are rejected; escrowed slashes re-scale as additional proofs arrive and execute when the window closes; and slashed stake cannot exit through the unbonding queue.
- **Activation latch.** Certificates are produced but remain advisory until depth and both distribution conditions hold throughout `W`. They stop binding only after `stake_deactivation` holds throughout `W′`, never solely because a distribution condition fails after activation. Historical instances resolve identically as binding or advisory from the `ActivationRecord` for both syncing nodes and certificate-only verifiers.

## Security Considerations

**Stake-weighted finality adds a large, slashable security budget.** In the companion analysis's two demand scenarios, 80–130M FIL is staked from year two through the first decade, depending on the return stakers require. That capital becomes economically slashable collateral for finality, a security layer that does not exist today. At the proposed 80M FIL activation level, a one-third finality coalition must put roughly 27M FIL of slashable capital at risk.

One way to compare the deterrent with the potential prize is to compare one-third of staked FIL with one year of storage-provider consensus rewards. In the worked scenarios, those series cross in the first year of signing, and the gap widens thereafter. The staked series comes from the worked scenario; the consensus-reward series uses the block-reward schedule adopted in FIP-0118.

| Year from finality launch | 1 | 2 | 3 | 4 | 5 | 6 |
|---|---|---|---|---|---|---|
| One-third of staked FIL (M) | 20.0 | 26.7 | 28.3 | 31.7 | 31.7 | 33.3 |
| Storage-provider consensus reward that year, FIP-0118 as passed (M) | 15.2 | 9.5 | 6.8 | 6.0 | 5.4 | 4.8 |

The crossing holds at both modeled required returns and occurs before the activation gate opens in year 2 on the modeled path (§6). By the time certificates become binding, the slashable budget is therefore already larger than this comparison's modeled prize. A funding choice that reduces storage-provider consensus rewards widens the gap.

**Slashing must be enforceable.** Stake is a stronger security base than storage power only if equivocation penalties are real and collectible. Attribution can come from either a conflicting signed-message pair or conflicting certificates (§4). The fault-proof subsystem and final slash schedule therefore determine whether the deterrent holds.

**While the pool accumulates, finality remains at today's baseline.** No committee runs until registered stake passes the formation floor (§6). After formation, certificates remain advisory until the pool is sufficiently deep and dispersed, so a thin committee is never trusted and block-production liveness is unaffected.

During this period, finality continues to use the Expected-Consensus fallback. Published analysis places its adversary tolerance near **~20% of power, compared with ~33% for a live BFT finality gadget** (the ~20% figure is from Wang, Azouvi and Vukolić, AFT 2023; ~33% is the standard BFT bound). The network is in that condition today, and no adopted FIP ends it; this proposal does.

The uncertain variable is *duration*: how long it takes the stake pool to reach the activation threshold. Both inputs to that estimate — reachable idle FIL and uptake rate — are low-confidence today (*Incentive Considerations*). The design therefore uses a review date rather than relying on a forecast. If no certificate has ever become binding, the year-3 non-activation review (§5) reports the pool's trailing-year behavior and provides an amendment path.

If reachable idle capital is at least the activation threshold, the modeled uptake rate of 5M FIL per month would reach 80M FIL in sixteen months of uninterrupted inflow. If reachable capital is lower, the gate does not open and the network remains on today's finality rule. Pre-activation rewards still pay for advisory certificates within §3's payout bound, and the chain still bears their blockspace and verification-gas cost (§3a).

The first empirical test is therefore whether staked FIL reaches the 80M FIL security level within two years of the upgrade epoch, as the model assumes. A pool that remains far below that depth would falsify the uptake case.

The addressable-pool estimate uses aggregate circulating-supply figures and revealed-preference comparables from existing yield products; it does not use per-holder data.

**A signup rush cannot overspend the pool, and stake depth is self-limiting.** Additional stake lowers the offered rate, so annual payout remains below the §3 bound regardless of how quickly stake arrives. At a 5% required return, the schedule falls below 5% once about 77M FIL is staked, so a 5% marginal staker stops adding. It falls below 4% around 110M FIL, where the modeled 5% staker begins leaving. In that case stake peaks around 100M FIL, inside the hold level, and settles rather than overshooting indefinitely.

**A pool that stalls below activation keeps the posted rate elevated.** With only 40M FIL of reachable capital, the rate remains above the early-termination bar (*Incentive Considerations*) for 46 months rather than four, another reason for the calendared year-3 review (§5).

Two qualifications apply to these comparisons. First, the bars are denominated in FIL and therefore move with the FIL price. Second, the modeling establishes the *direction* of feedback from consensus rewards into storage-side returns but does not yet establish its magnitude.

**This reduces the network's reliance on PoRep-weighted consensus for fast finality; it does not retire PoRep-weighted consensus.** Once certificates bind, the security behind fast finality comes from the staked pool, whose minimum depth is enforced by the activation gate, rather than from a storage-collateral series that has declined over time. The fallback remains the 900-epoch Expected-Consensus rule (*Appendix B*).

Continued decline in the PoRep layer therefore affects the slower fallback path rather than the stake-backed security of a binding certificate. That is the extent of the claim this proposal makes.

**A stall stops finality from advancing. It cannot undo finality already reached, and it opens no new attack.** A third of registered stake can decline to sign without equivocating: §2 counts registered stake, not participating stake, so §4 never fires, and while the stall runs the chain falls back to the 900-epoch rule it runs on today. So the finality-tolerance gain stated above holds only while the veto goes unexercised — but three things bound what the veto is worth. First, everything already finalized stays finalized; a certificate cannot be recalled by refusing to sign the next one. Second, a reorg still requires storage power, roughly 20% of quality-adjusted power under the fallback rule, exactly what it requires today and no easier, priced regime by regime in the *[storage-only attack analysis](https://hannahhoward.github.io/stake-finality-fip/storage-only-attacks.html)*. So the attack that matters needs *both*: a third of the stake first, and then the same storage-power attack that is available right now without any first step, because F3 is already stalled by apathy at zero cost. This proposal leaves the first lock exactly as it is and adds a second one in front of it. Third, the second lock cannot be picked quietly: certificates stopping *is* the alarm, visible to every exchange and bridge within hours, which is when they fall back to slow confirmations and take most of the prize off the table. What remains is the chance to reverse whatever moved in that shrinking window, while everything certified stays out of reach; for a profit-seeker the arithmetic does not close, and for an attacker who only wants damage, the carry and visibility below are the deterrent — this section claims no more than that.

Acquiring and carrying the veto is capital-intensive but relatively cheap to maintain. Across the funded years, obtaining one-third of the resulting pool requires roughly 30–43M FIL, more than one-third of the pre-attack pool because the attacker's own stake increases the denominator. A staller earns no participation reward (§3), so carrying the position forgoes about 1.42M FIL per year, or roughly 0.08% of the position per week.

Those figures follow the modeled path of the released pool and assumed staker return. If the market requires a return above what the schedule can offer, the activation gate never opens and there is no stake-based veto to exercise.

Relative to today's stalled F3 committee, some factors make a future stall harder and others make it easier. The capital requirement rises from roughly 21.5M FIL of recoverable pledge to about 36M FIL bought at market, and the current zero-cost route — simple non-participation — disappears. On the other hand, the carrying cost falls roughly fivefold *[re-check]*, and a single pseudonymous buyer could replace the eight-to-ten identifiable incumbents whose coordination difficulty is an important deterrent today.

A stall is also self-limiting. Nobody earns participation rewards while it continues, honest stake can exit, and the pool can fall toward §6's deactivation floor, at which point binding finality returns to advisory rather than remaining hostage to the stalled committee. The veto price therefore declines with pool depth, reaching 9.1M FIL by year 25 in the modeled path; the deactivation band bounds that decline (*Appendix B* analyzes a higher floor).

An inactivity leak — burning stake absent from the signer bitset that §3 already exposes — remains out of scope. The model estimates that even a 0.25-percentage-point premium in required return costs two years of the funded security window, while a leak also risks burning honest participants during partitions. **A later FIP should revisit an inactivity leak if a stall is actually exercised or if that premium falls below 0.25 percentage points.**

**Binding finality begins with an explicit dispersion requirement.** Certificates cannot become authoritative until the pool is both deep and dispersed (§6). Today's committee has no comparable dispersion requirement; its current reading is a Nakamoto coefficient of **7** at the one-third threshold, down from 84 in January 2023, with the top ten owners receiving 38.5% of block rewards *[Filecoin Data Portal series behind the public L1-health dashboard, 2026-07-13; re-check]*.

Concentration can still increase after activation. Delegation aggregates control, and Lido reached roughly 23% of all staked ETH according to its February-2026 tokenholder update through a similar aggregation path. For that reason, `N_min` and `max_share` should be re-checked against who actually controls signing weight, not address counts that are cheap to split. The participation API described in *Implementation* provides the required reporting; no client exposes it today.

**The power gained by concentrating stake is limited because this committee controls finality only.** An entity with one-third of stake can stall certificates, producing the fallback behavior priced above. Even an entity with two-thirds of stake cannot produce blocks, censor transactions, rewrite state by itself, or mint FIL, because leader election remains storage-power weighted.

The additional capability at two-thirds is to combine captured finality with a storage-power attack and certify a reorg. That compound attack requires roughly twice the veto's stake capital plus the separate storage-power position, all while the stake concentration is visible in an on-chain table. Concentrated stake therefore controls less of the protocol than concentrated storage power does today, because storage power elects every block.

Two important limits remain. First, the dispersion rules constrain participants, not jurisdictions. Delegated stake is likely to concentrate partly in exchanges and custodians, including regulated companies subject to government pressure. This proposal does not measure or solve that risk; it limits the consequence because compelled custodians can at most stall fast finality back to the network's current fallback without also acquiring storage power.

Second, the automatic remedy is intentionally one-way. A concentration finding after activation does not close the latch (§6); correcting it requires a FIP and network upgrade. The alternative — allowing concentration itself to disable finality — would create a cheaper shutdown lever. Dispersion rules therefore make accidental concentration harder to activate, while pool depth and correlation-scaled slashing (§4) provide the economic defense against actual concentrated control.

## Incentive Considerations

**Uptake is the largest open variable because the schedule buys security by recruiting stake.** If the marginal staker requires a higher return than the schedule offers, the pool may recruit and pay participants without ever crossing the activation threshold, so no certificate becomes binding (§6). The year-3 non-activation review is designed for exactly that outcome and therefore keys on activation rather than pool balance (§5). The modeled scenarios assume 130M FIL of reachable idle capital; the companion analysis provides the comparables behind that figure and the demand cases built on it.

**The effect on storage providers depends on the funding source, which remains open.** The companion analysis allocates each option's costs across constituencies. Under every option, storage providers can benefit in two ways: they may earn the participation reward as FIL holders, and their block producers may earn the includer share for submitting certificates (§3a).

Every option also shares the same review-and-sunset framework: a year-25 review whose stated default is to stop payment, followed by a hard year-30 sunset if payment continues. Schedule constants can be repriced only through a FIP and network upgrade (*Governance*).

**Effect on storage-committed capital.** The reward is intended to recruit *idle* FIL — capital that earns no protocol-native yield today — and the reachable pool is estimated from that idle float. Available evidence suggests that storage-power trends are driven primarily by storage-hardware and deal economics, making participation yield a secondary factor.

The incentive can nevertheless pull capital out of storage through two channels with very different costs:

- **Early termination.** FIP-0098 charges 8.5% of initial pledge for a mature sector. In the model, the posted rate exceeds that hurdle for only four months of the thirty-year program, and a single early termination does not recover the fee at any remaining sector life up to five years.
- **Non-renewal.** Allowing a sector to expire and then staking the returned pledge incurs no termination fee and must only beat the storage provider's net return on committed pledge, modeled at about 3.5% per year at current prices. The posted staking rate exceeds that level throughout the modeled thirty years.

The declining schedule is the primary mitigation and a constraint that the final constants must continue to satisfy. By the time the stake pool is deep, the marginal staking yield falls to about 4.3–4.9% at the security level, below the modeled return on committed pledge at prices above spot (9.5% at $1.00 and 19.6% at $2.00; see the companion analysis). The high initial rate is also capacity-limited because idle capital can enter more quickly than storage-committed capital can unwind.

One limitation remains: `r(S, P)` responds to stake depth, not to the return on storage pledge. If pledge yield falls because of the adopted block-reward schedule or because a funding choice affects it, the staking curve does not automatically adjust. *[needs precise spec]* whether the posted rate should be linked to on-chain pledge yield or whether depth pricing alone is sufficient, with the rationale stated explicitly.

**Delegation is not gaming.** A GLIF-style operator that stakes on behalf of depositors performs the signing work and shares rewards with those depositors; that is intended behavior. The mechanism prevents two different behaviors: earning on idle, non-signing balances through the certificate gate, and equivocation through slashing.

**Curve gaming is bounded.** A large staker could hold back deposits to keep the offered rate higher. The cost is the reward forgone on the withheld stake, while the benefit is limited by the slope of the curve; any unused capacity is also available to another participant at the same posted rate. Splitting one economic position across identities does not increase rewards because the rate is uniform.

Waiting is not unambiguously advantageous. Rewards paid to others reduce `P`, scheduled releases increase it, and new stake increases `S`; in the modeled range the depth term dominates. Stake arriving when the pool is shallow can be offered as much as 12%, while stake arriving around 80M FIL is offered about 4.9%.

## Product Considerations

**Fast confirmation is a feature with a funded lifetime, and integrators can observe its state.** An exchange or bridge that treats a binding certificate as settlement can reduce confirmation time from ~7.5 hours to tens of seconds. That guarantee is funded rather than perpetual. In the companion analysis's modeled scenarios, certificates become binding in years 2–4 and the pool remains near the target security level for roughly 12–14 years, with a first review in year 3 and a year-30 sunset.

Outside the binding state, the network uses today's finality rule (*Appendix B*). Integrators can therefore engineer the fallback they already use today and determine from chain state whether fast finality is authoritative.

**Reaching the stake threshold also requires off-chain software and recruitment.** Custodians and exchanges need a staked-and-accruing balance concept, pull-based claims, and withdrawal terms that reflect the unbonding delay (*Backwards Compatibility*). Delegated pools need operator software for BLS key registration and rotation, certificate-inclusion monitoring, reward distribution to depositors, and disclosure of slashing exposure. Wallets need deposit, key-registration, withdrawal, and claim flows for the staking actor.

All of these depend on the participation API described in *Implementation*, which resolves a certificate's signer bitset into per-participant inclusion.

Nothing in the protocol commits any party to build that software or recruit stake. Likely implementers include operators already running FIL yield products and exchanges or custodians already holding FIL. Recruitment may be the larger challenge: crossing the threshold requires convincing holders of tens of millions of FIL to stake, which depends on outreach and distribution outside the protocol.

If software or recruitment arrives slowly, the consequence is delayed activation rather than a weaker authoritative committee: certificates remain advisory until the threshold is met (§6).

## Implementation

Detailed design is out of scope for this discussion. The actor bundle is the only implementation deliverable shared across node clients: it contains the staking actor, reward pool, certificate-submission and verification path, and slashing logic. Power-table derivation, finalization gating, the participation API, supply accounting, staker-facing tooling, and migration are implemented separately in each client, in two languages, and kept consistent through shared conformance vectors. Block-producer software must also submit certificates.

The participation API maps each certificate's signer bitset to per-participant inclusion. Reward verification, delegated-pool operators, and concentration monitoring in *Security Considerations* all consume that same feed. Only protocol constants depend on the adopted funding source; the implementation path does not.

**Deployment sequence.** The stages occur in this order and are separately observable:

1. **Actor deployment — network upgrade.** The staking actor, certificate-submission path, slashing logic, and funding-side components activate. Staking opens. No F3 committee runs yet, so finality behavior is unchanged. This stage exercises migration and funding logic against real balances on the live chain.
2. **Committee formation and advisory accrual — coordinated client release after the formation floor is reached (§6).** The committee bootstraps on a fresh instance chain. Certificates are produced, submitted on chain, and rewarded; slashing is live. New subsystems with no production precedent — in-actor certificate verification, fault proofs, per-instance committee snapshots, and the signer/includer split — therefore run under real economics before the certificates are authoritative. Per-certificate gas cost (§3a) is measured here, and distribution statistics accumulate history before they gate activation.
3. **Activation — data-triggered (§6).** No separate release is required. Once the depth and distribution conditions have held for the required window, certificates become binding.

Each stage depends only on components exercised in an earlier stage. Deployment risk is therefore separated from committee-operation risk, and committee-operation risk from binding-finality risk. The later stages have no fixed date, so this is an ordering rather than a calendar.

This proposal also amends FIP-0086's committee-definition text for eligibility, power source, and signing key. That amendment should also resolve FIP-0086's quality-adjusted-versus-raw-byte divergence.

## Governance

**Upgradability and parameter governance.** The staking actor is built into the system actor bundle and can change only through a network upgrade adopted by node operators through the FIP process. There is no admin key, multisig, or unilateral upgrade path.

The mechanism's parameters — reward-schedule constants, activation and distribution thresholds, slash constants, and any constants fixed with the funding source — are protocol constants and can be amended only through the same process. This proposal deliberately avoids a runtime-governed tuning actor for consensus-critical parameters because hot-tunable finality economics would create an additional attack surface. If the network prefers runtime governance for these values, that should be evaluated separately with its own security review.

## Appendix A — the open items, by class

Every `[needs precise spec]` marker points to this appendix. `[re-check]` marks a citation, chain snapshot, or modeling input that must be verified before FIP submission. The table below lists **decisions**: open items that can change whether the design works, what it costs, or what risks it tolerates. Other unresolved details are treated as **refinement** bounded by the rules already stated.

| Section | Open item | Note |
|---|---|---|
| §1 | Unbonding delay | Bounded from below twice: by the fault-proof window (§4) and by the economic bound in §1 |
| §1 | Committee identity — worker key, dedicated key, or delegated operator key | Highest fan-out: determines whose stake burns when a delegated operator equivocates (operator bond, delegators' pro-rata stake, or both in sequence — §4) and therefore the slashing terms custodial products must expose (*Product Considerations*) |
| §3 | Per-epoch accrual formula and rounding rule; whether `α` is included in the posted rate or paid on top | Determines what rate a staker is quoted relative to the pool's total cost |
| §3a | Per-certificate gas cost, not yet measured | Measured in deployment stage 2 (*Implementation*); an excessive result would trigger the fallback architecture |
| §3a | Message container and size limits; batching consecutive submissions; sizing `α` against the FIP-0115 base-fee rule | Bounded by the 64 KiB message cap stated in §3a |
| §4 | Base and correlation slash constants | Strawman 1–5% |
| §4 | Fault-proof message format and submission window | The window binds the unbonding delay |
| §3 / §4 | Inactivity leak — burning stake absent from the signer bitset — not included in this proposal | *Security Considerations* prices the tradeoff and states a trigger for revisiting it |
| §6 | `stake_activation` | Partly closed: proposed at the 80M FIL security level, with the modeled peak as an upper bound |
| §6 | `N_min`, `max_share`, and measurement over the operator set | Strawmen defining "dispersed"; values should follow analysis of actual signing control rather than address counts, which are cheap to split |
| §6 | `stake_deactivation`, `W`, `W′` | Strawmen: 50M FIL, three months, and twelve months; *Appendix B* sizes the tradeoff |
| *Incentive Considerations* | Whether the posted rate should be linked to on-chain pledge yield or whether depth pricing alone is sufficient | Determines how the non-renewal channel responds if pledge yield falls |

For activation, the open questions are primarily parameter values rather than the mechanism itself. The hysteresis rule (§6), authority record and readers (*Appendix B*), and stall behavior (*Security Considerations*) are specified; the remaining choices are the constants listed above.

Funding-specific open items belong to the separate funding decision, most importantly the source itself.

## Appendix B — the activation state

**The three states.** The network is always in one of the following states. Block production and chain ordering are identical in all three:

| State | What is true | What a client reports as final | Reward |
|---|---|---|---|
| **Binding** | Latch open, quorum live | The F3 certificate head, within tens of seconds | Paid per certificate |
| **Advisory** | Latch closed — before activation, or after a deactivation — quorum live | The 900-epoch Expected-Consensus rule, ~7.5 hours | Paid per certificate: §3 gates on signatures, not on bindingness |
| **Halted** | No quorum | The 900-epoch Expected-Consensus rule, ~7.5 hours | Nothing; §3's payout stops automatically |

All three states retain today's Expected-Consensus fallback, whose published analysis places adversary tolerance near ~20% of power (*Security Considerations*). That fallback is the baseline contract for integrators. The certificate authority mark below tells an integrator whether a certificate is currently binding or advisory.

**The deactivation band, sized.** The deactivation floor in §6 trades recovery tolerance against the declining price of a stall. A 60M FIL floor would keep the veto price above roughly 20M FIL but end binding finality around year 17. The 50M FIL strawman permits a wider operating band at the cost of a lower guaranteed veto price.

The floor is intended as a safety catch, not a live control. In the companion model, 50M FIL sits roughly 30M below the depth maintained at the 5.5% required-return case. After an injected exit of 60% from a full pool, the model refills to the new equilibrium within seven to nine months. The pool falls below 50M for one month at a 5% required return and three months at 5.5%. A three-month `W′` is therefore close to triggering during a recovery the schedule can complete on its own, whereas a twelve-month window is not. The refill does not restore the pool's former margin above the activation threshold.

*[needs precise spec]* whether `W` and `W′` count epochs or F3 instances. Instances are the natural unit because distribution statistics are computed per committee. Also open is whether the staking actor should continue publishing distribution statistics after activation as a monitored signal with no automatic effect; this proposal assumes that it should.

**The `ActivationRecord`.** The staking actor stores an append-only ordered list of `(opened_at_epoch, opened_at_instance, closed_at_epoch | null, closed_at_instance | null)` entries together with the current latch state. The same consensus rule evaluated by every client at finalization writes the record. The normative binding rule is stated in §6: an instance is binding if an activation entry opened at or before that instance and had not closed by it. This appendix defines the record layout and its readers.

**The same rule serves three readers.** A node tracking chain state reads the record directly; a syncing node uses the same record to reconstruct whether the gate was open for a historical instance. A verifier that holds only certificates instead reads the latch state and active entry from the committee's supplemental-data commitment.

That certificate-only path has an important limitation: the committee is attesting to its own authority. The mark is reliable only after the committee is already authoritative, which is what activation establishes; by itself, it cannot prevent a pre-activation committee from claiming authority. An integrator could pin a published activation epoch, analogous to a network-upgrade height, rather than check the on-chain record for every certificate. Because §6 also permits later deactivation, however, that integrator still needs a trusted way to learn closure events.

*[needs precise spec]* the commitment encoding and whether a certificate-only verifier must check the mark against the on-chain record once at first use before trusting subsequent marks.


## References

- *[Funding the finality reward: options and a recommendation](https://hannahhoward.github.io/stake-finality-fip/funding-options-and-recommendation.html)* — the companion
  analysis: the candidate funding sources compared against §5's four requirements, and the
  worked scenario the rate table and the curve in §3 are drawn at.
- *[Analysis of storage-only attacks under Expected Consensus, QAP Fast Finality, and Stake
  Finality](https://hannahhoward.github.io/stake-finality-fip/storage-only-attacks.html)* — what storage power alone can do under each of the three regimes,
  and what each share of it costs.

## Copyright

Copyright and related rights waived via [CC0](https://creativecommons.org/publicdomain/zero/1.0/).
