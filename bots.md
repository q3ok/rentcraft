# The rule-based bots

Our bots are rule-based AI: each plays by a fixed set of hand-written rules and writes a short justification for
every decision, visible in replays. They serve three purposes: **opponents** during training, **teachers** to imitate,
and **yardsticks** for our models. The history of how they were built and tuned is in
[the journal](journal/01-from-rule-bots-to-v7.md#3-the-rule-based-bots).

| Bot | In one sentence | Wins in mixed tables |
|---|---|---|
| ["Calculator"](#calculator) | Computes every decision from board probabilities (a Markov chain); our strongest bot and main reference | 34.4% |
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
| ["Random"](#random) | Picks a random legal move and never proposes trades | 0% (b) |

"Wins in mixed tables": 1,000 games with 4 bots drawn at random from the 8 bots above "Simulator", with characters
restored after tuning ([tuning](journal/01-from-rule-bots-to-v7.md#tuning-and-why-we-undid-half-of-it)). With
four equal players, chance level is 25%.

(a) A different setup: a 3,000-game tournament with a 19-player pool, in which "GPT-Style" won 32.5%.

(b) The first 600-game tournament, with four-player tables drawn at random from six bots ("Investor", "Collector",
"Master", "Cautious", "Miser" and "Random"; "Calculator" did not exist yet).

In our main protocol (1,320 games; a model plus three copies of one bot on shared seeds), "Calculator" wins 35.7%; see
[how we measure](journal/01-from-rule-bots-to-v7.md#1-the-question-and-how-we-measure).

## Calculator

**Our strongest bot and main reference: it has no favorite colors and computes every decision from board
probabilities.**

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
Its trade variability is 0.1.

### Measured strength

- First tests (1,000 games each): 46.3% in four-player tables drawn at random from the seven bots that existed then
  ("Random", "Investor", "Collector", "Miser", "Cautious", "Master" and "Calculator"; "Investor" 37.0% in the same
  tables), 64.2% one-on-one with
  "Investor", 29.5% against three "Investors".
- Mixed tables after tuning (1,000 games, 4 bots drawn at random from 8): 31.9%; with characters restored **34.4%** (mean
  place 2.34), the best of the pool.
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
  place 2.26), the second-strongest bot.
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

In a 3,000-game tournament with a 19-player pool "Fast Simulator" won 31.3% ("GPT-Style" 32.5%). Both bots are too slow
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

## Random

**The floor: it picks a random legal move and never proposes trades.**

### How it plays

"Random" chooses uniformly among the legal actions at every decision and never proposes a trade. Because it bids
without restraint in auctions, it is a surprisingly good test of whether a player can keep its own bidding in check.

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
