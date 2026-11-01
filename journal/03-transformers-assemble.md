# Entry 3: Transformers assemble!

## First steps

Initially, we wanted to distill the MLP mini-evo5 into a transformer using DAgger. This work began before the ["Bonjour"](02-the-bonjour-bots.md) bots appeared. Despite their simplicity, those bots proved very effective, especially after we adapted them to our nearly full game rules.

So we decided to train a new model to imitate "Resistant Bonjour C + Windows", which also proposes trades in windows outside its own turn and resists exploitative offers.

Before launching training, we had to choose a network size. We searched for established formulas for sizing a network for a given task, but found that the choice is usually made empirically. We asked Gemini to recommend an initial architecture and, after a short discussion with an independent agent, chose 6x128 (feed-forward width 512, 4 attention heads).

I chose SiLU as the activation function. I also considered GELU, but Google searches suggested that SiLU was faster; that information informed my decision. Both are used in transformer models. We wanted to avoid ReLU because our analysis of the MLP mini-evo5 found that 16-22% of its neurons were inactive. We did not analyze or consider leaky ReLU or other activation functions.

We developed this training procedure through experimentation and some chance discoveries. Steps 1-4 form our kickstart:

1. **Auxiliary heads.** The model gets auxiliary heads for complete color sets, net worth, liquidity, rent on the player's own next roll, the type of the next card from each deck, and an estimate of the final finishing place.
2. **Rules phase.** We begin with the "rules phase".
   - Data: bots we choose play 2,000 games, of which we hold out 400.
   - Training: for up to 10 epochs, we feed the game state through the v7 encoder and train the auxiliary heads to predict the corresponding values.
   - Memorization: we check on held-out games whether a head starts memorizing. If it does, we detach it from the network trunk (memorizing_action: detach): its loss no longer updates the trunk. The place head is detached from the first epoch; the other heads are detached if memorization is detected. This replaces the head freezing described in [entry 1](01-from-rule-bots-to-v7.md#a-large-network-memorizes-games).
   - Early stopping: if the results do not improve for 2 epochs, we stop the training.
3. **Imitation of the teacher.** A teacher we choose plays 1,500 games, and we train the network to reproduce its moves, using a procedure similar to step 2 for up to 12 epochs. The auxiliary heads keep learning with a weight of 0.1.
4. **Teacher correction (DAgger).** The model then plays its own games (10 rounds of 250 games), and we compare its decisions with those its teacher would make. The model can reach situations the teacher would not encounter in its own games, so we teach it what the teacher would do there. In our runs, sufficiently large transformers already play reasonably well after this stage, while the MLPs do not; the smallest transformer, ymir 2x16, performs similarly to the MLPs.
5. **Learning to propose trades.** Next we work on improving the model's offers.
   - Why offers matter: the model has to compose an offer itself, much as a human would. It receives no ready-made candidates from a generator; the v7 encoder provides the offer history and the current draft. The model must open a draft, add the properties it wants to trade, and set the cash amounts. We consider this complex process central to the game and want the network to make the best possible offers.
   - Teacher moments and better offers: the teacher plays 1,500 games (4,000 in the first run of njord-gen0). We select up to 3 moments per game when the teacher made an offer that ended in an accepted trade in that game and ask the network to make a better one. A better offer is rewarded; a worse one is penalized.
   - Silent moments and rewards: we also want to encourage the model to propose trades when the teacher would make no offer. We randomly select moments when an offer is possible and let the model propose one. If the outcome is better across 8 simulated continuations (16 in njord-gen1) than without the offer, the model receives a large reward. I deliberately chose not to penalize offers that worsen the outcome in these silent moments, to encourage initiative. This choice applies only to silent moments; at teacher-offer moments, negative advantage is penalized.
   - What is rewarded: we evaluate the effect of a single change made by the network on the outcome of the whole game. The reward is based on the difference in finishing place and, when the place is unchanged, the difference in net worth.
   - How the update works: from each moment we play 8 pairs of continuations with the same dice and card order: in one the model makes its offer (and leads any further negotiation, the other player answers by itself), in the other the teacher does what it did in that game. In both, the rest of the game for the model's seat is played by the teacher and the other seats by their bots. The reward of a pair is the difference in final place; if the places are equal, 0.5 × tanh(net worth difference / 1000). The average over the 8 pairs is the advantage of that offer; advantages are normalized within each batch of 32 games, and the network is updated with REINFORCE (minus advantage × log-probability of every step of the offer it built), Adam, learning rate 3·10⁻⁵, gradient norm clipped to 1.
6. **Short PPO.** Finally, we run PPO for 1 M decisions (about 1,000 games), with small rewards for complete sets, net worth, rent, leading and mortgages, as a final stage of fine-tuning.

We have not yet tried combining steps 2 and 3 with full weights on all outputs; this is worth testing. Claude initially disagreed with the method in step 5 and did not understand the idea. Implementing it required a detailed explanation of the procedure I wanted.

## njord-gen0 and the beginnings of njord-gen1

Our first model trained this way was njord-gen0, our frontier model at the time of writing this entry. We experimented with stages 5 and 6 rather than following a fixed sequence; its actual training history is in [the lineage](../lineage.md#njord-gen0-6x128).

We have frozen njord-gen0. At the time of writing this entry, it had reached 1543 Elo in our ranking (2,832 games, 897 wins, mean place 2.33, mean game length 49 rounds), second among the rated players only to "Resistant Bonjour C + Windows" (1556 Elo). These ratings are a snapshot taken for this entry; other documents may report earlier snapshots.

In the elimination tournament (16,000 games in total), every table had one candidate and three sparring bots drawn from the same pool of five bots, so the candidates did not play each other. "Resistant Bonjour C + Windows" won 33.3% of its 4,533 games (95% CI 32.0-34.7%), an earlier version of njord-gen0 (before its last stage of training on trade offers) won 32.2% of 4,534 games (30.9-33.6%), and njord-gen0 won 31.5% of 2,533 games (29.7-33.3%; it was eliminated after round 3).

The reported tests did not detect a statistically significant difference from the teacher: p = 0.26 for the earlier version and p = 0.15 for njord-gen0. For the final njord-gen0, the test compares results only through round 3, with 2,533 games each for the model and teacher; it does not compare the model's 31.5% directly with the teacher's full-tournament 33.3%. These tests do not establish that the model and teacher are equally strong.

In a smaller tournament with four versions of the model at one table (200 games), njord-gen0 won 35.0%, while its earlier versions won 18.5-25.5%.

A fun fact: at a table of four copies of njord-gen0, the players accepted 41 trades in 42 rounds. Almost all of them were one street sold to a neighbour for cash and then bought back around the table: square 8 (light blue, first side of the board) changed owner 13 times, usually for its price of $100, the station on square 25 changed owner 7 times (for $100-425), and square 18 (orange) 6 times, for $200 each time. Nobody completed a set this way; the cash simply went around the table.

At the time of writing, we are working on njord-gen1. We have extended the method from step 5 to more areas of play and are testing it as a replacement for PPO. In our experience, PPO is difficult to tune in a game with this much randomness and such delayed feedback. We think it would require:

- a very low learning rate,
- close monitoring of KL divergence and the clipping fraction (clipfrac),
- monitoring for collapse and reducing the learning rate further if it occurs,
- perhaps a stronger KL anchor to the teacher or a smaller clipping coefficient.

We think this approach could produce the best results, but would require a great deal of time devoted to training.

When we observe a clear improvement from njord-gen0 to njord-gen1, we will publish a new entry describing the model and its training.

## Analysis: comparing network sizes against the same set of bots

We also decided to compare how well networks of different sizes play and imitate our best bot. They use the same teacher and training procedure, following steps 1-6 above. The evaluation uses the same set of 16 bots for every model, but the individual games need not use the same seeds or dice rolls. In DAgger and PPO, each model also generates its own training experience.

We added two MLPs for comparison. Bragi has a parameter count similar to the 6x128 transformer. Heimdall has the same hidden-layer widths, 1024 and 512, as the networks in [Decision Making in Monopoly using a Hybrid Deep Reinforcement Learning Approach](../related-work.md#bonjour-haliem-et-al-hybrid-deep-rl-2021-2022). Its total parameter count is different: our flattened input contains much more information (16,166 numbers rather than 240), and the game interface and outputs also differ. Heimdall therefore has 17.25 million parameters; matching the hidden-layer widths does not reproduce the paper's network or experiment.

| Model | Type | Network | Parameters | Step 2: complete-sets loss | Step 3: imitation loss | Wins after step 4 | Wins after step 5 | Wins after step 6 |
|---|---|---|---|---|---|---|---|---|
| ymir | transformer | 2x16 | 23,957 | 0.191 | 0.463 | 12.5% | 10.2% | 11.2% |
| hodr | transformer | 2x32 | 59,877 | 0.046 | 0.413 | 31.2% | 26.4% | 30.0% |
| vidar | transformer | 4x32 | 85,285 | 0.034 | 0.417 | - | - | - |
| ullr | transformer | 2x64 | 168,581 | 0.0101 | 0.353 | 30.8% | 33.6% | 41.2% |
| forseti | transformer | 4x64 | 268,549 | 0.0044 | 0.348 | 32.7% | 32.7% | 37.2% |
| hermod | transformer | 6x64 | 368,517 | 0.0049 | 0.339 | - | - | - |
| baldr | transformer | 2x128 | 533,445 | 0.0065 | 0.306 | 38.0% | 34.4% | 37.5% |
| vali | transformer | 4x128 | 929,989 | 0.0023 | 0.329 | 36.2% | 36.1% | 35.3% |
| bragi | MLP | [80, 64] | 1,322,301 | 0.057 | 0.595 | 9.5% | 11.4% | 10.8% |
| freyr | transformer | 6x128 | 1,326,533 | 0.0018 | 0.309 | 35.3% | 37.8% | 40.9% |
| tyr | transformer | 6x192 | 2,874,373 | 0.0017 | 0.316 | - | - | - |
| odin | transformer | 8x256 | 6,591,557 | 0.0041 | 0.335 | - | - | - |
| heimdall | MLP | [1024, 512] | 17,249,165 | 0.020 | 0.527 | 11.7% | 12.0% | 8.4% |

Losses are measured on held-out games (lower is better): step 2 is the complete-sets head after the rules phase, step 3 is the imitation of the teacher at the best epoch. Wins are the share of games won in the training evaluation (40 games against each of 16 bots; chance level is 25%). We skipped the remaining training and playing evaluations of vidar, hermod, tyr and odin because of limited computing resources; "-" marks those omitted stages. Each win share comes from 640 games, so its 95% confidence interval is about ±4 percentage points: smaller differences between models or stages are within the noise.

## Tournament: the models against each other

To compare the models directly, we ran a tournament in which every table had four different models drawn from the ten finished ones (8,458 games, about 3,400 per model). Chance level is 25%.

| Model | Network | Games | Wins (95% CI) | Mean place |
|---|---|---|---|---|
| hodr | transformer 2x32 | 3,375 | 34.5% (32.9-36.1) | 2.23 |
| njord-gen0 | transformer 6x128 | 3,403 | 32.1% (30.5-33.7) | 2.33 |
| baldr | transformer 2x128 | 3,382 | 30.0% (28.5-31.6) | 2.39 |
| forseti | transformer 4x64 | 3,386 | 29.9% (28.4-31.5) | 2.39 |
| ullr | transformer 2x64 | 3,419 | 28.7% (27.2-30.3) | 2.40 |
| freyr | transformer 6x128 | 3,477 | 27.9% (26.4-29.4) | 2.40 |
| vali | transformer 4x128 | 3,367 | 27.1% (25.6-28.6) | 2.45 |
| heimdall | MLP [1024, 512] | 3,306 | 17.3% (16.1-18.7) | 2.63 |
| ymir | transformer 2x16 | 3,348 | 13.8% (12.7-15.0) | 2.81 |
| bragi | MLP [80, 64] | 3,369 | 8.1% (7.3-9.1) | 2.99 |

We compared every pair of models with a test of two proportions and the Holm correction for 45 comparisons. hodr and njord-gen0 cannot be told apart; hodr is better than the remaining transformers, and njord-gen0 cannot be told apart from baldr and forseti. The five transformers from baldr to vali form one group. The two MLPs and ymir are clearly weaker than all the others.

## Experiment: letting the place head train the trunk

In all the models above, the head that predicts each player's final place was detached from the trunk from the first epoch of the rules phase, so the trunk never learned to judge who is winning. We had detached it because, with 2,000 games, the network memorized whole games instead of learning to evaluate positions. We wanted to check whether more data fixes this, because a trunk that can judge a position would be the basis for better training methods than imitating the teacher.

The test model, cassandra, has the same network as njord-gen0 (transformer 6x128) and ran only the rules phase, on 30,000 fresh bot games instead of 2,000, with every head, including the place head, training the trunk. Everything else was the same as in step 2.

| Place head loss | After epoch 1 | After epoch 2 |
|---|---|---|
| training games | 0.843 | 0.754 |
| held-out games | 1.275 | 1.364 |

For comparison, guessing uniformly gives 1.386, and the models above, with the place head detached, reached about 1.10 on held-out games. The other heads learned well (complete sets 0.022 on held-out games after two epochs).

The result is clear: even with 15 times more games, the network memorizes the games it trains on and gets worse on new ones. The cause is the task itself, not the amount of data: every position in a game shares the same final result, and in Monopoly that result depends mostly on future dice rolls, so a position from the early or middle game says very little about the final place. Letting the place head train the trunk made the prediction worse than keeping it detached. We stopped the run after two epochs.

## Conclusions so far

- **Network size stopped mattering early.** Above about 60,000 parameters, the transformers are within a few percentage points of each other, and the largest ones are not better than the small ones. The task we train, imitating one bot, does not need a large network.
- **Transformers imitate, MLPs do not.** With a similar or larger number of parameters, both MLPs stay far behind every transformer except the smallest one. The structure of the v7 encoder, one token per square, matters more than the parameter count.
- **The ranking depends on the opponents.** Against the 16 bots in the training evaluation, ullr and freyr were the best; against other models they are in the middle of the group. A model's strength in Monopoly cannot be summarized by a single number measured against one set of opponents.
- **We are close to the teacher, not beyond it.** In the elimination tournament neither version of njord-gen0 was significantly different from "Resistant Bonjour C + Windows". Our training copies its style well.
- **The game is random.** In our view, no tactic can guarantee victory at a four-player table. The practical goal is to minimize losses and avoid losing too early.

  In separate tournaments of roughly 6,000 games, we have reached win rates of about 33%, or about 38% against weaker opponents, with a best mean finishing place of about 2.17. Our interpretation is that the advantage we have gained from tactics alone remains limited.

  More broadly, we think interactions between players matter more than making better use of the game's mechanics. At a table of human players, we suspect that this also includes whether people like or dislike one another. We have not measured that human factor in our bot tournaments; it is our view of the game. All our experiments so far use four-player tables, and we still need to test two- and three-player games.
