# Rules of Rentcraft

Rentcraft implements the **same mechanics** as the classic modern edition of Monopoly: the same 40-square board layout and
numbers (prices, rents and building costs), two decks of 16 cards with the same effects as the original ones, auctions, jail,
even building, a bank with 32 houses and 12 hotels, mortgages with 10% interest, and bankruptcy. **It is not the
Monopoly game.** It has its own street names, its own card texts, its own logo and artwork, and two kinds of
differences from the official rules, both listed below: a few [simplifications](#simplifications) and
[our own decisions where the official rules are silent](#our-own-decisions-where-the-official-rules-are-silent).

In these write-ups we use our own names throughout: "Opportunity" and "Community Fund" cards, the "Start" square, the
jail release card, and streets described by square number and color group (for example "square 24, the third street
of the red group") rather than by their names in any edition of the original game.

## Simplifications

- **Turn-based auctions.** Instead of an open-outcry auction, players bid in turn order, raising by $1, $10, $50 or
  $100 or passing (a technical cap of $10,000 per bid). Whoever passes is out of that auction.
- **At most 4 players** (the original allows 2 to 8).
- **Trading and asset management at decision points, not literally "at any time".** A player can trade, build,
  sell buildings and mortgage at every moment where they make a decision (before and after rolling, when deciding
  to buy, in their auction turn, while paying a debt, when answering an offer, when taking over a mortgaged
  property) and in **trade windows** that open for the other players at fixed moments (before a payment, after a
  property is acquired, after a negotiation, at the end of a turn). A move is never interrupted halfway; a request
  to trade waits for the next safe moment.
- **House shortage is first come, first served.** The last houses end up with whoever buys them first; there is no
  house auction.
- **Rent is collected automatically.** In the original, rent not asked for before the next roll is lost.
- **Negotiation limits** (table settings): by default at most 3 negotiations started per turn (separately 3 per
  debt, per auction and per taken-over mortgaged property), at most 4 offers in one negotiation, one proposal per
  player per trade window, and at most 3 trade requests per player per turn.
- **An optional round limit** with a ranking by net worth. Tables with humans have no limit by default;
  simulations and training use 200 rounds.
- **Time limits** for AI agents (and, optionally, for humans): after the decision deadline a player resigns, or a
  passive autopilot plays for them if the table allows it; in trade windows a short reaction time applies, after
  which the player passes.

## Our own decisions where the official rules are silent

- Interest and mortgage repayment are rounded up to a whole dollar.
- Whoever receives a mortgaged property (in a trade or from a bankrupt player) is always asked, for each such
  property, whether to repay the mortgage at once or pay only the 10% interest, and has full rights (trading,
  mortgaging, selling buildings) before answering, even without the cash to repay.
- An auction winner without enough cash owes the bank and raises money like any debtor; if that fails, they are
  bankrupt to the bank and all their properties, including the one just won, are auctioned.
- Bankruptcy to several creditors at once splits the cash in proportion to the unpaid debts (leftover dollars are
  handed out one by one in turn order), and the properties are auctioned.
- Properties of a player bankrupt to the bank are auctioned in board order, each auction started by the player
  after the current one.
- Resigning first settles all debts through a bank-only liquidation (mortgages, building sales); only if that is
  not enough is the player declared bankrupt to the creditors.
- An offer may ask the recipient for more cash than they hold, as long as they could raise it from the bank before
  accepting (acceptance itself requires the cash).

## Where the official rules do say something

We follow them strictly, even when it costs speed or simplicity. For example, selling a hotel during a house shortage
breaks the whole color group down evenly, and a debtor may sell properties to other players to raise cash. The rules
went through five versions early in the project, and rankings are kept separately for each rules version (see
[the journal](journal/01-from-rule-bots-to-v7.md#4-rules-evolution-toward-full-trading-rights)).

*Monopoly is a trademark of Hasbro. Rentcraft is an independent research project, not affiliated with or endorsed
by Hasbro.*
