# Analysis of storage-only attacks under Expected Consensus, QAP Fast Finality, and Stake Finality

*Supporting analysis for the draft FIP [Stake-Weighted F3 Finality](https://github.com/filecoin-project/FIPs/discussions/XXXX).*

With storage power alone (no staked FIL), an attacker faces different odds under the three regimes this document prices: Expected Consensus as the network runs today, F3 fast finality weighted by quality-adjusted power (QAP), and F3 weighted by staked FIL. The most damaging attack, a reorg that reverses hours of settled history and reaches exchange deposits, costs 20% of QAP today and 33% once F3 is weighted by QAP, and storage power alone cannot run it at all once F3 is weighted by stake. Two cheaper attacks survive in every regime: reorgs of one to a few epochs at 20% of QAP, which reach integrations that settle within a few epochs (Filecoin Pay among them) but no exchange; and full forward control, meaning censoring transactions, setting throughput, or taking all block rewards, at 51% of QAP. The price of each share falls over the program window, in cash and in the number of owners who must collude, and in FIL too unless QAP settles above about 2.8 EiB.

**Scope.** Attacks that use storage power alone, under three regimes: Expected Consensus with no finality gadget (the network today), F3 live and weighted by QAP (FIP-0086 as designed), and F3 live and weighted by staked FIL (the draft FIP). Attacks that need stake are out of scope. 20% and 51% are shares of QAP; the 33% under QAP-weighted F3 is a third of the committee, which is the same third of QAP. Prices assume the attacker acquires existing miners, which transfer with their sectors and pledge at no fee or delay; building the same power new costs tens to hundreds of millions of dollars. Chain figures are as of 2026-08-23 (Filecoin Data Portal).

## 1. The attacks

Share of QAP needed, by the cheapest route:

| Attack | EC, no gadget | F3 on QAP | F3 on stake |
|---|---|---|---|
| Reorg of one to six epochs | 20% | 20% | 20% |
| Reorg of up to 900 epochs (7.5 hours) | 20% | 33% | n/a |
| Forward control: censor, set throughput, take all block rewards | 20% | 33% | 51% |

Two of the three attacks are available in every regime; only the deep reorg is closed, and only by stake weighting. The short reorg costs 20% of QAP throughout. Forward control is always available to a large enough majority, at 20% under Expected Consensus, 33% once the committee can be stalled, and 51% once it cannot, because block production stays weighted by storage power in every regime and whoever wins that weight race sets what the chain does next.

The two F3 columns differ because of one mechanism, the stall. Under QAP weighting a third of storage power is a third of the committee, and a third of the committee can stop signing at no penalty. Certificates stop, the network is back on the 900-epoch rule it runs today, and the same power then runs the 20% attack, which reaches both the 7.5-hour reorg and forward control. Under stake weighting the committee is a different set, no amount of storage power can stall it, and forward control has to be won on weight at 51%.

*The 900-epoch window.* Without a certificate, EC rejects forks older than 900 epochs and nothing shallower is final. Wang, Azouvi and Vukolić (AFT 2023) show that an adversary near 20% of power, equivocating in every epoch an honest block is mined and winning every weight tie, splits honest miners across forks and replaces them with a private chain of its own blocks. That reverses anything settled inside 7.5 hours. Under QAP-weighted F3 an attacker above a third who can also keep honest committee members' views split can sign two certificates for one instance; FIP-0086 has no slashing for it.

*The short reorg.* With a live committee, lotus proposes four epochs behind the head and the certificate lands about six behind it. In our simulation of go-f3's consensus code against an EC model with lotus's weight function, a 20% adversary running the equivocation attack never reorged deeper than 6 epochs under lotus's fork choice (11 in a stress variant that gives it every weight tie) and never replaced a certified tipset over 3,000-epoch runs. A simpler weight-race model prices the attempt, which is a private fork from the current head, costs only gas, and can be repeated every epoch: at 20%, a one-epoch reorg lands 6.6% of the time, three epochs 1.6%, six epochs 0.005% with a gadget live and 0.21% without.

*Forward control.* 51% wins the weight race outright in every regime, with no equivocation; under a live committee it must win inside each six-epoch window, so a comfortable majority is needed. The cheaper routes in the table are the equivocation attack with no gadget, where at 20% the canonical chain is attacker-only for as long as the split holds, and the stall under QAP weighting, which puts the network back in that regime at 33%. Under a live committee of either weighting the 20% attack is partial: 6% to 16% of certified tipsets carry attacker blocks only, and the certified chain stays 78% to 84% honest.

The protocol penalty is the same in every cell: a consensus-fault slash of 4 FIL, on provable double-fork mining only. Spread over many identities it is 10⁻⁵ of collateral per thousand epochs (Wang et al. §6.2).

## 2. What the shares cost over the next twelve years

- **Pledge** is the collateral standing on the acquired power. It is returned at sector expiry, or at 91.5% or more on voluntary termination. It is capital tied up rather than spent.
- **Cash at risk** is what the attacker consumes: the 8.5% termination fee, plus about 1.2% for 90 days of financing at 5%, together 9.7% of the pledge figure. A miner bought below its pledge and left to expire returns more FIL than it cost, so at a distressed price the cash figure is negative.
- **Owners** is how many distinct owners must sell or collude to reach the share, computed from the block-reward shares the public L1-health dashboard publishes; the power distribution may differ.

| Millions of FIL unless stated | 20% of QAP | 33% of QAP | 51% of QAP |
|---|---|---|---|
| Pledge, today | 13.1 | 21.5 | 33.3 |
| Pledge, 2028 | 14.2 | 23.4 | 36.2 |
| Pledge, 2032 (trough) | 6.4 | 10.6 | 16.4 |
| Pledge, 2039, if QAP settles at 4.1 EiB | 17.7 | 29.2 | 45.1 |
| Pledge, 2039, if QAP keeps falling | 2.4 to 7.9 | 3.9 to 13.1 | 6.0 to 20.2 |
| Cash at risk, today (FIL at $0.77) | 1.3 ($1.0M) | 2.1 ($1.6M) | 3.2 ($2.5M) |
| Cash at risk, 2032 | 0.6 | 1.0 | 1.6 |
| Owners, today | 4 | 10 | 18 to 42 |
| Owners, 2028, on the dashboard's 24-month trend | 2 | 4 | 11 |

The pledge line falls now because the pledge book priced before FIP-0081 is expiring. After 2032 it recovers only if QAP settles above about 2.8 EiB: above that level the FIP-0081 formula sets total network pledge near 9% of circulating supply whatever QAP is; below it the cap of 32 FIL per QAP-TiB binds and pledge tracks power down. The 4.1 EiB path is the output of our model of storage-provider economics under the funding analysis's worked scenario, held flat after 2032. Nothing yet shows QAP settling there: the last 30 days of decline annualise to −62% a year, and solving our supply and reward models together takes QAP below 2.8 EiB between 2029 and 2032, which is the last pledge row. The two cases separate soon: the 4.1 EiB path needs total network pledge to reverse and rise about 7%, to roughly 70M FIL, by the first quarter of 2027, and it is 65.3M and falling.

Cash and owner count fall on every path modelled. The protocol has no floor under the owner count: `ConsensusMinerMinMiners` counts miner actors, and one owner can hold any number of them.

## 3. What settles inside three epochs

Anything that treats a transaction as final at depth 0 to 3 is reachable by the short reorg at 20%, several times an hour. Depths below are read from source or published configuration on 2026-08-24; exchange depths are from the venues' public APIs.

| Depth | Integration |
|---|---|
| 0 | Filecoin Pay `settleRail` (settles to the head); USDFC liquidation and redemption (no confirmation wait); Pyth pull updates; lotus `StateWaitMsg` when the message has already executed (returns at depth 1 whatever confidence was requested) |
| 1 | lotus `latest`; Hyperlane's Filecoin route (`confirmations: 1`, `reorgPeriod: 0`); synapse-sdk synchronous calls (viem default); Curio's FEVM watcher (`MinEthConfidence = 1`), which piri's PDP piece-add inherits |
| 3 | Axelar deployment tooling |
| 5 to 10 | Curio's Filecoin-message watcher and lotus `MessageConfidence` (5); Boost sector-committed (6) and `PublishStorageDeals` (10) |
| 60 and up | Coinbase, Binance, Bitget, HTX, Bithumb (60); KuCoin, Coinone (120); Kraken (300, 2020 listing); withdrawals at 900 |

Filecoin Pay settles rails at the head, so a payee that acts on a settlement in the same epoch is inside the window an attacker holding four owners' miners can reach. No exchange credits inside the window. Under either live committee, 20% of storage power gets the short reorgs against the top rows of this table; the 7.5-hour window that reaches exchanges opens with 33% under QAP weighting and does not open with storage alone under stake weighting.

**Sources.** Wang, Azouvi, Vukolić, *Security Analysis of Filecoin's Expected Consensus in the Byzantine vs Honest Model*, AFT 2023; FIP-0086; lotus and go-f3 source (fork choice, weight, `HeadLookback`); builtin-actors and go-state-types (pledge, termination, slash); Filecoin Data Portal and the L1-health dashboard (pledge, QAP, owner shares); venue APIs and integration configs for §3. The simulation and the pricing model are Filecoin Foundation work, unpublished, available on request.
