# Entry 2: The Bonjour bots

*Covers where the tactics of the three fixed-policy opponents described by Bonjour et al. came from and how we implemented them as our own bots,
how strong they are against our bots, what tournaments with them taught us about our own bots, how resistant
variants that refuse bad offers fare, and a tournament without any player that accepts bad offers, our cleanest
measure of strength in this series. Then the story of what we built on them: a wary "Calculator", hybrids, absurd
versions of the leading bots, variants that adapt the tactic to the trade windows of our game, the elimination
tournaments that looked for the best bot, and the Elo ranking. All numbers come from our result files. Terms are
explained in the [glossary](../glossary.md); the earlier work itself is described in
[related work](../related-work.md#bonjour-haliem-et-al-hybrid-deep-rl-2021-2022).*

If you read only four sections, read [In short](#in-short) below,
[rock, paper, scissors](#4-two-tournaments-rock-paper-scissors),
[a tournament without donors](#6-a-tournament-without-donors) and [the plain result](#7-the-plain-result).

## In short

- **Where they came from:** reading the paper of Bonjour et al. and the code of the GNOME simulator it ran on, we
  saw that its high RL win rates (69.95% and 91.65%) were measured against three simple fixed-policy opponents,
  FP-A, FP-B and FP-C, in a game simpler than ours. We decided to implement bots with those tactics and see how
  strong they are in our game ([why](#1-why-implement-bots-from-earlier-tactics)).
- **What we built:** "Bonjour A", "Bonjour B" and "Bonjour C", our own rule-based bots, built on the tactics that
  the authors describe for FP-A, FP-B and FP-C. The tactics come from the paper and from the rule-based player in
  GNOME; the implementation and its adaptation to the rules of our game are ours, so every result here concerns our
  implementation in our game, not the authors' agents ([how they play](#2-how-the-bonjour-bots-play)).
  Elsewhere, "the Bonjour bots" always means these bots.
- **The plain result:** the bots we had considered strong, our own honest bots "Calculator", "Tactician" and "GPT-Style",
  turned out to be clearly weaker than the tactics of the Bonjour bots. The simple tactic of collecting sets by trade is
  much more effective in our game than the bots we designed by hand, in every kind of measurement, including a
  tournament where nobody accepts bad offers ([the plain result](#7-the-plain-result)).
- **They are stronger than we expected:** in a 3,000-game tournament with our
  [honest bots](../glossary.md#the-game-and-the-bots), "Bonjour C" wins 56.3% and "Bonjour B" 38.2%, ahead of
  "GPT-Style" (34.1%), "Tactician" (33.5%) and "Calculator" (29.7%)
  ([without the absurd bots](#without-the-absurd-bots)).
- **Why they win:** they buy the lone properties our bots are happy to sell at 1.125x the price, and then pay almost
  any price for the last property of a set. Our shared trade evaluation gives a bonus only for a complete set, so
  every single trade looks fair to our bots; the sequence of trades loses ([the mechanism](#the-mechanism)).
- **But they can be exploited:** once the absurd bots and "Hustler" join the tournament, "Bonjour C" falls to 21.8%,
  "Bonjour B" to 15.3% and "Bonjour A" to 8.5%, while our honest bots barely move: a rock-paper-scissors cycle
  ([with the absurd bots](#with-the-absurd-bots)).
- **Resistant variants close the weakness:** "Resistant Bonjour A", "Resistant Bonjour B" and "Resistant Bonjour C"
  accept an offer only if the original would and if it also passes two safeguards our own bots already use. With the
  absurd bots and "Hustler" in the pool they lose 1.4-6.9 points, the originals 9.2-23.8. "Resistant Bonjour C" has
  the highest win rate of all bots that do not send bad offers in both pools (50.7% and 43.8%)
  ([resistant variants](#5-resistant-bonjour-variants)).
- **Without donors, "Resistant Bonjour C" is still first:** in a 3,000-game tournament without donors, players that
  accept bad offers (no "Random", no non-resistant Bonjour bots), "Resistant Bonjour C" wins **43.1%** (39.7-46.4%), ahead of
  "Resistant Bonjour B" (35.2%), "Tactician" (32.3%) and "Calculator" (30.7%). The bots that send bad offers fall to
  about the level of their honest originals ("Hustler" 27.0%), so their earlier lead came from exploiting donors,
  while the lead of "Resistant Bonjour C" is strength in play ([a tournament without donors](#6-a-tournament-without-donors)).
- **Why we did not see it earlier:** we tuned our bots mostly against each other, and most of them share one trade
  evaluation and so its blind spot. As with [the absurd offers](01-from-rule-bots-to-v7.md#the-absurd-offers), a
  player from outside the pool we tuned against found the gap ([why we did not see it](#15-why-we-did-not-see-it-earlier)).
- **The lesson:** a ranking depends on the pool, and a strength claim needs a pool without donors. A test pool should
  include both set collectors and bots that send bad offers. "Strong bots" needs a qualifier: ours resist bad
  offers, but they have a gap in trading ([what it means](#16-what-it-means)).
- **Who can still be exploited:** "Random" and the non-resistant Bonjour bots accept bad offers; our bots and the resistant
  variants do not, but they still sell the first pieces of a set to a collector
  ([who can still be exploited](#who-can-still-be-exploited)).
- **Our answers, in the order we tried them:**
  - "Wary Calculator" does not sell the pieces of a set. It wins twice as often as "Calculator" against three "Bonjour C"
    bots, but it does not beat the tactic ([section 8](#8-closing-the-gap-with-a-wary-calculator)).
  - Hybrids that give "Resistant Bonjour C" the offers of "Calculator" lose about 9-10 points; its auctions make no
    difference ([sections 9 and 10](#9-bonjour-tactics-with-calculator-parts)).
  - In elimination tournaments "Resistant Bonjour B", "Resistant Bonjour C" and "Bonjour Mixed Auctions" are a tie at the
    top, and which leads depends on which other bots chase the same groups ([section 10](#10-a-first-search-for-the-best-bot)).
  - Absurd versions of the leading bots gain nothing in a pool without donors ([section 11](#11-absurd-versions-of-the-leading-bots)).
  - **The windows variants win.** "Resistant Bonjour C + Windows" and "Resistant Bonjour B + Windows" add offers in the
    trade windows of our game, when raising money for a debt and as counter-offers, which is exactly where the
    Bonjour bots are silent. They lead the Elo ranking, 74 and 62 points above "Calculator"
    ([sections 12 and 13](#12-the-windows-variants)).

## Contents

1. [Why implement bots from earlier tactics](#1-why-implement-bots-from-earlier-tactics)
2. [How the Bonjour bots play](#2-how-the-bonjour-bots-play)
3. [How strong they are](#3-how-strong-they-are)
4. [Two tournaments: rock, paper, scissors](#4-two-tournaments-rock-paper-scissors)
5. [Resistant Bonjour variants](#5-resistant-bonjour-variants)
6. [A tournament without donors](#6-a-tournament-without-donors)
7. [The plain result](#7-the-plain-result)
8. [Closing the gap with a wary Calculator](#8-closing-the-gap-with-a-wary-calculator)
9. [Bonjour tactics with Calculator parts](#9-bonjour-tactics-with-calculator-parts)
10. [A first search for the best bot](#10-a-first-search-for-the-best-bot)
11. [Absurd versions of the leading bots](#11-absurd-versions-of-the-leading-bots)
12. [The windows variants](#12-the-windows-variants)
13. [The final search and the Elo ranking](#13-the-final-search-and-the-elo-ranking)
14. [Variance parameters](#14-variance-parameters)
15. [Why we did not see it earlier](#15-why-we-did-not-see-it-earlier)
16. [What it means](#16-what-it-means)
17. [Limitations](#17-limitations)
18. [What comes next](#18-what-comes-next)

## 1. Why implement bots from earlier tactics

The work of Bonjour et al. is the most substantial neural-network work on the game
([related work](../related-work.md#bonjour-haliem-et-al-hybrid-deep-rl-2021-2022)). Reading the paper and the code
of the simulator it ran on, GNOME, we looked at where its high win rates come from: standard PPO won 69.95% of its
games and hybrid PPO 91.65%, both against three simple fixed-policy opponents, FP-A, FP-B and FP-C, in a game that is
simpler than ours in several places (no effect of doubles, no counter-offers, assets returned to the bank in
bankruptcy and others; [the GNOME simulator](../related-work.md#the-gnome-simulator)). The GNOME repository contains
a rule-based player, but neither the paper's RL players nor their weights, and it carries no license.

So we decided to implement the tactics of the opponents rather than rebuild the learning players: to write bots with
the tactics of FP-A, FP-B and FP-C from scratch, from the paper's description and from the rule-based player
`background_agent_v3_1`, let them play our game, and see how strong they are there and how our bots fare
against them. These are our bots: the implementation and the adaptation to the rules of our game are ours, and all
results in this entry concern our implementation in our game, not the authors' agents.

This also addresses in part a limitation we listed in [entry 1](01-from-rule-bots-to-v7.md#13-limitations): we
measure against the same bots we tuned against. Such an implementation gives us an opponent whose tactics we did not
design, and a rough bridge to the paper's setup. It is only a rough bridge, because the rules differ and our versions of two of the three opponents rest on the paper's
description rather than on code ([where we had to guess](#where-we-had-to-guess)).

## 2. How the Bonjour bots play

### Origin

The paper describes three fixed-policy opponents, FP-A, FP-B and FP-C. All three put a complete color set first,
trade one-way (buy or sell) and two-way (exchange), with or without cash, and send offers to several players at
once. They offer properties that are worth little to themselves but complete a set for the other player, and vice
versa. They differ in which properties they value:

- **FP-A** gives all properties equal priority;
- **FP-B** gives high priority to the four railroads and the two most expensive streets (the dark blue group), and
  low priority to the utilities;
- **FP-C** gives high priority to the railroads and the orange and light blue groups.

We reconstructed the shared rules from the paper and from the code of GNOME's rule-based player (`background_agent_v3_1`,
with its railroad-chasing variant and helper functions). The GNOME repository carries no license, so **we did not
copy its code**: our bots are written from scratch, and they match its thresholds and order of decisions, not its
text. We named them after the paper's first author. GNOME is the name of the whole simulator, not of a bot, so we do
not use it as a bot name. Our bots are "Bonjour A", "Bonjour B" and "Bonjour C", built on the tactics of FP-A, FP-B and FP-C.

### Shared rules

The rules revolve around one threshold, G = $200 (the salary for passing "Start"):

| Decision | Rule |
|---|---|
| Buying | Buys if at least G is left after the purchase. Buys a property that completes a set (including the fourth railroad and the second utility) whenever it can afford it; if cash is short, mortgages one property whose mortgage value alone covers the price (the cheapest such property, never the only one it owns in a color) |
| Auctions | Below the price, bids halfway between the highest bid and the price. At or above the price, bids only for a property that completes a set: a quarter of its cash above the highest bid. Otherwise passes |
| Building | Builds the house with the largest rent increase minus the house cost, with no reserve (it only needs to afford it). Then lifts mortgages if at least G is left |
| Offers (own turn) | With less than G, sells a lone property to the player whose set it completes, for the larger of 1.5x the price and half the buyer's cash. With G or more, wants rivals' lone properties that complete its own set: offers a property-for-property swap with cash on top, or, with no property to swap, buys for 1.125x the price while more than G/2 is left. At most one offer to each player per turn |
| Answering offers | Rejects if it gets nothing, cannot pay, or a mortgaged property is involved. Rejects if it loses more sets than it gains; accepts if it gains a set. With the number of sets unchanged, accepts if at least G/2 is left, and below G only if the trade is positive in property prices plus cash. Never counter-offers |
| Jail | With G or more, leaves with a jail release card or pays the fine; otherwise rolls |
| Debt | Mortgages the property with the lowest mortgage value if mortgages of unbuilt properties cover the debt; otherwise sells buildings (cheapest color first), then mortgages the rest, then goes bankrupt |

Here a **lone property** is the only property a player owns in its color group. Two rules matter most for what
follows: when it gains a set, the bot accepts whatever the price as long as it can pay; and when its sets are not
affected and it has enough cash, it accepts a trade even if the trade loses money.

### The three variants

In "Bonjour B" and "Bonjour C" a **high-priority** property is bought whenever affordable (no reserve G, as GNOME's
railroad-chasing variant does with railroads), bid for like a property that completes a set, wanted from rivals even
if it completes nothing, and never offered; to afford one, the bot mortgages a property outside its priorities. A
**low-priority** property ("Bonjour B"'s utilities) is bought only if 2G is left, and bid for up to half its price.
"Bonjour A" has no priorities and is the closest to GNOME's rule-based player.

### Adapting to our rules

Our game has rules that GNOME does not, so a few decisions had to be ours:

- **When to build.** GNOME's player builds and lifts mortgages only outside its own turn. Ours does so in
  [trade windows](../glossary.md#the-game-and-the-bots) during other players' turns and before rolling in its own
  turn, so it can still build when windows are off.
- **Offers only in its own turn.** As in GNOME, the bot proposes trades only in its own turn, before rolling. In
  trade windows it only builds or lifts mortgages, and passes in the windows of its own turn.
- **Auction steps.** Our bids go up in steps of $1, $10, $50 or $100; the bot uses the largest step not above the
  raise its rule asks for.
- **Cash on both sides.** A swap with cash in both directions is sent as the net difference, paid by one side; the
  value of the trade is the same.
- **No sale to the bank.** GNOME lets a player sell a property to the bank for half the price; our rules do not, so in
  debt the bot sells buildings and mortgages.
- **Jail release cards.** The bot never offers them, rejects offers that ask for them, and gives no value to cards it
  receives.
- **A mortgaged property taken over in a bankruptcy** is repaid at once if at least G is left, otherwise the bot pays
  the interest.
- **No shared safeguards.** Our own bots share a few safeguards (no feeding rivals, a debtor rescue, randomness in
  trades; see [tuning](01-from-rule-bots-to-v7.md#tuning-and-why-we-undid-half-of-it)). The Bonjour bots have none:
  they are a faithful copy of a fixed policy.

### Where we had to guess

The GNOME repository contains `background_agent_v3_1` (the counterpart of FP-A) and variants of it with other
goals, such as the railroad-chasing one; we did not find variants that favor the dark blue or the orange and light
blue groups. So "Bonjour B" and "Bonjour C" rest on the paper's description. We took their high-priority rule from the
railroad-chasing variant, and chose the low-priority rule (2G when buying, bids up to half the price) ourselves,
because the paper does not specify it. We also left out the higher amounts the railroad-chasing variant offers for
further railroads: every high-priority property is priced like any other (1.125x the price).

## 3. How strong they are

All results below use full trade windows and the 200-round limit, as in our main protocol. Intervals are 95% Wilson
intervals that treat every game as independent, which is the optimistic case
([how independent the games are](01-from-rule-bots-to-v7.md#1-the-question-and-how-we-measure)). With four equal
players chance level is 25%.

### Mixed tables

1,000 games, with four bots at each table drawn at random (with replacement) from our 8 bots on rules and the three
Bonjour bots, about 350-380 games per bot:

| Bot | Wins | 95% CI | Mean place |
|---|---|---|---|
| "Bonjour C" | **51.1%** | 46.0-56.1% | 2.05 |
| "Bonjour B" | **35.4%** | 30.6-40.5% | 2.24 |
| "Tactician" | 28.9% | 24.5-33.8% | 2.39 |
| "Calculator" | 26.9% | 22.6-31.7% | 2.47 |
| "GPT-Style" | 26.2% | 22.0-30.9% | 2.42 |
| "Investor" | 24.5% | 20.3-29.3% | 2.57 |
| "Master" | 20.1% | 16.2-24.6% | 2.66 |
| "Collector" | 19.7% | 15.9-24.2% | 2.59 |
| "Bonjour A" | 19.5% | 15.7-23.9% | 2.64 |
| "Cautious" | 11.9% | 9.0-15.5% | 2.75 |
| "Miser" | 10.8% | 8.0-14.3% | 2.73 |

The order of the three variants is the same as in the paper (C > B > A). "Bonjour C" is clearly first; "Bonjour B"'s
lead over "Tactician" (6.5 points) is about the size of the noise for a difference at this number of games, and the
middle of the table (from "Tactician" to "Investor") is within noise.

### Against three copies of one bot

400 games each (100 seeds x 4 seats):

| | 3 x "Random" | 3 x "Investor" | 3 x "Calculator" |
|---|---|---|---|
| "Bonjour A" | 100.0% (99.1-100) | 19.2% (15.7-23.4) | 14.0% (10.9-17.7) |
| "Bonjour B" | 99.2% (97.8-99.7) | 41.5% (36.8-46.4) | 33.8% (29.3-38.5) |
| "Bonjour C" | 98.2% (96.4-99.2) | 48.8% (43.9-53.6) | **41.5%** (36.8-46.4) |

"Bonjour A", the closest to GNOME's own player, is weak: it builds with no reserve and accepts losing trades.
"Bonjour C" wins 41.5% against three "Calculators", well above the 25% chance level; for comparison,
"Calculator" against three copies of itself won 22.5% in our main protocol ([bots](../bots.md#calculator)).

### Which priority helps

To find which priority makes the difference, we gave "Bonjour A" one priority group at a time and played it
against three "Calculators" (400 games each):

| "Bonjour A" with high priority on | Wins (95% CI) |
|---|---|
| nothing ("Bonjour A" itself) | 14.0% (10.9-17.7) |
| the dark blue group only | 20.2% (16.6-24.5) |
| the railroads only | 28.7% (24.5-33.4) |
| the orange and light blue groups | **34.8%** (30.2-39.5) |

The orange and light blue groups help most, then the railroads; the dark blue group alone is within noise of no
priority at all. This is the first hint of the mechanism: a priority makes the bot go after a few groups
persistently, and our bots sell it the pieces ([the mechanism](#the-mechanism)).

## 4. Two tournaments: rock, paper, scissors

To see how the Bonjour bots fit among all our bots, we ran two tournaments with the same setup: four-player tables
drawn at random from a pool, random seats, full trade windows, the 200-round limit (reached in 22 and 12 games of
3,000), 3,000 games each. The only difference is the pool.

### Without the absurd bots

A pool of 12: our 9 honest bots ("Random" included) and the three Bonjour bots, about 960-1,070 games per bot.

| Bot | Wins | 95% CI | Mean place |
|---|---|---|---|
| "Bonjour C" | **56.3%** | 53.3-59.4% | 1.89 |
| "Bonjour B" | **38.2%** | 35.2-41.3% | 2.09 |
| "GPT-Style" | 34.1% | 31.2-37.0% | 2.19 |
| "Tactician" | 33.5% | 30.7-36.5% | 2.18 |
| "Calculator" | 29.7% | 26.9-32.6% | 2.31 |
| "Bonjour A" | 22.0% | 19.6-24.6% | 2.53 |
| "Collector" | 21.4% | 19.0-24.1% | 2.56 |
| "Investor" | 20.7% | 18.3-23.2% | 2.58 |
| "Master" | 20.1% | 17.7-22.7% | 2.51 |
| "Cautious" | 13.1% | 11.2-15.4% | 2.56 |
| "Miser" | 9.5% | 7.8-11.5% | 2.67 |
| "Random" | 0.0% | 0.0-0.4% | 3.95 |

"Bonjour C" is first by a wide margin. "Bonjour B"'s lead over "GPT-Style" (4.1 points) is at the edge of noise
(about +/-4.2 points for a difference at these numbers of games).

### With the absurd bots

A pool of 16: the same 12 plus "Absurd GPT-Style", "Absurd Calculator", "Absurd Investor" and "Hustler"
([absurd bots](../bots.md#absurd-bots)), about 720-780 games per bot.

| Bot | Wins | 95% CI | Mean place | Without the absurd bots |
|---|---|---|---|---|
| "Hustler" | **54.6%** | 51.0-58.2% | 1.77 | - |
| "Absurd GPT-Style" | 49.0% | 45.4-52.5% | 1.87 | - |
| "Absurd Calculator" | 48.2% | 44.7-51.7% | 1.89 | - |
| "Absurd Investor" | 34.2% | 30.9-37.6% | 2.30 | - |
| "GPT-Style" | 30.0% | 26.8-33.5% | 2.23 | 34.1% |
| "Calculator" | 29.3% | 26.1-32.6% | 2.24 | 29.7% |
| "Tactician" | 28.8% | 25.7-32.2% | 2.22 | 33.5% |
| "Bonjour C" | 21.8% | 19.0-24.9% | 2.79 | 56.3% |
| "Master" | 19.8% | 17.2-22.8% | 2.45 | 20.1% |
| "Collector" | 19.3% | 16.6-22.3% | 2.51 | 21.4% |
| "Investor" | 17.5% | 15.0-20.4% | 2.57 | 20.7% |
| "Bonjour B" | 15.3% | 12.9-18.1% | 2.96 | 38.2% |
| "Cautious" | 13.7% | 11.4-16.3% | 2.51 | 13.1% |
| "Miser" | 8.6% | 6.7-10.8% | 2.64 | 9.5% |
| "Bonjour A" | 8.5% | 6.7-10.7% | 3.12 | 22.0% |
| "Random" | 0.0% | 0.0-0.5% | 3.95 | 0.0% |

The bots that send bad offers take the top four places, each above its honest original. "Bonjour C" falls by 34.5
points, "Bonjour B" by 22.9 and "Bonjour A" by 13.5, and all three end with a mean place worse than 2.5. Our honest
bots change little: "Calculator" -0.4, "GPT-Style" -4.1 and "Tactician" -4.7 points, changes within or at the edge
of noise (about +/-4.4 points for a difference at these numbers of games), and some drop is expected anyway in a pool
with four strong new players.

### The mechanism

We read the mechanism from the bots' rules; we have not yet counted the accepted trades game by game.

**How the Bonjour bots beat our honest bots.** With $200 or more, a Bonjour bot offers to buy any rival's lone property
that completes one of its sets (or that it has a high priority on) for 1.125x the price. Our bots value a lone
property low, so each such sale looks profitable to them on its own, and they accept. A Bonjour bot with a
high priority collects a group piece by piece this way. Then, when it lacks one property for a set, it accepts any
price it is asked, as long as it can pay: in one game against "Calculators" a Bonjour bot bought a light blue
street with a price of $100 for $390 (3.9x the price). Each trade is fair or even good for our bot when judged on its
own; the sequence of trades hands the Bonjour bot complete sets, and the sets win the game. The ablation above fits
this: a priority on the orange and light blue groups alone lifts the weak "Bonjour A" from 14.0% to 34.8% against
three "Calculators".

**Why our bots sell the pieces.** Most of our bots judge a trade with the
[shared trade evaluation](../glossary.md#the-game-and-the-bots), which adds its bonus for a color group only when the
group is complete, so selling the first piece of a group for a small premium looks profitable on its own. The
[feed rule](../glossary.md#the-game-and-the-bots) blocks mainly a sale that completes a rival's set at once or nearly
(gives the rival two of three properties in a group), and lets even that through when the price compensates. A
collector that buys the first pieces cheaply and pays enough for the last one is therefore not stopped.

**How the bots that send bad offers beat the Bonjour bots.** Our [absurd bots](../bots.md#absurd-bots) and
["Hustler"](../bots.md#hustler) send offers that are clearly unprofitable for the recipient. A Bonjour bot rejects
three of the four kinds: cash for nothing (it gets nothing), the jail release card for nothing (it never trades the
cards) and a mortgaged junk property for a fortune (it rejects any mortgaged property). But it accepts the fourth,
**a property for pennies**, whenever the property is not part of a complete set and it keeps enough cash: with its
sets unchanged and at least $200 left, it does not look at the price at all. A Bonjour bot with cash therefore gives
its properties away one by one, including the lone pieces it has been collecting.

**Why our honest bots resist.** Our bots judge every offer with a shared evaluation and reject the clearly bad ones;
in entry 1 "Calculator" and "GPT-Style" accepted none of about 100,000 offers for nothing
([the absurd offers](01-from-rule-bots-to-v7.md#the-absurd-offers)).

## 5. Resistant Bonjour variants

To separate the Bonjour bots' strength in collecting sets from their weakness against bad offers, we built
resistant variants: "Resistant Bonjour A", "Resistant Bonjour B" and "Resistant Bonjour C", with the same priorities
as the originals.

### What we changed

Only the answer to an offer or a counter-offer. The original rule decides first, and all its reasons to reject stay
(nothing received, not enough cash, a mortgaged property, jail release cards, a lost set, the $200 and $100
thresholds). An offer the original would accept must then pass two safeguards that our own bots already share; we
wrote no new rule:

- **the feed rule:** it does not give away a property that completes a rival's set or gives the rival two of three
  properties in a group, unless the price compensates, that is unless the rival's gain is at most 1.25x its own
  ([tuning](01-from-rule-bots-to-v7.md#tuning-and-why-we-undid-half-of-it));
- **a positive shared evaluation:** the trade must be worth more than zero by the shared trade evaluation that most
  of our bots on rules answer offers with.

If either check fails, it rejects. The readiness to overpay for the property that completes a set stays, but only
within the evaluation, which adds a bonus of 1.5x the prices of the group for a complete set: a light blue street
with a price of $100 is still worth buying for $300, but not for $1,000. Everything else is unchanged: buying,
auctions, building, jail, debts, mortgaged properties taken over, and its own offers, including the sale of lone
properties when it has less than $200, which is part of the Bonjour character. It still never counter-offers.

### The two tournaments again

We repeated both tournaments of [section 4](#4-two-tournaments-rock-paper-scissors) with the three resistant
variants added to each pool, with the same setup: four-player tables drawn at random, random seats, full trade
windows, the 200-round limit (reached in 12 and 18 games of 3,000), 3,000 games each.

Without the absurd bots, a pool of 15 (the 12 of section 4 and the three resistant variants), about 760-850 games per
bot:

| Bot | Wins | 95% CI | Mean place | Games |
|---|---|---|---|---|
| "Resistant Bonjour C" | **50.7%** | 47.3-54.1% | 1.91 | 848 |
| "Bonjour C" | 47.6% | 44.2-51.1% | 2.01 | 783 |
| "Resistant Bonjour B" | **35.8%** | 32.5-39.3% | 2.24 | 760 |
| "Bonjour B" | 30.1% | 27.0-33.4% | 2.25 | 797 |
| "GPT-Style" | 28.9% | 25.9-32.0% | 2.29 | 835 |
| "Tactician" | 27.9% | 24.9-31.2% | 2.32 | 774 |
| "Calculator" | 27.5% | 24.6-30.7% | 2.36 | 806 |
| "Collector" | 26.6% | 23.7-29.7% | 2.46 | 824 |
| "Resistant Bonjour A" | 23.2% | 20.3-26.3% | 2.55 | 773 |
| "Master" | 20.0% | 17.4-22.9% | 2.54 | 805 |
| "Investor" | 18.0% | 15.5-20.7% | 2.68 | 834 |
| "Bonjour A" | 17.2% | 14.7-19.9% | 2.68 | 810 |
| "Cautious" | 12.6% | 10.4-15.2% | 2.59 | 761 |
| "Miser" | 7.8% | 6.1-9.9% | 2.69 | 795 |
| "Random" | 0.0% | 0.0-0.5% | 3.94 | 795 |

With the absurd bots, a pool of 19 (the 16 of section 4 and the three resistant variants), about 595-680 games per
bot:

| Bot | Wins | 95% CI | Mean place | Games |
|---|---|---|---|---|
| "Hustler" | **46.3%** | 42.4-50.2% | 1.98 | 622 |
| "Absurd GPT-Style" | **46.3%** | 42.4-50.2% | 1.94 | 633 |
| "Absurd Calculator" | 45.2% | 41.5-49.1% | 1.99 | 652 |
| "Resistant Bonjour C" | 43.8% | 40.0-47.8% | 2.05 | 609 |
| "Absurd Investor" | 32.3% | 28.7-36.0% | 2.30 | 620 |
| "Resistant Bonjour B" | 32.2% | 28.7-35.8% | 2.24 | 653 |
| "Calculator" | 28.5% | 25.1-32.1% | 2.37 | 632 |
| "Tactician" | 26.7% | 23.4-30.2% | 2.29 | 634 |
| "Bonjour C" | 23.8% | 20.6-27.3% | 2.75 | 622 |
| "GPT-Style" | 23.6% | 20.4-27.1% | 2.39 | 602 |
| "Resistant Bonjour A" | 21.8% | 18.9-25.1% | 2.53 | 679 |
| "Collector" | 20.1% | 17.1-23.4% | 2.56 | 617 |
| "Master" | 18.7% | 15.9-21.9% | 2.54 | 641 |
| "Bonjour B" | 18.5% | 15.7-21.8% | 2.84 | 626 |
| "Investor" | 18.0% | 15.1-21.3% | 2.61 | 595 |
| "Cautious" | 12.3% | 10.1-15.1% | 2.55 | 664 |
| "Miser" | 9.4% | 7.3-11.9% | 2.58 | 641 |
| "Bonjour A" | 8.0% | 6.2-10.4% | 3.07 | 635 |
| "Random" | 0.0% | 0.0-0.6% | 3.94 | 623 |

The originals win less here than in section 4 ("Bonjour C" 47.6% against 56.3% without the absurd bots), as expected
when three more strong players share the wins.

### What the tournaments show

- **Resistance removes the weakness.** Adding the bots that send bad offers costs the resistant variants 1.4 ("A"),
  3.6 ("B") and 6.9 ("C") points, and the originals 9.2, 11.6 and 23.8 points. The drop of "Resistant Bonjour C" is
  slightly beyond the noise (about +/-5.2 points for this difference), but some drop is expected in a pool with four
  strong new players, as for our honest bots in section 4.
- **Resistance helps without the absurd bots too.** Each resistant variant wins more than its original in the same
  tournament: "A" 23.2% against 17.2% (+6.0 points, noise about +/-3.9), "B" 35.8% against 30.1% (+5.7, about
  +/-4.7), "C" 50.7% against 47.6% (+3.1, about +/-4.9, within noise). Refusing losing trades pays even when nobody
  sends deliberately bad ones; we have not checked which trades make the difference.
- **"Resistant Bonjour C" is the strongest bot that does not send bad offers, in both pools.** It is first of 15
  without the absurd bots, and fourth of 19 with them, within noise of "Hustler", "Absurd GPT-Style" (46.3% each) and
  "Absurd Calculator" (45.2%), and well ahead of "Calculator" (28.5%).
- **The bots that send bad offers still lead,** because the pool still contains players they can exploit: the
  non-resistant Bonjour bots and "Random". The next section removes them.

## 6. A tournament without donors

### The role of "Random"

"Random" plays in all four tournaments so far. It picks uniformly among the legal moves at every decision. When it
answers an offer, its legal moves are accept and reject (counter-offers are not among the moves it picks from), so it
accepts about half of all offers, whatever their terms. It never proposes a trade, buys an affordable property half
of the time, and picks its auction moves at random. It wins 0% and acts as a **donor**, a player that accepts trades
that lose: by these rules, a bot that sends it a property-for-pennies or cash-for-nothing offer gets it about half
the time, and a Bonjour bot that offers to buy its lone properties does too. We have not counted these trades table
by table. "Random" played 795 and 623 of the 3,000 games of the two tournaments with the resistant variants, so a
noticeable share of tables had such a donor. The non-resistant Bonjour bots are donors of the same kind for the bots that
send bad offers ([the mechanism](#the-mechanism)). Donors inflate the results of players that exploit weaker ones,
the bots that send bad offers above all.

### The setup

To measure strength without donors, we ran one more tournament with the same setup as before (four-player tables
drawn at random from the pool, random seats, full trade windows, the 200-round limit, reached in 32 games of 3,000;
3,000 games) and a pool of 15 in which no player accepts bad offers:

- our 8 honest bots without "Random": "Calculator", "Tactician", "GPT-Style", "Investor", "Master", "Collector",
  "Cautious" and "Miser";
- the bots that send bad offers: "Absurd GPT-Style", "Absurd Calculator", "Absurd Investor" and "Hustler";
- the three resistant variants.

This is the pool of 19 of [section 5](#the-two-tournaments-again) without its four donors ("Random" and the three
non-resistant Bonjour bots). As in all tournaments of this entry, "Simulator" and "Fast Simulator" are not in the pool.
Each bot played 737-836 games.

### Results

| Bot | Wins | 95% CI | Mean place | Games | With the donors (pool of 19) |
|---|---|---|---|---|---|
| "Resistant Bonjour C" | **43.1%** | 39.7-46.4% | 2.14 | 836 | 43.8% |
| "Resistant Bonjour B" | 35.2% | 31.9-38.6% | 2.29 | 796 | 32.2% |
| "Tactician" | 32.3% | 29.1-35.7% | 2.23 | 770 | 26.7% |
| "Calculator" | 30.7% | 27.4-34.1% | 2.41 | 737 | 28.5% |
| "GPT-Style" | 28.0% | 25.1-31.2% | 2.34 | 821 | 23.6% |
| "Absurd Calculator" | 27.9% | 24.9-31.1% | 2.43 | 778 | 45.2% |
| "Hustler" | 27.0% | 24.1-30.2% | 2.49 | 795 | 46.3% |
| "Absurd GPT-Style" | 25.5% | 22.6-28.6% | 2.45 | 828 | 46.3% |
| "Master" | 22.8% | 20.0-25.8% | 2.56 | 821 | 18.7% |
| "Collector" | 22.3% | 19.5-25.3% | 2.61 | 789 | 20.1% |
| "Investor" | 22.1% | 19.4-25.1% | 2.60 | 806 | 18.0% |
| "Absurd Investor" | 17.8% | 15.2-20.6% | 2.80 | 760 | 32.3% |
| "Resistant Bonjour A" | 16.7% | 14.4-19.4% | 2.79 | 830 | 21.8% |
| "Cautious" | 14.2% | 12.0-16.8% | 2.63 | 803 | 12.3% |
| "Miser" | 10.0% | 8.1-12.2% | 2.72 | 830 | 9.4% |

The same 15 bots won 28.3% on average in the pool of 19, where the donors won little, and exactly 25% here, so a
small drop for everyone would be expected. Instead the honest bots gained 0.6-5.6 points (each change within or at
the edge of noise, about +/-3-5 points), the bots that send bad offers lost 14.5-20.8 points (noise about +/-5), and
"Resistant Bonjour C" barely moved (-0.7 points).

### What it shows

This is our best measure of strength in this series: nobody at the table gives wins away by accepting bad offers.

- **Without donors, the bots that send bad offers lose their edge.** They play about like their honest originals,
  if anything slightly below them: "Absurd Calculator" 27.9% and "Hustler" 27.0% against "Calculator"'s 30.7%
  (-2.8 and -3.7 points, noise about +/-4.6), "Absurd GPT-Style" 25.5% against "GPT-Style"'s 28.0% (-2.5, about
  +/-4.3), "Absurd Investor" 17.8% against "Investor"'s 22.1% (-4.3, about +/-3.9, at the edge of noise). Their lead
  in the earlier tournaments came from exploiting "Random" and the non-resistant Bonjour bots.
- **"Resistant Bonjour C" is clearly the strongest, here too.** It wins 43.1%, 7.9 points ahead of "Resistant Bonjour
  B" and 10.7 ahead of "Tactician" (noise about +/-4.7 for each difference). Its lead does not depend on players
  that accept bad offers; it comes from how it plays: it hunts a few groups and collects them piece by piece through
  trades its partners accept by their own evaluation. The pool still contains such partners: our honest bots, and
  the bots that send bad offers, which answer offers like them ([why our bots sell the pieces](#the-mechanism)).
- **"Resistant Bonjour B" is in the leading group,** 2.8 points ahead of "Tactician", within noise.
- **"Resistant Bonjour A" is weak.** At 16.7% it is above only "Cautious" and "Miser": collecting piece by piece
  helps little without priority groups worth collecting, which fits the priority test of
  [section 3](#which-priority-helps).

## 7. The plain result

We say it directly: **the bots we had considered strong turned out to be clearly weaker than the tactic of the Bonjour
bots.** Our own honest bots, "Calculator", "Tactician" and "GPT-Style", were the product of a lot of design and tuning:
a Markov-chain valuation of the board, a 12-point strategy, safeguards, character tests. A short list of simple rules for
collecting sets by trade, which we wrote from the description in a paper and the rules of a simulator, is much more effective in our game than any
of them, by a margin well beyond the noise, in every kind of measurement we made:

| Measurement | Bots with the Bonjour tactic | Our best honest bots |
|---|---|---|
| Mixed tables, 1,000 games ([section 3](#mixed-tables)) | "Bonjour C" 51.1%, "Bonjour B" 35.4% | "Tactician" 28.9%, "Calculator" 26.9%, "GPT-Style" 26.2% |
| 3,000-game tournament, pool of 12 ([section 4](#without-the-absurd-bots)) | "Bonjour C" 56.3%, "Bonjour B" 38.2% | "GPT-Style" 34.1%, "Tactician" 33.5%, "Calculator" 29.7% |
| 3,000-game tournament without donors, pool of 15 ([section 6](#results)) | "Resistant Bonjour C" 43.1%, "Resistant Bonjour B" 35.2% | "Tactician" 32.3%, "Calculator" 30.7%, "GPT-Style" 28.0% |
| One against three copies, 400 games ([section 3](#against-three-copies-of-one-bot), [section 8](#8-closing-the-gap-with-a-wary-calculator)) | "Bonjour C" against three "Calculators": 41.5% | "Calculator" against three "Bonjour C" bots: 12.3% |
| Elo ranking ([section 13](#13-the-final-search-and-the-elo-ranking)) | "Resistant Bonjour C + Windows" 1569, "Resistant Bonjour B + Windows" 1557 | "Wary Calculator" 1521, "Tactician" 1508, "GPT-Style" 1507, "Calculator" 1495 |

(Chance level in the first four rows is 25%. The detail of each bot is in [bots.md](../bots.md).)

How to read it:

- **It is not an effect of donors.** The lead stays when nobody at the table accepts bad offers (third row) and in the
  ranking, whose pools have no such players either.
- **It is not an effect of missing safeguards.** The resistant variants carry the same safeguards as our own bots, and
  they win more than the originals even when nobody sends bad offers.
- **It is the tactic, not just any collector.** "Bonjour A", which gives every property the same priority, wins 19.5% in
  mixed tables, less than "Calculator". What wins is collecting piece by piece the right groups, the railroads and the
  orange and light blue groups (or the railroads and the dark blue group), as the test of
  [section 3](#which-priority-helps) shows.
- **Our bots lose to it because of a gap in how they value a trade** ([the mechanism](#the-mechanism)), not because they
  play badly in other areas: they still beat the weaker bots and refuse bad offers.
- **It is a statement about our game and our implementation**, not about the game of the paper, whose rules are simpler
  ([section 1](#1-why-implement-bots-from-earlier-tactics)).

The rest of this entry is the story of what we did next: first to close the gap in our own bots, then to find which
version of the tactic works best in our game.

## 8. Closing the gap with a wary Calculator

The first answer was to repair the cause. Our shared trade evaluation adds its bonus for a color group only when the
group is complete, so selling the first piece of a group for a small premium looks profitable. In 100 games of
"Calculator" against three "Bonjour C" bots, it accepted 61 offers to sell a railroad to a player without railroads,
30 offers to sell an orange square and 27 to sell a light blue square to a player with no squares of that color, each
time at 1.125x the price. We built a variant, "Wary Calculator" ([bots.md](../bots.md#wary-calculator)). It plays like
"Calculator", but values a property that brings a rival closer to a set at its price plus the set bonus, and so answers
a piece-by-piece offer with a refusal or with a counter-offer that makes the price match that value (a light blue street
priced $100, offered $112, is countered at about $590).

Two things about the tuning are worth recording:

- **The first version did nothing.** With low weights for the progress toward a set it won 12.8% against three "Bonjour
  C" bots, against 11.9% for "Calculator" (800 games). It sold the same squares, only at about 1.7x the price, because a
  Bonjour bot accepts any counter-offer that leaves it $100. The result grew as we raised the weights, up to counting a
  square of a free color at its price plus the whole bonus.
- **Railroads needed their own rule.** Without a bonus for railroads the results against "Bonjour C" and "Bonjour B"
  stayed at 25.3% and 23.8%, against 28.7% and 30.0% in the final setting (600 games each).

The result against single opponents is clear. On new seeds, 400 games each, "Wary Calculator" wins 26.3% against three
"Bonjour C" bots against 12.3% for "Calculator", and 30.0% against three "Bonjour B" bots against 23.8%. In the
tournaments the gain is small and within the intervals. With "Wary Calculator" added to the two pools of
[section 4](#4-two-tournaments-rock-paper-scissors) it won 33.9% against 32.2% for "Calculator" without the bots that
send bad offers, and 30.8% against 26.9% with them. It sits at only about 30% of the tables, and the Bonjour bots keep
collecting sets from the other bots: "Bonjour C" still won 51.0% in the pool without bad-offer senders. So the wary
version resists the Bonjour tactic but does not beat it, and it is not stronger than the plain resistant variants.

## 9. Bonjour tactics with Calculator parts

The opposite idea was to give a Bonjour bot some of what "Calculator" does well. Offers are the Bonjour bots' weapon,
while "Calculator" has its own way of proposing trades (and auctions with its own limits from board probabilities). We
built four hybrids of ["Resistant Bonjour C"](../bots.md#resistant-bonjour-a-b-c) ([bots.md](../bots.md#bonjour-hybrids-with-calculator)):

- "Bonjour Calculator": the offers of "Calculator"; in auctions, for squares in its high-priority groups or squares that
  complete a set, the more aggressive of the two bots' bids, and the bid of "Calculator" for other squares;
- "Bonjour Calculator Pure": the same offers, with every auction played by "Calculator" (an ablation of the previous one);
- "Bonjour Calculator Auctions": the offers of "Resistant Bonjour C" and every auction played by "Calculator";
- "Bonjour Mixed Auctions": the offers of "Resistant Bonjour C" and the auctions of "Bonjour Calculator".

Their results are in the next section, where they met the other candidates.

## 10. A first search for the best bot

By now we had twenty-odd bots, many of them near-equal, and a flat 3,000-game tournament spreads the games too thin to tell the
leaders apart. So we used successive halving ([glossary](../glossary.md#how-we-measure)): each table seats one candidate and
three sparring partners drawn from the pool, candidates are compared by their win rate and the weaker half drops out
after every round. All runs below used full trade windows, the 200-round limit and random seats.

### Twenty candidates

20 candidates: every bot of ours except "Random", the two simulators and the non-resistant Bonjour bots, so the pool had no
player that accepts bad offers. Every bot was also a sparring partner. Five rounds of 4,000 games.

| Round | Games per candidate | Candidates | Leaders (wins) |
|---|---|---|---|
| 1 | 200 | 20, 10 go on | "Resistant Bonjour C" 35.0%, "Bonjour Mixed Auctions" 35.0%, "Resistant Bonjour B" 34.0%; "Calculator" 23.0% did not go on |
| 2 | 600 | 10, 5 go on | "Bonjour Mixed Auctions" 34.7%, "Resistant Bonjour C" 33.2%, "Resistant Bonjour B" 33.0%, "Tactician" 31.0%, "Wary Calculator" 30.3% |
| 3 | 1,400 | 5, 3 go on | "Resistant Bonjour C" 33.1%, "Bonjour Mixed Auctions" 32.4%, "Resistant Bonjour B" 32.1%; "Wary Calculator" 29.9% and "Tactician" 29.2% out |
| 4 | 2,733 | 3, 2 go on | "Resistant Bonjour B" 33.6%, "Bonjour Mixed Auctions" 32.9%, "Resistant Bonjour C" 32.7% out |
| 5 | 4,733 | 2, 1 wins | "Resistant Bonjour B" 34.0% (32.7-35.4%) against "Bonjour Mixed Auctions" 33.1% (31.8-34.4%) |

What it showed:

- **The Calculator offers lose.** "Bonjour Calculator" and "Bonjour Calculator Pure" won 24.5% and 24.7% in round 2,
  about 9 points below "Resistant Bonjour C" (33.2%, noise of the difference about +/-6), and as sparring partners 21.9% and 21.1%
  against 30.5% for "Resistant Bonjour C". Replacing the Bonjour offers with ours costs about 9-10 points.
- **The Calculator auctions do not matter.** "Bonjour Calculator Auctions" 29.2% and "Bonjour Mixed Auctions" 34.7% in
  round 2, and "Bonjour Mixed Auctions" lost the final by 0.9 points: within noise of "Resistant Bonjour C".
- **The top three are a tie.** In round 4 the three leaders were within 0.9 points of each other, and the final gap of
  0.9 points is inside the noise (about +/-1.9).
- **Our own bots drop out early.** "Wary Calculator" and "Tactician" were cut in round 3, and "Calculator" in round 1,
  at 23.0% over 200 games (noise about +/-6 points).

A second run took five of them as candidates ("Resistant Bonjour B", "Resistant Bonjour C", "Bonjour Mixed Auctions",
"Wary Calculator" and "Tactician") with 20 sparring partners and three rounds of 3,000 games. In round 1 (500 games
each) "Resistant Bonjour B" 32.4%, "Resistant Bonjour C" 31.6% and "Wary Calculator" 30.2% went on; "Bonjour Mixed
Auctions" 29.8% and "Tactician" 27.8% did not. In round 2 (1,500 each) "Resistant Bonjour B" 34.1% and "Resistant
Bonjour C" 32.5% went on, "Wary Calculator" had 31.7%. In the final (3,000 each) "Resistant Bonjour B" won 33.9%
(32.3-35.6%) against 31.7% (30.1-33.4%).

### B against C

Both runs were won by "Resistant Bonjour B", which hunts the railroads and the dark blue group, although in the earlier
tournaments "Bonjour C" (railroads, orange, light blue) was ahead. In both runs the sparring partners included a second
copy of "Resistant Bonjour C" and four hybrids built on it, all of which hunt the same groups. So we ran the two as the
only candidates, with a pool of 16 sparring partners that had no hybrids on "Resistant Bonjour C" (but a second copy of
each of the two, "Resistant Bonjour A", our honest bots, the absurd bots and "Hustler"): 6,000 games, 3,000 each. "Resistant
Bonjour C" won 37.8% (36.0-39.5%) and "Resistant Bonjour B" 35.4% (33.7-37.1%). The gap of 2.4 points is at the edge of
the noise (about +/-2.4), but it points the other way than in the two runs with the hybrids.

This fits an effect we know from the classical analyses of the board
([related work](../related-work.md#markov-chain-analyses-of-the-board)): the board says which groups are worth owning, and
not which are worth chasing when several players chase them. The payoff of hunting a group falls with the number of
other bots that hunt it. We do not claim to have shown it: every gap here is within or at the edge of the noise, and
each comparison is a single run. What we can say is that "Resistant Bonjour B" and "Resistant Bonjour C" are about
equally strong and that which one leads depends on the pool.

## 11. Absurd versions of the leading bots

In the tournaments with donors the [absurd bots](../bots.md#absurd-bots) led the table, and we wondered whether the same
extra offer would lift the new leaders. We built absurd versions of five of them ("Absurd Wary Calculator", "Absurd
Tactician", "Absurd Resistant Bonjour B", "Absurd Resistant Bonjour C" and "Absurd Bonjour Mixed Auctions";
[bots.md](../bots.md#absurd-versions-of-the-leading-bots)): one absurd offer per turn after the roll and one in a trade
window, everything else as the original. For the Bonjour bots we had to let the original manage its assets first and send
the absurd offer only in place of the pass, because a Bonjour bot builds only in the windows of other players' turns.

Against three copies of its original (200 games), the absurd versions of the three Bonjour-based bots won 22.0%, 23.5%
and 23.0%, neutral within the noise: 120-160 absurd offers per game and not one accepted, because the resistant Bonjour
bots refuse them without a counter-offer. "Absurd Tactician" won 20.0% and its original accepted 0.2% of its 174 offers
per game, mostly the sale of a mortgaged property. "Absurd Wary Calculator" lost clearly (14.5%, interval 10.3-20.0%):
its original answers an absurd offer with a counter-offer, and the absurd version accepts about one such counter-offer
per game, a trade that would not have happened without the absurd offer. The order of moves in the window is not the
cause.

In a pool with no player that accepts bad offers, the absurd versions played as well as their originals but not
better ([bots.md](../bots.md#absurd-versions-of-the-leading-bots)). That is what [section 6](#6-a-tournament-without-donors)
predicted: the edge of the bad-offer senders comes from donors.

## 12. The windows variants

The most successful change came from reading the Bonjour tactic against the mechanics of our game. The paper's game has
no trade windows; ours lets every player, in turn, propose one trade at fixed moments of another player's turn
(before a payment, after a property is acquired, after a negotiation, at the end of a turn). The Bonjour bots follow the
paper and propose trades only in their own turn, before the roll. In a window they only build and lift mortgages and then
pass, and they say nothing when raising money for a debt or answering with a counter-offer. That is a lot of silent
decisions in a game with hundreds of them.

"Resistant Bonjour B + Windows" and "Resistant Bonjour C + Windows" adapt the tactic to this mechanic
([bots.md](../bots.md#resistant-bonjour-b-and-c-with-windows)). They are the resistant variants in full, plus offers
exactly where the Bonjour bots are silent. In windows of other players' turns they first build as the Bonjour bot does,
and then propose a trade, before a payment first buying a property from the player who is about to pay. When they have to
raise money for a debt and the bank cannot cover it they sell a property to a player. And when they refuse an offer
they may answer with a counter-offer, but only one they would accept themselves as an offer. All three kinds of offer
come from an instance of "Wary Calculator". We tried first to replace the Bonjour offers with the offers of "Calculator",
and it failed ([section 9](#9-bonjour-tactics-with-calculator-parts)); here nothing is replaced and the Bonjour
priorities and thresholds keep deciding what to want.

In a 12,000-game elimination tournament with the two windows variants and their originals as candidates, the windows
variants led from the first round. In round 1 (1,500 games each) "Resistant Bonjour C + Windows" won 47.0% (44.5-49.5%)
against 35.9% for "Resistant Bonjour C", and "Resistant Bonjour B + Windows" 38.3% (35.8-40.8%) against 32.3% for
"Resistant Bonjour B". The intervals of each pair do not overlap. In the final round "Resistant Bonjour C + Windows"
won 44.3% (3,000 games). Offers in windows are fewer than offers before the roll (3.5-4.1 per game against 8.8-22.2), and about one in six is
accepted; in addition about 0.9 sales in debt and 1.5 counter-offers per game are accepted. We have not separated the
share of the three.

## 13. The final search and the Elo ranking

To check the result on a larger field, we ran a second elimination tournament and recorded its games in the rating. The
candidates were the two windows variants, their originals, "Bonjour Mixed Auctions", "Wary Calculator" and "Tactician";
the 27 sparring partners were all our other bots, among them the absurd versions of the leading bots and the hybrids.
Again there was no player that accepts bad offers. Three rounds of 4,000 games, 12,000 in all.

| Round | Games per candidate | Results (wins) |
|---|---|---|
| 1 | about 570 | "Resistant Bonjour C + Windows" 37.4%, "Resistant Bonjour B + Windows" 34.4%, "Bonjour Mixed Auctions" 31.5%, "Wary Calculator" 28.7%, "Resistant Bonjour C" 28.7%, "Resistant Bonjour B" 28.4%, "Tactician" 23.8% |
| 2 | about 1,570 | "Resistant Bonjour C + Windows" 37.3%, "Resistant Bonjour B + Windows" 36.2%, "Bonjour Mixed Auctions" 32.8%, "Wary Calculator" 30.0% |
| 3 | about 3,570 | "Resistant Bonjour C + Windows" 38.5% (36.9-40.1%), "Resistant Bonjour B + Windows" 35.5% (33.9-37.1%) |

"Resistant Bonjour C + Windows" won. The two windows variants lead their originals again, by 8.7 and 6.0 points in round 1.
Their gap in the final, 3.0 points, is a little beyond the noise (about +/-2.2).

The games of this tournament and of a few shorter ones give the Elo ranking in [bots.md](../bots.md#current-elo-ranking).
Its main facts, with the caveat that neighbors a few points apart cannot be told apart:

- "Resistant Bonjour C + Windows" (1569) and "Resistant Bonjour B + Windows" (1557) are first and second, 29 and 41 points
  above the next bot. A gap of 74 points, which is how far "Resistant Bonjour C + Windows" is above "Calculator" (1495),
  corresponds to finishing ahead of it in about 60% of pairwise comparisons.
- Places 3 to 10 lie within 17 points of each other (1511-1528): four absurd versions of the leading bots, "Wary
  Calculator", "Resistant Bonjour C" and "Bonjour Mixed Auctions".
- Our honest bots are in the middle: "Tactician" 1508, "GPT-Style" 1507, "Calculator" 1495. The two hybrids with the
  offers of "Calculator" are near the bottom (1444 and 1442), with "Master", "Collector", "Cautious", "Investor" and
  "Miser".
- "Wary Calculator" is the highest of the bots we designed ourselves, about 50 points below the leading variant.

## 14. Variance parameters

One more thing changed in this period without changing any of the results above: we gave every bot two parameters that
make its play less repetitive, **trade variability** (the choice among the offers it could make, the cash in them and the
threshold at which it accepts an offer all vary a little) and **random moves** (a small chance of a random legal action).
Our own bots have used a trade variability of 0.1 to 0.3 since their characters were restored; the Bonjour bots, which
are fixed rules, now accept both parameters too, with default 0, which reproduces their earlier games exactly. The
description is in [bots.md](../bots.md#variance-parameters). All the results of this entry were measured with the
default values, so none of them depends on the extra randomness.

## 15. Why we did not see it earlier

When we designed our bots, we tuned them mostly against each other
([tuning](01-from-rule-bots-to-v7.md#tuning-and-why-we-undid-half-of-it)). All of them except "GPT-Style" and
"Tactician", which have their own trade rules, judge trades with the same shared trade evaluation, so they share its
blind spot: a lone property is worth little, and a group counts only when it is complete. Nobody in the pool played a
style that exploits this, so tuning against the pool could not reveal it. An opponent from outside our environment,
with very simple rules, showed the gap in our first measurement with it.

It is the same pattern as [the absurd offers](01-from-rule-bots-to-v7.md#the-absurd-offers) in entry 1. There, a weakness
(accepting offers for nothing) was invisible inside the pool we tuned against; here, our bots had never met a patient
set collector and sold it the pieces. In both cases a player from outside that pool found the gap.

## 16. What it means

**Rock, paper, scissors, in a pool with donors.** The Bonjour bots beat our honest bots, our bots that send bad
offers beat the Bonjour bots, and our honest bots resist bad offers. No bot in such a pool is best against all the
others. Without donors the cycle disappears: the bots that send bad offers fall back to the level of their
originals, and "Resistant Bonjour C" leads ([a tournament without donors](#6-a-tournament-without-donors)); the later
windows variants lead the pools without donors as well ([section 13](#13-the-final-search-and-the-elo-ranking)).

**The pool decides the ranking.** The same "Bonjour C" is first of 12 in one pool and eighth of 16 in another;
"Hustler" is first of 16 with donors and seventh of 15 without them. A donor such as "Random" at a share of the
tables rewards exploitation more than good play. A tournament result means little without its pool, and a claim
about strength should come from a pool without donors. The same holds for which groups to hunt: the classical
Markov chain analyses of the board say which groups are worth owning, not which are worth chasing against rivals who
chase them too ([related work](../related-work.md#markov-chain-analyses-of-the-board)). Even among the leading bots,
"Resistant Bonjour B" won two elimination tournaments whose pools held several bots that hunt the groups of "Resistant
Bonjour C", and "Resistant Bonjour C" won the head-to-head in a pool without them
([section 10](#b-against-c)).

**"Strong bots", with a qualifier.** We have called our best bots "strong bots", and in one sense they are: they
cannot be exploited by bad offers. But they have a gap in trading: they judge each trade on its own and do not see a
sequence of individually fair trades that hands an opponent a set. A very simple fixed policy found that gap
([the plain result](#7-the-plain-result)). Repairing it in our own bot ("Wary Calculator") helped against the
Bonjour bots but did not make our bot stronger than the tactic.

**What worked and what did not.** Three changes made the tactic stronger or weaker in our measurements. Refusing bad
offers (the resistant variants) removed its weakness and helped even where nobody sends bad offers. Offers added where the
tactic was silent, in the trade windows of our game, when raising money for a debt and as counter-offers, made it
stronger still (the windows variants). Replacing its offers with the offers of one of our own bots made it much weaker,
and borrowing our auctions made no difference. Absurd offers pay only against players that accept them.

**What it means for a test pool.** A pool that tests a bot should contain both kinds of players: bots that send bad
offers, and bots that collect sets the way the Bonjour bots do. A bot that does well against only one kind has not shown
that it is strong.

### Who can still be exploited

Two kinds of exploitation appeared in this entry: accepting bad offers, and selling the pieces of a set one by one to
a player who collects them. What follows comes from our measurements and from the bots' rules:

| Player | Accepts bad offers | Sells a set piece by piece to a collector |
|---|---|---|
| "Random" | Yes, about half of all offers, at random | Yes, at random |
| "Bonjour A", "Bonjour B", "Bonjour C" | Yes: a property for pennies whenever its sets are unchanged and enough cash is left ([the mechanism](#the-mechanism)) | Yes: it accepts any trade that leaves its own sets intact while enough cash is left, and with less than $200 offers its lone properties itself |
| Our honest bots | No: the feed rule and a trade evaluation (the shared one, or the own one of "GPT-Style" and "Tactician") | Yes, the first pieces of a group |
| The absurd bots and "Hustler" | No: they answer offers exactly like their originals | Yes, like their originals |
| The resistant variants | No: every bad offer of the absurd bots and "Hustler" scores below -100 for the recipient by the shared evaluation, and these variants need a score above zero | Partly: the same feed rule and shared evaluation as our honest bots, plus their own sales of lone properties below $200 |
| The windows variants | No: the same answers as the resistant variants | Partly: as the resistant variants; their extra sales in debt follow the rules of "Wary Calculator", which does not sell pieces that bring a rival closer to a set unless it cannot avoid bankruptcy |
| "Wary Calculator" | No: it needs a positive score on both of its evaluations | Mostly no: it refuses a piece-by-piece offer or counters with a price that matches the value of the piece to the buyer |

The non-resistant Bonjour bots are the extreme case of both weaknesses. The resistant variants refuse bad offers but keep
the same gap in selling pieces as our honest bots, since they reuse the same safeguards
([why our bots sell the pieces](#the-mechanism)).

## 17. Limitations

- **Two of the three bots are built from a description.** Only "Bonjour A" rests on GNOME's code; "Bonjour B" and
  "Bonjour C" rest on the paper's description, and their low-priority rule is our own choice. They may differ from FP-B
  and FP-C, possibly by being stronger.
- **Different rules.** The Bonjour bots play our game, with full trade windows, auction steps and counter-offers
  that GNOME does not have, so their results here say little about the paper's game.
- **Order of play.** Moving first helps some bots (about 6 points for "Calculator"), but seats are random in our
  tournaments, so the effect averages out ([who moves first](../bots.md#who-moves-first)).
- **Optimistic intervals.** All intervals treat games as independent. Games played from the same seed, and games at
  the same table in a tournament, are not fully independent.
- **The mechanism is read from the rules.** We have not counted, game by game, which trades moved which properties;
  the explanation fits the rules and the results, but it is not a measurement.
- **Donors in four of the five tournaments of sections 4 to 6.** All of them except the last include "Random" and the
  original Bonjour bots, which inflate the results of the bots that send bad offers ([the role of "Random"](#the-role-of-random)).
  The tournament without donors removes only the acceptance of bad offers: its pool still contains players that sell
  the pieces of a set, so "Resistant Bonjour C" still profits from that gap there.
- **One tournament per pool.** Each tournament result comes from a single 3,000-game run, and the gains of the
  resistant variants over the originals are close to the noise in two of the three cases. Comparisons between
  tournaments also compare different pools.
- **Elimination tournaments are noisy in their early rounds.** A bot with 200 or 600 games per round can be cut by
  chance, and a candidate is compared with the others by its win rate against a field that contains copies of the
  candidates themselves. Every comparison between the top bots (B against C, "Bonjour Mixed Auctions" against both) is a
  single run with a gap of 3 points or less.
- **The windows variants are measured as a package.** We have not separated the effect of the offers in windows, the sales
  in debt and the counter-offers, nor counted which trades produced the gain.
- **The Elo ranking is rough.** Most of its games come from one 12,000-game tournament plus a few short ones, the weak
  bots appear only as sparring partners, and the error we give for it is a rough estimate of ours, not a computed
  interval.

## 18. What comes next

Questions this entry leaves open:

- **Which part of the windows variants matters.** Offers in windows, sales in debt and counter-offers were added together;
  adding each one alone would show which carries the gain.
- **Which trades decide the games.** We still read the mechanism from the rules. Counting, game by game, the trades by which
  the Bonjour bots, the wary "Calculator" and the windows variants gather their sets would turn the explanation into a
  measurement.
- **Whether variability changes the picture.** All results here use the fixed rules of the Bonjour bots. Whether their
  strength survives non-zero trade variability and random moves is not measured.
- **B against C.** The two are about equally strong, and the pool decides which is ahead. Runs with a controlled number of
  bots that hunt each group would test that.
