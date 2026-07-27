---
title: "A Framework for the Fine-Grained Evaluation of the Instantaneous Expected Value of Soccer Possessions"
type: source_summary
tags: [summary, sports-analytics, action-valuation, optical-tracking-data, deep-learning, probability-surface, pitch-control, off-ball, model-decomposition, weak-supervision, calibration, policy-modelling, tactical-analysis, markov-model]
sources: [raw/papers/expected_value_possession_framework.md]
confidence: 0.9
provenance:
  extracted: 80%
  inferred: 15%
  ambiguous: 5%
lifecycle: reviewed
created: 2026-07-27
updated: 2026-07-27
---

# A Framework for the Fine-Grained Evaluation of the Instantaneous Expected Value of Soccer Possessions

**Authors:** [[javier-fernandez]] · [[luke-bornn]] · [[daniel-cervone]]
**Version held:** arXiv:2011.09426v1, 18 November 2020.

The paper this vault has been describing second-hand for three ingest runs. [[expected-possession-value]] has cited it since that page was written, and the [[epv-control-duel-skills-football|Shelopugin]] ingest flagged it as the most-cited football EPV reference the vault lacked.

## Dating, Resolved

The vault previously carried an uncertain note about whether this work is 2019 or 2021. With the primary source in hand the sequence is clear:

| Version | Venue |
|---|---|
| 2019 | MIT Sloan — "Decomposing the immeasurable sport" (conference) |
| **2020** | **arXiv:2011.09426, 18 Nov — this document** |
| 2021 | *Machine Learning* 110(6), 1389–1427 (journal) |

All three are the same line of work at increasing length. Citing "2019" or "2021" is defensible depending on version; the arXiv preprint sits between them. The citation note on [[expected-possession-value]] has been corrected accordingly.

## The Definition

$$EPV_t = \mathbb{E}[G \mid T_t], \qquad G \in \{-1, 1\}$$

where $T_t$ is a tracking-data snapshot and $G$ records **which team scores the next goal** — not whether this possession ends in one. This is the crucial difference from every event-data model in the vault, and it is what makes the metric signed and symmetric: conceding is $-1$, so risk is built into the target rather than bolted on.

Framed as a [[markov-game|Markov decision process]] with one unusual twist: the aim is **not** to find an optimal policy $\pi$, but to estimate value under the *average* policy learned from historical data. See [[policy-modelling]].

## The Central Idea: Decomposition

Rather than fit one model to $\mathbb{E}[G|T_t]$, the expectation is expanded by the law of total expectation over the action set $A = \{\text{pass}, \text{ball drive}, \text{shot}\}$ and each piece estimated separately:

$$EPV_t = \sum_{a \in A} \mathbb{E}[G \mid A = a, T_t]\, \mathbb{P}(A = a \mid T_t)$$

Passes expand further over destination — every location on the field, weighted by a pass *selection* surface — and both passes and ball drives expand again over success and failure:

$$\mathbb{E}[G|A=\rho, D_t, T_t] = \mathbb{E}[G|\cdot, O_\rho{=}1]\mathbb{P}(O_\rho{=}1|\cdot) + \mathbb{E}[G|\cdot, O_\rho{=}0]\mathbb{P}(O_\rho{=}0|\cdot)$$

This yields **nine separately-trained models** recombined into one estimate. The motivation is [[structured-model-decomposition|interpretability]]: a coach can see that passing is worth 0.0252, ball driving 0.0219, shooting 0.0242, and that passing is 29.3% likely — rather than receiving a single opaque 0.0239.

Notably, the decomposition **does not assume** a successful pass is positively rewarded or a failed one negatively; the surfaces are free to learn otherwise, and the risk/reward analysis later exploits this.

## The Modelling Contribution

### Full-field prediction surfaces

The hardest problem the paper identifies: **passes can go anywhere**. Prior work (Power et al., Cervone et al., Hubáček et al.) restricts passing options to the current locations of teammates. That cannot answer "where *should* this ball have gone", which is the question coaches actually ask.

The answer is to predict a value at all $104 \times 68$ locations — a [[probability-surface]] — using [[soccermap]], a fully convolutional architecture operating on layered spatial inputs at 1×, ½× and ¼× resolution, fused back up. Three surface models are trained: pass success probability, pass selection probability, and pass EPV (separately for successful and missed).

### Learning surfaces from one pixel

The striking methodological move. Ground truth exists at exactly **one location** per pass — where the ball actually went. A full surface is nonetheless learned by selecting the predicted value at the observed destination and back-propagating loss only through that single pixel. See [[single-pixel-supervision]].

The convolutional structure is what makes this work: weights are shared across locations, so gradient signal from one pixel updates parameters governing all of them.

### Spatial and contextual features

Two families, built with FC Barcelona match analysts:

- **Spatial** — locations, velocities, distances, angles, plus [[pitch-control]] and pitch influence surfaces.
- **Contextual** — [[dynamic-pressure-lines]] (players clustered into formation lines by complete-linkage clustering, $k=3$), outplayed-player counts, interceptability, and a baseline event-data xG.

Pressure lines encode something coaches already talk about. The analysts consulted regarded **line-breaking passes** as the key driver of goal expectation, and the risk/reward results bear this out.

## Reward Definition

An underrated design choice with direct bearing on the vault's [[action-valuation|credit-assignment axis]].

Possession is **not** ended at a turnover. It ends **when the next goal occurs** — the ball may change hands any number of times in between. All actions since the previous goal are labelled $+1$ if the scoring team took them, $-1$ otherwise.

To limit the resulting noise, a **vanishing threshold** $\epsilon = 15$s is applied: actions more than 15 seconds before the goal get a reward of exactly 0. This is chosen as the average possession duration in the dataset.

So the credit horizon is a **hard temporal cutoff** — a third position alongside VAEP's fixed action window and [[temporal-discounting|Shelopugin's geometric decay]].

## Data and Results

633 Premier League matches (2013/14, 2014/15) from [[stats-perform|STATS LLC]] at 10 Hz: 480,670 passes, 413,123 ball drives, 13,735 shots. A separate 117,948-shot OPTA event dataset trains the baseline xG via XGBoost.

| Model | Loss | ECE |
|---|---|---|
| Pass probability | 0.190 | 0.0047 |
| Ball drive probability | 0.2803 | 0.0051 |
| Pass successful EPV | 0.0075 | 0.0011 |
| Pass missed EPV | 0.0085 | 0.0015 |
| Shot EPV | 0.2421 | 0.0095 |
| **Joint EPV** | **0.0078** | **0.0023** |

Two things matter more than the absolute numbers.

**Calibration survives composition.** The joint estimate's ECE (0.0023) is comparable to its components', which is the paper's key empirical claim — decomposing does not degrade the whole. See [[probability-calibration]].

**It runs in real time.** The slowest model predicts 899 examples/second against a 10 Hz data rate — 89× headroom. EPV can be computed live, which is what the "control room" application depends on. Contrast [[martingale-epv|Cervone et al.'s basketball model]] at 461 processors.

Shot EPV loss is much higher than pass or ball-drive EPV, attributed to the far smaller sample (13,735 shots).

## Applications

Four, and they are the point of the decomposition rather than a coda.

**Real-time control room.** Frame-by-frame component values with an overlaid pass-EPV surface. In the worked example overall EPV is 0.032 while the best available pass would yield 0.112 — the gap between what happened and what was available is the coaching insight.

**Risk/reward distributions by action type.** Kernel density of $EPV_{added}$ across ten action categories:

- Line-breaking passes: the higher the line broken, the wider the distribution and the higher the extremes. Third-line breaks centre around +0.005 with mass across $[-0.025, 0.05]$.
- Actions under pressure: broader, with more negative mass — more likely to lose value, but more upside when the pressure is beaten.
- **Crosses**: the highest tendency to lose significant value, with a fat positive tail. A quantitative answer to a long-running tactical argument.
- **Backward passes**: the narrowest distribution, with nearly half the mass positive — mild vindication.
- Lost balls: usually negative, but *not always* — losing the ball near the opponent's box can raise EPV via rebound chance.

**Pressing Liverpool.** Off-ball and on-ball EPV heatmaps for the five formations most used against Liverpool's (2014/15) buildup. 4-3-3 emerges as the best press for forcing Liverpool wide; 5-3-2 concedes space both inside and behind.

**Growing around David Silva.** Pairwise on-ball and off-ball EPV between Silva and each Manchester City teammate, split by whether Silva is passing or receiving — used to argue which wingers to select depending on whether the aim is to feed Silva or benefit from him.

## Why This Matters to the Vault

**It is the first source here that values off-ball positioning.** Every prior valuation framework is on-ball only, and the vault lists that as a shared limitation across all four modelling tasks. Because pass EPV is estimated at *every* location, the value of a player standing somewhere — the EPV gain if he received the ball there — falls out for free. See [[off-ball-value]].

**It is the interpretability counter-example.** The vault's valuation table has a consistent trade-off: richer state buys sensitivity and costs interpretability. This work has the richest state of any framework here and argues that decomposition recovers interpretability anyway — not by simplifying the model, but by splitting it into parts each of which answers a question a coach already asks.

## Assessment

**Strengths.** The single-pixel learning setup is genuinely clever and well-validated by the calibration results. Real-time performance is a serious practical achievement. The application section is the most convincingly practitioner-facing work in the vault — these are recognisably tools rather than metrics.

**Weaknesses and gaps.**

- **No player-rating output, and no [[split-half-reliability|reliability]] analysis.** The framework values situations and actions, not seasons. It cannot be placed on the xT-versus-VAEP stability axis at all, which limits its use for [[recruitment]].
- **No comparison against any competing framework.** Losses are reported against nothing. There is no benchmark against VAEP, xT, or the basketball model on any shared task.
- **$\epsilon = 15$s is asserted**, justified only as the mean possession length, with no sensitivity analysis — the same criticism the vault makes of Shelopugin's $\gamma$.
- **Tracking data required**, restricting it to wealthy clubs and leagues. Two seasons of one league.
- **Applications are illustrative, not validated.** The Liverpool pressing and Silva analyses are plausible and well-presented, but nothing tests whether acting on them improves outcomes.
- **Handcrafted contextual features throughout**, built with club analysts — which sits awkwardly against [[understanding-football-possessions-path-signatures|the Sig-Model finding]] that engineered geometry can actively harm a representation able to learn it.

## See Also

- [[expected-possession-value]] · [[martingale-epv]] · [[action-valuation]]
- [[soccermap]] · [[probability-surface]] · [[single-pixel-supervision]] · [[fully-convolutional-network]]
- [[pitch-control]] · [[dynamic-pressure-lines]] · [[off-ball-value]]
- [[structured-model-decomposition]] · [[policy-modelling]] · [[probability-calibration]]
- [[expected-goals]] · [[optical-tracking-data]] · [[markov-game]]
- [[javier-fernandez]] · [[luke-bornn]] · [[daniel-cervone]] · [[fc-barcelona]] · [[stats-perform]]
- [[action-valuation-frameworks-compared]]
