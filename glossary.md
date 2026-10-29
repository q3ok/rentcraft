# Glossary

Short explanations of the terms used in the [README](README.md), the [journal](journal/01-from-rule-bots-to-v7.md)
and [bots.md](bots.md).

Contents: [the game and the bots](#the-game-and-the-bots), [what the model sees and does](#what-the-model-sees-and-does),
[how we train](#how-we-train), [how we measure](#how-we-measure), [models](#models) ([MLPs](#mlps),
[transformers](#transformers)).

## The game and the bots

- **Trade window:** a fixed moment in the game (before a payment, after a property is acquired, after a
  negotiation, at the end of a turn) in which every other player in turn may manage assets and propose one trade, or
  pass.
- **Generator candidates:** ready-made trade proposals produced by the same code the bots use. A model that sees
  them can pick one instead of composing an offer itself.
- **Offer draft:** an offer a model is composing step by step (add or remove a property, add or request cash, add a
  jail release card), which ends with "send" or "cancel".
- **Markov chain:** a model of movement in which the next square depends only on the current one (and the dice and
  cards). "Calculator" uses it to compute how often each square is visited in the long run.
- **Hybrid player:** one player whose areas of play (buying, auctions, asset management, trades) are each played by
  a different bot or by the model.

## What the model sees and does

- **Encoder:** the code that turns the game state into the numbers the network sees. Ours went from standard-v1 to
  v7; see [encoders](journal/01-from-rule-bots-to-v7.md#encoders-observations).
- **Block (encoder block):** a named group of numbers inside an encoder's input, such as the board state, the rules,
  the offer history or the event log. Some encoders let blocks be combined freely; v7 is a closed set of blocks.
- **Action space:** the set of moves the network chooses from. **standard-v1** (178 actions) offers trades only as a
  choice among generator candidates; **standard-v2** (221 actions) lets the model compose offers step by step. These
  are names of action spaces; the first two encoders happen to share them, but they are separate settings.
- **Token:** in a transformer, one item of the input sequence with its own vector of numbers. Our transformers turn
  every square, every player and every history or log entry into its own token.
- **Feed-forward layer:** the part of each transformer layer that processes every token on its own (two linear layers
  with an activation between them), after attention has mixed information between tokens.
- **Activation (ReLU, GELU, SiLU, leaky ReLU):** the nonlinear function applied to a neuron's output. ReLU outputs zero
  for every negative input; a **dead ReLU** neuron outputs zero for all inputs and stops learning. GELU, SiLU and
  leaky ReLU are smooth or leaky alternatives that avoid this.
- **Auxiliary head:** a small extra output next to the policy that learns to describe the state (complete sets, net
  worth, liquidity, expected rent, the next card, every player's final place). It does not choose moves; it is there
  to make the network read its input.

## How we train

- **PPO (Proximal Policy Optimization):** the RL algorithm we use. It improves the policy in small steps from games
  the model has just played, clipping each update so the policy does not change too much at once.
- **Policy:** the network's output that chooses moves: a probability for every legal move at a decision.
- **Value estimate:** a second output of the network that predicts how well the game will go from the current
  state; PPO uses it to judge whether a move turned out better or worse than expected.
- **Checkpoint:** a saved copy of the network's weights at some point of training, named by the step count (for
  example "at 1 M").
- **Seed, seat:** a seed is the number that fixes a game's random generator (card order and dice); a seat is a
  player's position at the table (0 to 3); the starting roll decides who begins, and play then follows seat order.
- **M steps:** millions of training steps. One step is one decision of the model; with full trade windows a game
  takes about a thousand of them (1,036 per game in one measurement, 434 without windows; see
  [rules evolution](journal/01-from-rule-bots-to-v7.md#4-rules-evolution-toward-full-trading-rights)). Step counts
  restart with every run. In the Loki series we give a checkpoint's own steps with the total since the start of Loki
  gen2 in parentheses, for example Loki gen2d at 1 M (6.5 M total).
- **Epoch:** one full pass of training over a data set. In supervised training (imitation, the rules phase, teacher
  correction) it is one pass over all the collected decisions, and training usually runs several epochs, stopping
  early when the loss on held-out games stops improving. In PPO the decisions from the latest batch of games are
  passed over a few times (a few epochs) before the model plays new games.
- **Epsilon (random moves):** the probability that a bot makes a random legal move instead of its own choice, used to
  make bots less predictable; 0 by default.
- **Entropy, entropy bonus:** entropy measures how spread out the policy's choices are; high entropy means the model
  is unsure or exploring, low means it almost always picks the same move. The entropy bonus is a reward in PPO for
  keeping entropy up, so that the model keeps exploring; we can set it separately per decision type.
- **Imitation (behavior cloning):** supervised training on the decisions of a bot or of a hybrid player.
- **Rules phase:** a stage before imitation in which the network learns only its auxiliary heads on bot games, so
  that it learns to read the state before it learns to act.
- **Teacher correction (DAgger):** the model plays, teachers label each of its decisions per area, the labeled data
  accumulates over rounds and the model trains on all of it.
- **KL anchor:** a penalty in PPO for moving the model's policy away from the starting model (measured by the KL
  divergence). Together with a teacher in RL it is one of the "anchors" in
  [PPO drifts without an anchor](journal/01-from-rule-bots-to-v7.md#ppo-drifts-without-an-anchor).
- **Teacher error (trade error):** how far the model's choices are from a teacher's. For each decision the teacher
  labeled, we take the probability the model gives to the move the teacher chose and compute minus its logarithm
  (cross-entropy); the teacher error is the average over held-out decisions. It is 0 when the model always puts all
  its probability on the teacher's move and grows when the model often prefers other moves or is unsure (for
  example, a model that always gave the teacher's move a probability of one half would score about 0.69). "Trade error" is the same measure
  restricted to trade decisions.
- **Validation loss, held-out loss:** the training loss (cross-entropy for choices, such as the teacher's move or the
  bot's next offer-building step) computed on decisions or whole games the network did not train on. A gap between
  training and held-out loss shows memorization.
- **Weight averaging:** averaging the weights of several checkpoints of one run into one model, which smooths out
  noise between checkpoints. mini-evo5 is such an average.
- **Coordinate search:** tuning several numbers by changing one at a time, keeping the best value, and moving on to
  the next; we used it to tune the bots.
- **Area delegation:** in training, some areas of play are handed to a bot so that the reward reflects mainly the
  area the model is learning (for example trades).
- **Paired reward:** a training reward computed from the difference between two plays of the same game (seed, rivals,
  seat): once by a bot alone and once with the model taking over one area. It isolates the effect of that area.
- **Opponent mix weight:** the share of training games in which a given kind of opponent sits at the table (for
  example 0.3 for earlier versions of the model itself).
- **Rival model:** a trained model, not a bot, used as an opponent in training.
- **League:** training against a set of rival models, usually earlier versions of the same series, alongside the
  bots.
- **Student, distillation:** distillation moves what one player knows into a new network by training the new network
  (the student) on that player's decisions, here by imitation and teacher correction; we used it to move from MLPs to
  transformers, whose weights cannot be copied from an MLP.
- **Gymnasium, PettingZoo:** standard Python interfaces for RL environments, Gymnasium for one agent and PettingZoo
  for several agents; our game is wrapped in them.
- **MCTS (Monte Carlo tree search):** a search method that plays out many simulated continuations from the current
  position and builds a tree of the most promising moves. None of our models uses it yet.

## How we measure

- **Chance level:** the share of games a player wins when all four players are equally strong, 25%. It is not the
  result of the "Random" bot, which won 0% of its games in our first tournament.
- **Win, mean place:** the last player left wins, or at the 200-round limit the richest by net worth; mean place is
  the average finishing place from 1 to 4 (2.5 for equal players); see
  [wins and places](journal/01-from-rule-bots-to-v7.md#1-the-question-and-how-we-measure).
- **Bradley-Terry fit:** a model that gives every player a strength so that the chance of one finishing ahead of
  another depends only on their strength difference; fitted to all game results at once and shown on the Elo scale,
  it is more stable than Elo updated game by game.
- **Successive halving:** a tournament in rounds that plays a few games with every candidate, drops the weaker half,
  and gives the survivors more games, so the best of many checkpoints is found without playing all of them to the
  end.
- **Shared seeds:** every compared player starts each game from the same seed, so the card order and the stream of
  dice are the same; once players' decisions change the flow of the game, later rolls and cards can fall to different
  turns or players. Results repeat to the decimal, which does not make them precise; see
  [how we measure](journal/01-from-rule-bots-to-v7.md#1-the-question-and-how-we-measure).
- **Paired test:** a measurement: the same game (seed, rivals, seat) played twice, once by a bot alone and once with
  the model taking over one area; the mean difference in results over many pairs measures that area. Not to be
  confused with the **paired reward**, which uses the same pairing as a training signal.
- **95% CI, SE:** a 95% confidence interval ("+/-X") and a standard error. For a win rate p over n games the 95% CI is
  about 1.96 x sqrt(p(1-p)/n); in our write-ups "+/-X" is a 95% CI unless marked as SE.
- **Main protocol (1,320 games):** our headline measurement: a model plus three copies of one bot, 11 bots x 30
  shared seeds x 4 seats, that is 330 deals each played once from every seat. The four games of a deal are not
  independent, so a mean near 30% carries about +/-2.5 points (95% CI) if all games count as independent and about
  +/-5 if only the deals do; see
  [how we measure](journal/01-from-rule-bots-to-v7.md#1-the-question-and-how-we-measure).
- **Old protocol (330 games):** the protocol of the Loki series: 12 bots (the 11 plus "Random") x 30 games, 330 in
  the mean over 11 bots. Each bot's 30 games use 30 seeds, the same for every model, with the model's seat rotating
  from game to game (8, 8, 7 and 7 games in seats 0 to 3). About +/-4.7-4.9 points (95% CI) on the mean, about +/-16
  on a single bot.
- **Fixed 300-game test:** an early test of small-32x32 and its successors against "GPT-Style", "Calculator" and
  "Investor", with the same 300 games for every checkpoint.
- **Training evaluation:** 20 to 100 games per bot (depending on the run) on new seeds during training. Cheap but noisy, and its set of
  opponents changed over time.

## Models

Naming convention: "32x32" is an MLP (multilayer perceptron) with two hidden layers of 32 neurons each; "3x64" is a
transformer with 3 layers of width 64 (so "4x128" has 4 layers of width 128). The prefixes simple, small and mini
only mark successive families of our small MLPs; Loki, Mjolnir and Njord are names of transformer series. Results
come from different protocols (see [how we measure](#how-we-measure)), and only results measured with the same
protocol can be compared directly; values in parentheses come from noisy training evaluations. A result reads "first
measurement -> best -> last measurement", each with its step count; when the best is also the last, it shows two
values, and a single value says when it was measured. Loki steps are the run's own steps with the total since the
start of Loki gen2 in parentheses (see "M steps" under [how we train](#how-we-train)). Which model was trained
from which is shown in the [model lineage](lineage.md) diagram.

### MLPs

| Model | Network | Encoder / action space | Started from | Result: first -> best -> last | Protocol |
|---|---|---|---|---|---|
| simple-small-32x32 | MLP 32x32 | blocks (trades, context, rents, sets) / standard-v1 | imitation of "GPT-Style" | (19.4% after imitation -> 25.4% at 0.5 M -> 13.4% at 5 M) | training evaluation |
| simple-kl-32x32 | MLP 32x32 | as above / standard-v1 | imitation of "GPT-Style", KL anchor 0.1 | (16.6% -> 32.0% at 1 M -> 17.7% at 2 M) | training evaluation |
| simple-kl2-32x32 | MLP 32x32 | as above / standard-v1 | simple-kl-32x32 at 1 M, KL anchor 0.05 | (31.4% -> 35.7% at 3 M -> 33.4% at 4 M) | training evaluation |
| small-32x32 | MLP 32x32 | blocks with memory, rents, sets, debts / standard-v1 | imitation of "GPT-Style" | 11.0% after imitation -> 27.0% at 18 M -> 26.3% at 20 M | fixed 300-game test |
| small-trades-32x32 | MLP 32x32 | blocks / standard-v2 | small-32x32 at 6 M | 11.3% at 1 M -> 13.3% at 2 M -> 13.0% at 6 M (9.0% at 5 M); the final model won 48.9% in a tournament with other models | fixed 300-game test; tournament |
| small-32x32-trade-teacher | MLP 32x32 | blocks with memory, liquidity and offer / standard-v2 | small-32x32, live trade teacher "Tactician" in PPO | (5.2% -> 14.2% at 1 M, the last) | training evaluation |
| small-32x32-trade-dagger | MLP 32x32 | blocks with offer history / standard-v2 | small-32x32, 4 DAgger rounds of trades from "Tactician" | (14.7% before DAgger -> 10.7% after) | training evaluation |
| small-evo | MLP 32x32 | blocks with generator candidates / standard-v2 | small-32x32 + 1 M steps with trade blocks | (19.7% -> 24.7% at 2 M -> 19.7% at 5 M) | training evaluation |
| mini-evo2 | MLP 32x32 | as small-evo | small-evo | (15.4-26.3% up to 37.9 M, no trend) | training evaluation |
| mini-evo3 | MLP 32x32 | as small-evo | mini-evo2, 2 DAgger rounds of trades from "Calculator" | (31.2% -> 37.1% at 3 M -> 34.2% at 5 M) | training evaluation |
| mini-evo4 | MLP 32x32 | as small-evo | mini-evo3 | (36.7% -> 44.2% at 5 M -> 38.3% at 10 M) | training evaluation |
| 32x32x32 student | MLP 32x32x32 | blocks / standard-v2 | an imitation model | (13.5% before RL -> 4.0% after); dropped | training evaluation |
| **mini-evo5** | MLP 32x32 | as small-evo | mini-evo4 at 8 M | final averaged model: **30.2%**; 30.3%; (39.4% -> 44.0% at 6 M -> 38.1% at 10 M) | main; old; training evaluation |

### Transformers

| Model | Network | Encoder / action space | Started from | Result: first -> best -> last | Protocol |
|---|---|---|---|---|---|
| 3x48 student | transformer 3x48 | blocks with generator candidates / standard-v2 | from scratch, teacher correction by mini-evo5 and "Calculator" | (27.3% after 10 rounds) | training evaluation |
| 3x64 student | transformer 3x64 | blocks without candidates / standard-v2 | as the 3x48 student | (14.8% after 6 rounds) | training evaluation |
| 4x64 student | transformer 4x64 | blocks without candidates / standard-v2 | as the 3x48 student | stopped in round 4, no result | - |
| Loki gen1 | transformer 3x64 | as the 3x64 student | the 3x64 student | (14.8% -> 8.6% at 1 M) | training evaluation |
| Loki gen2 | transformer 3x64 | as the 3x64 student | Loki gen1 | 13.9% at 1.0 M (1.0 M) -> 10.0% at 1.5 M (1.5 M) | old |
| Loki gen2b | transformer 3x64 | as the 3x64 student | Loki gen2 at 1.5 M | 10.6% at 1.0 M (2.5 M) -> 16.7% at 2.0 M (3.5 M) -> 4.8% at 3.5 M (5.0 M) | old |
| Loki gen2c | transformer 3x64 | as the 3x64 student | Loki gen2b at 2 M (3.5 M) | 18.2% at 0.5 M (4.0 M) -> 23.9% at 2.0 M (5.5 M), the last | old |
| Loki gen2d | transformer 3x64 | as the 3x64 student | the end of Loki gen2c | 20.9% at 0.5 M (6.0 M) -> 24.8% at 1.0 M (6.5 M) -> 20.6% at 2.0 M (7.5 M) | old |
| Loki gen2e | transformer 3x64 | as the 3x64 student | Loki gen2d at 1 M (6.5 M), 3 DAgger rounds | 19.4% after DAgger -> 23.9% at 1.0 M (7.5 M) -> 22.7% at 1.5 M (8.0 M) | old |
| Loki gen2f | transformer 3x64 | as the 3x64 student | Loki gen2e at 1 M (7.5 M) | 23.6% at 0.5 M (8.0 M) -> 24.5% at 1.0 M (8.5 M), the last | old |
| Loki gen3a | transformer 3x64 | as the 3x64 student | Loki gen2d at 1 M (6.5 M) | 17.3% at 1 M (7.5 M) -> 27.0% at 3 M (9.5 M), the last | old |
| Loki gen3b | transformer 3x64 | as the 3x64 student | the end of Loki gen3a | 24.2% at 1 M (10.5 M) -> 25.8% at 2 M (11.5 M) -> 20.6% at 3 M (12.5 M) | old |
| Loki gen3c | transformer 3x64 | as the 3x64 student | the end of Loki gen3a, DAgger from bot trade snapshots | 27.9% after DAgger -> 24.5% after 0.25 M RL (9.75 M) | old |
| Screening A | transformer 3x64 | the Loki encoder / standard-v2 | imitation of "Calculator" | 20.1% at 1 M; final not measured | main |
| Screening B, B' | transformer 3x64 | v6 (B': without the rent block) / standard-v2 | imitation of "Calculator" | B 15.9%, B' 14.9% at 1 M; final not measured | main |
| Screening B + memory | transformer 3x64 | v6 with memory / standard-v2 | imitation of "Calculator" | 13.8% at 1 M -> 12.8% at 3 M (final) | main |
| Screening C, C-4x128 | transformer 3x64 (C-4x128: 4x128) | v6 with auxiliary heads / standard-v2 | rules phase, imitation of "Calculator" | C 4.9%, C-4x128 8.6% at 1 M; final not measured | main |
| Screening D (log of 32, 64 or 128) | transformer 3x64 | v6 with heads, counters, offer history, event log / standard-v2 | rules phase, imitation of "Calculator" | at 1 M -> at 3 M (final): log of 32 4.2% -> 4.8%, of 64 8.9% -> 6.4%, of 128 9.0% -> 5.6% | main |
| Mjolnir | transformer | v6 / standard-v2 | rules phase, imitation of mini-evo5, 2 rounds of teacher correction by mini-evo5 and "Calculator" | 11.0% before RL; (about 8.5% -> 17.2% at 3 M -> 16.2% at 4 M) | main; training evaluation |
| Mjolnir gen0c | transformer 4x128 (about 0.66 M parameters) | v6 / standard-v2 | as Mjolnir | stopped after its place head memorized games; no result | - |
| first njord attempt | transformer 6x128 | v7, first version (log with trade-window entries) / standard-v2 | from scratch: rules phase | stopped in the rules phase; no play result | - |
| njord-gen0 | transformer 6x128 (1.33 M parameters) | v7, 64-entry log, 8 offers per direction / standard-v2 | from scratch: rules phase, imitation of a hybrid player, teacher correction, RL | in training, no results yet | - |
