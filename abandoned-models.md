# Abandoned models

## Abandoned branch: Loki

Loki was an experiment in handing the whole training to an AI: all decisions about the next runs, their settings and
the points to continue from were made without supervision by the language model Claude Opus. The branch was abandoned: its best model, Loki gen3a, was only about 2 points above Loki gen2d
(see the [journal](journal/01-from-rule-bots-to-v7.md#7-transformers-the-loki-series)), and the lessons from it went into
the encoder and training work that led to njord. It is kept here as a record.

In this series a checkpoint is given by its own steps, with the total since the start of Loki gen2 in parentheses
(see "M steps" in the [glossary](glossary.md#how-we-train)); for example, Loki gen2d at 1 M (6.5 M total). The results
of each Loki are in the [glossary](glossary.md#transformers); they come from the old protocol (330 games), not the
main one.

```mermaid
flowchart TD
    E5(["mini-evo5 (MLP)"])
    CALC(["Calculator (bot)"])
    U64["3x64 student"]
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
    E5 -.-> U64
    CALC -.-> U64
    U64 --> L1 --> L2
    L2 -->|"at 1.5 M"| L2B
    L2B -->|"at 2 M (3.5 M total)"| L2C --> L2D
    L2D -->|"at 1 M (6.5 M total)"| L2E
    L2E -->|"at 1 M (7.5 M total)"| L2F
    L2D -->|"at 1 M (6.5 M total)"| L3A
    L3A --> L3B
    L3A --> L3C
```

Several Loki runs also used "Calculator" as a trade teacher during RL, and Loki gen2f played against Loki gen2c and
Loki gen2d as rival models. Those links are left out of the diagram for clarity.

## Abandoned learning tree of transformers from mini-evo5

```mermaid
flowchart TD
    E5(["mini-evo5 (MLP)<br/>30.2%"])
    CALC(["Calculator (bot)<br/>35.7%"])
    U48["3x48 student"]
    U64["3x64 student"]
    U464["4x64 student"]
    LK["Loki series<br/>abandoned branch, see above"]
    SCR["Encoder screening A-D<br/>best: A (old encoder) 20.1%<br/>best v6: B 15.9%"]
    MJ["Mjolnir<br/>11.0% before RL"]
    E5 -.-> U48
    E5 -.-> U64
    E5 -.-> U464
    CALC -.-> U48
    CALC -.-> U64
    CALC -.-> U464
    U64 --> LK
    CALC -.-> SCR
    E5 -.-> MJ
    CALC -.-> MJ
    style LK stroke-dasharray: 5 5
```

The first njord attempt also started from scratch and stopped in the rules phase; it did not descend from this tree.
