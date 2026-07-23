# The path to a goal: Understanding soccer possessions via path signatures

David Hirnschall<sup>1*†</sup> and Robert Bajons<sup>1†</sup>

<sup>1</sup>Institute for Statistics and Mathematics, Vienna University of Economics and Business, Welthandelsplatz 1, Vienna, 1020, Vienna, Austria.

\*Corresponding author(s). E-mail(s): david.hirnschall@wu.ac.at;

Contributing authors: robert.bajons@wu.ac.at;

<sup>†</sup>These authors contributed equally to this work.

**Abstract**

We present a novel framework for predicting next actions in soccer possessions by leveraging path signatures to encode their complex spatio-temporal structure. Unlike existing approaches, we do not rely on fixed historical windows and handcrafted features, but rather encode the entire recent possession, thereby avoiding the inclusion of potentially irrelevant or misleading historical information. Path signatures naturally capture the order and interaction of events, providing a mathematically grounded feature encoding for variable-length time series of irregular sampling frequencies without the necessity for manual feature engineering. Our proposed approach outperforms a transformer-based benchmark across various loss metrics and considerably reduces computational cost. Building on these results, we introduce a new possession evaluation metric based on well-established frameworks in soccer analytics, incorporating both predicted action type probabilities and action location. Our metric shows greater reliability than existing metrics in domain-specific comparisons. Finally, we validate our approach through a detailed analysis of the 2017/18 Premier League season and discuss further applications and future extensions.

**Keywords:** Path signatures, sports analytics, possession value, deep learning

# 1 Introduction

A fundamental component of soccer analysis is the concept of possessions, which, broadly speaking, can be defined as consecutive sequences of actions executed by a single team. A game of soccer is naturally divided into these possessions. Hence, studying them has attracted a lot of interest recently. While several approaches have focused on evaluating players by estimating an outcome of possessions (Van Roy et al. 2020; Fernández et al. 2021; Bajons and Hornik 2025), a recently popularized approach is to evaluate possessions by predicting next actions. To this end, researchers have made a connection between natural language processing (NLP) and soccer events, which are both inherently sequential (Simpson et al. 2022; Mendes-Neves et al. 2024; Yeung et al. 2025). Hence, to solve the problem of predicting future actions, techniques predominantly used in NLP tasks such as recurrent neural networks (RNN, Elman 1990), gated recurrent units (GRU, Cho et al. 2014), long-short-term memory (LSTM, Hochreiter and Schmidhuber 1997), and transformers (Vaswani et al. 2023). More specifically, Simpson et al. (2022) use a transformer-based model architecture to predict next actions, whereas Zhang et al. (2022) use LSTMs to predict match results. Other approaches have made similar connections, but rely on either ordinary architectures such as sequentially concatenated multilayer perceptrons (MLP, Mendes-Neves et al. 2024), or structures motivated by spatio-temporal point-processes (Yeung et al. 2025). While these approaches are intriguing, they ignore the nature of soccer possessions. Soccer possessions are spatio-temporal paths of different lengths, i.e., actually ordered time series where the length of the possession contains critical information. Hence, the previously mentioned approaches, which use a fixed historic window of past actions paired with sequential machine learning architectures, are an unnatural choice for this task. On the one hand, a randomly chosen fixed historic window of past actions may take into account actions from past possessions. That is, this approach almost inevitably takes into account multiple possessions, leading either to including both teams' actions or, if solely considering one team's action sequences, potentially substantial spatio-temporal discontinuities by omitting the opponents' intervening possessions. On the other hand, the fixed window of actions ignores the length of the current possession completely. A more natural way is to only include actions within one possession, or at least within a certain amount of past possession. However, this leads to the problem of time series of different lengths and irregular sampling frequencies, which the previously mentioned sequential architectures are not able to handle. In machine learning, this leads to the need for effective feature extraction techniques, aiming to identify salient characteristics of a time series informative for forecasting. Handcrafted features are usually based on domain-specific knowledge or general statistics, such as mean, variance, skewness, and kurtosis. While often useful, these features typically fail to take into account the temporal order of events, which is crucial in sports analytics.

To address these challenges, we propose to model possessions via path signatures. The signature of a path is a sequence of iterated integrals that gradually encodes spatio-temporal details until the path obtained by interpolation of data can, under mild conditions, be uniquely characterized. Informally, they can be viewed as feature extraction tool for time series that preserves the order and interaction of events over

time. This makes signatures a natural choice for predicting actions. First, they are able to handle variable-length time-series, removing the need to fix a historic window size and avoid including potentially irrelevant or misleading information from distant possessions. Additionally, signatures retain all critical information without requiring geometrical or spatially handcrafted feature engineering, such as angles or differences used in previous works (Simpson et al. 2022; Yeung et al. 2025). More precisely, our approach encodes all relevant information from the full possession by signatures to predict both the next action and its location in the form of $(x, y)$-coordinates. This approach allows us to evaluate possessions in detail, leading to various use cases for coaches. Furthermore, using signatures offers a general and mathematically valid alternative to existing approaches for a wide range of applications in sports with a similar spatio-temporal structure as soccer.

Building on our enhanced action prediction model, we devise a novel possession value metric, which takes into account both the predicted action type and the predicted location from our model. We provide evidence in favor of our possession value metric over existing ones and show that our proposed signature action prediction model is practically more relevant than the benchmark model from Simpson et al. (2022). We use our model and the possession evaluation metrics to get in-depth insight into the 2017/18 Premier League season. Furthermore, we complement this analysis with additional use cases for our model in practice.

The main contributions of this paper can be summarized as follows:

* We propose a novel and intuitive framework for action prediction in soccer using path signatures and only information from the recent possession.

* We show various advantages of our approach: (i) it is a natural approach for the complex spatio-temporal structure of possession, (ii) it allows the usage of time series of different lengths and sampling frequencies without the need for transformations to predict future actions, (iii) it is computationally more efficient, and improves prediction loss over existing approaches.

* We provide a novel way to evaluate possessions, taking into account action type probabilities and predicted locations in an intuitive and interpretable way. The metric is compared to existing ones in a domain-specific evaluation, outperforming them across various comparison setups.

* We present a detailed analysis of the 2017/18 Premier League season and a number of practical applications for our action prediction model.

# 2 Literature review

## 2.1 Soccer modeling

The advancements in data collection in recent years have revolutionized the way sports are analyzed. Nowadays, there is an increasing demand in professional sports for data scientists able to analyze highly granular data such as event stream and tracking data (Lolli et al. 2025). Paired with advanced computational power and potent machine learning algorithms, the availability of new data has led to a vast range of data-driven analyses in the field of soccer analytics.

While earlier work on analyzing soccer centered around modeling specific actions, such as passes (Szczepański and McHale 2016; Chawla et al. 2017; Håland et al. 2019), or shots (Robberechts and Davis 2020; Anzer and Bauer 2021), more recently, a focus has been on considering possessions, i.e., consecutive sequences of actions of one team, as unit of interest in soccer. Van Roy et al. (2020) for example, compare and expected threat (xT) model to a model which values each action in a possession by estimating goal probabilities (VAEP). Both these models allow for explicitly assigning values to possessions and can be used for player evaluation. On the other hand, Fernández et al. (2021) estimate expected possession value (EPV) to analyze various situations in soccer. The authors derive the EPV as a composition of different models, each of which is estimated via neural networks. Bajons and Hornik (2025) use possessions as units to estimate the player strengths via different regularized regression methods.

Due to the spatio-temporal nature of possessions in soccer, researchers found similarities between analyzing possessions and analyzing sentences in natural language processing (NLP). Hence, powerful methods with a proven success record in NLP, such as LSTMs and transformers, have been suggested for modeling sports data. In Basketball, Sicilia et al. (2019) use LSTMs to analyze possession, whereas Watson et al. (2021) use these models to analyze possessions in rugby union. In soccer, Simpson et al. (2022) use transformers to account for the sequential nature of the game to analyze possessions. Their work is closest to our approach, and we use their model as a benchmark for our advanced action prediction model. Building on their work, Yeung et al. (2025) use a neural marked spatio-temporal point process (NMSTPP) architecture to model possessions. While they expand on Simpson et al. (2022) by accounting for interevent time, they also consider a fixed historic window size and similar time series for the predictions of actions. Additionally, they refrain from directly modeling the $(x, y)$-coordinates of the next action and instead group the coordinates into predefined zones on the pitch. Mendes-Neves et al. (2024) take a different route and derive a large event model (LEM), trying to predict a much more detailed set of 33 action types, location on a grid similar to Yeung et al. (2025), and time elapsed on a prespecified grid.

To the best of our knowledge, path signatures have not yet been used for modeling soccer possessions. Path signatures are, however, a natural choice for irregularly sampled spatio-temporal data such as possessions. Furthermore, they are able to handle possessions of different lengths while at the same time extracting all relevant information from possessions without the need for additional feature engineering. Due to their tremendous impact in other domains, as discussed in Section 2.2, they have the potential to enhance models for various tasks in sports analytics. Our framework is only one example for the usage of signatures in sports, and the results of this work provide a glimpse of the power of signatures.

## 2.2 Signatures for feature encoding

Signatures, originally introduced in the 50s (Chen 1954), gained popularity in the mathematical community through their connections to the Rough Path theory developed by Terry Lyons (Lyons 1998). Recently, (log)-signatures have gained significant traction in machine learning applications as a non-parametric feature encoding and

dimensionality reduction technique for time series data (Sturm 2025). Signatures are transformations that map a path to an infinite-dimensional sequence of statistics, capturing increasingly fine details of the underlying path. When appropriately augmented, the signature characterizes the path uniquely (Hambly and Lyons 2010; Boedihardjo et al. 2016). They have been successfully utilized in various domains, including human action recognition tasks (Yang et al. 2017, 2022) and gesture recognition (Shi et al. 2025), predicting a diagnosis of Alzheimer's disease (Moore et al. 2019) and detecting early signs of depressive and manic episodes in patients with bipolar disorder (Kormilitzin et al. 2017) as well as various applications in finance (Cuchiero et al. 2023; Bayer et al. 2023). In the context of generative modeling, Buehler et al. (2020) utilized path signatures paired with Variational Autoencoders to generate financial time series in a small data environment. Liao et al. (2019) combined log-signatures with recurrent neural networks to learn neural stochastic differential equations and Ni et al. (2021) measured time series similarities using a new metric based on signatures.

# 3 Methodology

## 3.1 Path signatures

When working with time series data, especially with irregularly sampled time intervals, as is typical for possession data streams, path signatures have emerged as powerful non-parametric feature maps. They provide a unique and concise representation of trajectories while encoding their structural properties in a mathematically principled way (Hambly and Lyons 2010; Chevyrev and Kormilitzin 2025).

**Definition 1** For a continuous path with finite variation $X : [s, t] \to \mathbb{R}^d$ and a set of all multi-indexes $I = \{(i_1, \dots, i_k) \mid k \ge 0, i_1, \dots, i_k \in \{1, \dots, d\}\}$ the signature is defined by a collection of iterated integrals of $X$, by

$$S(X)_{s,t} = \left( 1, S(X)_{s,t}^1, \dots, S(X)_{s,t}^d, S(X)_{s,t}^{1,1}, S(X)_{s,t}^{1,2}, \dots \right),$$

with

$$S(X)_{s,t}^{i_1, \dots, i_k} = \int_{s < t_k < t} \dots \int_{s < t_1 < t_2} dX_{t_1}^{i_1} \dots dX_{t_k}^{i_k}.$$

By convention, the 0-th entry is equal to 1.

The truncated signature of $X$ of order $M$ is defined as the finite collection of all terms $S(X)_{s,t}^{i_1, \dots, i_M}$, where the maximum length of the multi-index is $M$. The truncation error at level $M$ decays with factorial speed as $\mathcal{O}(1/M!)$ Lyons et al. (2007). Computational examples illustrating how to compute the signature for simple paths are provided in Chevyrev and Kormilitzin (2025).

To enrich the original possession stream, path augmentations are applied. In this work, we used a time augmentation and a visibility transformation, which add an extra dimension to encode information on the timestamps and the starting point, respectively. For more details on commonly used augmentations, we refer to Morrill et al. (2020). Under appropriate augmentations, path signatures uniquely determine the path up to tree-like equivalences (Hambly and Lyons 2010).

Originally defined for continuous paths with bounded variation, the signature transformation has been extended to discrete paths by linear interpolation Chevyrev and Kormilitzin (2025). For piecewise linear paths, computing signatures no longer involves integrals. Instead, according to Chen’s identity (Chen 1958), they can be calculated from the contributions of each line segment of the path, defined as

$$ S(X)_{t,t+1}^{i_1,i_2,\dots,i_k} = \frac{1}{k!} \prod_{j=1}^{k} \left( X_{t+1}^{i_j} - X_t^{i_j} \right), $$

where $X^{i_j}$ is the $i_j$-th coordinate of the path $X$.

Log-Signatures, denoted by $LogSig(X)_{s,t} = log(S(X)_{s,t})$ provide parsimonious representations of path signatures by significantly reducing their dimension. Due to the shuffle product property of signatures (Chevyrev and Kormilitzin 2025, Theorem 1.14), every polynomial function on signatures can be expressed as a linear combination of signature elements. This leads to repeated information in the signature representation, e.g. $S(X)_{s,t}^{i,i} = \frac{1}{2} \left( S(X)_{s,t}^i \right)^2$. The log-signature essentially removes such redundancies, capturing the same information in fewer terms.

They are robust to irregularly sampled observations, where actions may not occur at uniform time intervals, unaffected by the length of the time series (Morrill et al. 2020) and form a bijective map, uniquely determining the path up to tree-like equivalences (Hambly and Lyons 2010). For a detailed discussion of log signatures in machine learning, including their dimension reduction capabilities, we refer to Liao (2022); Morrill et al. (2020).

## 3.2 Model architecture

We process historical actions independently from other input features and propose log-signatures combined with weighted averages to effectively encode spatio-temporal structures of the underlying time series data. This enables the use of feed-forward neural networks instead of relying on computationally demanding transformer architectures, which yield faster training, improved scalability, and interpretability.

In contrast to previous work (Simpson et al. 2022; Yeung et al. 2025), our model relies solely on previous actions, the latest difference in goals, and the raw triplets $(x, y, T)$, directly observed during a match. We deliberately avoid handcrafted geometric or temporal features, such as angles to the goal or action durations, as all information needed is naturally encoded by the path signature. Moreover, such features are often noisy, overly task-specific, or require extensive domain knowledge. Hence, if not carefully tailored to the model and task, they may introduce bias or overemphasis on specific aspects, potentially harming performance, as seen in Table B7, where we compare our model to a variant that includes the additional features mentioned in Simpson et al. (2022).

More precisely, the time series of the continuous features $(x, y, T)$ are encoded as log-signatures, the sequence of actions is passed through an embedding layer, which maps each time step to a vector of dimension $emb\_dim$. To account for the temporal structure and the decreasing importance of features over time, we compute a

```mermaid
graph LR
    subgraph Encode_features [Encode features]
        direction TB
        Actions[ActionsDim: 1 x pos length] --> Emb[Embedding Layer 1 ... Embedding Layer n]
        Emb --> WA[Weighted averageDim: 7]
        
        CF[Continuous FeaturesDim: 3 x pos length] --> LS[Calculate Log-SignaturesDim out: 61]
        LS --> LogSig[Log SignatureDim: 55]
        
        Scrad[Last scradDim: 1]
    end

    WA --> Conc[ConcatenateDim out: 63]
    LogSig --> Conc
    Scrad --> Conc

    subgraph FFNN [Feed forward neural network]
        Conc --> DL1[Dense LayerDim out: 256]
        DL1 --> LReLU1[Leaky ReLU 0.2]
        LReLU1 --> DL2[Dense LayerDim out: 256]
        DL2 --> LReLU2[Leaky ReLU 0.2]
        LReLU2 --> DL3[Dense LayerDim out: 9]
    end

    subgraph Predictions [Predictions]
        DL3 --> x[x]
        DL3 --> y[y]
        DL3 --> A1[Action 1]
        DL3 --> Adots[...]
        DL3 --> A7[Action 7]
    end
```

**Fig. 1**: Sig-Model architecture in three stages. Input actions are embedded and concatenated with the log-signatures of continuous features $(x, y, T)$ and the current scrad (score advantage) value. Passing through the main neural network yields predictions for both the next action location and the distribution of next action type.

weighted average, assigning higher weights to more recent actions. The weights are set to the inverse of each action’s position in the sequence and are scaled to sum to one, thereby emphasizing more recent events while still incorporating information from earlier actions.

Both components are then concatenated together with the current score advantage, i.e., number of goals ahead or behind (referred to as *scrad*), and passed through a feed-forward neural network, as illustrated in Figure 1.

The network contains one input, one output, and one hidden layer of 256 nodes and utilizes leaky rectified linear unit (LeakyReLU) activations with parameter $\alpha = 0.2$ between layers. The output vector of length 9 is split into action logits of length 7 and $x$ and $y$ predictions. Our proposed model pipeline in its used hyperparameter configuration is illustrated in Figure 1. Details on the performed hyperparameter grid search can be found in 3.3.3.

Following Simpson et al. (2022), we use a two-part cost function. The error of the predicted ball’s location is measured by the root mean squared error (RMSE) in $x$ and $y$. Predicted action probabilities are evaluated using a weighted cross-entropy loss (CEL), where goals, change of possession events, and end-of-match events are considered as purely contextual rather than indicators of style, hence are given zero weights within the CEL function. As final cost function, we use the weighted sum

$$L(\theta) = RMSE_{(x,y)} + \lambda CEL_{actions},$$ <sup>(1)</sup>

where $\lambda$ is treated as a hyperparameter.

## 3.3 Empirical evaluation

In this section, we study the performance of our proposed methodology in various settings and compare it to a benchmark model from the literature. All computations were performed on a 2023 MacBook Pro equipped with an M3 Pro chip, 36GB of

RAM, and 12 CPU cores. Code for reproducing all results is available at [https://github.com/Rob2208/sig_actions](https://github.com/Rob2208/sig_actions).

## 3.3.1 Data and preprocessing

For model evaluation, we utilize WyScout Open Access Dataset Pappalardo et al. (2019), including match data from the 2017/18 season of the top men’s leagues in England, Germany, France, Italy, and Spain, enhanced by matches from the Euro 2016 and World Cup 2018. For the comparison of models in Section 3.3.4, we follow Simpson et al. (2022) and select 138 out of the total 1941 matches across leagues and success rates. However, we perform various permutations of this approach in additional validations of the model in Appendix A.

Opposed to existing work, we are not stipulating an arbitrary historic window of actions, but rather follow a more natural approach and treat every possession as its own self-contained sequence following tactical considerations, to align with how coaches and analysts evaluate the game. This approach is enabled by using path signatures, as they encode spatio-temporal properties regardless of a sequence’s length and sampling frequency. Further, we will only rely on input features that are directly observable during a match: performed actions, current goal difference, and the raw location-time triplet $(x, y, T)$ scaled by their respective maximum value. Hence, we deliberately avoid hand-crafted geometric and temporal features such as shot angles, action durations, or spatial differences in $x$ and $y$ coordinates. To support our choice of pre-processing and for comparison purposes, we examined how the length of the historic window and the exclusion of additional features impact the Seq2Event transformer model from Simpson et al. (2022). As shown in Table B8, the length of the historic window appears to be arbitrarily long without a clear indication of the optimal value. Furthermore, removing all handcrafted features and relying solely on observable features $(x, y, T)$, yields a clear reduced prediction accuracy of the Seq2Event transformer model, indicating a need for additional geometrical features when using their approach (see again Table B8).

We evaluate our model’s forecasting ability by starting at different points within a possession, namely 4th, 5th, 6th, 7th, or 8th action. Beginning to forecast only when at least a certain number of actions have occurred not only ensures that enough information for a realistic forecast is provided but also filters out short possessions, which, in general, provide little tactical context, hence are not meaningful for evaluating possession utilities.

For each test possession, we form a path $(x, y, T)$, apply a time and an invisibility-reset augmentation, followed by a piecewise linear interpolation, and calculate the log-signature of order 3. The order is chosen based on the hyperparameter tuning results in 3.3.3. Categorical action features are embedded directly in our proposed model. In order to ensure comparability to previous works (Simpson et al. 2022; Yeung et al. 2025), actions were grouped into seven categories: pass (‘p’), dribble (‘d’), cross (‘x’), shot (‘s’) and goal scored (‘g’), possession end (‘_’), and match end (‘@’). The ‘@’ category is only a helper category in order to distinguish between matches, but is not relevant for prediction. While there are different approaches to group actions (see

e.g. Mendes-Neves et al. 2024), we agree with previous work that this is a suitable representation of relevant soccer actions.

### 3.3.2 Evaluation metrics

Model evaluation is conducted on four losses. We present the test loss (1) split in its components, the RMSE in $x$ and $y$, and a weighted CEL, to evaluate the predicted location and action probabilities, respectively.

To assess the full predictive distribution of actions $A$ with values $a \in \mathcal{A}$, we investigate the Brier score

$$ \text{Brier Score} = \frac{1}{N} \sum_{i=1}^{N} \sum_{k=1}^{K} (f_{i,k} - \mathbf{1}_{\{a_i=k\}})^2, \tag{2} $$

where $f_{i,k}$ denoted the predicted probability that action $a_i$ belongs to class $k$. Moreover, the Kullback-Leiber (KL) divergence between the predicted and the empirical zone-conditioned action distribution is presented. For actions $A$ occurring in zone $z_i$ following the empirical distribution $\mathbb{Q}_{z_i} = (q_{z_i}(a))_{a \in \mathcal{A}}$ and predicted distribution $\mathbb{P}_{z_i} = (p_{z_i}(a))_{a \in \mathcal{A}}$ the KL divergence is given by

$$ KL(\mathbb{P}, \mathbb{Q}) = \sum_{a \in \mathcal{A}} p_{z_i}(a) \log \frac{p_{z_i}(a)}{q_{z_i}(a)}. \tag{3} $$

Whereas CEL and Brier score reward selecting the correct action, we include the KL divergence to measure the tactical plausibility of predicted actions. It shows whether a model is in line with typical soccer-specific expectations in the given pitch zone. The zones are chosen according to Figure 2. We used domain knowledge to create the zones and focused on the offensive side of the pitch, as more relevant actions happen in this half. The zones are chosen such that they clearly favor a specific set of actions. Looking at Figure 2, we expect more shots in Zone 7 than in other zones, whereas we expect more crosses in zones 4 and 6. This information is also contained in the empirical zone distribution. Hence, a model that is close to this distribution (in terms of KL divergence) and at the same time performs well in the other evaluation metrics indicates a good balance between predictive accuracy and tactical reasonability.

### 3.3.3 Hyperparameter search

A comprehensive model selection and hyperparameter grid search was performed, evaluating every parameter combination on four loss metrics across various datasets. Models were scored on possession sequences using $n_r \in \{3, 4, 5, 6, 7\}$ recent action features, which corresponds to forecasting from the 4<sup>th</sup>, 5<sup>th</sup>, 6<sup>th</sup>, 7<sup>th</sup> or 8<sup>th</sup> action onwards. In total, the search procedure resulted in 480 different configurations across all combinations of hyperparameters, data splits and values of $n_r$. We conduct the hyperparameter tuning process in two stages, with tested parameters listed in Table A1. In the first phase, we optimize the CEL scaling factor $\lambda$, the signature order $M$, and the batch size $n_{batch}$ (in that order). Each parameter was evaluated on the initial dataset described in 3.3.1. The best value for each hyperparameter was selected in a

A standard soccer pitch divided into 8 relevant zones: Zone 0 (defensive half), Zone 1 (bottom wing), Zone 2 (center), Zone 3 (top wing), Zone 4 (bottom attacking wing), Zone 5 (center attacking), Zone 6 (top attacking wing), and Zone 7 (penalty area).

**Fig. 2:** A standard soccer pitch divided into 8 relevant zones. For teams in possession, the play direction is from left to right.

robust way based on the average losses across the grid of the remaining hyperparameters. While Table A2, shows a superior performance for $\lambda = 1$ across all $n_r$, Table A3 indicates the best performance for signature order $M = 3$. Although higher signature order lowered the KL for longer possessions, $M = 3$ achieved the best overall trade-off and was chosen. Finally, a batch size of $n_{batch} = 32$ slightly decreased the Brier score but significantly increased the RMSE, hence it was discarded (see Table A4).

In the second phase, we retained the six most promising hyperparameter combinations from stage one as model candidates (see Table A5) and assessed their robustness. For this, we train and test each model across 10 randomly generated train-test splits, obtained by varying sets of teams and matches included in the corresponding datasets.

Average scores across the generated data splits for each model candidate are presented in Table A6 separately for each value of $n_r$. The final loss comparison resulted in a signature order of $M = 3$, a CEL scaling factor of $\lambda = 1$, a batch size of $n_{batch} = 4$, and a hidden dimension of 256.

### 3.3.4 Comparison with baseline model

Finally, we benchmark our proposed model (Sig-Model) against the transformer-based Seq2Event model in Simpson et al. (2022). For comparison reasons, we report losses

and runtimes in seconds for the identical set of 138 matches as used by Simpson et al. (2022) (see 3.3.1). Results are presented in Tables 1 and 2 respectively.

**Table 1**: Losses of our Sig-Model and the Seq2Event model for $n_r \in \{3, 4, 5, 6, 7\}$, where Test loss = MSE + CEL on the test dataset. Best values are highlighted in bold.

<table>
  <thead>
    <tr>
        <th>n<sub>r</sub></th>
        <th>Model</th>
        <th>Test loss</th>
        <th>MSE</th>
        <th>CEL</th>
        <th>Brier</th>
        <th>KL</th>
    </tr>
  </thead>
  <tbody>
    <tr>
        <td rowspan="2">3</td>
        <td>Sig-Model</td>
        <td><strong>0.2084</strong></td>
        <td><strong>0.1598</strong></td>
        <td>0.0486</td>
        <td><strong>0.7968</strong></td>
        <td><strong>0.1117</strong></td>
    </tr>
    <tr>
        <td>Seq2Event</td>
        <td>0.2129</td>
        <td>0.1652</td>
        <td><strong>0.0477</strong></td>
        <td>0.8034</td>
        <td>0.1121</td>
    </tr>
    <tr>
        <td rowspan="2">4</td>
        <td>Sig-Model</td>
        <td><strong>0.2096</strong></td>
        <td><strong>0.1601</strong></td>
        <td>0.0495</td>
        <td><strong>0.7938</strong></td>
        <td><strong>0.1125</strong></td>
    </tr>
    <tr>
        <td>Seq2Event</td>
        <td>0.2144</td>
        <td>0.1658</td>
        <td><strong>0.0486</strong></td>
        <td>0.8054</td>
        <td>0.1163</td>
    </tr>
    <tr>
        <td rowspan="2">5</td>
        <td>Sig-Model</td>
        <td><strong>0.2108</strong></td>
        <td><strong>0.1601</strong></td>
        <td>0.0507</td>
        <td><strong>0.7877</strong></td>
        <td><strong>0.1146</strong></td>
    </tr>
    <tr>
        <td>Seq2Event</td>
        <td>0.2142</td>
        <td>0.1652</td>
        <td><strong>0.0490</strong></td>
        <td>0.7984</td>
        <td>0.1177</td>
    </tr>
    <tr>
        <td rowspan="2">6</td>
        <td>Sig-Model</td>
        <td><strong>0.2121</strong></td>
        <td><strong>0.1606</strong></td>
        <td>0.0515</td>
        <td><strong>0.7858</strong></td>
        <td><strong>0.1155</strong></td>
    </tr>
    <tr>
        <td>Seq2Event</td>
        <td>0.2141</td>
        <td>0.1646</td>
        <td><strong>0.0495</strong></td>
        <td>0.7954</td>
        <td>0.1183</td>
    </tr>
    <tr>
        <td rowspan="2">7</td>
        <td>Sig-Model</td>
        <td><strong>0.2134</strong></td>
        <td><strong>0.1612</strong></td>
        <td>0.0522</td>
        <td><strong>0.7881</strong></td>
        <td><strong>0.1169</strong></td>
    </tr>
    <tr>
        <td>Seq2Event</td>
        <td>0.2152</td>
        <td>0.1642</td>
        <td><strong>0.0510</strong></td>
        <td>0.7998</td>
        <td>0.1202</td>
    </tr>
  </tbody>
</table>

Our model outperforms the benchmark in the test loss obtained by the cost function 1, as it reduced the MSE significantly while almost maintaining the CEL. Further, the Sig-model achieved a higher Brier score and a lower deviation from the empirical zone-conditioned action distribution in terms of KL divergence across all forecasting starting points in the given possessions.

**Table 2**: Runtimes in seconds of our Sig-Model and the Seq2Event model for each $n_r \in \{3, 4, 5, 6, 7\}$. Best values are highlighted in bold.

<table>
  <thead>
    <tr>
        <th>n<sub>r</sub></th>
        <th>Model</th>
        <th>Runtime in sec</th>
    </tr>
  </thead>
  <tbody>
    <tr>
        <td rowspan="2">3</td>
        <td>Sig-Model</td>
        <td><strong>280.67</strong></td>
    </tr>
    <tr>
        <td>Seq2Event</td>
        <td>687.97</td>
    </tr>
    <tr>
        <td rowspan="2">4</td>
        <td>Sig-Model</td>
        <td><strong>253.30</strong></td>
    </tr>
    <tr>
        <td>Seq2Event</td>
        <td>391.62</td>
    </tr>
    <tr>
        <td rowspan="2">5</td>
        <td>Sig-Model</td>
        <td><strong>237.21</strong></td>
    </tr>
    <tr>
        <td>Seq2Event</td>
        <td>383.09</td>
    </tr>
    <tr>
        <td rowspan="2">6</td>
        <td>Sig-Model</td>
        <td><strong>227.45</strong></td>
    </tr>
    <tr>
        <td>Seq2Event</td>
        <td>323.51</td>
    </tr>
    <tr>
        <td rowspan="2">7</td>
        <td>Sig-Model</td>
        <td><strong>194.71</strong></td>
    </tr>
    <tr>
        <td>Seq2Event</td>
        <td>250.31</td>
    </tr>
  </tbody>
</table>

On the tested dataset, the runtimes were significantly reduced. Especially when starting forecasting early in the possession, i.e., with an increased number of forecasts, we observe a decrease in runtime of a factor of about 2.5. Note that the previously reported runtime of approximately 45 minutes for the Seq2Event model (Simpson et al. 2022; Yeung et al. 2025) was reduced to the values reported in Table 2, due to technical improvements in the Pytorch implementation, particularly in how training samples are stored and loaded.

# 4 Practical application

## 4.1 Possession value

A popular use case for action prediction models is to evaluate possessions. Due to the dynamic nature of soccer, it is difficult to assess the value of a possession sequence as, in general, most possessions lack a valuable outcome, such as a goal. Simpson et al. (2022) assess possessions by analyzing the attacking weight of a possession. Intuitively, a possession that ends in, or contains, an attacking action (regardless of whether being successful or not) is more valuable than a possession without attacking intent. Hence, Simpson et al. (2022) assign a value to possession based on the cumulative predicted probabilities for the actions shot and cross, which can be seen as an attacking event. Their poss-util metric for a possession $p$ is defined as

$$ \text{poss-util} = \sum_{i=1}^{N_p} P_i(\text{cross, shot}) \cdot c, \tag{4} $$

where $N_p$ is the number of actions in possession $p$, $P_i(\text{cross, shot})$ is the predicted probability of a shot or cross for action $i$, and $c \in \{-1, 1\}$ is a factor depending on whether a shot, or cross actually happened or not. While poss-util allows for insights into a team's effectiveness and playing style, it is not a natural choice of metric for an action prediction model that outputs predictions for actions and locations on the pitch. Leveraging the information on the location of the actions within a possession allows for a more detailed evaluation of possession. To this end, Yeung et al. (2025) develop the HPUS metric for possessions:

$$ \text{HPUS} = \sum_{i=1}^{N_p} \phi(N_p + 1 - i) \text{HAS}_i . \tag{5} $$

HPUS is a weighted average of action values (HAS) for each action, where the weighting function $\phi$ emphasizes actions that happen later during the possession. The action score depends on a value for the predicted action type, a value for the predicted location, and the elapsed time between actions. In more detail, HAS is defined as

$$ \text{HAS} = \frac{\sqrt{AV \cdot ZV}}{t}, \tag{6} $$

where the action value (AV) is given as

$$ \text{AV} = 0 \cdot P(\text{possession loss}) + 5 \cdot P(\text{dribble, pass}) + 10 \cdot P(\text{cross, shot}), \quad (7) $$

and similarly, the location value (ZV) depends on zones and is given as

$$ \text{ZV} = 0 \cdot P(\text{Area}_0) + 5 \cdot P(\text{Area}_1) + 10 \cdot P(\text{Area}_2), \quad (8) $$

and $t$ represents the interevent time. While HPUS makes use of the location and the elapsed time of the actions (additional to action types), it is not clear how the factors (0, 5, and 10) and the zones ($\text{Area}_0$, $\text{Area}_1$, and $\text{Area}_2$) were chosen. In fact, Yeung et al. (2025) even suggest that these values can be adjusted arbitrarily. However, this makes HPUS difficult to interpret and also questions the reliability of the metric, as different values could potentially lead to different results.

To address the above deficiencies, we devise a new metric for possession value. Our metric is built upon well-known and widely used metrics for evaluating actions. We follow a similar approach to HPUS and assign values to each action in a possession. However, instead of multiplying values for the location on top of that, we make the value location-dependent. In particular, our location-based possession value (LPV) assigns a location-based action value (LAV) to each action of a possession as

$$ \text{LAV} = \widehat{\text{xG}} \cdot P(\text{shot}) + \widehat{\text{xT}} \cdot P(\text{dribble, pass, cross}), \quad (9) $$

where $\widehat{\text{xG}}$ is a simple expected goals model (Robberechts and Davis 2020; Anzer and Bauer 2021) evaluated at the predicted $(x, y)$-location, and $\widehat{\text{xT}}$ is an expected threat model (Singh 2019; Van Roy et al. 2020) again evaluated at the predicted $(x, y)$-location. We provide more details on the computation of these models in Appendix C. The proposed DPV for the full possession is then computed similarly to before by simply summing over all actions in a possession.

$$ \text{LPV} = \sum_{i=1}^{N_p} \text{LAV}_i . \quad (10) $$

Using (10) to obtain a value for a possession thus takes into account the action type predictions, while at the same time using the predicted location to assign each action a value that is interpretable in terms of widely used metrics in soccer. Finally, we briefly discuss some possible adaptations of (9) and (10). First, in comparison to HPUS, we do not take the elapsed time between actions into account. While the intuition from Yeung et al. (2025) that faster actions may lead to more threatening possessions seems reasonable, they do not provide a data-driven justification for that claim. To maintain comparability in model comparison (compare section 3.3.4), we did not explicitly model the time an action takes. While it would be relatively straightforward to extend our framework to include time, we leave it open for future work. Second, we also did not incorporate a weighting function $\phi$ as in (5). Again, we believe that the idea from

Yeung et al. (2025) is sensible, but it is not clear whether their choice of $\phi$ is suitable. We believe that a weighting function is more important for lengthier possessions, whereas for short possessions, all actions may be equally valuable. Hence, an adequate function $\phi$ should be carefully derived. Since this is not the purpose of this work, we leave this for future work as well.

## 4.1.1 Choosing a possession value metric

In this section, we attempt to validate our choice of possession value metric. First, in order to analyze which possession value may be the most suitable, we have to establish how metrics of the form (4), (5), and (10) can be used in practice. In their respective articles, Simpson et al. (2022) and Yeung et al. (2025) use their prediction models to obtain values for each possession. While this approach provides insights into a team's performance and potentially its playing style, it omits an important aspect, namely, what actually happened. In our opinion, the true strength of using action prediction models for deriving possession values relies on comparing the models' predictions with the actually observed possessions. Indeed, for all three metrics presented above, an "observed" possession value can be computed straightforwardly. Instead of plugging in the models' probabilities for each action type into (4), (5), and (10), we simply insert a 1 for the action type that actually took place and a 0 for the other types. To be more precise, in the case of (4) (poss-util), we hence count the number of crosses and shots in a possession. In the case of (5) (HPUS), (7) (AV) is either 0, if the actual action was a "possession loss", 5 if the actual action was a "dribble" or "pass", and 10 if it was a "cross" or "shot". Similarly, (8) (ZV) is 0, if the actual $(x, y)$-coordinates correspond to a location in Area<sub>0</sub>, 5 if they correspond to a location in Area<sub>1</sub>, and 10 if the actual action happened in Area<sub>2</sub><sup>1</sup>. For our LPV metric, (9) is equal to an xG value at the observed location of the action if it was a shot, and equal to an xT value at the observed location if it was any other action (different from a "possession loss").

One way to assess which metric is most suitable for capturing possession value is by examining its correlation with actual game outcomes. In particular, we analyzed all games of the 2017/18 season of the Premier League. For each game, we computed the total possession value of each team, i.e., we aggregated the possession values of each possession they had, and correlated them with goals and xG values in that same match. To be precise, we obtained goals and xG values, i.e., the aggregated xG values for each shot, for each match from understat, a company providing xG values and results for various leagues. These values are obtained using the `worldfootballR` package in R. The main idea is that a useful possession value metric, while certainly providing distinct information, should be at least to some degree correlated to relevant outcome indicators Davis et al. (2024). Figure 3 shows the results of this analysis. On the one hand, we note that all 3 possession value metrics are highly correlated. Furthermore, LPV is more strongly correlated with HPUS than poss-util. This is expected as both take into account location information. On the other hand, we observe that LPV has a higher correlation with xG and goals than the other metrics, suggesting a higher correlation with relevant outcome indicators compared to the alternatives. Hence, our LPV metric is able to capture context relevant for successful teams. Note that the

<sup>1</sup> Note that, for simplicity, we ignore time and set $t = 1$ in (6) for all actions.

<table>
  <thead>
    <tr>
        <th> </th>
        <th>poss-util</th>
        <th>HPUS</th>
        <th>LPV</th>
        <th>xG</th>
        <th>goals</th>
    </tr>
  </thead>
  <tbody>
    <tr>
        <td>poss-util</td>
        <td>1</td>
        <td>0.86</td>
        <td>0.77</td>
        <td>0.43</td>
        <td>0.16</td>
    </tr>
    <tr>
        <td>HPUS</td>
        <td>0.86</td>
        <td>1</td>
        <td>0.93</td>
        <td>0.41</td>
        <td>0.19</td>
    </tr>
    <tr>
        <td>LPV</td>
        <td>0.77</td>
        <td>0.93</td>
        <td>1</td>
        <td>0.51</td>
        <td>0.33</td>
    </tr>
    <tr>
        <td>xG</td>
        <td>0.43</td>
        <td>0.41</td>
        <td>0.51</td>
        <td>1</td>
        <td>0.68</td>
    </tr>
    <tr>
        <td>goals</td>
        <td>0.16</td>
        <td>0.19</td>
        <td>0.33</td>
        <td>0.68</td>
        <td>1</td>
    </tr>
  </tbody>
</table>

**Fig. 3**: Correlation matrix of different possession value metrics (poss-util, HPUS, and LPV) and relevant outcome indicators (xG and goals) for each match of the 2017/18 Premier League season.

correlation between our metric and goals is nevertheless lower than the correlation between xG and goals. However, the aim of our LPV metric is to measure possession value, and a team can have high possession value, but score few goals. Lastly, it should be noted that comparison of our LPV with an xG may seem unfair, as it also contains an "xG" term. However, the xG term in LPV is based on a very simple model (see Appendix C), whereas modern xG models take into account a wide range of features (Robberechts and Davis 2020; Anzer and Bauer 2021). Although it is not entirely clear how the xG value from understat is computed, it is mentioned that they model xG values via a neural network trained on a large dataset. This is in contrast to the simple xG methodology used for LPV as described in Appendix C. Hence, we believe it is sensible to include the comparison with the external xG from understat in addition to comparisons with actual goals.

Another popular approach for assessing metrics is to correlate them to future results (Hvattum and Gelade 2021; Davis et al. 2024). In particular, we follow a similar approach as Spearman (2018), and correlate the possession value metrics to future performance in terms of goals and xG, to check whether our metric is more predictive than scoring itself. To do so, we compute the correlation between possession value metrics, goals, and xG to goals and xG in the subsequent match on a team basis. Figure 4 displays the results of this analysis. In general, HPUS and our LPV show higher correlation with future performance than xG and goals themselves. In addition, our LPV is the most correlated with future performance among all other metrics.

<table>
  <thead>
    <tr>
        <th> </th>
        <th>poss-util</th>
        <th>HPUS</th>
        <th>LPV</th>
        <th>xG</th>
        <th>goals</th>
    </tr>
  </thead>
  <tbody>
    <tr>
        <td>future xG</td>
        <td>0.15</td>
        <td>0.27</td>
        <td>0.32</td>
        <td>0.21</td>
        <td>0.19</td>
    </tr>
    <tr>
        <td>future goals</td>
        <td>0.17</td>
        <td>0.26</td>
        <td>0.28</td>
        <td>0.17</td>
        <td>0.11</td>
    </tr>
  </tbody>
</table>

**Fig. 4**: Comparison of the possession value models (poss-util, HPUS, and LPV) and relevant outcome indicators (xG and goals) with future performance. Displayed is the correlations between poss-util, HPUS, LPV, xG, and goals from one match of a team and xG and goals in the subsequent game of the team (denoted by future xG and future goals).

In conclusion, the results presented in Figures 3 and 4 strengthen our belief that the possession value metric devised in this paper is superior to HPUS and poss-util.

## 4.1.2 Comparison of prediction models in practice

After establishing a practicable possession value metric, we can turn to analyzing prediction models from a domain-specific viewpoint. In Section 3.3.4, we compared the performance of our Sig-Model to a benchmark action prediction model from Simpson et al. (2022) in terms of runtime and loss on a test set. In this section, we again compare our Sig-Model to their Seq2Event model. In particular, we aggregate the predicted possession values over each match and compare them to future performances, as obtained by the aggregated LPV in the subsequent game. Similarly to before, a higher correlation between values from one match and the next match is a desirable characteristic of a performance metric Davis et al. (2024).

Figure 5 shows the correlation of LPV with future xG, future goals, and with a future LPV. The values for LPV are computed once with our Sig-Model and once with the transformer-based Seq2Event model from Simpson et al. (2022). In all three cases, we observe higher correlation with future performance for the Sig-Model than for the Seq2Event model. This result reinforces the findings from Section 3.3.4 and provides confidence for the use of our framework in practice.

## 4.2 Application to Premier League

To highlight the potential of our action prediction model and the LPV metric, we analyze the 2017/18 Premier League season through the lens of the Sig-Model. To this

<table>
  <thead>
    <tr>
        <th> </th>
        <th>future xG</th>
        <th>future goals</th>
        <th>future LPV</th>
    </tr>
  </thead>
  <tbody>
    <tr>
        <td>LPV (Sig-Model)</td>
        <td>0.31</td>
        <td>0.28</td>
        <td>0.51</td>
    </tr>
    <tr>
        <td>LPV (Seq2Ev)</td>
        <td>0.29</td>
        <td>0.26</td>
        <td>0.46</td>
    </tr>
  </tbody>
</table>

**Fig. 5**: Comparison of Sig-Model and Seq2Event model. For each match from a team, the (aggregated) LPV is calculated and the correlation with the xG, goals and (aggregated) LPV in the subsequent match of the team is computed.

<table>
  <thead>
    <tr>
        <th>Team</th>
        <th>Aggregated LPV</th>
        <th>Points (Panel A)</th>
        <th>xG (Panel B)</th>
    </tr>
  </thead>
  <tbody>
    <tr>
        <td>Manchester City (1)</td>
        <td>530</td>
        <td>100</td>
        <td>82</td>
    </tr>
    <tr>
        <td>Manchester United (2)</td>
        <td>340</td>
        <td>81</td>
        <td>58</td>
    </tr>
    <tr>
        <td>Tottenham Hotspur (3)</td>
        <td>380</td>
        <td>77</td>
        <td>65</td>
    </tr>
    <tr>
        <td>Liverpool (4)</td>
        <td>410</td>
        <td>75</td>
        <td>72</td>
    </tr>
    <tr>
        <td>Chelsea (5)</td>
        <td>360</td>
        <td>70</td>
        <td>59</td>
    </tr>
    <tr>
        <td>Arsenal (6)</td>
        <td>420</td>
        <td>63</td>
        <td>68</td>
    </tr>
  </tbody>
</table>

**Fig. 6**: Analysis of the LPV values for the 2017/18 Premier League season. Panel A displays a scatterplot of the aggregated LPV value of a team over the season and the total final points. Panel B shows a scatterplot of the aggregated LPV value of a team over the season and the aggregated xG values of a team over the season. Panel C shows density plots of LPV for the top 6 teams in that season.

end, we first compare teams by means of LPV. Figure 6 shows three different plots connecting a team’s points, xG, and possession value. Panel A shows the relationship

between a team's final points and the accumulated LPV. We observe a strong positive correlation ($R \approx 0.896$), indicating that teams with high possession value performed better in terms of final ranking. Indeed, the top-performing team of the 2017/18 Premier League season, Manchester City, also had the highest cumulative possession value over the season. Notably, Panel A reveals three distinct clusters of teams. First, Manchester City performed far better than the next competitors (in terms of points and possession value). Then, there is a visible cluster of teams ranked in places 2-6 in the final league table (Manchester United, Rank 2; Tottenham Hotspur, Rank 3; Liverpool, Rank 4; Chelsea, Rank 5; and Arsenal, Rank 6). Finally, there is a cluster for the rest of the teams. Among weaker teams, the correlation between cumulative possession value and total points is lower. Panel B displays the correlation between accumulated xG values and accumulated possession values. There is an even stronger positive correlation observable in Panel B ($R \approx 0.934$). In fact, we observe that the alignment between accumulated xG values and accumulated possession values is more pronounced also for weaker teams. From a domain-specific viewpoint, this result is expected, as the possession value metric and xG both are measures of offensive quality, whereas final points to a certain degree account for offensive and defensive team strength. A particular example is Manchester United, the runner-up of that season. They played an outstanding season from a defensive viewpoint, conceding only 28 goals, the second fewest this season after leader Manchester City (27 goals conceded). Offensively, however, they performed worse, ranking only 5th and 6th in total goals scored and xG created, respectively. This can be attributed to the playing style of José Mourinho, Manchester United's coach in that season, who is well-known for playing pragmatic and result-oriented football based on a solid defensive line<sup>2</sup>. On the other hand, Arsenal, ranked only 6th in the 2017/18 final league table, had an offensively strong season but struggled defensively, another well-studied fact<sup>3</sup>. Hence, our metric is able to capture playing style of different teams adequately, at least in terms of offensive style. Panel C displays the densities for the possession values of the top 6 teams. The story is similar to before: Manchester City is much more threatening, showing a higher density on high possession values; Arsenal (6) and Liverpool (4), both offensively strong teams follow with higher right tails; Manchester United (2) and Chelsea (5), defensively strong teams, have more mass at smaller possession values.

An interesting case is Tottenham Hotspur, the 3rd-ranked team in the 2017/18 Premier League season. While they certainly played a remarkable season, both their offensive and defensive quality were on a high level. Figure 7 allows us to investigate their season in more detail. Specifically, the figure considers the relative difference in predicted vs true possession values of teams. That is, for each possession, we compute the difference between the value for the possession according to our model and the actual "observed" possession value computed as described in Section 4.1.1. Then, we divide the difference by the predicted possession value to account for the fact that a higher predicted threat allows for a higher discrepancy from observed values. Finally, we aggregated the relative difference value for teams over the season, similar to what we did beforehand with the possession value. In this case, a smaller value (the value can

<sup>2</sup>See for example this article, or this Youtube video.

<sup>3</sup>See for example this article reviewing Arsenal's season by common soccer stats.

<table>
  <thead>
    <tr>
        <th>Team</th>
        <th>Relative difference in predicted vs true LPV (aggregated)</th>
        <th>Points</th>
    </tr>
  </thead>
  <tbody>
    <tr>
        <td>Manchester City (1)</td>
        <td>13.5</td>
        <td>100</td>
    </tr>
    <tr>
        <td>Manchester United (2)</td>
        <td>16.2</td>
        <td>81</td>
    </tr>
    <tr>
        <td>Tottenham Hotspur (3)</td>
        <td>15.5</td>
        <td>77</td>
    </tr>
    <tr>
        <td>Liverpool (4)</td>
        <td>15.8</td>
        <td>75</td>
    </tr>
    <tr>
        <td>Chelsea (5)</td>
        <td>15.7</td>
        <td>70</td>
    </tr>
    <tr>
        <td>Arsenal (6)</td>
        <td>15.1</td>
        <td>63</td>
    </tr>
    <tr>
        <td>Other Team 1</td>
        <td>17.5</td>
        <td>48</td>
    </tr>
    <tr>
        <td>Other Team 2</td>
        <td>18.2</td>
        <td>44</td>
    </tr>
    <tr>
        <td>Other Team 3</td>
        <td>18.5</td>
        <td>40</td>
    </tr>
    <tr>
        <td>Other Team 4</td>
        <td>18.8</td>
        <td>36</td>
    </tr>
    <tr>
        <td>Other Team 5</td>
        <td>19.1</td>
        <td>44</td>
    </tr>
    <tr>
        <td>Other Team 6</td>
        <td>19.3</td>
        <td>41</td>
    </tr>
    <tr>
        <td>Other Team 7</td>
        <td>19.5</td>
        <td>33</td>
    </tr>
    <tr>
        <td>Other Team 8</td>
        <td>19.7</td>
        <td>49</td>
    </tr>
    <tr>
        <td>Other Team 9</td>
        <td>19.8</td>
        <td>37</td>
    </tr>
    <tr>
        <td>Other Team 10</td>
        <td>20.1</td>
        <td>31</td>
    </tr>
    <tr>
        <td>Other Team 11</td>
        <td>20.3</td>
        <td>37</td>
    </tr>
  </tbody>
</table>
<table>
  <thead>
    <tr>
        <th>Team</th>
        <th>Relative difference in predicted vs true LPV (aggregated)</th>
        <th>xG</th>
    </tr>
  </thead>
  <tbody>
    <tr>
        <td>Manchester City (1)</td>
        <td>13.5</td>
        <td>91</td>
    </tr>
    <tr>
        <td>Liverpool (4)</td>
        <td>15.8</td>
        <td>78</td>
    </tr>
    <tr>
        <td>Arsenal (6)</td>
        <td>15.1</td>
        <td>74</td>
    </tr>
    <tr>
        <td>Tottenham Hotspur (3)</td>
        <td>15.5</td>
        <td>68</td>
    </tr>
    <tr>
        <td>Chelsea (5)</td>
        <td>15.7</td>
        <td>62</td>
    </tr>
    <tr>
        <td>Manchester United (2)</td>
        <td>16.2</td>
        <td>59</td>
    </tr>
    <tr>
        <td>Other Team 1</td>
        <td>17.5</td>
        <td>51</td>
    </tr>
    <tr>
        <td>Other Team 2</td>
        <td>18.2</td>
        <td>41</td>
    </tr>
    <tr>
        <td>Other Team 3</td>
        <td>18.5</td>
        <td>48</td>
    </tr>
    <tr>
        <td>Other Team 4</td>
        <td>18.8</td>
        <td>44</td>
    </tr>
    <tr>
        <td>Other Team 5</td>
        <td>19.1</td>
        <td>50</td>
    </tr>
    <tr>
        <td>Other Team 6</td>
        <td>19.3</td>
        <td>42</td>
    </tr>
    <tr>
        <td>Other Team 7</td>
        <td>19.5</td>
        <td>38</td>
    </tr>
    <tr>
        <td>Other Team 8</td>
        <td>19.7</td>
        <td>45</td>
    </tr>
    <tr>
        <td>Other Team 9</td>
        <td>19.8</td>
        <td>35</td>
    </tr>
    <tr>
        <td>Other Team 10</td>
        <td>20.1</td>
        <td>33</td>
    </tr>
    <tr>
        <td>Other Team 11</td>
        <td>20.3</td>
        <td>34</td>
    </tr>
  </tbody>
</table>
<table>
  <thead>
    <tr>
        <th>Team</th>
        <th>Peak Relative Difference (approx)</th>
        <th>Peak Density (approx)</th>
    </tr>
  </thead>
  <tbody>
    <tr>
        <td>Arsenal</td>
        <td>0.18</td>
        <td>2.4</td>
    </tr>
    <tr>
        <td>Liverpool</td>
        <td>0.16</td>
        <td>2.5</td>
    </tr>
    <tr>
        <td>Manchester United</td>
        <td>0.20</td>
        <td>2.3</td>
    </tr>
    <tr>
        <td>Chelsea</td>
        <td>0.17</td>
        <td>2.4</td>
    </tr>
    <tr>
        <td>Manchester City</td>
        <td>0.14</td>
        <td>2.6</td>
    </tr>
    <tr>
        <td>Tottenham Hotspur</td>
        <td>0.15</td>
        <td>2.7</td>
    </tr>
  </tbody>
</table>

**Fig. 7**: Analysis of the relative difference in predicted vs true LPV values for the 2017/18 Premier League season. Panel A displays a scatterplot of the aggregated relative difference in predicted vs true LPV value of a team over the season and the total final points. Panel B shows a scatterplot of the aggregated relative difference in predicted vs true LPV value of a team over the season and the aggregated xG values of a team over the season. Panel C shows density plots of the relative difference in predicted vs true LPV for the top 6 teams in that season.

potentially be negative) corresponds to a better performance. In general, we observe that the predicted value is higher than the actual value of the possession. Intuitively, this can be explained by the fact that the predicted value takes into account all possible options for the next action in each step of the possession. Hence, suboptimal decisions in terms of actions performed are accumulated within each possession. A team with a smaller relative difference can be interpreted as being more effective in their attacking intent, as indicated by being closer to their potential possession value from the model. As in Figure 6, Panel A of Figure 7 relates this difference in possession value to final points, whereas Panel B relates it to xG. In both cases, we observe a strong negative correlation ($R \approx -0.876$ in Panel A and $R \approx -0.907$ in Panel B), indicating that better teams generate more effective possessions, as shown by smaller accumulated relative difference in predicted vs true possession value. Panel C displays again the densities for this relative difference for the possessions of the six top-ranked teams. All three plots in Figure 7 show that Tottenham played a very effective season, meaning that the relative difference between predicted and true possession value was generally low. This may be one aspect why they ended up in 3<sup>rd</sup> position overall.

A visual representation of one possession on a soccer field with corresponding action probability and action value bar charts.

<table>
  <thead>
    <tr>
        <th>Action</th>
        <th>Action probability</th>
    </tr>
  </thead>
  <tbody>
    <tr>
        <td>P(cross)</td>
        <td>0.25</td>
    </tr>
    <tr>
        <td>P(shot)</td>
        <td>0.58</td>
    </tr>
    <tr>
        <td>P(pass)</td>
        <td>0.02</td>
    </tr>
    <tr>
        <td>P(dribble)</td>
        <td>0.05</td>
    </tr>
  </tbody>
</table>
<table>
  <thead>
    <tr>
        <th>Action</th>
        <th>Action value</th>
    </tr>
  </thead>
  <tbody>
    <tr>
        <td>AV(cross)</td>
        <td>0.085</td>
    </tr>
    <tr>
        <td>AV(shot)</td>
        <td>0.045</td>
    </tr>
    <tr>
        <td>AV(pass)</td>
        <td>0.055</td>
    </tr>
    <tr>
        <td>AV(dribble)</td>
        <td>0.065</td>
    </tr>
  </tbody>
</table>

**Fig. 8**: A visual representation of one possession in the data. The top barplot displays the action probabilities as predicted from our model. The bottom barplot displays the action value (LAV) when performing each of the possible actions.

## 4.3 Analyzing specific game situations

Beyond possession value analyses, our framework enables the evaluation of specific in-game situations in soccer, either in real time or as part of pre-game preparation. Figure 8 presents an example of how the model can be used to analyze a specific situation in detail. For the displayed possession, we are able to analyze the most likely next action as well as the value for certain options. In particular, our model expects a shot with high probability, but also assigns a fair portion of probability to a cross. For each action, we can additionally use our model to compute the hypothetical action value using (7) for performing any of the actions. It can be seen that the best option is to cross the ball instead of shooting the ball. Such a tool could support coaches in developing strategies and guiding players toward better decision-making.

## 5 Discussion and conclusion

In this work, we present a novel methodology for predicting the next action in a soccer possession. We propose to use path signatures to encode the spatio-temporal information contained in possessions. Path signatures are a natural choice for the task of action predictions, because they are able to handle possessions of different lengths and they implicitly extract all relevant information in a time-series, avoiding the need for explicit feature engineering. Our results show that our model accurately predicts next

actions and is computationally more efficient than popular existing methods. Furthermore, we present a novel method to evaluate possessions, taking into account action type and predicted locations in an intuitive and interpretable way. Our possession evaluation metric proves to be more reliable in predicting future performance than existing alternatives, and our action prediction model performs better paired with the metric than the benchmark model. We complement these findings with a detailed analysis of the 2017/18 Premier League season to show the effectiveness of our model and the possession evaluation metric. Finally, we present further use cases of our model in practice.

There are various ways to extend our framework. For comparison reasons, we mainly followed the definitions of possessions and actions from previous work (Simpson et al. 2022; Yeung et al. 2025). While we believe that this is a sensible approach, clearly capturing the most important aspects of soccer possessions, the data would allow for using a more granular set of action types. Depending on the interest of the analyst, our model could be implemented with a more diverse set of action types. Furthermore, for practical reasons, we refrained from explicitly modeling the interevent time. However, it is straightforward to adapt our framework to also account for the time between two events.

Another key factor for improving and extending our work is the available data. For this paper, we used openly available event stream data from Wyscout. Our action prediction model and the resulting applications are limited to the information available within this data. More seasons of data, as well as more granular data such as tracking data, would allow for more refined analyses. In particular, signatures could be used in a similar fashion for tracking data time series to improve the use cases presented. To be more precise, if the trajectories of all 22 players within possessions were available, one could encode this information via signatures to obtain far more predictive models for the current state and value of a possession. Due to the spatio-temporal nature of sports data in general, we believe there is an immense potential for using path signatures in various settings to enhance sports analytics.

In conclusion, the framework presented in this work is a step toward better modeling and understanding professional soccer. Using path signatures as a tool for capturing the complex spatio-temporal structure of the sport enhances possession evaluation models and opens up novel possibilities for analyzing the inherently dynamic nature of many sports.

**Supplementary information.** The code for reproducing the results and figures for this paper is available at [https://github.com/Rob2208/sig_actions](https://github.com/Rob2208/sig_actions). Data preprocessing and implementation of the benchmark transformer model are based on code from [https://github.com/calvinyeungck/Football-Match-Event-Forecast/](https://github.com/calvinyeungck/Football-Match-Event-Forecast/), distributed under the Apache License 2.0. We modified the code to fit our experimental requirements.

# Appendix A Hyperparameter Tuning

We conducted an extensive hyperparameter tuning, systematically testing various parameter combinations. The values used during the experiments are stated in Table

A1, where the final selection is marked in bold. As mentioned in Section 3.3.3,

**Table A1**: Grid of evaluated hyperparameters, with final selection highlighted in bold.

<table>
  <thead>
    <tr>
        <th>Hyperparameter</th>
        <th>Values</th>
    </tr>
  </thead>
  <tbody>
    <tr>
        <td>CEL scaling</td>
        <td><strong>1</strong>, 5</td>
    </tr>
    <tr>
        <td>Hidden dim</td>
        <td>64, 128, <strong>256</strong></td>
    </tr>
    <tr>
        <td>Batch size</td>
        <td><strong>4</strong>, 10, 32</td>
    </tr>
    <tr>
        <td>Signature order</td>
        <td><strong>3</strong>, 4</td>
    </tr>
  </tbody>
</table>

hyperparameters were selected in two phases based on model performance, using $n_r \in \{3, 4, 5, 6, 7\}$ recent action features as input, hence forecasting from the 4<sup>th</sup>, 5<sup>th</sup>, 6<sup>th</sup>, 7<sup>th</sup> or 8<sup>th</sup> action onward. In phase one, we investigate the CEL scaling parameter $\lambda$, signature orders $M$, and batch sizes $n_{batch}$. Table A2 shows losses for the evaluated CEL scaling values, indicating the best overall performance for $\lambda = 1$.

**Table A2**: Average losses across hyperparameter grid for CEL scaling $\lambda \in \{1, 5\}$, for each value of historic actions $n_r \in \{3, 4, 5, 6, 7\}$.

<table>
  <thead>
    <tr>
        <th rowspan="2">$n_r$</th>
        <th colspan="2">MSE</th>
        <th colspan="2">CEL</th>
        <th colspan="2">Brier</th>
        <th colspan="2">KL</th>
    </tr>
    <tr>
        <th>$\lambda=1$</th>
        <th>$\lambda=5$</th>
        <th>$\lambda=1$</th>
        <th>$\lambda=5$</th>
        <th>$\lambda=1$</th>
        <th>$\lambda=5$</th>
        <th>$\lambda=1$</th>
        <th>$\lambda=5$</th>
    </tr>
  </thead>
  <tbody>
    <tr>
        <td>3</td>
        <td>0.1643</td>
        <td>0.1649</td>
        <td>0.0488</td>
        <td>0.0489</td>
        <td>0.7985</td>
        <td>0.8005</td>
        <td>0.1122</td>
        <td>0.1118</td>
    </tr>
    <tr>
        <td>4</td>
        <td>0.1648</td>
        <td>0.1654</td>
        <td>0.0496</td>
        <td>0.0498</td>
        <td>0.7962</td>
        <td>0.7987</td>
        <td>0.1143</td>
        <td>0.1144</td>
    </tr>
    <tr>
        <td>5</td>
        <td>0.1648</td>
        <td>0.1656</td>
        <td>0.0507</td>
        <td>0.0508</td>
        <td>0.7899</td>
        <td>0.7930</td>
        <td>0.1159</td>
        <td>0.1163</td>
    </tr>
    <tr>
        <td>6</td>
        <td>0.1656</td>
        <td>0.1662</td>
        <td>0.0513</td>
        <td>0.0512</td>
        <td>0.7893</td>
        <td>0.7916</td>
        <td>0.1174</td>
        <td>0.1172</td>
    </tr>
    <tr>
        <td>7</td>
        <td>0.1655</td>
        <td>0.1661</td>
        <td>0.0523</td>
        <td>0.0524</td>
        <td>0.7927</td>
        <td>0.7950</td>
        <td>0.1183</td>
        <td>0.1185</td>
    </tr>
  </tbody>
</table>

Table A3 shows losses for the tested signature orders, indicating the best overall performance for $M = 3$. In table A4 we present the losses for our evaluated batch sizes, which led to the exclusion of batch size 32, due to its significantly higher RMSE.

During the second phase of hyperparameter tuning, we evaluated the six most promising model candidates. Their parameter configuration are shown in Table A5. We evaluate their robustness based on average losses across ten data sets, generated by randomly selecting teams and matches for the used train and test data sets. The average scores, along with their standard deviation across the generated datasets, are shown in Table A6.

**Table A3**: Average losses across hyperparameter grid for signature order $M \in \{3, 4\}$, for each value of historic actions $n_r \in \{3, 4, 5, 6, 7\}$.

<table>
  <thead>
    <tr>
        <th rowspan="2">$n_r$</th>
        <th colspan="2">MSE</th>
        <th colspan="2">CEL</th>
        <th colspan="2">Brier</th>
        <th colspan="2">KL</th>
    </tr>
    <tr>
        <th>M=3</th>
        <th>M=4</th>
        <th>M=3</th>
        <th>M=4</th>
        <th>M=3</th>
        <th>M=4</th>
        <th>M=3</th>
        <th>M=4</th>
    </tr>
  </thead>
  <tbody>
    <tr>
        <td>3</td>
        <td>0.1645</td>
        <td>0.1642</td>
        <td>0.0486</td>
        <td>0.0489</td>
        <td>0.7971</td>
        <td>0.8000</td>
        <td>0.1122</td>
        <td>0.1122</td>
    </tr>
    <tr>
        <td>4</td>
        <td>0.1650</td>
        <td>0.1646</td>
        <td>0.0496</td>
        <td>0.0497</td>
        <td>0.7961</td>
        <td>0.7963</td>
        <td>0.1142</td>
        <td>0.1144</td>
    </tr>
    <tr>
        <td>5</td>
        <td>0.1648</td>
        <td>0.1648</td>
        <td>0.0507</td>
        <td>0.0508</td>
        <td>0.7898</td>
        <td>0.7901</td>
        <td>0.1158</td>
        <td>0.1161</td>
    </tr>
    <tr>
        <td>6</td>
        <td>0.1658</td>
        <td>0.1655</td>
        <td>0.0513</td>
        <td>0.0513</td>
        <td>0.7890</td>
        <td>0.7896</td>
        <td>0.1171</td>
        <td>0.1178</td>
    </tr>
    <tr>
        <td>7</td>
        <td>0.1658</td>
        <td>0.1653</td>
        <td>0.0522</td>
        <td>0.0523</td>
        <td>0.7935</td>
        <td>0.7919</td>
        <td>0.1184</td>
        <td>0.1182</td>
    </tr>
  </tbody>
</table>

**Table A4**: Average losses across hyperparameter grid for batch size $n_{batch} \in \{4, 10, 32\}$, for each value of historic actions $n_r \in \{3, 4, 5, 6, 7\}$.

<table>
  <thead>
    <tr>
        <th rowspan="2">$n_r$</th>
        <th colspan="3">MSE</th>
        <th colspan="3">CEL</th>
    </tr>
    <tr>
        <th>4</th>
        <th>10</th>
        <th>32</th>
        <th>4</th>
        <th>10</th>
        <th>32</th>
    </tr>
  </thead>
  <tbody>
    <tr>
        <td>3</td>
        <td>0.1596</td>
        <td>0.1657</td>
        <td>0.1683</td>
        <td>0.0486</td>
        <td>0.0486</td>
        <td>0.0486</td>
    </tr>
    <tr>
        <td>4</td>
        <td>0.1601</td>
        <td>0.1662</td>
        <td>0.1688</td>
        <td>0.0497</td>
        <td>0.0495</td>
        <td>0.0496</td>
    </tr>
    <tr>
        <td>5</td>
        <td>0.1600</td>
        <td>0.1657</td>
        <td>0.1688</td>
        <td>0.0507</td>
        <td>0.0508</td>
        <td>0.0506</td>
    </tr>
    <tr>
        <td>6</td>
        <td>0.1606</td>
        <td>0.1670</td>
        <td>0.1698</td>
        <td>0.0512</td>
        <td>0.0512</td>
        <td>0.0515</td>
    </tr>
    <tr>
        <td>7</td>
        <td>0.1610</td>
        <td>0.1666</td>
        <td>0.1697</td>
        <td>0.0519</td>
        <td>0.0519</td>
        <td>0.0529</td>
    </tr>
    <tr>
        <th> </th>
        <th colspan="3">Brier</th>
        <th colspan="3">KL</th>
    </tr>
    <tr>
        <th> </th>
        <th>4</th>
        <th>10</th>
        <th>32</th>
        <th>4</th>
        <th>10</th>
        <th>32</th>
    </tr>
    <tr>
        <td>3</td>
        <td>0.7965</td>
        <td>0.7965</td>
        <td>0.7982</td>
        <td>0.1119</td>
        <td>0.1125</td>
        <td>0.1122</td>
    </tr>
    <tr>
        <td>4</td>
        <td>0.7960</td>
        <td>0.7956</td>
        <td>0.7968</td>
        <td>0.1135</td>
        <td>0.1144</td>
        <td>0.1147</td>
    </tr>
    <tr>
        <td>5</td>
        <td>0.7872</td>
        <td>0.7910</td>
        <td>0.7912</td>
        <td>0.1151</td>
        <td>0.1160</td>
        <td>0.1163</td>
    </tr>
    <tr>
        <td>6</td>
        <td>0.7873</td>
        <td>0.7887</td>
        <td>0.7910</td>
        <td>0.1164</td>
        <td>0.1173</td>
        <td>0.1176</td>
    </tr>
    <tr>
        <td>7</td>
        <td>0.7893</td>
        <td>0.7946</td>
        <td>0.7965</td>
        <td>0.1170</td>
        <td>0.1184</td>
        <td>0.1199</td>
    </tr>
  </tbody>
</table>

# Appendix B Additional tests

To support our pre-processing proposed in 3.3.1, we evaluate our model both with and without the additional hand-crafted features proposed in Simpson et al. (2022), with their results presented in Table B7. Moreover, we adapt the code from [https://github.com/calvinyeungck/Football-Match-Event-Forecast/](https://github.com/calvinyeungck/Football-Match-Event-Forecast/) and re-implemented the Seq2Event model proposed by Simpson et al. (2022) for various configurations and report its results in 1. We varied the length of past actions history used as model input during training or restricted the input to the same raw match data triplets $(x, y, T)$ used in our approach without engineering additional geometric or temporal features. This analysis provides empirical evidence for the effectiveness of our proposed pre-processing and the reduced dependency of our approach on hand-crafted features.

**Table A5**: Hyper-parameter settings for the six model candidates after tuning stage one.

<table>
  <thead>
    <tr>
        <th>Model</th>
        <th>CEL scale λ</th>
        <th>signature order $M$</th>
        <th>Batch size</th>
        <th>Hidden dimension</th>
    </tr>
  </thead>
  <tbody>
    <tr>
        <td>model 1</td>
        <td>1</td>
        <td>3</td>
        <td>4</td>
        <td>[64, 64]</td>
    </tr>
    <tr>
        <td>model 2</td>
        <td>1</td>
        <td>3</td>
        <td>4</td>
        <td>[128, 128]</td>
    </tr>
    <tr>
        <td>model 3</td>
        <td>1</td>
        <td>3</td>
        <td>4</td>
        <td>[256, 256]</td>
    </tr>
    <tr>
        <td>model 4</td>
        <td>1</td>
        <td>3</td>
        <td>10</td>
        <td>[64, 64]</td>
    </tr>
    <tr>
        <td>model 5</td>
        <td>1</td>
        <td>3</td>
        <td>10</td>
        <td>[128, 128]</td>
    </tr>
    <tr>
        <td>model 6</td>
        <td>1</td>
        <td>3</td>
        <td>10</td>
        <td>[256, 256]</td>
    </tr>
  </tbody>
</table>

**Table A6**: Mean (Std) across datasets for model candidates and Seq2Event (smallest value across model candidates per metric and number of historic actions in bold) Average losses (standard deviations) across test datasets for each model in tuning phase two. Best values for each metric are highlighted in bold, separately for each value of historic actions $n_r \in \{3, 4, 5, 6, 7\}$.

<table>
  <thead>
    <tr>
        <th>$n_r$</th>
        <th>Model</th>
        <th>Test loss</th>
        <th>MSE</th>
        <th>CEL</th>
        <th>KL</th>
        <th>Brier</th>
    </tr>
  </thead>
  <tbody>
    <tr>
        <td rowspan="6">3</td>
        <td>1</td>
        <td>0.2091 (0.0031)</td>
        <td>0.1601 (0.0019)</td>
        <td>0.0491 (0.0016)</td>
        <td>0.1126 (0.0020)</td>
        <td>0.8033 (0.0096)</td>
    </tr>
    <tr>
        <td>2</td>
        <td>0.2092 (0.0032)</td>
        <td>0.1600 (0.0019)</td>
        <td>0.0492 (0.0017)</td>
        <td>0.1126 (0.0020)</td>
        <td>0.8028 (0.0098)</td>
    </tr>
    <tr>
        <td>3</td>
        <td><strong>0.2090 (0.0030)</strong></td>
        <td><strong>0.1599 (0.0018)</strong></td>
        <td><strong>0.0490 (0.0016)</strong></td>
        <td><strong>0.1123 (0.0019)</strong></td>
        <td>0.8022 (0.0090)</td>
    </tr>
    <tr>
        <td>4</td>
        <td>0.2151 (0.0033)</td>
        <td>0.1661 (0.0020)</td>
        <td><strong>0.0490 (0.0017)</strong></td>
        <td>0.1126 (0.0021)</td>
        <td>0.8025 (0.0091)</td>
    </tr>
    <tr>
        <td>5</td>
        <td>0.2152 (0.0032)</td>
        <td>0.1661 (0.0018)</td>
        <td><strong>0.0490 (0.0017)</strong></td>
        <td>0.1128 (0.0020)</td>
        <td>0.8020 (0.0107)</td>
    </tr>
    <tr>
        <td>6</td>
        <td>0.2151 (0.0033)</td>
        <td>0.1661 (0.0019)</td>
        <td><strong>0.0490 (0.0017)</strong></td>
        <td>0.1125 (0.0020)</td>
        <td><strong>0.8010 (0.0096)</strong></td>
    </tr>
    <tr>
        <td rowspan="6">4</td>
        <td>1</td>
        <td>0.2104 (0.0029)</td>
        <td>0.1603 (0.0017)</td>
        <td>0.0501 (0.0017)</td>
        <td>0.1146 (0.0023)</td>
        <td>0.7973 (0.0111)</td>
    </tr>
    <tr>
        <td>2</td>
        <td>0.2104 (0.0030)</td>
        <td><strong>0.1601 (0.0018)</strong></td>
        <td>0.0503 (0.0018)</td>
        <td>0.1145 (0.0021)</td>
        <td>0.7966 (0.0107)</td>
    </tr>
    <tr>
        <td>3</td>
        <td><strong>0.2102 (0.0031)</strong></td>
        <td><strong>0.1601 (0.0018)</strong></td>
        <td>0.0501 (0.0018)</td>
        <td><strong>0.1140 (0.0023)</strong></td>
        <td>0.7947 (0.0103)</td>
    </tr>
    <tr>
        <td>4</td>
        <td>0.2166 (0.0031)</td>
        <td>0.1666 (0.0018)</td>
        <td><strong>0.0500 (0.0017)</strong></td>
        <td>0.1147 (0.0023)</td>
        <td>0.7955 (0.0110)</td>
    </tr>
    <tr>
        <td>5</td>
        <td>0.2164 (0.0031)</td>
        <td>0.1664 (0.0019)</td>
        <td><strong>0.0500 (0.0017)</strong></td>
        <td>0.1145 (0.0025)</td>
        <td>0.7953 (0.0112)</td>
    </tr>
    <tr>
        <td>6</td>
        <td>0.2164 (0.0032)</td>
        <td>0.1662 (0.0019)</td>
        <td>0.0501 (0.0018)</td>
        <td>0.1141 (0.0024)</td>
        <td><strong>0.7946 (0.0105)</strong></td>
    </tr>
    <tr>
        <td rowspan="6">5</td>
        <td>1</td>
        <td><strong>0.2112 (0.0035)</strong></td>
        <td>0.1604 (0.0020)</td>
        <td><strong>0.0509 (0.0020)</strong></td>
        <td>0.1156 (0.0026)</td>
        <td>0.7906 (0.0109)</td>
    </tr>
    <tr>
        <td>2</td>
        <td><strong>0.2112 (0.0035)</strong></td>
        <td><strong>0.1602 (0.0019)</strong></td>
        <td>0.0510 (0.0020)</td>
        <td>0.1156 (0.0030)</td>
        <td>0.7907 (0.0107)</td>
    </tr>
    <tr>
        <td>3</td>
        <td><strong>0.2112 (0.0033)</strong></td>
        <td><strong>0.1602 (0.0017)</strong></td>
        <td>0.0510 (0.0021)</td>
        <td><strong>0.1154 (0.0028)</strong></td>
        <td>0.7913 (0.0107)</td>
    </tr>
    <tr>
        <td>4</td>
        <td>0.2176 (0.0036)</td>
        <td>0.1667 (0.0019)</td>
        <td><strong>0.0509 (0.0021)</strong></td>
        <td>0.1162 (0.0034)</td>
        <td>0.7922 (0.0134)</td>
    </tr>
    <tr>
        <td>5</td>
        <td>0.2176 (0.0037)</td>
        <td>0.1667 (0.0021)</td>
        <td>0.0510 (0.0021)</td>
        <td>0.1158 (0.0029)</td>
        <td><strong>0.7901 (0.0112)</strong></td>
    </tr>
    <tr>
        <td>6</td>
        <td>0.2173 (0.0035)</td>
        <td>0.1664 (0.0019)</td>
        <td><strong>0.0509 (0.0020)</strong></td>
        <td>0.1156 (0.0026)</td>
        <td>0.7906 (0.0106)</td>
    </tr>
    <tr>
        <td rowspan="6">6</td>
        <td>1</td>
        <td>0.2127 (0.0035)</td>
        <td>0.1610 (0.0018)</td>
        <td>0.0517 (0.0022)</td>
        <td>0.1172 (0.0024)</td>
        <td>0.7934 (0.0129)</td>
    </tr>
    <tr>
        <td>2</td>
        <td><strong>0.2122 (0.0038)</strong></td>
        <td><strong>0.1605 (0.0019)</strong></td>
        <td>0.0516 (0.0023)</td>
        <td>0.1165 (0.0025)</td>
        <td>0.7906 (0.0130)</td>
    </tr>
    <tr>
        <td>3</td>
        <td>0.2124 (0.0039)</td>
        <td>0.1606 (0.0020)</td>
        <td>0.0518 (0.0022)</td>
        <td>0.1166 (0.0025)</td>
        <td><strong>0.7904 (0.0146)</strong></td>
    </tr>
    <tr>
        <td>4</td>
        <td>0.2186 (0.0038)</td>
        <td>0.1671 (0.0020)</td>
        <td><strong>0.0515 (0.0022)</strong></td>
        <td>0.1170 (0.0024)</td>
        <td>0.7918 (0.0129)</td>
    </tr>
    <tr>
        <td>5</td>
        <td>0.2187 (0.0039)</td>
        <td>0.1671 (0.0021)</td>
        <td>0.0517 (0.0023)</td>
        <td>0.1169 (0.0024)</td>
        <td>0.7906 (0.0129)</td>
    </tr>
    <tr>
        <td>6</td>
        <td>0.2186 (0.0035)</td>
        <td>0.1670 (0.0019)</td>
        <td>0.0517 (0.0021)</td>
        <td><strong>0.1164 (0.0026)</strong></td>
        <td>0.7914 (0.0130)</td>
    </tr>
    <tr>
        <td rowspan="6">7</td>
        <td>1</td>
        <td><strong>0.2141 (0.0041)</strong></td>
        <td>0.1618 (0.0020)</td>
        <td><strong>0.0523 (0.0025)</strong></td>
        <td>0.1170 (0.0029)</td>
        <td>0.7936 (0.0156)</td>
    </tr>
    <tr>
        <td>2</td>
        <td>0.2143 (0.0038)</td>
        <td><strong>0.1616 (0.0020)</strong></td>
        <td>0.0527 (0.0024)</td>
        <td>0.1176 (0.0031)</td>
        <td>0.7952 (0.0161)</td>
    </tr>
    <tr>
        <td>3</td>
        <td>0.2145 (0.0037)</td>
        <td>0.1617 (0.0018)</td>
        <td>0.0528 (0.0024)</td>
        <td><strong>0.1164 (0.0030)</strong></td>
        <td><strong>0.7917 (0.0162)</strong></td>
    </tr>
    <tr>
        <td>4</td>
        <td>0.2197 (0.0039)</td>
        <td>0.1674 (0.0019)</td>
        <td><strong>0.0523 (0.0023)</strong></td>
        <td>0.1176 (0.0035)</td>
        <td>0.7955 (0.0166)</td>
    </tr>
    <tr>
        <td>5</td>
        <td>0.2199 (0.0041)</td>
        <td>0.1674 (0.0020)</td>
        <td>0.0525 (0.0025)</td>
        <td>0.1177 (0.0030)</td>
        <td>0.7953 (0.0139)</td>
    </tr>
    <tr>
        <td>6</td>
        <td>0.2196 (0.0039)</td>
        <td>0.1672 (0.0020)</td>
        <td>0.0524 (0.0023)</td>
        <td>0.1175 (0.0032)</td>
        <td>0.7952 (0.0145)</td>
    </tr>
  </tbody>
</table>

It further highlights the ability of path signatures to capture rich spatial-temporal structures directly from raw data.

**Table B7**: Losses of our Sig-Model with and without hand-crafted features (HC) proposed in Simpson et al. (2022), where Test loss = MSE + CEL. Best values for each $n_r \in \{3, 4, 5, 6, 7\}$ are highlighted in bold.

<table>
  <thead>
    <tr>
        <th>$n_r$</th>
        <th>Sig-Model</th>
        <th>Test loss</th>
        <th>MSE</th>
        <th>CEL</th>
        <th>Brier</th>
        <th>KL</th>
    </tr>
  </thead>
  <tbody>
    <tr>
        <td rowspan="2">3</td>
        <td>w/o HC</td>
        <td><strong>0.2084</strong></td>
        <td><strong>0.1598</strong></td>
        <td>0.0486</td>
        <td><strong>0.7968</strong></td>
        <td><strong>0.1117</strong></td>
    </tr>
    <tr>
        <td>w/ HC</td>
        <td>0.2092</td>
        <td>0.0507</td>
        <td>0.1584</td>
        <td>0.8034</td>
        <td>0.1133</td>
    </tr>
    <tr>
        <td rowspan="2">4</td>
        <td>w/o HC</td>
        <td><strong>0.2096</strong></td>
        <td><strong>0.1601</strong></td>
        <td>0.0495</td>
        <td><strong>0.7938</strong></td>
        <td><strong>0.1125</strong></td>
    </tr>
    <tr>
        <td>w/ HC</td>
        <td>0.2105</td>
        <td>0.0515</td>
        <td>0.1590</td>
        <td>0.8008</td>
        <td>0.1138</td>
    </tr>
    <tr>
        <td rowspan="2">5</td>
        <td>w/o HC</td>
        <td><strong>0.2108</strong></td>
        <td><strong>0.1601</strong></td>
        <td>0.0507</td>
        <td><strong>0.7877</strong></td>
        <td><strong>0.1146</strong></td>
    </tr>
    <tr>
        <td>w/ HC</td>
        <td>0.2111</td>
        <td>0.0526</td>
        <td>0.1586</td>
        <td>0.7951</td>
        <td>0.1163</td>
    </tr>
    <tr>
        <td rowspan="2">6</td>
        <td>w/o HC</td>
        <td><strong>0.2121</strong></td>
        <td><strong>0.1606</strong></td>
        <td>0.0515</td>
        <td><strong>0.7858</strong></td>
        <td><strong>0.1155</strong></td>
    </tr>
    <tr>
        <td>w/ HC</td>
        <td>0.2123</td>
        <td>0.0526</td>
        <td>0.1597</td>
        <td>0.7893</td>
        <td>0.1167</td>
    </tr>
    <tr>
        <td rowspan="2">7</td>
        <td>w/o HC</td>
        <td><strong>0.2134</strong></td>
        <td><strong>0.1612</strong></td>
        <td>0.0522</td>
        <td><strong>0.7881</strong></td>
        <td><strong>0.1169</strong></td>
    </tr>
    <tr>
        <td>w/ HC</td>
        <td>0.2160</td>
        <td>0.0560</td>
        <td>0.1600</td>
        <td>0.7945</td>
        <td>0.1201</td>
    </tr>
  </tbody>
</table>

**Table B8**: Losses of Sig2Event model for three different historic window sizes $n_r \in \{5, 10, 40\}$ and for window size 40 only using $(x, y, T)$ as input for each $n_r \in \{3, 4, 5, 6, 7\}$. Test loss is the sum of MSE and CEL).

<table>
  <thead>
    <tr>
        <th>$n_r$</th>
        <th>Specification</th>
        <th>Test loss</th>
        <th>MSE</th>
        <th>CEL</th>
        <th>KL</th>
        <th>Brier</th>
    </tr>
  </thead>
  <tbody>
    <tr>
        <td rowspan="4">3</td>
        <td>40 past actions</td>
        <td>0.4036</td>
        <td>0.1652</td>
        <td>0.2383</td>
        <td>0.1121</td>
        <td>0.8034</td>
    </tr>
    <tr>
        <td>10 past actions</td>
        <td>0.3994</td>
        <td>0.1644</td>
        <td>0.2350</td>
        <td>0.1140</td>
        <td>0.8066</td>
    </tr>
    <tr>
        <td>5 past actions</td>
        <td>0.4014</td>
        <td>0.1644</td>
        <td>0.2370</td>
        <td>0.1138</td>
        <td>0.8053</td>
    </tr>
    <tr>
        <td>$x, y, T$ (40 past actions)</td>
        <td>0.4051</td>
        <td>0.1666</td>
        <td>0.2385</td>
        <td>0.1144</td>
        <td>0.8086</td>
    </tr>
    <tr>
        <td rowspan="4">4</td>
        <td>40 past actions</td>
        <td>0.4087</td>
        <td>0.1658</td>
        <td>0.2429</td>
        <td>0.1163</td>
        <td>0.8054</td>
    </tr>
    <tr>
        <td>10 past actions</td>
        <td>0.4047</td>
        <td>0.1645</td>
        <td>0.2402</td>
        <td>0.1168</td>
        <td>0.8033</td>
    </tr>
    <tr>
        <td>5 past actions</td>
        <td>0.4039</td>
        <td>0.1638</td>
        <td>0.2401</td>
        <td>0.1166</td>
        <td>0.8050</td>
    </tr>
    <tr>
        <td>$x, y, T$ (40 past actions)</td>
        <td>0.4100</td>
        <td>0.1661</td>
        <td>0.2439</td>
        <td>0.1151</td>
        <td>0.8029</td>
    </tr>
    <tr>
        <td rowspan="4">5</td>
        <td>40 past actions</td>
        <td>0.4101</td>
        <td>0.1652</td>
        <td>0.2449</td>
        <td>0.1177</td>
        <td>0.7984</td>
    </tr>
    <tr>
        <td>10 past actions</td>
        <td>0.4111</td>
        <td>0.1651</td>
        <td>0.2460</td>
        <td>0.1164</td>
        <td>0.7962</td>
    </tr>
    <tr>
        <td>5 past actions</td>
        <td>0.4079</td>
        <td>0.1639</td>
        <td>0.2440</td>
        <td>0.1219</td>
        <td>0.7995</td>
    </tr>
    <tr>
        <td>$x, y, T$ (40 past actions)</td>
        <td>0.4128</td>
        <td>0.1658</td>
        <td>0.2470</td>
        <td>0.1197</td>
        <td>0.7979</td>
    </tr>
    <tr>
        <td rowspan="4">6</td>
        <td>40 past actions</td>
        <td>0.4120</td>
        <td>0.1646</td>
        <td>0.2474</td>
        <td>0.1183</td>
        <td>0.7954</td>
    </tr>
    <tr>
        <td>10 past actions</td>
        <td>0.4090</td>
        <td>0.1637</td>
        <td>0.2453</td>
        <td>0.1207</td>
        <td>0.7972</td>
    </tr>
    <tr>
        <td>5 past actions</td>
        <td>0.4087</td>
        <td>0.1633</td>
        <td>0.2454</td>
        <td>0.1187</td>
        <td>0.7948</td>
    </tr>
    <tr>
        <td>$x, y, T$ (40 past actions)</td>
        <td>0.4159</td>
        <td>0.1681</td>
        <td>0.2478</td>
        <td>0.1186</td>
        <td>0.7925</td>
    </tr>
    <tr>
        <td rowspan="4">7</td>
        <td>40 past actions</td>
        <td>0.4193</td>
        <td>0.1642</td>
        <td>0.2551</td>
        <td>0.1202</td>
        <td>0.7998</td>
    </tr>
    <tr>
        <td>10 past actions</td>
        <td>0.4164</td>
        <td>0.1637</td>
        <td>0.2527</td>
        <td>0.1211</td>
        <td>0.8023</td>
    </tr>
    <tr>
        <td>5 past actions</td>
        <td>0.4160</td>
        <td>0.1635</td>
        <td>0.2524</td>
        <td>0.1224</td>
        <td>0.8041</td>
    </tr>
    <tr>
        <td>$x, y, T$ (40 past actions)</td>
        <td>0.4209</td>
        <td>0.1647</td>
        <td>0.2562</td>
        <td>0.1194</td>
        <td>0.8005</td>
    </tr>
  </tbody>
</table>

# Appendix C Possession value details

Expected goals (xG) models have emerged as popular tools for a more granular analysis of team and player performance. These models assign a probability of success to each shot, taking into account factors that influence the likelihood of scoring a goal from a shot. The earliest version of an xG model has already been proposed by Pollard and Reep (1997). The authors used a logistic regression to model the binary shot data and found that the most important factors for successful shots were the shot location and the angle between the shot and the two goalposts (henceforth, goal angle). Due to advancements in data collection and computing, nowadays powerful machine learning models are employed for the task of fitting an xG model Robberechts and Davis (2020); Anzer and Bauer (2021).

For this work and our possession value metric, we are only interested in a proxy for shot quality based on location and therefore follow a simple approach based on Pollard and Reep (1997) to fit an xG model. In particular, we model binary shots data via a logistic regression model, i.e., a model assuming a linear relationship between the log-odds of the probability of scoring from a shot $\pi(Z) = P(Y = 1 \mid Z)$ and features $Z$:

$$ \log \left( \frac{\pi(Z)}{1 - \pi(Z)} \right) = Z^\top \gamma, \tag{C1} $$

where the feature vector $Z$ contains the distance to the goal and the goal angle. We train our xG models on data from four big European leagues (Bundesliga, La Liga, Ligue 1, and Serie A), but we do not use Premier League data, to not interfere with the data used for our analyses in Section 4.

Expected threat (xT) was first popularized through a blog (Singh 2019). The idea is to estimate a value of each game state, where a game state is dependent only on the location on the pitch, via a Markov model. In particular, the pitch is divided into a grid of size $M \times N$ resulting in $M \cdot N$ location zones, for each of which a game state value is derived. These values for zone $z$ are derived by iteratively solving the equation

$$ \text{xT}(z) = P(S = 1|z) \cdot \text{xG} + (1 - P(S = 1|z)) \sum_{i=1}^{M \cdot N} T_{z,i} \, \text{xT}(i), \tag{C2} $$

where $P(S = 1|z)$ is the probability that a player shoots at location $z$ (hence, $1 - P(S = 1|z)$ is the probability of moving the ball instead of shooting), xG is a simple xG model as above, and $T_{z,i}$ is the $(z,i)$-th element of a transition matrix $T$, i.e., represents the probability of transitioning from zone $z$ to zone $i$. To solve the problem, the values for xT are initialized to zero and iterated for a number of times (ideally until some convergence is achieved). Intuitively, an xT value for zone $z$ derived from $k$ iterations represents the probability of scoring from the next $k$ actions, when starting in zone $z$. We fitted the above model on the same data as before with a $12 \times 16$ grid.

## References

Anzer, G., Bauer, P.: A Goal Scoring Probability Model for Shots Based on Synchronized Positional and Event Data in Football (Soccer). Frontiers in Sports and Active

Living **3**, 53 (2021) [https://doi.org/10.3389/fspor.2021.624475](https://doi.org/10.3389/fspor.2021.624475)

Boedihardjo, H., Geng, X., Lyons, T., Yang, D.: The signature of a rough path: Uniqueness. Advances in Mathematics **293**, 720–737 (2016) [https://doi.org/10.1016/j.aim.2016.02.011](https://doi.org/10.1016/j.aim.2016.02.011)

Bajons, R., Hornik, K.: Plus-minus models for evaluating and scouting soccer players using possession sequences. IMA Journal of Management Mathematics **36**(3), 593–614 (2025) [https://doi.org/10.1093/imaman/dpaf017](https://doi.org/10.1093/imaman/dpaf017)

Buehler, H., Horvath, B., Lyons, T., Perez Arribas, I., Wood, B.: Generating financial markets with signatures. Available at SSRN 3657366 (2020) [https://doi.org/10.2139/ssrn.3657366](https://doi.org/10.2139/ssrn.3657366)

Bayer, C., Hager, P.P., Riedel, S., Schoenmakers, J.: Optimal stopping with signatures. The Annals of Applied Probability **33**(1), 238–273 (2023) [https://doi.org/10.1214/22-AAP1814](https://doi.org/10.1214/22-AAP1814)

Chawla, S., Estephan, J., Gudmundsson, J., Horton, M.: Classification of passes in football matches using spatiotemporal data. ACM Trans. Spatial Algorithms Syst. **3**(2) (2017) [https://doi.org/10.1145/3105576](https://doi.org/10.1145/3105576)

Cuchiero, C., Gazzani, G., Svaluto-Ferro, S.: Signature-based models: Theory and calibration. SIAM journal on financial mathematics **14**(3), 910–957 (2023) [https://doi.org/10.1137/22M1512338](https://doi.org/10.1137/22M1512338)

Chen, K.-T.: Iterated integrals and exponential homomorphisms. Proceedings of the London Mathematical Society **3**(1), 502–512 (1954) [https://doi.org/10.1112/plms/s3-4.1.502](https://doi.org/10.1112/plms/s3-4.1.502)

Chen, K.-T.: Integration of paths—a faithful representation of paths by noncommutative formal power series. Transactions of the American Mathematical Society **89**(2), 395–407 (1958) [https://doi.org/10.1090/S0002-9947-1958-0106258-0](https://doi.org/10.1090/S0002-9947-1958-0106258-0)

Chevyrev, I., Kormilitzin, A.: A Primer on the Signature Method in Machine Learning (2025). [https://doi.org/10.48550/arXiv.1603.03788](https://doi.org/10.48550/arXiv.1603.03788)

Cho, K., Merrienboer, B., Bahdanau, D., Bengio, Y.: On the Properties of Neural Machine Translation: Encoder-Decoder Approaches (2014). [https://doi.org/10.48550/arXiv.1409.1259](https://doi.org/10.48550/arXiv.1409.1259)

Davis, J., Bransen, L., Devos, L., Jaspers, A., Meert, W., Robberechts, P., Van Haaren, J., Van Roy, M.: Methodology and evaluation in sports analytics: challenges, approaches, and lessons learned. Machine Learning **113**(9), 6977–7010 (2024) [https://doi.org/10.1007/s10994-024-06585-0](https://doi.org/10.1007/s10994-024-06585-0)

Elman, J.L.: Finding structure in time. Cognitive Science **14**(2), 179–211 (1990) https://doi.org/10.1016/0364-0213(90)90002-E90002-E)

//doi.org/10.1207/s15516709cog1402_1

Fernández, J., Bornn, L., Cervone, D.: A framework for the fine-grained evaluation of the instantaneous expected value of soccer possessions. Machine Learning **110**(6), 1389–1427 (2021) [https://doi.org/10.1007/s10994-021-05989-6](https://doi.org/10.1007/s10994-021-05989-6)

Hvattum, L.M., Gelade, G.A.: Comparing bottom-up and top-down ratings for individual soccer players. International Journal of Computer Science in Sport **20**(1), 23–42 (2021) [https://doi.org/10.2478/ijcss-2021-0002](https://doi.org/10.2478/ijcss-2021-0002)

Hambly, B., Lyons, T.: Uniqueness for the signature of a path of bounded variation and the reduced path group. Annals of Mathematics **171**(1), 109–167 (2010) [https://doi.org/10.4007/annals.2010.171.109](https://doi.org/10.4007/annals.2010.171.109)

Hochreiter, S., Schmidhuber, J.: Long short-term memory. Neural Computation **9**(8), 1735–1780 (1997) [https://doi.org/10.1162/neco.1997.9.8.1735](https://doi.org/10.1162/neco.1997.9.8.1735)

Håland, E.M., Wiig, A.S., Stålhane, M., Hvattum, L.M.: Evaluating passing ability in association football. IMA Journal of Management Mathematics **31**(1), 91–116 (2019) [https://doi.org/10.1093/imaman/dpz004](https://doi.org/10.1093/imaman/dpz004)

Kormilitzin, A., Saunders, K.E., Harrison, P.J., Geddes, J.R., Lyons, T.: Detecting early signs of depressive and manic episodes in patients with bipolar disorder using the signature-based model. arXiv preprint (2017) [https://doi.org/10.48550/arXiv.1708.01206](https://doi.org/10.48550/arXiv.1708.01206)

Lolli, L., Bauer, P., Irving, C., Bonanno, D., Höner, O., Gregson, W., Salvo, V.D.: Data analytics in the football industry: a survey investigating operational frameworks and practices in professional clubs and national federations from around the world. Science and Medicine in Football **9**(2), 189–198 (2025) [https://doi.org/10.1080/24733938.2024.2341837](https://doi.org/10.1080/24733938.2024.2341837) . PMID: 38745403

Lyons, T., Caruana, M., Lévy, T.: Differential Equations Driven by Rough Paths. Lecture Notes in Mathematics, vol. 1908, p. 116. Springer, Berlin, Heidelberg (2007). [https://doi.org/10.1007/978-3-540-71285-5](https://doi.org/10.1007/978-3-540-71285-5)

Liao, S.: Log signatures in machine learning. PhD thesis, UCL (University College London) (2022)

Liao, S., Lyons, T., Yang, W., Ni, H.: Learning stochastic differential equations using RNN with log signature features. arXiv preprint (2019) [https://doi.org/10.48550/arXiv.1908.08286](https://doi.org/10.48550/arXiv.1908.08286)

Lyons, T.J.: Differential equations driven by rough signals. Revista Matemática Iberoamericana **14**(2), 215–310 (1998)

Morrill, J., Fermanian, A., Kidger, P., Lyons, T.: A generalised signature method for

multivariate time series feature extraction. arXiv preprint (2020) [https://doi.org/10.48550/arXiv.2006.00873](https://doi.org/10.48550/arXiv.2006.00873)

Moore, P., Lyons, T., Gallacher, J., Initiative, A.D.N.: Using path signatures to predict a diagnosis of Alzheimer’s disease. PloS one **14**(9), 0222212 (2019) [https://doi.org/10.1371/journal.pone.0222212](https://doi.org/10.1371/journal.pone.0222212)

Mendes-Neves, T., Meireles, L., Mendes-Moreira, J.: Towards a foundation large events model for soccer. Machine Learning **113**(11), 8687–8709 (2024) [https://doi.org/10.1007/s10994-024-06606-y](https://doi.org/10.1007/s10994-024-06606-y)

Ni, H., Szpruch, L., Sabate-Vidales, M., Xiao, B., Wiese, M., Liao, S.: Sig-Wasserstein GANs for Time Series Generation. Proceedings of the Second ACM International Conference on AI in Finance, 1–8 (2021)

Pappalardo, L., Cintia, P., Rossi, A., Massucco, E., Ferragina, P., Pedreschi, D., Giannotti, F.: A public dataset of spatio-temporal match events in soccer competitions. Scientific Data **6**(1), 236 (2019) [https://doi.org/10.1038/s41597-019-0247-7](https://doi.org/10.1038/s41597-019-0247-7)

Pollard, R., Reep, C.: Measuring the effectiveness of playing strategies at soccer. Journal of the Royal Statistical Society: Series D (The Statistician) **46**(4), 541–550 (1997) [https://doi.org/10.1111/1467-9884.00108](https://doi.org/10.1111/1467-9884.00108)

Robberechts, P., Davis, J.: How Data Availability Affects the Ability to Learn Good xG Models. In: Brefeld, U., Davis, J., Van Haaren, J., Zimmermann, A. (eds.) Machine Learning and Data Mining for Sports Analytics, pp. 17–27. Springer, Cham (2020). [https://doi.org/10.1007/978-3-030-64912-8_2](https://doi.org/10.1007/978-3-030-64912-8_2)

Simpson, I., Beal, R.J., Locke, D., Norman, T.J.: Seq2event: Learning the language of soccer using transformer-based match event prediction. In: Proceedings of the 28th ACM Sigkdd Conference on Knowledge Discovery and Data Mining, pp. 3898–3908 (2022). [https://doi.org/10.1145/3534678.3539138](https://doi.org/10.1145/3534678.3539138)

Singh, K.: Expected Threat Blogpost. [https://karun.in/blog/expected-threat.html](https://karun.in/blog/expected-threat.html). Accessed: 18.8.2025 (2019)

Szczepański, L., McHale, I.: Beyond completion rate: evaluating the passing ability of footballers. Journal of the Royal Statistical Society. Series A (Statistics in Society) **179**(2), 513–533 (2016) [https://doi.org/10.1111/rssa.12115](https://doi.org/10.1111/rssa.12115)

Spearman, W.: Beyond expected goals. In: MIT Sloan Sports Analytics Conference (2018)

Sicilia, A., Pelechrinis, K., Goldsberry, K.: Deephoops: Evaluating micro-actions in basketball using deep feature representations of spatio-temporal data. In: Proceedings of the 25th ACM SIGKDD International Conference on Knowledge Discovery & Data Mining. KDD '19, pp. 2096–2104. Association for Computing Machinery,

New York, NY, USA (2019). [https://doi.org/10.1145/3292500.3330719](https://doi.org/10.1145/3292500.3330719)

Sturm, S.: Path Signatures for Feature Extraction. An Introduction to the Mathematics Underpinning an Efficient Machine Learning Technique (2025). [https://doi.org/10.48550/arXiv.2506.01815](https://doi.org/10.48550/arXiv.2506.01815)

Shi, D., Zhang, X., Cheng, J., Xiong, T., Ni, H.: Adaptive global gesture paths and signature features for skeleton-based gesture recognition. In: Antonacopoulos, A., Chaudhuri, S., Chellappa, R., Liu, C.-L., Bhattacharya, S., Pal, U. (eds.) Pattern Recognition, pp. 278–292. Springer, Cham (2025). [https://doi.org/10.1007/978-3-031-78354-8_18](https://doi.org/10.1007/978-3-031-78354-8_18)

Van Roy, M., Robberechts, P., Decroos, T., Davis, J.: Valuing On-the-Ball Actions in Soccer: A Critical Comparison of xT and VAEP. In: Proceedings of the AAAI-20 Workshop on Artificial Intelligence in Team Sports, pp. 1–8 (2020)

Vaswani, A., Shazeer, N., Parmar, N., Uszkoreit, J., Jones, L., Gomez, A. N., Kaiser, L., Polosukhin, I.: Attention Is All You Need (2023). [https://doi.org/10.48550/arXiv.1706.03762](https://doi.org/10.48550/arXiv.1706.03762)

Watson, N., Hendricks, S., Stewart, T., Durbach, I.: Integrating machine learning and decision support in tactical decision-making in rugby union. Journal of the Operational Research Society **72**(10), 2274–2285 (2021) [https://doi.org/10.1080/01605682.2020.1779624](https://doi.org/10.1080/01605682.2020.1779624)

Yang, W., Lyons, T., Ni, H., Schmid, C., Jin, L., Chang, J.: Leveraging the path signature for skeleton-based human action recognition. arXiv preprint **1** (2017) [https://doi.org/10.48550/arXiv.1707.03993](https://doi.org/10.48550/arXiv.1707.03993)

Yang, W., Lyons, T., Ni, H., Schmid, C., Jin, L.: Developing the path signature methodology and its application to landmark-based human action recognition. In: Stochastic Analysis, Filtering, and Stochastic Optimization, pp. 431–464. Springer, Cham (2022). [https://doi.org/10.1007/978-3-030-98519-6_18](https://doi.org/10.1007/978-3-030-98519-6_18)

Yeung, C.-H., Sit, T., Fujii, K.: Transformer-based neural marked spatio temporal point process model for analyzing football match events. Applied Intelligence **55**, 335 (2025) [https://doi.org/10.1007/s10489-024-05996-9](https://doi.org/10.1007/s10489-024-05996-9)

Zhang, Q., Zhang, X., Hu, H., Li, C., Lin, Y., Ma, R.: Sports match prediction model for training and exercise using attention-based lstm network. Digital Communications and Networks **8**(4), 508–515 (2022) [https://doi.org/10.1016/j.dcan.2021.08.008](https://doi.org/10.1016/j.dcan.2021.08.008)