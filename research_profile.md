# Research Profile

This file documents the researcher's interests, background, and goals. The routine reads `config.json` for machine-readable settings; this document is the human-readable narrative that explains *why* those settings are what they are.

---

## Background

Computational biomedical research, with a primary focus on lung cancer. Core methodological interests are statistical learning and machine learning applied to clinical oncology — specifically, survival analysis and treatment-effect modeling for adjuvant chemotherapy decisions in non-small cell lung cancer (NSCLC).

A secondary methodological thread is using LLMs for meta-analysis screening and automated evidence synthesis — applying NLP at the systematic-review layer of clinical evidence.

## Current research focus

- Survival analysis (Cox PH, deep survival models, accelerated failure time)
- ML for individualized treatment recommendation in adjuvant chemotherapy
- Causal inference in observational oncology data — treatment-effect heterogeneity
- LLMs for systematic review screening and evidence synthesis
- Lung cancer prognosis modeling, especially NSCLC

## Adjacent fields the digest should bridge to

These are fields the researcher does not actively work in but wants to be informed by, because the *bridge between them and clinical oncology* is where the most valuable insights are expected:

- **Mathematical oncology** — ODE, PDE, and agent-based mechanistic tumor models
- **Tumor microenvironment biology** — stromal, vascular, and immune compartments
- **Immuno-oncology** — with specific interest in CAR T-cell therapy
- **Tumor immunology** — T-cell exhaustion, antigen escape, neoantigen prediction
- **Scientific machine learning** — physics-informed neural networks, neural ODEs, hybrid mechanistic-ML models
- **Protein engineering and ML-driven protein design** — relevant to CAR construct design
- **Self-driving labs and autonomous experimentation** — closed-loop active learning, relevant for cellular therapy optimization

## The bridge

The highest-priority synthesis target is the **triple intersection**:

> **Statistical learning** (clinical ML in oncology)
> ↔ **Mechanistic biological models** (tumor-immune dynamics, ODE/PDE)
> ↔ **Immunology** (CAR T, T-cell biology, antigen escape)

A connection that touches all three is more valuable than a connection that bridges only two. Examples of what counts as a high-value bridge:

- A neural ODE survival model that embeds a mechanistic tumor-immune dynamical system as its inductive bias
- A mechanistic CAR T expansion model whose parameters are inferred via causal inference from clinical observational data
- An LLM-based evidence synthesis pipeline that surfaces conflicts between mechanistic theory and clinical observation
- A protein-engineering approach to CAR construct design that uses self-driving lab active learning, with treatment-effect heterogeneity feeding back into the design loop

Lower priority but still valuable: pairwise bridges where one paper sits in clinical/statistical oncology and the other in any of the adjacent fields.

## Author watchlist

Each author was selected because their work sits at one or more of the bridge nodes:

| Author | Why | Affiliation |
|---|---|---|
| **Helen M. Byrne** | Tumor microenvironment, multiscale mathematical oncology | Oxford |
| **Trachette Jackson** | Tumor-immune dynamics, ODE/PDE models of cancer | Michigan |
| **Franziska Michor** | Computational cancer biology, evolution, treatment resistance | Dana-Farber |
| **Carl H. June** | CAR T-cell therapy pioneer | UPenn |
| **Crystal L. Mackall** | CAR T in solid tumors, T-cell biology | Stanford |
| **Christopher Rackauckas** | Scientific ML, neural ODEs, hybrid mechanistic+ML | MIT |
| **Iain J. Marshall** | Automated systematic reviews, NLP for evidence synthesis | King's College London |
| **Philip A. Romero** | ML-driven protein engineering, directed evolution, self-driving labs | Duke |

Watched-author papers bypass the credibility filter and always appear in the digest if published in the lookback window. They are still scored, and may appear in cross-domain connection pairs.

## Quality philosophy

**Quality over quantity is a hard principle, not a soft preference.**

- The weekly digest caps at 10 papers total. If fewer than 10 papers pass the quality bar in a given week, the digest is shorter — not padded with mediocre work.
- Foundation-model papers (Evo2, AlphaFold, ESM, etc.) are capped at 2 per digest, and are included only when they introduce a new method, biological insight, or non-obvious reframing. Pure benchmarking and naive applications are filtered out at scoring time.
- The connection-type priority is **methodological reframing > method transfer > theoretical unification > analogical scaffolding**. A paper that changes how you'd think about a problem is more valuable than a paper that gives you a new tool to apply.
- A short digest with a "quality note" naming the shortage is the correct output for a quiet week. Padding is a failure mode.

## What to filter out

Topics that should not appear in the digest, even if they score well otherwise:

- General LLM benchmarking unrelated to biomedical use
- Non-cancer dynamical systems with no clear biomedical relevance
- Pure application papers — applying existing methods to new datasets without methodological insight
- Incremental benchmark improvements without new ideas
- Purely engineering applications of physics-informed ML

## Digest cadence

Weekly, Mondays. The lookback window is the past 7 days. The total digest cap is 10 papers including watched-author updates, connection-pair papers, and standalone papers.
