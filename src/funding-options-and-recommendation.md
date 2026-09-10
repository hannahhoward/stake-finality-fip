# Funding the finality reward: options and a recommendation

*[Stake-Weighted F3 Finality](https://github.com/filecoin-project/FIPs/discussions/1288)* pays staked FIL a reward for signing finality
certificates, and it leaves open where that money comes from. §5 of *Stake-Weighted F3 Finality*
states the requirement and the four things the mechanism reads from whatever supplies it.

The pool needs about 100M FIL of income over thirty years. Two sources could carry it: a one-time
transfer from the mining reserve, released on a fixed annual schedule, and a finality stream
inside the FIP-0118 (Solstice) block-reward split. They trade against each other at close to one
for one, across an option set running from a 48M FIL envelope with the deepest block-reward
rebalance to 91M FIL with FIP-0118 left exactly as passed.

**The recommendation is the reserve alone at the 91M FIL envelope, with FIP-0118 untouched.**
The reserve was set aside at genesis for new miner types and has never been spent, and paying
finality from it leaves the block reward alone. Storage providers are already absorbing
Solstice's ramp from 95% down to 50%. Cutting deeper is the direction that pushes marginal
sectors to terminate, which weakens the same consensus security the FIP exists to protect. And a reward that touches no one's share creates no contest between storage providers
and stakers over the split.

The reserve route is the largest total consensus spend in the set, running +38.8M FIL above
today's committed issuance curve over the first decade where the cheapest rebalance runs
+10.9M. The rebalance rows therefore stay on the table, and the mix is a community decision.

The FIL minted per epoch does not change under any option here. FIL entering circulation falls
every year under every option. Whatever the pool has not paid at the year-30 sunset is burned.

Numbers marked *[re-check]* are snapshots as of the dates given and have not been re-verified
against the chain; items marked *[needs precise spec]* are deliberately left open. The model runs
behind the figures are unpublished and available on request.

---

## 1. The requirement: about 100M FIL

**The pool needs roughly 100M FIL of income over the program's life**, 90M to 97M across the
option set below. Holding 80M FIL staked, the security level the activation gate reads
(*[Stake-Weighted F3 Finality](https://github.com/filecoin-project/FIPs/discussions/1288)* §6, and the level at which a third of the pool is a
meaningful forfeit), for twelve years costs roughly that at the 3–5% return holders of comparable
assets accept today. Less money buys a shallower pool held for fewer years. The figure is a
strawman calibrated to those comparables *[re-check: the comparable rates and the required return
they imply]*, and it is the constraint every funding design has to meet.

The 100M figure comes from the payment path. Holding 80M FIL just above the 4.0% rate at which
stakers start leaving costs about 4.0M FIL a year, and the design must ride there for nine
in-window years inside the first decade. Add the year-1 payout of about 2.1M and the cost of also
surviving a 5.5% required return, and the first decade of payments is **39M FIL**. It is the same
bill in every design examined, within a third of a million FIL: across six funding lanes the
fitted first-decade payout is 39.2M FIL with a residual standard deviation of 0.29M. The second
and third decades add the rest, and what the pool does not spend by the year-30 sunset burns.

**What the offer has to beat, and what it has to stay inside.** Custodial FIL products clear at
0.3–2%, and ~10% is where credit-risk-bearing FIL products have attracted supply. The ceiling is
the modal gross staking yield among current proof-of-stake networks, 4.7–5.5%, against issuance
across large networks that has converged to 1–4% of supply a year and is still falling
*[re-check]*. Staking yields refreshed 2026-08-11 run from SUI at 1.44% and DOT
at about 2.9% through NEAR 4.72%, TIA 5.18%, SEI 6.37%, APT 7.0% and INJ 7.15%, and staked share
across a comparable set runs 23–69% with a median near 45%, so a design recruiting 80M FIL, under a
tenth of Filecoin's circulating supply, is asking for less participation than is routine elsewhere.

The posted rate opens under a 12% ceiling and settles at 4.0–5.0% while staked FIL holds the
security level (*[Stake-Weighted F3 Finality](https://github.com/filecoin-project/FIPs/discussions/1288)* §3), inside that band at both ends.
Nothing in the design fixes the rate; the market sets it, and the model treats 5% as the central
case with 5.5% and 6% as the stress rows.

**The bet is the return the market requires.** Every option holds 80M FIL staked for twelve
contiguous years from year 2 at a 5% required return, and seven of those years at 5.5%. What
must be true is that the marginal staker requires no more than 5.5%, because above it the band
falls to zero in a single step rather than tapering (§4 below). Being wrong costs security
rather than money, and the year-3 review described in §2 below settles the bet against the
chain: a pool short of the activation threshold and not rising over the trailing year is the
falsifying result.

## 2. What any funding design has to build

Whatever the source, the same machinery has to ship, and the constants are the only part that
depends on the row adopted.

**A reward-pool actor with two ledgers.** A built-in actor with no controller, keeping an
**unreleased** and a **released** ledger. Only the released ledger would be spendable, and only the
released ledger is what the posted rate reads (*[Stake-Weighted F3 Finality](https://github.com/filecoin-project/FIPs/discussions/1288)* §3).

**A fixed release table.** Part of the transfer would be released at the upgrade epoch and the rest
by a fixed annual table, shaped like the year-over-year decline of the scheduled simple-minting
curve. That is a closed form with no chain-state input, so the table would ship as a constant
vector and the funding path would read no chain state. The table would have to spend the whole
transfer across the thirty years; its values are strawman *[needs precise spec]*, and the
illustrative table below does not spend the whole transfer. Releasing the whole transfer at the
upgrade epoch would post a launch rate of **19.9%** instead of 12.0% in the worked scenario, and
staked FIL would hold the 80M FIL security level for 11 years instead of 13. The annual table is
what holds the opening rate down.

**A ceiling written into the design.** The adopted envelope would be a hard ceiling on the total
committed from outside the simple-minting schedule, counting against it both the transfer and the
share of stream income that derives from baseline minting (a fraction of a million FIL over 30
years at any of the amendment pairs in §3 below). The rest of the stream, 28M to 44M FIL over thirty
years depending on the pair, is simple-minting money the protocol was going to mint in any case and
sits outside the ceiling by construction. What bounds it is the year-30 sunset and the fact that
the stream is a decaying share of a falling emission curve: it delivers less every year without
anyone deciding to cut it.

**Two conditions on supply accounting.** Reserve FIL was minted at genesis and counts toward total
supply already; it sits outside circulating supply until disbursed. Today's clients compute reserve
disbursement as the drop in the reserve's balance, which would count the whole transfer as
circulating on the day it moves. The calculation would have to be amended to subtract the reward
pool's unpaid balance, treating it as still reserved, network-version-gated in the shape FIP-0100
used for its reserve constants and identical in every client. Two conditions come with it:

- **The netting would be defined on the pool actor's whole unpaid balance**, not on a
  reserve-sourced sub-ledger. Where the stream is exercised it credits block reward into the pool, and block reward
  already counts as mined the moment it is minted, so a netting term that covered only the
  reserve-sourced part would step circulating supply up at every stream credit and count the same
  FIL twice.
- **Both new singletons, the reward pool and the staking actor, would be added to the
  circulating-supply traversal.** The non-consensus RPC path walks the state tree against a hard-coded list of
  singletons and errors on an actor it does not recognize, so without the addition
  `StateCirculatingSupply` fails outright. This has no consensus effect; it is the number external
  dashboards read.

Under the amendment neither the transfer nor an annual release steps circulating supply on the day
it happens: each is a move between balances that both sit outside circulation. Total supply and the
2B cap are unchanged, circulating supply rises exactly as rewards are paid, and the sunset burn
nets to zero, because burned FIL never circulated. Without the netting the transfer would count as
circulating on the day it moves, which is a consensus-visible error: `FilCirculating` sets the
initial-pledge lock target, so a mis-stated figure moves per-sector pledge.

**Two reviews and a sunset.** A **year-3 non-activation review**, triggered if the activation
conditions (*[Stake-Weighted F3 Finality](https://github.com/filecoin-project/FIPs/discussions/1288)* §6) have never been met, and annually
thereafter until they are. The failure it catches is a pool that recruits stake, pays for advisory
certificates, and never reaches the level at which any of them binds. Year 3 rather than year 2:
the pool crosses the threshold in year 2 at a 5% required return but not until years 3 to 6 in the
stress cases the design is sized to survive, so an earlier trigger would fire on uptake that is
slow rather than absent. It would report staked FIL at the review epoch, its change over the
trailing twelve months, and cumulative reward paid to date, which separates a pool that has
exhausted the capital available to it, and goes flat for years, from one held back by a high
required return, which still climbs. *[needs precise spec]* the measurement window, and the
trajectory test stated as a percentage change rather than a level.

Three outcomes would be available to it: continue unchanged; revise the constants by FIP, which is
a network upgrade rather than a knob, and where lowering the activation threshold is a security
decision to be re-priced against settlement volume; or wind down honoring accrued rewards, removing
any stream entry, stopping new accrual at a published epoch, keeping the claim path open until
accrued rewards are claimed or expire, and burning the remaining pool early. *[needs precise spec]*
the accrual-stop epoch, the claim-expiry rule, and whether the early burn is authorized in advance
or needs its own FIP.

A **funding review at year 25**, triggering earlier if the released pool falls below a set fraction
of its launch level *[needs precise spec]*. It takes up the successor question, including whether
fees can carry the reward, at whatever size the network has grown to by then. Its default is that
payment stops, so a review that reaches no conclusion ends the program on schedule.

A **year-30 sunset**. Whatever the pool holds thirty years after the upgrade is burned, released or
not, and any stream entry expires on the same date. *[needs precise spec]* the sunset epoch, the
review-trigger constants, and where the review obligation is recorded.

**The constants would be protocol constants.** The transfer size, the release table, any stream
weight record, the schedule constants and the sunset date would change only through a FIP and a
network upgrade, because a runtime-tunable rate would put finality economics on a governance
surface an attacker could reach. The exposure that comes with that is that the constants are chosen
before the market has revealed its required return, and a schedule that brackets it wrongly keeps
paying and accumulating without ever binding finality until a FIP corrects it, which is what the
year-3 review exists to put in front of the network. Burning the remainder at the sunset converts
an idle overhang into a supply commitment and gives that review a hard backstop.

**Production precedent for the pool's shape.** Cardano (0.3% of remaining reserves per epoch),
Polkadot (13.14% per year of the gap to its 2.1B cap, since March 2026), and Avalanche (minting
from the gap to its 720M cap) all express a capped reward budget as a fraction of what remains.
Each pays leader-election stakers from a reserve designed to last the network's lifetime; the
design here borrows the budget shape, adds depth pricing (which none of the three has), and
differs in paying only finality signers from a pool that burns at a sunset.

### The release table, illustrated

The release schedule for the worked scenario, in M FIL, in each year after the 30M released at the
upgrade epoch. Each entry is the year-over-year decline of the scheduled simple-minting curve
evaluated on the scenario's 49.789M FIL transfer *[needs precise spec: the final table for the
adopted envelope, the epoch within each year at which the release lands, and the fixed-point
representation]*.

| Year | 1 | 2 | 3 | 4 | 5 | 6 | 7 | 8 | 9 | 10 |
|---|---|---|---|---|---|---|---|---|---|---|
| Release | 2.0326 | 1.8985 | 1.6476 | 1.4678 | 1.2801 | 1.1956 | 1.0376 | 0.9244 | 0.8061 | 0.7529 |

| Year | 11 | 12 | 13 | 14 | 15 | 16 | 17 | 18 | 19 | 20 |
|---|---|---|---|---|---|---|---|---|---|---|
| Release | 0.6534 | 0.5821 | 0.5077 | 0.4742 | 0.4115 | 0.3666 | 0.3197 | 0.2986 | 0.2591 | 0.2309 |

| Year | 21 | 22 | 23 | 24 | 25 | 26 | 27 | 28 | 29 | 30 |
|---|---|---|---|---|---|---|---|---|---|---|
| Release | 0.2013 | 0.1881 | 0.1632 | 0.1454 | 0.1268 | 0.1184 | 0.1028 | 0.0916 | 0.0799 | 0.0746 |

The entries sum to 18.4391M FIL as tabulated (18.43919M exact), so on top of the 30M released at
the upgrade epoch 48.4392M of the transfer is released over the 30 years and 1.3501M stays
unreleased; that remainder and whatever the released ledger still holds are burned at the sunset.
An adopted table would normalize the same shape to full spend, which is worth about one more year
at or above the security level.

### The worked scenario

One configuration produces every illustration here and the rate table and curve in
*[Stake-Weighted F3 Finality](https://github.com/filecoin-project/FIPs/discussions/1288)* §3: a 49.789M FIL transfer with 30M released at the
upgrade epoch, a 25% service cap, a 35% storage-provider floor, and the constants `spend_rate =
0.22` and `S_offset = 55M`. It holds 80M FIL staked for 13 contiguous years at a 5% required
return, pays 83.6M FIL to stakers over 30 years (against 76M to 79M on the option rows), and runs
12.6M FIL above today's committed curve over the first decade. It is a 50M-envelope configuration
of the reference lane whose cheapest configuration reads 44.0M at +8.54M (§4 below). A storage-provider floor above the service cap would guarantee the
storage side more than the service side may take, and no package in §4 below places the floor
above the cap, so the pairing is an illustration. That same split is what puts its first-decade
figure below every option in that table but the cheapest. *[How the finality reward is paid
for](https://hannahhoward.github.io/stake-finality-fip/funding-explainer.html)* works it through end to end.

## 3. Where the money could come from

### The mining reserve

300M FIL was set aside at genesis for "future miner types that have yet to be determined," with
distribution left to the community through the FIP process. It holds ~283M FIL today *[re-check:
balance 282.93M]*, and no FIP has ever drawn on it.

A finality committee paid per signed certificate is a new miner type performing a new proven
service, and FIP-0085 already defines the governance path: the reserve is a keyless account, so a
disbursement requires a FIP and a network upgrade. Across the option set the envelope runs from 48M
to 91M FIL against that balance, so between **17% and 32% of the reserve** would move and the rest
stays where it is. It would happen once, since an annual pull would need either a network upgrade
every year or a standing privileged debit path out of the reserve, which is the thing FIP-0085
closed.

### A finality stream inside the FIP-0118 split

FIP-0118 has the reward actor divide each block reward across an ordered list of streams. A
finality stream would be one more entry, last in the list, taking whatever remains after the
storage-provider consensus share `w1` and the service share `w2`, exactly `1 − w1 − w2` every
epoch, leaving 0 FIL of the residue today's rules burn across all 30 years. It funds finality out
of FIL the protocol was already going to mint, and every option that uses it draws less from the
reserve than the reserve-only option does.

**Service is computed as if the stream were absent.** `w2` is evaluated first, on its own schedule
and its own quarterly volume gates, and the stream takes what is left, so service receives the same
FIL with the stream as without it and what a cap change itself costs service is the whole of the
cost to service. *[needs precise spec]* whether FIP-0118's ordered evaluation alone guarantees
this, or whether subordination has to be stated as a rule on the list.

**At FIP-0118's published values the stream has no room to work in at all**, so exercising it takes
two amendments. The service cap moves from 50% to 35% or 40% (`W2_CAP` from 0.50 to 0.35 or 0.40).
Because service is computed first, its cap sets the stream's ceiling: at the storage-provider
floor, every point of service entitlement above the cap is a point the stream cannot reach.
Service's quarterly volume gates below the cap would be unchanged. The storage-provider consensus
share then continues below Solstice's 50% floor, to 40%, 35%, 30% or 25%, leaving the published
nine-quarter ramp from 95% down to 50% untouched.

The stream earns nothing until the storage-provider share falls below one minus the service cap:
the residue first turns positive in the eighth quarter counting Solstice's activation as the first
at a 35% cap, and in the ninth at 40%. On the modeled dates the published ramp's 50% floor is
reached in March 2029.

Each FIP-0118 stream carries a `WeightRecord` of the form `{v_start, slope, t_start, floor, cap}`
evaluated as `clamp(v_start + slope × (epoch − t_start), floor, cap)`, so adding a stream is a
write to that list. The finality entry is a flat strawman record of 0.45 starting at the upgrade
epoch, and because it is last it takes `min(0.45, 1 − w1 − w2)`, a residue that reaches at most
0.40 under any amendment pair. It absorbs the whole residue, leaving 0 FIL burned, only while
FIP-0118's volume gates pass; if the gates stall, it also picks up residue today's rules would
burn.

At the published ramp's end a second record would take over for the storage-provider share,
`{v_start = 0.50, slope = −2.5pp per quarter, t_start = the end of the published ramp, floor = the
adopted floor, cap = 0.50}`, reaching 35% six quarters later, 30% eight, 25% ten. **No quarterly
step is larger than one Solstice already commits to:** the worst quarterly fall in per-EiB
consensus revenue across the whole schedule is 12.44%, which is the published ramp's own step, and
every continuation step is smaller because 2.5 points a quarter is half the ramp's 5. No quarter
breaches a 15% guard.

The dependency runs one way. Without the FIP-0118 split there is no stream, and if FIP-0118 does
not activate, or activates without the amendments, the mix falls back to the reserve-only option at
its own envelope. FIP-0118 does not depend on anything here, and the stream is a list entry
removable by the mechanism that adds it.

### Every workable design needs a transfer

**A design with no reserve transfer at all cannot reach the security level.** Roughly 54,000
zero-reserve configurations were run. At a 35% service cap with the deepest rebalance modeled, zero
of 1,440 configurations reach 80M FIL staked at a 5% required return in any year at all. Pushing to
curves nobody would ship, a spend rate at or above 0.35 with a 25% service cap, touches 80M and
holds it for at most six contiguous years, entering in years 4 to 7. No configuration anywhere
reaches twelve.

The reason is timing. The stream is a flow: it peaks at 4.00M FIL a year in year 5
and then decays about 11% a year with simple minting. Holding 80M FIL at a 5% required return costs
4.00M FIL a year, permanently. A decaying flow cannot cover a bill that does not decay, and with no
reserve the pool never accumulates a balance, so the posted rate never rises with pool depth.

### The alternatives set aside

| Alternative | Why not |
|---|---|
| **A different pair of launch constants** | Two anchors fix `spend_rate` and `S_offset` once the released pool at launch is chosen, and both bind in every option: the 12% ceiling on the launch rate (*[Stake-Weighted F3 Finality](https://github.com/filecoin-project/FIPs/discussions/1288)* §3), and the rate the schedule posts while staked FIL sits at or above the 80M FIL security level, 4.0–5.0% throughout that run at a 5% required return, checked month by month, inside the cross-chain band §1 above reports. In the worked configuration the pair puts the launch rate at exactly 12.00%, and a rate sitting exactly on the ceiling does not survive publication rounding, so the adopted pair leaves headroom below it |
| **A gated later draw alone** — draw nothing from the reserve until the network has run a year below a set issuance threshold, and nothing ever if that year never arrives | The least contested design available, and it costs six years: with no stream income the pool cannot post a deep enough rate to fill early, and staked FIL does not reach the 80M FIL security level until around year 8 — six more years on the weaker fallback tolerance *[Stake-Weighted F3 Finality](https://github.com/filecoin-project/FIPs/discussions/1288)* prices |
| **A bootstrap premium** — a separate tranche adding 3 percentage points to the offered rate for the first 36 months | The release schedule already pays early stake more than late stake: the rate opens in double digits while stake is shallow and falls to around 5% by the time 80M FIL is staked. The launch rate also leaves no room, since the constants sit against the 12% ceiling in every option, so a 3-point adder would open at 15% |
| **A smaller baseline cap** — reduce the 770M baseline cap and pay the pool from the difference | It cuts storage-provider reward from the first epoch, before the committee exists and before a single certificate is signed: baseline emission is proportional to the cap, so the baseline slice falls in proportion to the carve *[re-check: today's baseline slice runs ~3.1M FIL/yr]* |
| **A deeper cut to the block-reward split instead of a new fund** | Each cut lowers storage-provider revenue against fee obligations that do not fall with it, which makes terminating sectors to recover pledge the rational response, and that removes quality-adjusted power. The loop is not modeled here, and the modeling is owed before any deeper cut is proposed |
| **A larger reserve draw** — take more than the requirement | Beyond the ~100M requirement the money buys years the program is not sized for, at worsening returns: 85M reaches the twelve-year bar, 100M buys fourteen years, and above that each additional FIL buys less time and more sunset burn, since whatever the market never claims is destroyed at year 30. A first disbursement also sets the terms of every later one, and FIP-0118's own Design Rationale, at question Q3, treats the reserve's finiteness as the reason to draw carefully. Every FIL paid out of a draw also enters circulation, so a larger transfer sits further above today's committed curve |
| **The withheld baseline** — re-key effective network time to staked FIL and route withheld baseline emission to finality rewards | The largest available source and it needs no new fund, since the network sits far below its baseline target and most of the 770M baseline schedule is withheld. But a settled stake pool is static against an annually-doubling baseline target, so effective time crawls and the flow decays below the reward bill within roughly a decade. The baseline function is denominated in bytes with no natural conversion from staked FIL, so whatever constant is chosen sets the funding rate by fiat |
| **A fixed 10% slice of the block reward** | The simplest thing to ask for and the most expensive way to ask for it, in both constructions. Taking 10% of every stream's share cuts storage-provider revenue a further tenth on top of the published ramp and breaches the 15% quarterly pace guard at every viable ship date (16.9–20.2%); none of 12,072 configurations clears the bar, and even guard-waived it prices at +28.7M of first-decade delta on a 98.8M envelope — more reserve than the reserve-only row draws. Taking 10% of the residue instead pays nothing under FIP-0118 as passed and reaches ten points only at a 40% service cap after the ramp ends, which is a ten-point cap concession by another name: it harvests 8.2M FIL in the first decade where the 40%-cap / 25%-floor package harvests 23.5M on the same concession, and a further 7.1M of its apparent discount is burned as unused residue. Ten points buys about 6M FIL of reserve relief however it is split |
| **Release on need** — release each year only what the pool is short of its launch level | Returns one more year at or above the security level at the central 5% case and defers about 5M FIL of first-decade issuance, but only if the market clears at 5% or below; above that the pool never rises far enough above its launch level to hold the security level at all |
| **Extending `FilLocked` to subtract staked FIL** | It would improve the issuance picture further, and it is declined because `FilCirculating` is a consensus input rather than a reporting figure (§6 below) |

No option above changes minting: simple minting, baseline minting, the baseline target function,
effective network time and the 2B cap all stay exactly as they are.

## 4. What the options cost, and who pays

Each row is the cheapest configuration of its source mix on the first-decade measure, and every row
clears the same bar: twelve contiguous years at or above the 80M FIL security level at a 5%
required return, entry by year 2, seven of those years surviving a 5.5% required return, launch
rate at or below 12%, and falling issuance throughout. Two numbers are priced. **Reserve envelope**
is the ceiling committed to on total draw from the mining reserve. **First-decade delta** is the
cumulative FIL entering circulation above today's committed curve over ten years.

| Source mix | Service cap | Storage-provider floor | Reserve envelope | First decade above today's committed curve | Service entitlement given up, 10y / 30y | Storage-provider share given up, 10y / 30y |
|---|---|---|---|---|---|---|
| Reserve plus rebalance | 35% | 25% | 48M FIL | +10.9M FIL | 12.6M / 18.6M | 15.3M / 25.4M |
| Reserve plus rebalance | 35% | 30% | 55M FIL | +13.5M FIL | 12.6M / 18.6M | 13.0M / 21.1M |
| Reserve plus rebalance | 40% | 25% | 53M FIL | +15.5M FIL | 8.2M / 12.2M | 15.3M / 25.4M |
| Reserve plus rebalance | 40% | 35% | 67M FIL | +21.1M FIL | 8.2M / 12.2M | 10.2M / 16.3M |
| **Mining reserve alone, FIP-0118 unamended — recommended** | 50% | 50% | 91M FIL | +38.8M FIL | 0 | 0 |

The last two columns are what each constituency gives up against FIP-0118 as passed. The service
cap alone sets the service column, which is why the two 35% rows share a figure and the two 40%
rows share theirs; in every option that uses the stream, the storage-provider share gives up more
than service does. The reserve-only option asks nothing of either and is the largest total
consensus spend in the set.

At full precision, service-35 / floor-25 is the cheapest row that can be proposed at +10.88M on a
48.0M envelope, and the pure reserve the most expensive at +38.77M on a 90.75M envelope. Smaller
envelopes clear the same bar at a higher first-decade cost: 42M FIL at 35/25, 48M at 35/30, 47.5M
at 40/25, 58M at 40/35, and 85M FIL for the reserve-only design. No package offers a
storage-provider floor above the service cap; the reference lane that does, service 25% / floor
35%, reads 44.0M of envelope at +8.54M of first-decade delta and +28.4M over thirty years.

### The envelope ladder

Best first-decade delta at a capped envelope, in M FIL:

| Lane | Best delta at a 50M envelope | at 60M |
|---|---:|---:|
| Service 35% / floor 25% | +10.88 (cheapest anywhere, at 48M) | +10.88 |
| Service 35% / floor 30% | +14.62 | +13.54 (cheapest anywhere, at 55.4M) |
| Service 40% / floor 25% | +16.41 (at 48M) | +15.48 (cheapest anywhere, at 52.9M) |
| Service 40% / floor 35% | needs 58M | +23.11 |
| Pure mining reserve | needs 85M | needs 85M |

Service-40 / floor-35 reaches at most ten years at a 50M envelope and first reaches twelve at 58M.
The pure reserve reaches 4 / 7 / 9 / 10 / 11 years at 50 / 60 / 70 / 75 / 80M and first reaches
twelve at 85M. Capping the envelope at 50M costs delta in two lanes, and the extra cost is small:
service 35% / floor 30% pays +14.62 against +13.54, and service 40% / floor 25% pays +16.41 against
+15.48. Service 35% / floor 25% is unaffected, because its cheapest design already fits under 50M.

The cheapest-delta designs hold a large launch balance: the tranche released at launch is 53% to
81% of the envelope, which defers issuance by holding a large standing stock. Two measures
therefore rank differently, and both are carried here. Minimum first-decade delta favors designs
that hold a large launch balance; minimum envelope favors lean ones.

Each row's constants, the pair `spend_rate` and `S_offset` that set the posted rate, are strawman
values, lane-specific, and the release-table epoch and fixed-point representation remain
*[needs precise spec]* in every lane.

**The launch date is a modeling input.** Every figure here is computed on Solstice activating
December 2026 and the finality upgrade one quarter later *[re-check]*; nothing here fixes when the
work ships. Moving the launch later changes the first-decade delta by about 1.2M FIL in the lane
where it matters most and by less elsewhere, and the latest launch date modeled still holds the 80M
FIL security level for twelve contiguous years. Service-35 / floor-25 reaches +10.88M only at a
2028-04 launch, which spends the whole of its margin against a slipped ship date. At the common
2027-12 design ceiling it reads **+11.25M**.

### Reserve draw against stream income

**First-decade delta ≈ 39.2M FIL minus the stream income the pool receives through year 10.**
Fitted across all six lanes with the slope forced to −1: R² 0.999, residual standard deviation
0.29M. The identity is exact: cumulative issuance above today's committed curve is payouts minus
stream income, which holds to within 5 × 10⁻⁷ M FIL on every row, and the payout side is the same
39.2M bill everywhere.

The two knobs deliver at different rates. Through year 10 a point of service cap is worth about
**0.97M FIL** of stream income and a point of storage-provider floor about **0.55M**, a ratio near
1.8 to 1, because cap concessions arrive while the stream peaks in year 5 while floor extensions
phase in over 2029–2032. Against the envelopes above they price differently again: each point of
service cap above 25% costs about 1M FIL of extra reserve draw, and each point of storage-provider
floor about 1.4M to 1.5M. Requiring the design to survive a 5.5% required return for seven years
costs about 3.0M FIL of first-decade delta, near-identical in every lane at 2.6–3.3M.

### What storage providers see

**During the nine-quarter ramp, nothing changes under any option.** From Solstice activation to the
end of its published ramp, the storage-provider share steps from 95% down to 50% on the schedule
Solstice already commits to, and the service share is limited by the room the storage-provider
share leaves rather than by its own cap. The cap and the floor only reallocate the split between
service and the finality stream after the published ramp ends. **Storage-provider revenue over
those nine quarters is identical under every option, including the pure mining reserve.**

After the ramp, the reserve-only option leaves the split untouched. Under the options that use the
stream the floor continues to 40%, 35%, 30% or 25%, and in steady state per-EiB consensus revenue
is 80%, 70%, 60% or 50% of what Solstice as passed would pay, one figure per floor, which is 40%,
35%, 30% or 25% of the no-split level.

The only option that stops at the 35% floor ships in December 2027, and over the 30 years from
there storage providers receive **56.05M FIL** of consensus reward against **72.32M** under
Solstice as passed; a deeper floor takes more. Thirty-year totals are not comparable across ship
dates, since a later window opens further down a decaying schedule. The table tracks that option
from its December 2027 launch, per EiB of raw byte power held flat at 1.398 EiB to make the ratios
exact:

| Year | This option | Solstice as passed | No split at all |
|---|---|---|---|
| 1 | 7.67 | 7.67 | 11.60 |
| 3 | 3.41 | 4.46 | 8.91 |
| 5 | 2.46 | 3.52 | 7.04 |
| 10 | 1.37 | 1.96 | 3.93 |
| 15 | 0.77 | 1.10 | 2.20 |

M FIL per EiB per year.

Against that, storage providers are eligible for the participation reward as holders, and their
block producers earn the includer share for submitting certificates.

### The required return sets the band

Past 5.5% the band falls to zero in a single step:

| Required return | 5.00% | 5.25% | 5.50% | 5.75% | 6.00% |
|---|---|---|---|---|---|
| Band years, 35% cap / 30% floor | 12 | 10 | 7 | 0 | 0 |
| Band years, 40% cap / 35% floor | 12 | 9 | 7 | 0 | 0 |
| Peak staked, M FIL | 85–90 | 80 | 80 | 75 | 70 |

Seventy-five basis points above the modeled return, the pool still recruits 75M FIL, still pays
those stakers from the first certificate, and never crosses the activation threshold, so no
certificate ever binds. What is lost is the security objective the payment was for. Holding the
envelope fixed at 55.5M FIL and re-choosing only the two curve constants against the market's
actual required return, the same money buys 11 band years at 5.75%, 9 at 6.5% and 7 at 7.5%.
Repricing them is a FIP and a network upgrade, and in the interval the mechanism keeps paying,
keeps accumulating, and does not bind.

### The demand cases

The schedule is run at required returns of 3% and 5% on the worked scenario's configuration.
Staked FIL reaches the 80M FIL security level in year 2 in both and holds it for 13 contiguous
years at 5% and 18 at 3%; payments peak at 4.89M FIL in year 6 and fall every year after,
totalling 83.6M FIL over 30 years at 5% and 87.7M at 3%. Every funding option is sized against
the same 5% central case at a twelve-year bar, so the arrival profile is common to all of them
and only the pool balances differ. Both cases assume stake arrives at 5M FIL/month while the
offered rate clears the return stakers require, exits only when the offered rate falls below
80% of that required return, 130M FIL of reachable idle capital, and no restaking of rewards.
The 3% case is the likelier one given the acceptance band in §1 above and the largest
storage-provider lending product's 3.2% yield *[re-check]*; the 5% case is the modal gross
yield among current proof-of-stake networks, if FIL stakers anchor on cross-chain rates
instead.

*Marginal staker requires 3%:*

| Year | Staked | Posted rate | Released pool at year end |
|---|---|---|---|
| 1 | 60M | 5.70% | 29.8M |
| 2 | 120M | 3.74% | 29.8M |
| 3 | 130M | 3.65% | 30.7M |
| 5 | 130M | 3.86% | 32.4M |
| 10 | 130M | 3.36% | 28.2M |
| 20 | 75M | 2.44% | 14.4M |
| 30 | 25M | 2.44% | 8.9M |

*Marginal staker requires 5%:*

| Year | Staked | Posted rate | Paid that year (network-wide) | Released pool at year end |
|---|---|---|---|---|
| 1 | 60M | 5.70% | 2.3M | 29.8M |
| 2 | 80M | 4.89% | 3.9M | 30.0M |
| 3 | 85M | 4.96% | 4.1M | 31.6M |
| 5 | 95M | 4.99% | 4.8M | 34.0M |
| 10 | 100M | 4.32% | 4.5M | 30.4M |
| 20 | 45M | 3.99% | 1.9M | 18.1M |
| 30 | 15M | 4.09% | 0.6M | 13.0M |

**Thin demand costs almost as much as full demand.** The pool pays `spend_rate × P` whatever depth
it reaches, so a market that delivers less stake does not spend proportionally less; it pays a
higher posted rate to fewer stakers. Measured across the demand cases on the committed-curve
baseline, at the 40% cap / 25% floor row:

| Case | Peak staked | Above the curve, 10y | Above the curve, 30y | Per 1,000 FIL-years of stake, 30y |
|---|---|---|---|---|
| Required return 3% | 130M | 18.7M | 43.3M | 16.2 |
| Required return 5% (central) | 85M | 15.5M | 38.9M | 23.1 |
| Required return 6% | 70M | 13.5M | 36.6M | 26.8 |
| Reachable capital 60M | 60M | 12.3M | 38.7M | 26.7 |
| Reachable capital 40M | 40M | 7.7M | 37.6M | 33.1 |

Thirty-year spend varies by 13% across a 3.25× range in delivered stake, and the unit cost of
security roughly doubles. In the 40M row the pool never reaches the activation threshold, so every
certificate paid for over thirty years is advisory, and the bill is 97% of what full demand costs.
The first decade runs the other way, 7.7M against 15.5M, because payouts fall while the release
table does not.

Uptake is the largest open variable. Storage-provider lending products such as GLIF hold on the
order of tens of millions of FIL *[re-check: ~32M FIL TVL]* supplied voluntarily for
indirect, credit-exposed yield, and on other proof-of-stake networks native staking absorbs
30–60% of circulating supply *[re-check]*. Against circulating supply of 889.58M FIL at the
2026-07-17 capture *[re-check]*, already net of locked pledge, the ~32M FIL of revealed
preference is 3.6% and the cross-chain share would be 267M to 534M FIL. The 130M FIL of
reachable idle capital the scenarios assume is 14.6% of circulating supply, between the two
readings.

### What the network spends on consensus, in total

Counting PoRep reward plus finality payouts over thirty years on one aligned window, the network
spends **72.3M FIL** under Solstice alone and **123.3M to 148.9M** with a finality reward funded
from this set: 46.9M to 72.3M of storage-provider consensus share plus 0 to 44.1M of finality
stream, with 48M to 91M drawn from the reserve making up the balance and the sunset burning
whatever is never paid. Finality payouts land at 76M to 79M whichever mix is adopted, because the
pool's income constraint binds and the sunset burns the difference. The source mix moves who pays
that bill, and the reserve-only option carries the largest total, 148.9M. In share terms, the
rebalance rows move the steady-state split back to 65% consensus at a 35% service cap, or 60% at a
40% cap.

## 5. The recommendation

**The mining reserve alone, at the 91M FIL envelope, with FIP-0118 left as passed.**

It needs neither amendment to FIP-0118: the service cap stays at 50%, the storage-provider share stops at
Solstice's published floor, and nothing in the block-reward split moves.

The reserve was set aside at genesis for new miner types and has never been spent, and a finality
committee paid per signed certificate is a new miner type performing a new proven service. Paying
finality from it asks nothing of either constituency. The rebalance has two costs the reserve route
does not pay. It creates a constituency, storage providers against stakers, live from the upgrade
epoch whether or not a single FIL is ever staked, because the continuation of the consensus share
starts on its own schedule. And it deepens the consensus cut: consensus revenue falling against fee
obligations that do not fall with it makes terminating sectors to recover pledge the rational
response, which removes quality-adjusted power and lowers the same consensus threshold the
mechanism exists to raise. That loop is unmodeled, and the reserve-only row does not enter it.

The reserve also uniquely supplies income fixed at the upgrade epoch. It does not depend on
FIP-0118 activating, on its quarterly volume gates passing, or on the emission path a stream decays
with.

**Its price is the largest in the set on every money measure**: a 91M FIL envelope, 32% of the
reserve, +38.8M FIL above today's committed curve over the first decade where the cheapest
rebalance runs +10.9M, and the largest total consensus spend of any option, 148.9M FIL against
123.3M at the deepest rebalance. Paying it avoids a renegotiation with either constituency and a
deeper cut, and a negotiation cannot price either one.

The bar is twelve years at the security level at a 5% required return. The 91M row meets it, and
carries the same seven-year margin at 5.5% that every row does. Above the modeled return the
rebalance rows degrade more slowly, so at higher required returns they close the gap.

Approving a first disbursement sets the precedent that the reserve can be spent by FIP. The
precedent is bounded to one event, a ceiling on the envelope, a bounded payment speed, a calendared
review, and a sunset that burns the remainder. Its size, 91M FIL, is the largest single question
the recommendation puts to the community.

**The rebalance rows stay on the table.** They cost less on envelope and on first-decade delta,
they clear the same bar, and the machinery in §2 above is the same either way: only the
constants change.

## 6. What it does to supply

**Issuance falls every year under every option in the set.** Under the netted
circulating-supply definition in §2 above, FIL entering circulation is lower in every year than
in the year before it, at both modeled required returns; the float still grows every year, by
less each time. At a 40% service cap, ten of 42,198 runs that reach the security level show a
rising year, and none of them is the cheapest in its cell; no feasible pure-reserve design with
twelve held years has a rising year at all.

![FIL entering circulation each year, worked scenario](https://hannahhoward.github.io/stake-finality-fip/charts/issuance-yoy.png)

*Drawn for the worked scenario, where the path never runs more than 2.23M FIL above today's
committed curve.*

On the **total-supply ledger** the program adds nothing at all: the reserve transfer is
genesis-minted FIL, a stream re-weights FIL that was already going to be minted, and the sunset
burn reduces eventual total supply by whatever the market never claims.

The **gross ledger** charges the design with every FIL it pays, staked stock less cumulative
payouts. M FIL, at the 40% cap / 25% floor row:

| Year | 1 | 2 | 5 | 10 | 15 | 20 | 25 | 30 |
|---|---|---|---|---|---|---|---|---|
| Required return 5% | +58 | +74 | +67 | +46 | +9 | −26 | −48 | −62 |
| Required return 3% | +58 | +114 | +110 | +88 | +55 | −1 | −37 | −56 |

The compression peaks at +74M (5%) or +119M in year 3 (3%), crosses zero in year 17 or year 20, and
ends between −56M and −64M across the rebalance and reserve-only rows; the sunset burn of 13.9M to
16.4M FIL of unpaid pool moves those terminal figures to between −42M and −48M. On the rebalance
rows most of what that ledger charges was going to be minted and circulate anyway: 37.7M of the
76.5M paid over thirty years is block reward the stream re-routes.

The **committed-curve ledger** counts only what today's schedule would not have produced, and
it is the baseline every table in §4 above uses. Its permanent addition is an identity, reserve
drawn less the sunset burn, **38.9M FIL**:

| Year | 1 | 2 | 5 | 10 | 15 | 20 | 25 | 30 |
|---|---|---|---|---|---|---|---|---|
| Required return 5% | −58 | −76 | −78 | −70 | −40 | −8 | +11 | +24 |
| Required return 3% | −58 | −115 | −121 | −111 | −86 | −34 | +1 | +18 |

On that measure the float is compressed for 21 years at a 5% required return and 24 at 3%, and
the program ends with +18M to +24M FIL more in circulation than if it had never run. The source
mix changes how far above today's committed curve the path sits: **+10.9M to +38.8M FIL over
the first decade** across the option set, and +32.3M to +76.6M over the thirty. In the worked
scenario the largest single-year excess is **+2.23M FIL, in year 1**, no later year comes
within 0.10M of it, and year 4 runs 0.17M FIL *below* today's committed curve. All of these
depend on the netting condition in §2 above: without it the transfer lands as a single step of
its own size in year 1, both excesses become that number, and the comparison against today's
committed curve no longer holds.

Locked stake is reversible in a way a burn is not. If stake exits the compression unwinds, and the
rate rises as it goes, drawing replacement stake back. That response re-recruits from the reachable
pool rather than creating one, which makes it weakest in the deep-participation case the design
treats as success.

### Why staked FIL stays in circulating supply

`FilCirculating` is a consensus input rather than a reporting figure, and that is the decisive
reason for leaving staked FIL inside it against an issuance picture that would look better
without it. It reaches the actors as `rt.total_fil_circ_supply()`, and the miner actor sets the
initial-pledge lock target at 30% of it
([`monies.rs`](https://github.com/filecoin-project/builtin-actors/blob/master/actors/miner/src/monies.rs),
`LOCK_TARGET_FACTOR_NUM/DENOM = 3/10`), so subtracting staked FIL would lower per-sector pledge
and couple the finality-stake market to the storage-collateral market. On ETH, SOL, ADA and
ATOM the circulating figure is reporting only, so no other chain faces this consequence.

The reporting convention points the same way. No major aggregator subtracts staked tokens, and
staked tokens count as circulating everywhere else: Ethereum has 34.9% of supply staked against an
excluded pool of zero, and Injective reports circulating supply exactly equal to total supply,
100,000,000 INJ, with 59.47M staked.

Filecoin's own number behaves differently, which is why the change would reach the reported figure.
FIL's publicly reported circulating supply tracks the protocol's `FilCirculating`, which nets out
pledge collateral: at epoch 6,275,948 the protocol reports 903.13M FIL while CoinMarketCap and
CoinGecko both report 819.16M *[re-check: snapshot at epoch 6,275,948, 2026-08-13]*. The metric
would also become hostage to staking flows, so an unbonding wave would print a year in which
reported issuance rose.

The staking lock ships instead as what it is: a liquid-float line, circulating supply minus staked
FIL, the same treatment dashboards give ETH and SOL staking. Pledge collateral has roughly halved
in a year, from 127.79M FIL in August 2025 to 65.40M at the same snapshot.

## 7. What the numbers assume

The model has no price, no restaking or compounding, no delegation, and no distribution over
required returns. Capital arrives at a fixed 5M FIL per month up to a 130M FIL ceiling, in 5M FIL
steps, so a single-step difference between two nearby configurations is quantization rather than a
result. Raw byte power is held flat at `rbp.start` = **1.398 EiB**, which is an assumption rather
than a forecast, and no per-EiB figure derived from these runs should be quoted without replacing
it. Each required-return figure is one case run end to end; none carries a probability.

Every published row reproduces from its stated configuration. The two infeasibility results in
the envelope ladder and the zero-reserve impossibility in §3 above carry a disproof one rung
below the stated threshold.

## 8. Open decisions

| Decision | What turns on it |
|---|---|
| The funding source mix and the envelope | A five-row option set with published band-years and deltas; the recommendation here is the reserve-only row, and which row is adopted is open |
| The ~100M FIL pool-income requirement | Sized against comparables; the required return is the input it is most sensitive to |
| The release-table values for the adopted envelope | Worth about one band year; bounded by the full-spend rule |
| The depletion trigger and the review-trigger constants, and whether a termination-flow trigger joins the depth and activation triggers | Decides whether a slow decline trips a review before year 25; a termination-flow trigger is measurable, since terminations, the fee burn and staked balances are all on chain and only the join between an owner and a staker is not |
| Wind-down semantics: the accrual-stop epoch, claim expiry, and whether the early burn is authorized in advance or needs its own FIP | Sets what a non-activation review can actually execute |
| What FIP-0118 does with the weight a stream releases at the sunset or at a wind-down | Falls due on FIP-0118's side, on this schedule |
| Whether FIP-0118's ordered evaluation alone subordinates the stream to service, or the rule has to be stated on the list | Decides whether service can be affected by the stream at all |
| The sunset epoch and where the review obligation is recorded | Bounded: the date is thirty years from the upgrade |
