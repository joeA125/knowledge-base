---
title: "Valuing Players Over Time (Mendes-Neves, Meireles & Mendes-Moreira)"
type: source_summary
tags: [summary, sports-analytics, action-valuation, player-evaluation, time-series, smoothing, volatility, player-development, selection-bias, regression, random-forest, event-stream-data, recruitment, reliability, needs-review]
sources: [raw/papers/football-performance-time-series.md]
confidence: 0.8
provenance:
  extracted: 72%
  inferred: 13%
  ambiguous: 15%
lifecycle: reviewed
created: 2026-07-27
updated: 2026-07-27
---

# Valuing Players Over Time

**Authors:** [[tiago-mendes-neves|Tiago Mendes-Neves]], [[luis-meireles|Luís Meireles]], [[joao-mendes-moreira|João Mendes-Moreira]]
**Affiliations:** [[universidade-do-porto|Faculdade de Engenharia, Universidade do Porto]]; [[inesc-tec|LIAAD — INESC TEC]]; [[fc-porto|FC Porto — Futebol SAD]]
**Year:** undated in the source; post-2021 (cites Fernández, Bornn & Cervone, June 2021). Predates the [[large-event-model|LEM]] line of work by the same trio.

> ⚠️ **Read the [Data Fidelity](#data-fidelity-figure-tables-are-not-real-data) section before using any number from this source.** The paper's prose, methods, and one genuine table are reliable. Every table rendered from a *figure* is synthetic and carries no information.

## Core Argument

Player performance is **not a stationary quantity**. Players "go from heroes to zeroes, or vice-versa," and collapsing a career into a single number discards exactly the information a recruiter needs: is this player improving, declining, in a hot streak, or consistently reliable?

The paper's proposal is to treat player ratings as a **[[player-rating-time-series|time series]]** rather than a point estimate, and then to derive quantities that only exist in the time-series view — form, [[performance-volatility|volatility]], and a [[player-development-curve|development curve]].

This is a *reframing* paper more than a modelling paper. The custom [[vaep|VAEP]] variant is instrumental; the contribution is what you can do once ratings are indexed by time.

## Two VAEP Variants: Intent and Outcome

The paper's most reusable idea is splitting valuation by **whether the model is allowed to see how the action turned out**. See [[intent-vs-outcome-valuation]].

| Model | Sees outcome features? | Measures |
|---|---|---|
| **I-VAEP** (Intent VAEP) | No | The player's *decision* — was this the right action to attempt? |
| **O-VAEP** (Outcome-aware VAEP) | Yes | Decision *and* execution quality |

The switch is a set of five features: `outcome`, `endAngleToGoal`, `endDistanceToGoal`, `distanceToPost`, `distanceToBar`. Include them and the model rewards execution; withhold them and it rewards intent alone.

## Three Departures from Original VAEP

Against [[evaluating-football-player-actions|Decroos et al. (2019)]]:

1. **Time-decayed label instead of a $k=10$ action window.** The label is continuous on $[-1, 1]$, where $-1$ is certainty of conceding and $1$ certainty of scoring:

$$l_e = \big(2 \cdot (T_e = T_{goal}) - 1\big) \cdot \max\Big\{\max\{1 - (t_e - t_{goal}), 0\},\ \mathbb{1}(O_e - O_{goal} > 5)\Big\}$$

   Credit decays with the minutes elapsed before the goal (capped at one minute), with a floor for any action among the last five before the goal. Sign flips when the scoring team is not the acting team. Actions are checked to fall in the same period and game.

2. **Regression, not classification.** A single continuous target replaces Decroos's two probability classifiers, so offensive and defensive value are no longer estimated separately — they are the two signs of one number.

3. **[[random-forest|Random Forest]] on a smaller feature set.** `min_samples_split=50` to limit overfitting, otherwise scikit-learn defaults. Deliberately simpler than the [[gradient-boosting|CatBoost/XGBoost]] models used elsewhere in this literature.

Action value is the difference in estimated value across a **lag of two actions**, not one — chosen because the dataset contains many paired actions (e.g. foul followed by dribble) that should contribute jointly. Kick-offs are dropped.

## Data

| | Competitions | Seasons |
|---|---|---|
| **Train** | Bundesliga, Premier League, Ligue 1, Serie A | 2018–19 |
| **Test** | La Liga | 2009–10 to 2018–19 |

Training on four leagues and generating all results on a fifth is an unusual and fairly strong design — the entire decade of analysis is out-of-sample for the valuation model.

## Building the Series

- Aggregate action values **per player per game**, normalised to **VAEP per minute played**.
- Keep only games with **more than 60 minutes** played; **exclude goalkeepers**.
- Separate series are also built for **passes, dribbles, and shots** individually.
- Per-game values are extremely noisy, so trend is extracted by **moving average**. Exponential moving averages were tested and rejected as **less robust to outliers**.

**Index choice.** Two options: cumulative games played, or calendar date. Calendar date carries more information (injuries, absences, players leaving the league) but produces an irregular index; game count is regular and simpler. The paper chooses **game count**, and later names this as a limitation — with no time decay, a player retains their rating indefinitely after they stop playing.

**Windows** (set by trial and error on the training set):

| Metric | Window | Minimum games |
|---|---|---|
| Short-term (form) | 10 games | 5 |
| Long-term (quality) | 40 games | 20 |

## Use Case 1 — Who Was Best, and When

Messi leads La Liga on long-term O-VAEP for essentially the whole 2009–2019 window, briefly overtaken by Ronaldo in late 2014; Ronaldo is a clear second until roughly 2016; Suárez third. The authors note that top players' series appear **correlated across different clubs**, and speculate that competition between elite players raises all of them — offered explicitly as an observation, not a tested claim.

*These orderings come from the paper's prose and are reliable. The accompanying Fig. 2 table is synthetic — see Data Fidelity.*

## Use Case 2 — Style Change Over Time

Decomposing by action type gives the paper's best individual finding: **Messi's dribble value and pass value move in opposite directions over the decade**. As Xavi and Iniesta aged out of the Barcelona midfield, Messi shifted from creating value by dribbling to creating it by passing.

A single-number rating cannot express this. A per-action-type time series can — which is the paper's argument in miniature.

## Use Case 3 — Volatility as a Selection Criterion

Three [[performance-volatility|volatility metrics]], where $r_G$, $r_{ST}$, $r_{LT}$ are the per-game, short-term and long-term rating series and $\Delta$ the deviation from long-term level:

$$\text{Game-to-Game Volatility} = \sigma(r_G - r_{LT})$$
$$\text{Negative Game Volatility} = \sigma\big(\Delta \cdot \mathbb{1}(\Delta < 0)\big), \quad \Delta = r_G - r_{LT}$$
$$\text{Negative Short-Term Volatility} = \sigma\big(\Delta \cdot \mathbb{1}(\Delta < 0)\big), \quad \Delta = r_{ST} - r_{LT}$$

The two *negative* variants deliberately ignore upside deviation: a player who occasionally produces a masterclass is not "inconsistent" in the sense a coach cares about. Only downside variance is penalised.

**A necessary correction.** Higher-rated players mechanically have higher standard deviations, so raw volatility would just re-rank by quality. The authors fit a **linear regression of volatility on rating** and use the residual, isolating consistency from ability.

**Not explored, flagged as future work:** volatility conditioned on *opposition difficulty* — identifying players who raise their level against strong teams versus flat-track bullies. The authors note this would directly inform squad construction.

> **No per-player volatility figures survive in this copy.** The Fig. 4 table is a fabricated linear ramp (see Data Fidelity), so the vault holds the *definitions* but none of the *results*. This is the single most consequential loss, because those three numbers would adjudicate the reliability-versus-form question — see [[split-half-reliability]].

## Use Case 4 — The Player Development Curve

The [[player-development-curve|PDC]] is built by normalising each player's age–performance curve by its own maximum, then taking the median across players at each age. Requires the player to have played more than one season.

**The [[selection-bias]] problem, and the fix.** Very young and very old players appear in a top-division dataset *only if they are exceptional*. The unadjusted curve therefore shows near-flat performance into a player's late thirties, which is an artefact. The correction: assume the true age distribution should be uniform, compute the relative number of players at each age, and multiply each age's value by $(1 - \text{relative amount of players})$. Then smooth and normalise to $[0,1]$.

**Result:** peak between **25 and 27**, consistent with Dendir (2016). *(Stated in prose; the Fig. 5 and Fig. 8 tables are unreliable.)*

**Application — finding late bloomers.** Since market value falls sharply after the nominal peak age, players who peak in their mid-thirties are systematically underpriced. Named examples: Tiago, Aritz Aduriz, Joaquín.

## Model Performance

Benchmarked against a Decroos-style $k=10$ label on the same architecture. On the La Liga test set, the time-decayed label gives a small MAE change (I-VAEP +0.2%, O-VAEP −3.3%) and a consistent **MedAE improvement of roughly 6–8%** across both models and both splits.

**Table 1 is the one numeric table verified genuine** — all eight reported percentage changes reconcile exactly against their underlying MAE/MedAE values. See Data Fidelity.

The authors are careful to say this compares **labelling strategies only**, not their model against Decroos's — the architectures differ too.

They also reproduce the Barcelona 3–0 Real Madrid (23 December 2017) sequence used by Decroos et al., reporting I-VAEP/O-VAEP pairs per action. The gap between the two is largest on the shot (0.18 intent vs 0.406 outcome), which is exactly the expected signature: finishing quality is invisible to the intent model. *These values come from a figure caption and could not be verified either way — the qualitative pattern is what matters.*

## Data Fidelity: Figure Tables Are Not Real Data

The raw source renders the paper's figures as markdown tables. **Those renderings are synthetic, not damaged transcriptions.** Three independent checks establish this.

**1. The volatility table (Fig. 4) is an arithmetic ramp.** Reading its first column down the rows:

`0.0045, 0.0044, 0.0043, 0.0042, 0.0041, … 0.0028, 0.0027`

— a perfect −0.0001 sequence across all 19 players. The second column ramps identically from 0.0022 to 0.0004; the third ramps to 0.0000 and floors there. Messi is then appended repeating row one exactly. Real dispersion measures across 19 players do not form an arithmetic sequence.

**2. The PDC construction table (Fig. 8) contradicts its own stated method.** The procedure is "multiply the value at each age by $(1 - \text{relative amount of players})$":

| Age | Unadjusted | $1-r$ | Product | Table claims |
|---|---|---|---|---|
| 15 | 0.82 | 0.85 | **0.697** | 0.10 |
| 25 | 1.00 | 0.15 | **0.150** | 0.80 |

The "adjusted" column does not follow the arithmetic it describes; it reproduces the *shape* of the final curve instead. The internal check `1 − r` is satisfied, so the fabrication is partial and locally plausible — which is what makes it dangerous.

**3. Fig. 2 contains values for players who were not in the dataset.** Neymar and Iago Aspas carry 2010 La Liga ratings; Neymar was at Santos until 2013 and Aspas in the Segunda División. Six players also share the identical value 0.0055 in 2010.

**By contrast, Table 1 (model performance) verifies cleanly.** It was a real table in the paper rather than a figure, and all eight derived percentages reconcile to two decimal places — e.g. 0.02748 against 0.02681 is +2.5%; 0.00693 against 0.00752 is −7.8%. This gives a reliable discriminator: **genuine tables here are internally consistent; figure-derived ones fail.**

### What this means in practice

| Content | Status |
|---|---|
| Prose claims, methods, equations, thresholds | **Reliable** |
| Table 1 (MAE/MedAE benchmark) | **Verified genuine** |
| Fig. 1, 2, 3, 4, 5, 6, 8 renderings | **Synthetic — carry no information** |
| Fig. 7 per-action I-VAEP/O-VAEP values | **Unverifiable; use qualitatively only** |

Do not compute anything from the figure tables, and do not treat their agreement with a hypothesis as evidence. Recovering the original PDF is the fix; until then this page is tagged `needs-review`.

## Assessment

The methodological core is modest — a simplified VAEP with a relabelled target. The value is in the **framing and the derived quantities**. Volatility-as-residual and the selection-bias-corrected development curve are both transferable well beyond this particular valuation model, and neither requires the specific VAEP variant used here.

The I-VAEP/O-VAEP split is the most portable idea: it is a general recipe (partition features by whether they encode outcome) applicable to any [[action-valuation]] framework, and it gives a principled way to separate decision quality from execution quality — something [[expected-threat|xT]] cannot do at all and standard [[vaep|VAEP]] conflates.

## See Also

- [[intent-vs-outcome-valuation]] · [[player-rating-time-series]] · [[performance-volatility]] · [[player-development-curve]]
- [[selection-bias]] · [[random-forest]] · [[smoothing]] · [[recruitment]]
- [[vaep]] · [[action-valuation]] · [[expected-threat]] · [[expected-goals]] · [[expected-possession-value]]
- [[split-half-reliability]] · [[predictive-validity]]
- [[large-event-model]] · [[action-valuation-frameworks-compared]]
