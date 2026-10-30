# The rule-based bots

Our bots are rule-based AI: each plays by a fixed set of hand-written rules and writes a short justification for
every decision, visible in replays. They serve three purposes: **opponents** during training, **teachers** to imitate,
and **yardsticks** for our models. The history of how they were built and tuned is in
[the journal](journal/01-from-rule-bots-to-v7.md#3-the-rule-based-bots).

| Bot | In one sentence | Wins in mixed tables |
|---|---|---|
| ["Calculator"](#calculator) | Computes every decision from board probabilities (a Markov chain); our main reference and the best of the original eight in mixed tables | 34.4% |
| ["Tactician"](#tactician) | "GPT-Style" with two fixes in trade evaluation | 32.2% |
| ["GPT-Style"](#gpt-style) | A 12-point strategy the GPT-6.1-Sol language model wrote about its own play | 30.6% |
| ["Investor"](#investor) | Buys everything it can afford, builds up to hotels, bids up to 120% of the price | 26.3% |
| ["Master"](#master) | Follows the published advice of world champions and board analyses | 24.4% |
| ["Collector"](#collector) | Hunts orange and red, and properties that complete a set, even overpaying | 21.5% |
| ["Cautious"](#cautious) | Keeps $300 in reserve, at most 3 houses, bids up to 90% | 17.1% |
| ["Miser"](#miser) | Keeps $400-500 in reserve, at most 3 houses, bids at most 80% | 12.2% |
| ["Simulator" and "Fast Simulator"](#simulator-and-fast-simulator) | Play out short imaginary continuations before important decisions | "Fast Simulator" 31.3% (a) |
| ["Absurd GPT-Style", "Absurd Calculator", "Absurd Investor"](#absurd-bots) | The originals plus one deliberately bad offer per turn | not measured |
| ["Hustler"](#hustler) | Plays like "Calculator" and tries up to twice per turn to get something for nothing | not measured |
| ["Bonjour A", "Bonjour B", "Bonjour C"](#bonjour-a-b-c) | Our bots built on the tactics of the fixed-policy opponents FP-A, FP-B and FP-C of Bonjour et al. ([how they came about](#bonjour-a-b-c)); they collect sets by trade | C 51.1%, B 35.4%, A 19.5% (c) |
| ["Resistant Bonjour A", "Resistant Bonjour B", "Resistant Bonjour C"](#resistant-bonjour-a-b-c) | The Bonjour bots that accept an offer only if it also passes our bots' safeguards | C 43.1%, B 35.2%, A 16.7% (d) |
| ["Wary Calculator"](#wary-calculator) | Plays like "Calculator" but does not sell or give away properties that bring a rival closer to a set | not measured (e) |
| ["Bonjour Calculator", "Bonjour Calculator Pure", "Bonjour Calculator Auctions", "Bonjour Mixed Auctions"](#bonjour-hybrids-with-calculator) | "Resistant Bonjour C" with the offers and auctions, or only the auctions, of "Calculator" | not measured (e) |
| ["Resistant Bonjour B + Windows", "Resistant Bonjour C + Windows"](#resistant-bonjour-b-and-c-with-windows) | The resistant variants that also make offers in trade windows, when paying off a debt and as counter-offers | not measured (e) |
| ["Absurd Wary Calculator", "Absurd Tactician", "Absurd Resistant Bonjour B", "Absurd Resistant Bonjour C", "Absurd Bonjour Mixed Auctions"](#absurd-versions-of-the-leading-bots) | The leading bots plus one deliberately bad offer per turn | not measured (e) |
| ["Random"](#random) | Picks a random legal move and never proposes trades | 0% (b) |

"Wins in mixed tables": 1,000 games with 4 bots drawn at random from the 8 bots above "Simulator", with characters
restored after tuning ([tuning](journal/01-from-rule-bots-to-v7.md#tuning-and-why-we-undid-half-of-it)). With
four equal players, chance level is 25%.

(a) A different setup: a 3,000-game tournament with a pool of 20 players (15 of them bots), in which "GPT-Style" won 32.5%.

(b) The first 600-game tournament, with four-player tables drawn at random from six bots ("Investor", "Collector",
"Master", "Cautious", "Miser" and "Random"; "Calculator" did not exist yet).

(c) A different setup: 1,000 games with 4 bots drawn at random from the same 8 bots and the three Bonjour bots, with
full trade windows; "Calculator" won 26.9% there ([entry 2](journal/02-the-bonjour-bots.md#mixed-tables)).

(d) A different setup: a 3,000-game tournament without donors, players that accept bad offers: a pool of 15 (our 8
honest bots without "Random", the absurd bots, "Hustler" and the three resistant variants), with full trade windows;
"Calculator" won 30.7% there ([entry 2](journal/02-the-bonjour-bots.md#6-a-tournament-without-donors)).

(e) These bots were built after the mixed-table measurement and were not part of it; their results come from the
tournaments in their sections and from the [Elo ranking](#current-elo-ranking).

In our main protocol (1,320 games; a model plus three copies of one bot on shared seeds), "Calculator" wins 35.7%; see
[how we measure](journal/01-from-rule-bots-to-v7.md#1-the-question-and-how-we-measure). "Calculator" is the strongest
of the original eight bots in mixed tables, but it is not the strongest bot we have. The
[Bonjour bots](#bonjour-a-b-c), which we implemented ourselves from tactics described in earlier work, beat it and our
other honest bots in tournaments by exploiting a gap in how our bots trade, and their
[resistant variants](#resistant-bonjour-a-b-c) stay ahead of our honest bots even in a tournament without any player
that accepts bad offers. In the current [Elo ranking](#current-elo-ranking) the two variants with trade windows lead
by a wide margin. The story is in [entry 2](journal/02-the-bonjour-bots.md).

## Current Elo ranking

The ranking below is a Bradley-Terry fit on the Elo scale ([glossary](glossary.md#how-we-measure)) over the games of
the tournaments that were recorded to the rating, taken from the server at the time of writing. Every game counts as a
set of pairwise results ("who finished ahead"), 1500 is the average of all rated players, and every player gets 5
virtual wins and 5 virtual losses against a 1500 player. Players that are not bots are left out of the table. Most of
the games come from one 12,000-game elimination tournament with 7 candidates and 27 sparring partners
([entry 2](journal/02-the-bonjour-bots.md#13-the-final-search-and-the-elo-ranking): four-player tables with one
candidate and three sparring partners drawn from the pool, full trade windows, the 200-round limit), plus a few shorter
tournaments in which the leading bots played each other. The weakest bots therefore appear only as sparring partners,
with about 1,300 games each. "Random" and the non-resistant [Bonjour bots](#bonjour-a-b-c) played only in earlier
tournaments that were not recorded to the rating, so they have no Elo here.

| Place | Bot | Elo | Games |
|---|---|---|---|
| 1 | ["Resistant Bonjour C + Windows"](#resistant-bonjour-b-and-c-with-windows) | 1569 | 5,383 |
| 2 | ["Resistant Bonjour B + Windows"](#resistant-bonjour-b-and-c-with-windows) | 1557 | 5,407 |
| 3 | ["Absurd Resistant Bonjour C"](#absurd-versions-of-the-leading-bots) | 1528 | 1,660 |
| 4 | ["Absurd Bonjour Mixed Auctions"](#absurd-versions-of-the-leading-bots) | 1522 | 1,490 |
| 5 | ["Wary Calculator"](#wary-calculator) | 1521 | 3,377 |
| 6 | ["Absurd Resistant Bonjour B"](#absurd-versions-of-the-leading-bots) | 1517 | 1,740 |
| 7 | ["Absurd Tactician"](#absurd-versions-of-the-leading-bots) | 1516 | 1,753 |
| 8 | ["Resistant Bonjour C"](#resistant-bonjour-a-b-c) | 1514 | 2,159 |
| 8 | ["Bonjour Mixed Auctions"](#bonjour-hybrids-with-calculator) | 1514 | 3,346 |
| 10 | ["Absurd Wary Calculator"](#absurd-versions-of-the-leading-bots) | 1511 | 1,538 |
| 11 | ["Tactician"](#tactician) | 1508 | 2,156 |
| 12 | ["Bonjour Calculator Auctions"](#bonjour-hybrids-with-calculator) | 1508 | 1,553 |
| 13 | ["GPT-Style"](#gpt-style) | 1507 | 1,549 |
| 14 | ["Resistant Bonjour B"](#resistant-bonjour-a-b-c) | 1501 | 1,924 |
| 15 | ["Absurd GPT-Style"](#absurd-bots) | 1495 | 1,344 |
| 16 | ["Calculator"](#calculator) | 1495 | 1,405 |
| 17 | ["Hustler"](#hustler) | 1481 | 1,321 |
| 18 | ["Absurd Calculator"](#absurd-bots) | 1474 | 1,365 |
| 19 | ["Resistant Bonjour A"](#resistant-bonjour-a-b-c) | 1458 | 1,335 |
| 20 | ["Master"](#master) | 1451 | 1,322 |
| 21 | ["Bonjour Calculator"](#bonjour-hybrids-with-calculator) | 1444 | 1,360 |
| 22 | ["Collector"](#collector) | 1442 | 1,269 |
| 23 | ["Bonjour Calculator Pure"](#bonjour-hybrids-with-calculator) | 1442 | 1,331 |
| 24 | ["Cautious"](#cautious) | 1440 | 1,321 |
| 25 | ["Investor"](#investor) | 1440 | 1,279 |
| 26 | ["Miser"](#miser) | 1435 | 1,366 |
| 27 | ["Absurd Investor"](#absurd-bots) | 1401 | 1,301 |

How to read it:

- The two variants with trade windows are 29 and 41 points above the next bot, and "Resistant Bonjour C + Windows" is
  74 points above "Calculator". The differences between neighbors lower in the table are small. As a rough estimate of
  ours (not a computed interval), a rating based on 1,300-1,800 games carries about +/-15 points and one based on 5,400
  games about +/-8, so places that are a few points apart cannot be told apart.
- Our honest bots sit in the middle: "Tactician" (1508) and "GPT-Style" (1507) are slightly above "Calculator" (1495),
  although "Calculator" is ahead of them in the mixed tables of its section. "Wary Calculator" (1521) is the highest of
  the bots we designed ourselves.
- The absurd versions of the leading bots are not clearly above their originals here, because these pools have no
  player that accepts bad offers ([Absurd versions of the leading bots](#absurd-versions-of-the-leading-bots)).
- The ranking changes with every recorded tournament. The numbers in the sections below come from the specific
  tournaments they name.

## Who moves first

Does the order of play matter? Four identical copies of one bot at the table, standard rules with full trade windows,
the 200-round limit, 4,000 games per bot, and the win rate by the actual order of play (who moved first, second, third
and fourth, which the starting roll decides, not the seat number). Wilson 95% intervals.

| Bot | Moves 1st | Moves 2nd | Moves 3rd | Moves 4th |
|---|---|---|---|---|
| "Calculator" | 27.9% (26.5-29.3) | 26.2% (24.9-27.6) | 24.4% (23.1-25.7) | 21.5% (20.3-22.8) |
| "Resistant Bonjour C" | 25.5% (24.2-26.9) | 26.3% (24.9-27.7) | 25.4% (24.0-26.7) | 22.9% (21.6-24.2) |
| "Resistant Bonjour C + Windows" | 23.8% (22.5-25.1) | 27.0% (25.6-28.4) | 25.3% (24.0-26.6) | 23.9% (22.6-25.3) |

For "Calculator" the advantage of moving first is real, about 6 points between first and last. For the two Bonjour
variants it is small or absent: "Resistant Bonjour C" loses about 3 points moving last, and the windows variant shows
no consistent order effect. An earlier measurement (not stored) gave roughly 29.9% against 21.9% for "Calculator"
and 26.1% against 23.4% for "Resistant Bonjour C"; the direction and the size of the gap agree, with a smaller first
place for "Calculator" now. Seat order is random in our tournaments, so the effect averages out there.

## Calculator

**Our main reference: it has no favorite colors and computes every decision from board probabilities. The best of the
original eight bots in mixed tables.**

### How it plays

"Calculator" computes how often each square is visited with a Markov chain over two dice, movement cards and the
square that sends a player to jail. The result matches known analyses: jail is the most visited square (about 6% including
just visiting), followed by square 24, the third street of the red group on the third side of the board (3.2%), "Start", the
oranges and the railroads. On top of that:

- a house is worth the increase in expected rent from all opponents per dollar spent; it builds where that return is
  highest, including hotels, but does not convert houses to a hotel during a shortage when a rival has an unbuilt set;
- its cash reserve is not fixed: expected rent on the next roll plus a quarter of the worst case;
- it mortgages a loose property to build a house if the house earns more than twice what the property loses;
- auction value is price x the square root of relative visit frequency, x2 for completing a set, x1.5 for blocking;
- it proposes first the trades after which a set earns the most with three houses.

Like all bots it uses the shared safeguards added during tuning (no feeding rivals, big auction steps, a cash buffer
after building, a debtor rescue; see [tuning](journal/01-from-rule-bots-to-v7.md#tuning-and-why-we-undid-half-of-it)).
Its trade variability is 0.1 ([variance parameters](#variance-parameters)).

### Measured strength

- First tests (1,000 games each): 46.3% in four-player tables drawn at random from the seven bots that existed then
  ("Random", "Investor", "Collector", "Miser", "Cautious", "Master" and "Calculator"; "Investor" 37.0% in the same
  tables), 64.2% one-on-one with
  "Investor", 29.5% against three "Investors".
- Mixed tables after tuning (1,000 games, 4 bots drawn at random from 8): 31.9%; with characters restored **34.4%** (mean
  place 2.34), the best of the pool.
- Against the newer bots it is not first: it wins only 12.3% against three "Bonjour C" bots (400 games; see
  ["Wary Calculator"](#wary-calculator)), and in the [Elo ranking](#current-elo-ranking) it sits slightly below
  "Tactician" and "GPT-Style".
- Main protocol (1,320 games; model or bot plus three copies of one bot): **35.7%**, and it beats three
  "Random" bots in 120 of 120 games. A single-bot value rests on 120 games and carries about +/-8 points, so we do
  not rank its opponents by these values: against three copies of itself it should win 25% in expectation, by
  definition, and it won 22.5%, which shows the scale of this noise. Its mean over 11 bots includes that game against
  itself and two nearly identical ones ([how we measure](journal/01-from-rule-bots-to-v7.md#1-the-question-and-how-we-measure)).

### Role in the project

It is the yardstick every model is compared with, and the teacher for auctions and trades in most of our training
runs (the Loki students and njord; it was also one of Mjolnir's teachers). In the hybrid-player diagnosis it plays every area except the one the model
takes over, which showed that trades are the model's weakest area
([which part of the game loses?](journal/01-from-rule-bots-to-v7.md#which-part-of-the-game-loses)). "Hustler" and
"Absurd Calculator" are built on it.

## Tactician

**"GPT-Style" with two fixes in trade evaluation: it requires a gain for itself and never sells a property too
cheaply.**

### Where it came from

Tournaments with the [absurd bots](#absurd-bots) exposed two flaws in ["GPT-Style"](#gpt-style)'s trade evaluation:

- It accepted trades where both sides lose as long as the rival loses more (it gave away a utility for $30 at an
  evaluation of "I gain -120, the rival -147"), which in a four-player game helps the two other players.
- Its valuation treated a single property outside any group as almost worthless, so it would sell a street for $20.

### How it plays

Exactly like "GPT-Style", except that it accepts a trade only when it gains from it itself, and never sells an
unmortgaged property below its mortgage value.

### Measured strength

- Mixed tables (1,000 games, 4 bots drawn at random from 8): 28.8% after tuning, **32.2%** with characters restored (mean
  place 2.26), the second-strongest bot of the original eight.
- Main protocol (1,320 games): "Calculator" wins 26.7% against three "Tacticians".

### Role in the project

It was the trade teacher in our first attempts to teach a model to compose offers (small-32x32-trade-teacher with a
live teacher inside PPO, small-32x32-trade-dagger with DAgger rounds), and the teacher in the fit test, in which
networks of four sizes reached the same loss against it
([composing trades is hard](journal/01-from-rule-bots-to-v7.md#composing-trades-is-hard)).

## GPT-Style

**A 12-point strategy that the GPT-6.1-Sol language model wrote about its own play, turned into a rule-based bot.**

### Where it came from

Once our MCP server existed, a few games were played between our bots and the **GPT-6.1-Sol** language model,
connected through the MCP server as a regular player. We then asked GPT-6.1-Sol itself to **describe in text how it
plays**. It produced a 12-point tactics specification, and **Claude Opus implemented that description as a new
rule-based bot**.

### The twelve points

1. Safe execution: any logic error ends in a legal fallback move, marked in the justification.
2. Liquidity: track cash and quick liquidation (buildings at half price, mortgages) separately.
3. Color choice: buy the cheap brown and light-blue groups early even with a low reserve; bid higher on pink and
   orange; buy red, yellow, green and dark blue only with extra cash.
4. Property valuation: up to 2.5x price to complete a group, 1.8x to block a rival who has money to build, 1.5x
   for a group two rivals could complete with one trade.
5. Negotiate the missing street: offer 1.2x the price in cash, raising after each refusal (1.5x, 1.8x, 2.2x, 2.6x,
   3x), always keeping a reserve and money for houses, never repeating an identical offer.
6. Build order: the best color (largest rent increase per dollar) to three houses first, then compare a fourth
   house, a hotel and a second color; prefer properties a few squares ahead of rivals.
7. Reserve: about $250 with no rival buildings, growing with the board's danger (expected rent over the next three
   moves plus 30% of the worst case on the next one, up to $900).
8. Blockades: buy a rival's last missing street at a minimal reserve; mortgage single properties to fund houses
   (mortgaged properties still block).
9. Trade evaluation: compare both portfolios after the trade (new sets valued at three-house income within budget,
   cash, mortgages taken over).
10. Auctions: fix a limit once, before bidding, and never raise it.
11. Debt: sell single properties first, then the buildings that lose the least rent per dollar; going from three to
    two houses carries a triple penalty.
12. Jail and endgame: leave early, stay in when owning buildings on a dangerous board; with one rival left, redeem
    mortgages so there are more rent-paying squares.

### Measured strength

It became the strongest bot of its time:

| Test (1,000 games each) | "GPT-Style" wins | Other players' wins |
|---|---|---|
| Four-bot tables drawn at random from all bots | **45.8%** | "Calculator" 38.6%, "Investor" 32.8%, "Master" 25.5% |
| One-on-one with "Calculator" | 52.5% | 47.5% (within noise, about +/-3 points, 95% CI) |
| One-on-one with "Investor" | 66.5% | 33.5% |
| One-on-one with "Master" | 70.5% | 29.5% |
| Against three "Calculators" | 26.6% | 24.5% each |

Quality checks over 400 games against three "Calculators": zero fallback moves, zero offers rejected by the engine
as illegal, 19% of its trade proposals accepted (107 of 557), a third house somewhere in 66% of games.

Under rules version 3 it briefly moved to the top of 500-game mixed tables (36%, "Calculator" 33%). After the later
retuning it settled around 30-31% in mixed tables (30.6%, mean place 2.32), just behind "Calculator". In the
main protocol (1,320 games) "Calculator" wins 25.0% against three "GPT-Style" bots and mini-evo5 21.7%, its lowest
of 11 single-bot values. Each of these values rests on 120 games and carries about +/-8 points, and the lowest of 11
noisy values tends to be low by chance, so they do not show that "GPT-Style" is a harder opponent than the others
([how we measure](journal/01-from-rule-bots-to-v7.md#1-the-question-and-how-we-measure)).

### Role in the project

It was the first bot our first models imitated (simple-small-32x32, small-32x32). Tournaments with the absurd bots
exposed two flaws in its trade evaluation, which led to ["Tactician"](#tactician); "Absurd GPT-Style" is built on it
([absurd bots](#absurd-bots)).

## Investor

**An aggressive spender: it buys everything it can afford, builds up to hotels and bids up to 120% of the price.**

### How it plays

"Investor" was one of the four personalities planned in the design document as distinct styles. It buys every
property it can afford, builds up to hotels and bids in auctions up to 120% of the printed price. Its trade
variability is 0.3 (sometimes a different trade candidate, cash amounts +/-6%, a fuzzier acceptance threshold).

### Measured strength

- First 600-game tournament with four-player tables drawn at random: **41.3%**, the best of the first personalities.
  Spending aggressively paid off; hoarding cash did not.
- Four-player tables drawn at random from the seven bots that existed then (1,000 games, a mixed table, not a duel
  with "Calculator"): 37.0%, second to "Calculator"'s 46.3% (see ["Calculator"](#calculator)).
- One-on-one (1,000 games each): 35.8% against "Calculator"; "GPT-Style" beat it 66.5% to 33.5%.
- Mixed tables (1,000 games, 4 bots drawn at random from 8): 26.1% after tuning, **26.3%** with characters restored (mean
  place 2.57).
- Of 42,705 offers that our count classed as offers for nothing, it accepted 347. Most of them were one cheap but
  fair deal that this broad count had included, the jail release card for $25 when "Investor" values it at $50; we
  did not check the rest one by one ([the absurd offers](journal/01-from-rule-bots-to-v7.md#the-absurd-offers)).

"Absurd Investor" is built on it ([absurd bots](#absurd-bots)).

## Master

**A bot built from the published advice of Monopoly world champions and board analyses, each rule with its source.**

### How it plays

- Buy almost everything early (properties are trading chips). *Source: Bjørn Halvard Knappskog, a world champion.*
- Complete a buildable set at any price. *Source: Knappskog.*
- Value orange highest (it lies 6, 8 and 9 squares after jail, the most visited square). *Sources: Jason Bunn, a world
  champion; Markov-chain analyses of the board.*
- Avoid the most expensive dark-blue streets. *Source: Natalie Fitzsimons, UK and Ireland champion.*
- Build three houses everywhere before a fourth and **no hotels**, to hold houses while the bank has only 32.
  *Sources: Fitzsimons; analyses showing the third house as the point where building pays off.*
- Value railroads. *Source: an analysis by the University of Leicester (without houses, railroads give the best
  return).*
- Leave jail early, but sit in it late in the game. *Source: common advice.*
- **House blockade** (added when characters were restored): when 10 or fewer houses remain in the bank and a rival
  can build, it buys houses beyond its normal need and builds no hotels while a rival needs houses. *Source: the
  classic "housing shortage" tactic of champions.*

The sources are named as we recorded them when building the bot; we did not re-check each one for these
write-ups.

### Does champions' advice work?

We tested the advice by changing one rule at a time (800 games each, against bots drawn from "Investor",
"Collector", "Cautious" and "Miser"):

| Variant | Wins |
|---|---|
| "Master" as advised | 24.1% |
| + builds hotels | 26.6% |
| + always leaves jail | 21.9% |
| + buys dark blue | 28.2% |

With 800 games per variant, the difference between two variants carries about +/-4.3 points (95% CI), and three
one-rule variants were compared, which raises the chance that one of them looks better by luck. The largest change, **buying the dark blues (+4.1 points)**, is the only change near the
edge of noise, so avoiding them may hurt in our bot pool. The advice comes from
single-square analyses, while in a game with trades the dark blues are also a commodity and a blockade. Building
hotels (+2.5) and always leaving jail (-2.2) stay within noise: they only hint that refusing hotels costs something
(bots build so fast that the bank runs out of houses anyway) and that sitting in jail late helps.

Tuning for strength made "Master" build hotels and buy dark blue, which erased its character. We restored the advice
and allowed tuning only within the character
([tuning](journal/01-from-rule-bots-to-v7.md#tuning-and-why-we-undid-half-of-it)). That makes it a useful test: can a
model that imitates it learn to drop its weak rules?

### Measured strength

- First 600-game tournament with four-player tables drawn at random: 32.8%.
- Four-bot tables drawn at random from all bots (1,000 games): 25.5%; one-on-one with "GPT-Style": 29.5%.
- Mixed tables (1,000 games, 4 bots drawn at random from 8): 25.6% after tuning, **24.4%** with characters restored (mean
  place 2.52). Restoring its character (no hotels, avoiding dark blue, the house blockade) cost it about 1 point in
  these tables, within noise (about 500 games per bot; about +/-4.4-5 points, 95% CI, for the difference); note that this pool (all 8 bots) differs from the one-rule tests above.

## Collector

**A color hunter: it chases orange and red, and any property that completes a set, even at a high price.**

### How it plays

"Collector" was one of the four personalities planned in the design document. It hunts the orange and red groups,
bidding up to 160% of the price for them (even mortgaging to pay), and goes after properties that complete a set. Its
trade variability is 0.3.

Tuning for strength made it stop overpaying for its colors, which erased its character; we restored the overpaying
and allowed tuning only within the character
([tuning](journal/01-from-rule-bots-to-v7.md#tuning-and-why-we-undid-half-of-it)).

### Measured strength

- First 600-game tournament with four-player tables drawn at random: 36.2%.
- Mixed tables (1,000 games, 4 bots drawn at random from 8): 20.3% after tuning, **21.5%** with characters restored (mean
  place 2.61).

## Cautious

**A careful player: it keeps a $300 reserve, builds at most three houses and bids up to 90% of the price.**

### How it plays

"Cautious" was one of the four personalities planned in the design document. It keeps $300 in reserve, builds at
most 3 houses on a property and bids in auctions up to 90% of the printed price.

Tuning for strength made it build like everyone else; we restored its character and allowed tuning only within it
([tuning](journal/01-from-rule-bots-to-v7.md#tuning-and-why-we-undid-half-of-it)). The cautious character cost
it about 4 points (20.9% after tuning, 17.1% restored), which is within noise: each bot played about 500 games, so
the difference carries about +/-4.4-5 points (95% CI).

### Measured strength

- First 600-game tournament with four-player tables drawn at random: 25.4%.
- Mixed tables (1,000 games, 4 bots drawn at random from 8): 20.9% after tuning, **17.1%** with characters restored (mean
  place 2.66).

## Miser

**A cash hoarder: it keeps large reserves, builds at most three houses and bids at most 80% of the price.**

### How it plays

"Miser" was one of the four personalities planned in the design document. It keeps $400 after buying and $500 after
building, builds at most 3 houses on a property and bids in auctions at most 80% of the printed price.

Tuning for strength made it build like everyone else; we restored its cautious character and allowed tuning only
within it ([tuning](journal/01-from-rule-bots-to-v7.md#tuning-and-why-we-undid-half-of-it)). The cautious
character cost it about 3 points (15.0% after tuning, 12.2% restored), which is within noise: each bot played about
500 games, so the difference carries about +/-4.4-5 points (95% CI).

### Measured strength

- First 600-game tournament with four-player tables drawn at random: 13.7%.
- Mixed tables (1,000 games, 4 bots drawn at random from 8): 15.0% after tuning, **12.2%** with characters restored (mean
  place 2.75), the weakest of the pool apart from "Random".
- mini-evo5 wins 46.7% against three "Misers" in the main protocol, its highest single-bot value (120 games, about
  +/-8 points).

## Simulator and Fast Simulator

**Bots that measure instead of guessing: before important decisions they play out short imaginary continuations of
the game.**

### Simulator

All other bots guess the value of a decision with rules; "Simulator" measures it. For important decisions (buy or
not, answer a trade, which of the top three trades to propose, leave jail) it copies the game and plays 12 short
continuations of 25 rounds in which everyone plays as "Calculator", scoring 1 for a win, 0 for bankruptcy and the
share of net worth otherwise. Every option gets the same dice, and card decks are reshuffled in the imaginary games
so the bot cannot peek. A decision takes about 1 second and a game 15-20 seconds instead of a fraction of a second.

### Fast Simulator

"Fast Simulator" races the options in batches of 4 continuations and drops an option once it is clearly behind
(paired difference plus two standard errors below zero). Obvious decisions (a trade evaluation beyond +/-300, or a
purchase that leaves at least $600) are made instantly like "Calculator". It is about twice as fast as "Simulator".

### Measured strength

In a 3,000-game tournament with a pool of 20 players (15 of them bots) "Fast Simulator" won 31.3% ("GPT-Style" 32.5%). Both bots are too slow
to serve as training opponents and are used in tournaments; full strength numbers for "Simulator" alone are missing.

## Absurd bots

**The original bots plus one deliberately bad offer per turn, so that models see such offers in training and learn
to refuse them.**

### Where they came from

They exist because one of our models discovered offers for nothing and other models could not refuse them. The full
story is in the journal: [the absurd offers](journal/01-from-rule-bots-to-v7.md#the-absurd-offers).

### How they play

"Absurd GPT-Style", "Absurd Calculator" and "Absurd Investor" play exactly like ["GPT-Style"](#gpt-style),
["Calculator"](#calculator) and ["Investor"](#investor), but once per turn send an offer that is clearly
unprofitable for the recipient by the shared bot evaluation (below -100):

- cash for nothing ("give me $300");
- a property for pennies;
- the jail release card for nothing;
- a mortgaged junk property for a fortune.

On offers from others they answer like the originals, so they cannot be exploited themselves. They are mixed into
training opponents as a robustness test.

### Measured strength

In a 20,000-game tournament with a pool of 10, "Absurd GPT-Style" won 22.4% and "Absurd Calculator" 21.5%, close to
"GPT-Style" (24.2%) and "Calculator" (23.0%) in the same tournament. Tournaments with them also exposed two flaws in
"GPT-Style"'s trade evaluation, which led to ["Tactician"](#tactician).

How much they win depends on whether the pool contains players that accept bad offers. With "Random" and the non-resistant
[Bonjour bots](#bonjour-a-b-c) in the pool they lead the tournament; without them they play about like their
originals ("Absurd Calculator" 27.9% against "Calculator"'s 30.7%, "Absurd GPT-Style" 25.5% against 28.0%, "Absurd
Investor" 17.8% against 22.1%; [a tournament without donors](journal/02-the-bonjour-bots.md#6-a-tournament-without-donors)).

### Absurd versions of the leading bots

Five more absurd versions are built the same way on the bots that were leading at the time: "Absurd Wary Calculator",
"Absurd Tactician", "Absurd Resistant Bonjour B", "Absurd Resistant Bonjour C" and "Absurd Bonjour Mixed Auctions". Their
originals are ["Wary Calculator"](#wary-calculator), ["Tactician"](#tactician), ["Resistant Bonjour B" and "Resistant
Bonjour C"](#resistant-bonjour-a-b-c) and ["Bonjour Mixed Auctions"](#bonjour-hybrids-with-calculator). Each sends one
absurd offer per turn after the roll and one in a trade window; everything else is the original: answers to other
players' offers, counter-offers to its own absurd offer, auctions, building, parameters and variability.

The Bonjour bots needed one adaptation. Their only chance to build is in the trade windows of other players' turns,
where they build, lift mortgages and then pass, and a proposal uses up the player's place in the window. So the absurd
version lets the original manage its assets first and sends the absurd offer only instead of the pass. For the Bonjour
bots the absurd offers are the only thing they propose after the roll and in windows.

Measured: each absurd bot against three copies of its original (200 games, seats rotating, full trade windows, 200-round
limit; the neutral result is 25%):

| Bot | Wins | 95% CI | Absurd offers per game | Accepted by the originals |
|---|---|---|---|---|
| "Absurd Resistant Bonjour B" | 22.0% (44/200) | 16.8-28.2% | 157 | 0 |
| "Absurd Resistant Bonjour C" | 23.5% (47/200) | 18.2-29.8% | 125 | 0 |
| "Absurd Bonjour Mixed Auctions" | 23.0% (46/200) | 17.7-29.3% | 122 | 0 |
| "Absurd Wary Calculator" | 14.5% (29/200) | 10.3-20.0% | 148 | 0 |
| "Absurd Tactician" | 20.0% (40/200) | 15.0-26.1% | 174 | 70 (0.2%) |

The offers flow, but the leading bots almost never accept them; the 0.2% of "Tactician" is mostly the sale of a
mortgaged property for which the bank would give nothing. The resistant Bonjour bots reject the offers without a
counter-offer, so their absurd versions stay within the neutral 25%. "Absurd Wary Calculator" clearly loses: its original
answers absurd offers with a counter-offer (about 270 per game in 40 control games) and the absurd version accepts about
one such counter-offer per game, a trade that would not have happened without the absurd offer. The order of moves in
the window is not the cause: sending the offer instead of the pass, as the Bonjour bots do, gave 16.0% (11.6-21.7%),
and no offers in windows at all gave 19.5% (14.6-25.5%). "Tactician" also counters (about 195 per game), but its absurd
version accepted none of the counters.

In the largest recorded tournament, whose pool has no player that accepts bad offers
([Elo ranking](#current-elo-ranking)), the absurd versions and their originals both played as sparring partners (1,270-1,400 games each):

| Original | Absurd version wins | Original wins |
|---|---|---|
| "Resistant Bonjour C" | 31.1% | 27.8% |
| "Bonjour Mixed Auctions" | 31.1% | 28.2% |
| "Resistant Bonjour B" | 25.8% | 24.6% |
| "Wary Calculator" | 25.1% | 25.6% |
| "Tactician" | 22.5% | 21.7% |

The differences are within noise (about +/-3.5 points for a difference): no clear gain without donors, as in
[a tournament without donors](journal/02-the-bonjour-bots.md#6-a-tournament-without-donors).

See also ["Hustler"](#hustler), which tries harder.

## Hustler

**Plays like "Calculator" and tries up to twice per turn to extract something for nothing.**

### Where it came from

Like the [absurd bots](#absurd-bots), "Hustler" exists because one of our models discovered offers for nothing and
other models could not refuse them. The full story is in the journal:
[the absurd offers](journal/01-from-rule-bots-to-v7.md#the-absurd-offers).

### How it plays

It plays every area of the game like ["Calculator"](#calculator). Up to twice per turn it sends an offer that is
clearly unprofitable for the recipient by the shared bot evaluation (below -100): cash for nothing, a property for
pennies, the jail card for nothing, or a mortgaged junk property for a fortune. On offers from others it answers like
"Calculator", so it cannot be exploited itself. It is mixed into training opponents as a robustness test.

### Measured strength

It is not weak: in the main protocol (1,320 games) "Calculator" wins 28.3% against three "Hustlers", close to its 26.7%
against three "Tacticians". Loki gen2f, trained against its own earlier versions, reached 36.7% against three
"Hustlers" and only 6.7% against three "Calculators", but those come from our old protocol (330 games; 30 games per
bot, about +/-16 points on a single bot), so they are only a hint
([how we measure](journal/01-from-rule-bots-to-v7.md#1-the-question-and-how-we-measure)).

In tournaments its result depends on the pool: it won 46.3% in a pool of 19 with "Random" and the non-resistant Bonjour
bots to exploit, and 27.0% in the same pool without them, about as much as "Calculator" (30.7%) there
([a tournament without donors](journal/02-the-bonjour-bots.md#6-a-tournament-without-donors)).

## Bonjour A, B, C

**Our bots built on the tactics of FP-A, FP-B and FP-C, the fixed-policy opponents of the hybrid RL work of Bonjour
et al.: simple rules that collect complete sets by trade, and the first bots whose tactics we did not design
ourselves.**

### Where they came from

The paper of Bonjour et al. evaluated its PPO players against three fixed-policy opponents
([related work](related-work.md#bonjour-haliem-et-al-hybrid-deep-rl-2021-2022)). We took the tactics described by
the authors, from the paper and from the rule-based player of the GNOME simulator, and implemented them ourselves
for our version of the rules, writing the code from scratch since the GNOME repository carries no license
([the GNOME simulator](related-work.md#the-gnome-simulator)). The implementation and its adaptation to our game are
ours, so these are not the authors' agents and the results below concern our implementation in our game. "Bonjour A"
follows GNOME's player most closely; "Bonjour B" and "Bonjour C" rest on the paper's description. The full story is in
[entry 2](journal/02-the-bonjour-bots.md).

### How they play

All three put complete sets first and use one cash threshold, $200. They buy when $200 is left after the purchase,
and a property that completes a set whenever they can afford it. They build with no reserve. In their own turn,
before rolling, they offer to buy rivals' lone properties that complete their sets for 1.125x the price, or swap
properties with cash on top; they answer offers by counting sets: they accept any trade that gains them a set if
they can pay, and, with their sets unchanged and enough cash, accept even a losing trade. They never counter-offer
and never trade jail release cards. They differ in priorities:

- "Bonjour A": all properties equal;
- "Bonjour B": high priority on the railroads and the dark blue group, low priority on the utilities;
- "Bonjour C": high priority on the railroads and the orange and light blue groups.

A high-priority property is bought whenever affordable, bid for aggressively, wanted from rivals even if it
completes nothing, and never offered. They have none of the shared safeguards of our own bots. The full rules, our
adaptations to our game and where we had to guess are in
[entry 2](journal/02-the-bonjour-bots.md#2-how-the-bonjour-bots-play).

### Measured strength

- Mixed tables (1,000 games, 4 bots drawn from our 8 bots and the three Bonjour bots): "Bonjour C" **51.1%**
  (46.0-56.1%), "Bonjour B" 35.4%, "Bonjour A" 19.5%; "Tactician" 28.9%, "Calculator" 26.9%.
- Against three "Calculators" (400 games each): "Bonjour C" 41.5%, "Bonjour B" 33.8%, "Bonjour A" 14.0%.
- Tournament with our honest bots (3,000 games, pool of 12): "Bonjour C" **56.3%** (53.3-59.4%), "Bonjour B" 38.2%,
  ahead of "GPT-Style" 34.1%, "Tactician" 33.5% and "Calculator" 29.7%.
- The same tournament with the [absurd bots](#absurd-bots) and ["Hustler"](#hustler) added (pool of 16): "Bonjour C"
  21.8%, "Bonjour B" 15.3%, "Bonjour A" 8.5%, while "Hustler" won 54.6%.

They beat our honest bots because our bots sell them lone properties one at a time, and the Bonjour bots then pay
almost any price for the last property of a set; they lose to the bots that send bad offers because they accept a property-for-pennies
offer whenever it does not break a set and they keep enough cash
([the mechanism](journal/02-the-bonjour-bots.md#the-mechanism)).

### Role in the project

They are our first opponents from outside our own design, and a rough bridge to the setup of earlier work: at the
paper's table (one player against all three) mini-evo5 wins 15.4% and "Calculator" 13.9%
([entry 1](journal/01-from-rule-bots-to-v7.md#mini-evo5-against-the-bonjour-bots)). They are not part of our main protocol or
our training pool yet.

## Resistant Bonjour A, B, C

**The [Bonjour bots](#bonjour-a-b-c) with one change: they accept an offer only if the original would and if it also
passes two safeguards our own bots already use.**

### What we changed

Only the answer to an offer or a counter-offer. The original rule decides first; an offer it would accept must then
pass the feed rule (no property that completes a rival's set or gives the rival two of three, unless the price
compensates) and have a positive [shared trade evaluation](glossary.md#the-game-and-the-bots). The readiness to
overpay for the property that completes a set stays within the evaluation: a light blue street with a price of $100
is still worth $300 to them, but not $1,000. Priorities, buying, auctions, building, jail, debts and their own offers
are unchanged, including the sale of lone properties when they have less than $200. The full description is in
[entry 2](journal/02-the-bonjour-bots.md#what-we-changed).

### Measured strength

- Tournament with our honest bots and the non-resistant Bonjour bots (3,000 games, pool of 15): "Resistant Bonjour C"
  **50.7%** (47.3-54.1%), "Resistant Bonjour B" 35.8%, "Resistant Bonjour A" 23.2%; the originals 47.6%, 30.1% and
  17.2%; "GPT-Style" 28.9%, "Calculator" 27.5%.
- The same tournament with the [absurd bots](#absurd-bots) and ["Hustler"](#hustler) added (pool of 19): "Resistant
  Bonjour C" **43.8%** (40.0-47.8%), "Resistant Bonjour B" 32.2%, "Resistant Bonjour A" 21.8%; the originals 23.8%,
  18.5% and 8.0%; "Hustler" 46.3%, "Calculator" 28.5%.
- The same pool without its donors ("Random" and the non-resistant Bonjour bots; pool of 15, 3,000 games): "Resistant
  Bonjour C" **43.1%** (39.7-46.4%), "Resistant Bonjour B" 35.2%, "Resistant Bonjour A" 16.7%; "Tactician" 32.3%,
  "Calculator" 30.7%, "Hustler" 27.0%.
- Elimination tournaments with one candidate per table ([entry 2](journal/02-the-bonjour-bots.md#10-a-first-search-for-the-best-bot)):
  "Resistant Bonjour B" won the run with 20 candidates (34.0% against 33.1% for "Bonjour Mixed Auctions" in the final)
  and a run with five bots (33.9% against 31.7% for "Resistant Bonjour C"); in a head-to-head of the two,
  with the hybrids built on "Resistant Bonjour C" left out of the pool, "Resistant Bonjour C" won 37.8% to 35.4%
  (3,000 games each). The gaps are 0.9-2.4 points, at the edge of noise or within it: the two are about equally
  strong, and which is ahead depends on the pool.

With the bots that send bad offers in the pool the resistant variants lose 1.4-6.9 points, the originals 9.2-23.8
([what the tournaments show](journal/02-the-bonjour-bots.md#what-the-tournaments-show)). Without donors "Resistant
Bonjour C" is clearly first, 7.9 points ahead of "Resistant Bonjour B" and 10.7 ahead of "Tactician" (noise about
+/-4.7), and "Resistant Bonjour A" is weak
([a tournament without donors](journal/02-the-bonjour-bots.md#6-a-tournament-without-donors)).

### Role in the project

They separate the Bonjour bots' strength in collecting sets from their weakness against bad offers. "Resistant
Bonjour C" is the strongest bot in our tournament without donors, and the
[variants with trade windows](#resistant-bonjour-b-and-c-with-windows) built on them lead the current
[ranking](#current-elo-ranking). They still share our bots' gap in selling
the first pieces of a set ([who can still be exploited](journal/02-the-bonjour-bots.md#who-can-still-be-exploited)).
They are not part of our main protocol or our training pool yet.

## Wary Calculator

**Plays like "Calculator", but does not sell or trade away the pieces that bring a rival closer to a set, and answers a
piece-by-piece offer with a counter-offer or a refusal.**

### Where it came from

It is our answer to the gap that the [Bonjour bots](#bonjour-a-b-c) exploit (the story is in
[entry 2](journal/02-the-bonjour-bots.md#8-closing-the-gap-with-a-wary-calculator)). The shared trade evaluation gives
its bonus for a color group only when the group is complete, so one or two of three squares of a group are worth just
the sum of their prices. Selling a lone square for 1.125x its price to a player who does not yet complete a set
therefore looks profitable to "Calculator" (it gains 0.125 of the price, the rival loses 0.125). The shared
safeguards catch only a square that completes a rival's set at once or gives the rival two of three. "Bonjour B" and
"Bonjour C" send such offers to all players at once and collect sets piece by piece. In 100 games of "Calculator"
against three "Bonjour C" bots, "Calculator" accepted 61 offers to sell a railroad to a player without railroads, 30
offers to sell an orange square and 27 to sell a light blue square to a player with no squares of that color (the other
squares of the color still in the bank), each time at 1.125x the price.

### How it plays

Like "Calculator": the same purchases, building, reserves, jail and proposals. It differs in how it values giving a
property away:

- **Progress toward a set.** A player who owns part of a color group whose remaining squares are all still in the bank
  gets, on top of the prices, the set bonus (1.5x the sum of the prices of the group) times a factor for one missing
  square or for two missing squares (both 1 by default): a square of a free color is worth its price plus the whole
  bonus. A color blocked by a third player counts for nothing. Railroads: each railroad of a player who does not own
  all four is worth an extra $200 by default (twice a quarter of the bonus for four railroads), even when the other
  railroads belong to other players.
- **Evaluation of trades.** The lower of two scores: the shared evaluation, and the same evaluation with the progress
  counted on both sides (the rival's gain weighs 0.6). So it accepts only what "Calculator" would accept, and its
  resistance to bad offers stays. A piece-by-piece offer gets a counter-offer with a surcharge that satisfies both
  scores, or a refusal. Examples of answers to offers of the Bonjour bots: an orange square with the rest of the group
  in the bank, offered $202, counter about $1,030; a light blue street priced $100, offered $112, counter about $590; a
  railroad, offered $225, counter about $410. When a third player holds a square of the color, it sells at 1.125x like
  "Calculator".
- **Its own offers** (proposals, the sale of a loose property to afford an important one, the sale of a property while
  in debt without a threat of bankruptcy): it skips the ones that bring a rival closer to a set unless the score with
  progress is positive. A debtor with no way out sells as before, since bankruptcy would cost everything anyway.
- **Buying and auctions.** A square that would give a rival progress in a free color is bought with the smallest
  reserve ($100) and bid for up to 1.25x its price (1.5x when it completes a rival's set, as in "Calculator"). In our
  measurements this rule changes almost nothing.

Its trade variability is 0.1, as for "Calculator" ([variance parameters](#variance-parameters)). The two shares, the
railroad share and the bid factor are parameters of the bot.

### Choosing the numbers

Games against three copies of one bot, seats rotating, the same seeds for all variants, 600 games per variant (wins):

| Variant | 3 x "Bonjour C" | 3 x "Bonjour B" | 3 x "GPT-Style" | 3 x "Tactician" |
|---|---|---|---|---|
| "Calculator" | 14.0% | 23.3% | 29.7% | 26.3% |
| Shares 1 and 1, no railroad bonus | 25.3% | 23.8% | 29.5% | 26.3% |
| Shares 1 and 1, railroad share 1 | 27.0% | 26.8% | 28.8% | 25.5% |
| **Shares 1 and 1, railroad share 2 (default)** | **28.7%** | **30.0%** | 29.0% | 25.5% |

Higher thresholds help only against the Bonjour bots; against other bots offers of single squares hardly occur and
nothing changes. The first version (shares 0.4 and 0.15, no railroad bonus) did nothing: 12.8% against three "Bonjour C"
bots against 11.9% for "Calculator" (800 games, other seeds). It sold the same squares at about 1.7x the price, since a
Bonjour bot accepts any counter-offer that leaves it $100. Results grew with the thresholds up to 1 and 1 (0.35 and 0.6:
16.3%; 0.6 and 1: 19.5%; 1 and 1: 22.8%) and no further.

### Measured strength

- Against three copies of one Bonjour bot on new seeds (400 games each, 100 seeds x 4 seats): against "Bonjour C" 26.3%
  (22.2-30.8%, mean place 2.38) for "Wary Calculator" and 12.3% (9.4-15.8%, 2.75) for "Calculator"; against
  "Bonjour B" 30.0% (25.7-34.7%, 2.41) and 23.8% (19.8-28.2%, 2.51). The neutral result is 25%.
- 3,000-game tournaments in the two pools of [entry 2](journal/02-the-bonjour-bots.md#4-two-tournaments-rock-paper-scissors)
  (one without and one with the bots that send bad offers) with "Wary Calculator" added to each; the intervals imply about
  900 and 700 games per bot:

| Bot | Without the bots that send bad offers | With them |
|---|---|---|
| "Wary Calculator" | 33.9% (30.9-37.0%) | 30.8% (27.4-34.4%) |
| "Calculator" | 32.2% (29.3-35.3%) | 26.9% (23.8-30.3%) |
| "Bonjour C" | 51.0% (47.7-54.3%) | 23.9% (21.0-27.2%) |
| "Bonjour B" | 37.2% (34.2-40.3%) | 17.7% (15.1-20.7%) |
| "Tactician" | 35.6% (32.5-38.8%) | 29.0% (25.7-32.5%) |
| "GPT-Style" | 31.6% (28.6-34.7%) | 27.5% (24.4-31.0%) |

  Against the Bonjour bots alone "Wary Calculator" wins twice as often against "Bonjour C" as "Calculator" and returns to
  the 25-30% range. In the pools the difference is small and within the intervals: it sits at only about 30% of the
  tables, and the Bonjour bots keep collecting sets from the other bots (in the pool without the bots that send bad
  offers "Bonjour C" still wins about half the games).
- Elo: 1521, the highest of the bots we designed ourselves ([ranking](#current-elo-ranking)). In the elimination
  tournaments it did not reach the final ([entry 2](journal/02-the-bonjour-bots.md#10-a-first-search-for-the-best-bot)).

### Role in the project

It closes the gap only partly: it resists the piece-by-piece purchases, but the Bonjour tactic still wins more. It is
the base of the offers that the [windows variants](#resistant-bonjour-b-and-c-with-windows) add, and it has an
[absurd version](#absurd-versions-of-the-leading-bots).

## Bonjour hybrids with Calculator

**Four variants of "Resistant Bonjour C" that borrow parts of "Calculator": its offers and auctions, or only its
auctions.**

### Where they came from

Offers are the Bonjour bots' weapon, and "Calculator" proposes trades in a very different way: it proposes first the
trades after which a set earns the most with three houses, also in trade windows, and it buys a property from the
player who is about to pay. We wanted to know whether a Bonjour bot gets stronger with our offers or with our auctions.
All four are ["Resistant Bonjour C"](#resistant-bonjour-a-b-c) (the same priorities, cash thresholds and resistant
answers to offers) in which the borrowed part is played by an instance of "Calculator"; no code is copied.

### The four variants

| Bot | Offers | Auctions |
|---|---|---|
| "Bonjour Calculator" | Those of "Calculator", in its own turn before the roll and in trade windows; the Bonjour offers are not sent | For squares in the high-priority groups of "Bonjour C" or squares that complete a set, the more aggressive of the two bots' decisions (pass only if both pass); "Calculator" for the rest |
| "Bonjour Calculator Pure" | As "Bonjour Calculator" | Always "Calculator" (an ablation) |
| "Bonjour Calculator Auctions" | Those of "Resistant Bonjour C" | Always "Calculator" |
| "Bonjour Mixed Auctions" | Those of "Resistant Bonjour C" | As the auctions of "Bonjour Calculator" |

Everything else (buying, priorities, building, jail, debts, answers to offers) is "Resistant Bonjour C". The mixed
auctions exist because "Bonjour C" overpays on purpose for its groups (a quarter of its cash above the highest bid).

### Measured strength

In the first elimination tournament (20 candidates, 5 rounds of 4,000 games, each table with one candidate and three
sparring partners drawn from the pool; [entry 2](journal/02-the-bonjour-bots.md#10-a-first-search-for-the-best-bot)):

- Round 2 (600 games each, noise about +/-4 points): "Bonjour Mixed Auctions" 34.7%, "Resistant Bonjour C" 33.2%,
  "Resistant Bonjour B" 33.0%, "Bonjour Calculator Auctions" 29.2%, "Bonjour Calculator Pure" 24.7%, "Bonjour
  Calculator" 24.5%. All three hybrids with auctions or offers of "Calculator" were eliminated in this round, while
  "Tactician" (31.0%) and "Wary Calculator" (30.3%) went on.
- As sparring partners (2,900-3,100 games each): "Bonjour Calculator" 21.9% and "Bonjour Calculator Pure" 21.1%,
  against 30.5% for "Resistant Bonjour C", 30.1% for "Bonjour Calculator Auctions" and 30.5% for "Bonjour Mixed
  Auctions".
- "Bonjour Mixed Auctions" reached the final and lost it by 0.9 points, 33.1% to 34.0% for "Resistant Bonjour B" (4,733
  games each), within noise.
- Elo: "Bonjour Mixed Auctions" 1514, equal to "Resistant Bonjour C"; "Bonjour Calculator Auctions" 1508; "Bonjour
  Calculator" 1444 and "Bonjour Calculator Pure" 1442 ([ranking](#current-elo-ranking)).

So the offers of "Calculator" cost the Bonjour bot about 9-10 points, and its auctions neither help nor hurt beyond
noise.

## Resistant Bonjour B and C with Windows

**The resistant variants with offers added where the Bonjour tactic stays silent: in trade windows, when raising money
for a debt, and as counter-offers.**

### Where they came from

Our game has trade windows, which the game of earlier work lacks: besides the owner's turn, every other player in turn
may manage assets and propose one trade at fixed moments ([glossary](glossary.md#the-game-and-the-bots)). The Bonjour
bots follow the earlier work and propose trades only in their own turn, before the roll; in a window they only build and
lift mortgages and then pass, and they say nothing when raising money for a debt or answering with a counter-offer.
These variants adapt the tactic to the trade-window mechanic of our game. Earlier [hybrids](#bonjour-hybrids-with-calculator)
that replaced the Bonjour offers with the offers of "Calculator" did poorly (about 20% in the first elimination
tournament), so here nothing is replaced: offers are added only where a Bonjour bot did nothing.

### How they play

All of ["Resistant Bonjour B" or "Resistant Bonjour C"](#resistant-bonjour-a-b-c) (buying, auctions, building, jail,
offers before the roll, resistant answers) plus, delegated to an instance of ["Wary Calculator"](#wary-calculator):

- **Trade windows.** In another player's turn first the building and mortgage lifting of the Bonjour bot, then a
  proposal of "Wary Calculator". Before a payment, first the purchase of a property from the player who is about to pay,
  then ordinary proposals (no repeats for the same layout of properties, none that bring a rival closer to a set). In
  the rare windows of its own turn only the proposal of "Wary Calculator".
- **Debt.** When the bank cannot cover the debt, or buildings would have to be sold, it sells a property to a player the
  way "Wary Calculator" does; otherwise it raises money like a Bonjour bot (mortgages, buildings, bankruptcy).
- **Counter-offers.** The decision to accept or reject stays with the resistant Bonjour bot. When it rejects and "Wary
  Calculator" would counter (with a surcharge that equalizes its evaluation, or one that compensates for feeding a rival),
  it sends the counter-offer only if the resistant Bonjour bot itself would accept it as an offer: the same Bonjour rule,
  no feeding, a positive shared evaluation. Otherwise a plain refusal.

Unchanged: buying with money from a mortgage, taking over mortgaged properties, and the answers of a debtor with no way
out.

Offers per game of the variant in 600 games at tables like those of the first tournament below: in windows 3.5 for B
(0.54 accepted) and 4.1 for C (0.66); of these before a payment 2.0 (0.40) and 2.3 (0.52); sales of a property in debt
1.8-2.0 (about 0.9 accepted); counter-offers 39 for B (1.6 accepted) and 45 for C (1.5), mostly replies to repeated
absurd offers. For comparison, the offers before the roll: 8.8 for B (3.6 accepted) and 22.2 for C (4.1).

### Measured strength

- **A 12,000-game elimination tournament** (tables of 4 with one candidate and three sparring partners, full trade
  windows, 200-round limit; candidates: the two windows variants and their originals; sparring partners: 16 bots,
  including the absurd bots and the originals, and both windows variants). Round 1 (6,000 games, 1,500 per candidate):
  "Resistant Bonjour C + Windows" **47.0%** (44.5-49.5%), "Resistant Bonjour B + Windows" **38.3%** (35.8-40.8%),
  "Resistant Bonjour C" 35.9% (33.5-38.3%), "Resistant Bonjour B" 32.3% (30.0-34.7%). Round 2 (3,000 each): C with
  windows 44.3% (42.6-46.1%), B with windows 38.0% (36.2-39.7%); C with windows won. As sparring partners (about 2,000
  games each): C with windows 37.1%, B with windows 32.6%, "Resistant Bonjour C" 31.9%, "Wary Calculator" 27.1%,
  "Resistant Bonjour B" 25.8%.
- **The second tournament** ([entry 2](journal/02-the-bonjour-bots.md#13-the-final-search-and-the-elo-ranking)), with
  seven candidates and 27 sparring partners: round 1 (about 570 games each) C with windows 37.4%, B with windows 34.4%,
  "Resistant Bonjour C" 28.7%, "Resistant Bonjour B" 28.4%. The final (3,572 games each): C with windows 38.5%
  (36.9-40.1%), B with windows 35.5% (33.9-37.1%).
- **Elo:** 1569 and 1557, first and second ([ranking](#current-elo-ranking)).

The windows help both variants beyond noise: in the first tournament C gains 11 points (47.0% against 35.9%, intervals
that do not overlap) and B 6 (38.3% against 32.3%, intervals that do not overlap). We have not separated the share of
the windows, the debt sales and the counter-offers.

### Role in the project

The strongest bots we have measured. They show that the Bonjour tactic gets stronger when the trade-window mechanic of
our game, which the earlier work lacks, is used for its goal, collecting sets by trade.

## Variance parameters

**Two numbers that make a bot's play less repetitive: trade variability and random moves.**

Without them, bots that meet the same situation play the same way down to the dollar. Both are ordinary parameters of a
bot, from 0 to 1, and with 0 a bot plays exactly as before (we checked that on 200 games by comparing the decision
traces).

- **Trade variability.** (a) The choice of an offer among the candidates is a draw instead of always the first: the
  chance of the k-th candidate is proportional to exp(-k / t) with t = 3 x variability, so 0.1 gives the first candidate
  in about 96% of cases and 0.3 in about 67%. The order is the bot's own, so the draw does not change its character.
  (b) The cash in offers and counter-offers varies by up to variability x 20%, in whole $10. (c) The acceptance threshold
  moves by up to variability x 20% of the nominal value of the trade (prices of properties, cash, jail cards at $50), so
  an offer on the edge of profitability is sometimes accepted and sometimes not. The safeguards always hold: candidates
  that feed a rival are dropped before the draw, an offer with a changed amount must stay legal, not feed a rival and not
  become a loss for the bot, and refusing an offer that feeds a rival is not subject to the threshold draw.
- **Random moves.** With this probability a decision is replaced by a random legal action from the decision list (roll,
  pass, build, mortgage, accept or reject an offer and so on; proposing trades is not on that list). A bot never accepts
  an offer that feeds a rival and never declares bankruptcy when there is another way out.

Defaults (random moves are 0 for all bots; "Random" has no parameters):

| Trade variability | Bots |
|---|---|
| 0.1 | "Calculator", "Absurd Calculator", "Hustler", "Simulator", "Fast Simulator", "Wary Calculator", "Absurd Wary Calculator" |
| 0.15 | "GPT-Style", "Tactician", "Master" and the absurd versions of the first two |
| 0.2 | "Cautious", "Miser" |
| 0.3 | "Investor", "Collector", "Absurd Investor" |
| 0 | All the [Bonjour bots](#bonjour-a-b-c) and their variants: they keep fixed rules by default |

The absurd bots have the variability of their originals, and their absurd offers are random by nature. The imaginary
continuations of the simulators are played by "Calculator" without variability, which makes the comparison of options
less noisy. The Bonjour bots accept both parameters as well: their own offers then vary in the choice among the offers of
a turn and in cash amounts, and their cash thresholds in answers move; hard refusals (nothing received, not enough cash,
a mortgaged property, jail cards, a lost set) are never drawn, and for the resistant variants the threshold moves only
upward from zero, so the safeguards and a positive evaluation always hold. In the hybrids and the windows variants the
part played by "Calculator" or "Wary Calculator" uses the variability of the hybrid, or 0.1 when that is 0, since they
were measured that way. All results in this file were measured with the defaults.

## Random

**The floor: it picks a random legal move and never proposes trades.**

### How it plays

"Random" chooses uniformly among the legal actions at every decision and never proposes a trade. Because it bids
without restraint in auctions, it is a surprisingly good test of whether a player can keep its own bidding in check.
When it answers an offer, it picks between accept and reject (never a counter-offer), so it accepts about half of all
offers whatever their terms; in tournaments it acts as a donor to players that exploit weaker ones
([the role of "Random"](journal/02-the-bonjour-bots.md#the-role-of-random)).

### Measured strength

- First 600-game tournament with four-player tables drawn at random: 0%.
- "Mean over 11 bots" in our main protocol always excludes "Random", which is reported separately as a sanity
  check: a sensible player should beat three "Random" bots almost every time. "Calculator" does so in 120 of 120
  games.

### Models that lose to it

Several models failed this check: mini-evo5 wins only 66.7% against three "Random" bots, Loki gen2b at 3 M steps (4.5 M total) won
0%, and screening variant A also 0%. The cause was overbidding in auctions against bots that bid recklessly, then
bankruptcy; teaching auctions (Loki gen2e) brought it to 100%
([the Loki sprint](journal/01-from-rule-bots-to-v7.md#the-loki-sprint)).
