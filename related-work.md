# Related work

Rentcraft is not the first attempt to teach a computer this game. Below are the earlier works we know of, their
limitations, and how Rentcraft differs.

## Earlier work

### Bonjour, Haliem et al.: hybrid deep RL (2021-2022)

*Decision Making in Monopoly using a Hybrid Deep Reinforcement Learning Approach*, arXiv 2103.00683, later
IEEE Transactions on Emerging Topics in Computational Intelligence, vol. 6, no. 6, pp. 1335-1344 (2022). The most substantial neural-network work on
the game.

The first arXiv version (v1, 2021) carried a different title, *Learning Monopoly Gameplay: A Hybrid Model-Free Deep
Reinforcement Learning and Imitation Learning Approach* (Haliem, Bonjour et al.). In it the agent first imitated a
rule-based agent built to reflect human reasoning, which set its initial policy, and then refined that policy with
deep reinforcement learning. The imitation was built into exploration rather than run as a separate stage: a deep Q
network (DQN) agent that, with some probability, took the rule-based agent's action instead of its own.

Both versions ran on the open-source GNOME simulator (described [below](#the-gnome-simulator)), and the hybrid
results below come from the final version, against three fixed-policy agents it calls FP-A, FP-B and FP-C.

- **Method:** PPO (Proximal Policy Optimization) and double DQN; a fully connected network (1024-512, ReLU), a 240-number state, 2,922 masked actions.
- **Hybrid idea:** rare and simple decisions (buying a property, accepting offers) are handled by a fixed rule; the
  network learns the rest (building, mortgages, proposing trades).
- **Results:** the hybrid PPO agent wins about 91-92% of games against three fixed-policy bots (plain PPO about 70%);
  the hybrid gives roughly 20-30 points over learning everything at once.
- **Limitations:**
  - the paper does not discuss auctions;
  - trading only properties without buildings or mortgages, one open offer per player;
  - offers are single pre-built actions: property-for-property, or buy/sell at 0.75 / 1 / 1.25 of the price;
    no arbitrary amounts, no multi-property offers, no counter-offers;
  - doubles without special rules, cards and fees handled automatically;
  - opponents are simple fixed strategies (different property priorities), so a high win rate says little about
    performance against strong bots;
  - short training (PPO: 2,000 games).

### The GNOME simulator

GNOME is the open-source simulator of the game by Kejriwal and colleagues (Kejriwal and Thomas, *Simulation Modelling
Practice and Theory*, 2021), built for research on novelty in open-world games within DARPA's SAIL-ON program. The
code is on GitHub (GNOME-p3). It includes agents with fixed rules (for example `background_agent_v3_1`), but not the
code or weights of the RL agents from the paper above, and the repository carries no license. The fixed-policy
agents FP-A, FP-B and FP-C of the paper are probably built on these rule-based agents, but we could not confirm it.

From the code, GNOME's rules are simpler than ours in several places, which matters when comparing results:

- doubles have no effect (no extra roll, no jail for a third double);
- jail costs one skipped turn (or leaving earlier with a $50 fine or a card);
- trades cover properties and cash only: no counter-offers, no jail cards, mortgaged (and built-on) properties are
  excluded, and accepting requires the cash in hand;
- in bankruptcy all assets return to the bank, without an auction and without passing to the creditor;
- the bank's cash is finite.

Auctions, even building, the house and hotel limits and mortgages work much as in our game. The win rates of the
paper above therefore concern a simpler game and simple opponents. We are preparing our own reimplementation of
GNOME's latest rule-based agent as an additional opponent, without copying its code.

### Bailis, Fachantidis, Vlahavas: Q-learning (2014)

*Learning to play Monopoly: A Reinforcement Learning approach*, AISB 2014. The game modeled as a single-agent
Markov decision process (the other players are part of the environment), learned with Q(lambda)-learning and a small
neural network (one hidden layer) as the value approximator. An early, simple work: no deep networks and no trading
between players.

### yyongg, *Monopoly-Agent*: masked PPO with heuristic trading

An open-source hobby project (Python) with design notes and measured results.

- **Method:** MaskablePPO on a 256-256 MLP (multilayer perceptron), a 261-number state, 125 masked actions (jail, buying, building,
  liquidation, auction bids in six price buckets); four-player self-play against frozen snapshots of itself and
  heuristic bots; reward shaped on net worth relative to opponents, with bonuses for properties and sets.
- **Trading:** following Bonjour et al., trades are not learned: a heuristic engine proposes packages of up to three
  properties plus cash for every seat at a single price, and the partner accepts if its valuation does not drop. The
  author calls the earlier learned trade policy catastrophic: its trade responder accepted about 85% of the offers
  put to it and gave away complete sets.
- **Results:** 31.5% wins over 200 games (95% CI 25.5-38.2%) against three heuristic bots that differ only in color
  group priorities (chance level: 25%), after 2M steps; earlier configurations reportedly reached about 50%, without game
  counts.
- **Limitations:** the model neither proposes nor evaluates trades; no counter-offers; the agent mortgages and sells
  buildings only in forced liquidation; the opponents share the agent's trade rule.

### Other works

Arun et al. (TENCON 2019) also applied reinforcement learning to the game (we could not access the full text, so we
do not describe it; it is not related to the GNOME simulator above). Student and hobby projects, e.g. three Q-learning variants (Qu, Peng, Jing and Izabayo, Carleton College, 2025), Monte
Carlo tree search with actions sampled first by type and then within the type (Sammul, a fourth-year project at the
University of Edinburgh, 2018), "Bot-opoly" in MATLAB (regression-tree position evaluation; trades only one property for one property). They usually
simplify the rules heavily and skip trading or reduce it to a few fixed options.

## Closest related methods and games

Not about this game, but closest to what we do:

- **Trade negotiation in Settlers of Catan.** Cuayáhuitl, Keizer and Lemon (2015) trained a deep RL policy for the
  trading negotiations of Catan (offering resources and answering offers). Their learned policy won 53% of games
  against three rule-based bots, against 27% for a player trained by supervised learning on a dialogue corpus. As with our trades, the action set was constrained to legal moves.
- **Negotiation in Diplomacy: CICERO.** Meta's CICERO (Bakhtin, Brown, Dinan et al., *Science*, 2022) reached
  human-level play in Diplomacy, a seven-player game built around negotiation, by combining a language model for
  dialogue with a strategic reasoning engine.
- **A KL anchor during RL fine-tuning: AlphaStar.** Vinyals et al. (*Nature*, 2019) started StarCraft II agents from
  a policy trained by supervised learning on human games and, during reinforcement learning, kept a KL loss toward
  that supervised policy. Our KL anchor (a KL penalty in PPO toward the imitated starting model) is the same kind of
  regularization. A KL penalty toward a pretrained model is also standard when fine-tuning language models with RL
  (Jaques et al., 2019; Ziegler et al., 2019).
- **Related: KL-regularized search, piKL.** Jacob, Wu, Farina, Lerer, Hu, Bakhtin, Andreas and Brown (ICML 2022)
  regularize search toward a policy learned by imitation with a KL penalty, which keeps play both strong and close to
  the imitated behavior, and apply it to chess, Go and no-press Diplomacy. It regularizes search rather than RL
  training, but rests on the same idea.
- **Teacher correction: DAgger.** Ross, Gordon and Bagnell (AISTATS 2011) introduced the method we call teacher
  correction: the learner plays, an expert labels the states the learner actually reaches, and the learner is
  trained on all the labeled data accumulated over rounds.
- **PPO.** Schulman, Wolski, Dhariwal, Radford and Klimov (2017) introduced Proximal Policy Optimization, the RL
  algorithm we use. The paper also describes a variant with an adaptive KL penalty toward the previous policy; our
  anchor instead penalizes the distance to a fixed starting model.

## What Rentcraft does differently

We have not measured our models on the setups of the works above (their opponents, rules or benchmarks), so our win
rates and theirs are not directly comparable. The differences below are differences in setup, not in measured
strength.

- **Nearly full rules:** auctions, mortgages with a choice when a mortgaged property changes hands, multiple creditors, building shortages, even building, jail with
  doubles, and trading rights at every decision point. The few simplifications are
  [documented](rules.md#simplifications), and so are
  [our own decisions where the official rules are silent](rules.md#our-own-decisions-where-the-official-rules-are-silent),
  for example that bids are not limited by the bidder's cash (a technical cap of $10,000 per bid; a winner without
  enough cash owes the bank and may go bankrupt).
- **Trading without shortcuts:** offers with any number of properties and cash in both directions, counter-offers,
  trade windows during other players' turns, negotiation limits. A model can compose an offer **step by step**
  (select a property, add cash, request, send) or pick from a generator's suggestions.
- **Strong bots as opponents and teachers:** over a dozen rule-based bots with distinct characters; the best one wins about
  36% in our main protocol (three copies of one bot at the table; chance level: 25%). They are also our training
  opponents, so our results show strength against this pool
  ([limitations](journal/01-from-rule-bots-to-v7.md#13-limitations)).
- **Several learning methods in one toolkit:** imitation of bots, teacher-guided correction (DAgger) per decision type,
  PPO with anchors (trade teacher, KL penalty to the starting model), MLP and transformer networks, auxiliary
  prediction heads.
- **Evaluation:** checkpoint comparisons on shared seeds, ratings on the Elo scale from a Bradley-Terry fit
  to game results, tournaments, per-area diagnostics
  (which area of play loses most against a bot).
- **Play with humans and AI agents:** a browser table, an MCP and REST server, replays.

## Sources

- T. Bonjour, M. Haliem, A. Alsalem, S. Thomas, H. Li, V. Aggarwal, M. Kejriwal, B. Bhargava, *Decision Making in
  Monopoly using a Hybrid Deep Reinforcement Learning Approach*, IEEE Transactions on Emerging Topics in
  Computational Intelligence 6 (6), 1335-1344, 2022,
  [doi:10.1109/TETCI.2022.3166555](https://doi.org/10.1109/TETCI.2022.3166555),
  [arXiv 2103.00683](https://arxiv.org/abs/2103.00683), [IEEE Xplore](https://ieeexplore.ieee.org/document/9775710).
- M. Haliem, T. Bonjour, A. Alsalem, S. Thomas, H. Li, V. Aggarwal, B. Bhargava, M. Kejriwal, *Learning Monopoly
  Gameplay: A Hybrid Model-Free Deep Reinforcement Learning and Imitation Learning Approach*, 2021, first version of
  the paper above, [arXiv 2103.00683v1](https://arxiv.org/abs/2103.00683v1).
- M. Kejriwal, S. Thomas, *A multi-agent simulator for generating novelty in monopoly*, Simulation Modelling
  Practice and Theory 112, 102364, 2021,
  [doi:10.1016/j.simpat.2021.102364](https://doi.org/10.1016/j.simpat.2021.102364); code:
  [github.com/mayankkejriwal/GNOME-p3](https://github.com/mayankkejriwal/GNOME-p3).
- M. Kejriwal, S. Thomas, *Generating Novelty in Open-World Multi-Agent Strategic Board Games*, 2025,
  [arXiv 2507.03802](https://arxiv.org/abs/2507.03802) (the GNOME simulator and the DARPA SAIL-ON program).
- E. Arun, H. Rajesh, D. Chakrabarti, H. Cherala, K. George, *Monopoly Using Reinforcement Learning*, TENCON 2019,
  2019 IEEE Region 10 Conference, 858-862, 2019,
  [doi:10.1109/TENCON.2019.8929523](https://doi.org/10.1109/TENCON.2019.8929523).
- P. Bailis, A. Fachantidis, I. Vlahavas, *Learning to play Monopoly: A Reinforcement Learning approach*, AISB 2014,
  [PDF](https://www.doc.gold.ac.uk/aisb50/AISB50-S02/AISB50-S2-Bailis-paper.pdf).
- X. Qu, D. Peng, A. Jing, P. C. Izabayo, *Reinforcement Learning in Monopoly Through Three Q-learning Variants*,
  Carleton College, 2025, [digitalcommons.carleton.edu/comps/3824](https://digitalcommons.carleton.edu/comps/3824).
- S. Sammul, *Learning to Play Monopoly with Monte Carlo Tree Search*, 4th Year Project Report, School of Informatics,
  University of Edinburgh, 2018.
- yyongg, *Monopoly-Agent*, [github.com/yyongg/Monopoly-Agent](https://github.com/yyongg/Monopoly-Agent),
  design notes and results at [yyongg.github.io/Monopoly-Agent](https://yyongg.github.io/Monopoly-Agent/).
- J. Zia, *Bot-opoly* (MATLAB), [github.com/jonzia/Monopoly](https://github.com/jonzia/Monopoly).
- H. Cuayáhuitl, S. Keizer, O. Lemon, *Strategic Dialogue Management via Deep Reinforcement Learning*, NIPS 2015
  Workshop on Deep Reinforcement Learning, [arXiv 1511.08099](https://arxiv.org/abs/1511.08099).
- Meta Fundamental AI Research Diplomacy Team (FAIR), A. Bakhtin, N. Brown, E. Dinan et al., *Human-level play in the
  game of Diplomacy by combining language models with strategic reasoning*, Science 378 (6624), 1067-1074, 2022,
  [doi:10.1126/science.ade9097](https://doi.org/10.1126/science.ade9097).
- O. Vinyals, I. Babuschkin, W. M. Czarnecki, M. Mathieu et al., *Grandmaster level in StarCraft II using multi-agent
  reinforcement learning*, Nature 575 (7782), 350-354, 2019,
  [doi:10.1038/s41586-019-1724-z](https://doi.org/10.1038/s41586-019-1724-z).
- N. Jaques, A. Ghandeharioun, J. H. Shen, C. Ferguson, A. Lapedriza, N. Jones, S. Gu, R. Picard, *Way Off-Policy
  Batch Deep Reinforcement Learning of Implicit Human Preferences in Dialog*, 2019,
  [arXiv 1907.00456](https://arxiv.org/abs/1907.00456).
- D. M. Ziegler, N. Stiennon, J. Wu, T. B. Brown, A. Radford, D. Amodei, P. Christiano, G. Irving, *Fine-Tuning
  Language Models from Human Preferences*, 2019, [arXiv 1909.08593](https://arxiv.org/abs/1909.08593).
- A. P. Jacob, D. J. Wu, G. Farina, A. Lerer, H. Hu, A. Bakhtin, J. Andreas, N. Brown, *Modeling Strong and
  Human-Like Gameplay with KL-Regularized Search*, ICML 2022, [arXiv 2112.07544](https://arxiv.org/abs/2112.07544).
- S. Ross, G. J. Gordon, J. A. Bagnell, *A Reduction of Imitation Learning and Structured Prediction to No-Regret
  Online Learning*, AISTATS 2011, [arXiv 1011.0686](https://arxiv.org/abs/1011.0686).
- J. Schulman, F. Wolski, P. Dhariwal, A. Radford, O. Klimov, *Proximal Policy Optimization Algorithms*, 2017,
  [arXiv 1707.06347](https://arxiv.org/abs/1707.06347).
