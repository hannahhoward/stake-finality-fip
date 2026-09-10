# Two classes of stake: an idea for exploration

*An idea being explored alongside the draft FIP [Stake-Weighted F3 Finality](https://github.com/filecoin-project/FIPs/discussions/1288). It is not part of that proposal, and nothing in the draft depends on it. It changes what the staking actor would have to hold.*

## Summary

The draft FIP creates a pool of staked FIL that decides when a block is final, and pays for it out of a funded reward pool. Storage providers separately need FIL locked as pledge before the network will accept their storage, and a provider who does not already hold that FIL borrows it at 15% a year. The pool holds idle FIL earning about 4.9% at the level the draft targets, and providers pay triple that to the lender at scale.

A second class of stake would connect them. At-will stake works as drafted, earns the posted rate, and can leave after the unbonding delay. Locked stake is pledged behind a named provider's sectors for the life of those sectors, cannot leave when the price falls, and earns a lower pool rate plus a fee paid out of the provider's block reward. On plausible clearing assumptions the staker earns more than the posted rate, the provider's financing cost falls from a loss to a margin, and the reward pool pays 12% to 24% less per year. Locked stake displaces the idle float an attacker would have to buy, which lowers the cost of attacking the pool, and it is the most concentrated capital on the network. Whether the trade is worth making comes down to one constant, a cap on how much of finality weight locked stake may hold.

## The problem

Pledge is working capital. Before the network accepts a sector, the provider locks FIL, which is returned in full when the sector ends its term. After FIP-0118 (Solstice) every new sector counts ten times its raw size in quality-adjusted power, the measure that decides a provider's chance of winning a block, so the reward a sector earns and the pledge it must post both scale with that count. The pledge formula is unchanged by Solstice and by the finality draft.

Who holds the FIL decides whether storage is a business. A provider who funds pledge from their own balance keeps the whole block reward. On the storage-provider schedule in the [funding analysis](https://hannahhoward.github.io/stake-finality-fip/funding-options-and-recommendation.html), under its recommended option, that is a return on their own capital of 10% to 13% a year in 2028 and 7% to 9% in 2029, depending on whether pledge is priced at the stock average of 4.96 FIL per quality-adjusted TiB or the marginal 6.72. Today's legacy pledge stock earns about 26%. Many incumbents may be in that position. A provider who is not has to borrow, and the lender at scale is GLIF, which charges 15% APR and pays its depositors 3 to 4%. At 15% the interest exceeds the block reward outright in both years, and the provider is out of pocket. The FIL price does not change the sign, since reward and pledge are both denominated in FIL.

Capital earns 3 to 4% inside a lending pool, borrowers pay 15%, and the draft would open a third pool paying roughly 4.9%.

The finality pool as drafted cannot lend into that spread. Its stake is at-will. The unbonding delay is a placeholder 90 days, with no term parameter and no term premium priced anywhere. The draft's own analysis says the marginal staker stops adding below 5% and starts leaving below 4%. Pledge is locked for a sector's life, years rather than days. A lender wants the posted rate plus fault risk plus a premium for money they cannot pull back, and none of that clears at 5%.

![Posted rate against total staked FIL, worked scenario](https://hannahhoward.github.io/stake-finality-fip/charts/curve-depth.png)

## The idea

Two classes of stake in the same staking actor.

**At-will stake** is the draft unchanged: it earns the posted rate off the declining schedule, which at the draft's worked configuration is about 4.9% at the 80M FIL level the draft wants behind finality, and unbonds after the delay.

**Locked stake** is pledged behind a named provider's sectors for their life. It cannot enter the unbonding queue while those sectors live, and it cannot leave when the price falls. It earns a pool rate well below the posted one, plus a fee the provider pays out of their block reward.

The market is assumed to clear where a locked staker earns the at-will rate plus a premium for the longer lock: about 7.5% in total, of which the pool pays 2.5% and the provider pays 5%. That split is an assumption about where the price lands. Every figure that follows depends on it.

## What each party gets

**The provider** keeps the block reward minus 5%. At 5% the interest is covered and the reward still leaves a margin in both 2028 and 2029. At GLIF's 15% the same sectors run at a loss in both years. For a provider who does not hold their own pledge, financing at 5% clears against the block reward; at 15% it consumes it.

**The staker** earns 7.5% rather than 4.9%, for a lock measured in sector lifetimes rather than 90 days. Committed pledge now earns while it is committed, which raises the return a non-renewing staker gives up. Letting a sector expire and staking the returned pledge pays no early-termination fee, and the return it gives up, about 3.5% a year net at current prices, rises by whatever the locked fee pays.

**The network** pays less per staked FIL, because part of the staker's return now comes from the storage reward rather than the reward pool. At the 80M level:

| Locked stake in the 80M pool | Pool outlay, M FIL per year | Against one class at 4.9% |
|---|---|---|
| none, as drafted | 3.91 | |
| 20M at 2.5% | 3.43 | 12% less |
| 40M at 2.5% | 2.96 | 24% less |

The saving is a transfer. It moves the locked staker's return onto providers' block rewards, and it is only worth taking if providers prefer paying 5% out of the reward to paying 15% out of pocket.

## What it costs, and who it favors

It costs security, and it favors incumbents. The pool's security comes from the cost of acquiring a third of it. Locked stake lowers that cost, because every FIL of idle float it displaces is a FIL an attacker no longer has to buy. The float a storage attacker holding a third of quality-adjusted power must still purchase falls from 26.7M FIL as drafted to 20M at 20M locked and 13M at 40M locked. That attacker already holds about 21.5M FIL of pledge, so if pledge can self-lock into the pool it counts toward the same third of finality weight it is being asked to defend against.

Haircutting the weight of locked stake does not fix this, because a haircut scales the attacker's weight and the threshold together. The at-will float is what binds: under a two-class design the real security level is a third of it, and the constant that has to be set is the cap on the locked share of finality weight. Nobody has priced that cap.

Self-locking favors whoever already owns FIL, since a provider must own the stake to lend it to itself. Seven to ten owners hold a third of power today, on the two readings the draft and the storage-side analysis cite. The 80M level needs depth and dispersion. Locked pledge answers depth with the most concentrated capital on the network.

The draft calls the inputs behind depth low-confidence. At the modelled arrival rate of 5M FIL a month, reaching 80M takes sixteen months of uninterrupted inflow. Locked stake cannot be recruited by pushing the at-will rate down, because the draft's own numbers say the float leaves below 4%. The pool saving above comes from the locked class alone.

## What the spec would need

- A locked state in the staking actor that cannot enter the unbonding queue while sectors live.
- A pledge claim in the miner actor, drawn on for termination fees, with a top-up window after an equivocation slash.
- A cap on the locked share of finality weight.
- Posted-rate pricing with two inputs, which touches an accrual formula the draft has not yet specified precisely.
- Dispersion rules applied to operators regardless of where their stake came from.

The at-will rate stays on the schedule.

## Status

The locked-share cap is the one constant still unpriced. The figures in the draft and in the [funding analysis](https://hannahhoward.github.io/stake-finality-fip/funding-options-and-recommendation.html) are all computed on at-will stake. The pledge levels and the cost of acquiring storage power used here are set out in the [storage-side analysis](https://hannahhoward.github.io/stake-finality-fip/storage-only-attacks.html).

Comments belong on the [draft FIP](https://github.com/filecoin-project/FIPs/discussions/1288).
