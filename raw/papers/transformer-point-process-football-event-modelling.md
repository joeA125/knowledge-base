# Transformer-Based Neural Marked Spatio Temporal Point Process Model for Football Match Events Analysis

Calvin C. K. Yeung<sup>1</sup>, Tony Sit<sup>2</sup>, and Keisuke Fujii<sup>1,3,4</sup>

<sup>1</sup> Graduate School of Informatics, Nagoya University, Nagoya, Japan.

<sup>2</sup> Department of Statistics, The Chinese University of Hong Kong, Shatin, Hong Kong SAR.

<sup>3</sup> Center for Advanced Intelligence Project, RIKEN, Fukuoka, Japan.

<sup>4</sup> PRESTO, Japan Science and Technology Agency, Saitama, Japan.

**Abstract.** With recently available football match event data that record the details of football matches, analysts and researchers have a great opportunity to develop new performance metrics, gain insight, and evaluate key performance. However, most sports sequential events modeling methods and performance metrics approaches could be incomprehensive in dealing with such large-scale spatiotemporal data (in particular, temporal process), thereby necessitating a more comprehensive spatiotemporal model and a holistic performance metric. To this end, we proposed the Transformer-Based Neural Marked Spatio Temporal Point Process (NMSTPP) model for football event data based on the neural temporal point processes (NTPP) framework. In the experiments, our model outperformed the prediction performance of the baseline models. Furthermore, we proposed the holistic possession utilization score (HPUS) metric for a more comprehensive football possession analysis. For verification, we examined the relationship with football teams’ final ranking, average goal score, and average xG over a season. It was observed that the average HPUS showed significant correlations regardless of not using goal and details of shot information. Furthermore, we show HPUS examples in analyzing possessions, matches, and between matches.

**Keywords:** neural point process · deep learning · event data · sports · football

## 1 Introduction

Football<sup>5</sup> has simultaneously been an influential sport and important industry around the globe [15]. Estimate has shown that the FIFA World Cup Qatar 2022 entertained over 5 billion viewers<sup>6</sup>, and it has also been established to be a pillar industry for countries like Italy and Great Britain [35]. In modern football, data analysis plays an important role for fans, players, and coaches. Data analysis can be leveraged by players to improve performance, and generate

insight for the coaching process and tactical decision-making. Furthermore, it provides fans with quantified measures and deeper insights into the game [2, 25]. For years, analysis and research have focused on players’ actions when they are in possession of the ball, with each player statistically shown to have 3 minutes on average with the ball [9]. Therefore, it is critical for players to utilize these three minutes, and for analysts and researchers to evaluate the effectiveness and efficiency of these on-ball action events.

There have been numerous attempts in the existing sports analysis literature to understand how the sequence of events in the past affects the next event or its outcome. The majority of the studies [21, 24, 25, 32, 35] used machine learning (ML) models to handle the long sequences of event data. The ML models encoded the long history events vectors (each vector representing a historical event, and with the event being described by features in the vector) into a vector representation by incorporating Long Short-Term Memory (LSTM) [10], Gated Recurrent Unit (GRU) [3], or Transformer Encoder [31].

The Seq2Event model was recently proposed by Simpson et al. [25]. The model combines the encoding method and dense layers to forecast the location and type of the next event. Furthermore, the poss-util metric is developed based on the predicted event probability to evaluate the effectiveness of a football team’s possession. However, in order to assess the effectiveness and efficiency of the team’s on-ball actions, all three factors, temporal, spatial, and action type, should be considered and modeled dependently. For example, shots are frequently taken when the players are close to the opponent’s goalpost, but rarely when they are far away. Furthermore, if it takes a long time before the team gets the ball close to the opponent’s goal, the opponent will have sufficient time to react, with shot-taking no longer being the best option. Such an example demonstrates the need for a dependent model capable of handling all three factors.

In this paper, we proposed modeling football event data under the Neural temporal point process (NTPP) framework [23]. The proposed model utilizes the long sequence encoding method and models the forecast of the next event’s temporal, spatial, and action types factors under the point process literature. First, we introduced method of modeling the football match event data under the NTPP framework, explaining how to model the event factors dependently. Second, we presented the best-fitted model Transformer-Based Neural Marked Spatio Temporal Point Process Model (NMSTPP). The model is capable of modeling temporal information of events, which has been overlooked in previous studies. Finally, we demonstrated how the NMSTPP model can be applied for evaluating the effectiveness and efficiency of the team’s possession by proposing a new performance metric: Holistic possession utilization score (HPUS).

Summarily, our main contributions are as follows: (1) With the NTPP framework, we proposed the NMSTPP model to model football events data interevent time, location, and action simultaneously and dependently; (2) To evaluate possession in football, we have proposed a more holistic metric, HPUS. The HPUS validation and application have been provided in this paper; (3) Using open-source data, we determined the optimal architecture and validated the forecast

results of the model. The ablation study of the NMSTPP model showed that the dependency could increase forecast performance and the validated HPUS showed the necessity of simultaneous modeling for holistic analysis.

# 2 Proposed framework

In this section, we describe how to model football event data under the NTPP framework and use the model to evaluate a possession period. In Section 2.1, we first describe how we define football event data as a point process and how we incorporate ML to form the NMSTPP model. Afterward, in Section 2.2, we introduce the model architecture of the NMSTPP model. Lastly, in Section 2.3, we describe the HPUS for possession period evaluation.

## 2.1 Define football event data as NMSTPP

Although there are multiple ways to define a point process, for a temporal point process $\{(t_i)\}_{i=1}^N$, one method is by defining the conditional probability density function (PDF) $f(t_i|H_i)$ of the interevent time for the next event $t_i$ given the history event $H_i = \{t_1, t_2, ..., t_{i-1}\}$ [22]. With factorization, the joint PDF of the events' interevent time can be represented with the following formula [22]:

$$ f(..., t_1, t_2, ...) = \prod_{i=1}^N f(t_i|t_1, t_2, ..., t_{i-1}) = \prod_{i=1}^N f(t_i|H_i) \eqno(1) $$

Furthermore, by taking the marks $m$ and spatial $z$ information of an event into consideration, the joint PDF of a marked spatio temporal point process (MSTPP) $\{(t_i, z_i, m_i)\}_{i=1}^N$ can be extended from equation 1 and represented as follows:

$$ f(..., (t_1, z_1, m_1), (t_2, z_2, m_2), ...) = \prod_{i=1}^N f(t_i, z_i, m_i|H_i) \eqno(2) $$

Prior to defining the conditional PDF for MSTPP, we first connected the football match on-ball action event data with MSTPP. The marks $m$ correspond to the on-ball action type (e.g., shot, cross, pass, and so on), spatial $z$ corresponds to the location (zone) of the football pitch indicating where the event happened (further explained in Section 3.1), and temporal $t$ corresponds to the interevent time.

Afterward, rather than defining the conditional PDF (PMF) for MSTPP in equation 2 directly, we applied the decomposition of multivariate density function [4] to equation 2 [19]. This results in conditional PDF as follows:

$$ \prod_{i=1}^N f(t_i, z_i, m_i|H_i) = \prod_{i=1}^N f_t(t_i|H_i) f_z(z_i|t_i, H_i) f_m(m_i|t_i, z_i, H_i). \eqno(3) $$

This equation is the multiplication of $t, z, m$ conditional PDF, where $t, z,$ and $m$ are interchangeable. Using equation 3 allows us to define PDFs $f_t, f_z, f_m$ differently, but without assuming $t, z,$ and $m$ are independent as long as the defined conditional PDFs take all given information into consideration.

Although defining the PDFs with distributions or models based on point processes (e.g., Poisson process [14], Hawkes process [11], Reactive point process [8], and so on) are common, we applied ML algorithms to estimate the PDFs. This has been proven to be more effective in multiple fields [6,33,34,36]. Based on maximum negative log-likelihood estimation, the MSTPP loss function to be minimized can be presented as follows.

$$ L(\theta) = \sum_{i=1}^{N} 10 \times RMSE_{t_i} + CEL_{z_i} + CEL_{m_i}. \eqno(4) $$

This equation composes of the root mean square error (RMSE) for $t$ and cross-entropy loss (CEL) for $z, m$. The CELs are weighted to deal with unbalanced classes (more details in Appendix B) and RMSE was multiplied by 10 to keep the balance between the three cost functions.

It should be noted that taking all events data as input directly for the ML model would be ineffective and inefficient. The data may consist of a large amount of noise, with large amount of input features potentially increasing the number of trainable parameters in the models, and consequently leading to a long training time. The feasible solution from the NTPP framework [23] is to encode the information from the history events information $(\vec{y}_1, \vec{y}_2, ... \vec{y}_{i-1}), \vec{y}_i = [t_i, m_i, z_i]$ into a fixed-size single vector $\vec{h}_i$ with LSTM [10], GRU [3], or Transformer Encoder [31]. Based on a previous study [25], Transformer Encoder has been found to be slightly less effective, but significantly more efficient than LSTM. Therefore, in this study, we applied Transformer Encoder to encode the history events information. Furthermore, MSTPP models that are based on the combination of point process literature and ML methods can be referred to as Neural MSTPP (NMSTPP) models.

## 2.2 NMSTPP model architecture

In this subsection, the NMSTPP model architecture and related hyperparameter are explained. The NMSTPP model with the optimal hyperparameter is presented in Fig. 1. The grid searched hyperparameter values are presented in Appendix B, while a more detailed description of transformer encoder [31] and NTPP [23] are presented in Appendix D and E, respectively.

**Stage 1: Input.** First, we summarized the features set for the model. For each event $(t_j, z_j, m_j), j \in [i - seqlen : i - 1]$, we used the following input features, which resulted in a matrix of size $(seqlen, 1 + 1 + 1 + 5)$:

* Interevent time $t_i$: the time between the current event and the previous event.

* Zone $z_i$: zone on the football pitch where the event takes place; the zone number was assigned randomly from 1 to 20 (more details on Section 3.1).

* Action $m_i$: type of action in the event; feasible actions are pass $p$, possession end \_, dribble $d$, cross $x$, and shot $s$.

* Other continuous features: engineered features mainly describe the change in zone (further explanation in Appendix C).

NMSTPP model architecture diagram showing 5 stages: 1. Input, 2. Historical encoding, 3. Forecasting, 4. Output, and 5. Cost function. The diagram details the flow from historical events (j = [i-40 : i-1]) through embedding, dense layers, and a transformer encoder to forecast the next event (j = i).

**Fig. 1.** NMSTPP model architecture. (Stage 1) The input of the model, interevent time, zone, action, and other continuous features of events at $j \in [i - 40 : i - 1]$ (here, we set *seqlen* to 40). (Stage 2) Apply embedding and dense layer to the input, with positional encoding and transformer encoder to obtain the history vector and pass the vector through a dense layer. (Stage 3) Apply neural network to forecast the interevent time, zone, and action of event $j = i$. (Stage 4) The outputs of the model are one value for interevent time, 20 logits for zone, and 5 logits for action. (Stage 5) The output in stage 4 and the ground truth are used to calculate the cost function directly.

Hyperparameter: *seqlen*, the sequence length of the historical events.

**Stage 2: History encoding.** In this stage, a dense layer is first applied to interevent time $t_i$ and other continuous features, with an embedding layer applied to zone $z_i$ and action $m_i$ respectively, allowing the model to better capture information in the features [25]. Afterward, with the position encoding and transformer encoder from the Transformer model [31] (more details on Appendix D), a fixed-size encoded history vector with size (31) can be retrieved. Lastly, the history vector passes through another dense layer to allow better information capturing [25].

Hyperparameter: *dim_feedforward*, numbers of feedforward layers in the transformer encoder.

**Stage 3: Forecasting.** The purpose of this stage is to forecast the interevent time, zone, and action of the next event $(t_i, z_i, m_i)$. In general, we estimated the conditional PDFs in equation 3 with neural network (NN). Specifically, for zone $z$ and action $m$, the NNs are estimating the conditional probability mass function (PMF) as they are discrete classes. On the other hand, we decided to model the relationship between history $H$ and the interevent time $t$ directly. As a result, with the history vector $H$, the models for forecasting can approximately be presented in the following formulas:

$$
\begin{aligned}
f_t(t_i | H_i) &\approx NN_t(H_i) = t_i \\
f_z(z_i | t_i, H_i) &\approx NN_z(t_i, H_i) = \vec{z_i} \\
f_m(m_i | t_i, z_i, H_i) &\approx NN_m(t_i, \vec{z_i}, H_i) = \vec{m_i}
\end{aligned}
\eqno(5)
$$

where the outputs of neural networks $NN_z$ and $NN_m$: $\vec{z_i}$ and $\vec{m_i}$ are a vector of predicted logits for all zones and action types with sizes 20 and 5 respectively.

Hyperparameter:

- *order* : order of $t, z,$ and $m$ in Equation 5, which are interchangeable.

- *num_layer* : numbers of hidden layers for $NN$, where $NN_t$, $NN_z$, and $NN_m$ can have different numbers of hidden layers.

- *activation_function* : activation function for hidden layers in $NN_t$, $NN_z$, and $NN_m$.

- *drop_out* : dropout rate for hidden layers in $NN_t$, $NN_z$, and $NN_m$.

**Stage 4: Output.** The final outputs of the model are $t_i, \vec{z_i}, \vec{m_i}$. We considered the class with max logit in $\vec{z_i}, \vec{m_i}$ as the predicted class. When probabilities are required, we scaled the logits into range [0,1].

**Stage 5: Cost function.** Furthermore, $t_i, \vec{z_i}, \vec{m_i}$, and the ground truth were used to calculate the cost function directly. The cost function in equation 4 would still apply after the modification in stage 3. With the cost function, the NMSTPP model can be trained from end to end with a gradient descent algorithm, in which the popular adam optimizer [13] has been selected.

## 2.3 Holistic possession utilization score (HPUS)

For a more comprehensive possession analysis in football, we developed the holistic possession utilization score (HPUS) metric by extending poss-util metric [25]. The poss-util is a metric for analyzing possession utilization. Firstly, the attack probability is calculated by summing the predicted probability of the cross and shot of an event. Then, the attack probability of $n$ events in possession to obtain the poss-util is summed. The calculation is concluded with equation 6. Furthermore, -1 is multiplied to the poss-util if a shot or cross event does not exist in the possession. Lastly, the percentile rank is applied to both positive and negative poss-util, with the resulting metrics poss-util in range [-1,1].

$$ \text{poss-util} = \sum_{i=1}^{n} P(\text{Cross, Shot}) \eqno(6) $$

On the other hand, with the NMSTPP model, the expected interevent time, zone, and action type can be calculated and applied to the metrics. Given the information, we proposed the HPUS for analyzing the effectiveness and efficiency of a possession period. The calculations of HPUS are presented as follows.

Holistic action score (HAS) $\in$ [0:10] is first computed as follows:

$$ \text{HAS} = \frac{\sqrt{E(Zone \cdot Action|H)}}{t} = \frac{\sqrt{E(Zone|H)E(Action|Zone, H)}}{t}, \eqno(7) $$

$$E(zone|H) = 0P(Area_0) + 5P(Area_1) + 10P(Area_2), \tag{8}$$

$$
\begin{aligned}
E(Action|Zone, H) = & 0P(\text{Possession loss}) + 5P(\text{Dribble, Pass}) \\
& + 10P(\text{Cross, Shot}),
\end{aligned} \tag{9}
$$

$$
t = \begin{cases} 1, & \text{if } t < 1, \\ t, & \text{o/w}, \end{cases} \tag{10}
$$

In equation 7, the expected value of zone and action were used to evaluate the effectiveness of each action. The multiplication of the two expected values allows for a more detailed score assignment. In HAS, a shot is assigned with a high score of 10, but the distance affects how likely the shot will lead to goal-scoring. Consequently, depending on the distance to the opponent’s goal, the score should be lower when far from the opponent’s goal and vice versa. In addition, the assignment of areas in equation 8 is visualized in Fig. 13.

Furthermore, the division of interevent time is used to account for the efficiency of the action. The more efficient the action is, the less time it takes, and the harder for the opponent to respond. Therefore, a higher score is awarded for less time taken. Additionally, we took the square root to scale the score in range [0:10] and let $t = 1$ if $t < 1$ to avoid the score from exploding.

HPUS summarizes the actions in possession, and is computed as follows:

$$HPUS = \sum_{i=1}^{n} \phi(n + 1 - i) \frac{\sqrt{E(Zone_i \cdot Action_i|H_i)}}{E(Time|H_i)} = \sum_{i=1}^{n} \phi(n + 1 - i) \text{HAS}_i, \tag{11}$$

$$\phi(x) = exp(-0.3(x - 1)). \tag{12}$$

In equation 11, for each possession with $n$ actions, the HPUS was calculated as the weighted sum of the $n$ actions’ HAS. The weights assignment starts from the last action and the weights are calculated with an exponentially decaying function as in equation 12, and visualized in Fig. 14. This exponentially decaying function allows the HPUS to give the most focus on the last action, which is the result of the entire possession period. In addition, the remaining actions were given lesser focus as they get far away from the last action. As a result, the HPUS is able to reflect the final outcome and the performance in the possession period at the same time.

Furthermore, similar to poss-util metric [25] we created HPUS+ that only considers possession that leads to an attack (cross or shots) at the end of the possession.

# 3 Experiments

In this section, we validate the architecture and the performance of the NMSTPP model and the HPUS. The training, validation, and testing set include 73, 7, and 178 matches, respectively, with more details of the dataset splitting presented in table 4. The code is available at [https://github.com/calvinyeungck/Football-Match-Event-Forecast](https://github.com/calvinyeungck/Football-Match-Event-Forecast). All models were trained with two AMD EPYC 7F72 24-Core Processors and one Nvidia RTX A6000.

## 3.1 Dataset and preprocessing

**Dataset.** Based on the 2017/2018 football season, we used football match event data from the top five leagues, the Premier League, La Liga, Ligue 1, Serie A, and Bundesliga. The event data used in this study were retrieved from the WyScout Open Access Dataset [20]. Currently, this dataset is the largest for football match event data, and it is published for the purpose of facilitating research in football data analytic development. In the event data, the action of the player who controls the football is captured in the event data in this dataset. Including the type of action (passes, shots, fouls, and so on), there are 21 action types and 78 subtypes in total. Further, the football pitch position where the action happens, is recorded in (x,y) coordinates, along with the time the event happens, the outcome of the action, amongst others. In addition, the xG data for validation were retrieved from [https://understat.com/](https://understat.com/). More details of dataset preprocessing are presented in Appendix A.

**Features engineering.** In most football match on-ball action events data, including the WyScout Open Access Dataset [20], the record of location and action are usually (x,y) coordinated and detailed with classified action types. However, to increase the explainability and reduce the complexity of the data, the (x,y) coordinates are first grouped into 20 zones (numbered randomly) according to the Juego de posición (position game) method. This method has been applied by famous football coach Pep Guardiola and the famous football team Bayern Munich in training. The grouping method allows the output of our model to provide a clear indication for football coaches and players. Moreover, detailed classified action types are grouped into 5 action classes (pass, dribble, cross, shot, and possession end). Similar methods have been applied in previous studies [25,30] and have proven to be effective. More details and summary of football pitch (x,y) coordinates, 20 zones, and 5 action classes have been provided in Figs. 7, 8, and 9, and Table 7.

Furthermore, from the created zone feature, we created extra features to provide the model with more information. The extra features include the distance from the previous zone to the current zone, change in the zone (x,y) coordinates, and distance and angle from the opposition goal center point to the zone. Detailed description of the extra features is presented in Appendix C.

## 3.2 Comparison with baseline models

To show the effectiveness and efficiency of the NMSTPP model, we compared the NMSTPP model with baseline models. The baseline models we applied are the statistical model and modified Seq2event model [25]. The statistical model is a combination of the second-order autoregression AR(2) model for interevent time forecast and transition probabilities for estimating the PMF for zones and actions. Three modified Seq2event models were obtained by first adding one extra output on the last dense layer serving as the interevent time forecast and trained with the cost function in equation 4. Furthermore, in historical encoding, the transformer encoder (Transformer) and unidirectional LSTM (Uni-LSTM)

were applied. Additionally, for a fair comparison, we fine-tuned the Modified Seq2Event’s (Transformer) transformer encoder layer feedforward network dimension, increasing it from 8 to 2048.

**Table 1.** Quantitative comparisons with baseline models. Model total loss, RMSE on interevent time $t$, CEL on zone, CEL on action, training time (in minutes), and the number of trainable parameters (in thousand) are reported.

<table>
  <thead>
    <tr>
        <th>Model</th>
        <th>Total loss</th>
        <th>$RMSE_t$</th>
        <th>$CEL_{zone}$</th>
        <th>$CEL_{action}$</th>
        <th>$T_{training}$ (min)</th>
        <th>Params (K)</th>
    </tr>
  </thead>
  <tbody>
    <tr>
        <td>AR(2)-Trans-prob</td>
        <td>6.98</td>
        <td>0.12</td>
        <td>2.34</td>
        <td>3.40</td>
        <td>N/A</td>
        <td>N/A</td>
    </tr>
    <tr>
        <td>Modified Seq2Event (Transformer)</td>
        <td>4.57</td>
        <td>0.11</td>
        <td>2.11</td>
        <td>1.39</td>
        <td>47</td>
        <td>13</td>
    </tr>
    <tr>
        <td>Modified Seq2Event (Uni-LSTM)</td>
        <td>4.51</td>
        <td>0.10</td>
        <td>2.11</td>
        <td>1.37</td>
        <td>129</td>
        <td>4</td>
    </tr>
    <tr>
        <td>Fine-tuned Seq2Event (Transformer)</td>
        <td>4.48</td>
        <td>0.10</td>
        <td>2.09</td>
        <td>1.36</td>
        <td>79</td>
        <td>137</td>
    </tr>
    <tr>
        <td>NMSTPP</td>
        <td><strong>4.40</strong></td>
        <td>0.10</td>
        <td>2.04</td>
        <td>1.33</td>
        <td>49</td>
        <td>79</td>
    </tr>
  </tbody>
</table>

Table 1 compares the performance based on the validation set, training time, and the number of trainable parameters the model had. In terms of effectiveness, The NMSTPP model had the best performance in forecasting the validation set matches events. Compared with the baseline models, the NMSTPP model outperformed in the total loss, zone CEL loss, and action CEL loss, and shared the best interevent time $t$ RMSE performance. In terms of efficiency, the modified Seq2event model (Transformer)[25] had the fastest training time, followed by the NMSTPP model (+2 min). However, the NMSTPP model had significantly 66 thousand more trainable parameters than the modified Seq2event model, and better performed (−0.17) in total loss. Overall, the NMSTPP model was the most effective and relatively efficient model, showing our methods could better model the football event data.

## 3.3 Ablation Studies

Upon validating the effectiveness and efficiency of the NMSTPP model, we validated the architecture of the NMSTPP model. First, we focused on stage 3 of the model (Section 2.2), comparing the performance when the forecasting models for interevent time, zones, and actions are dependent and independent (i.e., $NN_t, NN_z, NN_m$ in equation 5 will be a function of only $H_i$).

**Table 2.** Performance comparisons with disconnected NMSTPP models on the validation set. Model total loss, RMSE on interevent time $t$, CEL on zone, and CEL on action are reported.

<table>
  <thead>
    <tr>
        <th>Dependence</th>
        <th>Total loss</th>
        <th>$RMSE_t$</th>
        <th>$CEL_{zone}$</th>
        <th>$CEL_{action}$</th>
    </tr>
  </thead>
  <tbody>
    <tr>
        <td>Independent NMSTPP</td>
        <td>4.44</td>
        <td>0.10</td>
        <td>2.04</td>
        <td>1.37</td>
    </tr>
    <tr>
        <td>Dependent NMSTPP</td>
        <td><strong>4.40</strong></td>
        <td>0.10</td>
        <td>2.04</td>
        <td>1.33</td>
    </tr>
  </tbody>
</table>

Table 2 compares the performance of the independent NMSTPP and dependent NMSTPP. The result implied that the dependent NMSTPP had a better

performance than the independent NMSTPP by 0.04 total loss, with the difference coming from the CEL of action. Therefore, it is necessary to model the forecasting model for interevent time, zones, and actions dependently, as in equation 5.

In addition, we compared the use of (x,y) coordinate [25] and zone features in this study. Table 3 compares NMSTPP model’s RMSE of interevent time $t$ and CEL of action when using the two features. The result indicated that there are no significant differences in the performance. Therefore, the use of zone did not decrease the performance of the NMSTPP model, but could increase the explainability of the model’s output for football players and coaches.

**Table 3.** Performance comparisons with (x,y) coordinates features on the validation set. Model RMSE on interevent time $t$ and CEL on action are reported.

<table>
  <thead>
    <tr>
        <th>Features set</th>
        <th>$RMSE_t$</th>
        <th>$CEL_{action}$</th>
    </tr>
  </thead>
  <tbody>
    <tr>
        <td>zone</td>
        <td>0.10</td>
        <td>1.33</td>
    </tr>
    <tr>
        <td>(x,y)</td>
        <td>0.10</td>
        <td>1.33</td>
    </tr>
  </tbody>
</table>

## 3.4 Model verification

In this subsection, we further analyze the prediction result of the NMSTPP model. The following results are based on the forecast of the NMSTPP model on the testing set. The model was trained with a slightly adjusted CEL weight of the action dribble for higher accuracy in the dribble class (more details in Appendix B).

First, we analyzed the use of long sequences (40) of historical events. Fig. 3 (left) shows the self-attention score heatmap for the last row of the self-attention matrix. The score identified the contribution of each historical event to the history vector. In the heatmap, the weights of the events were between 0.01 and 0.06, and there were no trends or indications implying that the length of the historical events sequence 40 was either too long or too short.

Second, we analyzed the forecast of interevent time by comparing the CDF of the predicted interevent time and the true interevent time. Fig. 2 (right) shows that the CDF of the predicted and true interevent time were generally matched. Therefore, even without specifying a distribution for interevent time, the NMSTPP model could match the sample distribution.

Lastly, we analyzed the forecast of the zone and action with the mean probability confusion matrix (CM). Fig. 3 shows the CM heatmap for zone and action, respectively. In addition, the detailed zone accuracy was presented in Fig. 12. In both CM and on average, the correct assigned class had the highest probability and could be identified from the figures. This result suggests that the NMSTPP model was able to infer the zone and action of the next event.

11

<table>
  <thead>
    <tr>
        <th>Time</th>
        <th>Predicted t</th>
        <th>True t</th>
    </tr>
  </thead>
  <tbody>
    <tr>
        <td>0</td>
        <td>0.0</td>
        <td>0.0</td>
    </tr>
    <tr>
        <td>10</td>
        <td>0.9</td>
        <td>0.95</td>
    </tr>
    <tr>
        <td>20</td>
        <td>0.95</td>
        <td>0.98</td>
    </tr>
    <tr>
        <td>30</td>
        <td>0.98</td>
        <td>0.99</td>
    </tr>
    <tr>
        <td>40</td>
        <td>0.99</td>
        <td>1.0</td>
    </tr>
    <tr>
        <td>50</td>
        <td>1.0</td>
        <td>1.0</td>
    </tr>
    <tr>
        <td>60</td>
        <td>1.0</td>
        <td>1.0</td>
    </tr>
  </tbody>
</table>

**Fig. 2.** CDF of the predicted interevent time and the true interevent time (left) and self-attention heatmap (right).

<table>
  <thead>
    <tr>
        <th>Action</th>
        <th>p</th>
        <th>_</th>
        <th>d</th>
        <th>x</th>
        <th>s</th>
    </tr>
  </thead>
  <tbody>
    <tr>
        <td>p</td>
        <td>0.31</td>
        <td>0.16</td>
        <td>0.27</td>
        <td>0.12</td>
        <td>0.14</td>
    </tr>
    <tr>
        <td>_</td>
        <td>0.06</td>
        <td>0.36</td>
        <td>0.21</td>
        <td>0.16</td>
        <td>0.22</td>
    </tr>
    <tr>
        <td>d</td>
        <td>0.20</td>
        <td>0.18</td>
        <td>0.26</td>
        <td>0.16</td>
        <td>0.19</td>
    </tr>
    <tr>
        <td>x</td>
        <td>0.02</td>
        <td>0.07</td>
        <td>0.07</td>
        <td>0.48</td>
        <td>0.37</td>
    </tr>
    <tr>
        <td>s</td>
        <td>0.02</td>
        <td>0.08</td>
        <td>0.03</td>
        <td>0.22</td>
        <td>0.65</td>
    </tr>
  </tbody>
</table>

**Fig. 3.** Zone (left) and action (right) confusion matrix heatmap (mean probability).

## 3.5 HPUS verification and application to premier league

Upon verifying the NMSTPP model, we verified the HPUS and demonstrated the application of HPUS to 2017-2018 premier league season. In validating the HPUS, we first calculated the average HPUS and HPUS+ for each team in the premier league. Afterward, we calculated the correlation between the average HPUS, HPUS+, xG, goal, and the final ranking. Table 8 shows the value of the metrics and Fig. 4 (left) shows the correlation matrix heatmap for the metrics. From the correlation matrix, the average goal (-0.84), xG (-0.81), HPUS (-0.78), and HPUS+ (-0.74) had significant negative correlation to the final ranking of the team, implying that the four metrics could reflect the final outcome in a season and could be applied to compare different teams' performances. Nevertheless, the HPUS and HPUS+ had slightly less ($\le 0.07$) significant correlation than goal and xG. However, in the NMSTPP model, HPUS or HPUS+, the goal data (directly related to the match outcome) had never been used. Therefore, the slightly less significant correlation was reasonable. In addition, the HPUS (0.92, 0.92) and HPUS+ (0.91, 0.90) had significant correlation with goal and xG, thereby implying that the proposed metrics were able to reflect the attacking performances of the teams. Summarily, the HPUS metrics were capable of

evaluating all types of major events in football, and were able to reflect a team's final ranking and attacking performance.

Subsequently, the applications of the HPUS metrics are described. As an initial step, we analyzed teams' possession by plotting the HPUS densities. In Fig. 4 (right), three teams (final ranking) are considered: Manchester City (1), Chelsea (5), and Newcastle United (10). As Fig. 4 (right) shows, the team with a higher ranking was able to utilize the possession and generated more high HPUS possession and less low HPUS possession.

<table>
  <thead>
    <tr>
        <th> </th>
        <th>Ranking</th>
        <th>Goal</th>
        <th>xG</th>
        <th>HPUS</th>
        <th>HPUS+</th>
    </tr>
  </thead>
  <tbody>
    <tr>
        <td>Ranking</td>
        <td>1.00</td>
        <td>-0.84</td>
        <td>-0.81</td>
        <td>-0.78</td>
        <td>-0.77</td>
    </tr>
    <tr>
        <td>Goal</td>
        <td>-0.84</td>
        <td>1.00</td>
        <td>0.97</td>
        <td>0.92</td>
        <td>0.91</td>
    </tr>
    <tr>
        <td>xG</td>
        <td>-0.81</td>
        <td>0.97</td>
        <td>1.00</td>
        <td>0.92</td>
        <td>0.90</td>
    </tr>
    <tr>
        <td>HPUS</td>
        <td>-0.78</td>
        <td>0.92</td>
        <td>0.92</td>
        <td>1.00</td>
        <td>0.96</td>
    </tr>
    <tr>
        <td>HPUS+</td>
        <td>-0.77</td>
        <td>0.91</td>
        <td>0.90</td>
        <td>0.96</td>
        <td>1.00</td>
    </tr>
  </tbody>
</table>
<table>
  <thead>
    <tr>
        <th>HPUS</th>
        <th>Manchester City (1)</th>
        <th>Chelsea (5)</th>
        <th>Newcastle United (10)</th>
    </tr>
  </thead>
  <tbody>
    <tr>
        <td>0</td>
        <td>0.00</td>
        <td>0.00</td>
        <td>0.00</td>
    </tr>
    <tr>
        <td>5</td>
        <td>0.07</td>
        <td>0.06</td>
        <td>0.14</td>
    </tr>
    <tr>
        <td>10</td>
        <td>0.07</td>
        <td>0.05</td>
        <td>0.03</td>
    </tr>
    <tr>
        <td>15</td>
        <td>0.04</td>
        <td>0.01</td>
        <td>0.01</td>
    </tr>
    <tr>
        <td>20</td>
        <td>0.00</td>
        <td>0.00</td>
        <td>0.00</td>
    </tr>
  </tbody>
</table>

**Fig. 4.** 2017-2018 season premier league team statistics correlation matrix heatmap (left) and teams' HPUS density for possession in matches over 2017-2018 premier league season (right).

Lastly, we analyzed the change in HPUS and HPUS+ in a match. In Fig. 5, two matches are selected, Manchester City vs Newcastle United (Time: 2018, Jan 21, Result: 3:1) and Chelsea vs Newcastle United (Time: 2017, Dec 2, Result: 3:1). Primarily, the change in HPUS (left) and HPUS+ (right) provided different information. The former quantified the potential attack opportunities a team had created, while the latter quantified how many of those opportunities were converted to attack. In the Manchester City vs Newcastle (top) match, although Newcastle United was able to create opportunities, but was unable to convert them into attacks.

Secondarily, both changes in HPUS and HPUS+ provided a good indication of the team's performance. Although both matches ended in 3:1 against Newcastle United, the match against Chelsea (Bottom), shows that Newcastle United created more opportunities and converted more opportunities into an attack. Therefore, we concluded that Newcastle United performed better against Chelsea than against Manchester City. In conclusion, HPUS and HPUS+ provided in-depth information on teams' performance. Furthermore, the analysis based on HPUS was still feasible even if important events like goals and shots were absent.

# 4 Related work

There are many types of sequential data in sports (football, basketball, and rugby union), match results, event data of the ball and the player, and so on. To model

<table>
  <thead>
    <tr>
        <th>Time (min)</th>
        <th>Manchester City</th>
        <th>Newcastle United</th>
    </tr>
  </thead>
  <tbody>
    <tr>
        <td>0</td>
        <td>0</td>
        <td>0</td>
    </tr>
    <tr>
        <td>5</td>
        <td>25</td>
        <td>10</td>
    </tr>
    <tr>
        <td>10</td>
        <td>68</td>
        <td>25</td>
    </tr>
    <tr>
        <td>15</td>
        <td>55</td>
        <td>15</td>
    </tr>
    <tr>
        <td>20</td>
        <td>58</td>
        <td>10</td>
    </tr>
    <tr>
        <td>25</td>
        <td>78</td>
        <td>10</td>
    </tr>
    <tr>
        <td>30</td>
        <td>55</td>
        <td>5</td>
    </tr>
    <tr>
        <td>35</td>
        <td>68</td>
        <td>15</td>
    </tr>
    <tr>
        <td>40</td>
        <td>68</td>
        <td>5</td>
    </tr>
    <tr>
        <td>45</td>
        <td>55</td>
        <td>25</td>
    </tr>
    <tr>
        <td>60</td>
        <td>0</td>
        <td>0</td>
    </tr>
    <tr>
        <td>65</td>
        <td>68</td>
        <td>5</td>
    </tr>
    <tr>
        <td>70</td>
        <td>55</td>
        <td>5</td>
    </tr>
    <tr>
        <td>75</td>
        <td>55</td>
        <td>10</td>
    </tr>
    <tr>
        <td>80</td>
        <td>75</td>
        <td>10</td>
    </tr>
    <tr>
        <td>85</td>
        <td>45</td>
        <td>25</td>
    </tr>
    <tr>
        <td>90</td>
        <td>68</td>
        <td>5</td>
    </tr>
    <tr>
        <td>95</td>
        <td>55</td>
        <td>15</td>
    </tr>
    <tr>
        <td>100</td>
        <td>35</td>
        <td>10</td>
    </tr>
    <tr>
        <td>105</td>
        <td>20</td>
        <td>25</td>
    </tr>
  </tbody>
</table>
<table>
  <thead>
    <tr>
        <th>Time (min)</th>
        <th>Manchester City</th>
        <th>Newcastle United</th>
    </tr>
  </thead>
  <tbody>
    <tr>
        <td>0</td>
        <td>0</td>
        <td>0</td>
    </tr>
    <tr>
        <td>5</td>
        <td>0</td>
        <td>10</td>
    </tr>
    <tr>
        <td>10</td>
        <td>32</td>
        <td>0</td>
    </tr>
    <tr>
        <td>15</td>
        <td>25</td>
        <td>0</td>
    </tr>
    <tr>
        <td>20</td>
        <td>18</td>
        <td>0</td>
    </tr>
    <tr>
        <td>25</td>
        <td>35</td>
        <td>0</td>
    </tr>
    <tr>
        <td>30</td>
        <td>42</td>
        <td>0</td>
    </tr>
    <tr>
        <td>35</td>
        <td>48</td>
        <td>0</td>
    </tr>
    <tr>
        <td>40</td>
        <td>28</td>
        <td>0</td>
    </tr>
    <tr>
        <td>45</td>
        <td>38</td>
        <td>0</td>
    </tr>
    <tr>
        <td>60</td>
        <td>0</td>
        <td>0</td>
    </tr>
    <tr>
        <td>65</td>
        <td>14</td>
        <td>0</td>
    </tr>
    <tr>
        <td>70</td>
        <td>12</td>
        <td>0</td>
    </tr>
    <tr>
        <td>75</td>
        <td>18</td>
        <td>0</td>
    </tr>
    <tr>
        <td>80</td>
        <td>20</td>
        <td>12</td>
    </tr>
    <tr>
        <td>85</td>
        <td>35</td>
        <td>10</td>
    </tr>
    <tr>
        <td>90</td>
        <td>32</td>
        <td>5</td>
    </tr>
    <tr>
        <td>95</td>
        <td>28</td>
        <td>0</td>
    </tr>
    <tr>
        <td>100</td>
        <td>18</td>
        <td>0</td>
    </tr>
    <tr>
        <td>105</td>
        <td>0</td>
        <td>10</td>
    </tr>
  </tbody>
</table>
<table>
  <thead>
    <tr>
        <th>Time (min)</th>
        <th>Chelsea</th>
        <th>Newcastle United</th>
    </tr>
  </thead>
  <tbody>
    <tr>
        <td>0</td>
        <td>0</td>
        <td>0</td>
    </tr>
    <tr>
        <td>5</td>
        <td>25</td>
        <td>10</td>
    </tr>
    <tr>
        <td>10</td>
        <td>42</td>
        <td>30</td>
    </tr>
    <tr>
        <td>15</td>
        <td>32</td>
        <td>32</td>
    </tr>
    <tr>
        <td>20</td>
        <td>42</td>
        <td>15</td>
    </tr>
    <tr>
        <td>25</td>
        <td>32</td>
        <td>42</td>
    </tr>
    <tr>
        <td>30</td>
        <td>42</td>
        <td>25</td>
    </tr>
    <tr>
        <td>35</td>
        <td>35</td>
        <td>10</td>
    </tr>
    <tr>
        <td>40</td>
        <td>45</td>
        <td>5</td>
    </tr>
    <tr>
        <td>45</td>
        <td>25</td>
        <td>25</td>
    </tr>
    <tr>
        <td>60</td>
        <td>0</td>
        <td>0</td>
    </tr>
    <tr>
        <td>65</td>
        <td>42</td>
        <td>18</td>
    </tr>
    <tr>
        <td>70</td>
        <td>45</td>
        <td>18</td>
    </tr>
    <tr>
        <td>75</td>
        <td>42</td>
        <td>25</td>
    </tr>
    <tr>
        <td>80</td>
        <td>42</td>
        <td>30</td>
    </tr>
    <tr>
        <td>85</td>
        <td>32</td>
        <td>25</td>
    </tr>
    <tr>
        <td>90</td>
        <td>45</td>
        <td>18</td>
    </tr>
    <tr>
        <td>95</td>
        <td>42</td>
        <td>15</td>
    </tr>
    <tr>
        <td>100</td>
        <td>35</td>
        <td>32</td>
    </tr>
    <tr>
        <td>105</td>
        <td>25</td>
        <td>15</td>
    </tr>
  </tbody>
</table>
<table>
  <thead>
    <tr>
        <th>Time (min)</th>
        <th>Chelsea</th>
        <th>Newcastle United</th>
    </tr>
  </thead>
  <tbody>
    <tr>
        <td>0</td>
        <td>0</td>
        <td>0</td>
    </tr>
    <tr>
        <td>5</td>
        <td>0</td>
        <td>5</td>
    </tr>
    <tr>
        <td>10</td>
        <td>12</td>
        <td>0</td>
    </tr>
    <tr>
        <td>15</td>
        <td>11</td>
        <td>0</td>
    </tr>
    <tr>
        <td>20</td>
        <td>28</td>
        <td>0</td>
    </tr>
    <tr>
        <td>25</td>
        <td>10</td>
        <td>0</td>
    </tr>
    <tr>
        <td>30</td>
        <td>28</td>
        <td>0</td>
    </tr>
    <tr>
        <td>35</td>
        <td>18</td>
        <td>0</td>
    </tr>
    <tr>
        <td>40</td>
        <td>22</td>
        <td>0</td>
    </tr>
    <tr>
        <td>45</td>
        <td>5</td>
        <td>5</td>
    </tr>
    <tr>
        <td>60</td>
        <td>0</td>
        <td>0</td>
    </tr>
    <tr>
        <td>65</td>
        <td>16</td>
        <td>0</td>
    </tr>
    <tr>
        <td>70</td>
        <td>12</td>
        <td>0</td>
    </tr>
    <tr>
        <td>75</td>
        <td>18</td>
        <td>0</td>
    </tr>
    <tr>
        <td>80</td>
        <td>12</td>
        <td>12</td>
    </tr>
    <tr>
        <td>85</td>
        <td>15</td>
        <td>10</td>
    </tr>
    <tr>
        <td>90</td>
        <td>22</td>
        <td>5</td>
    </tr>
    <tr>
        <td>95</td>
        <td>0</td>
        <td>0</td>
    </tr>
    <tr>
        <td>100</td>
        <td>5</td>
        <td>0</td>
    </tr>
    <tr>
        <td>105</td>
        <td>0</td>
        <td>12</td>
    </tr>
  </tbody>
</table>

**Fig. 5.** Matches cumulative HPUS (left) and HPUS+ (right) values change per 5 minutes in regular time. (Top) Manchester City vs Newcastle United (Time: 2018, Jan 21, Result: 3:1); (Bottom) Chelsea vs Newcastle United (Time: 2017, Dec 2, Result: 3:1). The first half is in 0-45 minutes and the second half is in 60-105 minutes; dotted line implies a goal scored in the 5 minutes period.

sequential event data in sports, ML techniques, and point process techniques are the most common techniques applied by researchers.

In the proposed ML models, recurrent neural networks (RNN) and self-attention are the most popular key components. For RNN, GRU [3] and LSTM [10] have been applied to model the possession termination action in rugby union [24], next event location and action type in football [25], as well as the outcome of a sequence of play in basketball [32]. However, in long sequence data, the gradient calculation for models with RNN components are usually complex and thus leading to a long training time. Meanwhile, in recent times, the self-attention mechanism in natural language processing has been found to model long sequential data more efficiently. Therefore, the self-attention mechanism has been applied to replace the RNN component [34,36]. For self-attention, the transformer encoder [31] that includes self-attention mechanism has been applied to model the next event location and action type in football [25]. In addition, the combination of self-attention and LSTM has been applied to model match result in football [35].

In the proposed point process models, the player's shooting location in basketball can be defined as a Log-Gaussian Cox process [18] [17]. Moreover, football event data can be defined as a marked spatial-temporal point process as in equation 3. In [19], the interevent time, zone, and action types are defined as gamma

distribution, transition probability, and Hawkes process [11] based model, respectively. The Hawkes process based model for action types is based on history and the predicted interevent time, demonstrating how the important component of an event can be modeled dependently.

Summarily, in modeling event data, most sports sequential ML models only consider the partial component (location, action, or outcome type) of the next event and model the forecast independently. While point processes are able to model all components of event data, the combination of ML and point process (e.g., NTPP models [6,33,34,36]) are found to be more effective than the point process model. Therefore, to provide a more comprehensive analysis of football event data, we proposed modeling the football event data based on the NTPP framework.

Subsequent to modeling the football event data, performance metrics based on the model result could provide a clear indication on the performance and summarize the data. The most famous performance metric, expected goal (xG) was first purposed in hockey [16], and later applied to football [7]. In [7], xG is equivalent to the probability that a goal-scoring opportunity is converted into a goal. The xG is modeled directly from the spatial, player, and tactical features with a random forest model. Despite the popularity of xG, it is inapplicable without the existence of a goal-scoring opportunity, and from Table 7, goal-scoring opportunities (shots) are rare events in football matches. Since then, there have been multiple metrics proposed to resolve the limitation. For instance, the probability an off-ball player will score in the next action known as an off-ball scoring opportunity (OBSO) [26] (the variant is [27]), the probability that a pass is converted into an assist known as an expected assist (xA) https://www.statsperform.com/opta-analytics/, and score opportunities a player can create via passing or shooting known as an expected threat (xT) https://karun.in/blog/expected-threat.html.

Nevertheless, most metrics solely focused on inferencing the following event or outcome with only one previous event. Meanwhile, the metric valuing actions by estimating probabilities (VAEP) [5] showcases success in using three previous events to model the probability of scoring and conceding (the variants are [28,29]). Moreover, the possession utilization (poss-util) [25] using sequence of historical events to forecast the attacking probability of the next event has also found success in possession performance analysis. Yet, as mentioned previously, a football event is composed of three important components: time, location, and action type. Hence, based on poss-util, we have proposed a more holistic possession performance metrics HPUS with the proposed NMSTPP model.

# 5 Conclusion

In this study, we have proposed the NMSTPP model to model the time, location, and action types of football match events more effectively, and the HPUS metric, a more comprehensive performance metric for team possessions analysis. Our result suggested that the NMSTPP model is more effective than the baseline models, and that the model architecture is optimized under the proposed framework. Moreover, the HPUS was able to reflect the team's final ranking, average

goal scored, and average xG, in a season. In the future, since we have reduced the training set and validation set to consist only of matches from Bundesliga for computation efficiency, further improvement in the model's performance is expected when training the model with more data. Last but not least, the HPUS metric is only one of the many metrics that could possibly be derived based on the NMSTPP model. Conclusively, the NMSTPP model could be applied to develop more performance metrics, and hence, other sports with sequential events consisting of multiple important components can also benefit from this model.

## Acknowledgments

The authors would like to thank Mr. Ian Simpson for the fruitful discussions about football event data modeling. This work was financially supported by JST SPRING, Grant Number JPMJSP2125. The author Calvin C. K. Yeung would like to take this opportunity to thank the "Interdisciplinary Frontier Next-Generation Researcher Program of the Tokai Higher Education and Research System."

## References

1. Ba, J.L., Kiros, J.R., Hinton, G.E.: Layer normalization. arXiv preprint arXiv:1607.06450 (2016)

2. Berrar, D., Lopes, P., Davis, J., Dubitzky, W.: Guest editorial: special issue on machine learning for soccer. Machine Learning **108**(1), 1–7 (2019)

3. Chung, J., Gulcehre, C., Cho, K., Bengio, Y.: Empirical evaluation of gated recurrent neural networks on sequence modeling. arXiv preprint arXiv:1412.3555 (2014)

4. Cox, D.R.: Partial likelihood. Biometrika. **62**(2), 269-276 (1975)

5. Decroos, T., Bransen, L., Van Haaren, J., Davis, J.: Actions speak louder than goals: Valuing player actions in soccer. In: Proceedings of the 25th ACM SIGKDD international conference on knowledge discovery & data mining. pp. 1851–1861 (2019)

6. Du, N., Dai, H., Trivedi, R., Upadhyay, U., Gomez-Rodriguez, M., Song, L.: Recurrent marked temporal point processes: Embedding event history to vector. In: Proceedings of the 22nd ACM SIGKDD international conference on knowledge discovery and data mining. pp. 1555–1564 (2016)

7. Eggels, H., van Elk, R., Pechenizkiy, M.: Expected goals in soccer: Explaining match results using predictive analytics. In: The machine learning and data mining for sports analytics workshop. vol. 16 (2016)

8. Ertekin, Ş., Rudin, C., McCormick, T.H.: Reactive point processes: A new approach to predicting power failures in underground electrical systems. The Annals of Applied Statistics **9**(1), 122–144 (2015)

9. Fernandez, J., Bornn, L.: Wide open spaces: A statistical technique for measuring space creation in professional soccer. In: Sloan sports analytics conference. vol. 2018 (2018)

10. Graves, A.: Long short-term memory. Supervised sequence labelling with recurrent neural networks pp. 37–45 (2012)

11. Hawkes, A.G.: Spectra of some self-exciting and mutually exciting point processes. Biometrika **58**(1), 83–90 (1971)

12. He, K., Zhang, X., Ren, S., Sun, J.: Deep residual learning for image recognition. In: Proceedings of the IEEE conference on computer vision and pattern recognition. pp. 770–778 (2016)

13. Kingma, D.P., Ba, J.: Adam: A method for stochastic optimization. arXiv preprint arXiv:1412.6980 (2014)

14. Kingman, J. Poisson processes. Clarendon Press (1992)

15. Li, H.: Analysis on the construction of sports match prediction model using neural network. Soft Computing **24**(11), 8343–8353 (2020)

16. Macdonald, B.: An expected goals model for evaluating NHL teams and players. In: Proceedings of the 2012 MIT Sloan Sports Analytics Conference (2012)

17. Miller, A., Bornn, L., Adams, R., Goldsberry, K.: Factorized point process intensities: A spatial analysis of professional basketball. In: International conference on machine learning. pp. 235–243. PMLR (2014)

18. Møller, J., Syversveen, A.R., Waagepetersen, R.P.: Log gaussian cox processes. Scandinavian journal of statistics **25**(3), 451–482 (1998)

19. Narayanan, S.: Bayesian modelling of flexible marked point processes with applications to event sequences from association football. Ph.D. thesis, University of Warwick (2020)

20. Pappalardo, L., Cintia, P., Rossi, A., Massucco, E., Ferragina, P., Pedreschi, D., Giannotti, F.: A public dataset of spatio-temporal match events in soccer competitions. Scientific data **6**(1), 1–15 (2019)

21. Queiroz Gongora, L.: Estimating football position from context (2021)

22. Rasmussen, J.G.: Lecture notes: Temporal point processes and the conditional intensity function. arXiv preprint arXiv:1806.00221 (2018)

23. Shchur, O., Türkmen, A.C., Januschowski, T., Günnemann, S.: Neural temporal point processes: A review. arXiv preprint arXiv:2104.03528 (2021)

24. Sicilia, A., Pelechrinis, K., Goldsberry, K.: Deephoops: Evaluating micro-actions in basketball using deep feature representations of spatio-temporal data. In: Proceedings of the 25th ACM SIGKDD International Conference on Knowledge Discovery & Data Mining. pp. 2096–2104 (2019)

25. Simpson, I., Beal, R.J., Locke, D., Norman, T.J.: Seq2event: Learning the language of soccer using transformer-based match event prediction. In: Proceedings of the 28th ACM SIGKDD Conference on Knowledge Discovery and Data Mining. pp. 3898–3908 (2022)

26. Spearman, W.: Beyond expected goals. In: Proceedings of the 12th MIT sloan sports analytics conference. pp. 1–17 (2018)

27. Teranishi, M., Tsutsui, K., Takeda, K., Fujii, K.: Evaluation of creating scoring opportunities for teammates in soccer via trajectory prediction. In: International Workshop on Machine Learning and Data Mining for Sports Analytics. Springer (2022)

28. Toda, K., Teranishi, M., Kushiro, K., Fujii, K.: Evaluation of soccer team defense based on prediction models of ball recovery and being attacked. PLoS One **17**(1), e0263051 (2022)

29. Umemoto, R., Tsutsui, K., Fujii, K. Location analysis of players in UEFA EURO 2020 and 2022 using generalized valuation of defense by estimating probabilities. arXiv preprint arXiv:2212.00021 (2022)

30. Van Haaren, J.: "why would i trust your numbers?" on the explainability of expected values in soccer. arXiv preprint arXiv:2105.13778 (2021)

31. Vaswani, A., Shazeer, N., Parmar, N., Uszkoreit, J., Jones, L., Gomez, A.N., Kaiser, Ł., Polosukhin, I.: Attention is all you need. Advances in neural information processing systems **30** (2017)

32. Watson, N., Hendricks, S., Stewart, T., Durbach, I.: Integrating machine learning and decision support in tactical decision-making in rugby union. Journal of the operational research society **72**(10), 2274–2285 (2021)

33. Xiao, S., Yan, J., Yang, X., Zha, H., Chu, S.: Modeling the intensity function of point process via recurrent neural networks. In: Proceedings of the AAAI Conference on Artificial Intelligence. vol. 31 (2017)

34. Zhang, Q., Lipani, A., Kirnap, O., Yilmaz, E.: Self-attentive hawkes process. In: International conference on machine learning. pp. 11183–11193. PMLR (2020)

35. Zhang, Q., Zhang, X., Hu, H., Li, C., Lin, Y., Ma, R.: Sports match prediction model for training and exercise using attention-based lstm network. Digital Communications and Networks **8**(4), 508–515 (2022)

36. Zuo, S., Jiang, H., Li, Z., Zhao, T., Zha, H.: Transformer hawkes process. In: International conference on machine learning. pp. 11692–11702. PMLR (2020)

# Appendix

In the appendix, we provide more information on model reproductions, descriptions of features, the model component, and results.

## A Dataset preprocessing

Primarily for the dataset, we drop all matches with own-goal, since it is rare and hard to classify into any group of action, but has a significant impact on the match results. Next, we split the dataset for train/valid/test according to the 0.8/0.1/0.1 ratio for matches in each football league. However, training the models on the entire dataset requires a significant amount of time (more than 20 hours). Therefore, in order to verify more model architectures and applied grid searching, we have reduced the training set and validation set to 100,000 (5% of the training set) and 10,000 rows of record respectively. In general, Table 4 has summarized the number of matches in the train/valid/test set.

Table 4 shows how the dataset is split according to football leagues.

Table 4. Dataset splitting method.

<table>
  <thead>
    <tr>
        <th>Football league</th>
        <th>Training (Matches)</th>
        <th>Validation (Matches)</th>
        <th>Testing (Matches)</th>
    </tr>
  </thead>
  <tbody>
    <tr>
        <td>Premier League</td>
        <td>-</td>
        <td>-</td>
        <td>37</td>
    </tr>
    <tr>
        <td>La Liga</td>
        <td>-</td>
        <td>-</td>
        <td>37</td>
    </tr>
    <tr>
        <td>Ligue 1</td>
        <td>-</td>
        <td>-</td>
        <td>37</td>
    </tr>
    <tr>
        <td>Serie A</td>
        <td>-</td>
        <td>-</td>
        <td>37</td>
    </tr>
    <tr>
        <td>and Bundesliga</td>
        <td>73</td>
        <td>7</td>
        <td>30</td>
    </tr>
  </tbody>
</table>

Furthermore, Fig 6 presents how the events record rows being sliced into a time window as input features and target features. We take 40 events recorded in time $i - 40$ to $i - 1$ to forecast the event in time $i$ using the NMSTPP model. In addition, the slicing only happens within the events of a match but not across matches and disregards which team the possession belongs to.

```mermaid
graph TD
    subgraph Match1 [Match 1]
        M1_1["X₀ | Y₀"]
        M1_2["X₁ | Y₁"]
        M1_3["X₂ | Y₂"]
        M1_dots["..."]
        M1_1 --> M1_2 --> M1_3 --> M1_dots
    end

    Dots1["..."]

    subgraph MatchJ [Match j]
        MJ_dots1["..."]
        MJ_1["Xᵢ₋₃ | Yᵢ₋₃"]
        MJ_2["Xᵢ₋₂ | Yᵢ₋₂"]
        MJ_3["Xᵢ₋₁ | Yᵢ₋₁"]
        MJ_dots1 --> MJ_1 --> MJ_2 --> MJ_3
    end

    subgraph MatchJ1 [Match j+1]
        MJ1_1["Xᵢ | Yᵢ"]
        MJ1_2["Xᵢ₊₁ | Yᵢ₊₁"]
        MJ1_3["Xᵢ₊₂ | Yᵢ₊₂"]
        MJ1_dots["..."]
        MJ1_1 --> MJ1_2 --> MJ1_3 --> MJ1_dots
    end

    Dots2["..."]

    Match1 --- Dots1 --- MatchJ
    MatchJ --- MatchJ1 --- Dots2
```

Fig. 6. The time window slicing method for input features and target features.

# B Hyperparameter grid search

Primarily, Table 5 summarize all the hyperparameter value or option being grid searched and the best hyperparameter for the NMSTPP model is bolded.

**Table 5.** Grid searched Hyperparameter and its value (option). The best value (option) for each Hyperparameter is bolded.

<table>
  <thead>
    <tr>
        <th>Hyperparameter</th>
        <th>Grid searched value (option)</th>
    </tr>
  </thead>
  <tbody>
    <tr>
        <td>Seqlen</td>
        <td>1,10,<strong>40</strong>,100</td>
    </tr>
    <tr>
        <td>dim_feedforward</td>
        <td>1,2,4,8,16,32,64,128,256,512,<strong>1024</strong>,2048,4096,8192,16384</td>
    </tr>
    <tr>
        <td>order</td>
        <td><strong>{t, z, m}</strong>,{t, m, z},...,{z, t, m}</td>
    </tr>
    <tr>
        <td>num_layer_t</td>
        <td><strong>1</strong>,2,4,8,16</td>
    </tr>
    <tr>
        <td>num_layer_z</td>
        <td><strong>1</strong>,2,4,8,16</td>
    </tr>
    <tr>
        <td>num_layer_m</td>
        <td>1,<strong>2</strong>,4,8,16</td>
    </tr>
    <tr>
        <td>activation_function</td>
        <td><strong>None</strong>, ReLu, Sigmoid, Tanh</td>
    </tr>
    <tr>
        <td>drop_out</td>
        <td><strong>0</strong>,0.1,0.2,0.5</td>
    </tr>
  </tbody>
</table>

For the hyperparameter order, we compared the order of interevent time $t_i$, zones $z_i$, and actions $m_i$ in equation 5. Table 6 compares the performance of NMSTPP model when we interchange the order. The result shows that following the order interevent time $t_i$, zones $z_i$ as in equation 5 provides the best result. Moreover, the order mainly affects the CEL of action and is able to create a difference up to 0.11.

**Table 6.** Performance comparisons with different connection orders NMSTPP models on the validation set. Model total loss, RMSE on interevent time $t$, CEL on zone, and CEL on action are reported.

<table>
  <thead>
    <tr>
        <th>Order (first/second/third)</th>
        <th>Total loss</th>
        <th>RMSE<sub>t</sub></th>
        <th>CEL<sub>zone</sub></th>
        <th>CEL<sub>action</sub></th>
    </tr>
  </thead>
  <tbody>
    <tr>
        <td>zone/t/action</td>
        <td>4.58</td>
        <td>0.10</td>
        <td>2.06</td>
        <td>1.44</td>
    </tr>
    <tr>
        <td>action/zone/t</td>
        <td>4.57</td>
        <td>0.11</td>
        <td>2.06</td>
        <td>1.38</td>
    </tr>
    <tr>
        <td>zone/action/t</td>
        <td>4.49</td>
        <td>0.10</td>
        <td>2.06</td>
        <td>1.39</td>
    </tr>
    <tr>
        <td>t/action/zone</td>
        <td>4.46</td>
        <td>0.10</td>
        <td>2.06</td>
        <td>1.36</td>
    </tr>
    <tr>
        <td>action/t/zone</td>
        <td>4.43</td>
        <td>0.10</td>
        <td>2.06</td>
        <td>1.34</td>
    </tr>
    <tr>
        <td>t/zone/action</td>
        <td><strong>4.40</strong></td>
        <td>0.10</td>
        <td>2.04</td>
        <td>1.33</td>
    </tr>
  </tbody>
</table>

Furthermore in model validation, to deal with imbalance classes in the zone and action, The CELs are weighted. The weight was calculated with the `compute_class_weight` function from the python scikit-learn package. The calculation follows the following equation:

$$ \text{weight of class i} = \frac{\text{number of sample}}{\text{number of class} \times \text{number of sample in class i}} \quad (13) $$

In addition, for better forecast result validation, we have multiplied the weight for the action dribble by 1.16. This method increases the accuracy of the dribble forecast while decreasing the accuracy in other action classes.

# C Features description and summary

Firstly, Fig 7 and 8 gives a more detailed description of event location represented in (x,y) coordinate and in the zone according to Juego de posición (position game) respectively.

Pitch coordinates diagram showing (x,y) coordinate points on a football pitch

**Fig. 7.** WyScout pitch (x,y) coordinate. The goal on the left side belongs to the team in possession and the goal on the right side belongs to the opponent, figure retrieved from [https://apidocs.wyscout.com/#section/Data-glossary-and-definitions/Pitch-coordinates](https://apidocs.wyscout.com/#section/Data-glossary-and-definitions/Pitch-coordinates).

Secondly, Table 7 summarize how the action type defined by WyScout are being grouped into the 5 action group used in this study.

Thirdly, Fig 9 shows the heatmap of each grouped action, and the pitch is zoned according to the Juego de posición (position game).

Lastly, a more detailed description of other continuous features is given below and the calculation of the features is given in the code. Using the center point of the zone to represent the location of the zone. We created the following features:

- *zone_s* : distance from the previous zone to the current zone.

- *zone_deltay* : change in the zone y coordinate.

- *zone_deltax* : change in the zone x coordinate.

- *zone_sg* : distance from the opposition goal center point to the zone.

- *zone_thetag* : angles from the opposition goal center point to the zone.

Pitch zoning according to Juego de posición (position game)

**Fig. 8.** Pitch zoning according to Juego de posición (position game). The goal on the left side belongs to the team in possession and the goal on the right side belongs to the opponent. More details of Juego de posición can be found in [https://spielverlagerung.com/2014/11/26/juego-de-posicion-a-short-explanation/](https://spielverlagerung.com/2014/11/26/juego-de-posicion-a-short-explanation/)

**Table 7.** WyScout action type and subtype grouping [25].

<table>
  <thead>
    <tr>
        <th>Action type (subtype)</th>
        <th>Grouped action type (proportion)</th>
    </tr>
  </thead>
  <tbody>
    <tr>
        <td>Pass (Hand pass)</td>
        <td rowspan="9">Pass $p$ (66.99%)</td>
    </tr>
    <tr>
        <td>Pass (Head pass)</td>
    </tr>
    <tr>
        <td>Pass (High pass)</td>
    </tr>
    <tr>
        <td>Pass (Launch)</td>
    </tr>
    <tr>
        <td>Pass (Simple pass)</td>
    </tr>
    <tr>
        <td>Pass (Smart pass)</td>
    </tr>
    <tr>
        <td>Others on the ball (Clearance) &amp; Free Kick (Goal kick)</td>
    </tr>
    <tr>
        <td>Free Kick (Throw in)</td>
    </tr>
    <tr>
        <td>Free Kick (Free Kick)</td>
    </tr>
    <tr>
        <td>Duel (Ground attacking duel)</td>
        <td rowspan="3">Dribble $d$ (8.48%)</td>
    </tr>
    <tr>
        <td>Others on the ball (Acceleration)</td>
    </tr>
    <tr>
        <td>Others on the ball (Touch)</td>
    </tr>
    <tr>
        <td>Pass (Cross)</td>
        <td rowspan="3">Cross $x$ (3.27%)</td>
    </tr>
    <tr>
        <td>Free Kick (Corner)</td>
    </tr>
    <tr>
        <td>Free Kick (Free kick cross)</td>
    </tr>
    <tr>
        <td>Shot (Shot)</td>
        <td rowspan="3">Shot $s$ (1.68%)</td>
    </tr>
    <tr>
        <td>Free Kick (Free kick shot)</td>
    </tr>
    <tr>
        <td>Free Kick (Penalty)</td>
    </tr>
    <tr>
        <td>After all action in a possession</td>
        <td>Possession end _ (19.58%)</td>
    </tr>
    <tr>
        <td>Other</td>
        <td>N/A</td>
    </tr>
  </tbody>
</table>

# D Transformer encoder [31]

The architecture of the transformer encoder is shown in Fig 10. The main component in the transformer encoder is the multi-head attention which composes

<table>
  <thead>
    <tr>
        <th colspan="6">Action: Pass</th>
    </tr>
  </thead>
  <tbody>
    <tr>
        <td>2</td>
        <td>5</td>
        <td>5</td>
        <td>6</td>
        <td>4</td>
        <td>1</td>
    </tr>
    <tr>
        <td rowspan="3">8</td>
        <td>8</td>
        <td>6</td>
        <td rowspan="3">1</td>
        <td colspan="2"></td>
    </tr>
    <tr>
        <td>11</td>
        <td>7</td>
        <td colspan="2"></td>
    </tr>
    <tr>
        <td>8</td>
        <td>6</td>
        <td colspan="2"></td>
    </tr>
    <tr>
        <td>1</td>
        <td>4</td>
        <td>5</td>
        <td>6</td>
        <td>4</td>
        <td>2</td>
    </tr>
    <tr>
        <th colspan="6">Action: Dribble</th>
    </tr>
    <tr>
        <td>1</td>
        <td>4</td>
        <td>5</td>
        <td>6</td>
        <td>7</td>
        <td>5</td>
    </tr>
    <tr>
        <td rowspan="3">3</td>
        <td>6</td>
        <td>6</td>
        <td rowspan="3">7</td>
        <td colspan="2"></td>
    </tr>
    <tr>
        <td>7</td>
        <td>8</td>
        <td colspan="2"></td>
    </tr>
    <tr>
        <td>5</td>
        <td>6</td>
        <td colspan="2"></td>
    </tr>
    <tr>
        <td>1</td>
        <td>3</td>
        <td>4</td>
        <td>5</td>
        <td>6</td>
        <td>5</td>
    </tr>
    <tr>
        <th colspan="6">Action: Cross</th>
    </tr>
    <tr>
        <td>0</td>
        <td>0</td>
        <td>0</td>
        <td>1</td>
        <td>10</td>
        <td>28</td>
    </tr>
    <tr>
        <td rowspan="3">0</td>
        <td>0</td>
        <td>2</td>
        <td rowspan="3">12</td>
        <td colspan="2"></td>
    </tr>
    <tr>
        <td>0</td>
        <td>1</td>
        <td colspan="2"></td>
    </tr>
    <tr>
        <td>0</td>
        <td>3</td>
        <td colspan="2"></td>
    </tr>
    <tr>
        <td>0</td>
        <td>0</td>
        <td>0</td>
        <td>1</td>
        <td>11</td>
        <td>31</td>
    </tr>
    <tr>
        <th colspan="6">Action: Shot</th>
    </tr>
    <tr>
        <td>0</td>
        <td>0</td>
        <td>0</td>
        <td>0</td>
        <td>1</td>
        <td>0</td>
    </tr>
    <tr>
        <td rowspan="3">0</td>
        <td>0</td>
        <td>9</td>
        <td rowspan="3">61</td>
        <td colspan="2"></td>
    </tr>
    <tr>
        <td>0</td>
        <td>21</td>
        <td colspan="2"></td>
    </tr>
    <tr>
        <td>0</td>
        <td>7</td>
        <td colspan="2"></td>
    </tr>
    <tr>
        <td>0</td>
        <td>0</td>
        <td>0</td>
        <td>0</td>
        <td>1</td>
        <td>0</td>
    </tr>
    <tr>
        <th colspan="6">Action: Possession End</th>
    </tr>
    <tr>
        <td>3</td>
        <td>4</td>
        <td>4</td>
        <td>4</td>
        <td>5</td>
        <td>4</td>
    </tr>
    <tr>
        <td rowspan="3">11</td>
        <td>5</td>
        <td>5</td>
        <td rowspan="3">8</td>
        <td colspan="2"></td>
    </tr>
    <tr>
        <td>6</td>
        <td>7</td>
        <td colspan="2"></td>
    </tr>
    <tr>
        <td>5</td>
        <td>5</td>
        <td colspan="2"></td>
    </tr>
    <tr>
        <td>2</td>
        <td>4</td>
        <td>4</td>
        <td>4</td>
        <td>5</td>
        <td>6</td>
    </tr>
  </tbody>
</table>

**Fig. 9.** Heatmap for Grouped action type. (Top left) heatmap for action pass, (Top right) heatmap for action dribble, (Middle left) heatmap for action cross, (Middle Right) heatmap for action shot, (Bottom) heatmap for possession end. The goal on the left side belongs to the team in possession and the goal on the right side belongs to the opponent.

```mermaid
graph TD
    Inputs --> InputEmbedding[Input Embedding]
    InputEmbedding --> Add1((+))
    PositionalEncoding((~)) -- Positional Encoding --> Add1
    Add1 --> SubBlock1
    
    subgraph SubBlock1 [Nx]
        direction TB
        MultiHeadAttention[Multi-Head Attention] --> Add2((+))
        Add2 --> FeedForward[Feed Forward]
        FeedForward --> Add3((+))
        
        subgraph AddNorm1 [Add & Norm]
            Add2
        end
        
        subgraph AddNorm2 [Add & Norm]
            Add3
        end
    end

    Add1 -.-> Add2
    Add2 -.-> Add3
```

Fig. 10. Transformer encoder [31].

of multiple self-attention heads. For one self-attention head as applied in this study, let $X$ be the input matrix with each row representing the features of an event. The matrix first passes through the positional encoding as the following equation.

$$X = (X + Z) \tag{14}$$

Assume $X \in \mathbb{R}^{N \times K}$, meaning $X$ consists of $N$ events, and each event consists of $K$ features. The entries in matrix $Z$ can be determined with the following equation. Where, $n \in 1, 2, ..., N$, $k \in 1, 2, ..., K$ and $d$ is a scalar, set to 10,000 in [31].

$$\begin{aligned} Z(n, 2k) &= \sin(\frac{n}{n^{2k/d}}) \\ Z(n, 2k + 1) &= \cos(\frac{n}{n^{2k/d}}) \end{aligned} \tag{15}$$

Afterward, the following equation shows the calculation of the self-attention head. Where $Q, K, V$ are the queries, keys, and values matrix respectively, $W^Q, W^K \in \mathbb{R}^{K \times d_k}$, and $W^V \in \mathbb{R}^{K \times d_v}$ are trainable parameters, and $d_k, d_v$ are hyperparameters.

$$\begin{aligned} Attention(Q, K, V) &= softmax(\frac{QK^T}{\sqrt{d_k}})V \\ Q = XW^Q, K &= XW^K, V = XW^V \end{aligned} \tag{16}$$

Lastly, after the output matrix from the self-attention head passes through add and norm layer [1, 12], feedforward layers, and a final add and norm layers. It results in an encoded matrix.

# E Neural temporal point process (NTPP) framework [23]

In general, the NTPP framework is a combination of ML and the ideas of the point process, allowing for flexible model architecture. To begin with, a marked temporal point process in time $[0, T]$ can be defined as $X = (m_i, t_i)_{i=1}^N$, where N is the total number of events, $m \in \{1, 2, \dots, K\}$ is the mark, and $0 < t_1 < \dots < t_i < \dots < t_N < T$ is the arrival time under the definition in [23]. In addition the history of at time $t$ is defined as $H_t = (m_i, t_i)_{i=1}^{t-1}$. Thus the conditional intensity function for type m can be defined as follows:

$$ \lambda_m(t|H_t) = \lim_{\Delta t \downarrow 0} \frac{P(\text{event of type m in } [t, t + \Delta t))}{\Delta t} \eqno(17) $$

```mermaid
graph LR
    subgraph Step1 ["1 Represent events (tj, mj) as feature vectors yj"]
        E1["(t1, m1)"] --> Y1["y1"]
        E2["(t2, m2)"] --> Y2["y2"]
        Ei["(ti-1, mi-1)"] --> Yi["yi-1"]
    end
    
    subgraph Step2 ["2 Encode history (y1, ..., yi-1) as a vector hi"]
        Y1 --> H["hi"]
        Y2 --> H
        Yi --> H
    end
    
    subgraph Step3 ["3 Obtain conditional distribution Pi(ti, mi|Hti) = P(ti, mi|hi)"]
        H --> Dist["Probability Distribution"]
    end
    
    TimeLine["0 --- (t1, m1) --- (t2, m2) --- ... --- (ti-1, mi-1) --- T --- time"]
```

Fig. 11. NTPP model construct method [23].

The construct of an NTPP model can be defined with 3 steps as in Fig 11 [23]. First, represent the event into a features vector. Second, encode the history into a history vector. Third, predict the next event.

For the first step, depending on different event data, the exact procedure might be different. But in general, applying embedding to class features and concatenating them with continuous features will result in a feature vector.

Next, in the second step, RNN, GRU [3], LSTM [10], and transformer encoder [31] are often used for history encoding. Nevertheless, transformer encoders are found to be more efficient in recent studies but further verification is needed [23].

Lastly, in the third step, there are many ways to define the distribution of the interevent time. For example, probability density function, cumulative distribution function, survival function, hazard function, and cumulative hazard function.

# F NMSTPP model validation

Fig 12 shows the accuracy for each zone forecasting, in another word, the value in the main diagonal of Fig 3 confusion matrix.

<table>
  <tbody>
    <tr>
        <td>59%</td>
        <td>40%</td>
        <td>37%</td>
        <td>34%</td>
        <td>45%</td>
        <td>40%</td>
    </tr>
    <tr>
        <td rowspan="2">34%</td>
        <td colspan="2">25%</td>
        <td colspan="2">30%</td>
        <td rowspan="2">55%</td>
    </tr>
    <tr>
        <td colspan="2">29%</td>
        <td colspan="2">32%</td>
    </tr>
    <tr>
        <td> </td>
        <td colspan="2">14%</td>
        <td colspan="2">28%</td>
        <td> </td>
    </tr>
    <tr>
        <td>67%</td>
        <td>37%</td>
        <td>22%</td>
        <td>39%</td>
        <td>44%</td>
        <td>51%</td>
    </tr>
  </tbody>
</table>

**Fig. 12.** NMSTPP model zone forecast accuracy.

# G HPUS details and validation

In this section, more details on the calculation of HPUS, the application, and validation are provided. For HPUS, Fig 13 demonstrated how the zones of the pitch are further converted into areas for the calculation of HPUS. Moreover, Fig 14 is the plot for the exponential decaying function. The 0.3 in the function is a hyperparameter, it was selected such that it gives significant weight to 5-6 events matching the average length of possession 5.2 (from the training set data).

<table>
  <tbody>
    <tr>
        <td>Area 0</td>
        <td>Area 1</td>
        <td>Area 2</td>
    </tr>
  </tbody>
</table>

**Fig. 13.** Pitch Area for HUPS. The goal on the left side belongs to the team in possession and the goal on the right side belongs to the opponent.

Moreover, Table 8 shows the premier league 2017-2018 team ranking, team name, average goal scored, average xG, average HPUS, average HPUS+, and the

<table>
  <thead>
    <tr>
        <th>x</th>
        <th>y</th>
    </tr>
  </thead>
  <tbody>
    <tr>
        <td>0</td>
        <td>1.5</td>
    </tr>
    <tr>
        <td>1</td>
        <td>1.1</td>
    </tr>
    <tr>
        <td>2</td>
        <td>0.8</td>
    </tr>
    <tr>
        <td>3</td>
        <td>0.6</td>
    </tr>
    <tr>
        <td>4</td>
        <td>0.4</td>
    </tr>
    <tr>
        <td>5</td>
        <td>0.3</td>
    </tr>
    <tr>
        <td>6</td>
        <td>0.2</td>
    </tr>
    <tr>
        <td>7</td>
        <td>0.1</td>
    </tr>
  </tbody>
</table>

Fig. 14. Exponentially decaying function for HUPS weights assignment.

HPUS ratio. The HPUS ratio is the ratio of average HPUS+ to average HPUS, it is surprising that each team has an HPUS ratio near 0.3. This emphasizes the importance of a high HPUS, creating possessions that are likely to be converted into scoring opportunities. In addition, Fig 15 shows teams' HPUS+ density for possession in matches over 2017-2018 premier league season.

**Table 8.** Preimer league 2017-2018 team ranking and match averaged performance statistics and metrics.

<table>
  <thead>
    <tr>
        <th>Ranking</th>
        <th>Team name</th>
        <th>Goal</th>
        <th>xG</th>
        <th>HPUS</th>
        <th>HPUS+</th>
        <th>HPUS ratio</th>
    </tr>
  </thead>
  <tbody>
    <tr>
        <td>1</td>
        <td>Manchester City</td>
        <td>2.79</td>
        <td>2.46</td>
        <td>626.86</td>
        <td>213.41</td>
        <td>0.34</td>
    </tr>
    <tr>
        <td>2</td>
        <td>Manchester United</td>
        <td>1.79</td>
        <td>1.63</td>
        <td>537.44</td>
        <td>174.03</td>
        <td>0.32</td>
    </tr>
    <tr>
        <td>3</td>
        <td>Tottenham Hotspur</td>
        <td>1.95</td>
        <td>1.87</td>
        <td>600.71</td>
        <td>192.40</td>
        <td>0.32</td>
    </tr>
    <tr>
        <td>4</td>
        <td>Liverpool</td>
        <td>2.21</td>
        <td>2.08</td>
        <td>586.66</td>
        <td>186.07</td>
        <td>0.32</td>
    </tr>
    <tr>
        <td>5</td>
        <td>Chelsea</td>
        <td>1.63</td>
        <td>1.55</td>
        <td>557.87</td>
        <td>187.25</td>
        <td>0.34</td>
    </tr>
    <tr>
        <td>6</td>
        <td>Arsenal</td>
        <td>1.95</td>
        <td>1.93</td>
        <td>603.23</td>
        <td>169.23</td>
        <td>0.28</td>
    </tr>
    <tr>
        <td>7</td>
        <td>Burnley</td>
        <td>0.95</td>
        <td>0.89</td>
        <td>412.62</td>
        <td>125.11</td>
        <td>0.30</td>
    </tr>
    <tr>
        <td>8</td>
        <td>Everton</td>
        <td>1.16</td>
        <td>1.18</td>
        <td>435.69</td>
        <td>117.64</td>
        <td>0.27</td>
    </tr>
    <tr>
        <td>9</td>
        <td>Leicester City</td>
        <td>1.47</td>
        <td>1.35</td>
        <td>461.40</td>
        <td>139.62</td>
        <td>0.30</td>
    </tr>
    <tr>
        <td>10</td>
        <td>Newcastle United</td>
        <td>1.03</td>
        <td>1.19</td>
        <td>423.97</td>
        <td>124.12</td>
        <td>0.29</td>
    </tr>
    <tr>
        <td>11</td>
        <td>Crystal Palace</td>
        <td>1.18</td>
        <td>1.53</td>
        <td>446.03</td>
        <td>136.75</td>
        <td>0.31</td>
    </tr>
    <tr>
        <td>12</td>
        <td>Bournemouth</td>
        <td>1.18</td>
        <td>1.06</td>
        <td>470.88</td>
        <td>130.71</td>
        <td>0.28</td>
    </tr>
    <tr>
        <td>13</td>
        <td>West Ham United</td>
        <td>1.26</td>
        <td>1.01</td>
        <td>438.44</td>
        <td>135.98</td>
        <td>0.31</td>
    </tr>
    <tr>
        <td>14</td>
        <td>Watford</td>
        <td>1.16</td>
        <td>1.23</td>
        <td>467.41</td>
        <td>139.15</td>
        <td>0.30</td>
    </tr>
    <tr>
        <td>15</td>
        <td>Brighton and Hove Albion</td>
        <td>0.89</td>
        <td>0.97</td>
        <td>418.84</td>
        <td>126.12</td>
        <td>0.30</td>
    </tr>
    <tr>
        <td>16</td>
        <td>Huddersfield Town</td>
        <td>0.74</td>
        <td>0.85</td>
        <td>437.80</td>
        <td>128.00</td>
        <td>0.29</td>
    </tr>
    <tr>
        <td>17</td>
        <td>Southampton</td>
        <td>0.97</td>
        <td>1.11</td>
        <td>486.45</td>
        <td>156.15</td>
        <td>0.32</td>
    </tr>
    <tr>
        <td>18</td>
        <td>Swansea City</td>
        <td>0.74</td>
        <td>0.80</td>
        <td>417.77</td>
        <td>120.04</td>
        <td>0.29</td>
    </tr>
    <tr>
        <td>19</td>
        <td>Stoke City</td>
        <td>0.92</td>
        <td>0.98</td>
        <td>399.63</td>
        <td>116.44</td>
        <td>0.29</td>
    </tr>
    <tr>
        <td>20</td>
        <td>West Bromwich Albion</td>
        <td>0.82</td>
        <td>0.93</td>
        <td>410.14</td>
        <td>119.54</td>
        <td>0.29</td>
    </tr>
  </tbody>
</table>

27

<table>
  <thead>
    <tr>
        <th>X-axis</th>
        <th>Manchester City (1)</th>
        <th>Chelsea (5)</th>
        <th>Newcastle United (10)</th>
    </tr>
  </thead>
  <tbody>
    <tr>
        <td>0.0</td>
        <td>0.038</td>
        <td>0.021</td>
        <td>0.035</td>
    </tr>
    <tr>
        <td>2.5</td>
        <td>0.055</td>
        <td>0.042</td>
        <td>0.058</td>
    </tr>
    <tr>
        <td>5.0</td>
        <td>0.051</td>
        <td>0.065</td>
        <td>0.075</td>
    </tr>
    <tr>
        <td>7.5</td>
        <td>0.065</td>
        <td>0.085</td>
        <td>0.098</td>
    </tr>
    <tr>
        <td>10.0</td>
        <td>0.091</td>
        <td>0.097</td>
        <td>0.085</td>
    </tr>
    <tr>
        <td>12.5</td>
        <td>0.065</td>
        <td>0.055</td>
        <td>0.030</td>
    </tr>
    <tr>
        <td>15.0</td>
        <td>0.025</td>
        <td>0.015</td>
        <td>0.005</td>
    </tr>
    <tr>
        <td>17.5</td>
        <td>0.005</td>
        <td>0.005</td>
        <td>0.001</td>
    </tr>
    <tr>
        <td>20.0</td>
        <td>0.001</td>
        <td>0.001</td>
        <td>0.000</td>
    </tr>
  </tbody>
</table>

**Fig. 15.** Team HPUS+ density for possession in matches over 2017-2018 premier league season.