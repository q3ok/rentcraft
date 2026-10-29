# Model lineage

Which of our models was trained from which, taken from the models' training configurations. This page grows with
every new model; the story behind each one is in the [research journal](journal/01-from-rule-bots-to-v7.md), and
every model is listed in the [glossary](glossary.md#models).

A solid arrow means that a model continued training from the weights of the previous one; a dotted arrow means that
it learned from a bot or a model by imitation or teacher correction. The percentages are results in our main
protocol (1,320 games; see [how we measure](journal/01-from-rule-bots-to-v7.md#1-the-question-and-how-we-measure)).
Step counts restart with every run; in the Loki series a checkpoint is given by its own steps, with the total since
the start of Loki gen2 in parentheses.

## MLPs: from the bots to mini-evo5

```mermaid
flowchart TD
    GS(["GPT-Style (bot)"])
    TAC(["Tactician (bot)"])
    CALC(["Calculator (bot)<br/>35.7%"])
    SS["simple-small-32x32"]
    SK["simple-kl-32x32"]
    SK2["simple-kl2-32x32"]
    S32["small-32x32"]
    ST["small-trades-32x32"]
    TT["small-32x32-trade-teacher"]
    DAG["small-32x32-trade-dagger"]
    SE["small-evo"]
    E2["mini-evo2"]
    E3["mini-evo3"]
    E4["mini-evo4"]
    E5["mini-evo5<br/>30.2%"]
    GS -.-> SS
    GS -.-> SK
    SK -->|"at 1 M"| SK2
    GS -.-> S32
    S32 -->|"at 6 M"| ST
    S32 --> TT
    S32 --> DAG
    TAC -.-> TT
    TAC -.-> DAG
    S32 -->|"+1 M steps with trade blocks"| SE
    SE --> E2 --> E3 --> E4
    CALC -.-> E3
    E4 -->|"at 8 M"| E5
```

simple-small-32x32 and simple-kl-32x32 both started from their own imitation of "GPT-Style"; simple-kl2-32x32 is the
one that continued an earlier run.

## Transformers: from mini-evo5 and "Calculator" to njord

```mermaid
flowchart TD
    E5(["mini-evo5 (MLP)<br/>30.2%"])
    CALC(["Calculator (bot)<br/>35.7%"])
    U48["3x48 student"]
    U64["3x64 student"]
    U464["4x64 student"]
    L1["Loki gen1"]
    L2["Loki gen2"]
    L2B["Loki gen2b"]
    L2C["Loki gen2c"]
    L2D["Loki gen2d"]
    L2E["Loki gen2e"]
    L2F["Loki gen2f"]
    L3A["Loki gen3a"]
    L3B["Loki gen3b"]
    L3C["Loki gen3c"]
    SCR["Encoder screening A-D<br/>best: A (old encoder) 20.1%<br/>best v6: B 15.9%"]
    MJ["Mjolnir<br/>11.0% before RL"]
    NJ["first njord attempt,<br/>njord-gen0"]
    E5 -.-> U48
    E5 -.-> U64
    E5 -.-> U464
    CALC -.-> U48
    CALC -.-> U64
    CALC -.-> U464
    U64 --> L1 --> L2
    L2 -->|"at 1.5 M"| L2B
    L2B -->|"at 2 M (3.5 M total)"| L2C --> L2D
    L2D -->|"at 1 M (6.5 M total)"| L2E
    L2E -->|"at 1 M (7.5 M total)"| L2F
    L2D -->|"at 1 M (6.5 M total)"| L3A
    L3A --> L3B
    L3A --> L3C
    CALC -.-> SCR
    E5 -.-> MJ
    CALC -.-> MJ
    E5 -.-> NJ
    CALC -.-> NJ
```

Several Loki runs also used "Calculator" as a trade teacher during RL, and Loki gen2f played against Loki gen2c and
Loki gen2d as rival models; njord-gen0 started from scratch, not from the first njord attempt. Those links are left
out of the diagram for clarity.
