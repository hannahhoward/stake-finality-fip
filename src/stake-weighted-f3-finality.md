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

> **Draft for discussion.** Numbers marked *[re-check]* are chain snapshots, citations or
> modeling inputs to be re-verified before submission as a FIP; constants marked *[strawman]*
> are chosen to make the design work; items marked *[needs precise spec]* are deliberately
> left open.

**How to read this.** Four questions, in order.

1. **Do I accept the premise?** The proposal takes as given that the storage base is declining and that the token has to hold through years of storage-economics work. If not, cheaper instruments exist and *Change Motivation* names them; say why and stop there.
2. **Is a stake-weighted finality committee the right first step?** It secures finality only; leader election, block production and PoRep are untouched. If the first step should be something else, that is the feedback to give.
3. **Does the mechanism hold?** A staking actor, a stake-derived power table, a reward paid only for signatures in canonical certificates at a rate that falls as stake arrives, correlation-scaled slashing, and an activation gate on depth and dispersion. Where would you tighten it, and which strawman constants would you set differently?
4. **Is the price worth paying, whatever the source?** The pool needs about 100M FIL of income over the program's life. Where that comes from is left open here. Is slashable, stake-secured finality worth a bill of that size?

Supporting analyses are linked in the comments below.

*The reward this proposal pays needs a funded pool. Several ways to fund it exist and have been
explored in depth, and we leave the source deliberately open here, to assess consensus on the
mechanism and its rationale first. The companion analysis, **[Funding the finality reward: options
and a recommendation](https://hannahhoward.github.io/stake-finality-fip/funding-options-and-recommendation.html)**, carries that exploration; it is not the subject of this
thread. Should consensus emerge that we *want* to do this, we move to the thornier discussion of
*how* to fund it. §5 states what the mechanism requires of any source, and the mechanism activates
only with an accepted funding source adopted together with it.*

# FIP-XXXX: Stake-Weighted F3 Finality

## Simple Summary

This proposal secures the token. The deeper fixes to Filecoin's storage economics — the PDP
market, and a cheaper storage proof or stake-weighted leader election — are years of work each,
and the token has to hold through them. It revives Filecoin's fast-finality gadget (F3) by
deriving its committee voting weight from **staked FIL** instead of storage power and paying a
market-priced participation reward to the FIL that signs finality certificates, from a funded
pool §5 specifies, leaving leader election and Proof-of-Replication (PoRep) unchanged.

## Abstract

This instrument secures **finality**, which chain is irreversible, with staked FIL. In F3
(FIP-0086), finality is determined by a Group-based Practical Byzantine Fault Tolerance (GPBFT)
committee weighted by quality-adjusted storage power, the same resource that decides **leader
election**, which storage provider proposes the next block via Expected Consensus and PoRep.
Participation pays nothing, and without an incentive the committee has struggled to attain
quorum; the network has fallen back to the slower 900-epoch Expected-Consensus rule that
preceded F3's introduction.

The mechanism is a finality committee weighted by staked FIL and paid to sign. GPBFT already
operates over an abstract power table; we re-derive that table from staked FIL rather than
storage power. A new staking actor becomes the on-chain source of truth for stake, equivocation
in GPBFT becomes slashable against it, and the FIL whose signature lands in a certificate earns
a reward: the offered rate is high while staked FIL is shallow and declines as stake arrives, a
posted schedule whose declining rate lets the market set where stake stops arriving, with no
artificial floor. It is a rate on the pool's released balance. The pool needs about 100M FIL of
income over the program's life, and §5 states the four things the mechanism reads from whatever
supplies that money.

Because a fresh staking actor starts empty, staking opens at the upgrade epoch and
stake-weighted finality **activates only once about 80M FIL is staked and dispersed**, specified
in §6.

This proposal covers finality only: leader election, block production and PoRep are untouched.
Stake-weighting *leader election* (a full consensus pivot that would retire PoRep) is a separate
and much larger question, explicitly **out of scope** here.

## Change Motivation

**No FIL is staked.** Filecoin has no native staking mechanism: the FIL securing the network
today is pledge collateral bonded to storage sectors, and none of the liquid float earns a
protocol-native yield. What does stand behind the token and
behind consensus is storage, and the storage base is declining. The decline is public record,
published by the network's own dashboards and priced daily, and this proposal takes it as given.
On that premise the goal is to secure the token. A protocol-native yield gives FIL holders a
reason to hold and stake rather than exit, and the FIL they stake gives finality a security
basis that does not depend on where storage power goes next.

**If the premise is rejected, this is the wrong instrument.** A committee weighted by a stable
resource does not need re-basing. Discussion #1106 restores the same two-thirds threshold at
roughly a storage provider's marginal cost of signing, out of a block-reward slice and no
additional funding (*Prior art*), and a vote-escrow instrument obtains locking
without touching consensus at all. On that premise, those are the correct answers, and they are
cheaper. On the
premise as stated, a quorum patch restores certificates on a base that keeps shrinking, backed
by collateral the protocol returns to an attacker, and touches the token problem not at all.
Weighting the committee by stake instead of storage power is what costs the whole of the funding
bill (priced in the companion analysis).

Capital has shown up voluntarily for credit-exposed instruments (the companion analysis),
which is the appetite this proposal offers a protocol-native home. The posted rate opens under a
12% ceiling and falls as stake arrives (§3), settling at 4.0–5.0% while staked FIL holds the
security level (yield comparables: the companion analysis).

Filecoin's lack of a staking base is also why finality has no security base of its own. FIP-0086 shipped
F3 as a finality gadget running in parallel with Expected Consensus: participants sign GPBFT
messages, and a supermajority (≥⅔ of committee power) produces a finality certificate that
makes a tipset permanently irreversible within tens of seconds, versus the ~900-epoch (~7.5h)
Expected-Consensus rule. FIP-0086 explicitly chose **not** to reward participation, on the
reasoning that the marginal cost of signing is small and participants benefit from faster
finality of their own transactions. In practice the committee lost quorum and F3 stopped
producing certificates; the network has run on Expected-Consensus fallback finality since.
Stake is the base finality lacks: it is slashable, and its depth is a design constant rather than a market outcome.

## Specification

The mechanism comprises a new staking actor (§1), a stake-weighted F3 power table (§2), a
certificate-gated participation reward (§3) carried on chain by a certificate submission
path (§3a), and equivocation slashing (§4); §5 is the ~100M FIL bill and the four things the
funding must supply, and §6 is the stake-threshold activation that governs when stake-weighted
finality becomes authoritative. The funding itself is open. Only the power-table
formula (§2) is a trivial change. The certificate submission
path (§3a), the fault-proof machinery (§4), and the transition (§6) are genuine new protocol
surface, and everything outside the actor bundle is implemented separately in each node client.

### 1. Staking actor (new)

A new built-in actor is the on-chain source of truth for staked FIL. It records, per
participant identity, a staked balance and a registered BLS signing key; it supports
deposit, withdrawal (behind an unbonding delay), and delegation (an operator may sign on
behalf of many depositors); and it maintains the activation state the finality switch
reads (§6), including the append-only `ActivationRecord` that says which finality
certificates bind the chain (*Appendix B*).

**The unbonding delay has two lower bounds.** The first is the slashing bound in §4: the delay
must strictly exceed the fault-proof submission window, so an equivocator cannot withdraw ahead
of the slash. The second is economic. The delay is the only interval over which the pool cannot
drain, and it is what makes a staked balance different from a liquid one: it is set long enough
that an exit is a decision priced against the posted rate rather than an intraday trade
(**strawman: 90 days**), and it is a delay rather than a term. Any delay is a lock for its
duration; this proposal offers no term parameter beyond it, and every figure here and in the
companion analysis is computed on at-will stake.

*[needs precise spec — Appendix A]*

### 2. Stake-weighted F3 power table

GPBFT's ⅔ quorum, its BLS12-381/BDN signature aggregation, its certificate format and signer
bitset, and the modified Expected-Consensus fork-choice rule are all power-source-agnostic.
The change is therefore a one-input swap in power-table construction. Where FIP-0086 computes

```
ParticipantPower ← truncate(0xffff * MinerQAP / TotalQAP)
```

this proposal computes

```
ParticipantPower ← truncate(0xffff * Stake_i / TotalStake)
```

read from the staking actor at the same `PowerTableLookback = 10` lookback (ten GPBFT
instances; the implementation names the constant `CommitteeLookback`), with the same
16-bit normalization and the same certificate machinery. Eligibility cannot be inherited:
today's committee filters (the 10 TiB power floor, zero fee debt, no unelapsed consensus
fault) are all miner-actor state with no meaning for a staking actor, so stake-side
eligibility is a minimum stake plus the staking actor's own fault state (§4). Two encoding
limits bound the minimum stake from below: a participant holding less than 1/65535 of
total stake scales to zero weight and cannot sign, and the CBOR encoding caps the power
table and each certificate's delta at 8192 entries, so the floor must also keep the
committee inside that count. Rewards accrue outside the balance the table reads (§3);
compounding them into the numerator each epoch would put every participant in every
certificate's power-table delta. This weighting binds finality only at activation (§6).

### 3. Participation reward, gated on signed certificates

Staked FIL earns a reward **only** when the participant's signature is included in a valid,
canonical finality certificate, and the reward is denominated in the epochs that
certificate newly finalizes. A certificate that repeats an already-final head finalizes
nothing, for signers and submitter alike. Epochs advance at one per 30 seconds no matter
who holds the committee, so producing certificates faster than the chain grows pays no
more, and stalling pays nothing. Idle stake earns nothing. If F3 halts, payout stops
automatically. The split borrows the inclusion structure sketched in Discussion #1106 (see
*Prior art*): a signer share plus a smaller includer share (α on the order of 10%) paid to
the account that submits the certificate on chain (§3a), in practice the block producer,
since only the producer collecting α has a reason to include a submission. A signer's
reward does not come out of any other signer's, so a producer gains nothing by censoring a
peer's signature and forgoes α by declining to submit the certificate.

The offered rate is a published function of two numbers anyone can read on chain: the depth
of the staked pool, `S`, and the **released** balance of the reward pool, `P`:

```
r(S, P) = spend_rate × P / (S + S_offset)
```

`S_offset` keeps the rate finite when little is staked and sets how quickly it falls as
stake arrives. `P` is the released ledger alone: income lands in an unreleased ledger and
moves to the released one on a fixed schedule (§5), and the rate reads only what has been
released. The two constants are set together with the funding envelope, so their values
follow the source; the table and the curve below are drawn at the worked configuration,
`spend_rate = 0.22` and `S_offset = 55M` (the companion analysis). A 12% ceiling on the launch rate binds every option *[strawman]*.

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

*The posted rate falls as stake arrives, so the first 80M FIL of stake is recruited at a
declining price rather than at one fixed rate. Curve drawn at the same worked configuration.*

The schedule has two further properties:

- **Pool scaling.** The whole schedule moves with the released pool. Paying rewards lowers it;
  each year's release raises it. A quiet year leaves the money
  in the pool and the offer standing, so a slow bootstrap never starves later stakers, and a
  pool filling faster than it pays raises the standing offer until stake answers it.
- **Annual payout is below `spend_rate × P` at every depth**, whatever stake arrives and
  however fast: 6.60M FIL in year 1 in the worked scenario, on a 30M released pool. There is
  no banked budget and no catch-up payment — an epoch that pays nothing is not owed later.

Where the schedule stops recruiting is where the activation threshold (§6) stops being reached;
the year-3 non-activation review (§5) fires if no certificate has ever bound
(*Incentive Considerations* carries that case).

Lifetime payout is bounded by the pool's income, the rate of payment above, and the sunset (§5).

A holder who stakes 10,000 FIL in the tenth month of signing, when 50M FIL is staked
network-wide, is offered 6.29% in the worked scenario: roughly 630 FIL over the following
year, accruing only for epochs in which their signature lands in canonical finality
certificates. Once staked FIL reaches 100M — year 6 on that path, with 34.2M FIL released and
unpaid — the same 10,000 FIL earns 4.85%, about 485 FIL a year. Claims are pull-based, so the
posted rates above are gross of the includer share α and of the gas a staker pays to claim.

The reward accrues from the committee's first certificate, including through the
pre-activation accumulation phase (§6).

*[needs precise spec — Appendix A]*

### 3a. Certificate submission and verification (new)

Nothing F3 produces touches the chain today: certificates live in each node's local store
and travel over peer-to-peer protocols. The reward in §3 requires the chain to see and
verify them, which makes the submission path a component in its own right — the largest
new subsystem in this proposal. Discussion #1106 raised the same question and leaned
toward message submission over embedding certificates in block headers, which must stay
small; this proposal adopts that direction.

The block producer submits each new certificate as an ordinary message to the reward-pool
actor. The actor keeps a per-instance watermark: the first valid submission for an
instance credits its signers and pays the submitter α, and duplicates fail cheaply. A
second certificate for an already-credited instance that decides a different chain is
accepted and credits nothing: it is the on-chain evidence of correlated equivocation (§4).
Submission is permissionless: any account may submit, so suppressing a certificate
requires every producer to forgo α against their own interest.

The actor verifies the certificate itself. The aggregate-signature check is a
coefficient-weighted multi-scalar multiplication over the signer set plus one pairing, and
the BLS12-381 arithmetic for both already ships inside the actor bundle (FIP-0105); no new
syscall is required. The per-certificate gas cost has not been measured; that measurement
is an early implementation deliverable. If it comes back prohibitive, the fallback is
client-side validation feeding the actor a system-attested summary, which adds a
block-validation rule to every client and would depend on FIP-0107. This proposal assumes
in-actor verification.

Resolving a certificate's signer bitset into staker identities requires the ordered
committee table for that certificate's instance. The staking actor is the power source, so
it retains per-instance committee snapshots as part of its state, and the retention window
must cover the fault-proof submission window (§4). §3, §4, and §6 all read the same
structures: the per-instance snapshots, the instance-to-epoch anchoring the certificate
record provides, and the count of epochs each certificate newly finalized.

A chain message is capped at 64 KiB and the certificate format permits larger power-table
deltas, so the submitted deltas are chunked or the table reads stake snapshotted at a bounded
cadence. *[needs precise spec — Appendix A]*

### 4. Slashing for equivocation

Double-signing (equivocation) in GPBFT — signing conflicting messages for the same instance —
is slashable against the offender's staked FIL. This is what gives stake its security edge
over storage power: an attacker's capital can be *burned*, whereas acquired storage power is
recoverable. GPBFT is an explicit-signature protocol, so a pair of conflicting signed
messages attributes the fault to a key beyond dispute. Nothing collects that pair today:
F3 nodes drop a conflicting message on receipt and retain nothing, and no GPBFT message has
ever been verified on chain. Slashing therefore adds a fault-proof subsystem: nodes retain
and gossip conflicting message pairs, anyone may submit a pair to the staking actor, and
the actor reconstructs the signed payloads (their encoding is normative in FIP-0086) and
verifies both signatures against the registered key. The correlated case needs no
collection at all: two valid certificates for the same instance over different chains are
themselves proof that every signer in both bitsets double-signed, and certificates are
already on chain (§3a).

The slash scales with correlation. A
small base slash (strawman 1–5%, plus committee ejection) applies to isolated equivocation —
an operator accident must be survivable, or the penalty deters participation instead of
attack. Ejection takes effect at the power-table lookback, ten instances after the fault is
recorded, because the table for in-flight instances is already fixed. The slash then scales
with the total stake that equivocates within the same window, reaching a **total (~100%)
burn as the equivocating fraction approaches the ⅓ needed to threaten finality**. Because
the scale grows as further proofs arrive, slashes sit in escrow until the window closes and
execute at the final scale. A mass event is the other reason for the escrow: simultaneous
equivocation by a third of the committee is what an attack looks like, and also what a
network partition or a defective release looks like. An actual finality attack costs
approximately the attack stake itself: roughly ⅓ of the pool, tens of millions of FIL at
the activation threshold. The security case requires that this exceed the double-spend
value fast finality protects at any time, and the activation band should be re-checked
against settlement volume as it grows.

Whoever submits a valid fault proof receives a reporter share of the slashed amount,
clamped to what is actually burned, as today's consensus-fault report clamps its reporter
reward to the funds it burns. Without a reporter share nobody pays gas to police the
committee. The remainder is burned; returning it to the reward pool would raise the
offered rate through `P`, so an attack would subsidize every other staker. Slashing is
live from the committee's first certificate, before activation (§6). The unbonding delay (§1) must strictly
exceed the fault-proof submission window, so an equivocator cannot withdraw ahead of the
slash; the window itself is defined over instances and anchored to epochs by the
certificate record (§3a). Under delegation the signing key is the operator's. Which stake
burns (the operator's bond, the delegators' pro-rata share, or both in some order) follows
from the committee identity model — worker key, dedicated key, or delegated operator key —
which *Appendix A* records as open.

*[needs precise spec — Appendix A]*

### 5. What the funding has to supply

**The reward pool needs about 100M FIL of income over the program's life** — the twelve-year cost
of holding the 80M FIL security level the activation gate reads (§6), and a strawman. Less money
buys a shallower pool held for fewer years, and what the network spends on consensus in total
rises with the figure.

The mechanism reads four things from whatever supplies the money.

- **A reward-pool actor**, built-in and with no controller, keeping two ledgers, **unreleased**
  and **released**. Only the released ledger is spendable, and only the released ledger is what
  `r(S, P)` reads (§3).
- **A release schedule** that moves balance from the unreleased ledger to the released one on a
  fixed table with no chain-state input, so the posted rate is a published function of numbers
  anyone can read. Pacing the release is what holds the opening rate down, and the rate steps
  with it (§3).
- **Netting conditions on supply accounting.** The pool's whole unpaid balance is netted out of
  circulating supply rather than a source-tagged sub-ledger, and both new singletons — the pool
  and the staking actor (§1) — are added to the circulating-supply traversal, without which
  `StateCirculatingSupply` fails outright.
- **Review and sunset hooks.** A calendared funding review at year 25, a year-30 sunset at which
  payment stops and the pool's remainder is disposed of, and (the one this mechanism turns on) a **year-3 non-activation review**, triggered if the activation conditions (§6) have never been met and
  annually thereafter until they are. It catches a pool that recruits stake, pays for advisory
  certificates, and never reaches the level at which any of them binds, and it is the amendment
  path for a schedule that has bracketed the market's required return wrongly.

**The funding source is open.** This proposal does not choose one. The companion analysis
compares the candidate sources against the four requirements above and recommends one, and every
illustration in this proposal is drawn at that analysis's worked scenario. Adopting this
mechanism requires an accepted funding source, adopted together with it.

### 6. Transition and activation

At the upgrade epoch the staking actor is new and total staked FIL is ~0. F3 will not
start without a non-empty initial power table, and a nearly-empty committee is worse than
none: a single participant is a ⅔ quorum. *Implementation* states the deployment sequence in
order; this section specifies the conditions each stage turns on.

We introduce a simple stake threshold for F3 activation. What the accumulation period costs is
the pre-activation reward paid for advisory certificates, and the blockspace and gas of
carrying them on chain (§3a; see also *Security Considerations*).

- **Staking opens at the upgrade epoch; the committee bootstraps when it can form.**
  Stake-weighted F3 begins as a fresh instance chain: a new F3 network name, a bootstrap
  epoch, and a pinned initial power table, shipped in a coordinated client release. The
  release goes out once registered stake passes a formation floor set far below the
  activation threshold *[needs precise spec]*. F3's own mainnet rollout staged its code
  and its activation separately and pinned the initial power table in a follow-up release;
  the formation-floor trigger itself is new. A fresh chain also retires the QAP-era
  certificate stream cleanly: verifiers, bridges, and snapshots built against it reject
  stake-era certificates outright rather than mistaking them for authoritative. (A handoff
  inside the existing chain, one certificate replacing the whole power table, is
  representable, but it must be signed by two-thirds of the old QAP-weighted committee —
  the quorum this proposal exists to replace.) From the first signed certificate, stakers
  earn the participation reward (§3). This is what makes the pool accumulate — without a
  reward during accumulation there is no reason to stake, and the threshold would never be
  reached.
- **Certificates do not bind Expected-Consensus fork-choice until the activation conditions
  are met: `total_staked ≥ stake_activation` *and* the distribution conditions below.**
  Until then the network uses the 900-epoch rule (= today) and the committee's certificates
  are advisory only. This closes the *thin-but-trusted* hazard: a small stake pool whose
  quorum is cheap to acquire never holds authority anyone relies on. The staking actor
  maintains the activation state (total stake, the distribution statistics, and the
  latch), and every client reads one field at its finalize step, so the gate is
  deterministic, and advisory pre-activation certificates are suppressed in the clients'
  finality reporting. The depth threshold and the latch have a precedent in the power actor's
  minimum-miner count, a consensus rule of the same shape maintained incrementally since
  genesis. The distribution statistics have no incremental form: they are a sort over the
  operator set, and that is the part with no precedent. Once the conditions are met, stake-weighted fast finality
  becomes authoritative: by then the pool is deep enough, and dispersed enough, that acquiring ⅓
  of it is expensive.
- **Activation requires distribution, not only depth.** Pool depth makes ⅓ expensive only if
  the stake is actually dispersed; a deep pool concentrated in a few hands is cheap to tip.
  The gate therefore also requires (strawmen, *[needs precise spec]*): the smallest coalition
  of distinct participants controlling ⅓ of stake must exceed `N_min` participants, and no
  single participant may hold more than `max_share` of the pool — both measured over
  delegated *operators* (the unit that actually signs), not raw addresses. On-chain
  identities are cheap to multiply, so these conditions are **necessary, not
  sufficient**: they prevent a merely *accidental* concentration from being switched on and
  force a deliberate attacker to at least manufacture apparent distribution; the economic
  backstop remains depth plus correlation-scaled slashing (§4).

**The latch opens and closes on asymmetric conditions, so activation is one-way in practice.**
It opens when `total_staked ≥ stake_activation` and the distribution conditions hold for every epoch of a
sustained window `W` (strawman: three months), so a single deep epoch cannot open the gate.
It closes only when `total_staked < stake_deactivation` for every epoch of a much longer
window `W′` (strawmen: 50M FIL and twelve months). **A distribution failure never closes the
latch.** If concentration could turn binding finality off, an adversary who acquired or
manufactured concentration would hold a switch requiring neither ⅓ of stake nor any
equivocation, which is cheaper than stalling the committee; concentration after activation is
answered by correlation-scaled slashing (§4) and, at the limit, by a FIP. The deactivation
band is a safety catch rather than a live control, sized so an ordinary exit and refill does
not trip it; a higher floor holds the veto price up and ends binding finality sooner, which
*Appendix B* sizes.

The latch's history is the authority rule: the `ActivationRecord` (§1) is an append-only
list of open and close events, and **a certificate for instance `i` is binding exactly when
the record shows an entry opened at or before `i` and not closed by it.** *Appendix B* carries
the record's layout and the certificate authority mark that resolves against it.

*[needs precise spec — Appendix A]*

### Prior art

GitHub **Discussion #1106, "Incentivisation of F3" (Kubuxu, Jan 2025)** proposes tying **up to
~10% of block rewards** to F3 participation, proven by evidence embedded in on-chain finality
certificates and split between signers and the block producers who include them. It remains an
open discussion, never adopted as a FIP, and its own text says the total rewards available to
storage providers remain unchanged, so it redistributes among them. This proposal borrows its
inclusion and anti-gaming structure and its message-submission direction (§3a); where the two
differ, and what the difference costs, is argued in *Change Motivation*.

## Design Rationale

The security half turns on what stands behind the consensus threshold, not on where the
threshold sits. Acquired storage power transfers wholesale at no protocol fee, its pledge is
recoverable collateral, and the only penalty for manipulating chain selection, on provable
double-fork mining, is a fixed slash of roughly 4 FIL. Staked FIL that equivocates is burned, at a
rate that scales with the equivocating fraction (§4). At the activation threshold the two
stand near the same nominal size — roughly 21.5M FIL of recoverable pledge behind a third of
storage power *[re-check]*, 26.7M of burnable stake behind a third of the pool — and opposite in what an
attacker forfeits. And staked FIL stops shrinking: the pool's size is a design constant rather
than a market outcome. (The contrast covers safety, not liveness — a stall needs no equivocation
under either design; *Security Considerations* prices it. The 26.7M is computed at the base and
correlation slash constants in §4.)

**Why finality, and only finality.** The two functions can be weighted by different resources.
Stake-weighting finality is cheap and additive because GPBFT already runs over an abstract
power table; stake-weighting leader election would mean retiring or replacing PoRep, an
enormous and separate undertaking. Almost all of the security and value benefit of a
"stake-secured Filecoin" is captured by securing finality alone, leaving the storage market's
proof machinery intact. If future PoRep research yields a better leader-election proof,
upgrading that proof while finality stays stake-secured is again a separate proposal.

**What this does and does not build toward stake-based consensus.** Adopting this is not a
decision to move Filecoin to proof-of-stake consensus. It builds four things such a transition
would otherwise have to build first: an on-chain stake registry with deposit, delegation and
unbonding; equivocation slashing with fault proofs and correlation scaling; an assembled pool
of staked FIL of known depth and measured distribution; and operating experience running a BFT
gadget whose committee weight comes from stake. It does not build leader election. Deciding
block production by stake is the hard part of any pivot, and it brings its own defenses —
grinding, nothing-at-stake, long-range attacks — none of which arise here, precisely because
leader election remains PoRep/QAP-weighted and finality is explicit-signature BFT, where the
failure mode is equivocation (slashable) rather than costless simulation. If the network ever chose
that pivot, the substrate built here would shorten it, and the remaining step would still be
the largest one.

**Why a stake threshold rather than a QAP→stake ramp.** A blended power table (storage power
fading into stake) could restore fast finality immediately, storage-secured. But fast finality
is already offline, so there is nothing to restore *urgently*, and a blend would reintroduce
QAP into the finality table and require paying storage signers through the transition. The
threshold is simpler: pure stake-weighting from the start, made authoritative only when the
pool is deep enough to be safe.

**Why the reward gates on certificates.** Paying registered stake regardless of work would
recreate F3's free-rider problem. Gating on inclusion in a valid canonical certificate buys a
real service, a finality signature over the real chain, and self-corrects: no certificates, no
payout.

### Alternatives considered and set aside

#1106's block-reward slice is the cheaper proposal, and the fork between it and this one is
argued in *Change Motivation*; the funding-side alternatives — other sources, other envelopes,
and the designs that reach the requirement another way — are in the companion analysis.

**Why the offered rate has no floor.** Ethereum's staking yield follows an
inverse-square-root curve that stays near 1.5% however much ETH is staked, so — in the words
of EIP-8363, *Tapered Issuance Burn* (Draft, created 2026-07-14) — there is "no point at
which the incentive to stake switches off," and repairing that later means bolting a burn on
top of issuance against the products built on the base rate. `r(S, P)` is strictly decreasing
in staked FIL with no floor and approaches zero, the pool that funds it is depleted by its
own payments, and the whole instrument sunsets at year 30 (§5): the offer switches itself off
without a governance decision.

## Backwards Compatibility

- **Supply accounting.** The reward pool and the staking actor are new singletons, and the
  netting conditions in §5 are what any funding source has to satisfy. What a funding source
  itself changes belongs to the funding decision.
- **F3 certificate format, verification, and the ⅔ quorum rule: unchanged. The certificate
  chain restarts.** The stake-weighted committee begins a fresh instance chain under a new
  F3 network name (§6); consumers of the QAP-era chain (bridges, verifying contracts,
  F3-aware snapshots) re-initialize against it. Only the power table's *input* changes from
  QAP to stake.
- **Finality committee membership changes.** Committee seats move
  from "storage providers with ≥ threshold QAP" to "identities with ≥ threshold staked FIL."
  Integrators plan against a switch that has no date, since the finality switch is
  data-triggered (§6): a consensus rule every client evaluates identically from staking-actor
  state. Today the equivalent switch is build-time configuration.
- **Every client implements this independently.** The actor bundle is the only shared
  deliverable; the rest lands once per client, held identical by shared conformance
  vectors (*Implementation*).
- **Finality reporting moves once, at activation.** With advisory certificates suppressed
  (§6), the finalized/safe tipset a node reports jumps forward by up to ~900 epochs in a
  single step when certificates become binding; anything computing confirmations from
  those tags sees that discontinuity once.
- **Off-chain consumers.** Staked FIL leaves the holder's account balance and lives in
  actor state, so wallets, custodians, and exchanges need a staked-and-accruing balance
  concept; the circulating-supply RPC must classify the two new actors; and
  storage-provider F3 participation software is no longer eligible to participate.
- **No liveness or finality regression.** Block production is never affected, and the finality
  floor is today's 900-epoch Expected-Consensus rule (the *Halted* row in *Appendix B*) until
  stake makes fast finality strictly better.

## Test Cases

*[to be expanded]* Five test families, with the invariant each must establish, all cross-client;
the funding side has its own:

- **Actor lifecycle.** Deposit, delegated deposit, withdrawal behind the unbonding delay and key
  registration leave staked balances and the operator table consistent.
- **Power-table derivation.** The normalized 16-bit table matches
  `truncate(0xffff * Stake_i / TotalStake)` at the lookback, resolves the certificate's signer
  bitset to the same participants for every client, and a certificate signed by ≥⅔ of stake
  verifies while one below ⅔ does not.
- **Certificate-gated reward.** A signature in a canonical certificate accrues at `r(S, P)` on
  the current depth and released ledger for exactly the epochs that certificate newly finalizes,
  counted once across overlapping certificates; non-signers, base-only certificates, replays and
  halts accrue nothing; and annual payout never exceeds `spend_rate × P`.
- **Slashing and fault proofs.** A valid conflicting pair slashes the signer and pays the
  reporter within the clamp; an invalid pair is rejected before signature-verification cost; a
  proof past the window is rejected; escrowed slashes re-scale as further proofs arrive and
  execute at window close, and slashed stake cannot leave through the unbonding queue.
- **Activation latch.** Certificates are produced but do not bind fork-choice until depth and
  both distribution conditions hold for every epoch of `W`; they stop binding only after
  `stake_deactivation` holds for every epoch of `W′`, and never on a distribution condition
  failing after activation; and a historical instance resolves to binding or advisory against
  the `ActivationRecord` alone, identically for a syncing node and for a certificate-only
  verifier.

## Security Considerations

**Stake-weighted finality adds a large, slashable security budget, and the handoff happens in
year one.** Under the companion analysis's two demand scenarios — 80–130M FIL staked from
year two through the first decade, depending on the return stakers require — that much
economically-slashable capital secures finality, a security layer that does not exist today.
The proposed threshold, the 80M FIL
security level, sits where a ⅓ finality coalition of the pool must put ~27M FIL of slashable
capital at risk.

What an attacker forfeits is a third of staked FIL; what an attack could earn is a year of
storage-provider consensus reward. The two cross in the first year of signing, and the gap
widens every year afterwards. The staked series is the worked scenario's; the consensus-reward
series is the block-reward schedule already adopted, FIP-0118 as passed.

| Year from finality launch | 1 | 2 | 3 | 4 | 5 | 6 |
|---|---|---|---|---|---|---|
| One-third of staked FIL (M) | 20.0 | 26.7 | 28.3 | 31.7 | 31.7 | 33.3 |
| Storage-provider consensus reward that year, FIP-0118 as passed (M) | 15.2 | 9.5 | 6.8 | 6.0 | 5.4 | 4.8 |

The crossing holds at both modeled required returns, and it happens before the activation gate
opens in year 2 on the modeled path (§6), so the slashable budget is already larger than the prize when certificates
start to bind. A funding choice that lowers the consensus reward widens the gap.

**Slashing must bite.** Stake is only stronger than storage power if equivocation slashing is
real and enforceable. Attribution is tractable, by a conflicting signed pair or by conflicting
certificates (§4); the fault-proof subsystem and the slash fraction decide whether the
deterrent holds.

**While the pool accumulates, the network's finality is exactly today's status quo.** No
committee runs until registered stake passes the formation floor (§6), and the committee is thin
for some time after it forms; certificates do not bind until the pool is deep, so a thin
committee is never trusted and liveness is unaffected throughout. Finality
meanwhile is the Expected-Consensus fallback, whose published security analysis places adversary
tolerance near **~20% of power, versus ~33% with a live finality gadget** (the ~20% is from
Wang, Azouvi and Vukolić, AFT 2023; the ~33% is the BFT bound). The network is in that condition
today, and no adopted FIP ends it; this proposal does. What participation decides is *duration*: how long the pool takes to reach the
threshold, and so when fast finality returns rather than whether the network is safe
while it accumulates. Both inputs behind any estimate — reachable idle float, uptake rate — are
low-confidence today (*Incentive Considerations*), which is why the design answers duration with
a date rather than a forecast: the year-3 non-activation review (§5) fires if no certificate has
ever bound, reports what the pool did over the trailing year, and carries the
amendment path. If reachable idle capital is at least the activation threshold, plausible uptake
fills the pool in one to a few years: at the modeled 5M FIL a month of arrival, 80M FIL takes
sixteen months of uninterrupted inflow. If it is not, the gate never fires and the network keeps
today's finality. Pre-activation rewards still pay for advisory certificates, under §3's payment
bound and drawn only on the pool, and the chain still carries the blockspace and verification gas
of those certificates (§3a). The first test is whether staked FIL reaches the 80M FIL security level
within two years of the upgrade epoch, which is where the model puts it, and depth far short of
that is the result that would falsify the case.

Sizing the addressable pool uses aggregate circulating-supply figures and revealed-preference
comparables from existing yield products; no per-holder data is involved.

**A signup rush cannot overspend the pool, and depth is self-limiting.** Arriving stake lowers
the rate one for one, so annual payout stays under §3's bound whatever arrives. At a 5%
required return the schedule posts under 5% once 77M FIL is staked, so a 5% staker stops adding
there, and under 4% once 110M FIL is staked, which is where a 5% staker starts leaving. Stake
peaks at 100M FIL in that case, inside the hold level, so the pool settles rather than
overshooting and unwinding.

**A pool that stalls short of the gate holds the posted rate high.** At 40M FIL of reachable
capital the rate stays above the early-termination bar (*Incentive Considerations*) for 46
months rather than four, which is the other reason the year-3 review (§5) is calendared. Two
limits on all of the above: the bars are denominated in FIL and move with the FIL price, and the
feedback from consensus reward into the storage-side return is a direction the modeling has
established rather than a quantity it has settled.

**This reduces the network's reliance on PoRep-weighted consensus; it does not retire it.**
What changes is that fast finality stops decaying with quality-adjusted power: the security
behind a binding certificate is the staked pool, whose level is a design constant the
activation gate enforces, rather than a collateral series that has fallen every year. What
does not change is the fallback, the 900-epoch Expected-Consensus rule (*Appendix B*). The continued
decline of the PoRep layer therefore costs the network a slower fallback rather than the loss of finality
altogether, and that is the whole of the claim this proposal supports.

**A stall stops finality from advancing. It cannot undo finality already reached, and it opens
no new attack.** A
third of registered stake can decline to sign without equivocating: §2 counts registered
stake, not participating stake, so §4 never fires, and while the stall runs the chain falls
back to the 900-epoch rule it runs on today. So the finality-tolerance gain stated above holds
only while the veto goes unexercised — but three things bound what the veto is worth. First,
everything already finalized stays finalized; a certificate cannot be recalled by refusing to
sign the next one. Second, a reorg still requires storage power, roughly 20% of
quality-adjusted power under the fallback rule, exactly what it requires today and no easier,
priced regime by regime in the *[storage-only attack analysis](https://hannahhoward.github.io/stake-finality-fip/storage-only-attacks.html)*.
So the attack that matters needs *both*: a third of the stake first, and then the same
storage-power attack that is available right now without any first step, because F3 is
already stalled by apathy at zero cost. This proposal leaves the first lock exactly as it is
and adds a second one in front of it. Third, the second lock cannot be picked quietly:
certificates stopping *is* the alarm, visible to every exchange and bridge within hours,
which is when they fall back to slow confirmations and take most of the prize off the table.
What remains is the chance to reverse whatever moved in that shrinking window, while
everything certified stays out of reach; for a profit-seeker the arithmetic does not close,
and for an attacker who only wants damage, the carry and visibility below are the deterrent —
this section claims no more than that.

Bought and carried, the veto is expensive in capital and cheap to hold. A third costs 30–43M
FIL across the funded years, more than a third of the pool, because the attacker's own stake
inflates the total it must hold a third of. A staller earns nothing (§3), so holding the
position forgoes about 1.42M FIL a year, 0.08% of it per week. Those figures follow the modeled
path of the released pool and the return stakers require; at a required return the schedule
cannot meet, the gate never opens and there is nothing to veto.

Against today's veto, some of what changes makes the stall harder and some makes it easier. The
capital bar rises from roughly 21.5M FIL of recoverable pledge to 36M bought at market, and the
free route closes: F3 has already sat stalled for months on simple non-participation. Against
that, the carry falls roughly fivefold *[re-check]*, and one pseudonymous buyer
replaces the eight-to-ten identifiable incumbents whose coordination difficulty is today's real
deterrent.

A stall is also self-limiting: nobody earns while it runs, honest stake exits, and the pool
descends toward §6's deactivation floor, where the stall ends as a return to advisory rather
than a hostage state. The veto's price decays with the pool, to 9.1M FIL by year 25, and the
deactivation band bounds that decay (*Appendix B* prices a higher floor). An inactivity leak, burning stake
absent from the signer bitset §3 already produces, stays deferred: a
0.25-point premium on the required return costs two years of the funded security window, and
a leak burns honest partition victims. **A later FIP takes it up if a stall is exercised, or
if that premium falls below 0.25 points.**

**Binding finality starts at least as spread out as today's committee, by rule, not by hope.**
Certificates do not become authoritative until the pool is both deep enough and
dispersed enough (§6), however much FIL has arrived. Today's committee has never had a
dispersion requirement at all, and its current reading is a Nakamoto coefficient of **7** at
the ⅓ threshold, down from 84 in January 2023, with the top ten owners holding 38.5% of
block rewards *[Filecoin Data Portal series behind the public L1-health dashboard,
2026-07-13; re-check]*. After activation the pool can still drift toward
concentration: delegation aggregates, and Lido reached roughly 23% of all staked ETH (its
own February-2026 tokenholder update) through exactly the route this proposal opens. That is why the dispersion constants (`N_min`, `max_share`)
are continuously re-checked against analysis of who actually controls signing weight rather
than against address counts, which are cheap to fake. That reporting is the participation API named in
*Implementation*; no client ships it today.

**The prize for concentrating stake is small, because this committee only does finality.**
Whoever holds a third can stall certificates — the priced attack above, which reverts the
chain to today's rules at a running cost. Whoever holds two-thirds still cannot produce a
block, censor a transaction, rewrite state, or mint a FIL, because leader election stays with
storage power; the one thing that position adds is the compound attack already priced above —
pairing captured finality with a storage-power attack to cement a reorg, at roughly twice the
veto's capital, accumulated in a public on-chain table. Concentrated stake therefore controls
strictly less of this chain than concentrated storage already does: today's seven owners hold
a third of the resource that elects every block.

Two honest limits remain. The dispersion rules bound participants, not countries: delegated
stake will tend to sit with exchanges and custodians — regulated companies a government can
lean on — and nothing here measures or fixes that; what bounds the damage is the small prize
above, since compelled custodians can at worst stall finality back to today's network. And
the remedy runs one way: activation does not reverse on a concentration finding (§6), so
answering one after the fact takes a FIP and a network upgrade — where the storage side's
answer to a committee it could not sustain was simply to let F3 lapse, which is the situation
today. Dispersion rules make it expensive to *appear* decentralized; the pool's depth and
correlation-scaled slashing (§4) are what make it expensive to *be* concentrated.

## Incentive Considerations

**Uptake is what the schedule buys security with, and it is the largest open variable.** If the
marginal staker requires more than the schedule offers, the pool recruits and pays but never
crosses the activation threshold, so no certificate binds (§6). That is the case the year-3
non-activation review answers, which is why it keys on activation rather than on the pool
balance (§5). The scenarios assume 130M FIL of reachable idle capital; the companion analysis
carries the comparables behind that figure and the demand cases run against it.

**What this asks of storage providers depends on the funding source, and that is open.** The companion analysis prices each option's ask of each constituency. Under every
option storage providers gain twice: they are eligible for the participation reward as holders,
and their block producers earn the includer share for submitting certificates (§3a). Every
option shares the sunset: the year-25 review's default is that payment stops, and the
schedule constants are repriced only by FIP and network upgrade (*Governance*).

**Effect on storage-committed capital.** The reward targets *idle* FIL — capital earning no
protocol-native yield today — and the reachable pool is sized from that idle float. Trends in
storage power are driven primarily by storage-hardware and deal economics on the evidence to
date, which makes the participation yield a second-order factor. The pull it does exert runs
through two channels priced very differently. Early termination pays FIP-0098's fee, 8.5% of
initial pledge for a mature sector. The posted rate sits above that bar for four months of the
thirty years, and one early termination does not clear it at any remaining sector life up to
five years. Non-renewal is the exposure. Letting a sector expire and staking the returned pledge pays
no fee and clears a much lower bar, the sunk provider's net return on committed pledge, about
3.5% a year at current prices, which the posted rate exceeds in every month of the thirty years
modeled.

The declining schedule is the mitigation, and a design constraint the constants have to keep
meeting: by the time the pool is deep, the marginal yield has fallen to 4.3–4.9% at the security
level, below what committed pledge earns at any price above spot (9.5% at $1.00, 19.6% at $2.00;
the companion analysis carries the series). So the standing offer to storage-committed capital is
never large for long, and the transient top-of-schedule rate is
capacity-limited (the pool fills from idle float faster than slow-moving storage capital can
respond). Depth pricing is blind in one direction: `r(S, P)` reads pool depth and nothing else,
so if the pledge yield falls, whether from the adopted block-reward schedule or from a funding
choice that touches it, the posted rate does not know it. *[needs precise spec]* whether the
posted rate should be tied to the on-chain pledge yield, or whether depth pricing alone is
accepted as sufficient, with the reason stated.

**Delegation is not gaming.** A GLIF-style operator staking on behalf of depositors performs
the real signing work and shares the reward with depositors; this is expected and healthy. What
is disallowed is collecting on idle balances (prevented by the certificate gate) and
equivocation (slashed).

**Curve gaming is bounded.** A large staker could under-stake, holding back deposits to keep
the offered rate high; the cost is the volume it forgoes, the benefit is
bounded by the schedule's slope, and the gap it leaves is available to any other
participant at the same posted rate. Identity-splitting earns nothing (the rate is uniform
across participants). Waiting cuts both ways: rewards paid to others while you wait lower the
schedule, while releases raise it, and the depth term dominates both —
stake that arrives while the pool is shallow is offered 12% and stake that arrives at 80M is
offered 4.9%.

## Product Considerations

**Fast confirmation is a feature with a funded lifetime, and integrators can read it.** An
exchange or bridge that treats a certificate as settlement cuts confirmations
from ~7.5 hours to tens of seconds. That guarantee is funded rather than perpetual: on the
companion analysis's modeled scenarios, certificates bind from years 2–4 and the pool holds the
security level for 12–14 years, with a first review at year 3 and a year-30 sunset. Past that window the pool no longer holds the security level. Every state below
binding is today's network (*Appendix B*), so the fallback an integrator engineers for is the
one it runs on now.

**Getting stake to the threshold takes additional off-chain software — and a recruitment
campaign.** Custodians and exchanges need a staked-and-accruing balance concept, a path for
pull-based claims, and the unbonding delay reflected in withdrawal terms (*Backwards
Compatibility*). Delegated pools need
operator software: BLS key registration and rotation, certificate-inclusion monitoring, a
reward split back to depositors, and a stated slashing exposure. Wallets need deposit, key
registration, withdrawal and claim against the staking actor's methods. All three build on the
participation API named in *Implementation* — the feed that turns a certificate's signer
bitset into "this participant signed."

Nothing here commits any party to that work. The software candidates are the operators
already running FIL yield products and the exchanges and custodians already holding FIL. The
recruitment is arguably the larger endeavor: the threshold is crossed by persuading holders
of tens of millions of FIL to stake, which is outreach and marketing work no protocol text
performs. If either
arrives late, the consequence is duration rather than risk: certificates stay advisory until
the threshold is met (§6).

## Implementation

Detailed design is out of scope here. The actor bundle is the only deliverable the
node implementations share — the staking actor, the certificate submission and verification
path, and slashing all ship inside it, alongside the reward pool (§5). The power-table derivation, finalize gating, participation API, supply accounting,
staker-facing tooling and the migration are each implemented per client, in two languages, held
identical by shared conformance vectors; block-producer software must also submit certificates.
The participation API translates each certificate's signer bitset into per-participant
inclusion, and it is the one feed that payment checking, pool operators, and the concentration
monitoring in *Security Considerations* all read.
Only the constants depend on the adopted funding source; the code path does not.

**Deployment sequence.** In order, each separately observable.

1. **Actor deployment**, at the network upgrade. The staking actor, the certificate submission
   path and slashing activate, together with everything the funding side deploys. Staking
   opens. No committee runs, and finality behavior is unchanged. This stage exercises the
   migration and the funding side against real balances on a live chain.
2. **Committee formation and advisory accrual**, at a coordinated client release triggered by
   registered stake passing the formation floor (§6). The committee bootstraps on a fresh
   instance chain; certificates are produced, submitted on chain and paid for; slashing is
   live. The subsystems with no production precedent —
   in-actor certificate verification, the fault-proof path, per-instance committee snapshots,
   the signer and includer split — run under real economics while a failure costs the network
   no finality it is not already living without. The per-certificate gas cost (§3a) is measured
   here rather than estimated, and the distribution statistics accumulate a record before they
   gate anything.
3. **Activation** (§6), data-triggered, with no release of its own: depth and distribution
   conditions met, certificates bind.

No stage relies on a stage that has not yet run, so deployment risk is separated in time from
finality risk and finality risk from binding risk. Neither of the later stages carries a date,
so the sequence is a structure rather than a schedule.

This proposal also amends FIP-0086's committee-definition
text (eligibility, power source, signing key), and the amendment should fix that FIP's
quality-adjusted-vs-raw-byte divergence in passing.

## Governance

**Upgradability and parameter governance.** The staking actor is a built-in actor: it ships
in the system actor bundle and changes only through a network upgrade — adopted by node
operators, governed by the FIP process. There is no admin key, no multisig, and no
unilateral upgrade path. The mechanism's parameters (the schedule constants, the activation and
distribution thresholds, the slash constants, and the constants fixed with the funding source) are
protocol constants, amendable only the same way. The proposal introduces no governed tunable-knob actor for
consensus-critical parameters, because hot-tunable finality economics would themselves be
an attack surface. Some reward designs route weights through a governance-tunable table;
this one does not. If the network prefers runtime-governed parameters, that is a separate
discussion with its own security review.

## Appendix A — the open items, by class

Every `[needs precise spec]` marker in the text points here, and `[re-check]` markers
flag a citation or a chain snapshot to **verify** at submission. Listed below are the **decisions** —
the items that change whether the design works, what it costs, or what it is safe against.
Everything else is **refinement**, mechanical and bounded by rules already stated in the text.

| Section | Open item | Note |
|---|---|---|
| §1 | Unbonding delay | Bounded from below twice: by the fault-proof window (§4) and by the economic bound in §1 |
| §1 | Committee identity — worker key, dedicated key, or delegated operator key | Highest fan-out: decides whose stake burns when a delegated operator equivocates (operator's bond, depositors' pro-rata, or both in order — §4), which is what custodial product terms follow from (*Product Considerations*) |
| §3 | Per-epoch accrual formula and its rounding rule; whether α is paid inside the posted rate or on top of it | Decides what a staker is quoted against what the pool pays |
| §3a | Per-certificate gas cost, unmeasured | Measured at deployment stage 2 (*Implementation*); the named fallback changes the architecture |
| §3a | Message container and size discipline, batching of consecutive submissions, and α sized against the FIP-0115 base-fee rule | Bounded by the 64 KiB message cap already stated in §3a |
| §4 | Base and correlation slash constants | Strawman 1–5% |
| §4 | Fault-proof message format and submission window | The window binds the unbonding delay |
| §3 / §4 | An inactivity leak — burning stake absent from the signer bitset — which is not part of this proposal | Priced, with its trigger, in *Security Considerations* |
| §6 | `stake_activation` | Partly closed: proposed at the 80M FIL security level, with the modeled peak as an upper bound |
| §6 | `N_min`, `max_share`, and their measurement over the operator set | Strawmen; the constants that decide what "dispersed" means, set from analysis of who actually controls signing weight rather than address counts, which are cheap to fake |
| §6 | `stake_deactivation`, `W`, `W′` | Strawmen 50M FIL, three months, twelve months, with the trade behind them sized in *Appendix B* |
| *Incentive Considerations* | Whether the posted rate is tied to the on-chain pledge yield, or depth pricing alone is accepted | Decides how the non-renewal channel is answered when the pledge yield falls |

Around activation, what is open is values rather than mechanism: the hysteresis rule (§6),
the authority mark and its readers (*Appendix B*), and the liveness answer (*Security
Considerations*) are specified, and only their constants appear above.

The funding open items belong to the funding decision, chief among them which source is adopted.

## Appendix B — the activation state

**The three states.** The network is always in one of three, and the chain is produced and
ordered identically in all of them.

| State | What is true | What a client reports as final | Reward |
|---|---|---|---|
| **Binding** | Latch open, quorum live | The F3 certificate head, within tens of seconds | Paid per certificate |
| **Advisory** | Latch closed — before activation, or after a deactivation — quorum live | The 900-epoch Expected-Consensus rule, ~7.5 hours | Paid per certificate: §3 gates on signatures, not on bindingness |
| **Halted** | No quorum | The 900-epoch Expected-Consensus rule, ~7.5 hours | Nothing; §3's payout stops automatically |

The floor under all three is today's network, whose Expected-Consensus fallback tolerates an
adversary near ~20% of power (*Security Considerations*). That is the contract an integrator can
build against, and an integrator reads the certificate authority mark below to tell which of the
three states the network is in.

**The deactivation band, sized.** Setting §6's deactivation floor trades against the decay in
the veto price *Security Considerations* carries: a floor at 60M FIL would hold the veto above
about 20M FIL and end binding finality around year 17, while the 50M strawman keeps a wider band
and a lower guaranteed price. The constants are a safety catch rather than a live control:
50M FIL sits about 30M below the depth the companion analysis's modeled scenarios hold at the
5.5% required-return check. In the model an injected exit of 60% of a full pool refills within seven to nine
months to the depth the rate then clears, dipping below 50M for one month at a 5% required
return and three at 5.5%. So a three-month `W′` is on the edge of firing during a recovery the
schedule completes by itself, and twelve months is not. The refill does not restore the pool's
former margin above the threshold. *[needs precise spec]* whether `W` and `W′` count epochs or
F3 instances (instances are the natural unit, since the distribution statistics are computed
per committee), and whether the staking actor keeps publishing the distribution statistics
after activation as a monitored signal with no automatic consequence — it should.

**The `ActivationRecord`.** The staking actor keeps an append-only ordered list of
`(opened_at_epoch, opened_at_instance, closed_at_epoch | null, closed_at_instance | null)`
entries plus the current latch state, written by the same consensus rule every client already
evaluates at its finalize step. The binding rule over the record — an entry opened at or
before an instance and not closed by it — is normative in §6; this appendix carries the layout
and its readers.

**The same rule serves three readers.** A node tracking chain state reads the record directly,
which is also how a syncing node reconstructs whether the gate was open for a historical
instance. A verifier holding only certificates reads the latch state and the active entry out
of the committee's supplemental-data commitment. There a committee is attesting to its own
authority. The mark is therefore worth relying on only where the committee is already
authoritative, which is what activation establishes, and it does not by itself stop a
pre-activation committee claiming authority. An integrator that takes one-way activation at its word pins a single epoch, the way
a network upgrade height is published, and checks nothing per certificate. *[needs precise
spec]* the commitment's encoding, and whether a certificate-only verifier checks the mark
against the on-chain record once at first use and thereafter trusts it.

## References

- *[Funding the finality reward: options and a recommendation](https://hannahhoward.github.io/stake-finality-fip/funding-options-and-recommendation.html)* — the companion
  analysis: the candidate funding sources compared against §5's four requirements, and the
  worked scenario the rate table and the curve in §3 are drawn at.
- *[Analysis of storage-only attacks under Expected Consensus, QAP Fast Finality, and Stake
  Finality](https://hannahhoward.github.io/stake-finality-fip/storage-only-attacks.html)* — what storage power alone can do under each of the three regimes,
  and what each share of it costs.

## Copyright

Copyright and related rights waived via
[CC0](https://creativecommons.org/publicdomain/zero/1.0/).
