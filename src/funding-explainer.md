# How the finality reward is paid for

A companion to the draft FIP *[Stake-Weighted F3 Finality](https://github.com/filecoin-project/FIPs/discussions/1288)*, which specifies the
mechanism (the staking actor, the stake-weighted committee, certificate-gated rewards, slashing,
and the activation gate), and to the funding analysis *[Funding the finality reward: options and
a recommendation](https://hannahhoward.github.io/stake-finality-fip/funding-options-and-recommendation.html)*. The FIP leaves the funding source open; the analysis
explores the options and recommends one. This covers one question: where the FIL that pays
finality signers comes from, and what each option costs the people already being paid.

Constants marked *[strawman]* are chosen to make the design work; items marked
*[needs precise spec]* are unresolved.

## 1. What changes

Finality signers are paid a posted rate on their staked FIL. Paying them takes about 100M FIL
of income to a reward pool over thirty years, 90M to 97M across the options in section 2. Two
sources could supply it, neither of them new minting.

One is required: a one-time transfer from the mining reserve, capped at an envelope the
design commits to, which reaches the reward pool on a fixed schedule and enters circulation
only as stakers are actually paid. The transfer can carry the whole program on its own, and
that is what the analysis recommends.

The other is optional: a finality stream inside the Solstice block-reward split, opened to
shrink the transfer. It is paid out of block reward that FIP-0118 (Solstice) as passed would
pay to service and to storage providers. The service entitlement is Solstice's share of block
reward for network services other than storage-provider consensus; opening the stream caps it
below its published 50%, and continues the storage-provider consensus share below Solstice's
50% floor at half the pace of the ramp Solstice already publishes. What those two amendments
leave unclaimed, the residue under the two shares, is what the stream is paid.

**How much comes from each source is open.** The two trade against each other almost exactly:
opening the stream deeper means a smaller draw on the reserve, and leaving FIP-0118 untouched
means a larger one. The preferred option is the 91M FIL envelope with FIP-0118 as passed; the
deepest rebalance brings it down to 48M FIL. Section 2 gives the reasons.

Simple minting, baseline minting, the baseline target, and the 2B total-supply cap are
untouched under every option. So is falling issuance: FIL entering circulation is lower every
year than the year before, in every option and at both modeled required returns. The float
still grows each year, by less each time.

## 2. Where the money comes from

Nothing is minted. The pool needs roughly 100M FIL of income over the program's life, 90M to
97M across the options below, which is what holding 80M FIL staked for twelve years costs at
the 3–5% return holders of comparable assets accept today. Two sources could supply it. The
required one is a one-time transfer from the mining reserve. The optional one is a finality
stream inside the Solstice split, opened by amending FIP-0118, which shrinks the transfer.

The transfer is required because a stream on its own cannot work at any curve: the stream is a
flow that peaks near 4M FIL a year and then decays, while holding 80M FIL at a 5% required
return costs 4M FIL a year indefinitely. The stream also pays little in the first years after
the upgrade, which are the years the offer has to recruit stake, so the transfer carries them
under every option.

**The analysis recommends the reserve on its own.** The reserve was set aside at genesis for new
miner types and has never been spent, and finality signers are a new type doing new work.
Paying signers out of the reserve leaves the block reward alone. Storage providers are already
partway through Solstice's ramp from 95% down to 50%, and cutting deeper is the direction that
pushes marginal sectors to terminate, which removes the storage power the network's security
rests on. A reward that takes nothing from anyone's share also leaves storage providers and
stakers with no contest over the split.

That costs money. The reserve on its own is the biggest draw, and the most the network spends
on consensus in total. It runs +38.8M FIL above today's committed curve over the first decade,
where the cheapest rebalance runs +10.9M. The rebalance options stay on the table for that
reason, and which mix is adopted is a governance decision.

Every option below meets the requirement. They differ in how much reserve FIL is committed and
in how much FIL enters circulation over the first decade above today's committed curve, and
the two move together.

| Service cap | Storage-provider floor | Reserve envelope | First decade above today's committed curve |
|---|---|---|---|
| 35% | 25% | 48M FIL | +10.9M FIL |
| 35% | 30% | 55M FIL | +13.5M FIL |
| 40% | 25% | 53M FIL | +15.5M FIL |
| 40% | 35% | 67M FIL | +21.1M FIL |
| **50%, unchanged — preferred** | **50%, unchanged** | **91M FIL** | **+38.8M FIL** |

Each row holds 80M FIL staked for twelve consecutive years from year 2 at a 5% required
return. For any row, a smaller envelope works at a higher first-decade cost: the smallest that still clears
the bar is 42M FIL at a 35% cap with a 25% floor, then 48M at 35/30, 47.5M at 40/25, 58M at
40/35, and 85M for the last row. Every FIL the stream delivers in the first decade is a FIL
the reserve does not have to release, so the first-decade column falls one for one with what
the stream brings in. The envelope column moves further, 48M to 91M against +10.9M to +38.8M,
because part of every envelope sits in the pool rather than entering circulation in the first
decade.

The floor is never set above the cap. The analysis does not offer a package whose
storage-provider floor exceeds the service cap, since it would guarantee the storage side more
than the service side may take.

The last row has no stream at all. At FIP-0118's published 50% cap the residue never grows
enough to recruit stake, so there is nothing worth paying a stream and the transfer carries
the whole bill: 91M FIL, about a third of the reserve.

**One rebalance scenario is worked through end to end** in the numbers that follow, a 25%
service cap with a 35% storage-provider floor: a 49.789M FIL transfer and the rate constants
0.22 / 55M. That pairing is not one of the options above, because it puts the storage-provider
floor above the service cap, and that split is also what puts its first-decade figure below
every proposable option's but the cheapest. The preferred row looks different: a larger transfer
and no stream, at the figures the funding analysis carries. It illustrates the mechanics: how the posted rate
moves with depth and with the pool, what the release schedule does to the opening rate, and
what the quarter-by-quarter path looks like on the storage-provider side. Unless a figure says
otherwise, sections 3 to 5 are that scenario.

The **required return** is the yield below which a holder leaves their FIL unstaked; this
document models 5% as the central case and 3% as the favorable one, and nothing in the design fixes it — the market does. The **80M FIL security
level** is the pool depth at which a one-third coalition must put about 27M FIL of slashable
capital at risk, which is the security level the mechanism FIP's activation gate uses.

At a 5% required return the pool pays stakers **83.593M FIL over thirty years**. At 3% it pays
87.728M. The largest single year is 4.890M FIL, in year 6, and payments decline after that.

**The reserve ledger, in the worked scenario.** The shape is the same under every option; the
figures move with the envelope.

| Line | M FIL |
|---|---|
| Reserve envelope, the cap the worked scenario commits to | 50.000 |
| Set aside for stream income that comes from baseline minting | 0.211 |
| **One-time transfer at the upgrade epoch** | **49.789** |
| Spendable in the pool at launch | 30.000 |
| Released by the annual table, years 1–30 | 18.439 |
| **Total released over thirty years** | **48.439** |
| Never released, burns at the year-30 sunset | 1.350 |
| Pool balance left at year 30, burns at the sunset | 13.012 |
| **Total burned at the sunset** | **14.362** |

A small share of the stream's income comes from baseline minting rather than simple minting.
The design counts everything that is not simple minting against the same envelope limit, so
0.211M is set aside for it here and the transfer is 49.789M rather than a round 50M.

For context, the mining reserve holds 282.93M FIL. Across the option set the envelope is 17%
to 32% of it.

The transfer and the annual releases move FIL between two balances that both sit outside
circulating supply, so neither steps circulating supply on the day it happens. Circulating
supply rises only as the pool pays a staker, which requires the reserve-disbursement amendment
the funding analysis recommends.

Against Solstice as passed, which is a different baseline from today's committed curve, the
worked scenario puts 33.56M FIL more into circulation over the first decade and 67.39M more
over thirty years. Against the same 35% floor and 25% cap with no pool at all, the difference
is exactly the payments to stakers: 43.15M over the first decade and 83.59M over thirty years.

**What the numbers assume.** They come from a model with no price, no restaking or
compounding, no delegation, and no distribution over required returns. Each required-return
figure is one case run end to end. Capital arrives at a fixed 5M FIL per month up to a 130M
FIL ceiling, in 5M FIL steps, so a single-step difference between two nearby configurations
is noise. Unless a figure says otherwise it is the 5% required-return case, with the 3% case
shown alongside. Two conventions carry through the charts in section 4: the pool is one
balance, so any split of payments between the reserve and the stream is an attribution by
cumulative inflow rather than a traced flow; and per-EiB figures hold raw byte power flat at
1.398 EiB, which is what makes the steady-state ratios exact.

## 3. The timeline

Dates below assume Solstice activates on 2026-12-01 and finality launches one quarter later.
Everything is anchored to activation, so if activation moves, every date moves with it. The
storage-provider floor shown is the worked scenario's 35%; a deeper floor takes two more
quarters at 30% and four more at 25%, and under the reserve-only option the continuation
never starts.

| Milestone | Quarters from activation | Date | Storage-provider share |
|---|---|---|---|
| Activation | 0 | 2026-12-01 | 95% |
| Finality launch | 1 | 2027-03-01 | 90% |
| End of Solstice's published ramp | 9 | **2029-03-01** | **50%** |
| Continuation begins, at 2.5pp per quarter | 9 | 2029-03-01 | 50% |
| **Final floor** | **15** | **2030-09-01** | **35%** |

Solstice's published ramp moves the storage-provider share 45 percentage points over nine
quarters, 5 points a quarter. The continuation moves the remaining 15 points at 2.5 points a
quarter, half that pace, and takes six more quarters. There are two floor dates because there
are two floors: the published 50% floor lands 2029-03-01, two years after finality launches,
and the 35% floor lands 2030-09-01, three and a half years after launch.

Where the stream is opened, the service share is computed as if it were absent and is never
reduced by it. The stream takes only what is left. At a 35% floor the residue never exceeds
40% of block reward, so the stream is limited by the room available from the first quarter,
and no block reward goes unallocated in any of the thirty years.

**Stake reaches the 80M FIL security level at the end of year 2 after launch** under every
funding option, and stays at or above it for at least twelve consecutive years at a 5%
required return. In the worked scenario the run is thirteen years, through year 14; at a 3%
required return it is eighteen. The thirteen-year figure is checked month by month, not only
at year ends: inside the window stake never dips below 80.0M FIL and the posted rate never
leaves 4.001–4.997%. The mechanism FIP's activation threshold, which decides when finality certificates
start deciding which chain is final rather than only advising, is proposed at this level; its
final value is open.

## 4. What the model shows

Every chart in this section is the worked scenario: a 49.789M FIL transfer, a 25% service cap
and a 35% storage-provider floor. The shapes carry across the rebalance options; the levels move
with the envelope. Under the recommended option there is no stream and no continuation below 50%,
so the two block-reward charts have no counterpart there.

![Annual issuance under this design against today's committed curve](https://hannahhoward.github.io/stake-finality-fip/charts/issuance-yoy.png)

FIL entering circulation falls every year, under every option. In this scenario it never runs
more than 2.23M FIL above today's committed curve, cumulatively +12.61M FIL over the first
decade and +35.43M over thirty years. Across the option set those cumulative figures run
+10.9M to +38.8M for the first decade and +32.3M to +76.6M for the thirty years. The worked scenario
sits below all but the cheapest of them because its storage-provider floor is above its service cap.

![Block-reward shares by quarter from activation](https://hannahhoward.github.io/stake-finality-fip/charts/weights.png)

The storage-provider share steps down to 50% in March 2029 and to its 35% floor in September
2030, and the finality stream takes only the residue the other two shares leave.

![Posted rate against total staked FIL, at four pool levels](https://hannahhoward.github.io/stake-finality-fip/charts/curve-depth.png)

The posted rate falls as stake arrives, so the first 80M FIL of security is bought at a
declining price rather than at one fixed rate.

![Staked FIL and posted rate, years 1 to 30](https://hannahhoward.github.io/stake-finality-fip/charts/stake-apr-time.png)

Stake clears 80M FIL in year 2 and holds it for thirteen years while the posted rate settles
into a 4.0–5.0% band.

![Annual payments to stakers, split by funding source](https://hannahhoward.github.io/stake-finality-fip/charts/funding-stack.png)

The reserve carries the first two years; the block-reward stream carries the rest and overtakes
it in year 3.

![Consensus revenue per EiB under three schedules](https://hannahhoward.github.io/stake-finality-fip/charts/per-eib.png)

Per-EiB consensus revenue settles at 70% of Solstice's level and 35% of the no-split level at
a 35% floor. At a 30% floor it is 60% of Solstice's, at 25% it is 50%, and under the
reserve-only option it is unchanged from Solstice.

![Slashable stake against annual consensus reward](https://hannahhoward.github.io/stake-finality-fip/charts/handoff.png)

Slashable stake exceeds a full year of storage-provider consensus reward from year 1, 20.0M FIL
against 15.2M, and the gap widens every year after.

## 5. What a staker and a storage provider see

Both run the worked scenario from section 2.

### A staker

You stake 10,000 FIL in month 10 after finality launches, when 50M FIL is staked network-wide.
The pool holds close to its 30M FIL launch balance, and the posted rate at that depth is
**6.29%**. Held still for a year, that is about **629 FIL**, gross of the includer share paid to
whoever submits the certificate and of the gas you pay to claim.

The rate does not hold still, and it is the same rate for everyone. Two things move it. More
stake lowers it: at the launch pool, 80M FIL staked posts 4.89% and 100M posts 4.26%. A fuller
pool raises it: by year 6 the pool has grown to 34.15M FIL, where 80M staked posts 5.57%. In the
5% case the two effects settle against each other, and from year 2 through year 14 the posted
rate stays inside 4.0–5.0%. Staking early does not lock in a rate, and waiting does not improve
one. The opening rate sits exactly on its 12% ceiling with no headroom above it.

| Total staked | 0 | 25M | 50M | 80M | 100M | 130M | 200M |
|---|---|---|---|---|---|---|---|
| Posted rate at launch, pool 30M FIL | 12.00% | 8.25% | 6.29% | 4.89% | 4.26% | 3.57% | 2.59% |
| Posted rate in year 6, pool 34.15M FIL | 13.66% | 9.39% | 7.16% | 5.57% | 4.85% | 4.06% | 2.95% |

### A storage provider

Consensus revenue per EiB, on a flat 1.398 EiB basis and the section 3 timeline (finality launch
2027-03-01). Years 1 and 2 are identical to Solstice
because the published ramp is untouched and the continuation has not started.

| Year | Under this design, 35% floor | Solstice as passed, 50% floor | No split at all |
|---|---|---|---|
| 1 | 10.881 | 10.881 | 13.393 |
| 2 | 6.823 | 6.823 | 11.165 |
| 3 | 4.458 | 4.894 | 9.787 |
| 5 | 2.692 | 3.846 | 7.693 |
| 10 | 1.500 | 2.142 | 4.285 |

Units are M FIL per EiB per year. In steady state the ratios are exact: 35% of the no-split
level and 70% of Solstice's. Over thirty years storage providers receive 67.991M FIL of
consensus reward, against 84.197M under Solstice as passed. A 30% or 25% floor takes the
steady-state ratio to 60% or 50% of Solstice's and costs storage providers more over the
thirty years; the reserve-only option costs them nothing.

The quarterly steps are smaller than the ones already published. The sharpest quarterly drop in
per-EiB consensus revenue anywhere in the combined schedule is **12.44%**, and that is a step
the published ramp already takes. The continuation below 50% moves 2.5 points a quarter, so every
quarter it adds is a smaller step than the ones Solstice already commits to. No quarter breaches
a 15% guard.

Staking is open to storage providers on the same terms as anyone else: the posted rate is
uniform across participants, and an SP that holds idle FIL can stake it for finality without
touching sectors, pledge, or block production. Block producers additionally earn the includer
share for submitting certificates on chain.

## 6. FAQ

**How much comes out of the reserve, and why spend it at all?**
Between 48M and 91M FIL, in one transfer at the upgrade epoch, against a capped envelope. The reserve holds 282.93M FIL, so that is 17% to 32% of it. The preferred option is the 91M
end, where the reserve pays for everything: it is FIL set aside at genesis for new miner types,
no FIP has ever drawn on it, and spending it asks nothing of anyone already being paid. It costs
about twice the draw and more than three times the first-decade issuance of the cheapest option.
FIP-0118's Design Rationale Q3 sets the reserve aside as finite and non-renewable, which is an
argument for drawing on it carefully rather than never: one transfer, under a written ceiling,
released on a fixed schedule, and a sunset that burns whatever is never paid. The options that
open a stream draw less and pay for it out of someone's share.

**If the pool pays stakers every year, how can total FIL entering circulation fall every year?**
Because what the pool pays is small next to what the consensus stream gives up over the same
period, and block reward itself is declining on the existing minting schedule. In year 1
storage-provider consensus reward is 15.212M FIL and payments to stakers are 2.300M. By year 6,
when payments peak at 4.890M, consensus reward has fallen to 3.336M. FIL entering circulation
falls in every year of the thirty, at both required-return cases and under every funding
option, including the reserve-only one at 91M FIL. The figures here are the worked scenario's,
where the early falls are the large ones: 3.65M FIL from year 1 to year 2 and 3.54M from year
2 to year 3 at a 5% required return. Even the flattest year still falls,
by 0.059M FIL at 5% and 0.125M at 3%, and there is no rising year. Year 4 runs 0.165M FIL
*below* today's committed curve.

**What happens if a huge amount of stake signs up at once? Is there a cap, and does early
signup activate finality sooner?**
There is a cap, and it binds whatever arrives: in any year the pool can pay out at most a
fixed fraction of what it currently holds, 22% in the worked scenario, which is **6.60M FIL
in year 1** on the 30M FIL released at launch. A rush cannot overshoot the pool, because the
payment is a rate on the pool and a larger stake
lowers the rate one-for-one. At a 5% required return, stake above 77M FIL posts under 5% at
the launch pool, so the next FIL in stops arriving, and stake above 110M FIL posts under 4%,
so stake starts leaving. The modeled peak is 100M FIL. Early signup does bring activation
forward, because the mechanism FIP's activation gate reads stake depth and distribution rather than a
calendar; at the modeled 5M FIL per month it takes sixteen months of uninterrupted inflow to
reach 80M FIL.

**When does the consensus reward reach its floor?**
Under the preferred option the storage-provider share stops at Solstice's published 50% floor
and the continuation never starts. Where a stream is opened, the floor lands six quarters
after the end of Solstice's published ramp at 35%, which on the modeled dates is 2030-09-01,
three and a half years after finality launch. A 30% floor takes eight quarters and a 25% floor
ten. The 50% waypoint is 2029-03-01 under every option, and all of these dates move with
activation.

**Doesn't the falling-issuance claim depend on how circulating supply is defined?**
Yes. The netting amendment is the accounting change the analysis
recommends: FIL sitting in the pool is not counted as circulating until it is paid out. Under it
the design runs +10.9M to +38.8M FIL above today's committed curve over the first decade
depending on the option, +12.61M in the worked scenario. Without the amendment, the whole
transfer lands as a single step in the year it happens, and both the decade and the
thirty-year figure become the size of that transfer. Issuance still falls every year in that
world, because the step lands in year 1. What breaks without the amendment is the comparison
against today's committed curve; the shape of the curve is the same either way. Staked FIL is
a separate question and the answer there is no: staked FIL keeps counting as circulating, as
it does on every other chain, because `FilCirculating` is a consensus input that sets the
per-sector pledge lock target.

**Why would the service cap move at all, when FIP-0118 says 50%?**
Only if the stream is opened. It is paid last: service is computed as if the stream were
absent and is never reduced by it, so service receives the same FIL over thirty years with the
stream as without it. The stream lives on the residue, and the size of that residue is what decides
whether the reward can recruit stake at all. At the published 50% cap the residue never grows
enough and stake never reaches the 80M FIL security level at a 5% required return, which is
why the preferred option leaves the cap alone and the reserve pays for everything. Where the
cap lands if it does move, 35% or 40%, sets how much of the bill the stream carries and
therefore how much smaller the reserve draw can be.

**What if participation is weaker than modeled?**
Each row below changes one assumption in the worked scenario and holds everything else,
including the size of the transfer. The count is contiguous years at or above 80M FIL. The
options in section 2 respond to a higher service cap by drawing more from the reserve
instead, which is what keeps every one of them at twelve years or more.

| Stress | Contiguous years | Entry year | Peak stake | Issuance still falls every year |
|---|---|---|---|---|
| Central case | 13 | 2 | 100M | yes |
| Service entitlement 30% | 11 | 3 | 90M | yes |
| Service entitlement 35% | 9 | 4 | 85M | yes |
| Service entitlement 50% | 0 | — | 75M | yes |
| Required return 5.5% | 10 | 4 | 90M | yes |
| Required return 6% | 7 | 6 | 80M | yes |
| Gas burn 1M / 3M / 6M FIL per year | 13 / 13 / 13 | 2 | 100M | yes |
| Finality launch slips to 2028-04-01 | 12 | 2 | 100M | yes |

Gas burn does not matter at all, because the release table is a fixed vector and nothing in the
funding path reads chain state. A slipped launch costs one year. At a fixed transfer the
service cap decides everything: at 50% the pool never reaches the security level, which is why
the preferred option is sized around a larger transfer, the 91M FIL envelope.

## 7. Formula appendix

**The posted rate.**

```
r(S, P) = spend_rate × P / (S + S_offset)
```

with the worked scenario's `spend_rate = 0.22` per year and `S_offset = 55M FIL`
*[strawman]*. `S` is total staked FIL and `P` is the released, spendable balance of the
reward pool. Each year the pool offers a fixed fraction of what it currently holds, divided
across the FIL staked plus a constant that keeps the rate finite when almost nothing is
staked. More stake lowers the rate;
paying rewards lowers the pool and lowers the rate again. At launch, with 30M FIL released and
nothing staked, this reads 0.22 × 30 / 55 = 12.00% exactly, so the opening rate sits on its
ceiling with no headroom.

The two constants move together and they move with the funding envelope, so each option in
section 2 carries its own pair, chosen with the envelope. Two things hold in all of them: the
opening rate sits at or just under a 12% ceiling, and the posted rate stays in a 4–5% band
while stake is at or above the security level. The worked scenario's pair puts the opening
rate exactly on the ceiling; the adopted pair leaves headroom below it, so publication
rounding cannot breach it.

**The stream weight.**

```
service share  = min(cap, service entitlement)       # computed as if the stream were absent
finality share = 1 − storage-provider share − service share
```

with a flat 0.45 record for the finality stream *[strawman]*, which the residue never reaches
at any cap and floor in the option set. Storage providers are paid their scheduled share
first, service is paid its entitlement up to the cap and is never cut to make room, and the
finality stream takes what remains. Because the remainder tops out at 40%, the
stream's own record never binds and no block reward goes unallocated.

**The release rule.**

```
release(y) = scheduled simple minting in year y−1 − scheduled simple minting in year y
```

with 30M FIL released to the pool at launch in the worked scenario and the table summing to
18.4391M FIL as tabulated (18.43919M exact) over years 1–30 *[needs precise spec: the final
table for the adopted envelope, the epoch within each year at which the release lands, and
the fixed-point representation]*. Each year the pool receives the amount by
which the existing minting schedule shrinks that year. Those amounts are known in advance and
depend on nothing that happens on chain, so the schedule ships as a constant vector in the
actor and no gas-burn or network-growth path can change it. The rule is that the adopted table
releases the whole transfer across the thirty years, which is worth about one more year at or
above the security level than a table that strands part of it. The worked scenario's table is
drawn before that rule and leaves 1.350M unreleased; whatever is left in the pool at the year-30
sunset burns.

A release-on-need variant, which releases each year only what the pool is short of its launch
level, was evaluated and set aside. It returns one more year at or above the security level at
the central 5% case and defers about 5M FIL of first-decade issuance, but the extra year
exists only if the market clears at 5% or below: above that the pool never rises far enough
above its launch level to hold the 80M FIL security level at all. The full comparison is in
the funding model's release-on-need runs (unpublished, available on request).

## Public records referenced

FIP-0118 (Solstice), Design Rationale Q3, "Why not the f090 mining reserve?"; FIPs discussions
[#1249](https://github.com/filecoin-project/FIPs/discussions/1249),
[#887](https://github.com/filecoin-project/FIPs/discussions/887), and
[#1030](https://github.com/filecoin-project/FIPs/discussions/1030).
