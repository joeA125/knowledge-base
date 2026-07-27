# EXPECTED POSSESSION VALUE OF CONTROL AND DUEL ACTIONS FOR SOCCER PLAYER’S SKILLS ESTIMATION

PREPRINT

Andrei Shelopugin
Independent Researcher
shelopuginandrey@gmail.com

### ABSTRACT

Estimation of football players’ skills is one of the key tasks in sports analytics. This paper introduces multiple extensions to a widely used model, expected possession value (EPV), to address some key challenges such as selection problem. First, we assign greater weights to events occurring immediately prior to the shot rather than those preceding them (decay effect). Second, our model incorporates possession risk more accurately by considering the decay effect and effective playing time. Third, we integrate the assessment of individual player ability to win aerial and ground duels. Using the extended EPV model, we predict this metric for various football players for the upcoming season, particularly taking into account the strength of their opponents.

**Keywords** soccer analytics, evaluation metrics, rating systems, glicko, performance prediction, epv

## 1 Introduction

Estimating players’ skills is a key challenge for football managers, scouts, and analysts. While video analysis can offer insights [1], conducting it for all players in the market is impractical. Analysts must therefore use filtering mechanisms to narrow down the selection pool. This can involve basic profile data like age, position, and nationality. Alternatively, leveraging statistics or metrics correlated with actual player performance offers a more effective approach. By using these metrics, analysts can make more informed decisions in player selection.

Currently, accurately assessing the skill levels of players based on historical data has become a complex task. Soccer, being a complicated game, presents various challenges in this regard, one of which is the fact that out of the 22 players on the field, only one possesses the ball at any given time. In recent years, sports analytics, including soccer analytics, has made significant advancements in the understanding how to effectively estimate players’ abilities, particularly in terms of ball possession skills.

In the past, player analysis predominantly relied on basic statistics such as goals scored, successful passes, and fouls committed. However, there has been a shift towards utilizing advanced metrics, which provide a more comprehensive and nuanced evaluation of player performance.

The challenge with these metrics lies in their reliance on historical data, limiting their use in predicting future performance. While they may work well in "closed" leagues like the NBA, where teams share similar strengths, soccer’s diverse leagues present a more complex scenario. This complicates player selection, as managers must understand how players will adapt to new clubs or leagues. Analysts need to assess the resistance levels of different leagues and the stylistic characteristics of new clubs for optimal player hiring decisions.

In this paper, we modify the expected possession value model. Using this model, we calculate a reward, accumulated by each player during the season, that characterizes a footballer’s ability to play with the ball and predict the values of this reward for the next season.

## 2 Related Work

J. Hollinger introduced the Player Efficiency Rating ($PER$)[2] as a metric for evaluating NBA players, utilizing basic statistics. He introduced the concept of the value of possession and suggested a formula that rewards and punishes players for successful and unsuccessful actions.

Pollard, Ensum, and Taylor developed the expected goals model (xG) based on logistic regression, which predicts the likelihood of a shot resulting in a goal[3]. Pollard and Reep[4] suggested estimating possessions in risk-reward terms by assigning a value of xG to shots taken during a possession and a value of -$xG$ to shots taken by the opponent team in the subsequent possession.

Spearman [5] introduced the term off-ball scoring opportunities (OBSO) which represents the posterior probability of scoring with the next on-ball event at a particular location based on tracking data. Spearman et al. proposed the idea of pitch control, based on a physical model that predicts that a pass will be accurate[6]. Power et al. developed a pass model to analyze decision-making in risk-reward contexts[7].

Singh introduced the Expected Threat metric based on a Markov chain approach[8]. Cervone, D., D’Amour, A., Bornn, L., and Goldsberry suggested the expected possession value model (EPV) for basketball tracking data, taking into account individual player skills[9], [10]. Fernandez et al. apply EPV for soccer data[11].

Garnier and Gregoir utilized deep reinforcement learning techniques and introduced a metric called Expected Discounted Goal (EDG)[12]. Liu et al.[13] approached the estimation of EPV as a reinforcement learning (RL) task. They developed the Goal Impact Metric (GIM) as a result of their RL-based solution.

Dinsdale and Gallagher[14] predicted values of several player metrics for the first 1,000 minutes of a player at a new club, or the next 1,000 minutes if a player remains at their current club, taking into account Elo[15] ratings of clubs and leagues.

## 3 Proposed Approach for EPV

### 3.1 Background

This section introduces our customized version of the Expected Possession Value (EPV) model. The model has been built using events data, which includes actions such as passes, shots, dribbles, etc. Each event is described by relevant details, including coordinates, time, and other information. This paper employs the following notation to formalize the problem:

* $i$ - index of event,

* $c_i$ - control action,

* $d_i$ - duel action,

* $t_i$ - effective playing time of event $i$,

* $s_i$ - index of possession of event $i$,

* $xG_i$ - $xG$ of event $i$, if the event is a shot,

* $PV(e_i)$ - possession value of event $i$,

* $EPV(e_i)$ - expected possession value of event $i$ (prediction of EPV model).

By *control actions* we consider the actions such as a pass, a shot, a dribble, a carry (ball control), a free kick, a goal kick, a penalty, a corner kick, a throw in. The idea behind this definition is that it is clear which player possesses the ball during these actions.

Aerial and ground duels are considered *symmetrical duels*. Dribble is not a symmetrical duel.

We define *possession* as a series of control actions by the same team uninterrupted by the opponent. In theory, team may touch the ball during the opponent's possession, like intercepting the ball, but if it doesn't lead to their control action, the opponent's possession continues. A goalkeeper's save, leading to a corner or offensive rebound, does not interrupt the opponent's possession. Therefore, one possession could include several shots.

*Effective playing time* is the total amount of time that the ball is in play during the match. Most football metrics are normalized based on dirty playing time. The duration of pauses during VAR reviews, substitutions, and set-piece situations should be excluded. Therefore, we will recalculate our metrics based on effective playing time, considering only the actual time of active gameplay. Furthermore, when we use terms like playing time, minutes, seconds, etc., we are referring to effective playing time.

## 3.2 Possession Value of Control Actions

We define our model's target variable as the *possession value*. We then refer to the model's prediction as the *expected possession value (EPV)*. Traditionally, there are two approaches to defining the EPV model's target. The first assigns a value of 1 to each event within a possession if it results in a goal, and 0 otherwise. However, we avoid this approach due to the limited number of goals, which could cause overfitting.

The alternative approach involves assigning each event within a possession the cumulative sum of $xG$ from *future shots* during that possession.

$$PV(c_i) = (1 - \prod_{j=1}^{\infty} (1 - xG_j [t_j \ge t_i][s_i = s_j])) \tag{1}$$

We would like to highlight a couple of key points. Firstly, this formula is correct only to control actions, as exclusively in these cases can we definitively determine which player has possession. Secondly, we consider only the $xG$ of shots that occur after a specific event because the reward is determined by future events rather than past ones. Thirdly, we take into account the possibility of a subsequent goal occurring because the initial shot did not result in a goal. For that reason, the term $(1 - xG_j)$ is included in equation (1). A description of our $xG$ model can be found in the fourth chapter.

## 3.3 Decay Effect

Formula (1) has a significant drawback. It assigns the same possession value ($PV$) to all events preceding the corresponding shots, implying equal importance for each action. This assumption is invalid. Thus, we introduce a decay effect where actions leading to shots carry more weight. Hence, we modify the formula as follows:

$$PV(c_i) = (1 - \prod_{j=1}^{\infty} (1 - \gamma^{(t_j - t_i)} xG_j [t_j \ge t_i][s_i = s_j]) \tag{2}$$

The discount factor was proposed by Samuelson [16] and has found widespread use in economics and reinforcement learning, prioritizing receiving rewards sooner. However, in our context, the interpretation of the discount factor differs. We don't encourage earlier shots. Instead, we suggest that if an action doesn't lead to a shot soon, it may not significantly advance the attacking phase. We use a discount factor of 0.95. This value is a hyperparameter that can be adjusted based on preferences. Analysts favoring a vertical attacking style might use a lower gamma value like 0.9, while those preferring a *tiki-taka* style might choose a higher value like 0.99.

Another point to highlight is that the formula operates correctly due to our consideration of effective playing time. For example, if a player draws a penalty, he receives a substantial reward since the penalty kick occurs immediately after. This ensures rewards accurately reflect the timing and impact of events in the game.

The concept of employing a discount factor in soccer is not entirely new. For example, in [12], the discounted expected number of goals that a team will score (or concede) was used as a target value for an RL algorithm. However, this target value lacks precision without taking into account the $xG$ and effective time concepts.

## 3.4 Possession Risk

Some *EPV* models incorporate possession risk[4]. An accurate forward pass not only increases the team’s scoring chances but also reduces the threat near their own goal. These models consider the difference between the team’s $PV$ and the opponent’s $PV$ in the subsequent possession, rather than just the team’s possession value.

The penalty for possession risk in this approach has a significant drawback. For instance, consider a player who completes a pass, but the team loses possession 10 seconds later, leading to the opponent earning a penalty kick 20 seconds after that. In this approach, the player’s pass would be assigned a target value of -0.75 (the $xG$ value of the penalty kick), which is counterintuitive due to the elapsed time. To address this, we incorporate the decay effect from (2). Using the decay factor, the value is $-0.95^{30} * 0.75 = -0.16$, showing the player’s action was less influential in the penalty.

Another issue with the original approach to assessing possession risk is that it only considers two possessions. However, a team might lose possession, quickly regain it, and score on the third possession. By incorporating the decay effect, we can account for a larger number of possessions.

Thus, we arrive at the following formula for the $PV$, which will be used as the target value for our *EPV* model for control actions:

$$PV(c_i) = \sum_{s_j \in \text{team}} \left( 1 - \prod_{j=1}^{\infty} (1 - \gamma^{(t_j - t_i)} xG_j [t_j \geq t_i]) \right) - \sum_{s_j \in \text{opponent}} \left( 1 - \prod_{i=1}^{\infty} (1 - \gamma^{(t_j - t_i)} xG_j [t_j \geq t_i]) \right) \tag{3}$$

## 3.5 Possession Value of Symmetrical Duels

We also calculate *EPV* for symmetrical duels. To address the challenge of assigning possession for duel events, we assign the possession value of the first control action following the duel to that duel. If there is a series of symmetrical duels, all are recursively assigned the same possession value.

$$PV(d_i) = \begin{cases} PV(e_{i+1}) & \text{if } s_i = s_{i+1} \\ -PV(e_{i+1}) & \text{if } s_i \neq s_{i+1} \end{cases} \tag{4}$$

## 3.6 Reward Metrics

Before building the *EPV* model, we define our research goal. Our objective is to implement a metric that measures a player/team’s ability to "improve" possession. We highlight five possible outcomes of a player’s actions:

1. The action leads to a control action by the same team.

2. The action leads to a control action by the opponent team.

3. The goal was scored.

4. The action was the last in the half.

5. The action leads to a symmetrical duel.

In the first scenario, if a control action or symmetrical duel maintains possession, the player receives a *reward* equal to the difference between the *EPV* values of neighboring events. In the second scenario, if a control

action or duel results in a turnover, the next ($i + 1$) event is by the opposing team. Thus, the player is penalized twice: once for the turnover and once for providing a goal opportunity to the opponent.

If possession ends with a goal, the player receives a reward of ($1 - EPV(c_i) - EPV(c_{i+1})$), where $c_{i+1}$ denotes a pass from the center circle after a goal. In the fourth scenario, we assign a reward value of zero to neither reward nor punish the player.

The fifth scenario is the most challenging. For example, a pass into an aerial duel initially gets a negative reward for creating a 50/50 situation. However, we must consider the potential mismatch between players involved. We address this by using an improved approach to estimate symmetrical duel skills [17]. By adding the probability of winning a duel as a feature in the EPV model, we can more accurately evaluate the reward. If the teammate has a high probability of winning the duel, the passing player receives a greater reward. However, when calculating the reward for players in symmetrical duels, we won't consider the probability of winning the duel. Instead, we compare the actual outcome with the "average" outcome to avoid penalizing a player for superior skills by inflating the EPV.

Different feature sets describe control actions and symmetrical duels. Hence, we built three types of models: one for control actions, one for symmetrical duels for the "average" player, and one for symmetrical duels with player duel skill estimation. These are denoted as $EPV$, $EPV_{duel}^{avg}$, and $EPV_{duel}^{ind}$, respectively. Thus, we've designed this $reward$ formula to evaluate players' control actions and symmetrical duels.

If $i$ is a control action:

$$
\Delta EPV(c_i) = \begin{cases} EPV(c_{i+1}) - EPV(c_i), & \text{if scenario 1} \\ -EPV(c_{i+1}) - EPV(c_i), & \text{if scenario 2} \\ 1 - EPV(c_i) - EPV(c_{i+1}), & \text{if scenario 3} \\ 0, & \text{if scenario 4} \\ EPV_{duel}^{ind}(d_{i+1}) - EPV(c_i), & \text{if scenario 5} \end{cases} \tag{5}
$$

If $i$ is a symmetrical duel:

$$
\Delta EPV(d_i) = \begin{cases} EPV(c_{i+1}) - EPV_{duel}^{avg}(d_i), & \text{if scenario 1} \\ -EPV(c_{i+1}) - EPV_{duel}^{avg}(d_i), & \text{if scenario 2} \\ 0, & \text{if scenario 4} \\ EPV_{duel}^{ind}(d_{i+1}) - EPV_{duel}^{avg}(d_i), & \text{if scenario 5} \end{cases} \tag{6}
$$

Regarding our target function, it's crucial to understand that pass accuracy doesn't directly affect rewards. For instance, an inaccurate cross leading to a defender's handball still results in a penalty kick attempt. This underscores the use of the concept of $possession$ in our approach. Additionally, our analysis excludes certain events like interceptions. Hence, when calculating rewards, we omit these actions. For instance, a sequence like "pass-interception-shot" is interpreted as "pass-shot" in our analysis.

# 4 EPV Implementation

## 4.1 Expected Goals

Soccer's low-scoring nature, averaging 2.6 goals per game, poses challenges in assessing player performance. The emergence of $xG$ as a measure of scoring opportunities addresses this challenge, representing the probability of a shot resulting in a goal, solely based on game situation, not individual player skills. Our goal is to develop a model that evaluates scoring opportunities for the average player.

We create two distinct models: one for set-piece shots and another for open-play shots. Factors such as shot coordinates, distance, angle from the goal, and set-piece type (penalty, free-kick, corner) were considered. For open-play shots, attributes like body part used and preceding actions (e.g., aerial duel, pass) were included, along with spatial details. Separate model was built for set-pieces, as $xG$ should be independent of previous events.

Certain features like current game score, championship, and player’s team were omitted due to their correlation with player skill. Another challenge is the dataset revealed a non-uniform distribution of shots, with top players having more scoring opportunities. To tackle this, we implemented a custom loss function, choosing a log-loss function as the basic loss function for each shot. We divided the loss function value for each shot by its appearance count in the training set, reducing the focus on overrepresented players.

$$customlogloss_i = \frac{1}{|player_i \in D|} [y_i \log(p_i) + (1 - y_i) \log(1 - p_i)] \tag{7}$$

We trained the LightGBM [18] and CatBoost [19] frameworks, with LightGBM showing superior accuracy. We implemented equation (7) as an *objective* parameter in this framework.

## 4.2 Symmetrical Duels

Here, we present an improved version of our approach [17]. However, we focus only on *symmetrical duels*.

Analysts often use win percentage as a measure of player duel skill, but this overlooks opponent strength. Players may have high win rates against weaker opponents. Additionally, coaches often match players of similar strength, especially in set-pieces, leading to win rates converging to fifty percent as strong players face strong opponents and weak players face weak ones.

Another approach suggested by Garry Gelade [20] uses the Bradley-Terry model [21] to calculate player ratings for duels. This approach is superior as it considers opponent strength and enables modeling of future situations. The metric is transferable across leagues, allowing comparisons between players in diverse competitions.

Gelade’s method has limitations. It assumes that players have equal chances to win duels without considering external factors, which isn’t always accurate. For example, in aerial duels, defending players have an advantage as they face the opponent’s goal during the defensive phase. Another limitation is that the Bradley-Terry model is not state-of-the-art. Therefore, we opted to use a modified version of Glicko-2 [22].

Introduce a definition of a duel winner. In both cases (aerial and ground duels), we will adhere to the following logic:

1. If a player suffered a foul, he is considered the winner.

2. A player who makes the first touch on the ball is considered the winner.

3. If no one touches the ball, we will consider the player whose team gains possession after the duel as the winner.

We must consider that duels aren’t fully symmetrical. Outcomes depend not only on players’ skills but also on external factors, such as the location of the duel or the type of pass leading to it.

The original Glicko-2 version updates rating in this way:

$$\mu' = \mu + \phi'^2 g(\phi_j)(s_j - E(\mu, \mu_j, \phi_j)) \tag{8}$$

We have modified it for a defender:

$$\mu' = \mu + \phi'^2 g(\phi_j)(s_j - E(\mu + a, \mu_j, \phi_j)) \tag{9}$$

We determine advantage as follows: using a model predicting duel outcomes based on contextual features like duel and pass coordinates, pass type (e.g., corner, free kick), and number of opponents. We exclude player skill-related features, aiming to create an "average" model describing duel difficulty.

We encounter a data leak due to two factors. Firstly, the defending team holds an advantage in aerial duels. Secondly, central defenders typically excel in aerial duels compared to other positions, but they occur more frequently in defensive roles, leading to underestimated ratings. To address this, we categorize player positions into six groups: central defenders, full-backs, midfielders, central forwards, wingers, and goalkeepers.

We train model by filtering aerial duels where players involved occupy the same position category. We apply the same logic as described in formula (7).

Therefore, we train a LightGBM model to calculate the probability of winning a duel. Based on this probability, we calculate the average advantage ($a$ from (9)) for a Glicko-2-based model. This allows us to determine individual aerial and ground duel ratings for each player, considering the duel context. The ratings are presented in Tables 4 and 5.

## 4.3 Expected Possession Value

We built the EPV model in a similar way to the $xG$ model, utilizing spatial characteristics of the action and the preceding one. Six separate models were trained:

1. For control actions in open-player situations.

2. For set pieces, as they should depend only on the current action.

3-4. Aerial and ground average EPV models, $EPV_{duel}^{avg}$. These models describe the context of the duel while ignoring the player's skill in aerial duels. We used the predictions of the LightGBM model from chapter 4.2 as a feature to describe the context of the duel. These models help calculate the reward (6), which allows to estimate players' abilities to "improve" possession in situations where the player participates in a duel.

5-6. In contrast to the previous point, here we want to describe the aerial or ground duel situation considering player skills. It helps calculate pass reward more accurately, as seen in scenario 5 of formulas (5), (6). We used the following features: the probability to win the duel (taking into account player ratings), the player's rating, the opponent's rating, as well as the spatial features of the duel and the pass leading to the duel.

Analogously to equation (7), we implemented a custom mean squared error:

$$customMSE_i = \frac{1}{|player_i \in D|} (y_i - \hat{y}_i)^2 \eqno(10)$$

# 5 Metrics Prediction for the Upcoming Season

## 5.1 Season Pass Carry Reward

We introduce the metric *Pass Carry Reward (PCR)* as a measure of how a player enhances possession through passes or carries over the season. For the term *carry*, we encompass all events where a player controls the ball, including dribbles, carries, or simply maintaining possession without significant movement. By summing all $\Delta EPV(e_i)$ values throughout the season, we can derive a season reward, designated as $PCR$, focusing solely on pass and carry events. To assess a player's season performance accurately, it's crucial to normalize this reward based on the player's effective playing time, specifically their effective time per 60 minutes of play.

$$PCR(player) = \frac{60 \times \sum \Delta EPV(e_i | player, e_i = pass \lor e_i = carry)}{\sum minutes} \eqno(11)$$

The idea behind this paper is straightforward: to predict players' $PCR$ for the upcoming season. However, there are certain limitations and restrictions that we need to address and discuss.

## 5.2 Training Set

First and foremost, it is important to define the group of players for whom we will train our model. Our decision is to focus only on players who have accumulated at least 100 minutes in both the current season and the next season. This criterion is implemented to ensure the stability of $PCR$ predictions, as values of $PCR$ for players with limited playing time tend to be less reliable. The potential drawbacks of dropping data based on this criterion will be discussed in the subsequent subsection.

## 5.3 Features

The calculated features, approximately 600 in total, can be categorized into several groups:

**Player-specific features.** These include attributes such as age, height, position, and other characteristics that describe the individual player.

**Performance features.** This group encompasses metrics like $PCR$, $xG$, goals, played minutes, and other raw statistics from the previous seasons. Additionally, average values of these metrics over the past three or five seasons are also considered.

**Contribution to team success.** These statistics capture the player's impact on the team's overall performance. For instance, features like the player's share of the team's total $xG$ when they were on the field are included. These features are crucial as they provide insights into a player's effectiveness, especially when playing for a weaker team where their absolute metrics may be lower.

**League style features.** This category includes metrics that describe the prevalent playing style in the league. For example, the average $PCR$ of the league in the previous season is considered. These features help address challenges such as the influence of league-specific playing styles on $PCR$. For a player in a league that prioritizes attacking play, it may be easier to achieve higher $PCR$ values.

**Team and league strength features.** These features account for factors such as team and league strength, particularly in the context of player transfers. They provide information regarding a player's transfer, such as moving from a strong team to a weaker one, and help contextualize the player's performance accordingly. A detailed discussion regarding this last group of features will be presented in the subsequent subsection.

## 5.4 Ratings of Clubs and Leagues

To address the issue of player transfers and the potential impact on $PCR$ prediction, it is important to consider the information associated with changes in clubs or leagues. One approach could involve introducing a categorical feature that indicates the transfer from one league to another, such as "player transfers from Denmark Superliga to France Ligue 1". However, this approach may encounter challenges related to the curse of dimensionality, as there may not be a significant number of transfers between specific leagues. Additionally, it is worth noting that the strengths of leagues can vary over time.

To overcome these challenges, we propose utilizing a modified Glicko-2 rating system [23]. This model calculates ratings for teams based on match outcomes, allowing for the estimation of team strength. The strength of a league is defined as the average rating of a specified number of teams, with the exact number being adjustable to account for variations across leagues.

With this modified rating system, we can calculate various features, such as the rating of the player's old team, the rating of the player's new team, and the difference in ratings between the player's old league and the new league. These features provide valuable information related to the player's transition and enable us to account for changes in team and league strength. Additionally, we calculate the average rating of team opponents, as two players can demonstrate similar metrics; however, the level of opponents can differ.

It is important to note that we utilized the actual club/league ratings at the start of the season. This approach allows the model to incorporate up-to-date information about the strength of the clubs/leagues, considering that their levels may change over time. Moreover, we must adhere to the natural constraint that we cannot use the future ratings of the clubs/leagues.

## 5.5 Probability to Stay in the Data

As mentioned earlier, our training set includes only instances where players have played at least 100 minutes in the next season. This limitation poses a challenge as it is not possible to know in advance how many minutes a player will play in the next season. To address this issue, we build an auxiliary model.

We predict whether a football player will play at least 100 minutes in the next season. A player may have less than 100 minutes for various reasons: retirement from professional football, injury, underperformance

resulting in exclusion from matches by the coach, or transfer to a team or league not presented in the dataset, indicating the transfer to a less competitive league.

We train the model using the same features as those employed in the previous model, excluding the features related to the ratings of a new team or league. In addition, we incorporated data on players’ contract durations at the beginning of the season, leveraging information sourced from the FIFA video game series [24].

## 5.6 Presence-only Data Problem

Another challenge in training our model is the non-random nature of real-world transfers. Football club managers make informed decisions when acquiring players, leading to two key scenarios. When a player moves from a weaker club to a stronger one, it suggests the new club sees potential or talent in the player. When a player moves from a strong club to a weaker one, it suggests a decline in performance. These scenarios introduce bias in our predictions. We use features like the "rating of the new team" to inform the model about transfers, but the outcomes remain uncertain. Consequently, predictions can be overly optimistic in the first scenario and overly pessimistic in the second, reflecting the uncertainties and biases inherent in real-world transfers.

This problem is known as *presence-only data* or *learning from positive and unlabeled data* [25]. First, account for the data leak from leaving data, assigning a probability $p_l$. Second, consider the data leak from subsequent transfers, which is highly correlated with league rating differences. Assign this as $\Delta ratings$, with 1500 as the average club rating.

$$\Delta ratings = \frac{(rating(league_{new}) - rating(league_{old}))}{1500} \tag{12}$$

$$PCR_{adj} = PCR * 0.8^{(\Delta ratings + p_l)} \tag{13}$$

## 6 Results

As a baseline, we calculate the root mean squared error ($RMSE$) and mean absolute error ($MAE$) between the $PCR$ from the previous season and the next season (refer to Table 1). This imitates a selection process based on historical data without applying machine learning. We separate the data into several groups based on the following binary rules: whether the player is in the same team/league as in the previous season or not, and whether the player played more than 1000 minutes in the previous season or not. We remember that we are trying to solve a selection problem; therefore, we assume that a player may change clubs or even leagues.

Table 1: Baseline Metrics Without Applying Machine Learning

<table>
  <thead>
    <tr>
        <th>Data Sample</th>
        <th>RMSE, &gt;100 min</th>
        <th>MAE, &gt;100 min</th>
        <th>RMSE, &gt;1000 min</th>
        <th>MAE, &gt;1000 min</th>
    </tr>
  </thead>
  <tbody>
    <tr>
        <td>all data</td>
        <td>0.053</td>
        <td>0.036</td>
        <td>0.042</td>
        <td>0.029</td>
    </tr>
    <tr>
        <td>the same team, the same league</td>
        <td>0.05</td>
        <td>0.034</td>
        <td>0.039</td>
        <td>0.027</td>
    </tr>
    <tr>
        <td>the same team, a new league</td>
        <td>0.051</td>
        <td>0.035</td>
        <td>0.042</td>
        <td>0.031</td>
    </tr>
    <tr>
        <td>a new team, the same league</td>
        <td>0.055</td>
        <td>0.038</td>
        <td>0.044</td>
        <td>0.031</td>
    </tr>
    <tr>
        <td>a new team, a new league</td>
        <td>0.061</td>
        <td>0.043</td>
        <td>0.051</td>
        <td>0.036</td>
    </tr>
  </tbody>
</table>

By comparing the RMSE and MAE with the predictions of our model (without adjustment), we can estimate the contribution of our approach (refer to Table 2).

However, the above-mentioned evaluation method has its limitations. Our end product is a player ranking based on predicted $PCR$, and there is no guarantee that the calculated shortlist is accurate. Additionally, there is no mathematical way to prove that $EPV$-based metrics truly correlate with player skills. We see two ways to evaluate our shortlists. The first is to ask experts, which may be done in future work. The second way is to assume that the transfer market is enough effective at least at the top level and compare top players from our shortlists with actual transfers to top clubs.

Table 2: Achieved Metrics Based On Our Approach

<table>
  <thead>
    <tr>
        <th>Data Sample</th>
        <th>RMSE, &gt;100 min</th>
        <th>MAE, &gt;100 min</th>
        <th>RMSE, &gt;1000 min</th>
        <th>MAE, &gt;1000 min</th>
    </tr>
  </thead>
  <tbody>
    <tr>
        <td>all data</td>
        <td>0.033</td>
        <td>0.023</td>
        <td>0.031</td>
        <td>0.021</td>
    </tr>
    <tr>
        <td>the same team, the same league</td>
        <td>0.032</td>
        <td>0.022</td>
        <td>0.029</td>
        <td>0.02</td>
    </tr>
    <tr>
        <td>the same team, a new league</td>
        <td>0.031</td>
        <td>0.021</td>
        <td>0.027</td>
        <td>0.019</td>
    </tr>
    <tr>
        <td>a new team, the same league</td>
        <td>0.034</td>
        <td>0.024</td>
        <td>0.032</td>
        <td>0.023</td>
    </tr>
    <tr>
        <td>a new team, a new league</td>
        <td>0.037</td>
        <td>0.026</td>
        <td>0.036</td>
        <td>0.025</td>
    </tr>
  </tbody>
</table>

Table 6 presents the top season performances based on the $PCR$ metric among players who played at least 1,000 minutes during the season. The $PCR$ predictions for Manchester City, Barcelona, Milan, and Brighton can be found in Tables 7-10. Brighton was chosen because this club has a reputation for having a strong data analysis department and making transfers based on data. The actual date is June 1, 2024.

There is also a case study section in the appendix that demonstrates the advantage of using individual duel skills in the $EPV$ model.

# 7 Conclusion

We improve the EPV model by accurately accounting for effective playing time, decay effect, and possession risk. The incorporation of player duel skills allows us to calculate rewards more accurately. Therefore, we partially solve the problem of reward distribution between the passing player and the target player in cases of passes leading to duels. However, the problem still exists in cases of accurate passes, and we do not have a solution based on event data.

This model can potentially demonstrate higher performance with tracking data due to the consideration of more features. The inclusion of injury data is promising not only for enhancing the predictive capacity of the model that calculates the probability of a player remaining in the dataset but also for improving the prediction of $PCR$ for the next season. Additionally, we need to consider the context of new and old teams more accurately. For instance, we can replace $PCR$ with a conditional expectation that takes into account the player's position in the next club. For example, our model lacks accuracy when a center forward starts to play as a winger in a new club. Finally, the presence-only data problem should be tackled with a more elegant approach, which is a subject for further research.

# 8 Acknowledgments

The author is thankful to Nikita Kozodoi, Viktoria Lokteva, Nikita Vasyukhin, Iskander Safiulin, Daniil Babaev, and Alexander Sirotkin for their invaluable consultations on machine learning and soccer analytics.

# References

[1] T. L. Bergkamp, W. G. Frencken, A. S. M. Niessen, R. R. Meijer, and R. J. den Hartigh, "How soccer scouts identify talented players," *European Journal of Sport Science*, vol. 22, no. 7, pp. 994–1004, 2022.

[2] J. Hollinger, "Pro basketball forecast," (*No Title*), 2005.

[3] R. Pollard, J. Ensum, and S. Taylor, "Estimating the probability of a shot resulting in a goal: The effects of distance, angle and space," *International Journal of Soccer and Science*, vol. 2, no. 1, pp. 50–55, 2004.

[4] R. Pollard and C. Reep, "Measuring the effectiveness of playing strategies at soccer," *Journal of the Royal Statistical Society Series D: The Statistician*, vol. 46, no. 4, pp. 541–550, 1997.

[5] W. Spearman, "Beyond expected goals," in *Proceedings of the 12th MIT sloan sports analytics conference*, 2018, pp. 1–17.

[6] W. Spearman, A. Basye, G. Dick, R. Hotovy, and P. Pop, "Physics-based modeling of pass probabilities in soccer," in *Proceeding of the 11th MIT Sloan Sports Analytics Conference*, 2017.

[7] P. Power, H. Ruiz, X. Wei, and P. Lucey, “Not all passes are created equal: Objectively measuring the risk and reward of passes in soccer from tracking data,” in *Proceedings of the 23rd ACM SIGKDD international conference on knowledge discovery and data mining*, 2017, pp. 1605–1613.

[8] K. Singh, “Introducing expected threat,” [https://karun.in/blog/expected-threat.html](https://karun.in/blog/expected-threat.html), 2018, accessed: 2024-05-30.

[9] D. Cervone, A. D’amour, L. Bornn, and K. Goldsberry, “Pointwise: Predicting points and valuing decisions in real time with nba optical tracking data,” in *Proceedings of the 8th MIT Sloan Sports Analytics Conference*, Boston, MA, USA, vol. 28, no. 3, 2014.

[10] D. Cervone, A. D’Amour, L. Bornn, and K. Goldsberry, “A multiresolution stochastic process model for predicting basketball possession outcomes,” *Journal of the American Statistical Association*, vol. 111, no. 514, pp. 585–599, 2016.

[11] J. Fernández, L. Bornn, and D. Cervone, “Decomposing the immeasurable sport: A deep learning expected possession value framework for soccer,” in *13th MIT Sloan Sports Analytics Conference*, 2019.

[12] P. Garnier and T. Gregoir, “Evaluating soccer player: From live camera to deep reinforcement learning,” *arXiv preprint arXiv:2101.05388*, 2021.

[13] G. Liu, Y. Luo, O. Schulte, and T. Kharrat, “Deep soccer analytics: learning an action-value function for evaluating soccer players,” *Data Mining and Knowledge Discovery*, vol. 34, pp. 1531–1559, 2020.

[14] D. Dinsdale and J. Gallagher, “Transfer portal: Accurately forecasting the impact of a player transfer in soccer,” *arXiv preprint arXiv:2201.11533*, 2022.

[15] A. E. Elo, “The proposed uscf rating system, its development, theory, and applications,” *Chess Life*, vol. 22, no. 8, pp. 242–247, 1967.

[16] P. A. Samuelson, “A note on the pure theory of consumer’s behaviour,” *Economica*, vol. 5, no. 17, pp. 61–71, 1938.

[17] A. Shelopugin and A. Sirotkin, “Evaluating of football player 1v1 abilities based on the glicko-2 with modifications,” in *2023 IEEE 28th Pacific Rim International Symposium on Dependable Computing (PRDC)*. IEEE, 2023, pp. 287–291.

[18] G. Ke, Q. Meng, T. Finley, T. Wang, W. Chen, W. Ma, Q. Ye, and T.-Y. Liu, “Lightgbm: A highly efficient gradient boosting decision tree,” *Advances in neural information processing systems*, vol. 30, 2017.

[19] L. Prokhorenkova, G. Gusev, A. Vorobev, A. V. Dorogush, and A. Gulin, “Catboost: unbiased boosting with categorical features,” *Advances in neural information processing systems*, vol. 31, 2018.

[20] G. Gelade, “A new metric for evaluating 1v1 ability.” [Online]. Available: [https://www.statsperform.com/resource/a-new-metric-for-evaluating-1v1-ability](https://www.statsperform.com/resource/a-new-metric-for-evaluating-1v1-ability)

[21] R. A. Bradley and M. E. Terry, “Rank analysis of incomplete block designs: I. the method of paired comparisons,” *Biometrika*, vol. 39, no. 3/4, pp. 324–345, 1952.

[22] M. E. Glickman, “Example of the glicko-2 system,” *Boston University*, vol. 28, 2012.

[23] A. Shelopugin and A. Sirotkin, “Ratings of european and south american football leagues based on glicko-2 with modifications,” *arXiv preprint arXiv:2310.11459*, 2023.

[24] S. Leone, “Ea sports fc 24 complete player dataset,” https://www.kaggle.com/datasets/stefanoleone992/ea-sports-fc-24-complete-2024, accessed: 2024-05-30.

[25] G. Ward, T. Hastie, S. Barry, J. Elith, and J. R. Leathwick, “Presence-only data and the em algorithm,” *Biometrics*, vol. 65, no. 2, pp. 554–563, 2009.

# Appendix

## A A Case Study: Gianluigi Donnarumma Passes to Duels in Seasons 19/20, 20/21

In this paper, we focus on solving the selection problem. The main idea of the work is to suggest a mechanism to narrow down the selection pool. However, our approach can find applications not only in selection tasks but also in team performance and opposition analysis. Below, we provide an example of such an analysis.

Gianluigi Donnarumma was a great goalkeeper in terms of shot-stopping during his career at AC Milan. However, he had some struggles with his passes. Coach Stefano Pioli suggested the following solution: if Donnarumma was under pressure, he would make a long forward pass (usually leading to a duel); otherwise, he would make a short pass to a defender or defensive midfielder.

By long forward pass, we define a pass with a distance of more than 40 metres and an increment of y-projection of more than 10 metres. The most popular target players for Donnarumma were Zlatan Ibrahimovic (35 aerial duels, 8 ground duels) and Rafael Leao (28 aerial duels, 11 ground duels).

The traditional EPV approach does not take into account players' duel skills; however, our approach provides a more detailed analysis. In Table 3, we compare passes to Leao and Ibrahimovic. The column *saved* represents the proportion of possessions that were saved after duels. The column *apriori* represents the probability of winning a duel without considering a player's skills, and the column *win_duel* represents the probability of winning a duel while accounting for duel skills. As we can see, Leao and Ibrahimovic were in similar situations in terms of the difficulty of winning duels. However, despite facing slightly stronger opponents (as indicated by *opp_rating*), Zlatan managed to convert these situations into winning scenarios: his average probability of winning was 61.1%, compared to Leao's 35.8% in aerial duels. As a result, Ibrahimovic's $EPV$ (column $epv\_ind\_duel$) values are higher than Leao's. Take note that the Glicko ratings have a property to update over time; therefore, the column *rating* represents the average ratings of Ibrahimovic and Leao against their opponents after Donnarumma's passes.

Table 3: Gianluigi Donnarumma Passes to Rafael Leao or Zlatan Ibrahimovic Duels

<table>
  <thead>
    <tr>
        <th>player</th>
        <th>duel</th>
        <th>duels</th>
        <th>saved</th>
        <th>apriori</th>
        <th>win_duel</th>
        <th>rating</th>
        <th>opp_rating</th>
        <th>duel_epv</th>
        <th>epv_ind_duel</th>
    </tr>
  </thead>
  <tbody>
    <tr>
        <td>Leão</td>
        <td>aerial</td>
        <td>35</td>
        <td>28.6</td>
        <td>40.5</td>
        <td>35.8</td>
        <td>1519</td>
        <td>1545</td>
        <td>0.00075</td>
        <td>0.00069</td>
    </tr>
    <tr>
        <td>Leão</td>
        <td>ground</td>
        <td>8</td>
        <td>12.5</td>
        <td>45.0</td>
        <td>28.2</td>
        <td>1408</td>
        <td>1534</td>
        <td>0.00073</td>
        <td>0.00062</td>
    </tr>
    <tr>
        <td>Zlatan</td>
        <td>aerial</td>
        <td>28</td>
        <td>53.6</td>
        <td>39.2</td>
        <td>61.1</td>
        <td>1746</td>
        <td>1573</td>
        <td>0.00077</td>
        <td>0.00135</td>
    </tr>
    <tr>
        <td>Zlatan</td>
        <td>ground</td>
        <td>11</td>
        <td>54.5</td>
        <td>44.8</td>
        <td>44.9</td>
        <td>1539</td>
        <td>1533</td>
        <td>-0.00012</td>
        <td>-0.0001</td>
    </tr>
  </tbody>
</table>

Table 4: Aerial Duel Ratings

<table>
  <thead>
    <tr>
        <th>#</th>
        <th>player</th>
        <th>position</th>
        <th>rating</th>
        <th>#duels</th>
        <th>wins%</th>
    </tr>
  </thead>
  <tbody>
    <tr>
        <td>1</td>
        <td>V. van Dijk</td>
        <td>central_def</td>
        <td>1762</td>
        <td>2167</td>
        <td>71.9</td>
    </tr>
    <tr>
        <td>2</td>
        <td>L. Sobiech</td>
        <td>central_def</td>
        <td>1761</td>
        <td>1239</td>
        <td>72.2</td>
    </tr>
    <tr>
        <td>3</td>
        <td>B. Matić</td>
        <td>midfielder</td>
        <td>1757</td>
        <td>1076</td>
        <td>71.1</td>
    </tr>
    <tr>
        <td>4</td>
        <td>M. Djurić</td>
        <td>central_forward</td>
        <td>1744</td>
        <td>2763</td>
        <td>67.7</td>
    </tr>
    <tr>
        <td>5</td>
        <td>L. Došek</td>
        <td>central_forward</td>
        <td>1743</td>
        <td>253</td>
        <td>66.0</td>
    </tr>
    <tr>
        <td>6</td>
        <td>S. Coates</td>
        <td>central_def</td>
        <td>1742</td>
        <td>1729</td>
        <td>69.3</td>
    </tr>
    <tr>
        <td>7</td>
        <td>Rafael Donato</td>
        <td>central_def</td>
        <td>1742</td>
        <td>868</td>
        <td>71.5</td>
    </tr>
    <tr>
        <td>8</td>
        <td>P. Budkivskyi</td>
        <td>central_forward</td>
        <td>1738</td>
        <td>1238</td>
        <td>56.2</td>
    </tr>
    <tr>
        <td>9</td>
        <td>A. Papazoglou</td>
        <td>wing</td>
        <td>1734</td>
        <td>1137</td>
        <td>61.3</td>
    </tr>
    <tr>
        <td>10</td>
        <td>M. Jakubko</td>
        <td>central_forward</td>
        <td>1726</td>
        <td>239</td>
        <td>63.6</td>
    </tr>
    <tr>
        <td>11</td>
        <td>S. Memišević</td>
        <td>central_def</td>
        <td>1725</td>
        <td>762</td>
        <td>67.2</td>
    </tr>
    <tr>
        <td>12</td>
        <td>N. Bendtner</td>
        <td>central_forward</td>
        <td>1725</td>
        <td>742</td>
        <td>52.4</td>
    </tr>
    <tr>
        <td>13</td>
        <td>S. Ganvoula</td>
        <td>central_forward</td>
        <td>1722</td>
        <td>1364</td>
        <td>58.7</td>
    </tr>
    <tr>
        <td>14</td>
        <td>J. Tarkowski</td>
        <td>central_def</td>
        <td>1721</td>
        <td>2097</td>
        <td>66.3</td>
    </tr>
    <tr>
        <td>15</td>
        <td>R. Uldriķis</td>
        <td>central_forward</td>
        <td>1721</td>
        <td>1615</td>
        <td>56.6</td>
    </tr>
    <tr>
        <td>16</td>
        <td>C. Bolger</td>
        <td>central_def</td>
        <td>1715</td>
        <td>670</td>
        <td>74.0</td>
    </tr>
    <tr>
        <td>17</td>
        <td>F. Benković</td>
        <td>central_def</td>
        <td>1715</td>
        <td>690</td>
        <td>77.2</td>
    </tr>
    <tr>
        <td>18</td>
        <td>A. Flint</td>
        <td>central_def</td>
        <td>1713</td>
        <td>2676</td>
        <td>68.8</td>
    </tr>
    <tr>
        <td>19</td>
        <td>R. Rodelin</td>
        <td>wing</td>
        <td>1713</td>
        <td>1461</td>
        <td>65.6</td>
    </tr>
    <tr>
        <td>20</td>
        <td>K. Moore</td>
        <td>central_forward</td>
        <td>1713</td>
        <td>2478</td>
        <td>58.2</td>
    </tr>
    <tr>
        <td>21</td>
        <td>C. Keeley</td>
        <td>central_def</td>
        <td>1712</td>
        <td>687</td>
        <td>72.6</td>
    </tr>
    <tr>
        <td>22</td>
        <td>J. Greaves</td>
        <td>lateral</td>
        <td>1712</td>
        <td>774</td>
        <td>71.2</td>
    </tr>
    <tr>
        <td>23</td>
        <td>J. Cooper</td>
        <td>central_def</td>
        <td>1712</td>
        <td>2915</td>
        <td>69.6</td>
    </tr>
    <tr>
        <td>24</td>
        <td>R. Leeuwin</td>
        <td>central_def</td>
        <td>1711</td>
        <td>754</td>
        <td>67.8</td>
    </tr>
    <tr>
        <td>25</td>
        <td>L. Lindsay</td>
        <td>central_def</td>
        <td>1711</td>
        <td>1732</td>
        <td>68.7</td>
    </tr>
    <tr>
        <td>26</td>
        <td>E. Sipović</td>
        <td>central_def</td>
        <td>1711</td>
        <td>266</td>
        <td>76.3</td>
    </tr>
    <tr>
        <td>27</td>
        <td>M. Papadopulos</td>
        <td>central_forward</td>
        <td>1710</td>
        <td>1807</td>
        <td>55.2</td>
    </tr>
    <tr>
        <td>28</td>
        <td>J. Owens</td>
        <td>central_forward</td>
        <td>1709</td>
        <td>1590</td>
        <td>59.7</td>
    </tr>
    <tr>
        <td>29</td>
        <td>C. Casadei</td>
        <td>wing</td>
        <td>1708</td>
        <td>106</td>
        <td>70.8</td>
    </tr>
    <tr>
        <td>30</td>
        <td>V. Posmac</td>
        <td>central_def</td>
        <td>1707</td>
        <td>870</td>
        <td>77.2</td>
    </tr>
    <tr>
        <td>31</td>
        <td>J. Arango</td>
        <td>wing</td>
        <td>1706</td>
        <td>165</td>
        <td>64.2</td>
    </tr>
    <tr>
        <td>32</td>
        <td>F. Mayembo</td>
        <td>central_def</td>
        <td>1705</td>
        <td>446</td>
        <td>69.7</td>
    </tr>
    <tr>
        <td>33</td>
        <td>R. Sykes</td>
        <td>central_def</td>
        <td>1705</td>
        <td>195</td>
        <td>73.3</td>
    </tr>
    <tr>
        <td>34</td>
        <td>V. Koskimaa</td>
        <td>central_def</td>
        <td>1704</td>
        <td>596</td>
        <td>74.8</td>
    </tr>
    <tr>
        <td>35</td>
        <td>Y. Khacheridi</td>
        <td>central_def</td>
        <td>1704</td>
        <td>532</td>
        <td>76.3</td>
    </tr>
    <tr>
        <td>36</td>
        <td>N. Ziabaris</td>
        <td>central_def</td>
        <td>1704</td>
        <td>262</td>
        <td>72.9</td>
    </tr>
    <tr>
        <td>37</td>
        <td>I. Soumaré</td>
        <td>wing</td>
        <td>1703</td>
        <td>684</td>
        <td>61.3</td>
    </tr>
    <tr>
        <td>38</td>
        <td>C. Stewart</td>
        <td>central_def</td>
        <td>1702</td>
        <td>458</td>
        <td>63.3</td>
    </tr>
    <tr>
        <td>39</td>
        <td>J. Chabot</td>
        <td>central_def</td>
        <td>1702</td>
        <td>808</td>
        <td>63.2</td>
    </tr>
    <tr>
        <td>40</td>
        <td>C. Bocşan</td>
        <td>central_def</td>
        <td>1700</td>
        <td>235</td>
        <td>71.1</td>
    </tr>
    <tr>
        <td>41</td>
        <td>J. Blayac</td>
        <td>central_forward</td>
        <td>1700</td>
        <td>720</td>
        <td>59.9</td>
    </tr>
    <tr>
        <td>42</td>
        <td>Héliton</td>
        <td>central_def</td>
        <td>1700</td>
        <td>360</td>
        <td>70.8</td>
    </tr>
    <tr>
        <td>43</td>
        <td>Nilton</td>
        <td>midfielder</td>
        <td>1699</td>
        <td>356</td>
        <td>64.0</td>
    </tr>
    <tr>
        <td>44</td>
        <td>M. Janko</td>
        <td>central_forward</td>
        <td>1699</td>
        <td>647</td>
        <td>59.7</td>
    </tr>
    <tr>
        <td>45</td>
        <td>M. Baša</td>
        <td>central_def</td>
        <td>1699</td>
        <td>242</td>
        <td>76.9</td>
    </tr>
    <tr>
        <td>46</td>
        <td>D. Lemajić</td>
        <td>central_forward</td>
        <td>1699</td>
        <td>1420</td>
        <td>54.7</td>
    </tr>
    <tr>
        <td>47</td>
        <td>Kim Min-Jae</td>
        <td>central_def</td>
        <td>1699</td>
        <td>485</td>
        <td>67.2</td>
    </tr>
    <tr>
        <td>48</td>
        <td>A. Taylor</td>
        <td>central_def</td>
        <td>1699</td>
        <td>1353</td>
        <td>71.2</td>
    </tr>
    <tr>
        <td>49</td>
        <td>M. Beevers</td>
        <td>central_def</td>
        <td>1698</td>
        <td>787</td>
        <td>67.0</td>
    </tr>
    <tr>
        <td>50</td>
        <td>A. Zabolotny</td>
        <td>central_forward</td>
        <td>1698</td>
        <td>1572</td>
        <td>57.8</td>
    </tr>
  </tbody>
</table>

Table 5: Ground Duel Ratings

<table>
  <thead>
    <tr>
        <th>#</th>
        <th>player</th>
        <th>position</th>
        <th>rating</th>
        <th>#duels</th>
        <th>wins%</th>
    </tr>
  </thead>
  <tbody>
    <tr>
        <td>1</td>
        <td>B. Ostojić</td>
        <td>central_def</td>
        <td>1695</td>
        <td>279</td>
        <td>73.1</td>
    </tr>
    <tr>
        <td>2</td>
        <td>Felipe</td>
        <td>central_def</td>
        <td>1672</td>
        <td>643</td>
        <td>67.8</td>
    </tr>
    <tr>
        <td>3</td>
        <td>Vasco Fernandes</td>
        <td>central_def</td>
        <td>1671</td>
        <td>408</td>
        <td>66.4</td>
    </tr>
    <tr>
        <td>4</td>
        <td>V. van Dijk</td>
        <td>central_def</td>
        <td>1669</td>
        <td>742</td>
        <td>71.6</td>
    </tr>
    <tr>
        <td>5</td>
        <td>R. García</td>
        <td>central_def</td>
        <td>1666</td>
        <td>427</td>
        <td>68.1</td>
    </tr>
    <tr>
        <td>6</td>
        <td>L. Valenti</td>
        <td>central_def</td>
        <td>1666</td>
        <td>135</td>
        <td>74.1</td>
    </tr>
    <tr>
        <td>7</td>
        <td>V. Mudrac</td>
        <td>central_def</td>
        <td>1665</td>
        <td>274</td>
        <td>69.3</td>
    </tr>
    <tr>
        <td>8</td>
        <td>M. Vítor</td>
        <td>central_def</td>
        <td>1663</td>
        <td>691</td>
        <td>68.7</td>
    </tr>
    <tr>
        <td>9</td>
        <td>Raúl Navas</td>
        <td>central_def</td>
        <td>1663</td>
        <td>350</td>
        <td>70.9</td>
    </tr>
    <tr>
        <td>10</td>
        <td>N. Elvedi</td>
        <td>central_def</td>
        <td>1660</td>
        <td>735</td>
        <td>66.9</td>
    </tr>
    <tr>
        <td>11</td>
        <td>L. McCullough</td>
        <td>central_def</td>
        <td>1658</td>
        <td>317</td>
        <td>75.7</td>
    </tr>
    <tr>
        <td>12</td>
        <td>G. Valsvik</td>
        <td>central_def</td>
        <td>1656</td>
        <td>702</td>
        <td>64.2</td>
    </tr>
    <tr>
        <td>13</td>
        <td>P. Jagielka</td>
        <td>central_def</td>
        <td>1656</td>
        <td>388</td>
        <td>69.8</td>
    </tr>
    <tr>
        <td>14</td>
        <td>J. Matip</td>
        <td>central_def</td>
        <td>1655</td>
        <td>543</td>
        <td>69.2</td>
    </tr>
    <tr>
        <td>15</td>
        <td>A. Maxsø</td>
        <td>central_def</td>
        <td>1655</td>
        <td>552</td>
        <td>65.4</td>
    </tr>
    <tr>
        <td>16</td>
        <td>A. Flint</td>
        <td>central_def</td>
        <td>1655</td>
        <td>917</td>
        <td>68.5</td>
    </tr>
    <tr>
        <td>17</td>
        <td>J. Kucka</td>
        <td>midfielder</td>
        <td>1654</td>
        <td>877</td>
        <td>55.6</td>
    </tr>
    <tr>
        <td>18</td>
        <td>Marcão</td>
        <td>central_def</td>
        <td>1654</td>
        <td>427</td>
        <td>65.6</td>
    </tr>
    <tr>
        <td>19</td>
        <td>A. Bengtsson</td>
        <td>lateral</td>
        <td>1654</td>
        <td>118</td>
        <td>72.0</td>
    </tr>
    <tr>
        <td>20</td>
        <td>Rodrigão</td>
        <td>central_def</td>
        <td>1654</td>
        <td>293</td>
        <td>68.9</td>
    </tr>
    <tr>
        <td>21</td>
        <td>Aleksandar Luković</td>
        <td>central_def</td>
        <td>1653</td>
        <td>120</td>
        <td>76.7</td>
    </tr>
    <tr>
        <td>22</td>
        <td>T. Petrášek</td>
        <td>central_def</td>
        <td>1653</td>
        <td>254</td>
        <td>72.0</td>
    </tr>
    <tr>
        <td>23</td>
        <td>C. McJannet</td>
        <td>central_def</td>
        <td>1653</td>
        <td>445</td>
        <td>67.2</td>
    </tr>
    <tr>
        <td>24</td>
        <td>N. Belaković</td>
        <td>wing</td>
        <td>1652</td>
        <td>351</td>
        <td>61.5</td>
    </tr>
    <tr>
        <td>25</td>
        <td>Gabriel Magalhaes</td>
        <td>central_def</td>
        <td>1651</td>
        <td>566</td>
        <td>67.3</td>
    </tr>
    <tr>
        <td>26</td>
        <td>M. Hasebe</td>
        <td>midfielder</td>
        <td>1651</td>
        <td>572</td>
        <td>63.5</td>
    </tr>
    <tr>
        <td>27</td>
        <td>M. Peersman</td>
        <td>lateral</td>
        <td>1651</td>
        <td>817</td>
        <td>62.5</td>
    </tr>
    <tr>
        <td>28</td>
        <td>G. Mancini</td>
        <td>central_def</td>
        <td>1650</td>
        <td>759</td>
        <td>63.9</td>
    </tr>
    <tr>
        <td>29</td>
        <td>O. Bačo</td>
        <td>central_def</td>
        <td>1648</td>
        <td>478</td>
        <td>61.3</td>
    </tr>
    <tr>
        <td>30</td>
        <td>H. Moukoudi</td>
        <td>central_def</td>
        <td>1647</td>
        <td>408</td>
        <td>61.3</td>
    </tr>
    <tr>
        <td>31</td>
        <td>L. Querfeld</td>
        <td>central_def</td>
        <td>1647</td>
        <td>330</td>
        <td>67.9</td>
    </tr>
    <tr>
        <td>32</td>
        <td>C. Halkett</td>
        <td>central_def</td>
        <td>1646</td>
        <td>374</td>
        <td>68.2</td>
    </tr>
    <tr>
        <td>33</td>
        <td>W. Pacho</td>
        <td>central_def</td>
        <td>1646</td>
        <td>269</td>
        <td>66.2</td>
    </tr>
    <tr>
        <td>34</td>
        <td>S. Bell</td>
        <td>central_def</td>
        <td>1646</td>
        <td>543</td>
        <td>61.7</td>
    </tr>
    <tr>
        <td>35</td>
        <td>Luiz Otávio</td>
        <td>central_def</td>
        <td>1645</td>
        <td>344</td>
        <td>68.0</td>
    </tr>
    <tr>
        <td>36</td>
        <td>B. Utvik</td>
        <td>central_def</td>
        <td>1644</td>
        <td>398</td>
        <td>64.3</td>
    </tr>
    <tr>
        <td>37</td>
        <td>S. Papagiannopoulos</td>
        <td>central_def</td>
        <td>1644</td>
        <td>716</td>
        <td>70.4</td>
    </tr>
    <tr>
        <td>38</td>
        <td>L. Nielsen</td>
        <td>central_def</td>
        <td>1644</td>
        <td>708</td>
        <td>65.5</td>
    </tr>
    <tr>
        <td>39</td>
        <td>R. Grodzicki</td>
        <td>central_def</td>
        <td>1644</td>
        <td>171</td>
        <td>70.2</td>
    </tr>
    <tr>
        <td>40</td>
        <td>A. Sørensen</td>
        <td>central_def</td>
        <td>1644</td>
        <td>589</td>
        <td>65.9</td>
    </tr>
    <tr>
        <td>41</td>
        <td>C. Goldson</td>
        <td>central_def</td>
        <td>1644</td>
        <td>820</td>
        <td>72.8</td>
    </tr>
    <tr>
        <td>42</td>
        <td>F. Aguilar</td>
        <td>central_def</td>
        <td>1644</td>
        <td>117</td>
        <td>72.6</td>
    </tr>
    <tr>
        <td>43</td>
        <td>O. Kúdela</td>
        <td>central_def</td>
        <td>1644</td>
        <td>465</td>
        <td>66.2</td>
    </tr>
    <tr>
        <td>44</td>
        <td>J. Boller</td>
        <td>central_def</td>
        <td>1643</td>
        <td>280</td>
        <td>68.2</td>
    </tr>
    <tr>
        <td>45</td>
        <td>M. Lovato</td>
        <td>central_def</td>
        <td>1643</td>
        <td>267</td>
        <td>69.7</td>
    </tr>
    <tr>
        <td>46</td>
        <td>L. Morgillo</td>
        <td>central_def</td>
        <td>1643</td>
        <td>130</td>
        <td>73.1</td>
    </tr>
    <tr>
        <td>47</td>
        <td>Tiago Ilori</td>
        <td>central_def</td>
        <td>1643</td>
        <td>200</td>
        <td>67.5</td>
    </tr>
    <tr>
        <td>48</td>
        <td>M. Connolly</td>
        <td>central_def</td>
        <td>1642</td>
        <td>380</td>
        <td>69.7</td>
    </tr>
    <tr>
        <td>49</td>
        <td>T. Vion</td>
        <td>lateral</td>
        <td>1642</td>
        <td>419</td>
        <td>59.4</td>
    </tr>
    <tr>
        <td>50</td>
        <td>A. Seck</td>
        <td>central_def</td>
        <td>1642</td>
        <td>444</td>
        <td>68.7</td>
    </tr>
  </tbody>
</table>

Table 6: Top Season Performances Based On PCR

<table>
  <thead>
    <tr>
        <th>#</th>
        <th>player</th>
        <th>team</th>
        <th>competition</th>
        <th>season</th>
        <th>PCR</th>
        <th>eff_time</th>
    </tr>
  </thead>
  <tbody>
    <tr>
        <td>1</td>
        <td>H. Ziyech</td>
        <td>Ajax</td>
        <td>Netherlands. First</td>
        <td>2019/2020</td>
        <td>0.355</td>
        <td>1237</td>
    </tr>
    <tr>
        <td>2</td>
        <td>J. Iličić</td>
        <td>Atalanta</td>
        <td>Italy. First</td>
        <td>2020/2021</td>
        <td>0.315</td>
        <td>1143</td>
    </tr>
    <tr>
        <td>3</td>
        <td>A. Zoubir</td>
        <td>Qarabag</td>
        <td>Azerbaijan. First</td>
        <td>2020/2021</td>
        <td>0.314</td>
        <td>1465</td>
    </tr>
    <tr>
        <td>4</td>
        <td>H. Ziyech</td>
        <td>Ajax</td>
        <td>Netherlands. First</td>
        <td>2017/2018</td>
        <td>0.308</td>
        <td>2257</td>
    </tr>
    <tr>
        <td>5</td>
        <td>O. Dembélé</td>
        <td>Barcelona</td>
        <td>Spain. First</td>
        <td>2021/2022</td>
        <td>0.305</td>
        <td>1008</td>
    </tr>
    <tr>
        <td>6</td>
        <td>J. Hauge</td>
        <td>Bodo Glimt</td>
        <td>Norway. First</td>
        <td>2020</td>
        <td>0.305</td>
        <td>1149</td>
    </tr>
    <tr>
        <td>7</td>
        <td>J. Grealish</td>
        <td>Aston Villa</td>
        <td>England. First</td>
        <td>2020/2021</td>
        <td>0.301</td>
        <td>1481</td>
    </tr>
    <tr>
        <td>8</td>
        <td>J. Doku</td>
        <td>Manchester City</td>
        <td>England. First</td>
        <td>2023/2024</td>
        <td>0.299</td>
        <td>1207</td>
    </tr>
    <tr>
        <td>9</td>
        <td>Malcom</td>
        <td>Zenit</td>
        <td>Russia. First</td>
        <td>2020/2021</td>
        <td>0.297</td>
        <td>1184</td>
    </tr>
    <tr>
        <td>10</td>
        <td>A. Zoubir</td>
        <td>Qarabag</td>
        <td>Azerbaijan. First</td>
        <td>2023/2024</td>
        <td>0.287</td>
        <td>1134</td>
    </tr>
    <tr>
        <td>11</td>
        <td>A. Schjelderup</td>
        <td>Nordsjaelland</td>
        <td>Denmark. First</td>
        <td>2023/2024</td>
        <td>0.283</td>
        <td>1207</td>
    </tr>
    <tr>
        <td>12</td>
        <td>L. Messi</td>
        <td>Barcelona</td>
        <td>Spain. First</td>
        <td>2018/2019</td>
        <td>0.277</td>
        <td>2058</td>
    </tr>
    <tr>
        <td>13</td>
        <td>T. Ali</td>
        <td>Malmo FF</td>
        <td>Sweden. First</td>
        <td>2023</td>
        <td>0.277</td>
        <td>1258</td>
    </tr>
    <tr>
        <td>14</td>
        <td>K. Mbappé</td>
        <td>PSG</td>
        <td>France. First</td>
        <td>2019/2020</td>
        <td>0.276</td>
        <td>1146</td>
    </tr>
    <tr>
        <td>15</td>
        <td>J. Ito</td>
        <td>Genk</td>
        <td>Belgium. First</td>
        <td>2021/2022</td>
        <td>0.273</td>
        <td>2286</td>
    </tr>
    <tr>
        <td>16</td>
        <td>Neymar</td>
        <td>PSG</td>
        <td>France. First</td>
        <td>2017/2018</td>
        <td>0.272</td>
        <td>1353</td>
    </tr>
    <tr>
        <td>17</td>
        <td>L. Abada</td>
        <td>Celtic</td>
        <td>Scotland. First</td>
        <td>2022/2023</td>
        <td>0.271</td>
        <td>1078</td>
    </tr>
    <tr>
        <td>18</td>
        <td>Y. Soteldo</td>
        <td>Santos</td>
        <td>Brazil. First</td>
        <td>2023</td>
        <td>0.27</td>
        <td>1105</td>
    </tr>
    <tr>
        <td>19</td>
        <td>Musa Al Tamari</td>
        <td>APOEL</td>
        <td>Cyprus. First</td>
        <td>2018/2019</td>
        <td>0.27</td>
        <td>1387</td>
    </tr>
    <tr>
        <td>20</td>
        <td>J. Clarke</td>
        <td>Sunderland</td>
        <td>England. Second</td>
        <td>2023/2024</td>
        <td>0.268</td>
        <td>2503</td>
    </tr>
    <tr>
        <td>21</td>
        <td>R. Mahrez</td>
        <td>Manchester City</td>
        <td>England. First</td>
        <td>2019/2020</td>
        <td>0.268</td>
        <td>1506</td>
    </tr>
    <tr>
        <td>22</td>
        <td>G. Kanga</td>
        <td>Crvena Zvezda</td>
        <td>Serbia. First</td>
        <td>2016/2017</td>
        <td>0.268</td>
        <td>1261</td>
    </tr>
    <tr>
        <td>23</td>
        <td>M. Marin</td>
        <td>Crvena Zvezda</td>
        <td>Serbia. First</td>
        <td>2018/2019</td>
        <td>0.262</td>
        <td>1145</td>
    </tr>
    <tr>
        <td>24</td>
        <td>Dodo</td>
        <td>Hamrun Spartans</td>
        <td>Malta. First</td>
        <td>2020/2021</td>
        <td>0.258</td>
        <td>1033</td>
    </tr>
    <tr>
        <td>25</td>
        <td>P. Zinckernagel</td>
        <td>Bodo Glimt</td>
        <td>Norway. First</td>
        <td>2020</td>
        <td>0.254</td>
        <td>1818</td>
    </tr>
    <tr>
        <td>26</td>
        <td>Daniel Podence</td>
        <td>Olympiacos Piraeus</td>
        <td>Greece. First</td>
        <td>2023/2024</td>
        <td>0.252</td>
        <td>1199</td>
    </tr>
    <tr>
        <td>27</td>
        <td>C. McCloskey</td>
        <td>Glenavon</td>
        <td>Northern Ireland. First</td>
        <td>2020/2021</td>
        <td>0.251</td>
        <td>1107</td>
    </tr>
    <tr>
        <td>28</td>
        <td>K. Mbappé</td>
        <td>Monaco</td>
        <td>France. First</td>
        <td>2016/2017</td>
        <td>0.25</td>
        <td>1088</td>
    </tr>
    <tr>
        <td>29</td>
        <td>J. Cooper</td>
        <td>Linfield</td>
        <td>Northern Ireland. First</td>
        <td>2019/2020</td>
        <td>0.249</td>
        <td>1463</td>
    </tr>
    <tr>
        <td>30</td>
        <td>A. Limbombe</td>
        <td>Almere City</td>
        <td>Netherlands. Second</td>
        <td>2022/2023</td>
        <td>0.248</td>
        <td>1360</td>
    </tr>
    <tr>
        <td>31</td>
        <td>I. Kallon</td>
        <td>Cambuur</td>
        <td>Netherlands. Second</td>
        <td>2020/2021</td>
        <td>0.248</td>
        <td>1530</td>
    </tr>
    <tr>
        <td>32</td>
        <td>Antony</td>
        <td>Ajax</td>
        <td>Netherlands. First</td>
        <td>2021/2022</td>
        <td>0.244</td>
        <td>1323</td>
    </tr>
    <tr>
        <td>33</td>
        <td>N. Lang</td>
        <td>Club Brugge</td>
        <td>Belgium. First</td>
        <td>2022/2023</td>
        <td>0.242</td>
        <td>1646</td>
    </tr>
    <tr>
        <td>34</td>
        <td>A. Abreu</td>
        <td>UT Petange</td>
        <td>Luxembourg. First</td>
        <td>2022/2023</td>
        <td>0.241</td>
        <td>1437</td>
    </tr>
    <tr>
        <td>35</td>
        <td>D. Payet</td>
        <td>Olympique Marseille</td>
        <td>France. First</td>
        <td>2017/2018</td>
        <td>0.241</td>
        <td>1660</td>
    </tr>
    <tr>
        <td>36</td>
        <td>Ângelo Gabriel</td>
        <td>Santos</td>
        <td>Brazil. First</td>
        <td>2022</td>
        <td>0.24</td>
        <td>1085</td>
    </tr>
    <tr>
        <td>37</td>
        <td>H. Ziyech</td>
        <td>Ajax</td>
        <td>Netherlands. First</td>
        <td>2018/2019</td>
        <td>0.24</td>
        <td>1794</td>
    </tr>
    <tr>
        <td>38</td>
        <td>J. Croux</td>
        <td>Roda JC</td>
        <td>Netherlands. Second</td>
        <td>2019/2020</td>
        <td>0.236</td>
        <td>1856</td>
    </tr>
    <tr>
        <td>39</td>
        <td>L. Messi</td>
        <td>Barcelona</td>
        <td>Spain. First</td>
        <td>2015/2016</td>
        <td>0.236</td>
        <td>2025</td>
    </tr>
    <tr>
        <td>40</td>
        <td>Á. Di María</td>
        <td>PSG</td>
        <td>France. First</td>
        <td>2020/2021</td>
        <td>0.235</td>
        <td>1372</td>
    </tr>
    <tr>
        <td>41</td>
        <td>Marquinhos</td>
        <td>Ferencvaros</td>
        <td>Hungary. First</td>
        <td>2023/2024</td>
        <td>0.234</td>
        <td>1391</td>
    </tr>
    <tr>
        <td>42</td>
        <td>K. De Bruyne</td>
        <td>Manchester City</td>
        <td>England. First</td>
        <td>2019/2020</td>
        <td>0.234</td>
        <td>2133</td>
    </tr>
    <tr>
        <td>43</td>
        <td>Francisco Conceição</td>
        <td>Porto</td>
        <td>Portugal. First</td>
        <td>2023/2024</td>
        <td>0.233</td>
        <td>1133</td>
    </tr>
    <tr>
        <td>44</td>
        <td>V. Birmančević</td>
        <td>Cukaricki</td>
        <td>Serbia. First</td>
        <td>2020/2021</td>
        <td>0.233</td>
        <td>1217</td>
    </tr>
    <tr>
        <td>45</td>
        <td>A. Gómez</td>
        <td>Atalanta</td>
        <td>Italy. First</td>
        <td>2019/2020</td>
        <td>0.233</td>
        <td>2095</td>
    </tr>
    <tr>
        <td>46</td>
        <td>Neymar</td>
        <td>PSG</td>
        <td>France. First</td>
        <td>2020/2021</td>
        <td>0.233</td>
        <td>1051</td>
    </tr>
    <tr>
        <td>47</td>
        <td>M. Melikson</td>
        <td>Hapoel Be er Sheva</td>
        <td>Israel. First</td>
        <td>2016/2017</td>
        <td>0.232</td>
        <td>1341</td>
    </tr>
    <tr>
        <td>48</td>
        <td>Cesc Fàbregas</td>
        <td>Chelsea</td>
        <td>England. First</td>
        <td>2016/2017</td>
        <td>0.231</td>
        <td>1006</td>
    </tr>
    <tr>
        <td>49</td>
        <td>M. Tomasov</td>
        <td>Astana</td>
        <td>Kazakhstan. First</td>
        <td>2023</td>
        <td>0.231</td>
        <td>1347</td>
    </tr>
    <tr>
        <td>50</td>
        <td>N. Lang</td>
        <td>Club Brugge</td>
        <td>Belgium. First</td>
        <td>2021/2022</td>
        <td>0.231</td>
        <td>2052</td>
    </tr>
  </tbody>
</table>

Table 7: Player Ranking Based On PCR Predictions In The Case Of Their Transfer To Manchester City

<table>
  <thead>
    <tr>
        <th>#</th>
        <th>player</th>
        <th>team</th>
        <th>age</th>
        <th>PCR</th>
        <th>PCR_pred</th>
        <th>PCR_adj</th>
        <th>stay_proba</th>
    </tr>
  </thead>
  <tbody>
    <tr>
        <td>1</td>
        <td>J. Doku</td>
        <td>Manchester City</td>
        <td>21.2</td>
        <td>0.299</td>
        <td>0.197</td>
        <td>0.196</td>
        <td>0.961</td>
    </tr>
    <tr>
        <td>2</td>
        <td>O. Dembélé</td>
        <td>PSG</td>
        <td>26.3</td>
        <td>0.215</td>
        <td>0.173</td>
        <td>0.166</td>
        <td>0.868</td>
    </tr>
    <tr>
        <td>3</td>
        <td>Sávio</td>
        <td>Girona</td>
        <td>19.4</td>
        <td>0.174</td>
        <td>0.165</td>
        <td>0.162</td>
        <td>0.96</td>
    </tr>
    <tr>
        <td>4</td>
        <td>Nico Williams</td>
        <td>Athletic Bilbao</td>
        <td>21.1</td>
        <td>0.199</td>
        <td>0.157</td>
        <td>0.154</td>
        <td>0.956</td>
    </tr>
    <tr>
        <td>5</td>
        <td>Bryan Zaragoza</td>
        <td>Bayern Munchen</td>
        <td>21.9</td>
        <td>0.162</td>
        <td>0.154</td>
        <td>0.151</td>
        <td>0.961</td>
    </tr>
    <tr>
        <td>6</td>
        <td>K. Kvaratskhelia</td>
        <td>Napoli</td>
        <td>22.5</td>
        <td>0.195</td>
        <td>0.153</td>
        <td>0.15</td>
        <td>0.954</td>
    </tr>
    <tr>
        <td>7</td>
        <td>Francisco Conceicão</td>
        <td>Porto</td>
        <td>20.7</td>
        <td>0.233</td>
        <td>0.156</td>
        <td>0.149</td>
        <td>0.905</td>
    </tr>
    <tr>
        <td>8</td>
        <td>M. Olise</td>
        <td>Crystal Palace</td>
        <td>21.7</td>
        <td>0.182</td>
        <td>0.152</td>
        <td>0.148</td>
        <td>0.862</td>
    </tr>
    <tr>
        <td>9</td>
        <td>O. Sahraoui</td>
        <td>Heerenveen</td>
        <td>22.2</td>
        <td>0.151</td>
        <td>0.155</td>
        <td>0.146</td>
        <td>0.882</td>
    </tr>
    <tr>
        <td>10</td>
        <td>Vinícius Júnior</td>
        <td>Real Madrid</td>
        <td>23.1</td>
        <td>0.136</td>
        <td>0.148</td>
        <td>0.144</td>
        <td>0.938</td>
    </tr>
    <tr>
        <td>11</td>
        <td>K. Coman</td>
        <td>Bayern Munchen</td>
        <td>27.2</td>
        <td>0.095</td>
        <td>0.149</td>
        <td>0.144</td>
        <td>0.861</td>
    </tr>
    <tr>
        <td>12</td>
        <td>R. Sterling</td>
        <td>Chelsea</td>
        <td>28.7</td>
        <td>0.152</td>
        <td>0.149</td>
        <td>0.144</td>
        <td>0.828</td>
    </tr>
    <tr>
        <td>13</td>
        <td>O. Niang</td>
        <td>Riga</td>
        <td>21.4</td>
        <td>0.296</td>
        <td>0.161</td>
        <td>0.144</td>
        <td>0.734</td>
    </tr>
    <tr>
        <td>14</td>
        <td>C. Ejuke</td>
        <td>Antwerp</td>
        <td>25.6</td>
        <td>0.158</td>
        <td>0.152</td>
        <td>0.143</td>
        <td>0.848</td>
    </tr>
    <tr>
        <td>15</td>
        <td>N. Lang</td>
        <td>PSV</td>
        <td>24.2</td>
        <td>0.196</td>
        <td>0.156</td>
        <td>0.143</td>
        <td>0.74</td>
    </tr>
    <tr>
        <td>16</td>
        <td>Rafael Leão</td>
        <td>Milan</td>
        <td>24.2</td>
        <td>0.178</td>
        <td>0.145</td>
        <td>0.142</td>
        <td>0.913</td>
    </tr>
    <tr>
        <td>17</td>
        <td>J. Dompé</td>
        <td>Hamburger SV</td>
        <td>28.0</td>
        <td>0.158</td>
        <td>0.155</td>
        <td>0.141</td>
        <td>0.768</td>
    </tr>
    <tr>
        <td>18</td>
        <td>F. Chiesa</td>
        <td>Juventus</td>
        <td>25.8</td>
        <td>0.195</td>
        <td>0.144</td>
        <td>0.141</td>
        <td>0.932</td>
    </tr>
    <tr>
        <td>19</td>
        <td>M. Edwards</td>
        <td>Sporting CP</td>
        <td>24.7</td>
        <td>0.277</td>
        <td>0.149</td>
        <td>0.141</td>
        <td>0.866</td>
    </tr>
    <tr>
        <td>20</td>
        <td>T. Corbeanu</td>
        <td>Granada</td>
        <td>21.2</td>
        <td>0.152</td>
        <td>0.148</td>
        <td>0.14</td>
        <td>0.869</td>
    </tr>
    <tr>
        <td>21</td>
        <td>B. Gruda</td>
        <td>Mainz 05</td>
        <td>19.2</td>
        <td>0.163</td>
        <td>0.144</td>
        <td>0.138</td>
        <td>0.823</td>
    </tr>
    <tr>
        <td>22</td>
        <td>S. Ltaief</td>
        <td>Winterthur</td>
        <td>23.3</td>
        <td>0.114</td>
        <td>0.148</td>
        <td>0.137</td>
        <td>0.786</td>
    </tr>
    <tr>
        <td>23</td>
        <td>I. Akhomach</td>
        <td>Villarreal</td>
        <td>19.3</td>
        <td>0.094</td>
        <td>0.14</td>
        <td>0.137</td>
        <td>0.936</td>
    </tr>
    <tr>
        <td>24</td>
        <td>J. Enciso</td>
        <td>Brighton</td>
        <td>19.6</td>
        <td>0.215</td>
        <td>0.147</td>
        <td>0.137</td>
        <td>0.677</td>
    </tr>
    <tr>
        <td>25</td>
        <td>Pedro Neto</td>
        <td>Wolverhampton</td>
        <td>23.4</td>
        <td>0.131</td>
        <td>0.14</td>
        <td>0.136</td>
        <td>0.855</td>
    </tr>
    <tr>
        <td>26</td>
        <td>T. Kubo</td>
        <td>Real Sociedad</td>
        <td>22.2</td>
        <td>0.162</td>
        <td>0.139</td>
        <td>0.135</td>
        <td>0.933</td>
    </tr>
    <tr>
        <td>27</td>
        <td>T. Ali</td>
        <td>Malmo FF</td>
        <td>24.8</td>
        <td>0.277</td>
        <td>0.145</td>
        <td>0.135</td>
        <td>0.828</td>
    </tr>
    <tr>
        <td>28</td>
        <td>J. Bynoe-Gittens</td>
        <td>Borussia Dortmund</td>
        <td>19.0</td>
        <td>0.208</td>
        <td>0.139</td>
        <td>0.135</td>
        <td>0.898</td>
    </tr>
    <tr>
        <td>29</td>
        <td>J. Sancho</td>
        <td>Borussia Dortmund</td>
        <td>23.4</td>
        <td>0.224</td>
        <td>0.14</td>
        <td>0.135</td>
        <td>0.858</td>
    </tr>
    <tr>
        <td>30</td>
        <td>A. Mitriță</td>
        <td>Universitatea Craiova</td>
        <td>28.4</td>
        <td>0.195</td>
        <td>0.143</td>
        <td>0.135</td>
        <td>0.906</td>
    </tr>
    <tr>
        <td>31</td>
        <td>A. Nusa</td>
        <td>Club Brugge</td>
        <td>18.3</td>
        <td>0.143</td>
        <td>0.142</td>
        <td>0.134</td>
        <td>0.872</td>
    </tr>
    <tr>
        <td>32</td>
        <td>J. Bakayoko</td>
        <td>PSV</td>
        <td>20.3</td>
        <td>0.194</td>
        <td>0.141</td>
        <td>0.134</td>
        <td>0.928</td>
    </tr>
    <tr>
        <td>33</td>
        <td>E. Zhegrova</td>
        <td>Lille</td>
        <td>24.4</td>
        <td>0.097</td>
        <td>0.139</td>
        <td>0.134</td>
        <td>0.907</td>
    </tr>
    <tr>
        <td>34</td>
        <td>Y. Minteh</td>
        <td>Feyenoord</td>
        <td>19.1</td>
        <td>0.217</td>
        <td>0.145</td>
        <td>0.134</td>
        <td>0.79</td>
    </tr>
    <tr>
        <td>35</td>
        <td>Lamine Yamal</td>
        <td>Barcelona</td>
        <td>16.1</td>
        <td>0.136</td>
        <td>0.138</td>
        <td>0.134</td>
        <td>0.922</td>
    </tr>
    <tr>
        <td>36</td>
        <td>Haissem Hassan</td>
        <td>Sporting Gijon</td>
        <td>21.5</td>
        <td>0.125</td>
        <td>0.144</td>
        <td>0.133</td>
        <td>0.873</td>
    </tr>
    <tr>
        <td>37</td>
        <td>K. Mitoma</td>
        <td>Brighton</td>
        <td>26.2</td>
        <td>0.215</td>
        <td>0.137</td>
        <td>0.133</td>
        <td>0.856</td>
    </tr>
    <tr>
        <td>38</td>
        <td>A. Sbaï</td>
        <td>Grenoble</td>
        <td>22.8</td>
        <td>0.185</td>
        <td>0.145</td>
        <td>0.132</td>
        <td>0.818</td>
    </tr>
    <tr>
        <td>39</td>
        <td>Serginho</td>
        <td>Viborg</td>
        <td>22.5</td>
        <td>0.182</td>
        <td>0.144</td>
        <td>0.132</td>
        <td>0.719</td>
    </tr>
    <tr>
        <td>40</td>
        <td>R. Sottil</td>
        <td>Fiorentina</td>
        <td>24.2</td>
        <td>0.097</td>
        <td>0.137</td>
        <td>0.132</td>
        <td>0.84</td>
    </tr>
    <tr>
        <td>41</td>
        <td>A. Schjelderup</td>
        <td>Nordsjaelland</td>
        <td>19.2</td>
        <td>0.283</td>
        <td>0.137</td>
        <td>0.131</td>
        <td>0.936</td>
    </tr>
    <tr>
        <td>42</td>
        <td>N. Madueke</td>
        <td>Chelsea</td>
        <td>21.4</td>
        <td>0.177</td>
        <td>0.136</td>
        <td>0.131</td>
        <td>0.823</td>
    </tr>
    <tr>
        <td>43</td>
        <td>M. Fofana</td>
        <td>Lyon</td>
        <td>18.3</td>
        <td>0.169</td>
        <td>0.136</td>
        <td>0.131</td>
        <td>0.93</td>
    </tr>
    <tr>
        <td>44</td>
        <td>Milson</td>
        <td>Maccabi Tel Aviv</td>
        <td>23.9</td>
        <td>0.178</td>
        <td>0.14</td>
        <td>0.13</td>
        <td>0.856</td>
    </tr>
    <tr>
        <td>45</td>
        <td>Ângelo Gabriel</td>
        <td>Strasbourg</td>
        <td>18.6</td>
        <td>0.125</td>
        <td>0.134</td>
        <td>0.13</td>
        <td>0.918</td>
    </tr>
    <tr>
        <td>46</td>
        <td>Dudu</td>
        <td>Palmeiras</td>
        <td>31.3</td>
        <td>0.172</td>
        <td>0.147</td>
        <td>0.13</td>
        <td>0.57</td>
    </tr>
    <tr>
        <td>47</td>
        <td>Alejandro Garnacho</td>
        <td>Manchester United</td>
        <td>19.1</td>
        <td>0.088</td>
        <td>0.132</td>
        <td>0.129</td>
        <td>0.922</td>
    </tr>
    <tr>
        <td>48</td>
        <td>David Neres</td>
        <td>Benfica Portugal</td>
        <td>26.5</td>
        <td>0.25</td>
        <td>0.142</td>
        <td>0.129</td>
        <td>0.708</td>
    </tr>
    <tr>
        <td>49</td>
        <td>Gabriel Martinelli</td>
        <td>Arsenal</td>
        <td>22.2</td>
        <td>0.132</td>
        <td>0.133</td>
        <td>0.129</td>
        <td>0.874</td>
    </tr>
    <tr>
        <td>50</td>
        <td>Luis Guilherme</td>
        <td>Palmeiras</td>
        <td>17.2</td>
        <td>0.127</td>
        <td>0.144</td>
        <td>0.129</td>
        <td>0.62</td>
    </tr>
  </tbody>
</table>

Table 8: Player Ranking Based On PCR Predictions In The Case Of Their Transfer To Barcelona

<table>
  <thead>
    <tr>
        <th>#</th>
        <th>player</th>
        <th>team</th>
        <th>age</th>
        <th>PCR</th>
        <th>PCR_pred</th>
        <th>PCR_adj</th>
        <th>stay_proba</th>
    </tr>
  </thead>
  <tbody>
    <tr>
        <td>1</td>
        <td>J. Doku</td>
        <td>Manchester City</td>
        <td>21.2</td>
        <td>0.299</td>
        <td>0.182</td>
        <td>0.182</td>
        <td>0.961</td>
    </tr>
    <tr>
        <td>2</td>
        <td>O. Dembélé</td>
        <td>PSG</td>
        <td>26.3</td>
        <td>0.215</td>
        <td>0.179</td>
        <td>0.173</td>
        <td>0.868</td>
    </tr>
    <tr>
        <td>3</td>
        <td>Sávio</td>
        <td>Girona</td>
        <td>19.4</td>
        <td>0.174</td>
        <td>0.163</td>
        <td>0.162</td>
        <td>0.96</td>
    </tr>
    <tr>
        <td>4</td>
        <td>Nico Williams</td>
        <td>Athletic Bilbao</td>
        <td>21.1</td>
        <td>0.199</td>
        <td>0.155</td>
        <td>0.153</td>
        <td>0.956</td>
    </tr>
    <tr>
        <td>5</td>
        <td>Bryan Zaragoza</td>
        <td>Bayern Munchen</td>
        <td>21.9</td>
        <td>0.162</td>
        <td>0.153</td>
        <td>0.152</td>
        <td>0.961</td>
    </tr>
    <tr>
        <td>6</td>
        <td>K. Kvaratskhelia</td>
        <td>Napoli</td>
        <td>22.5</td>
        <td>0.195</td>
        <td>0.151</td>
        <td>0.15</td>
        <td>0.954</td>
    </tr>
    <tr>
        <td>7</td>
        <td>Vinícius Júnior</td>
        <td>Real Madrid</td>
        <td>23.1</td>
        <td>0.136</td>
        <td>0.148</td>
        <td>0.146</td>
        <td>0.938</td>
    </tr>
    <tr>
        <td>8</td>
        <td>E. Zhegrova</td>
        <td>Lille</td>
        <td>24.4</td>
        <td>0.097</td>
        <td>0.147</td>
        <td>0.143</td>
        <td>0.907</td>
    </tr>
    <tr>
        <td>9</td>
        <td>M. Olise</td>
        <td>Crystal Palace</td>
        <td>21.7</td>
        <td>0.182</td>
        <td>0.146</td>
        <td>0.143</td>
        <td>0.862</td>
    </tr>
    <tr>
        <td>10</td>
        <td>Lamine Yamal</td>
        <td>Barcelona</td>
        <td>16.1</td>
        <td>0.136</td>
        <td>0.144</td>
        <td>0.141</td>
        <td>0.922</td>
    </tr>
    <tr>
        <td>11</td>
        <td>N. Lang</td>
        <td>PSV</td>
        <td>24.2</td>
        <td>0.196</td>
        <td>0.152</td>
        <td>0.14</td>
        <td>0.74</td>
    </tr>
    <tr>
        <td>12</td>
        <td>T. Kubo</td>
        <td>Real Sociedad</td>
        <td>22.2</td>
        <td>0.162</td>
        <td>0.142</td>
        <td>0.14</td>
        <td>0.933</td>
    </tr>
    <tr>
        <td>13</td>
        <td>I. Akhomach</td>
        <td>Villarreal</td>
        <td>19.3</td>
        <td>0.094</td>
        <td>0.142</td>
        <td>0.14</td>
        <td>0.936</td>
    </tr>
    <tr>
        <td>14</td>
        <td>Rafael Leão</td>
        <td>Milan</td>
        <td>24.2</td>
        <td>0.178</td>
        <td>0.142</td>
        <td>0.14</td>
        <td>0.913</td>
    </tr>
    <tr>
        <td>15</td>
        <td>C. Ejuke</td>
        <td>Antwerp</td>
        <td>25.6</td>
        <td>0.158</td>
        <td>0.146</td>
        <td>0.14</td>
        <td>0.848</td>
    </tr>
    <tr>
        <td>16</td>
        <td>R. Sterling</td>
        <td>Chelsea</td>
        <td>28.7</td>
        <td>0.152</td>
        <td>0.143</td>
        <td>0.139</td>
        <td>0.828</td>
    </tr>
    <tr>
        <td>17</td>
        <td>Francisco Conceição</td>
        <td>Porto</td>
        <td>20.7</td>
        <td>0.233</td>
        <td>0.144</td>
        <td>0.139</td>
        <td>0.905</td>
    </tr>
    <tr>
        <td>18</td>
        <td>O. Niang</td>
        <td>Riga</td>
        <td>21.4</td>
        <td>0.296</td>
        <td>0.153</td>
        <td>0.138</td>
        <td>0.734</td>
    </tr>
    <tr>
        <td>19</td>
        <td>O. Sahraoui</td>
        <td>Heerenveen</td>
        <td>22.2</td>
        <td>0.151</td>
        <td>0.145</td>
        <td>0.138</td>
        <td>0.882</td>
    </tr>
    <tr>
        <td>20</td>
        <td>F. Chiesa</td>
        <td>Juventus</td>
        <td>25.8</td>
        <td>0.195</td>
        <td>0.139</td>
        <td>0.137</td>
        <td>0.932</td>
    </tr>
    <tr>
        <td>21</td>
        <td>K. Coman</td>
        <td>Bayern Munchen</td>
        <td>27.2</td>
        <td>0.095</td>
        <td>0.14</td>
        <td>0.137</td>
        <td>0.861</td>
    </tr>
    <tr>
        <td>22</td>
        <td>T. Corbeanu</td>
        <td>Granada</td>
        <td>21.2</td>
        <td>0.152</td>
        <td>0.143</td>
        <td>0.137</td>
        <td>0.869</td>
    </tr>
    <tr>
        <td>23</td>
        <td>J. Dompé</td>
        <td>Hamburger SV</td>
        <td>28.0</td>
        <td>0.158</td>
        <td>0.147</td>
        <td>0.136</td>
        <td>0.768</td>
    </tr>
    <tr>
        <td>24</td>
        <td>Pedro Neto</td>
        <td>Wolverhampton</td>
        <td>23.4</td>
        <td>0.131</td>
        <td>0.138</td>
        <td>0.135</td>
        <td>0.855</td>
    </tr>
    <tr>
        <td>25</td>
        <td>J. Sancho</td>
        <td>Borussia Dortmund</td>
        <td>23.4</td>
        <td>0.224</td>
        <td>0.138</td>
        <td>0.135</td>
        <td>0.858</td>
    </tr>
    <tr>
        <td>26</td>
        <td>T. Ali</td>
        <td>Malmo FF</td>
        <td>24.8</td>
        <td>0.277</td>
        <td>0.142</td>
        <td>0.134</td>
        <td>0.828</td>
    </tr>
    <tr>
        <td>27</td>
        <td>M. Edwards</td>
        <td>Sporting CP</td>
        <td>24.7</td>
        <td>0.277</td>
        <td>0.14</td>
        <td>0.133</td>
        <td>0.866</td>
    </tr>
    <tr>
        <td>28</td>
        <td>Haissem Hassan</td>
        <td>Sporting Gijon</td>
        <td>21.5</td>
        <td>0.125</td>
        <td>0.141</td>
        <td>0.133</td>
        <td>0.873</td>
    </tr>
    <tr>
        <td>29</td>
        <td>Ângelo Gabriel</td>
        <td>Strasbourg</td>
        <td>18.6</td>
        <td>0.125</td>
        <td>0.135</td>
        <td>0.132</td>
        <td>0.918</td>
    </tr>
    <tr>
        <td>30</td>
        <td>J. Bynoe-Gittens</td>
        <td>Borussia Dortmund</td>
        <td>19.0</td>
        <td>0.208</td>
        <td>0.134</td>
        <td>0.132</td>
        <td>0.898</td>
    </tr>
    <tr>
        <td>31</td>
        <td>R. Sottil</td>
        <td>Fiorentina</td>
        <td>24.2</td>
        <td>0.097</td>
        <td>0.135</td>
        <td>0.13</td>
        <td>0.84</td>
    </tr>
    <tr>
        <td>32</td>
        <td>K. Mitoma</td>
        <td>Brighton</td>
        <td>26.2</td>
        <td>0.215</td>
        <td>0.133</td>
        <td>0.13</td>
        <td>0.856</td>
    </tr>
    <tr>
        <td>33</td>
        <td>A. Nusa</td>
        <td>Club Brugge</td>
        <td>18.3</td>
        <td>0.143</td>
        <td>0.135</td>
        <td>0.129</td>
        <td>0.872</td>
    </tr>
    <tr>
        <td>34</td>
        <td>M. Politano</td>
        <td>Napoli</td>
        <td>30.1</td>
        <td>0.188</td>
        <td>0.134</td>
        <td>0.129</td>
        <td>0.796</td>
    </tr>
    <tr>
        <td>35</td>
        <td>B. Gruda</td>
        <td>Mainz 05</td>
        <td>19.2</td>
        <td>0.163</td>
        <td>0.133</td>
        <td>0.128</td>
        <td>0.823</td>
    </tr>
    <tr>
        <td>36</td>
        <td>J. Bakayoko</td>
        <td>PSV</td>
        <td>20.3</td>
        <td>0.194</td>
        <td>0.133</td>
        <td>0.128</td>
        <td>0.928</td>
    </tr>
    <tr>
        <td>37</td>
        <td>Ez Abde</td>
        <td>Real Betis</td>
        <td>21.7</td>
        <td>0.126</td>
        <td>0.13</td>
        <td>0.128</td>
        <td>0.948</td>
    </tr>
    <tr>
        <td>38</td>
        <td>N. Madueke</td>
        <td>Chelsea</td>
        <td>21.4</td>
        <td>0.177</td>
        <td>0.132</td>
        <td>0.128</td>
        <td>0.823</td>
    </tr>
    <tr>
        <td>39</td>
        <td>Alejandro Garnacho</td>
        <td>Manchester United</td>
        <td>19.1</td>
        <td>0.088</td>
        <td>0.129</td>
        <td>0.128</td>
        <td>0.922</td>
    </tr>
    <tr>
        <td>40</td>
        <td>M. Fofana</td>
        <td>Lyon</td>
        <td>18.3</td>
        <td>0.169</td>
        <td>0.131</td>
        <td>0.127</td>
        <td>0.93</td>
    </tr>
    <tr>
        <td>41</td>
        <td>A. Mitriță</td>
        <td>Universitatea Craiova</td>
        <td>28.4</td>
        <td>0.195</td>
        <td>0.133</td>
        <td>0.127</td>
        <td>0.906</td>
    </tr>
    <tr>
        <td>42</td>
        <td>A. Schjelderup</td>
        <td>Nordsjaelland</td>
        <td>19.2</td>
        <td>0.283</td>
        <td>0.13</td>
        <td>0.127</td>
        <td>0.936</td>
    </tr>
    <tr>
        <td>43</td>
        <td>Milson</td>
        <td>Maccabi Tel Aviv</td>
        <td>23.9</td>
        <td>0.178</td>
        <td>0.134</td>
        <td>0.126</td>
        <td>0.856</td>
    </tr>
    <tr>
        <td>44</td>
        <td>S. Ltaief</td>
        <td>Winterthur</td>
        <td>23.3</td>
        <td>0.114</td>
        <td>0.134</td>
        <td>0.126</td>
        <td>0.786</td>
    </tr>
    <tr>
        <td>45</td>
        <td>M. Simon</td>
        <td>Nantes</td>
        <td>28.1</td>
        <td>0.125</td>
        <td>0.132</td>
        <td>0.126</td>
        <td>0.801</td>
    </tr>
    <tr>
        <td>46</td>
        <td>J. Boga</td>
        <td>Nice</td>
        <td>26.6</td>
        <td>0.079</td>
        <td>0.129</td>
        <td>0.125</td>
        <td>0.894</td>
    </tr>
    <tr>
        <td>47</td>
        <td>Dudu</td>
        <td>Palmeiras</td>
        <td>31.3</td>
        <td>0.172</td>
        <td>0.14</td>
        <td>0.125</td>
        <td>0.57</td>
    </tr>
    <tr>
        <td>48</td>
        <td>J. Clarke</td>
        <td>Sunderland</td>
        <td>22.7</td>
        <td>0.268</td>
        <td>0.131</td>
        <td>0.125</td>
        <td>0.939</td>
    </tr>
    <tr>
        <td>49</td>
        <td>Serginho</td>
        <td>Viborg</td>
        <td>22.5</td>
        <td>0.182</td>
        <td>0.135</td>
        <td>0.124</td>
        <td>0.719</td>
    </tr>
    <tr>
        <td>50</td>
        <td>M. Daramy</td>
        <td>Reims</td>
        <td>21.6</td>
        <td>0.077</td>
        <td>0.127</td>
        <td>0.124</td>
        <td>0.926</td>
    </tr>
  </tbody>
</table>

Table 9: Player Ranking Based On PCR Predictions In The Case Of Their Transfer To Milan

<table>
  <thead>
    <tr>
        <th>#</th>
        <th>player</th>
        <th>team</th>
        <th>age</th>
        <th>PCR</th>
        <th>PCR_pred</th>
        <th>PCR_adj</th>
        <th>stay_proba</th>
    </tr>
  </thead>
  <tbody>
    <tr>
        <td>1</td>
        <td>J. Doku</td>
        <td>Manchester City</td>
        <td>21.2</td>
        <td>0.299</td>
        <td>0.173</td>
        <td>0.173</td>
        <td>0.961</td>
    </tr>
    <tr>
        <td>2</td>
        <td>O. Dembélé</td>
        <td>PSG</td>
        <td>26.3</td>
        <td>0.215</td>
        <td>0.168</td>
        <td>0.162</td>
        <td>0.868</td>
    </tr>
    <tr>
        <td>3</td>
        <td>Sávio</td>
        <td>Girona</td>
        <td>19.4</td>
        <td>0.174</td>
        <td>0.159</td>
        <td>0.157</td>
        <td>0.96</td>
    </tr>
    <tr>
        <td>4</td>
        <td>Nico Williams</td>
        <td>Athletic Bilbao</td>
        <td>21.1</td>
        <td>0.199</td>
        <td>0.151</td>
        <td>0.15</td>
        <td>0.956</td>
    </tr>
    <tr>
        <td>5</td>
        <td>K. Kvaratskhelia</td>
        <td>Napoli</td>
        <td>22.5</td>
        <td>0.195</td>
        <td>0.148</td>
        <td>0.147</td>
        <td>0.954</td>
    </tr>
    <tr>
        <td>6</td>
        <td>Bryan Zaragoza</td>
        <td>Bayern Munchen</td>
        <td>21.9</td>
        <td>0.162</td>
        <td>0.147</td>
        <td>0.145</td>
        <td>0.961</td>
    </tr>
    <tr>
        <td>7</td>
        <td>Vinícius Júnior</td>
        <td>Real Madrid</td>
        <td>23.1</td>
        <td>0.136</td>
        <td>0.145</td>
        <td>0.142</td>
        <td>0.938</td>
    </tr>
    <tr>
        <td>8</td>
        <td>Rafael Leão</td>
        <td>Milan</td>
        <td>24.2</td>
        <td>0.178</td>
        <td>0.145</td>
        <td>0.142</td>
        <td>0.913</td>
    </tr>
    <tr>
        <td>9</td>
        <td>R. Sterling</td>
        <td>Chelsea</td>
        <td>28.7</td>
        <td>0.152</td>
        <td>0.142</td>
        <td>0.137</td>
        <td>0.828</td>
    </tr>
    <tr>
        <td>10</td>
        <td>M. Olise</td>
        <td>Crystal Palace</td>
        <td>21.7</td>
        <td>0.182</td>
        <td>0.141</td>
        <td>0.137</td>
        <td>0.862</td>
    </tr>
    <tr>
        <td>11</td>
        <td>N. Lang</td>
        <td>PSV</td>
        <td>24.2</td>
        <td>0.196</td>
        <td>0.149</td>
        <td>0.137</td>
        <td>0.74</td>
    </tr>
    <tr>
        <td>12</td>
        <td>O. Sahraoui</td>
        <td>Heerenveen</td>
        <td>22.2</td>
        <td>0.151</td>
        <td>0.144</td>
        <td>0.137</td>
        <td>0.882</td>
    </tr>
    <tr>
        <td>13</td>
        <td>C. Ejuke</td>
        <td>Antwerp</td>
        <td>25.6</td>
        <td>0.158</td>
        <td>0.144</td>
        <td>0.137</td>
        <td>0.848</td>
    </tr>
    <tr>
        <td>14</td>
        <td>T. Ali</td>
        <td>Malmo FF</td>
        <td>24.8</td>
        <td>0.277</td>
        <td>0.143</td>
        <td>0.134</td>
        <td>0.828</td>
    </tr>
    <tr>
        <td>15</td>
        <td>K. Coman</td>
        <td>Bayern Munchen</td>
        <td>27.2</td>
        <td>0.095</td>
        <td>0.137</td>
        <td>0.133</td>
        <td>0.861</td>
    </tr>
    <tr>
        <td>16</td>
        <td>J. Dompé</td>
        <td>Hamburger SV</td>
        <td>28.0</td>
        <td>0.158</td>
        <td>0.145</td>
        <td>0.133</td>
        <td>0.768</td>
    </tr>
    <tr>
        <td>17</td>
        <td>T. Kubo</td>
        <td>Real Sociedad</td>
        <td>22.2</td>
        <td>0.162</td>
        <td>0.135</td>
        <td>0.133</td>
        <td>0.933</td>
    </tr>
    <tr>
        <td>18</td>
        <td>T. Corbeanu</td>
        <td>Granada</td>
        <td>21.2</td>
        <td>0.152</td>
        <td>0.139</td>
        <td>0.132</td>
        <td>0.869</td>
    </tr>
    <tr>
        <td>19</td>
        <td>Francisco Conceição</td>
        <td>Porto</td>
        <td>20.7</td>
        <td>0.233</td>
        <td>0.137</td>
        <td>0.132</td>
        <td>0.905</td>
    </tr>
    <tr>
        <td>20</td>
        <td>Pedro Neto</td>
        <td>Wolverhampton</td>
        <td>23.4</td>
        <td>0.131</td>
        <td>0.135</td>
        <td>0.132</td>
        <td>0.855</td>
    </tr>
    <tr>
        <td>21</td>
        <td>E. Zhegrova</td>
        <td>Lille</td>
        <td>24.4</td>
        <td>0.097</td>
        <td>0.135</td>
        <td>0.131</td>
        <td>0.907</td>
    </tr>
    <tr>
        <td>22</td>
        <td>F. Chiesa</td>
        <td>Juventus</td>
        <td>25.8</td>
        <td>0.195</td>
        <td>0.133</td>
        <td>0.131</td>
        <td>0.932</td>
    </tr>
    <tr>
        <td>23</td>
        <td>K. Mitoma</td>
        <td>Brighton</td>
        <td>26.2</td>
        <td>0.215</td>
        <td>0.134</td>
        <td>0.131</td>
        <td>0.856</td>
    </tr>
    <tr>
        <td>24</td>
        <td>I. Akhomach</td>
        <td>Villarreal</td>
        <td>19.3</td>
        <td>0.094</td>
        <td>0.133</td>
        <td>0.131</td>
        <td>0.936</td>
    </tr>
    <tr>
        <td>25</td>
        <td>J. Sancho</td>
        <td>Borussia Dortmund</td>
        <td>23.4</td>
        <td>0.224</td>
        <td>0.134</td>
        <td>0.13</td>
        <td>0.858</td>
    </tr>
    <tr>
        <td>26</td>
        <td>Haissem Hassan</td>
        <td>Sporting Gijon</td>
        <td>21.5</td>
        <td>0.125</td>
        <td>0.139</td>
        <td>0.13</td>
        <td>0.873</td>
    </tr>
    <tr>
        <td>27</td>
        <td>O. Niang</td>
        <td>Riga</td>
        <td>21.4</td>
        <td>0.296</td>
        <td>0.144</td>
        <td>0.129</td>
        <td>0.734</td>
    </tr>
    <tr>
        <td>28</td>
        <td>J. Bynoe-Gittens</td>
        <td>Borussia Dortmund</td>
        <td>19.0</td>
        <td>0.208</td>
        <td>0.132</td>
        <td>0.129</td>
        <td>0.898</td>
    </tr>
    <tr>
        <td>29</td>
        <td>R. Sottil</td>
        <td>Fiorentina</td>
        <td>24.2</td>
        <td>0.097</td>
        <td>0.134</td>
        <td>0.129</td>
        <td>0.84</td>
    </tr>
    <tr>
        <td>30</td>
        <td>M. Edwards</td>
        <td>Sporting CP</td>
        <td>24.7</td>
        <td>0.277</td>
        <td>0.134</td>
        <td>0.128</td>
        <td>0.866</td>
    </tr>
    <tr>
        <td>31</td>
        <td>Lamine Yamal</td>
        <td>Barcelona</td>
        <td>16.1</td>
        <td>0.136</td>
        <td>0.129</td>
        <td>0.127</td>
        <td>0.922</td>
    </tr>
    <tr>
        <td>32</td>
        <td>Ângelo Gabriel</td>
        <td>Strasbourg</td>
        <td>18.6</td>
        <td>0.125</td>
        <td>0.129</td>
        <td>0.126</td>
        <td>0.918</td>
    </tr>
    <tr>
        <td>33</td>
        <td>M. Politano</td>
        <td>Napoli</td>
        <td>30.1</td>
        <td>0.188</td>
        <td>0.132</td>
        <td>0.126</td>
        <td>0.796</td>
    </tr>
    <tr>
        <td>34</td>
        <td>Milson</td>
        <td>Maccabi Tel Aviv</td>
        <td>23.9</td>
        <td>0.178</td>
        <td>0.134</td>
        <td>0.126</td>
        <td>0.856</td>
    </tr>
    <tr>
        <td>35</td>
        <td>Ez Abde</td>
        <td>Real Betis</td>
        <td>21.7</td>
        <td>0.126</td>
        <td>0.127</td>
        <td>0.125</td>
        <td>0.948</td>
    </tr>
    <tr>
        <td>36</td>
        <td>B. Gruda</td>
        <td>Mainz 05</td>
        <td>19.2</td>
        <td>0.163</td>
        <td>0.128</td>
        <td>0.124</td>
        <td>0.823</td>
    </tr>
    <tr>
        <td>37</td>
        <td>Alejandro Garnacho</td>
        <td>Manchester United</td>
        <td>19.1</td>
        <td>0.088</td>
        <td>0.125</td>
        <td>0.124</td>
        <td>0.922</td>
    </tr>
    <tr>
        <td>38</td>
        <td>A. Nusa</td>
        <td>Club Brugge</td>
        <td>18.3</td>
        <td>0.143</td>
        <td>0.129</td>
        <td>0.124</td>
        <td>0.872</td>
    </tr>
    <tr>
        <td>39</td>
        <td>N. Madueke</td>
        <td>Chelsea</td>
        <td>21.4</td>
        <td>0.177</td>
        <td>0.127</td>
        <td>0.123</td>
        <td>0.823</td>
    </tr>
    <tr>
        <td>40</td>
        <td>J. Bakayoko</td>
        <td>PSV</td>
        <td>20.3</td>
        <td>0.194</td>
        <td>0.128</td>
        <td>0.123</td>
        <td>0.928</td>
    </tr>
    <tr>
        <td>41</td>
        <td>David Neres</td>
        <td>Benfica Portugal</td>
        <td>26.5</td>
        <td>0.25</td>
        <td>0.133</td>
        <td>0.122</td>
        <td>0.708</td>
    </tr>
    <tr>
        <td>42</td>
        <td>J. Clarke</td>
        <td>Sunderland</td>
        <td>22.7</td>
        <td>0.268</td>
        <td>0.128</td>
        <td>0.122</td>
        <td>0.939</td>
    </tr>
    <tr>
        <td>43</td>
        <td>A. Mitriță</td>
        <td>Universitatea Craiova</td>
        <td>28.4</td>
        <td>0.195</td>
        <td>0.128</td>
        <td>0.122</td>
        <td>0.906</td>
    </tr>
    <tr>
        <td>44</td>
        <td>Daniel Podence</td>
        <td>Olympiacos Piraeus</td>
        <td>27.8</td>
        <td>0.252</td>
        <td>0.131</td>
        <td>0.121</td>
        <td>0.8</td>
    </tr>
    <tr>
        <td>45</td>
        <td>A. Schjelderup</td>
        <td>Nordsjaelland</td>
        <td>19.2</td>
        <td>0.283</td>
        <td>0.125</td>
        <td>0.121</td>
        <td>0.936</td>
    </tr>
    <tr>
        <td>46</td>
        <td>Dudu</td>
        <td>Palmeiras</td>
        <td>31.3</td>
        <td>0.172</td>
        <td>0.136</td>
        <td>0.121</td>
        <td>0.57</td>
    </tr>
    <tr>
        <td>47</td>
        <td>M. Fofana</td>
        <td>Lyon</td>
        <td>18.3</td>
        <td>0.169</td>
        <td>0.125</td>
        <td>0.121</td>
        <td>0.93</td>
    </tr>
    <tr>
        <td>48</td>
        <td>Serginho</td>
        <td>Viborg</td>
        <td>22.5</td>
        <td>0.182</td>
        <td>0.129</td>
        <td>0.119</td>
        <td>0.719</td>
    </tr>
    <tr>
        <td>49</td>
        <td>A. Sbaï</td>
        <td>Grenoble</td>
        <td>22.8</td>
        <td>0.185</td>
        <td>0.128</td>
        <td>0.118</td>
        <td>0.818</td>
    </tr>
    <tr>
        <td>50</td>
        <td>J. Boga</td>
        <td>Nice</td>
        <td>26.6</td>
        <td>0.079</td>
        <td>0.122</td>
        <td>0.118</td>
        <td>0.894</td>
    </tr>
  </tbody>
</table>

Table 10: Player Ranking Based On PCR Predictions In The Case Of Their Transfer To Brighton

<table>
  <thead>
    <tr>
        <th>#</th>
        <th>player</th>
        <th>team</th>
        <th>age</th>
        <th>PCR</th>
        <th>PCR_pred</th>
        <th>PCR_adj</th>
        <th>stay_proba</th>
    </tr>
  </thead>
  <tbody>
    <tr>
        <td>1</td>
        <td>J. Doku</td>
        <td>Manchester City</td>
        <td>21.2</td>
        <td>0.299</td>
        <td>0.161</td>
        <td>0.159</td>
        <td>0.961</td>
    </tr>
    <tr>
        <td>2</td>
        <td>O. Dembélé</td>
        <td>PSG</td>
        <td>26.3</td>
        <td>0.215</td>
        <td>0.15</td>
        <td>0.143</td>
        <td>0.868</td>
    </tr>
    <tr>
        <td>3</td>
        <td>Sávio</td>
        <td>Girona</td>
        <td>19.4</td>
        <td>0.174</td>
        <td>0.141</td>
        <td>0.139</td>
        <td>0.96</td>
    </tr>
    <tr>
        <td>4</td>
        <td>R. Sterling</td>
        <td>Chelsea</td>
        <td>28.7</td>
        <td>0.152</td>
        <td>0.134</td>
        <td>0.129</td>
        <td>0.828</td>
    </tr>
    <tr>
        <td>5</td>
        <td>K. Kvaratskhelia</td>
        <td>Napoli</td>
        <td>22.5</td>
        <td>0.195</td>
        <td>0.13</td>
        <td>0.127</td>
        <td>0.954</td>
    </tr>
    <tr>
        <td>6</td>
        <td>Nico Williams</td>
        <td>Athletic Bilbao</td>
        <td>21.1</td>
        <td>0.199</td>
        <td>0.129</td>
        <td>0.127</td>
        <td>0.956</td>
    </tr>
    <tr>
        <td>7</td>
        <td>M. Olise</td>
        <td>Crystal Palace</td>
        <td>21.7</td>
        <td>0.182</td>
        <td>0.13</td>
        <td>0.126</td>
        <td>0.862</td>
    </tr>
    <tr>
        <td>8</td>
        <td>J. Dompé</td>
        <td>Hamburger SV</td>
        <td>28.0</td>
        <td>0.158</td>
        <td>0.138</td>
        <td>0.126</td>
        <td>0.768</td>
    </tr>
    <tr>
        <td>9</td>
        <td>C. Ejuke</td>
        <td>Antwerp</td>
        <td>25.6</td>
        <td>0.158</td>
        <td>0.133</td>
        <td>0.125</td>
        <td>0.848</td>
    </tr>
    <tr>
        <td>10</td>
        <td>O. Sahraoui</td>
        <td>Heerenveen</td>
        <td>22.2</td>
        <td>0.151</td>
        <td>0.131</td>
        <td>0.124</td>
        <td>0.882</td>
    </tr>
    <tr>
        <td>11</td>
        <td>Bryan Zaragoza</td>
        <td>Bayern Munchen</td>
        <td>21.9</td>
        <td>0.162</td>
        <td>0.126</td>
        <td>0.124</td>
        <td>0.961</td>
    </tr>
    <tr>
        <td>12</td>
        <td>Haissem Hassan</td>
        <td>Sporting Gijon</td>
        <td>21.5</td>
        <td>0.125</td>
        <td>0.132</td>
        <td>0.123</td>
        <td>0.873</td>
    </tr>
    <tr>
        <td>13</td>
        <td>T. Corbeanu</td>
        <td>Granada</td>
        <td>21.2</td>
        <td>0.152</td>
        <td>0.127</td>
        <td>0.12</td>
        <td>0.869</td>
    </tr>
    <tr>
        <td>14</td>
        <td>Vinícius Júnior</td>
        <td>Real Madrid</td>
        <td>23.1</td>
        <td>0.136</td>
        <td>0.123</td>
        <td>0.12</td>
        <td>0.938</td>
    </tr>
    <tr>
        <td>15</td>
        <td>T. Ali</td>
        <td>Malmo FF</td>
        <td>24.8</td>
        <td>0.277</td>
        <td>0.127</td>
        <td>0.119</td>
        <td>0.828</td>
    </tr>
    <tr>
        <td>16</td>
        <td>N. Lang</td>
        <td>PSV</td>
        <td>24.2</td>
        <td>0.196</td>
        <td>0.128</td>
        <td>0.117</td>
        <td>0.74</td>
    </tr>
    <tr>
        <td>17</td>
        <td>K. Mitoma</td>
        <td>Brighton</td>
        <td>26.2</td>
        <td>0.215</td>
        <td>0.12</td>
        <td>0.117</td>
        <td>0.856</td>
    </tr>
    <tr>
        <td>18</td>
        <td>O. Niang</td>
        <td>Riga</td>
        <td>21.4</td>
        <td>0.296</td>
        <td>0.13</td>
        <td>0.116</td>
        <td>0.734</td>
    </tr>
    <tr>
        <td>19</td>
        <td>R. Sottil</td>
        <td>Fiorentina</td>
        <td>24.2</td>
        <td>0.097</td>
        <td>0.121</td>
        <td>0.116</td>
        <td>0.84</td>
    </tr>
    <tr>
        <td>20</td>
        <td>Pedro Neto</td>
        <td>Wolverhampton</td>
        <td>23.4</td>
        <td>0.131</td>
        <td>0.12</td>
        <td>0.116</td>
        <td>0.855</td>
    </tr>
    <tr>
        <td>21</td>
        <td>I. Akhomach</td>
        <td>Villarreal</td>
        <td>19.3</td>
        <td>0.094</td>
        <td>0.119</td>
        <td>0.116</td>
        <td>0.936</td>
    </tr>
    <tr>
        <td>22</td>
        <td>Rafael Leão</td>
        <td>Milan</td>
        <td>24.2</td>
        <td>0.178</td>
        <td>0.118</td>
        <td>0.115</td>
        <td>0.913</td>
    </tr>
    <tr>
        <td>23</td>
        <td>Francisco Conceição</td>
        <td>Porto</td>
        <td>20.7</td>
        <td>0.233</td>
        <td>0.12</td>
        <td>0.114</td>
        <td>0.905</td>
    </tr>
    <tr>
        <td>24</td>
        <td>K. Coman</td>
        <td>Bayern Munchen</td>
        <td>27.2</td>
        <td>0.095</td>
        <td>0.118</td>
        <td>0.114</td>
        <td>0.861</td>
    </tr>
    <tr>
        <td>25</td>
        <td>Ângelo Gabriel</td>
        <td>Strasbourg</td>
        <td>18.6</td>
        <td>0.125</td>
        <td>0.117</td>
        <td>0.114</td>
        <td>0.918</td>
    </tr>
    <tr>
        <td>26</td>
        <td>E. Zhegrova</td>
        <td>Lille</td>
        <td>24.4</td>
        <td>0.097</td>
        <td>0.117</td>
        <td>0.113</td>
        <td>0.907</td>
    </tr>
    <tr>
        <td>27</td>
        <td>A. Mitriță</td>
        <td>Universitatea Craiova</td>
        <td>28.4</td>
        <td>0.195</td>
        <td>0.12</td>
        <td>0.113</td>
        <td>0.906</td>
    </tr>
    <tr>
        <td>28</td>
        <td>J. Clarke</td>
        <td>Sunderland</td>
        <td>22.7</td>
        <td>0.268</td>
        <td>0.119</td>
        <td>0.112</td>
        <td>0.939</td>
    </tr>
    <tr>
        <td>29</td>
        <td>S. Ltaief</td>
        <td>Winterthur</td>
        <td>23.3</td>
        <td>0.114</td>
        <td>0.121</td>
        <td>0.112</td>
        <td>0.786</td>
    </tr>
    <tr>
        <td>30</td>
        <td>A. Nusa</td>
        <td>Club Brugge</td>
        <td>18.3</td>
        <td>0.143</td>
        <td>0.118</td>
        <td>0.112</td>
        <td>0.872</td>
    </tr>
    <tr>
        <td>31</td>
        <td>N. Madueke</td>
        <td>Chelsea</td>
        <td>21.4</td>
        <td>0.177</td>
        <td>0.116</td>
        <td>0.112</td>
        <td>0.823</td>
    </tr>
    <tr>
        <td>32</td>
        <td>Milson</td>
        <td>Maccabi Tel Aviv</td>
        <td>23.9</td>
        <td>0.178</td>
        <td>0.12</td>
        <td>0.112</td>
        <td>0.856</td>
    </tr>
    <tr>
        <td>33</td>
        <td>M. Edwards</td>
        <td>Sporting CP</td>
        <td>24.7</td>
        <td>0.277</td>
        <td>0.118</td>
        <td>0.111</td>
        <td>0.866</td>
    </tr>
    <tr>
        <td>34</td>
        <td>F. Chiesa</td>
        <td>Juventus</td>
        <td>25.8</td>
        <td>0.195</td>
        <td>0.114</td>
        <td>0.111</td>
        <td>0.932</td>
    </tr>
    <tr>
        <td>35</td>
        <td>J. Boga</td>
        <td>Nice</td>
        <td>26.6</td>
        <td>0.079</td>
        <td>0.115</td>
        <td>0.111</td>
        <td>0.894</td>
    </tr>
    <tr>
        <td>36</td>
        <td>Alejandro Garnacho</td>
        <td>Manchester United</td>
        <td>19.1</td>
        <td>0.088</td>
        <td>0.112</td>
        <td>0.111</td>
        <td>0.922</td>
    </tr>
    <tr>
        <td>37</td>
        <td>T. Kubo</td>
        <td>Real Sociedad</td>
        <td>22.2</td>
        <td>0.162</td>
        <td>0.113</td>
        <td>0.11</td>
        <td>0.933</td>
    </tr>
    <tr>
        <td>38</td>
        <td>J. Bynoe-Gittens</td>
        <td>Borussia Dortmund</td>
        <td>19.0</td>
        <td>0.208</td>
        <td>0.112</td>
        <td>0.109</td>
        <td>0.898</td>
    </tr>
    <tr>
        <td>39</td>
        <td>A. Schjelderup</td>
        <td>Nordsjaelland</td>
        <td>19.2</td>
        <td>0.283</td>
        <td>0.113</td>
        <td>0.109</td>
        <td>0.936</td>
    </tr>
    <tr>
        <td>40</td>
        <td>Serginho</td>
        <td>Viborg</td>
        <td>22.5</td>
        <td>0.182</td>
        <td>0.119</td>
        <td>0.109</td>
        <td>0.719</td>
    </tr>
    <tr>
        <td>41</td>
        <td>M. Fofana</td>
        <td>Lyon</td>
        <td>18.3</td>
        <td>0.169</td>
        <td>0.113</td>
        <td>0.108</td>
        <td>0.93</td>
    </tr>
    <tr>
        <td>42</td>
        <td>B. Gruda</td>
        <td>Mainz 05</td>
        <td>19.2</td>
        <td>0.163</td>
        <td>0.113</td>
        <td>0.108</td>
        <td>0.823</td>
    </tr>
    <tr>
        <td>43</td>
        <td>Y. Soteldo</td>
        <td>Santos</td>
        <td>25.8</td>
        <td>0.27</td>
        <td>0.125</td>
        <td>0.108</td>
        <td>0.485</td>
    </tr>
    <tr>
        <td>44</td>
        <td>Dudu</td>
        <td>Palmeiras</td>
        <td>31.3</td>
        <td>0.172</td>
        <td>0.122</td>
        <td>0.108</td>
        <td>0.57</td>
    </tr>
    <tr>
        <td>45</td>
        <td>J. Sancho</td>
        <td>Borussia Dortmund</td>
        <td>23.4</td>
        <td>0.224</td>
        <td>0.111</td>
        <td>0.107</td>
        <td>0.858</td>
    </tr>
    <tr>
        <td>46</td>
        <td>Danilo Al Saed</td>
        <td>Sandefjord</td>
        <td>24.1</td>
        <td>0.11</td>
        <td>0.118</td>
        <td>0.107</td>
        <td>0.741</td>
    </tr>
    <tr>
        <td>47</td>
        <td>C. Hudson-Odoi</td>
        <td>Nottingham Forest</td>
        <td>22.8</td>
        <td>0.114</td>
        <td>0.11</td>
        <td>0.107</td>
        <td>0.884</td>
    </tr>
    <tr>
        <td>48</td>
        <td>J. Enciso</td>
        <td>Brighton</td>
        <td>19.6</td>
        <td>0.215</td>
        <td>0.114</td>
        <td>0.106</td>
        <td>0.677</td>
    </tr>
    <tr>
        <td>49</td>
        <td>A. Sbaï</td>
        <td>Grenoble</td>
        <td>22.8</td>
        <td>0.185</td>
        <td>0.116</td>
        <td>0.106</td>
        <td>0.818</td>
    </tr>
    <tr>
        <td>50</td>
        <td>David Neres</td>
        <td>Benfica Portugal</td>
        <td>26.5</td>
        <td>0.25</td>
        <td>0.117</td>
        <td>0.106</td>
        <td>0.708</td>
    </tr>
  </tbody>
</table>