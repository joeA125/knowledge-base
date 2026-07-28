# Evaluation of creating scoring opportunities for teammates in soccer via trajectory prediction

Masakiyo Teranishi<sup>1</sup>, Kazushi Tsutsui<sup>1</sup>, Kazuya Takeda<sup>1</sup>, and Keisuke Fujii<sup>1,2,3</sup>

<sup>1</sup> Graduate School of Informatics, Nagoya University, Nagoya, Japan.

<sup>2</sup> Center for Advanced Intelligence Project, RIKEN, Fukuoka, Japan.

<sup>3</sup> PRESTO, Japan Science and Technology Agency, Saitama, Japan.

fujii@i.nagoya-u.ac.jp

**Abstract.** Evaluating the individual movements for teammates in soccer players is crucial for assessing teamwork, scouting, and fan engagement. It has been said that players in a 90-min game do not have the ball for about 87 minutes on average. However, it has remained difficult to evaluate an attacking player without receiving the ball, and to reveal how movement contributes to the creation of scoring opportunities for teammates. In this paper, we evaluate players who create off-ball scoring opportunities by comparing actual movements with the reference movements generated via trajectory prediction. First, we predict the trajectories of players using a graph variational recurrent neural network that can accurately model the relationship between players and predict the long-term trajectory. Next, based on the difference in the modified off-ball evaluation index between the actual and the predicted trajectory as a reference, we evaluate how the actual movement contributes to scoring opportunity compared to the predicted movement. For verification, we examined the relationship with the annual salary, the goals, and the rating in the game by experts for all games of a team in a professional soccer league in a year. The results show that the annual salary and the proposed indicator correlated significantly, which could not be explained by the existing indicators and goals. Our results suggest the effectiveness of the proposed method as an indicator for a player without the ball to create a scoring chance for teammates.

**Keywords:** multi-agent · deep learning · trajectory · sports · football

# 1 Introduction

Assessing the movements of individual players for teammates in team sports is an important aspect of building teamwork, assessment of players’ salaries, player recruitment, and scouting. In soccer, most analytics has focused on the outcomes of discrete events near the ball (on-ball) [34,12,39,40,3,10,33,32] whereas much of the importance in player movements exist in the events without the ball (off-ball). For example, it is said that players in a 90-min game do not have the ball for about 87 minutes on average [15]. However, continuous off-ball movements are usually not discretized and difficult to understand except for core fans, experienced players, and coaches. Also for the media and building fan engagement, quantitative evaluation of off-ball players is an issue in demand, which provides

a common reference for beginners and experts in the sport e.g., when arguing a play of a favorite player.

Regarding the off-ball player evaluation methods, the positioning itself related to the goal was evaluated from the location data of all players and the ball. For example, the method called off-ball scoring opportunity (OBSO) to evaluate the player who receives the ball [45] and the method to evaluate the movement to create space [15] have been proposed. However, it has been still difficult to clarify how movements contribute to the creation of scoring opportunities for teammates, to evaluate other attacking players who do not receive it (e.g., a player moving tactically for teammates), and often to evaluate a score prediction to reflect the position of the multiple defenders.

In this paper, we propose a new evaluation indicator, Creating Off-Ball Scoring Opportunity (C-OBSO in Fig. 1A), aiming for evaluating players who create scoring opportunities when the attacking player is without the ball. The overview of our method is shown in Fig. 6. (i) First, we modify the score model in the framework of OBSO [45] with the potential score model that reflects the positions of multiple defenders with a mixed Gaussian distribution (Fig. 1B). (ii) Next, we accurately model the relationship between athletes and perform long-term trajectory predictions (Fig. 1A) using the graph variational recurrent neural network (GVRNN) [51]. (iii) Finally, based on the difference in the modified off-ball evaluation index between the actual and the predicted trajectory (Fig. 1A), we evaluate how the actual movement contributes to scoring opportunity relative to the predicted movement as a reference.

In summary, our main contributions were as follows. (1) We proposed an evaluation method of how movements contributed to the creation of scoring opportunities compared to the predicted movements of off-ball players in team sports attacks. (2) As a score predictor, we proposed a potential score model that considers the positions of multiple defenders in a mixed Gaussian distribution. (3) In the experiment, we analyzed the relationship between the annual salary, the goals, and the game rating by experts, and show the effectiveness of the proposed method as an indicator for an off-ball player to create scoring opportunities for teammates. Our approach can evaluate continuous movements of players by comparing with the reference (here predicted) movements, which are difficult to be discretized or labeled but crucial for teamwork, scouting, and fan engagement. The structure of this paper is as follows. First, we overview the related works in Section 4 and present experimental results in Section 3. Next, we describe our methods in Section 2 and conclude this paper in Section 5.

# 2 Proposed framework

Here, we propose C-OBSO based on the motivation to evaluate players who create off-ball scoring opportunities for teammates. To this end, in Section 2.1, we first propose a potential score model that reflects the positions of multiple defenders with a mixed Gaussian distribution. Next, in Section 2.2, we predict multi-agent trajectory using GVRNN [51] and evaluate the difference between the actual value of the modified OBSO and the predicted value (as a reference) to evaluate how the movement contributed to the creation of scoring opportunities.

Diagram showing C-OBSO example and potential score model. Part A illustrates player trajectories and scoring chance creation. Part B shows shot block distribution and score probability relative to goal position.

**Fig. 1.** Our C-OBSO example and potential score model. (A) Example of C-OBSO computation. A1 is the player to be finally evaluated, A2 is the shooting player, D1 and D2 are the defender of A1 and A2, respectively. $V_1$ is the C-OBSO value of A1, $V_{OBSO}^2$ is the actual OBSO, and $V_{OBSO}^{\prime 2}$ is the reference OBSO value of A2 using the predicted trajectory. (B) Potential score model. Left: shot-blocking distribution formed by defenders. Right: shot probability corresponding to each shot vector. The vertical axis is the goal position (m), and the horizontal axis is the shot probability corresponding to each shot vector.

## 2.1 Potential score model in modified OBSO

First, we describe the base model of our evaluation method called OBSO [45] and then propose the potential score model. OBSO evaluates off-ball players by computing the following joint probability

$$P(G|D) = \sum_{r \in R \times R} P(S_r \cap C_r \cap T_r | D)$$ (1)

$$= \sum_r P(S_r | C_r, T_r, D) P(C_r | T_r, D) P(T_r | D),$$ (2)

where $D$ is the instantaneous state of the game (e.g., player positions and velocities). The details in OBSO are given in Appendix B. $P(S_r)$ is the probability of scoring from an arbitrary point $r \in R \times R$ on the pitch, assuming the next on-ball event occurs there. $P(C_r)$ is the probability that the passing team will control a ball at point $r$. $P(T_r)$ is the probability that the next on-ball event occurs at point $r$. Here, for simplicity, we can assume that $P(S_r|D), P(T_r|D), P(C_r|D)$ are independent if the parameter $\alpha = 0$ in the original work implementation (Eq. (6) in [45]). Then, the joint probability can be decomposed into a series of conditional probabilities as follows:

$$P(G|D) = \sum_{r \in R \times R} P(S_r|D) P(C_r|D) P(T_r|D).$$ (3)

$P(C_r|D)$ is the probability that the attacking team will control the ball at point $r$ assuming the next on-ball event occurs there, which is called the potential pitch control field (PPCF). $P(T_r|D)$ is defined as a two-dimensional Gaussian distribution with the current ball coordinates as the mean. $P(S_r|D)$ is simply calculated as a value that decreases with the distance from the goal. We used the grid data and computed $P(C_r|D)$ and $P(T_r|D)$ based on the code at [https://github.com/Friends-of-Tracking-Data-FoTD/LaurieOnTracking](https://github.com/Friends-of-Tracking-Data-FoTD/LaurieOnTracking).

In the original OBSO [45], the scoring probability was calculated as the output $P(S_r|D)$ of the score model as a function of the distance from the goal.

However, the scoring probability may depend on the angle to the goal and the defensive position of the opponent. Therefore, in this paper, we propose a score model that reflects the angle to the goal and the position of multiple defenders. Here, we consider the shot-blocking distribution of the defenders who can block shots in the field, and propose a potential model where the scoring probability decreases when defenders exist. The basic idea shown in Fig. 1B is to calculate the scoring probability from the angle to the goal at which the shot tends to be scored, considering the mixed distribution of the positions of multiple defenders. The proposed scoring probability $P(S_{r}^{p}|D)$ at a certain point $r$ is calculated as the sum of the shot value $V_{shot}$ as follows:

$$P(S_{r}^{p}|D) = \sum_{i=1}^{n} V_{shot}(\vec{s}_{i}), \eqno(4)$$

$$V_{shot}(\vec{s}) = C(c - V_{block}), \eqno(5)$$

where $n$ is determined by the angle from the shooting position to the goal, and $\vec{s}$ is a shot vector per degree ($n$ is larger when the shot from the center and smaller from the side). The shot value $V_{shot}$ is calculated by subtracting the shot block value $V_{block}$ from a certain constant $c$ ($c, C$ are parameters determined from data to be adjusted so that $V_{shot} \geq 0$ and $P(S_{r}^{p}) \in [0, 1]$). Let $V_{block}$ be the sum of the shot block distribution values along the shot vector $\vec{s}$. The shot blocking distribution is the sum of the normal distributions (variance $\sigma^{2} = 0.5 + l_{d}$) assigned to each defender on the goal side of the shooting position (shot blockable players using legs), where $l_{d}$ is the distance between the shooting position and the defender. We consider that goalkeepers have a shot blocking distribution with twice the value of normal defenders because of a higher shot-blocking ability. Here, we assume that the block distribution is not changed with the distance from the ball. A defender near to the ball may affect the ball, but far players use the flight time of the ball for their movement. This formulation is left for future work.

## 2.2 C-OBSO with trajectory prediction

Here, we describe the base model of our trajectory prediction method called GVRNN [51] and then describe our C-OBSO framework. Our contribution here is to evaluate how the actual "off-ball" movement contributes to scoring opportunity compared to the predicted movement (or trajectory) as a reference. In our method, we use GVRNN [51], which is a VRNN [9] combined with a graph neural network (GNN [26]). For the details in VRNN and GVRNN, see Appendices C and D. In GVRNN, the graph encoder-decoder network models the relationship between players as a graph, which is one of the best performing models for predicting player trajectories in team sports [51]. This is a probabilistic model which can sample multiple possible trajectories.

Based on the trajectory prediction, we propose an evaluation index C-OBSO of players who create scoring opportunities for teammates. The basic idea is to evaluate an off-ball player from the difference in the modified OBSO values

5

between the predicted and actual movements of the players. The C-OBSO value of a player $i$ without the ball can be expressed as follows.

$$V_i = V_{OBSO}^k - V_{OBSO}^{\prime k} , \tag{6}$$

where the player $k$ is the ball carrier who performs a final action (e.g., shot), $V_{OBSO}^k$ is the modified OBSO in the actual game situation, and $V_{OBSO}^{\prime k}$ is the modified OBSO based on the predicted trajectory as a reference. For example, in Fig. 1A, C-OBSO is positive and the player to be evaluated (A1) contributes more to the shooter (A2) than the referenced (predicted) player. Specifically, A1 has created a more advantageous situation for A2 by attracting D1 more than expected. C-OBSO can evaluate a player in such situations with an interpretable value (i.e., the increase in scoring probability). Theoretically, if perfectly predicted, C-OBSO is always zero, but actually, if we apply this to a test data, the perfect prediction is impossible. In other words, we assume the imperfect trajectory prediction in this framework.

# 3 Experiments

In this section, we validate the proposed method of the potential score model, the trajectory prediction model (GVRNN), and the C-OBSO itself. For our implementation, the code is available at [https://github.com/keisuke198619/C-OBSO](https://github.com/keisuke198619/C-OBSO).

## 3.1 Dataset

In this study, we used all 34 games data of Yokohama F Marinos in the Meiji J1 League 2019 season to perform specific player-level evaluations in limited data. Note that the tracking data for all players and timesteps were not publicly shared in such amounts. The dataset includes event data (i.e., labels of actions, e.g., passing and shooting, recorded at 30 Hz and the simultaneous xy coordinates of the ball) and tracking data (i.e., xy coordinates of all players recorded at 25 Hz) provided by Data Stadium Inc. The company was licensed to acquire this data and sell it to third parties, and it was guaranteed that the use of the data would not infringe on any rights of the players or teams. For annual salaries, we used the salaries of the same team (Yokohama) in 2019 [44] because they were different valuation criteria for different teams and the transfer of the players took place during the season. The goals for each player in each match were collected from [23]. The rating by experts in each match [43] was also used for verification, which was scored in 0.5 point increments with a maximum of 10 points.

## 3.2 Data processing for verification

We used the attacking data of Yokohama F Marinos for the test and those of the opponent teams for training the model or parameter fitting. Again, since the data was limited in this study, we split the data in such a way, and if we have more data, we can analyze all teams with the training data with the same team. Here we describe the processing of the potential score model, the trajectory prediction model, the C-OBSO, and their statistical analyses.

**Potential score model.** To validate the potential score model, the opponent’s shots (345 shots, 34 goals) were used for fitting the parameters $c$ and $C$, and Yokohama F Marinos’ shots (494 shots, 59 goals) were used for verification. The parameters $c, C$ of the potential score model were determined to be $c = 1.1, C = 1/150$ using the data of the opponents. The potential score model was verified by the root mean square error (RMSE) between the actual score and the calculated scoring probability. We compared the RMSE with that of a simple score model as a function of distance from the goal for implementing the original OBSO [45] (see also Section 2.1). Although there have been more holistic score models such as [16,1], to fairly compare with our potential model as a component of the modified OBSO, we consider the simple score model as an appropriate baseline.

**Trajectory prediction model.** For the test data of trajectory prediction and C-OBSO, we used 412 shot scenes of Yokohama F Marinos (we selected the sequences of consecutive events and excluded too short events such as a free kick). The trajectory prediction model was trained using the opponents’ data to generate “league average” trajectories. The tracking data were down-sampled to 10 Hz (after prediction, up-sampled at the original 25 Hz) based on [19]. To verify the accuracy of the long-term trajectory prediction, we set various time lengths (6, 8, 10, and 12 s) using mean trajectories in 10 samples. We divided into the opponent data for batch training (6 s: 94208 sequences, 8 s: 49152 sequences, 10 s: 33536 sequences, 12 s: 24320 sequences) and the validation (6 s: 10477 sequences, 8 s: 5479 sequences, 10 s: 3730 sequences, 12 s: 2721 sequences). Note that the end of all sequences was the moment of a shot. The input feature has 92 dimensions (the xy coordinates and the velocity of 22 players and the ball). During training, the model was trained based on the one-step prediction error of all combinations of the two attackers who invaded the attacking third. We simultaneously predicted the three players: one of the off-ball attackers and the defenders closest to each attacker. Note that we only consider the three players’ interactions and ignore others’ interactions, because the prediction error will increase if the numbers increase, and the increase of the predicted players is left for future work.

For the test data of the 412 sequences, the three relevant players and the attacker from the same criterion were predicted. At the inference, using 2 s sequences as burn-in period, we predicted the sequences for the subsequent time lengths (i.e., 4, 6, 8, and 10 s) by updating the estimated position and velocity (i.e., performed long-term prediction). For the training of the proposed and baseline models, we used the Adam optimizer [24] with a learning rate of 0.001 and 10 training epochs. We set the batchsize to 256. For the performance metrics, we used the endpoint error (mean absolute error: MAE) from the actual trajectory.

**C-OBSO.** To compute C-OBSO, predicted trajectories with 4 s (total 6 s) were used. This is because a longer prediction time will result in a larger prediction error, while a shorter prediction time will not make a difference in the evaluation of C-OBSO. Although the negative values of C-OBSO are also calculated by comparison with the reference, the negative values were calculated as 0, assuming that they may not have a negative effect on the behavioral players. This is

because there were many situations with negative values in which the shooter’s defender did not take an appropriate defensive position in the predicted trajectory.

**Statistical analysis.** For the verification of C-OBSO, we examined the relationship with the annual salary, the goals, and the expert’s rating. Note that there is no ground truth available for the verification. We also compared them with the existing OBSO [45]. Since some of the data often did not follow normal distributions, we used Spearman’s rank correlation coefficient $\rho$ for these relationships. Regarding the RMSE in the potential score model and MAE in the trajectory prediction, for the same reason, we used nonparametric statistical tests to compare with the baselines. Regarding the potential score model, we used the Wilcoxon rank sum test. For all statistical calculations, $p < 0.05$ was considered as significant.

## 3.3 Our model verification

First, we validated the potential score model needed to calculate the C-OBSO. The RMSE with the actual scores was $0.324 \pm 0.014$ for the conventional score model [45] without considering the defenders and goal angles, and $0.309 \pm 0.0014$ for the potential score model ($p < 10^{-10}$). This result suggests that the proposed method models the scores more accurately.

Figure 2 shows an example of the two methods in two actual situations where a shot is attempted from a similar distance. In the existing method, the probabilities were the same (both 0.1237) because the shots were taken from almost the same distance. The proposed method had a lower scoring probability with more defenders (upper: 0.0489, lower: 0.1202). We indicate that the proposed method reflects the position of multiple defenders and can model the score accurately.

Next, we show the results of the trajectory prediction model for computing C-OBSO. Endpoint errors (MAE and standard error, [m]) in GVRNN were $0.608 \pm 0.014$, $0.867 \pm 0.022$, $1.701 \pm 0.045$, $1.606 \pm 0.042$ in 4, 6, 8, 10 s prediction. In GVRNN, longer predictions show larger prediction errors except for the difference between 8 s and 10 s. Since the 4 s prediction of GVRNN achieved a low the MAE of less than 0.7 m, the GVRNN trajectory prediction of 4 s was used in the next C-OBSO. For details, see also Appendix E.

## 3.4 C-OBSO results

Verification of C-OBSO is challenging because of no ground truth values or player ratings. Therefore, we analyzed the relationship with the annual salary, the goals, and the game rating by experts, whereas we admit that these variables include various confounding factors. The relationships between the average C-OBSO and OBSO values of each player of Yokohama F Marinos in 2019 and the annual salary of each player in 2019 are shown in Fig. 3 (note that the tracking data for all players and timesteps were not publicly shared). Here we analyzed 15 players with more than 10 sequences under evaluation. As a result, there was a significant positive correlation between annual salary and C-OBSO ($\rho = 0.45, p = 0.046$). In addition, the two players with the higher evaluation values but lower salaries (in red in Fig. 3A) were highly evaluated players, who won the individual awards

Comparison between the score model of conventional and the proposed potential model, showing two scenarios A and B with attack, defense, and ball positions on a soccer field.

**Fig. 2.** Comparison between the score model of conventional and the proposed potential model. The scoring probability of our model is lower (A) when the defenders are crowded than (B), whereas that of the conventional score model in (A) was the same as (B).

(the most valuable player and valuable player award). In fact, their annual salary for the following year (2020) was also increased (valuable player: increased from 11 million yen to 40 million yen; the most valuable player: increased from 20 million yen to 60 million yen).

We found that these tendencies were similar to the C-OBSO and OBSO without the potential score model (see Appendix F). On the other hand, there was no significant correlation ($\rho = -0.28, p = 0.154$) for OBSO, which evaluates a player's own scoring opportunities (Fig. 3B). We also examined the relationship between annual salary and goals (Fig. 3C), and found no significant correlation ($\rho = -0.23, p = 0.208$). Therefore, there was no relationship between annual salary and goals. There were many players with zero goals, and it is difficult to evaluate them only with the goals.

Next, in order to examine the relationship with player performance in more detail, we show the relationship between C-OBSO and the rating by experts of the top three scorers (Nakagawa with 15 goals, Marcos with 15 goals, and Edigar with 11 goals in this season) in Fig. 4. We analyzed the games in which there were two or more C-OBSO evaluations using the average of C-OBSO values on each game (17 games for Nakagawa, 14 games for Marcos, and 10 games for Edigar). A strong positive correlation was found only for Nakagawa ($\rho = 0.75, p = 0.0003$) but not for Marcos ($\rho = 0.27, p = 0.174$) and Edigar ($\rho = -0.37, p = 0.145$).

<table>
    <tr>
        <th>Player</th>
        <th>Annual salary (million yen)</th>
        <th>C-OBSO</th>
    </tr>
    <tr>
        <td>Lee</td>
        <td>40</td>
        <td>0.0058</td>
    </tr>
    <tr>
        <td>Hatanaka</td>
        <td>12</td>
        <td>0.0052</td>
    </tr>
    <tr>
        <td>Nakagawa</td>
        <td>20</td>
        <td>0.0045</td>
    </tr>
    <tr>
        <td>Ohgihara</td>
        <td>35</td>
        <td>0.0043</td>
    </tr>
    <tr>
        <td>Edigar</td>
        <td>25</td>
        <td>0.0038</td>
    </tr>
    <tr>
        <td>Amano</td>
        <td>40</td>
        <td>0.0038</td>
    </tr>
    <tr>
        <td>Otsu</td>
        <td>35</td>
        <td>0.0035</td>
    </tr>
    <tr>
        <td>Marcos</td>
        <td>30</td>
        <td>0.0032</td>
    </tr>
    <tr>
        <td>Matsubara</td>
        <td>35</td>
        <td>0.0032</td>
    </tr>
    <tr>
        <td>Thiago</td>
        <td>30</td>
        <td>0.0028</td>
    </tr>
    <tr>
        <td>Endo</td>
        <td>18</td>
        <td>0.0023</td>
    </tr>
    <tr>
        <td>Theerathon</td>
        <td>25</td>
        <td>0.0022</td>
    </tr>
    <tr>
        <td>Miyoshi</td>
        <td>10</td>
        <td>0.0015</td>
    </tr>
    <tr>
        <td>Hirose</td>
        <td>8</td>
        <td>0.0012</td>
    </tr>
    <tr>
        <td>Kida</td>
        <td>20</td>
        <td>0.0012</td>
    </tr>
</table>
<table>
    <tr>
        <th>Player</th>
        <th>Annual salary (million yen)</th>
        <th>OBSO</th>
    </tr>
    <tr>
        <td>Miyoshi</td>
        <td>10</td>
        <td>0.081</td>
    </tr>
    <tr>
        <td>Edigar</td>
        <td>25</td>
        <td>0.075</td>
    </tr>
    <tr>
        <td>Nakagawa</td>
        <td>20</td>
        <td>0.068</td>
    </tr>
    <tr>
        <td>Marcos</td>
        <td>30</td>
        <td>0.066</td>
    </tr>
    <tr>
        <td>Lee</td>
        <td>40</td>
        <td>0.066</td>
    </tr>
    <tr>
        <td>Hatanaka</td>
        <td>12</td>
        <td>0.062</td>
    </tr>
    <tr>
        <td>Thiago</td>
        <td>30</td>
        <td>0.062</td>
    </tr>
    <tr>
        <td>Matsubara</td>
        <td>35</td>
        <td>0.062</td>
    </tr>
    <tr>
        <td>Endo</td>
        <td>18</td>
        <td>0.056</td>
    </tr>
    <tr>
        <td>Ohgihara</td>
        <td>35</td>
        <td>0.055</td>
    </tr>
    <tr>
        <td>Hirose</td>
        <td>8</td>
        <td>0.053</td>
    </tr>
    <tr>
        <td>Kida</td>
        <td>20</td>
        <td>0.052</td>
    </tr>
    <tr>
        <td>Amano</td>
        <td>40</td>
        <td>0.052</td>
    </tr>
    <tr>
        <td>Otsu</td>
        <td>35</td>
        <td>0.048</td>
    </tr>
    <tr>
        <td>Theerathon</td>
        <td>25</td>
        <td>0.042</td>
    </tr>
</table>
<table>
    <tr>
        <th>Player</th>
        <th>Goals in 2019 season</th>
        <th>Annual salary (million yen)</th>
    </tr>
    <tr>
        <td>Amano</td>
        <td>1</td>
        <td>40</td>
    </tr>
    <tr>
        <td>Lee</td>
        <td>1</td>
        <td>40</td>
    </tr>
    <tr>
        <td>Otsu</td>
        <td>1</td>
        <td>35</td>
    </tr>
    <tr>
        <td>Matsubara</td>
        <td>1</td>
        <td>35</td>
    </tr>
    <tr>
        <td>Ohgihara</td>
        <td>2</td>
        <td>35</td>
    </tr>
    <tr>
        <td>Theerathon</td>
        <td>3</td>
        <td>30</td>
    </tr>
    <tr>
        <td>Edigar</td>
        <td>11</td>
        <td>30</td>
    </tr>
    <tr>
        <td>Marcos</td>
        <td>15</td>
        <td>30</td>
    </tr>
    <tr>
        <td>Thiago</td>
        <td>1</td>
        <td>25</td>
    </tr>
    <tr>
        <td>Kida</td>
        <td>0</td>
        <td>20</td>
    </tr>
    <tr>
        <td>Nakagawa</td>
        <td>15</td>
        <td>20</td>
    </tr>
    <tr>
        <td>Hatanaka</td>
        <td>0</td>
        <td>12</td>
    </tr>
    <tr>
        <td>Miyoshi</td>
        <td>3</td>
        <td>10</td>
    </tr>
    <tr>
        <td>Endo</td>
        <td>7</td>
        <td>15</td>
    </tr>
    <tr>
        <td>Hirose</td>
        <td>1</td>
        <td>8</td>
    </tr>
</table>

**Fig. 3.** Relationship between indicators, goal, and annual salary in a team. (A) Relationship between C-OBSO and the salary. (B) Relationship between OBSO [45] and the salary. (C) Relationship between each player’s goals and annual salary. Red players received individual awards (Hatanaka: valuable player Award, Nakagawa: the most valuable player award).

<table>
    <tr>
        <th>Rating (Nakagawa)</th>
        <th>C-OBSO</th>
    </tr>
    <tr>
        <td>5.0</td>
        <td>0.002</td>
    </tr>
    <tr>
        <td>5.5</td>
        <td>0.001</td>
    </tr>
    <tr>
        <td>5.5</td>
        <td>0.002</td>
    </tr>
    <tr>
        <td>6.0</td>
        <td>0.001</td>
    </tr>
    <tr>
        <td>6.0</td>
        <td>0.002</td>
    </tr>
    <tr>
        <td>6.5</td>
        <td>0.003</td>
    </tr>
    <tr>
        <td>6.5</td>
        <td>0.006</td>
    </tr>
    <tr>
        <td>6.5</td>
        <td>0.011</td>
    </tr>
    <tr>
        <td>7.0</td>
        <td>0.005</td>
    </tr>
    <tr>
        <td>7.5</td>
        <td>0.013</td>
    </tr>
</table>
<table>
    <tr>
        <th>Rating (Marcos)</th>
        <th>C-OBSO</th>
    </tr>
    <tr>
        <td>5.0</td>
        <td>0.001</td>
    </tr>
    <tr>
        <td>5.5</td>
        <td>0.001</td>
    </tr>
    <tr>
        <td>6.0</td>
        <td>0.001</td>
    </tr>
    <tr>
        <td>6.0</td>
        <td>0.003</td>
    </tr>
    <tr>
        <td>6.0</td>
        <td>0.007</td>
    </tr>
    <tr>
        <td>6.5</td>
        <td>0.001</td>
    </tr>
    <tr>
        <td>6.5</td>
        <td>0.008</td>
    </tr>
    <tr>
        <td>7.0</td>
        <td>0.003</td>
    </tr>
</table>
<table>
    <tr>
        <th>Rating (Edigar)</th>
        <th>C-OBSO</th>
    </tr>
    <tr>
        <td>5.0</td>
        <td>0.005</td>
    </tr>
    <tr>
        <td>5.5</td>
        <td>0.006</td>
    </tr>
    <tr>
        <td>5.5</td>
        <td>0.007</td>
    </tr>
    <tr>
        <td>6.5</td>
        <td>0.001</td>
    </tr>
    <tr>
        <td>6.5</td>
        <td>0.005</td>
    </tr>
    <tr>
        <td>6.5</td>
        <td>0.007</td>
    </tr>
</table>

**Fig. 4.** Relationship between C-OBSO and the rating by experts of the top three scorers (A: Nakagawa, B: Marcos, C: Edigar) for each game.

In Appendix G, we show the results of the other four players who played seven games or more and had two related scoring opportunities or more (for C-OBSO). Similarly, there were no significant correlations between them for all players ($\rho_s < 0.190, p_s > 0.05$). In addition to the number of scoring opportunities for teammates (17 times), the results found that Nakagawa would be subjectively and quantitatively an outstanding player.

For reference, we also show the relationship between the goals of the top three scorers and the ratings by experts in Fig. 5. We analyzed the games in which each player played (33 games for Nakagawa, 33 for Marcos, and 16 for Edigar). For each player, there were strong correlations between the goals and the rating (Nakagawa $\rho = 0.63, p = 4.33 \times 10^{-5}$, Marcos $\rho = 0.71, p = 1.98 \times 10^{-6}$, Edigar $\rho = 0.91, p = 4.40 \times 10^{-7}$). We found that the rating of each game depends on a rare event (i.e., goals). In Appendix G, we show the results of the other four players who scored two points or more. Similarly, there were significant correlations between them for all players ($\rho_s > 0.516, p_s < 0.018$). Recall that there was a stronger correlation between C-OBSO and Nakagawa’s rating than for the other two players. Nakagawa also had higher average ratings than the other players (6.26 for Nakagawa, 5.97 for Marcos, and 6.09 for Edigar), and he was the player who won the most valuable player award. The game rating by experts would depend on the goals, but it may also evaluate the creation

<table>
  <thead>
    <tr>
        <th colspan="4">Relationship between goals and rating for top three scorers</th>
    </tr>
    <tr>
        <th>Player</th>
        <th>Rating</th>
        <th>Goals</th>
        <th>Frequency (Visual Size)</th>
    </tr>
  </thead>
  <tbody>
    <tr>
        <td>Nakagawa</td>
        <td>5.2</td>
        <td>0</td>
        <td>Small</td>
    </tr>
    <tr>
        <td>Nakagawa</td>
        <td>5.8</td>
        <td>0</td>
        <td>Medium</td>
    </tr>
    <tr>
        <td>Nakagawa</td>
        <td>6.2</td>
        <td>0</td>
        <td>Small</td>
    </tr>
    <tr>
        <td>Nakagawa</td>
        <td>6.5</td>
        <td>0</td>
        <td>Small</td>
    </tr>
    <tr>
        <td>Nakagawa</td>
        <td>6.8</td>
        <td>0</td>
        <td>Small</td>
    </tr>
    <tr>
        <td>Nakagawa</td>
        <td>6.2</td>
        <td>1</td>
        <td>Large</td>
    </tr>
    <tr>
        <td>Nakagawa</td>
        <td>6.5</td>
        <td>1</td>
        <td>Large</td>
    </tr>
    <tr>
        <td>Nakagawa</td>
        <td>6.8</td>
        <td>1</td>
        <td>Large</td>
    </tr>
    <tr>
        <td>Nakagawa</td>
        <td>7.2</td>
        <td>1</td>
        <td>Large</td>
    </tr>
    <tr>
        <td>Marcos</td>
        <td>4.5</td>
        <td>0</td>
        <td>Small</td>
    </tr>
    <tr>
        <td>Marcos</td>
        <td>5.0</td>
        <td>0</td>
        <td>Small</td>
    </tr>
    <tr>
        <td>Marcos</td>
        <td>5.5</td>
        <td>0</td>
        <td>Large</td>
    </tr>
    <tr>
        <td>Marcos</td>
        <td>5.8</td>
        <td>0</td>
        <td>Large</td>
    </tr>
    <tr>
        <td>Marcos</td>
        <td>6.2</td>
        <td>0</td>
        <td>Medium</td>
    </tr>
    <tr>
        <td>Marcos</td>
        <td>6.5</td>
        <td>0</td>
        <td>Small</td>
    </tr>
    <tr>
        <td>Marcos</td>
        <td>5.5</td>
        <td>1</td>
        <td>Small</td>
    </tr>
    <tr>
        <td>Marcos</td>
        <td>6.0</td>
        <td>1</td>
        <td>Small</td>
    </tr>
    <tr>
        <td>Marcos</td>
        <td>6.5</td>
        <td>1</td>
        <td>Large</td>
    </tr>
    <tr>
        <td>Marcos</td>
        <td>6.8</td>
        <td>1</td>
        <td>Medium</td>
    </tr>
    <tr>
        <td>Marcos</td>
        <td>7.0</td>
        <td>2</td>
        <td>Medium</td>
    </tr>
    <tr>
        <td>Marcos</td>
        <td>7.5</td>
        <td>2</td>
        <td>Small</td>
    </tr>
    <tr>
        <td>Edigar</td>
        <td>5.2</td>
        <td>0</td>
        <td>Large</td>
    </tr>
    <tr>
        <td>Edigar</td>
        <td>5.5</td>
        <td>0</td>
        <td>Large</td>
    </tr>
    <tr>
        <td>Edigar</td>
        <td>5.8</td>
        <td>0</td>
        <td>Small</td>
    </tr>
    <tr>
        <td>Edigar</td>
        <td>6.2</td>
        <td>1</td>
        <td>Large</td>
    </tr>
    <tr>
        <td>Edigar</td>
        <td>6.5</td>
        <td>1</td>
        <td>Large</td>
    </tr>
    <tr>
        <td>Edigar</td>
        <td>6.8</td>
        <td>1</td>
        <td>Small</td>
    </tr>
    <tr>
        <td>Edigar</td>
        <td>7.0</td>
        <td>2</td>
        <td>Large</td>
    </tr>
  </tbody>
</table>

**Fig. 5.** Relationship between the goals and the rating by experts of the top three scorers (A: Nakagawa, B: Marcos, C: Edigar) for each game. The size of the circle represents the frequency because there are many combinations of the goals and the rating with the same value.

of scoring opportunities only for Nakagawa. From these results, we speculate that Nakagawa was highly evaluated not only for his scoring but also for his contribution to other attacking players. Our method can also evaluate players difficult to be evaluated by conventional indicators, which is crucial for assessing teamwork and player salary, player recruitment, and scouting.

## 4 Related work

In the tactical behaviors of team sports, agents select an action that follows a policy (or strategy) in a state, receives a reward from the environment and others, and updates the state [18]. This is similar to a reinforcement learning framework (e.g., [?]). Due to the difficulty in modeling the entire framework from data for various reasons [49] (e.g., a sparse reward and difficulty in estimating intents), we can adopt two approaches: to estimate the related variables and functions from data (i.e., inverse approach) as a sub-problem, and to build a model (e.g., reinforcement learning model) to generate data in virtual space (i.e., forward approach, e.g., [27, 41]). Here, we focus on the former approach and introduce the research from the view of inverse approaches.

There have been many approaches to quantitatively evaluate the actions of attacking players about the scoring, such as based on the expected scores using tracking data [34, 12, 39, 40, 3], action data such as dribbling and passing [10, 14], and estimating state-action value function (Q-function) [50, 33, 32]. Some researchers have evaluated passes [36, 4, 13], and others evaluated actions to receive a ball by assigning a value to the location with the highest expected score [45, 31] and a rule-based manner [21]. In particular, Spearman [45] proposed an evaluation metric called OBSO to evaluate behavior based on location data and rule-based modeling. Defensive behaviors have also been evaluated based on data-driven [38, 48] and rule-based manners (e.g., [47]). However, these score evaluations do not often reflect the position of multiple defenders and goal angles in rule-based manner.

From the perspective of reinforcement learning, there have been many studies on inverse approaches. As for the study of state evaluation, there are several studies based on score expectation (e.g., [6, 7, 16]) and based on the value of space (e.g., [5, 15]). There is also research on estimating reward functions by

inverse reinforcement learning [35,37]. Researchers performed trajectory prediction sometimes in terms of the policy function estimation, as imitation learning [29,28,47,19] and behavioral modeling [53,51,30,20] to mimic (not optimize) the policy using neural network approaches. In this paper, we first propose a method to evaluate how the actual "off-ball" movement contributes to scoring opportunity based on the difference between the state values generated from the actual and the reference policies.

# 5 Conclusion

In this paper, we evaluated players who create off-ball scoring opportunities by comparing actual movements with the reference movements generated by trajectory prediction. Our results suggest the effectiveness of the proposed method as an indicator for a player without the ball to create scoring opportunities for teammates. For future work, although the number of players to be evaluated was determined in the minimum setting, it is possible to evaluate the contribution to the scoring opportunities for teammates in a less limited way by predicting a larger number of players in both offense and defense. Furthermore, since our method evaluates off-ball players by comparing them with the referenced trajectory, the value becomes too small. Computing the evaluation value in a more interpretable way (e.g., in a score scale) would be future work. Finally, computing our indicators from broadcast videos (e.g., [11]) or other videos (e.g., top- or side-view [42]) would also be future work.

## Acknowledgments

This work was supported by JSPS KAKENHI (Grant Numbers 20H04075 and 21H05300) and JST Presto (Grant Number JPMJPR20CA).

## References

1. Anzer, G., Bauer, P.: A goal scoring probability model for shots based on synchronized positional and event data in football (soccer). Frontiers in Sports and Active Living **3**, 53 (2021)

2. Becker, S., Hug, R., Hübner, W., Arens, M.: Red: A simple but effective baseline predictor for the trajnet benchmark. In: European Conference on Computer Vision. pp. 138–153. Springer (2018)

3. Bransen, L., Van Haaren, J.: Measuring football players' on-the-ball contributions from passes during games. In: International workshop on machine learning and data mining for sports analytics. pp. 3–15. Springer (2018)

4. Brooks, J., Kerr, M., Guttag, J.: Developing a data-driven player ranking in soccer using predictive model weights. In: Proceedings of the 22nd ACM SIGKDD International Conference on Knowledge Discovery and Data Mining. pp. 49–55 (2016)

5. Cervone, D., Bornn, L., Goldsberry, K.: Nba court realty. In: 10th MIT Sloan Sports Analytics Conference (2016)

6. Cervone, D., D’Amour, A., Bornn, L., Goldsberry, K.: Pointwise: Predicting points and valuing decisions in real time with nba optical tracking data. In: Proceedings of the 8th MIT Sloan Sports Analytics Conference, Boston, MA, USA. vol. 28, p. 3 (2014)

7. Cervone, D., D’Amour, A., Bornn, L., Goldsberry, K.: A multiresolution stochastic process model for predicting basketball possession outcomes. Journal of the American Statistical Association **111**(514), 585–599 (2016)

8. Cho, K., Van Merriënboer, B., Bahdanau, D., Bengio, Y.: On the properties of neural machine translation: Encoder-decoder approaches. arXiv preprint arXiv:1409.1259 (2014)

9. Chung, J., Kastner, K., Dinh, L., Goel, K., Courville, A.C., Bengio, Y.: A recurrent latent variable model for sequential data. Advances in neural information processing systems **28**, 2980–2988 (2015)

10. Decroos, T., Bransen, L., Van Haaren, J., Davis, J.: Actions speak louder than goals: Valuing player actions in soccer. In: KDD. pp. 1851–1861 (2019)

11. Deliege, A., Cioppa, A., Giancola, S., Seikavandi, M.J., Dueholm, J.V., Nasrollahi, K., Ghanem, B., Moeslund, T.B., Van Droogenbroeck, M.: Soccernet-v2: A dataset and benchmarks for holistic understanding of broadcast soccer videos. In: 7th International Workshop on Computer Vision in Sports (CVsports) at IEEE/CVF Conference on Computer Vision and Pattern Recognition (CVPR’21). pp. 4508–4519 (2021)

12. Decroos, T., Dzyuba, V., Van Haaren, J., Davis, J.: Predicting soccer highlights from spatio-temporal match event streams. In: Proceedings of the AAAI Conference on Artificial Intelligence. vol. 31 (2017)

13. Dick, U., Link, D., Brefeld, U.: Who can receive the pass? a computational model for quantifying availability in soccer. Data Mining and Knowledge Discovery **36**(3), 987–1014 (2022)

14. Dick, U., Tavakol, M., Brefeld, U.: Rating player actions in soccer. Frontiers in Sports and Active Living p. 174 (2021)

15. Fernandez, J., Bornn, L.: Wide open spaces: A statistical technique for measuring space creation in professional soccer. In: 12th MIT sloan sports analytics conference (2018)

16. Fernández, J., Bornn, L., Cervone, D.: Decomposing the immeasurable sport: A deep learning expected possession value framework for soccer. In: 13th MIT Sloan Sports Analytics Conference (2019)

17. Fraccaro, M., Sønderby, S.K., Paquet, U., Winther, O.: Sequential neural models with stochastic layers. In: Advances in Neural Information Processing Systems 29. pp. 2199–2207 (2016)

18. Fujii, K.: Data-driven analysis for understanding team sports behaviors. Journal of Robotics and Mechatronics **33**(3), 505–514 (2021)

19. Fujii, K., Takeishi, N., Kawahara, Y., Takeda, K.: Policy learning with partial observation and mechanical constraints for multi-person modeling. arXiv preprint arXiv:2007.03155 (2020)

20. Fujii, K., Takeuchi, K., Kuribayashi, A., Takeishi, N., Kawahara, Y., Takeda, K.: Estimating counterfactual treatment outcomes over time in complex multi-agent scenarios. arXiv preprint arXiv:2206.01900 (2022)

21. Fujii, K., Yoshihara, Y., Matsumoto, Y., Tose, K., Takeuchi, H., Isobe, M., Mizuta, H., Maniwa, D., Okamura, T., Murai, T., et al.: Cognition and interpersonal coordination of patients with schizophrenia who have sports habits. PLoS One **15**(11), e0241863 (2020)

22. Goyal, A.G.A.P., Sordoni, A., Côté, M.A., Ke, N.R., Bengio, Y.: Z-forcing: Training stochastic recurrent networks. In: Advances in Neural Information Processing Systems 30. pp. 6713–6723 (2017)

23. JLEAGUE: Jleague.jp 2019 data (2019), [https://www.jleague.jp/stats/2019/goal.html](https://www.jleague.jp/stats/2019/goal.html)

24. Kingma, D.P., Ba, J.: Adam: A method for stochastic optimization. In: International Conference on Learning Representations (2015)

25. Kingma, D.P., Welling, M.: Auto-encoding variational bayes. In: International Conference on Learning Representations (2014)

26. Kipf, T., Fetaya, E., Wang, K.C., Welling, M., Zemel,R.: Neural relational inference for interacting systems. In: International Conference on Machine Learning. pp. 2688–2697 (2018)

27. Kurach, K., Raichuk, A., Stańczyk, P., Zając, M., Bachem, O., Espeholt, L., Riquelme, C., Vincent, D., Michalski, M., Bousquet, O., et al.: Google research football: A novel reinforcement learning environment. In: Proceedings of the AAAI Conference on Artificial Intelligence. vol. 34, pp. 4501–4510 (2020)

28. Le, H.M., Carr, P., Yue, Y., Lucey, P.: Data-driven ghosting using deep imitation learning. In: Proceedings of MIT Sloan Sports Analytics Conference (2017)

29. Le, H.M., Yue, Y., Carr, P., Lucey, P.: Coordinated multi-agent imitation learning. In: Proceedings of the 34th International Conference on Machine Learning-Volume 70. pp. 1995–2003. JMLR. org (2017)

30. Li, L., Yao, J., Wenliang, L., He, T., Xiao, T., Yan, J., Wipf, D., Zhang, Z.: Grin: Generative relation and intention network for multi-agent trajectory prediction. Advances in Neural Information Processing Systems **34** (2021)

31. Link, D., Lang, S., Seidenschwarz, P.: Real time quantification of dangerousity in football using spatiotemporal tracking data. PloS one **11**(12), e0168768 (2016)

32. Liu, G., Luo, Y., Schulte, O., Kharrat, T.: Deep soccer analytics: learning an action-value function for evaluating soccer players. Data Mining and Knowledge Discovery **34**(5), 1531–1559 (2020)

33. Liu, G., Schulte, O.: Deep reinforcement learning in ice hockey for context-aware player evaluation. arXiv preprint arXiv:1805.11088 (2018)

34. Lucey, P., Bialkowski, A., Monfort, M., Carr, P., Matthews, I.: quality vs quantity: Improved shot prediction in soccer using strategic features from spatiotemporal data. In: Proceedings of MIT Sloan Sports Analytics Conference. pp. 1–9 (2014)

35. Luo, Y., Schulte, O., Poupart, P.: Inverse reinforcement learning for team sports: Valuing actions and players. In: Bessiere, C. (ed.) Proceedings of the Twenty-Ninth International Joint Conference on Artificial Intelligence, IJCAI-20. pp. 3356–3363. International Joint Conferences on Artificial Intelligence Organization (7 2020)

36. Power, P., Ruiz, H., Wei, X., Lucey, P.: Not all passes are created equal: Objectively measuring the risk and reward of passes in soccer from tracking data. In: KDD. pp. 1605–1613 (2017)

37. Rahimian, P., Toka, L.: Inferring the strategy of offensive and defensive play in soccer with inverse reinforcement learning. In: Machine Learning and Data Mining for Sports Analytics (MLSA 2018) in ECML-PKDD Workshop (2020)

38. Robberechts, P.: Valuing the art of pressing. In: Proceedings of the StatsBomb Innovation In Football Conference. pp. 1–11. StatsBomb (2019)

39. Routley, K., Schulte, O.: A markov game model for valuing player actions in ice hockey. In: Proceedings of the Thirty-First Conference on Uncertainty in Artificial Intelligence. p. 782–791. UAI’15, AUAI Press, Arlington, Virginia, USA (2015)

40. Schulte, O., Khademi, M., Gholami, S., Zhao, Z., Javan, M., Desaulniers, P.: A markov game model for valuing actions, locations, and team performance in ice hockey. Data Mining and Knowledge Discovery **31**(6), 1735–1757 (2017)

41. Scott, A., Fujii, K., Onishi, M.: How does AI play football? An analysis of RL and real-world football strategies. In: 14th International Conference on Agents and Artificial Intelligence (ICAART’ 22). vol. 1, pp. 42–52 (2022)

42. Scott, A., Uchida, I., Onishi, M., Kameda, Y., Fukui, K., Fujii, K.: Soccertrack: A dataset and tracking algorithm for soccer with fish-eye and drone videos. In: 8th International Workshop on Computer Vision in Sports (CVsports) at IEEE/CVF Conference on Computer Vision and Pattern Recognition (CVPR’ 22). pp. 3569–3579 (2022)

43. Soccer-digest: Soccer digest web j1 rating (2019), [https://www.soccerdigestweb.com](https://www.soccerdigestweb.com)

44. Soccer-Money.net: Soccer-money.net (2019), [https://www.soccer-money.net](https://www.soccer-money.net)

45. Spearman, W.: Beyond expected goals. In: Proceedings of the 12th MIT sloan sports analytics conference. pp. 1–17 (2018)

46. Spearman, W., Basye, A., Dick, G., Hotovy, R., Pop, P.: Physics-based modeling of pass probabilities in soccer. In: Proceeding of the 11th MIT Sloan Sports Analytics Conference (2017)

47. Teranishi, M., Fujii, K., Takeda, K.: Trajectory prediction with imitation learning reflecting defensive evaluation in team sports. In: 2020 IEEE 9th Global Conference on Consumer Electronics (GCCE). pp. 124–125. IEEE (2020)

48. Toda, K., Teranishi, M., Kushiro, K., Fujii, K.: Evaluation of soccer team defense based on prediction models of ball recovery and being attacked. PLoS One **17**(1), e0263051 (2022)

49. Van Roy, M., Robberechts, P., Yang, W.C., De Raedt, L., Davis, J.: Learning a markov model for evaluating soccer decision making. In: Reinforcement Learning for Real Life (RL4RealLife) Workshop at ICML 2021 (2021)

50. Wang, J., Fox, I., Skaza, J., Linck, N., Singh, S., Wiens, J.: The advantage of doubling: a deep reinforcement learning approach to studying the double team in the nba. arXiv preprint arXiv:1803.02940 (2018)

51. Yeh, R.A., Schwing, A.G., Huang, J., Murphy, K.: Diverse generation for multi-agent sports games. In: The IEEE Conference on Computer Vision and Pattern Recognition (CVPR) (June 2019)

52. Zaheer, M., Kottur, S., Ravanbakhsh, S., Poczos, B., Salakhutdinov, R.R., Smola, A.J.: Deep sets. Advances in Neural Information Processing Systems **30** (2017)

53. Zhan, E., Zheng, S., Yue, Y., Sha, L., Lucey, P.: Generating multi-agent trajectories using programmatic weak supervision. In: International Conference on Learning Representations (2019)

# Appendix

## A Overview of our method

The overview of our method is shown in the Fig. 6. (i) First, we modify the score model in the framework of OBSO [45] with the potential score model that reflects the positions of multiple defenders with a mixed Gaussian distribution (Fig. 1B). (ii) Next, we accurately model the relationship between athletes and perform long-term trajectory predictions (Fig. 1A) using the graph variational recurrent neural network (GVRNN) [51]. (iii) Finally, based on the difference in the modified off-ball evaluation index between the actual and the predicted trajectory (Fig. 1A), we evaluate how the actual movement contributes to scoring opportunity relative to the predicted movement as a reference.

C-OBSO framework diagram showing actual vs reference player trajectories and a bar chart of off-ball chance creation for players A-E

actual value $V_k$ $-$ reference value $V'_k$ via prediction of $i$ = value $V_i$ created by $i$

Diagram of (i) modifying OBSO with potential score model showing pitch control, transition, and potential score model components

Diagram of (ii) Trajectory prediction with GVRNN architecture

**Fig. 6.** Overview of our method. (i) First, we propose a potential score model to improve OBSO [45]. (ii) We then predict players’ trajectories using GVRNN [51] to generate a reference player trajectory. (iii) Finally, the proposed C-OBSO is calculated by the difference between the evaluation value in the actual game situation and the referenced or predicted game situation.

## B Off-ball scoring opportunity [45]

Here, we describe the base model of our evaluation method called OBSO [45]. OBSO evaluates off-ball players by computing the following joint probability

$$P(G|D) = \sum_{r \in R \times R} P(S_r \cap C_r \cap T_r|D) \tag{7}$$

$$= \sum_r P(S_r|C_r, T_r, D)P(C_r|T_r, D)P(T_r|D), \tag{8}$$

where $D$ is the instantaneous state of the game (e.g., player positions and velocities). $P(S_r)$ is the probability of scoring from an arbitrary point $r \in R \times R$ on the pitch, assuming the next on-ball event occurs there. $P(C_r)$ is the probability that the passing team will control a ball at point $r$. $P(T_r)$ is the probability that the next on-ball event occurs at point $r$. Here, for simplicity, we can assume that $P(S_r|D), P(T_r|D), P(C_r|D)$ are independent if the parameter $\alpha = 0$ in the original work implementation (Eq. (6) in [45]). Then, the joint probability can be decomposed into a series of conditional probabilities as follows:

$$P(G|D) = \sum_{r \in R \times R} P(S_r|D)P(C_r|D)P(T_r|D). \tag{9}$$

We show the illustrative example of OBSO in Fig 7. $P(C_r|D)$ is the probability that the attacking team will control the ball at point $r$ assuming the next on-ball event occurs there, which is called the potential pitch control field (PPCF). $P(T_r|D)$ is defined as a two-dimensional Gaussian distribution with the current ball coordinates as the mean. The standard deviation is set to 14 m, which is the average distance of the next event [45]. $P(S_r|D)$ is simply calculated as a value that decreases with the distance from the goal. We used the grid data and computed $P(C_r|D)$ and $P(T_r|D)$ based on the code at [https://github.com/Friends-of-Tracking-Data-FoTD/LaurieOnTracking](https://github.com/Friends-of-Tracking-Data-FoTD/LaurieOnTracking).

Although $P(T_r|D)$ and $P(S_r|D)$ are simple functions, we need to explain $P(C_r|D)$ (PPCF) in more detail. PPCF [45] (a previous version is [46]) assumes that a player's ability to make a controlled touch on the ball (when near the ball) can be treated as a Poisson point process. That is, the longer a player is near the ball without another player interfering, the more likely it becomes that they can make a controlled touch on the ball. The model quantifies the probability of control for each player at each location on the pitch. The differential equation used to compute the control probability for each player at a specified location $r$ at time $t$ is:

$$\frac{dPPCF_j(t, r, T|s, \lambda_j)}{dT} = \left( 1 - \sum_k PPCF_k(t, r, T|s, \lambda_j) \right) f_j(t, r, T|s)\lambda_j, \tag{10}$$

where $f_j(t, r, T|s)$ represents the probability that player $j$ at time $t$ can reach location $r$ within time $T$. The parameter $s$ is the temporal uncertainty of player-ball intercept time (has units of s), which is used in $f_j(t, r, T|s)$ (we set $s = 0.45$ based on [46]). The parameter $\lambda_j$ is the rate of control representing the inverse of the mean time (has units of 1/s) which would take a player to make a controlled touch on the ball. Conceptually, we consider the probability that player $j$ will be able to control the ball during time $T$ to $T + dT$ with the decay rate $f_j(t, r, T|s)\lambda_j$. We set $PPCF_j(t, r, T|s, \lambda_j) = 0$ for the attacking or defending team if the opponent team can arrive significantly before the attacking

or defending team. By integrating Eq. (10) over $T$ from 0 to $\infty$, we obtain a per-player probability of control. We integrate it over the players of the attacking team. $f_j(t, r, T|s)$ is represented as a logistic function such that

$$f_j(t, r, T|s) = \left[ 1 + \exp \left( -\frac{T - \tau_{exp}(t, r)}{\sqrt{3}s/\pi} \right) \right]^{-1}, \tag{11}$$

where $\tau_{exp}(t, r)$ is a expected intercept time computed from the location and velocity of the player $j$ (including other constants, see [45] for the details). Conceptually, if $T - \tau_{exp}(t, r) \geq 0$, the player will tend to intercept the ball and a temporal uncertainty $\sqrt{3}s/\pi$ is assumed. For the control rate parameter $\lambda_j$, higher values of $\lambda_j$ mean less time is required before the player can control the ball. We set $\lambda_j = 4.3$ based on [46].

Off-ball scoring opportunities (OBSO) calculation diagram showing the product of Control C, Transition T, and Score S maps on a soccer field.

**Fig. 7.** Off-ball scoring opportunities (OBSO) [45]: an evaluation index for scoring opportunities in the off-ball state. The OBSO on the right is calculated by the joint probability of control, transition, and score probability.

## C Variational recurrent neural network [9]

In this section, we briefly overview recurrent neural networks (RNNs), variational autoencoders (VAEs), and variational RNNs (VRNNs).

From the perspective of a probabilistic generative model, an RNN models the conditional probabilities with a hidden state $h_t$ that summarizes the past history in the first $t - 1$ timesteps:

$$p_\theta(x_t|x_{<t}) = \varphi(h_{t-1}), \quad h_t = f(x_t, h_{t-1}), \tag{12}$$

where $\varphi$ maps the hidden state to a probability distribution over states and $f$ is a deterministic function such as LSTMs or GRUs. RNNs with simple output distributions often struggle to capture highly variable and structured sequential data. Recent work in sequential generative models addresses this issue by injecting stochastic latent variables into the model and using amortized variational inference to infer latent variables from data. VRNNs [9] is one of the methods using this idea and combining RNNs and VAEs.

VAE [25] is a generative model for non-sequential data that injects latent variables $z$ into the joint distribution $p_\theta(a, z)$ and introduces an inference network parameterized by $\phi$ to approximate the posterior $q_\phi(z \mid a)$. The learning objective is to maximize the evidence lower-bound (ELBO) of the log-likelihood with respect to the model parameters $\theta$ and $\phi$:

$$ \mathbb{E}_{q_\phi(z \mid a)} \left[ \log p_\theta(a \mid z) \right] - D_{KL}(q_\phi(z \mid a) || p_\theta(z)) \eqno(13) $$

The first term is known as the reconstruction term and can be approximated with Monte Carlo sampling. The second term is the Kullback-Leibler divergence between the approximate posterior and the prior, and can be evaluated analytically if both distributions are Gaussian with diagonal covariance. The inference model $q_\phi(z \mid a)$, generative model $p_\theta(a \mid z)$, and prior $p_\theta(z)$ are often implemented with neural networks.

VRNNs combine VAEs and RNNs by conditioning the VAE on a hidden state $h_t$:

$$ p_\theta(z_t | x_{<t}, z_{<t}) = \varphi_{\text{prior}}(h_{t-1}) \quad \quad \quad \quad \text{(prior)} \eqno(14) $$

$$ q_\phi(z_t | x_{\le t}, z_{<t}) = \varphi_{\text{enc}}(x_t, h_{t-1}) \quad \quad \quad \quad \text{(inference)} \eqno(15) $$

$$ p_\theta(x_t | z_{\le t}, x_{<t}) = \varphi_{\text{dec}}(z_t, h_{t-1}) \quad \quad \quad \quad \text{(generation)} \eqno(16) $$

$$ h_t = f(x_t, z_t, h_{t-1}). \quad \quad \quad \quad \text{(recurrence)} \eqno(17) $$

VRNNs are also trained by maximizing the ELBO, which can be interpreted as the sum of VAE ELBOs over each timestep of the sequence:

$$ \mathbb{E}_{q_\phi(z_{\le T} | x_{\le T})} \left[ \sum_{t=1}^T \log p_\theta(x_t \mid z_{\le T}, x_{<t}) \right. \eqno(18) $$
$$ \left. - D_{KL} \Big( q_\phi(z_t \mid x_{\le T}, z_{<t}) || p_\theta(z_t \mid x_{<t}, z_{<t}) \Big) \right] $$

Note that the prior distribution of latent variable $z_t$ depends on the history of states and latent variables (Eq. (14)).

# D Graph variational recurrent neural network [51]

Here, we briefly describe VRNN, GNN, and GVRNN.

In general, RNNs with simple output distributions often struggle to capture highly variable and structured sequential data (e.g., multimodal behaviors)

[53]. Recent work in sequential generative models addressed this issue by injecting stochastic latent variables into the model and optimization using amortized variational inference to learn the latent variables (e.g., [9, 17, 22]). Among these methods, a variational RNN (VRNN) [9] has been widely used in base models for multi-agent trajectories [51, 53, 19] with unknown governing equations. A VRNN is essentially a variational autoencoder (VAE) conditioned on the hidden state of an RNN and is trained by maximizing the (sequential) evidence lower-bound (ELBO), described in Appendix A.

Next, we overview a graph neural network (GNN) based on [26]. Let $v_k$ be a feature vector for each node $k$ of $K$ agents. Next, a feature vector for each edge $e_{(k,j)}$ is computed based on the nodes to which it is connected. The edge feature vectors are sent as "messages" to each of the connected nodes to compute their new output state $o_k$. Formally, a single round of message passing operations of a graph net is characterized below:

$$v \rightarrow e : e_{(k,j)} = f_e([v_k, v_j]), \tag{19}$$

$$e \rightarrow v : o_i = f_v \left( \sum_{j \in N(k)} e_{(k,j)} \right), \tag{20}$$

where $N(k)$ is the set of neighbors of node $k$, and $f_e$ and $f_v$ are neural networks. In summary, a GNN takes in feature vectors $v_{1:K}$ and outputs a vector for each node $o_{1:K}$, i.e., $o_{1:K} = \text{GNN}(v_{1:K})$. The operations of the GNN satisfy the permutation equivariance property as the edge construction is symmetric between pairs of nodes and the summation operator ignores the ordering of the edges [52].

Next, we describe GVRNN [51], which models the interactions between them at each step using GNNs. Let $x_{\leq T} = \{x_1, \dots, x_T\}$ denote a sequence of locations. In this paper, GVRNN update equations are as follows:

$$p_\theta(z_t | x_{<t}, z_{<t}) = \prod_k \mathcal{N}(z_{t,k} | \mu_{t,k}^{\text{pri}}, (\sigma_{t,k}^{\text{pri}})^2), \tag{21}$$

$$q_\phi(z_t | x_{\leq t}, z_{<t}) = \prod_k \mathcal{N}(z_{t,k} | \mu_{t,k}^{\text{enc}}, (\sigma_{t,k}^{\text{enc}})^2), \tag{22}$$

$$p_\theta(x_t | z_{\leq t}, x_{<t}) = \prod_k \mathcal{N}(z_{t,k} | \mu_{t,k}^{\text{dec}}, (\sigma_{t,k}^{\text{dec}})^2), \tag{23}$$

$$h_{t,k} = f_{rnn}(x_{t,k}, z_{t,k}, h_{t-1,k}). \tag{24}$$

where $h_t$ and $z_t$ are deterministic and stochastic latent variables. $p_\theta(x_t \mid z_{\leq t}, x_{<t})$, $q_\phi(z_t \mid x_{\leq t}, z_{<t})$, and $p_\theta(z_t \mid x_{<t}, z_{<t})$ are generative model, the approximate posterior or inference model, and the prior model, respectively. $\mathcal{N}(\cdot | \mu, \sigma^2)$ denotes a multivariate normal distribution with mean $\mu$ and covariance matrix $\text{diag}(\sigma^2)$,

and

$$ [\mu_{t, 1:K}^{\text{pri}}, \sigma_{t, 1:K}^{\text{pri}}] = \text{GNN}_{\text{pri}}(h_{t-1, 1:K}), \tag{25} $$

$$ [\mu_{t, 1:K}^{\text{enc}}, \sigma_{t, 1:K}^{\text{enc}}] = \text{GNN}_{\text{enc}}([x_{t, 1:K}, h_{t-1, 1:K}]), \tag{26} $$

$$ [\mu_{t, 1:K}^{\text{dec}}, \sigma_{t, 1:K}^{\text{dec}}] = \text{GNN}_{\text{dec}}([z_{t, 1:K}, h_{t-1, 1:K}]). \tag{27} $$

The prior network $\text{GNN}_{\text{pri}}$, encoder $\text{GNN}_{\text{enc}}$, and decoder $\text{GNN}_{\text{dec}}$ are GNNs with learnable parameters $\phi$ and $\theta$. Here we used the mean value $\mu_{t+1, 1:K}^{\text{dec}}$ as input variables $\hat{x}_{t+1}^{l'}$ in the following theory-based computation. GVRNN is trained by maximizing the sequential ELBO in a similar way to VRNN as described in Appendix C.

# E Validation results of trajectory prediction model

To verify the accuracy of the trajectory prediction model, We compared our approach with two baselines: VRNN [9] and RNN (RNN+Gauss) implemented using a gated recurrent unit (GRU) [8] and a decoder with Gaussian distribution for prediction [2].

For the MAE in the trajectory, to compare the various methods and time lengths, we first performed the Kruskal-Wallis test. As the post-hoc comparison, since we are interested in the differences from GVRNN (VRNN and RNN for four time lengths) and time lengths in GVRNN (4 and 6, 6 and 8, and 8 and 10 s), we performed the Wilcoxon rank sum test with Bonferroni correction such that the p-value was multiplied by 11 ($4 \times 2 + 3$).

We show the results of the trajectory prediction model for computing C-OBSO. The endpoint errors (MAE) of the three players are shown in Table 1. In the statistical evaluation, there were significant differences in all classification performance and tasks ($p < 10^{-10}$) using the Kruskal-Wallis test. In the following evaluations, we indicate the post-hoc comparison results. The trajectory prediction model used in C-OBSO (GVRNN) shows a lower prediction error than other models ($ps < 10^{-10}$).

In GVRNN, longer predictions show larger prediction errors ($ps < 10^{-10}$) except for the difference between 8 s and 10 s ($p > 0.05$). Note that we verified the existing GVRNN [51] performance, which uses a centralized optimization whereas VRNN and RNN+Gauss use the decentralized optimization (for each player). Since the 4 s prediction of GVRNN achieved a low the MAE of less than 0.7 m, the GVRNN trajectory prediction of 4 s was used in the next C-OBSO.

# F C-OBSO and OBSO results without the potential score model

To investigate the effect of the potential model on the C-OBSO and OBSO computations, we also computed C-OBSO and OBSO results without the potential score model. Results shown in Fig. 8 were similar to those with the potential

21

**Table 1.** Trajectory prediction endpoint errors (MAE and standard error) in three methods.

<table>
  <thead>
    <tr>
        <th> </th>
        <th>4 s</th>
        <th>6 s</th>
        <th>8 s</th>
        <th>10 s</th>
    </tr>
  </thead>
  <tbody>
    <tr>
        <td>GVRNN</td>
        <td><strong>0.608 ± 0.014</strong></td>
        <td><strong>0.867 ± 0.022</strong></td>
        <td><strong>1.701 ± 0.045</strong></td>
        <td><strong>1.606 ± 0.042</strong></td>
    </tr>
    <tr>
        <td>VRNN</td>
        <td>5.952 ± 0.118</td>
        <td>7.767 ± 0.160</td>
        <td>9.127 ± 0.188</td>
        <td>10.168 ± 0.225</td>
    </tr>
    <tr>
        <td>RNN+Gauss</td>
        <td>9.101 ± 0.144</td>
        <td>11.396 ± 0.202</td>
        <td>13.312 ± 0.245</td>
        <td>15.327 ± 0.302</td>
    </tr>
  </tbody>
</table>

model, but there were no significant correlations between the C-OBSO and salary ($\rho = 0.38, p = 0.08$) and between the OBSO and salary ($\rho = -0.18, p = 0.26$).

<table>
  <thead>
    <tr>
        <th>Player</th>
        <th>C-OBSO (A)</th>
        <th>OBSO (B)</th>
    </tr>
  </thead>
  <tbody>
    <tr>
        <td>Nakagawa</td>
        <td>0.009</td>
        <td>0.095</td>
    </tr>
    <tr>
        <td>Lee</td>
        <td>0.0092</td>
        <td>0.11</td>
    </tr>
    <tr>
        <td>Hatanaka</td>
        <td>0.008</td>
        <td>0.085</td>
    </tr>
    <tr>
        <td>Ohgihara</td>
        <td>0.0082</td>
        <td>0.075</td>
    </tr>
    <tr>
        <td>Amano</td>
        <td>0.0078</td>
        <td>0.08</td>
    </tr>
    <tr>
        <td>Edigar</td>
        <td>0.0068</td>
        <td>0.11</td>
    </tr>
    <tr>
        <td>Otsu</td>
        <td>0.0065</td>
        <td>0.065</td>
    </tr>
    <tr>
        <td>Endo</td>
        <td>0.0058</td>
        <td>0.075</td>
    </tr>
    <tr>
        <td>Thiago</td>
        <td>0.006</td>
        <td>0.1</td>
    </tr>
    <tr>
        <td>Matsubara</td>
        <td>0.0058</td>
        <td>0.085</td>
    </tr>
    <tr>
        <td>Marcos</td>
        <td>0.0052</td>
        <td>0.09</td>
    </tr>
    <tr>
        <td>Miyoshi</td>
        <td>0.0045</td>
        <td>0.11</td>
    </tr>
    <tr>
        <td>Hirose</td>
        <td>0.004</td>
        <td>0.082</td>
    </tr>
    <tr>
        <td>Theerathon</td>
        <td>0.0038</td>
        <td>0.06</td>
    </tr>
    <tr>
        <td>Kida</td>
        <td>0.0028</td>
        <td>0.068</td>
    </tr>
  </tbody>
</table>

**Fig. 8.** Relationship between indicators without the potential score model and annual salary in a team. (A) Relationship between C-OBSO without the potential model and the salary. (B) Relationship between OBSO [45] without the potential model and the salary. Configurations are same as Fig. 3.

## G Relationship between rating, C-OBSO, and goal

We additionally analyzed the relationship between the game rating by experts, C-OBSO, and the number of goals. First, we show the relationship between C-OBSO and the rating by experts of the top seven scorers in Table 2. We analyzed

seven players who played seven games or more and had two related scoring opportunities or more (for C-OBSO). There were no significant correlations between them for all players ($\rho s < 0.190$, $ps > 0.05$) except for Nakagawa.

Next, we also show the relationship between the goals of the top seven scorers and the ratings by experts in Table 3. We analyzed the games in which each player scored two points or more. There were significant correlations between them for all players ($\rho s > 0.516$, $ps < 0.018$).

**Table 2.** Relationship between C-OBSO and the rating by experts of the top seven creator of scoring opportunities for teammate for each game. We analyzed seven players who played seven games or more and had two related scoring opportunities or more (for C-OBSO). The OBSO values were different from Fig. 3 because the values in this table were computed by the mean and standard deviation of the mean value of each game.

<table>
  <thead>
    <tr>
        <th>Name</th>
        <th>Position</th>
        <th>No. of games</th>
        <th>Rating</th>
        <th>C-OBSO</th>
        <th>ρ</th>
        <th>p</th>
    </tr>
  </thead>
  <tbody>
    <tr>
        <td>Nakagawa</td>
        <td>FW</td>
        <td>17</td>
        <td>6.18 ± 0.64</td>
        <td>0.0043 ± 0.00624</td>
        <td>0.751</td>
        <td>0.0003</td>
    </tr>
    <tr>
        <td>Marcos</td>
        <td>FW</td>
        <td>14</td>
        <td>6.05 ± 0.60</td>
        <td>0.0032 ± 0.00324</td>
        <td>0.272</td>
        <td>0.1738</td>
    </tr>
    <tr>
        <td>Edigar</td>
        <td>FW</td>
        <td>10</td>
        <td>6.00 ± 0.71</td>
        <td>0.0038 ± 0.00291</td>
        <td>-0.371</td>
        <td>0.1454</td>
    </tr>
    <tr>
        <td>Endo</td>
        <td>MF</td>
        <td>7</td>
        <td>5.86 ± 0.56</td>
        <td>0.0020 ± 0.00248</td>
        <td>-0.418</td>
        <td>0.1751</td>
    </tr>
    <tr>
        <td>Amano</td>
        <td>MF</td>
        <td>7</td>
        <td>5.86 ± 0.38</td>
        <td>0.0086 ± 0.00617</td>
        <td>-0.116</td>
        <td>0.4024</td>
    </tr>
    <tr>
        <td>Ohgihara</td>
        <td>MF</td>
        <td>7</td>
        <td>6.00 ± 0.29</td>
        <td>0.0123 ± 0.01073</td>
        <td>-0.134</td>
        <td>0.3876</td>
    </tr>
    <tr>
        <td>Matsubara</td>
        <td>DF</td>
        <td>7</td>
        <td>6.14 ± 0.63</td>
        <td>0.0079 ± 0.01335</td>
        <td>0.189</td>
        <td>0.3426</td>
    </tr>
  </tbody>
</table>

**Table 3.** Relationship between the goals and the rating by experts of the top seven scorers. We analyzed the games in which each player scored two points or more. The ratings were different from Table 2 because of the different data selection criteria.

<table>
  <thead>
    <tr>
        <th>Name</th>
        <th>Position</th>
        <th>No. of games</th>
        <th>No. of goals</th>
        <th>Rating</th>
        <th>ρ</th>
        <th>p</th>
    </tr>
  </thead>
  <tbody>
    <tr>
        <td>Nakagawa</td>
        <td>FW</td>
        <td>33</td>
        <td>15</td>
        <td>6.33 ± 0.79</td>
        <td>0.630</td>
        <td>4.33E-05</td>
    </tr>
    <tr>
        <td>Marcos</td>
        <td>FW</td>
        <td>33</td>
        <td>15</td>
        <td>6.09 ± 0.92</td>
        <td>0.709</td>
        <td>1.98E-06</td>
    </tr>
    <tr>
        <td>Edigar</td>
        <td>FW</td>
        <td>16</td>
        <td>11</td>
        <td>6.14 ± 0.75</td>
        <td>0.912</td>
        <td>4.40E-07</td>
    </tr>
    <tr>
        <td>Erik</td>
        <td>FW</td>
        <td>12</td>
        <td>8</td>
        <td>6.13 ± 0.77</td>
        <td>0.599</td>
        <td>1.97E-02</td>
    </tr>
    <tr>
        <td>Endo</td>
        <td>MF</td>
        <td>29</td>
        <td>7</td>
        <td>5.88 ± 0.61</td>
        <td>0.613</td>
        <td>2.03E-04</td>
    </tr>
    <tr>
        <td>Theerathon</td>
        <td>DF</td>
        <td>25</td>
        <td>3</td>
        <td>5.88 ± 0.62</td>
        <td>0.517</td>
        <td>4.09E-03</td>
    </tr>
    <tr>
        <td>Miyoshi</td>
        <td>MF</td>
        <td>16</td>
        <td>3</td>
        <td>6.03 ± 0.64</td>
        <td>0.529</td>
        <td>1.75E-02</td>
    </tr>
  </tbody>
</table>