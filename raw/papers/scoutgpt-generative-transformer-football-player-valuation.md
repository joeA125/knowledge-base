# Modeling Matches as Language: A Generative Transformer Approach for Counterfactual Player Valuation in Football

Miru Hong<sup>1</sup>, Minho Lee<sup>2</sup>, Geonhee Jo<sup>1</sup>, Hyeokje Jo<sup>1</sup>, Pascal Bauer<sup>2,4</sup>, and Sang-Ki Ko<sup>1</sup>

<sup>1</sup> Department of Artificial Intelligence, University of Seoul, Seoul, Republic of Korea
{mirunoyume,geonhee,brandon56,sangkiko}@uos.ac.kr

<sup>2</sup> Institute for Sports and Preventive Medicine, Saarland University, Saarbrücken, Germany minho.lee@uni-saarland.de

<sup>3</sup> Chair for Sports Analytics, Saarland University; Deutscher Fussball-Bund (DFB), Germany pascal.bauer@uni-saarland.de

**Abstract.** Evaluating football player transfers is challenging because player actions depend strongly on tactical systems, teammates, and match context. Despite this complexity, recruitment decisions often rely on static statistics and subjective expert judgment, which do not fully account for these contextual factors. This limitation stems largely from the absence of counterfactual simulation mechanisms capable of predicting outcomes in hypothetical scenarios. To address these challenges, we propose ScoutGPT, a generative model that treats football match events as sequential tokens within a language modeling framework. Utilizing a NanoGPT-based Transformer architecture trained on next-token prediction, ScoutGPT learns the dynamics of match event sequences to simulate event sequences under hypothetical lineups, demonstrating superior predictive performance compared to existing baseline models. Leveraging this capability, the model employs Monte Carlo sampling to enable counterfactual simulation, allowing for the assessment of unobserved scenarios. Experiments on *K League* data show that simulated player transfers lead to measurable changes in offensive progression and goal probabilities, indicating that ScoutGPT captures player-specific impact beyond traditional static metrics.

**Keywords:** Sports Event Sequence Modeling · Counterfactual Transfer Simulation · Player Valuation · Autoregressive Transformer

## 1 Introduction

Evaluating individual contribution is challenging in complex multi-agent environments, where behavior depends not only on an agent’s own ability but also on interactions with surrounding agents and context. Football provides a particularly demanding instance of this problem: player actions are shaped by tactical roles, teammates, opponents, and match state. As a result, player transfer evaluation cannot be reduced to a like-for-like replacement problem, since moving a

player to a new team alters the tactical configuration and reshapes interaction patterns on the pitch. Transfer evaluation therefore requires estimating how a player will behave under this distribution shift, rather than extrapolating directly from past performance alone.

Existing methods only partially address this problem. Traditional valuation frameworks such as Expected Threat (xT) [21] and Valuing Actions by Estimating Probabilities (VAEP) [7,8] quantify the value of observed events, but they do not generate how action sequences would evolve under a new tactical context. Projection systems in other sports typically operate at the level of aggregate season outcomes and therefore do not capture the micro-interactions that shape football actions on the pitch. Recent generative approaches in sports analytics often focus on continuous trajectories, which represent spatial movement but not the tactical semantics of discrete football events [5,24,6]. Prior work has also studied event-based sequence modeling for next-event prediction in football [20,26,15], but these approaches are generally designed to predict observed continuations rather than generate event sequences under hypothetical transfer scenarios. Another line of work estimates On-Ball Value (OBV) by predicting future tokens in an event sequence, enabling counterfactual continuation of play [12]. However, these approaches generate only short fragments of a sequence, limiting value estimation to that small segment of play. In contrast, evaluating transfer scenarios requires generating full event sequences under a new context, enabling value computation over the entire simulated possession.

To address this problem, we introduce ScoutGPT, an autoregressive generative framework for football event streams related to Large Event Models (LEMs) [16]. ScoutGPT treats a match as a structured sequence in which each event is decomposed into discrete attributes through tokenization and predicted sequentially via next-token prediction, conditioned on player identity and match context. Alongside next-action prediction, the model estimates scoring and conceding probabilities at each step, aligning generated sequences with match value (VAEP) and supporting event-level simulation of hypothetical player transfers under new tactical environments [2,9,15].

To summarize, our main contributions are as follows:

- **Structured Event Modeling for Context-Aware Simulation:** We introduce a fine-grained tokenization scheme that decomposes football events into semantic components (e.g., actor, location, and action type). This structure enables ScoutGPT to capture dependencies across event attributes and model football event sequences at a finer granularity.

- **Value-Aware Generative Modeling:** We propose a multi-task learning objective that combines next-token prediction with explicit scoring and conceding probability estimation. This design encourages the model to reflect both event likelihood and match value, and improves predictive performance over non-value-aware variants.

- **Counterfactual Simulation for Player Recruitment:** We show that ScoutGPT can simulate how a player's on-ball contribution profile shifts in a new tactical environment, supporting data-driven analysis of transfer fit.

Overview of the ScoutGPT framework showing the model architecture, prediction heads for goal scoring/conceding probabilities, and the counterfactual simulation process where one player is replaced by another to generate new event sequences.

**Fig. 1.** Overview of the ScoutGPT framework. Our nanoGPT-based Transformer model autoregressively predicts event tokens, enabling counterfactual ‘what-if’ simulations. For instance, replacing Kevin De Bruyne with Scott McTominay could alter actions (e.g., pass/shot) or modify the same action with a different location, outcome, or VAEP.

## 2 Related Work

Our work sits at the intersection of three lines of research: data-driven player valuation, generative modeling of sports event streams, and counterfactual simulation for player transfers.

**Data-Driven Player Valuation** Action-value frameworks have become the standard for data-driven player valuation. VAEP quantifies player contribution by aggregating short-horizon changes in scoring and conceding probabilities across all on-ball actions [7], while EPV decomposes instantaneous possession value into interpretable subcomponents [11]. PlayeRank extends this further by constructing multi-dimensional, role-aware player ratings from large-scale event logs [18]. Collectively, these methods provide strong discriminative estimators for observed behavior. However, they evaluate actions that have already occurred and are not designed to generate counterfactual event sequences under hypothetical team configurations—a requirement that arises when assessing transfer fit.

**Generative Modeling of Sports Data** Seq2Event [20] and Large Event Models (LEMs) [16] frame football events as structured sequential prediction prob-

lems, decomposing each event into multiple attributes and supporting match continuation rollouts from a given game state. NMSTPP [25] and related neural point process models [10,29] extend event-sequence modeling to continuous-time streams with explicit timing and mark distributions. Despite strong short-horizon predictive accuracy, these approaches optimize primarily for sequence likelihood and do not incorporate goal-oriented supervision. Moreover, entity-conditioning for player substitution is either absent or indirect, making it difficult to hold the surrounding context fixed while replacing a specific player—a requirement for counterfactual transfer simulation.

**Sequence Modeling for Event Streams** Transformer-based architectures [23,4] have been applied to sports event streams by treating matches as sequences of discrete tokens to be predicted autoregressively [1,3,17,15]. These models capture complex long-range dependencies across event sequences more effectively than recurrent alternatives. Standard next-token objectives, however, prioritize frequent actions and do not account for the tactical value of decisions or their impact on match outcomes. In addition, unconstrained generation can produce logically inconsistent event transitions over longer horizons. ScoutGPT addresses both limitations by pairing the autoregressive objective with explicit value supervision and VERSA-based constraint masking [13].

**Counterfactual Simulation in Sports** Macro-level transfer forecasting—including baseball projection systems (ZiPS, PECOTA) and soccer ability-curve regression [2]—predicts aggregate season statistics from historical data and age curves, but operates at a coarse granularity that cannot capture event-level tactical dynamics. Graph-based methods represent players as nodes in a relational network to recommend positionally similar replacements [27], but do not model how a player’s behavior would change in a new team context. At the micro level, hierarchical Bayesian xG estimation [14] and causal player evaluation frameworks [22] isolate the counterfactual impact of individual actions, yet they cannot generate the sequential tactical event sequences needed to assess a full transfer scenario. TacEleven [28] leverages language models to explore attacking tactics but focuses on fragmented tactical paths and does not account for the systemic behavioral distribution shift that arises when a player moves to a new team. EventGPT [12] applies generative language modeling to football event sequences, but its generation is limited to short fragments of play, requiring the remaining value to be approximated via residual OBV instead of being computed from fully simulated sequences.

# 3 Methodology

This section describes ScoutGPT, including VERSA-based data verification [13], structured tokenization, and a value-aware multi-task objective.

## 3.1 Data Representation and Verification

Reliable generative modeling requires training data that satisfies football’s logical and physical constraints. Raw event streams often contain inconsistencies such as missing events or temporal ordering errors, so we preprocess all data with VERSA. VERSA uses a formal state-transition model to enforce validity rules and automatically correct anomalies (e.g., inserting missing *Pass Received* events or reordering physically impossible sequences). This preprocessing yields logically consistent training sequences and prevents the model from internalizing annotation errors as valid tactical behaviors.

## 3.2 Problem Formulation

We represent a football match as a collection of discrete episodes, $\mathcal{M} = \{E_1, E_2, \dots, E_K\}$. Each episode $E_k$ is a coherent phase of play (e.g., a possession chain starting from a recovery or set-piece), consisting of a global context $C_k$ and an event sequence $\mathcal{E}_k = \{e_1, e_2, \dots, e_T\}$.

In the raw data, each event $e_t^{\text{raw}} \in \mathcal{E}_k$ is recorded as a 12-dimensional tuple, including explicit labels for goal occurrences:

$$e_t^{\text{raw}} = (h_t, \text{pos}_t, p_t, a_t, x_t^{\text{start}}, y_t^{\text{start}}, x_t^{\text{end}}, y_t^{\text{end}}, \Delta t_t, o_t, \text{gs}_t, \text{gc}_t),$$

where $\text{gs}_t, \text{gc}_t \in \{0, 1\}$ indicate whether a goal was scored or conceded at step $t$. Crucially, to prevent label leakage during the autoregressive generation process, we remove $\text{gs}_t$ and $\text{gc}_t$ from the input sequence. Thus, the model observes a 10-dimensional input tuple:

$$e_t = (h_t, \text{pos}_t, p_t, a_t, x_t^{\text{start}}, y_t^{\text{start}}, x_t^{\text{end}}, y_t^{\text{end}}, \Delta t_t, o_t).$$

Our objective is to model the joint probability $P(e_t, \text{gs}_t, \text{gc}_t \mid C_k, e_{<t})$, jointly predicting next-action components and estimating strategic value (goal-scoring/conceding probabilities) for the current state.

## 3.3 Structured Event Tokenization

To process this hybrid data structure with a Transformer [23], we use a tokenization strategy that separately encodes context and events.

**Context Encoding** The initial match state $C$ is encoded into a fixed sequence of $N_{\text{ctx}} = 56$ tokens, capturing lineups (11 tuples of Position and Player ID per team) and metadata such as period and score. This conditions the model on agent capabilities and match state before event generation.

**Event Encoding** Each 10-dimensional input event $e_t$ is flattened into a sequence of atomic tokens. Continuous variables (coordinates, time deltas) are quantized into discrete bins (0-105). The global sequence $S$ for an episode is constructed by concatenating the context tokens and the flattened event tokens:

$$ S = [\underbrace{c_1, \dots, c_{56}}_{\text{Context}}, \underbrace{s_{1,1}, \dots, s_{1,10}}_{e_1}, \dots, \underbrace{s_{T,1}, \dots, s_{T,10}}_{e_T}], $$

where each event corresponds to a fixed block of 10 tokens.

To handle varying episode length, we impose a maximum sequence length $L_{\text{max}}$. If an episode exceeds this limit (e.g., $T > 100$), we apply a sliding window with fixed stride (e.g., 50 events), creating overlapping chunks while keeping the context window fixed. Explicit position tokens are masked from the input to encourage the model to learn player representations from broader event context rather than direct position labels; we verify this effect in the player-embedding analysis.

## 3.4 ScoutGPT Architecture

We utilize the nanoGPT architecture<sup>4</sup>, an efficient implementation of the GPT-2 decoder-only Transformer [19].

**Backbone** Given the input sequence $S$, we map tokens to dense vectors using a learned token embedding matrix $W_{\text{wte}}$ and add learned absolute positional embeddings $W_{\text{wpe}}$. The model employs a stack of Pre-LayerNorm Transformer blocks.

Let $x^{(l)}$ denote the input to the $l$-th Transformer block. The block computes:

$$ \tilde{x}^{(l)} = x^{(l)} + \text{MSA}(\text{LN}(x^{(l)})) $$
$$ x^{(l+1)} = \tilde{x}^{(l)} + \text{MLP}(\text{LN}(\tilde{x}^{(l)})), $$

where MSA is Causal Multi-Head Self-Attention and MLP is a feed-forward network with GELU activation.

**Auxiliary Heads for Value Estimation** To model action value, we attach two auxiliary classification heads ($\text{Head}_{\text{GS}}$ and $\text{Head}_{\text{GC}}$) to the model's final hidden state $h^{(L)}$:

$$ \text{logit}_t^{\text{GS}} = \text{Head}_{\text{GS}}(h_{t,\text{outcome}}^{(L)}) \text{ and } \text{logit}_t^{\text{GC}} = \text{Head}_{\text{GC}}(h_{t,\text{outcome}}^{(L)}). $$

Unlike the language modeling head, which predicts every next token, these auxiliary heads are activated only at Outcome-token indices ($o_t$), i.e., the last token of each event block. This lets the model estimate immediate scoring and conceding probabilities after each action outcome.

## 3.5 Multi-Task Training Objective

We train ScoutGPT using a composite loss function that balances generative capability with value estimation.

**Generative Loss ($\mathcal{L}_{\text{gen}}$)** The primary objective is the cross-entropy loss for next-token prediction. We apply a masking strategy to ignore padding tokens and, where applicable, mask specific fields (e.g., player IDs) to prevent overfitting or to focus learning on tactical dynamics. In particular, player-ID prediction is excluded because, during inference, player identity is injected based on positional assignment rather than generated through unconstrained autoregressive decoding. This keeps the training objective consistent with the generation procedure:

$$\mathcal{L}_{\text{gen}} = - \sum_{i} \log P(s_{i+1} \mid s_{\leq i}).$$

**Goal-Oriented Auxiliary Loss ($\mathcal{L}_{\text{aux}}$)** We compute auxiliary Cross-Entropy (CE) losses for Goal Scored (GS) and Goal Conceded (GC) predictions at outcome-token positions. For each outcome position, the model predicts whether the current action leads to a goal scored or conceded event:

$$\mathcal{L}_{\text{aux}} = \sum_{t \in \mathcal{T}_{\text{out}}} \left[ \text{CE}(\hat{y}_{t}^{GS}, y_{t}^{GS}) + \text{CE}(\hat{y}_{t}^{GC}, y_{t}^{GC}) \right],$$

where $\mathcal{T}_{\text{out}}$ is the set of indices corresponding to outcometokens, and $y_{t}^{GS}, y_{t}^{GC}$ are the ground-truth labels retrieved from the raw data.

**Total Loss** The final objective is a weighted sum:

$$\mathcal{L}_{\text{total}} = \mathcal{L}_{\text{gen}} + \mathcal{L}_{\text{aux}}.$$

## 3.6 Inference with Structural Constraints

Generating realistic football sequences requires strict game-rule and logical consistency. Standard sampling can produce syntactically valid but physically invalid sequences, so we use State-Dependent Logit Masking with a spatial heuristic for agent resolution.

**Hierarchical Decoding and State-Dependent Masking** The model generates tokens in the fixed hierarchical order defined in Section 3.3. At each step $t$, we apply a validity mask $M_{t}$ to the output logits based on the partial state $s_{<t}$:

$$P(s_{t} \mid s_{<t}) \propto \text{softmax}(z_{t} + M_{t}),$$

where $M_{t}(k) = -\infty$ for tokens that are invalid under the current partial state. The masking strategy is field-dependent. Team tokens are restricted to the valid

team set defined by the context, action tokens are filtered using the VERSA transition validator, and spatial coordinates are restricted to admissible pitch ranges. In addition, several local consistency rules are enforced during decoding. For example, immediate self-receptions such as *Pass Received* or *Ball Received* by the same player as the previous actor are disallowed, and defensive actions such as *Block* and *Interception* are suppressed when possession is retained by the same team. This procedure ensures that token generation respects both event-transition logic and local interaction constraints.

**Spatial-Aware Entity Resolution** After a team token is generated, the model first predicts a compatible position token. Given the resulting (team, position) pair, the player token is then deterministically selected from the corresponding candidate set defined by the context lineup. When multiple candidate players share the same team and position, ScoutGPT selects the candidate whose reference location is closest to the currently generated event coordinates $(x_t, y_t)$:

$$ p^* = \underset{p \in \mathcal{P}(h_t, \text{pos}_t)}{\text{argmin}} \|(x_t, y_t) - \mu_p\|_2, $$

where $\mathcal{P}(h_t, \text{pos}_t)$ denotes the set of candidate players matching the generated team and position, and $\mu_p$ is the reference location associated with player $p$. For known players, this reference is derived from average observed event locations; otherwise, a position-dependent default location is used. Furthermore, when possession is maintained across consecutive events, the generator can preserve the same acting player through an ownership-lock mechanism, which further stabilizes local event continuity.

**Dynamic Episode Termination** Episode generation uses semantic stopping rules rather than only a generic EOS token. Decoding stops when: (1) an EOS token is produced, (2) a newly generated action starts a new episode, (3) a successful *shot* or *penalty kick* results in a goal, or (4) an event from a predefined set of episode-ending actions is generated. These rules prevent generation beyond the logical boundary of the current phase of play.

# 4 Experiments

We evaluate ScoutGPT on *K League* event data across three axes: next-event prediction accuracy, value estimation quality, and counterfactual transfer simulation. We first describe the experimental setup, then report main results and ablations, and finally assess simulation fidelity via self-to-self reconstruction.

## 4.1 Experimental Setup

**Table 1. Summary of the dataset**

**Dataset** We evaluated our model using event data from five seasons of the *K League* 1 and 2, spanning from 2021 to 2025. We standardized all data according to the VERSA event representation. To analyze the continuous flow of a match, we segmented the match events into discrete

<table>
  <thead>
    <tr>
        <th>Statistic</th>
        <th>Value</th>
    </tr>
  </thead>
  <tbody>
    <tr>
        <td>Seasons</td>
        <td>2021 - 2025</td>
    </tr>
    <tr>
        <td>Number of matches</td>
        <td>2,283</td>
    </tr>
    <tr>
        <td>Number of episodes</td>
        <td>222,940</td>
    </tr>
    <tr>
        <td>Events per episode</td>
        <td>27.50</td>
    </tr>
    <tr>
        <td>Number of players</td>
        <td>1,490</td>
    </tr>
  </tbody>
</table>

units, *episodes*. We define an episode as the period between ball-out-of-play instances. Specifically, an episode begins when a player puts the ball back into play following a stoppage—such as a corner kick, free kick, or throw-in—and concludes at the next stoppage. Each episode is composed of two primary components, which are context tokens and event tokens.

The context represents the match state at the onset of the episode, encompassing the list of players on the pitch, their role, the current scoreline for both home and away teams. We structure each individual event into 10 tokens. While the model processes up to 100 events per episode, we implemented a sliding window mechanism for the cases where an episode exceeds this limit. In such instances, the system preserves the context tokens but truncates the earliest event tokens to maintain a sequence length of up to 100 events.

We partitioned the dataset chronologically for model training and evaluation. We utilized data from the 2021, 2022, and 2023 seasons as the training set and used the 2024 season as the validation set. We reserved the 2025 season for final testing. Detailed dataset statistics can be found in Table 1.

**Baselines** We compare our model against sequence-based event prediction approaches derived from prior work on next-event modeling. However, existing models such as Seq2Event and NMSTPP are not directly comparable in their original form, because they predict only coarse-grained action types (e.g., pass, dribble, shot) and do not model additional event attributes such as player identity, spatial coordinates, temporal intervals, or action value. Furthermore, these models typically do not incorporate player conditioning, which is central to our evaluation of transfer fit. To enable a fair comparison, we adapt the Transformer backbone introduced in NMSTPP to our output formulation: preserving the architecture while replacing the output prediction heads with those that match our multi-attribute event representation. We denote this baseline as LEM Transformer.

## 4.2 Main Results

In Table 2, we first evaluate short-horizon value prediction (within 15 seconds) using AUC, Brier score, and ECE. The auxiliary head is better calibrated for GC-related risk signals (higher GC AUC and lower calibration errors), while CatBoost yields stronger GS discrimination and calibration (higher GS AUC with lower ECE). This complementary pattern supports our design choice: coupling autoregressive sequence modeling with explicit value supervision improves goal-related signal quality beyond pure token prediction.

**Table 2.** Performance comparison on predicting goals scored (GS) and goals conceded (GC) within 15 seconds.

<table>
  <thead>
    <tr>
        <th rowspan="2">Method</th>
        <th colspan="3">Goals Scored (GS)</th>
        <th colspan="3">Goals Conceded (GC)</th>
    </tr>
    <tr>
        <th>AUC</th>
        <th>Brier</th>
        <th>ECE</th>
        <th>AUC</th>
        <th>Brier</th>
        <th>ECE</th>
    </tr>
  </thead>
  <tbody>
    <tr>
        <td>ScoutGPT with Auxiliary Head</td>
        <td>0.8344</td>
        <td><strong>0.0069</strong></td>
        <td>0.0024</td>
        <td><strong>0.8153</strong></td>
        <td><strong>0.0016</strong></td>
        <td><strong>0.00081</strong></td>
    </tr>
    <tr>
        <td>CatBoost [8]</td>
        <td><strong>0.8424</strong></td>
        <td>0.0075</td>
        <td><strong>0.0003</strong></td>
        <td>0.8051</td>
        <td>0.0021</td>
        <td>0.00082</td>
    </tr>
  </tbody>
</table>

**Table 3.** Performance comparison across categorical and continuous attributes. For categorical attributes, Accuracy / F1 are reported; for continuous attributes, $R^2$ / MAE are reported.

<table>
  <thead>
    <tr><th rowspan="2">Group</th><th rowspan="2">Attribute</th><th>LEM</th><th>LEM Trans.</th><th>ScoutGPT (Ours)</th></tr>
    <tr>
        <th>VERSA</th>
        <th>VERSA</th>
        <th>VERSA</th>
        <th>SPADL</th>
    </tr>
  </thead>
  <tbody>
    <tr>
        <td rowspan="4">Categorical</td>
        <td>Team</td>
        <td>66.49 / 0.37</td>
        <td><strong>96.10</strong> / 0.49</td>
        <td>92.11 / <strong>0.92</strong></td>
        <td>91.12 / 0.91</td>
    </tr>
    <tr>
        <td>Pos.</td>
        <td><strong>71.01</strong> / 0.32</td>
        <td>44.52 / 0.22</td>
        <td>63.38 / 0.47</td>
        <td>62.56 / <strong>0.51</strong></td>
    </tr>
    <tr>
        <td>Type</td>
        <td>71.46 / 0.20</td>
        <td>76.52 / 0.43</td>
        <td><strong>78.25</strong> / <strong>0.53</strong></td>
        <td>77.88 / 0.50</td>
    </tr>
    <tr>
        <td>Out.</td>
        <td>92.18 / 0.14</td>
        <td>89.02 / 0.61</td>
        <td><strong>94.68</strong> / <strong>0.86</strong></td>
        <td>94.43 / 0.86</td>
    </tr>
    <tr>
        <td rowspan="5">Continuous</td>
        <td>Start $x$</td>
        <td>0.91 / 2.14</td>
        <td>0.78 / 4.59</td>
        <td><strong>0.96</strong> / <strong>0.97</strong></td>
        <td>0.94 / 1.10</td>
    </tr>
    <tr>
        <td>Start $y$</td>
        <td><strong>0.94</strong> / 1.63</td>
        <td>0.78 / 4.09</td>
        <td>0.93 / <strong>1.00</strong></td>
        <td><strong>0.94</strong> / 0.97</td>
    </tr>
    <tr>
        <td>End $x$</td>
        <td>0.48 / 6.27</td>
        <td>0.67 / 8.12</td>
        <td><strong>0.89</strong> / <strong>4.11</strong></td>
        <td>0.87 / 4.24</td>
    </tr>
    <tr>
        <td>End $y$</td>
        <td>0.76 / 5.16</td>
        <td>0.66 / 7.18</td>
        <td><strong>0.82</strong> / <strong>3.85</strong></td>
        <td>0.80 / 4.00</td>
    </tr>
    <tr>
        <td>Time</td>
        <td>0.46 / 1.03</td>
        <td>0.54 / 1.42</td>
        <td><strong>0.71</strong> / <strong>0.75</strong></td>
        <td>0.63 / 0.78</td>
    </tr>
  </tbody>
</table>

Table 3 then reports end-to-end event modeling across event formats and model families. The ScoutGPT-VERSA setting is strongest on structurally important generation targets, including type (78.25 / 0.53), outcome (94.68 / 0.86), end coordinates (0.89 / 4.11 and 0.82 / 3.85), and time (0.71 / 0.75), and also achieves the best start-$x$ accuracy/precision (0.96 / 0.97). Compared with LEM Transformer, gains are especially pronounced on continuous control variables (e.g., start-$x$ MAE 4.59 $\rightarrow$ 0.97 and time MAE 1.42 $\rightarrow$ 0.75), indicating much better spatial-temporal fidelity during rollout. While other settings remain competitive on selected metrics (e.g., team or position accuracy, and start-$y$ tie in $R^2$), ScoutGPT-VERSA provides the most reliable overall trade-off for realistic sequence continuation.

## 4.3 Ablation Study

To isolate what drives these gains, Table 4 compares ablations on lineup and context usage. Removing lineup/context can improve a few isolated metrics (e.g., Position or Time), but the full model remains strongest on most high-impact targets, particularly Team consistency, Type F1, and spatial end-point quality (End $x$, End $y$). This pattern suggests that lineup-aware context is most beneficial for preserving coherent tactical structure, even when simplified variants can

**Table 4.** Ablation study on lineup information and the context block. The Ours column reports the absolute performance of the full model. The remaining columns report changes relative to the full model after removing each component. For Acc., F1, and $R^2$, positive changes indicate improvement, whereas for MAE, negative changes indicate improvement. Green and red denote improvement and degradation, respectively.

<table>
  <thead>
    <tr>
        <th>Group</th>
        <th>Attr.</th>
        <th>Ours</th>
        <th>w/o Context</th>
        <th>w/o Lineup</th>
        <th>w/o Both</th>
    </tr>
  </thead>
  <tbody>
    <tr>
        <td> </td>
        <td>Team</td>
        <td>92.11 / 0.92</td>
        <td>−0.32 / +0.00</td>
        <td>−0.76 / −0.01</td>
        <td>−0.28 / +0.00</td>
    </tr>
    <tr>
        <td rowspan="4">Categorical</td>
        <td>Pos.</td>
        <td>63.38 / 0.47</td>
        <td>+0.16 / +0.05</td>
        <td>−9.01 / −0.16</td>
        <td>−6.88 / −0.13</td>
    </tr>
    <tr>
        <td>Type</td>
        <td>78.25 / 0.53</td>
        <td>+0.05 / −0.02</td>
        <td>+0.56 / −0.03</td>
        <td>−0.44 / −0.03</td>
    </tr>
    <tr>
        <td>Out.</td>
        <td>94.68 / 0.86</td>
        <td>−0.07 / +0.00</td>
        <td>−0.10 / +0.00</td>
        <td>−0.05 / +0.01</td>
    </tr>
    <tr>
        <td> </td>
        <td> </td>
        <td> </td>
        <td> </td>
        <td></td>
    </tr>
    <tr>
        <td rowspan="5">Continuous</td>
        <td>Start $x$</td>
        <td>0.96 / 0.97</td>
        <td>+0.00 / +0.05</td>
        <td>+0.01 / −0.02</td>
        <td>+0.01 / −0.07</td>
    </tr>
    <tr>
        <td>Start $y$</td>
        <td>0.93 / 1.00</td>
        <td>−0.02 / +0.11</td>
        <td>−0.03 / +0.16</td>
        <td>−0.01 / +0.01</td>
    </tr>
    <tr>
        <td>End $x$</td>
        <td>0.89 / 4.11</td>
        <td>−0.01 / +0.14</td>
        <td>−0.01 / +0.09</td>
        <td>−0.02 / +0.16</td>
    </tr>
    <tr>
        <td>End $y$</td>
        <td>0.82 / 3.85</td>
        <td>−0.02 / +0.16</td>
        <td>−0.02 / +0.12</td>
        <td>−0.01 / +0.07</td>
    </tr>
    <tr>
        <td>Time</td>
        <td>0.71 / 0.75</td>
        <td>−0.18 / +0.07</td>
        <td>+0.01 / −0.01</td>
        <td>−0.21 / +0.10</td>
    </tr>
  </tbody>
</table>

fit specific marginals slightly better. Taken together, these results indicate that ScoutGPT improves both event realism and value-awareness, which is critical for reliable counterfactual transfer simulation.

**Table 5.** Average VAEP under different match minute and score-state conditions. The control value corresponds to the fully controlled reference condition and is fixed at 0.02672 for all rows. $\Delta$ denotes the change relative to this reference ($\Delta = \text{Simulated} - \text{Original}$).

<table>
  <thead>
    <tr>
        <th>Minute</th>
        <th>Score state</th>
        <th>Original</th>
        <th>Simulated</th>
        <th>$\Delta$</th>
    </tr>
  </thead>
  <tbody>
    <tr>
        <td rowspan="3">0</td>
        <td>Drawing</td>
        <td>0.02678</td>
        <td>0.023477</td>
        <td>−0.003303</td>
    </tr>
    <tr>
        <td>Trailing</td>
        <td>0.02678</td>
        <td>0.025916</td>
        <td>−0.000864</td>
    </tr>
    <tr>
        <td>Leading</td>
        <td>0.02678</td>
        <td>0.026392</td>
        <td>−0.000388</td>
    </tr>
    <tr>
        <td rowspan="3">40</td>
        <td>Drawing</td>
        <td>0.02678</td>
        <td>0.029647</td>
        <td>+0.002867</td>
    </tr>
    <tr>
        <td>Trailing</td>
        <td>0.02678</td>
        <td>0.029864</td>
        <td>+0.003084</td>
    </tr>
    <tr>
        <td>Leading</td>
        <td>0.02678</td>
        <td>0.029644</td>
        <td>+0.002864</td>
    </tr>
  </tbody>
</table>

To further examine whether ScoutGPT captures context-dependent player impact under controlled perturbations, we conduct a context intervention ablation on episodes of Jeonbuk Hyundai FC and report aggregated team-level episode VAEP under different combinations of match minute and score state. As summarized in Table 5, the intervention produces negative effects across all score states at minute 0, with $\Delta = -0.0033$ when drawing, $\Delta = -0.0009$ when trailing, and $\Delta = -0.0004$ when leading. This pattern suggests that, in the early part of the second half, teams tend to adopt a generally cautious style of play, resulting in lower VAEP than the common control condition regardless of the

Per-episode mean
<table>
  <thead>
    <tr>
        <th>Number of samples</th>
        <th>Per-episode mean ($\cdot 10^{-3}$)</th>
    </tr>
  </thead>
  <tbody>
    <tr>
        <td>1</td>
        <td>1.9</td>
    </tr>
    <tr>
        <td>5</td>
        <td>1.8</td>
    </tr>
    <tr>
        <td>10</td>
        <td>1.6</td>
    </tr>
    <tr>
        <td>20</td>
        <td>1.5</td>
    </tr>
  </tbody>
</table>

Cumulative mean
<table>
  <thead>
    <tr>
        <th>Number of samples</th>
        <th>Cumulative mean</th>
    </tr>
  </thead>
  <tbody>
    <tr>
        <td>1</td>
        <td>1.7</td>
    </tr>
    <tr>
        <td>5</td>
        <td>1.5</td>
    </tr>
    <tr>
        <td>10</td>
        <td>1.4</td>
    </tr>
    <tr>
        <td>20</td>
        <td>1.2</td>
    </tr>
  </tbody>
</table>

**Fig. 2.** Comparison of the mean absolute delta under different numbers of samples. The left shows the per-episode mean absolute delta, while the right shows the cumulative mean absolute delta.

scoreline. The effect is most pronounced when the team is drawing, indicating especially conservative game management in tied situations. In contrast, the effects become consistently positive at minute 40, yielding gains of $\Delta = +0.0029$ when drawing, $\Delta = +0.0031$ when trailing, and $\Delta = +0.0029$ when leading. This indicates that, by the later stage of the second half, teams shift toward more proactive play regardless of the score state, leading to higher VAEP overall. The tendency is strongest when trailing, which is consistent with the intuition that losing teams become the most aggressive in search of recovery. Overall, these results suggest that ScoutGPT captures not only context-dependent adaptation over time but also meaningful differences in strategic behavior across score states.

Figure 2 summarizes the discrepancy between ground-truth (GT) and self-to-self simulated episode VAEP across different numbers of samples. Here, the left histogram refers to the per-episode average VAEP and the right one refers to the cumulative episode VAEP. As the number of samples increases, both the per-episode mean and cumulative absolute differences consistently decrease. This indicates that larger sample sizes lead to more stable self-to-self simulation results and improve the agreement with the GT episode VAEP.

**Table 6.** Cross-season player retrieval performance using player embeddings learned from the 2021–2023 seasons. For each query embedding of a player from one season, retrieval is performed over embeddings from other seasons only. Evaluation is restricted to players who appear in at least two seasons. A query is counted as correct if an embedding of the same player from a different season is retrieved. ⌞

<table>
  <thead>
    <tr>
        <th>Model</th>
        <th>Top-1 (%)</th>
        <th>Top-5 (%)</th>
        <th>Top-10 (%)</th>
    </tr>
  </thead>
  <tbody>
    <tr>
        <td>ScoutGPT (Ours)</td>
        <td><strong>9.20</strong></td>
        <td><strong>21.97</strong></td>
        <td><strong>30.90</strong></td>
    </tr>
    <tr>
        <td>w/o position masking</td>
        <td>8.48</td>
        <td>20.02</td>
        <td>27.05</td>
    </tr>
    <tr>
        <td>Statistics-based embedding</td>
        <td>8.98</td>
        <td>21.36</td>
        <td>30.34</td>
    </tr>
  </tbody>
</table>

# 5 Applications

Beyond next-event prediction, ScoutGPT supports two applications that leverage the learned player representations and counterfactual generation capability: embedding-based player retrieval and hypothetical transfer simulation.

## 5.1 Player Embedding for Similar Player Retrieval

Table 6 provides a quantitative retrieval view of the same embedding space. Here, *position masking* means that during event prediction, position tokens are masked so that future event prediction must rely on player identity and the surrounding event context rather than explicit position labels. This prevents position information from being used as a shortcut bias and encourages embeddings to encode player-specific behavioral patterns. The result is consistently better same-player retrieval (Top-1/Top-5/Top-10: 9.20/21.97/30.90) than both the variant without position masking (8.48/20.02/27.05) and the stats-based embedding baseline (8.98/21.36/30.34), indicating that more meaningful player embeddings are learned when positional bias is controlled.

To examine whether the learned player embeddings capture meaningful clustering structures related to on-field roles, we project the player embedding vectors into two dimensions using $t$-Distributed Stochastic Neighbor Embedding ($t$-SNE), as illustrated in Figure 3. $t$-SNE is a non-linear visualization technique adept at preserving high-dimensional local neighborhood structures, making it highly effective for revealing distinct player archetypes. Each point corresponds to a single player, and colors denote coarse positional categories (e.g., defenders, wing backs, central midfielders, attacking midfielders, wingers, and goalkeepers).

Figure 3 shows the visualization results of player ID embeddings using $t$-SNE. Position information was pre-masked from the input data. The model separated players by position using only the surrounding event context information. This indicates that the model spontaneously learned the players' tactical roles. The hierarchy of clusters within the embedding space aligns with the spatial intuition of actual soccer. Defensive midfielders are positioned between the center back cluster on the right and the attacking midfielder cluster on the left. Fullback clusters appear separated vertically based on their tactical positions on the field.

For instance, Jinsub Park exists between the defensive midfielder and center back clusters. Seungwon Jeong is distributed at the boundary between fullbacks and midfielders. Sangho Na and Masatoshi Ishida also exhibit complex characteristics at role group boundaries. Notably, these embedding patterns align well with their actual playing profiles, as all four players are known for their positional versatility and have performed across multiple roles in real matches.

## 5.2 Hypothetical Transfer Simulation

Table 7 evaluates transfer-fit prediction by comparing simulated post-transfer episode VAEP against ground truth and a naive baseline. Here, the naive projection estimates a player's next-season performance by simply extrapolating

<table>
  <thead>
    <tr>
        <th>Player</th>
        <th>t-SNE 1</th>
        <th>t-SNE 2</th>
        <th>Position</th>
    </tr>
  </thead>
  <tbody>
    <tr>
        <td>Sangho Na</td>
        <td>-20</td>
        <td>6</td>
        <td>Forward</td>
    </tr>
    <tr>
        <td>Zeca</td>
        <td>-35</td>
        <td>2</td>
        <td>Forward</td>
    </tr>
    <tr>
        <td>Masatoshi Isida</td>
        <td>-5</td>
        <td>2</td>
        <td>Attacking Midfielder</td>
    </tr>
    <tr>
        <td>Hojae Lee</td>
        <td>-35</td>
        <td>-1</td>
        <td>Forward</td>
    </tr>
    <tr>
        <td>Jinsub Park</td>
        <td>20</td>
        <td>-1</td>
        <td>Defensive Midfielder</td>
    </tr>
    <tr>
        <td>Seungwon Jeong</td>
        <td>-5</td>
        <td>-9</td>
        <td>Defensive Midfielder</td>
    </tr>
  </tbody>
</table>

**Fig. 3.** t-SNE projection of ScoutGPT player embeddings from the 2024 *K League* season, colored by positional role.

from the previous season’s VAEP while adjusting only for playing time, without accounting for changes in tactical context, team structure, or role. Across all 40 transferred players, ScoutGPT reduces mean absolute error from 1.86 (naive) to 1.28 (simulated), corresponding to a substantial relative error reduction. This indicates that the model captures context-dependent changes in player contribution more reliably than static carry-over assumptions. The representative examples also show that improvements are not confined to one role: gains appear for full-backs, wingers, and central midfielders, suggesting that the framework generalizes across distinct tactical functions.

Jinsu Kim provides a clear example of the practical value of context-aware simulation. The naive estimate (7.00) substantially underestimates his observed post-transfer contribution (GT sum = 11.07), whereas ScoutGPT predicts 11.63, yielding a much smaller error (0.56 vs. 4.07). In other words, our prediction is markedly more accurate than the naive projection for this transfer case. This aligns with real-world outcomes: his move was widely regarded as highly successful relative to initial expectations, and he was appointed team captain in the following season. This case highlights how counterfactual sequence modeling can identify upside that is missed by static baseline forecasts.

# 6 Conclusions

We present ScoutGPT, a player-conditioned and value-aware autoregressive framework for football event modeling. By jointly predicting event attributes and resid-

**Table 7.** Comparison of post-transfer episode VAEP in 2025 using naive and simulated estimates against ground truth (GT). The top 40 transferred players are selected by post-transfer minutes and used to compute the overall average. Only five players are shown for readability. Columns “Naive”, “GT”, and “Sim” report episode VAEP sums, while |GT − Sim| and |GT − Naive| report absolute errors with respect to GT.

<table>
  <thead>
    <tr>
        <th rowspan="2">Role</th>
        <th rowspan="2">Player</th>
        <th colspan="3">Episode VAEP Sum</th>
        <th colspan="2">Absolute Error w.r.t. GT</th>
    </tr>
    <tr>
        <th>Naive</th>
        <th>GT</th>
        <th>Sim</th>
        <th>|GT − Sim|</th>
        <th>|GT − Naive|</th>
    </tr>
  </thead>
  <tbody>
    <tr>
        <td colspan="2">Average</td>
        <td>4.89</td>
        <td>4.74</td>
        <td>4.64</td>
        <td><strong>1.25</strong></td>
        <td>1.88</td>
    </tr>
    <tr>
        <td>Left Back</td>
        <td>Jinsu Kim</td>
        <td>7.00</td>
        <td>11.07</td>
        <td>11.63</td>
        <td><strong>0.56</strong></td>
        <td>4.07</td>
    </tr>
    <tr>
        <td>Left Wing</td>
        <td>Reis</td>
        <td>16.15</td>
        <td>12.19</td>
        <td>12.06</td>
        <td><strong>0.13</strong></td>
        <td>3.96</td>
    </tr>
    <tr>
        <td>Center Back</td>
        <td>Hoik Jang</td>
        <td>3.79</td>
        <td>4.78</td>
        <td>5.83</td>
        <td>1.05</td>
        <td><strong>0.99</strong></td>
    </tr>
    <tr>
        <td>Central Midfielder</td>
        <td>Jihoon Cho</td>
        <td>8.83</td>
        <td>5.09</td>
        <td>5.31</td>
        <td><strong>0.22</strong></td>
        <td>3.74</td>
    </tr>
    <tr>
        <td>Left Back</td>
        <td>Juyong Lee</td>
        <td>4.61</td>
        <td>6.51</td>
        <td>7.31</td>
        <td><strong>0.80</strong></td>
        <td>1.90</td>
    </tr>
  </tbody>
</table>

ual on-ball value, the model captures both local action structure and downstream tactical impact. ScoutGPT outperforms sequence-based baselines in next-event prediction, spatial precision, and future contribution estimation, while learning interpretable player embeddings without positional supervision. Counterfactual player substitution further enables transfer-fit evaluation under new tactical contexts. Future work will integrate tracking-based trajectory signals to better model off-ball behavior and extend generation to full-possession or match-level simulation.

# References

1. Adjileye, A.A.: Risingballer: A player is a token, a match is a sentence, a path towards a foundational model for football players data analytics (2024)

2. van Arem, K., Goes-Smit, F., Söhl, J.: Forecasting the future development in quality and value of professional football players. Applied Sciences (2025). [https://doi.org/10.3390/app15168916](https://doi.org/10.3390/app15168916)

3. Baron, E., Hocevar, D., Salehe, Z.: A foundation model for soccer. arXiv preprint arXiv:2407.14558 (2024). [https://doi.org/10.48550/arXiv.2407.14558](https://doi.org/10.48550/arXiv.2407.14558)

4. Brown, T., Mann, B., Ryder, N., Subbiah, M., Kaplan, J., Dhariwal, P., Neelakantan, A., Shyam, P., Sastry, G., Askell, A., Agarwal, S., Herbert-Voss, A., Krueger, G., Henighan, T., Child, R., Ramesh, A., Ziegler, D., Wu, J., Winter, C., Amodei, D.: Language models are few-shot learners. arXiv preprint arXiv:2005.14165 (2020). [https://doi.org/10.48550/arXiv.2005.14165](https://doi.org/10.48550/arXiv.2005.14165)

5. Capellera, G., Ferraz, L., Rubio, A., Agudo, A., Moreno-Noguer, F.: Transportmer: A holistic approach to trajectory understanding in multi-agent sports. In: Proceedings of the asian conference on computer vision. pp. 1652–1670 (2024)

6. Capellera, G., Ferraz, L., Rubio, A., Alahi, A., Agudo, A.: Jointdiff: Bridging continuous and discrete in multi-agent trajectory generation. arXiv preprint arXiv:2509.22522 (2025)

7. Decroos, T., Bransen, L., Van Haaren, J., Davis, J.: Actions speak louder than goals: Valuing player actions in soccer. In: Proceedings of the 25th ACM SIGKDD

international conference on knowledge discovery & data mining. pp. 1851–1861 (2019)

8. Decroos, T., Bransen, L., Van Haaren, J., Davis, J.: Vaep: An objective approach to valuing on-the-ball actions in soccer (extended abstract). In: Proceedings of the 29th International Joint Conference on Artificial Intelligence (IJCAI). pp. 4696–4700 (07 2020). [https://doi.org/10.24963/ijcai.2020/648](https://doi.org/10.24963/ijcai.2020/648)

9. Dinsdale, D., Gallagher, J.: Transfer portal: Accurately forecasting the impact of a player transfer in soccer. arXiv preprint arXiv:2201.11533 (2022). [https://doi.org/10.48550/arXiv.2201.11533](https://doi.org/10.48550/arXiv.2201.11533)

10. Du, N., Dai, H., Trivedi, R., Upadhyay, U., Gomez-Rodriguez, M., Song, L.: Recurrent marked temporal point processes: Embedding event history to vector. In: Proceedings of the 22nd ACM SIGKDD international conference on knowledge discovery and data mining. pp. 1555–1564 (2016)

11. Fernández, J., Bornn, L., Cervone, D.: A framework for the fine-grained evaluation of the instantaneous expected value of soccer possessions. Machine Learning **110**(6), 1389–1427 (2021)

12. Hong, M., Lee, M., Jo, G., So, J.H., Bauer, P., Ko, S.K.: Eventgpt: Capturing player impact from team action sequences using gpt-based framework. arXiv preprint arXiv:2512.17266 (2025)

13. Jo, G., Kang, M., Lee, K., Lee, M., Bauer, P., Ko, S.K.: Versa: Verified event data format for reliable soccer analytics (2026)

14. Mahmudlu, M., Karakuş, O., Arkadaş, H.: What if they took the shot? a hierarchical bayesian framework for counterfactual expected goals. arXiv preprint arXiv:2511.23072 (2025)

15. Mendes-Neves, T., Meireles, L., Mendes-Moreira, J.: Forecasting events in soccer matches through language. arXiv preprint arXiv:2402.06820 (2024)

16. Mendes-Neves, T., Meireles, L., Mendes-Moreira, J.: A scalable approach for unified large events models in soccer. In: Machine Learning and Knowledge Discovery in Databases (ECML PKDD). vol. 16022 (2026)

17. Mendes-Neves, T., Meireles, L., Moreira, J.: Towards a foundation large events model for soccer. Machine Learning **113**, 8687–8709 (2024). [https://doi.org/10.1007/s10994-024-06606-y](https://doi.org/10.1007/s10994-024-06606-y)

18. Pappalardo, L., Cintia, P., Ferragina, P., Massucco, E., Pedreschi, D., Giannotti, F.: Playerank: data-driven performance evaluation and player ranking in soccer via a machine learning approach. ACM Transactions on Intelligent Systems and Technology (TIST) **10**(5), 1–27 (2019)

19. Radford, A., Wu, J., Child, R., Luan, D., Amodei, D., Sutskever, I., et al.: Language models are unsupervised multitask learners. OpenAI blog **1**(8), 9 (2019)

20. Simpson, I., Beal, R.J., Locke, D., Norman, T.J.: Seq2event: Learning the language of soccer using transformer-based match event prediction. In: Proceedings of the 28th ACM SIGKDD Conference on Knowledge Discovery and Data Mining. pp. 3898–3908 (2022). [https://doi.org/10.1145/3534678.3539138](https://doi.org/10.1145/3534678.3539138)

21. Singh, K.: Expected threat. [https://karun.in/blog/expected-threat.html](https://karun.in/blog/expected-threat.html) (2019), accessed: 2025-07-19

22. Susmann, H.P., D’Alessandro, A.: The counterfactual combine: A causal framework for player evaluation. arXiv preprint arXiv:2602.23233 (2026). [https://doi.org/10.48550/arXiv.2602.23233](https://doi.org/10.48550/arXiv.2602.23233)

23. Vaswani, A., Shazeer, N., Parmar, N., Uszkoreit, J., Jones, L., Gomez, A.N., Kaiser, Ł., Polosukhin, I.: Attention is all you need. Advances in neural information processing systems **30** (2017)

24. Xu, Y., Fu, Y.: Sports-traj: A unified trajectory generation model for multi-agent movement in sports. arXiv preprint arXiv:2405.17680 (2024)

25. Yeung, C., Sit, T., Fujii, K.: Transformer-based neural marked spatio-temporal point process model for analyzing football match events. Applied Intelligence **55**(5) (2025)

26. Yeung, C., Sit, T., Fujii, K.: Transformer-based neural marked spatio temporal point process model for analyzing football match events: C. yeung et al. Applied Intelligence **55**(5), 335 (2025)

27. Yılmaz, Ö.İ., Öğüdücü, Ş.G.: Learning football player features using graph embeddings for player recommendation system. In: Proceedings of the 37th ACM/SIGAPP Symposium on Applied Computing. pp. 577–584 (2022). [https://doi.org/10.1145/3477314.3507257](https://doi.org/10.1145/3477314.3507257)

28. Zhao, S., Ma, H., Pu, Z., Huang, J., Pan, Y., Wang, S., Ming, Z.: Taceleven: generative tactic discovery for football open play. arXiv preprint arXiv:2511.13326 (2025). [https://doi.org/10.48550/arXiv.2511.13326](https://doi.org/10.48550/arXiv.2511.13326)

29. Zuo, S., Jiang, H., Li, Z., Zhao, T., Zha, H.: Transformer hawkes process. In: International conference on machine learning. pp. 11692–11702. PMLR (2020)

# A Appendix

## A.1 Episode Component

$$
\begin{aligned}
\text{<CTX>} & \text{TrueTeam } t^{(H)} p_1^{(H)} u_1^{(H)} \dots p_{11}^{(H)} u_{11}^{(H)} \\
& | \text{ FalseTeam } t^{(A)} p_1^{(A)} u_1^{(A)} \dots p_{11}^{(A)} u_{11}^{(A)} \\
& \phi(\text{period, minute, score, cards}) \text{ </CTX>}
\end{aligned}
\eqno(1)
$$

Table 8. Structure of the context block used in the input sequence. The block encodes team identity, on-pitch lineup, and compact match-state information.

<table>
  <thead>
    <tr>
        <th>Symbol</th>
        <th>Description</th>
    </tr>
  </thead>
  <tbody>
    <tr>
        <td>$t^{(H)}, t^{(A)}$</td>
        <td>Home and away team tokens</td>
    </tr>
    <tr>
        <td>$p_i$</td>
        <td>Position token of the $i$-th player</td>
    </tr>
    <tr>
        <td>$u_i$</td>
        <td>Player token of the $i$-th player</td>
    </tr>
    <tr>
        <td>$\phi(\cdot)$</td>
        <td>Match-state summary function</td>
    </tr>
    <tr>
        <td> </td>
        <td>(period, minute, home goals, away goals, yellow/red cards)</td>
    </tr>
  </tbody>
</table>

## A.2 Multi-Position Players

**Table 9.** Positional distribution of representative multi-role players. Minutes indicate total playing time in each position.

<table>
  <thead>
    <tr>
        <th>Player</th>
        <th>Played positions (minutes)</th>
    </tr>
  </thead>
  <tbody>
    <tr>
        <td>Jinsub Park</td>
        <td>CB (4,383), CDM (2,081), CM (1,849)</td>
    </tr>
    <tr>
        <td>Masatoshi Ishida</td>
        <td>CM (2,282), CAM (1,693), RW (287), CF (237), RF (141), LM (90), LW (77), LF (67)</td>
    </tr>
    <tr>
        <td>Sangho Na</td>
        <td>LW (3,670), RW (1,508), LM (1,073), RM (692), CF (451), LF (405), RF (90)</td>
    </tr>
    <tr>
        <td>Seungwon Jeong</td>
        <td>RWB (2,355), CM (2,021), RW (450), RM (270), RB (199), CAM (90)</td>
    </tr>
  </tbody>
</table>