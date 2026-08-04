StatsBomb Conference 2023 logo

# Evaluation of Team Defense Positioning by Computing Counterfactuals using StatsBomb 360 data

Final draft

Rikuhei Umemoto<sup>1</sup> and Keisuke Fujii<sup>2</sup>

Graduate School of Informatics, Nagoya University, Nagoya, Japan.

## Abstract

Computing the optimal defensive player positioning in football is challenging but valuable for the decision-making of both players and coaches. Previous studies have utilized mathematical-based probabilistic models to represent off-ball scoring opportunities (OBSO). However, these studies did not focus on defending sides, where the usage of the event label is limited. In addition, the previous studies on defensive evaluation cannot explain where each defender should be positioned at that time and evaluate defensive positioning. In this study, we propose an evaluation method of team defense positioning by computing counterfactuals using StatsBomb 360 data. We also identify the optimal positioning of the defensive teams by searching counterfactual positionings. This study will quantitatively allow us to evaluate team defenses and help the players and their coaches more easily suggest ways to improve their decision-making abilities in future games.

## Introduction

What should our team have done in the game? This question is one of the most valuable for players and coaches in team sports. If we can answer this, we expect we will support decision-making on what to do in the next match because we can know what led to this result in the previous game. Therefore, we establish a quantitative evaluation metric for sports that helps us answer this question.

Most evaluation metrics for football are for on-ball players [1-5]. However, the off-ball duration when they are not touching the ball is longer than the on-ball duration when they are performing the ball. Therefore, off-ball evaluation metrics are required more than on-ball evaluation ones. Some researchers proposed various off-ball player evaluation methods for attacking [6-10] and defending [11-15] sides. For example, Off-Ball Scoring Opportunity (OBSO) [6] is a method to compute the probability of whether off-ball attackers can score a goal in the next event. However, most of them use data containing positional information of all players, which we cannot use as it is for data with incomplete

<sup>1</sup> umemoto.rikuhei@g.sp.m.is.nagoya-u.ac.jp

<sup>2</sup> fujii@i.nagoya-u.ac.jp

1

player information, such as most open data. Therefore, we develop methods that enable us to evaluate teams or players only with incomplete data.

Previous studies proposed some evaluation methods for football teams using this incomplete data. Concerning attacks, some have evaluated the value of the creativity of a player's passes [16], others have evaluated line-breaking passes [17], and others have applied this to evaluate passes that pass between defenders [18]. However, there are few metrics for defending, such as generalized defensive metrics using predicted probabilities of defensive events [19] or evaluating ball recovery [20], because they are less well-defined than offensive events. Therefore, we use incomplete data to consider where players should have been and to evaluate team defense by applying an evaluation method of attackers.

In this study, we propose an evaluation method of team defense positioning by computing counterfactuals using StatsBomb 360 data called EF-OBSO (Event and Freeze frame based OBSO). We also find the optimal positioning of the defensive teams by searching counterfactual positionings, which is called DRSO (Defense Response to Scoring Opportunity). This study will allow us to quantitatively evaluate team defenses and help the players and their coaches more easily suggest ways to improve their decision-making abilities in future games.

The contributions of our research are threefold. (1) We propose EF-OBSO to compute OBSO [6] with incomplete location data by adding prior knowledge; (2) we propose a new approach for considering counterfactuals by computing the optimal positioning of the defensive teams called DRSO; and (3) we performed experiments to examine the validity with various parameters in EF-OBSO and to analyze team defense and their season-by-season defensive improvements using DRSO.

# Materials and methods

## 2.1 Dataset

In this study, we used a dataset from 290 English Premier League games in the 2021-22 season and 290 games in the 2022-23 season provided by StatsBomb Inc. (UK). This dataset includes ten teams (Arsenal, Brentford, Brighton & Hove Albion, Chelsea, Everton, Leeds United, Liverpool, Manchester City, Manchester United and Tottenham Hotspur) that have data for all games per season and twelve teams (Aston Villa, Bournemouth, Burnley, Crystal Palace, Fulham, Leicester City, Newcastle United, Norwich, Nottingham Forest, Southampton, Watford, West Ham and Wolverhampton) that have data for 20 games per season. Due to calculation time and fair evaluation, we used 180 game data, which was the number of combinations of the ten teams with data for all games in each season. In addition, this dataset has event data and 360 data. The former data includes the type and result of an event, such as a pass, the time, and the location coordinates of the ball when an event happened. On the other hand, the latter data includes the

positional coordinates of the players who appear in the frame at the start of the event, called the freeze frame of the broadcast video. Note that we cannot obtain the coordinates of all 22 players in many frames because many scenes in live football broadcasts do not show all 22 players.

## 2.2 Data preprocessing

In this subsection, we describe data preprocessing and feature creation. We used the data preprocessed here to calculate OBSO and DRSO. First, we defined the terms and equations utilized in the subsequent sections. Assume that $s_i$ represents the $i$th state within a game, and $S = [s_1, \dots, s_N]$ chronologically represents the complete set of such states throughout the game. To evaluate all state transitions for attacking and defending actions in this research, we employed the time index as the Ith "event." Next, using event and 360 data described above, we created the features at the Ith state, such as id, team, event type, event result, period, starting time, ending time, duration, actor name, and starting and ending xy coordinates of the ball and players. In this study, concerning events, we considered the following: ball recovery, block, carry, clearance, corner, corner shot, cross, dispossessed, dribble, foul, free kick, free kick shot, goal kick, ground pass, high pass, interception, keeper collect, keeper claim, keeper punch, keeper save, low pass, miscontrol, penalty shot, shield, shot, tackle, and throw-in.

## 2.3 Event and Freeze-frame based OBSO (EF-OBSO)

In this section, we describe one of the core ideas of our proposed method with data preprocessed in the way described above.

### 2.3.1 OBSO

OBSO, proposed by Spearman [6], uses tracking data to express the probability of whether an off-ball player in a given moment can score a goal in the next state using a physical model. Let $T_r$ represent the probability of the next on-ball event occurring at point $r$ on the pitch, $C_r$ the one of an attacking player controlling the ball at $r$, and $G_r$ the one of the player's shot resulting in a score at $r$, we can express OBSO using the following equation:

$$P(G|D) = \sum_{r \in \text{pitch}} P(G_r|C_r, T_r, D)P(C_r|T_r, D)P(T_r|D) \tag{1}$$

where $D$ is the instantaneous state of a game (including players’ locations and velocities). To simplify this model, we considered replacing $G_r$ with $S_r$, which represents the likelihood of scoring at point $r$.

In addition, to calculate $P(C_r|T_r, D)$ in this model, Spearman employed data containing the position coordinates and velocities of all players at any given event and a model called Potential Pitch Control Field (PPCF, [21]). PPCF is a mathematical model for ball control in football based on the notions of the time required for a player to reach the ball and the

time necessary for a player to assume control over the ball. We calculated this value from the following differential equation:

$$ \frac{dPPCF_j}{dT}(t, r, T|\sigma, \lambda_j) = (1 - \sum_{k} PPCF_k(t, T|\sigma, \lambda_j))f_j(t, r, T|\sigma)\lambda_j, \tag{2} $$

where $f_j(t, r, T|\sigma)\lambda_j$ represents the probability that the player $j$ at a time $t$ can reach a point $r$ within some time $T$ and is given by:

$$ f_j(t, r, T|\sigma) = [1 + \exp(-\pi \frac{T - \tau(t, r)}{\sqrt{3}\sigma})]^{-1}, \tag{3} $$

where $\tau$ represents the expected intercept time. This equation described that it is necessary to consider the velocity at which the player reached some points and to fix the values $\sigma$ and $\lambda_j$. In this study, we set these values as $0.45$ and $4.3$ ($12.9$ if the player $j$ was a goalkeeper) following the previous study [6].

This method has the advantage that we can evaluate and interpret off-ball attackers' states. Since the model aimed to see whether a shot can lead to a score at every point on the pitch in the next state, it can describe not only an on-ball player at a certain point but also off-ball attackers on the field. However, this method has three disadvantages:

1. It used data containing the location and speed information of all 22 players. This kind of data is private or expensive, so the public and amateur teams must wait to obtain and analyze it quickly.

2. It focuses only on the attackers. Since scoring opportunities for attackers are conceding ones for defenders, it applies to their evaluation.

3. It only reflects what happened in the data. It is possible to provide feedback on positioning by looking at how the OBSO values would change if a player have been in a different location than that in the data.

### 2.3.2 EF-OBSO

Here, we explain how to apply the OBSO to the StatsBomb event and 360 data, and verify this method. We showed the specific flow in Figure 1. First, we calculated players' locations and their estimated velocities. As mentioned above, the number of players in the freeze frame for each event is often not 22. Additionally, the players do not have any identifying information, such as uniform numbers (except when a shot occurs). Hence, it is impossible to calculate the actual velocity between events for each player.

Given these issues, we in this study only consider an attacker was invading in the attacking third or was taking a shot. This assumption has the following three advantages.

The first is to include the location information of more players than non-attacking third scenes, especially a defensive goalkeeper, in the freeze frame, which only captures a limited area on the pitch. Figure 2 shows the number of events (vertical axis) as a function of the number of players in a freeze frame (horizontal axis). Also, Figure 3 shows the number of events (vertical axis) based on the existence of a defensive keeper in a freeze

StatsBomb Conference 2023 logo

```mermaid
graph TD
    Start([start]) --> EventData[Event and 360 data]
    EventData --> DataPre[Data Preprocessing]
    DataPre --> PreData[Preprocessed Data]
    
    subgraph Calculate_EF_OBSO [Calculate EF-OBSO]
        InitGames[game_num = 1N = Total number of target games]
        CheckGame{game_num > N?}
        InitEvents[event_num = 1M = Total number of events of game_num]
        CheckEvent{event_num > M}
        InAttackingThird{Did this event happen<font color="red">in the attacking third?</font>}
        CalcOBSO[<font color="red">calculate OBSO</font>]
        
        PreData --> InitGames
        InitGames --> CheckGame
        CheckGame -- No --> InitEvents
        InitEvents --> CheckEvent
        CheckEvent -- No --> InAttackingThird
        InAttackingThird -- Yes --> CalcOBSO
    end

    subgraph Verify_EF_OBSO [Verify EF-OBSO]
        Verification[<b>Verification</b>Calculate RMSE → Mean and Std]
        RMSE_attackers[RMSE<sub>attackers</sub> : for each event]
        SE_scorer[SE<sub>scorer</sub> and SE<sub>non-scorer</sub> : for each game and for each team]
        AddResults[Add the results to metric_listsRMSE<sub>attackers</sub>SE<sub>scorer</sub>SE<sub>non-scorer</sub>]
        CalcRMSE[Calculate RMSE for all attackersEq. 5 in the main text]
        ShotNotPenalty{The event was<font color="red">"shot" and not "penalty shot"</font>}
        IsGoal{The event<font color="red">was "goal"</font>}
        CalcSE[Calculate squared error (SE) for shooter]
        ScorerGoal[scorer: (1 - EF-OBSO<sub>shooter</sub>)<sup>2</sup>non_scorer: ignored]
        ScorerNoGoal[scorer: ignorednon_scorer: (0 - EF-OBSO<sub>shooter</sub>)<sup>2</sup>]
        End([end])

        CheckGame -- Yes --> Verification
        Verification --> RMSE_attackers
        Verification --> SE_scorer
        CalcOBSO --> ShotNotPenalty
        InAttackingThird -- No --> ShotNotPenalty
        ShotNotPenalty -- Yes --> IsGoal
        IsGoal -- Yes --> CalcSE
        CalcSE --> ScorerGoal
        IsGoal -- No --> ScorerNoGoal
        ScorerGoal --> AddResults
        ScorerNoGoal --> AddResults
        AddResults --> CalcRMSE
        CalcRMSE --> event_inc[event_num += 1]
        event_inc --> CheckEvent
        CheckEvent -- Yes --> game_inc[game_num += 1]
        game_inc --> CheckGame
        ShotNotPenalty -- No --> event_inc
        RMSE_attackers --> End
        SE_scorer --> End
    end
```

Figure 1: The flowchart of calculating and verifying EF-OBSO. This figure shows the flow of calculation and verification of our proposed method. The red letters are important parts of this study.

frame (horizontal axis). Each figure shows (a) the case of attacking third and (b) otherwise. According to Figure 2, the median number of visible players in the attacking third was more than otherwise. In addition, the ratio of the number of events showing 21 players to the total number of events is also higher in the attacking third (1085/92294 ~ 0.01176) than otherwise (976/290711 ~ 0.003357). Moreover, according to Figure 3, it can be seen that the assumption of (a) can capture more defensive keepers. In the OBSO, note that an attacking player in an offside position has zero scoring probability. Therefore, it is better to assume that we can calculate the OBSO with more information about the defending goalkeeper.

The second is that we can fix the directions of players' movements. We observe more diverse behaviors of attacking players in places other than the attacking third, such as a forward player coming down to the midfield to receive the ball. Hence, it is challenging to estimate players' directions using this data without knowing the speed information of each player. On the other hand, when an attacking player invades the attacking third,

StatsBomb Conference 2023 logo

(a) The number of visible players in attacking-third (b) The number of visible players otherwise
The number of event : 92294 The number of event : 290711

<table>
  <thead>
    <tr>
        <th>Number of players</th>
        <th>(a) Attacking-third</th>
        <th>(b) Otherwise</th>
    </tr>
  </thead>
  <tbody>
    <tr>
        <td>0</td>
        <td>8603</td>
        <td>27434</td>
    </tr>
    <tr>
        <td>1</td>
        <td> </td>
        <td> </td>
    </tr>
    <tr>
        <td>2</td>
        <td> </td>
        <td> </td>
    </tr>
    <tr>
        <td>3</td>
        <td> </td>
        <td> </td>
    </tr>
    <tr>
        <td>4</td>
        <td> </td>
        <td> </td>
    </tr>
    <tr>
        <td>5</td>
        <td> </td>
        <td> </td>
    </tr>
    <tr>
        <td>6</td>
        <td> </td>
        <td> </td>
    </tr>
    <tr>
        <td>7</td>
        <td> </td>
        <td> </td>
    </tr>
    <tr>
        <td>8</td>
        <td> </td>
        <td> </td>
    </tr>
    <tr>
        <td>9</td>
        <td> </td>
        <td> </td>
    </tr>
    <tr>
        <td>10</td>
        <td> </td>
        <td> </td>
    </tr>
    <tr>
        <td>11</td>
        <td> </td>
        <td>17328</td>
    </tr>
    <tr>
        <td colspan="3">12</td>
    </tr>
    <tr>
        <td>13</td>
        <td>6496</td>
        <td> </td>
    </tr>
    <tr>
        <td>14</td>
        <td> </td>
        <td>27269</td>
    </tr>
    <tr>
        <td colspan="3">15</td>
    </tr>
    <tr>
        <td>16</td>
        <td>12110</td>
        <td>28468</td>
    </tr>
    <tr>
        <td>17</td>
        <td>11882</td>
        <td> </td>
    </tr>
    <tr>
        <td>18</td>
        <td> </td>
        <td> </td>
    </tr>
    <tr>
        <td>19</td>
        <td> </td>
        <td> </td>
    </tr>
    <tr>
        <td>20</td>
        <td> </td>
        <td> </td>
    </tr>
    <tr>
        <td>21</td>
        <td>1085</td>
        <td>976</td>
    </tr>
    <tr>
        <td colspan="3">22</td>
    </tr>
  </tbody>
</table>

Figure 2: The histogram of visible players in freeze frames. These figures show the number of events (vertical axis) as a function of the number of players in a freeze frame (horizontal axis). Also, The red bar represents the median, and the green bars represent the first and third quartiles.

(a) The number of defense GK in attacking-third (b) The number of defense GK otherwise
The number of event : 92294 The number of event : 290711

<table>
  <thead>
    <tr>
        <th>Number of GK</th>
        <th>(a) Attacking-third</th>
        <th>(b) Otherwise</th>
    </tr>
  </thead>
  <tbody>
    <tr>
        <td>0</td>
        <td>37935</td>
        <td>265741</td>
    </tr>
    <tr>
        <td>1</td>
        <td>54359</td>
        <td>24970</td>
    </tr>
    <tr>
        <td colspan="3">2</td>
    </tr>
  </tbody>
</table>

Figure 3: The histogram of visible defense goalkeepers in freeze frames. These figures show the number of events (vertical axis) as a function of the number of a defensive goal keeper in a freeze frame (horizontal axis). Also, this figure represents 1 if the player is visible in a given freeze frame, 0 otherwise.

both attackers and defenders in a freeze frame will often care about the goal of the same side. Therefore, it will be fine even if we assume the player is moving toward this goal.

The third is that we can consider the optimal positionings of players in areas that can be more dangerous for defenders. The final goal of this study is to consider what kind of positioning should be done for the defenders when the attackers threaten their own team's goal. Therefore, it is reasonable to assume the attacking third, where many shooting events will occur.

StatsBomb Conference 2023 logo

We describe below how we calculated OBSO for events that satisfy the above assumptions. First, we calculated the ball velocity for each event using the event data. Next, we set players' speeds using the computed speeds and 360 data. However, since both data do not include them, we needed to verify these parameters. We described these verification details in subsection 2.3.3. As a result of this verification, we adopted the most suitable values for calculating EF-OBSO and DRSO. Third, we set the offside line based on the players' information and did not compute the OBSO of the players in the offside position. We should have considered the event with details about a defensive keeper. Fourth, we calculated the value of the PPCF for each grid of 32×50 divisions of the pitch. If the event name is ground pass, low pass, or carry, we calculated the value based on the PPCF formula. However, if not, we figured the one based on the following formula:

$$ \frac{dPPCF_j}{dT}(t, r, T|\sigma, \lambda_j) = (1 - \sum_k PPCF_k(t, r, T|\sigma, \lambda_j))f_j(t, r, T|\sigma)\lambda_j. \tag{4} $$

This is because players cannot intercept the movement of the ball that goes over the player's head, such as a high pass, so it is possible to intercept it only at the point where the ball arrives. Finally, we calculated OBSO using the obtained PPCF and the formula 1. To calculate equations 1 to 4, we used and modified the grid data and code on the GitHub repository.<sup>3</sup>

## 2.3.3 Verification of EF-OBSO

Next, we verified the EF-OBSO method concerning the speeds of players. As the average speed of a football player during a game is 10~11[mph] (4.47~4.92[m/s]), we set and verified player speeds in this study as follows.

I. All players' speeds were 0.0[m/s],
   Assumed that the actor velocity was the same as the ball velocity if the actor carried it or 0.0[m/s] otherwise, and the defensive keeper speed was 2.0[m/s] towards the y-coordinate of the endpoint of the ball if the event was a shot or 0.0[m/s] otherwise, and
II. all other players' speeds were 0.0[m/s] towards the goal,
III. all other players' speeds were 4.0[m/s] towards the goal,
IV. all other players' speeds were 4.5[m/s] towards the goal,
V. all other players' speeds were 5.0[m/s] towards the goal.

Note that for conditions 2 to 5, when the actor is an attacker, the defender closest to the actor went to the final point of the ball at the same speed as the other players.

For each case, we verified this EF-OBSO value of attackers with the actual score (0 or 1). First, while calculating OBSO for an event, we created branches according to whether the event was a shooting scene. If it was not a shooting scene, we skipped the following operations, and if it was, we further branches based on whether a score event was. For a

StatsBomb Conference 2023 logo

```mermaid
graph TD
    Start([start]) --> Event360[Event and 360 data]
    Event360 --> DataPre[Data Preprocessing]
    DataPre --> PreData[Preprocessed Data]
    
    subgraph Calculate_EF_OBSO [Calculate EF-OBSO]
        GNumInit[game_num = 1N = Total number oftarget games] --> GNumCheck{game_num > N?}
        GNumCheck -- No --> ENumInit[event_num = 1M = Total number ofevents of game_num]
        ENumInit --> ENumCheck{event_num > M?}
        ENumCheck -- No --> AttThird{Did this event happenin the attacking third?}
        AttThird -- Yes --> CalcOBSO[<font color="red">calculate OBSO</font>]
        AttThird -- No --> AddResult[Add this result to thevalue list of thedefensive team]
        CalcOBSO --> ENumInc[event_num += 1]
        AddResult --> ENumInc
        ENumInc --> ENumCheck
        ENumCheck -- Yes --> GNumInc[game_num += 1]
        GNumInc --> GNumCheck
    end

    PreData --> GNumInit
    GNumCheck -- Yes --> TNumInit

    subgraph Calculate_DRSO [Calculate DRSO and evaluate team defenses with it]
        TNumInit[team_num = 1A = Total numberof target teams] --> TNumCheck{team_num > A?}
        TNumCheck -- No --> AvgTeam[Average the teamvalue list]
        AvgTeam --> CalcDRSO[<font color="red">calculate DRSO</font>]
        CalcDRSO --> TNumInc[team_num += 1]
        TNumInc --> TNumCheck
        TNumCheck -- Yes --> Eval[<b>Evaluation</b> (per team)Compare the total number ofconcedes and the resultsCompare results from the 2021-22 seasonwith results from the 2022-23 season]
    end

    Eval --> End([end])
```

Figure 4: The flowchart of calculating DRSO and evaluating team defenses with it. This figure shows the flow of calculation of another proposed method and evaluation of team defenses. The red letters are important parts of this study.

player who took the shot (a shooter), the ground truth was set to 1 if it was a goal and 0 if it was not. In other words, we defined the squared error for a shooter (denoted by $\text{SE}_{\text{shooter}}$) as $(1 - \text{EF-OBSO}_{\text{shooter}})^2$ or $(0 - \text{EF-OBSO}_{\text{shooter}})^2$. While The former error was used as the squared error for a scorer (denoted by $\text{SE}_{\text{scorer}}$) if the event was a goal, the latter error was used as the squared error for a non-scorer (denoted by $\text{SE}_{\text{non-scorer}}$) if it was not. After this manipulation, we calculated squared errors for all attackers except for the shooter in the freeze frame of that shooting scene (denoted by $\text{SE}_{\text{non-shooter}}$), with 0 being the ground truth ($(0 - \text{EF-OBSO}_{\text{non-shooter}})^2$). Using these attackers' squared errors, we calculated the following equation and got the RMSE for the attackers at that event:

$$ \text{RMSE}_{\text{attackers}} = \sqrt{\frac{1}{|\text{attackers}|} \sum_{i \in \text{non-shooters}} \text{SE}_{\text{non-shooter},i} + \text{SE}_{\text{shooter}}} \tag{5} $$

where $|\text{attackers}|$ represents the number of all attackers except for the shooter in the freeze frame of that shooting scene. For all the shoot events in data, we performed the above and obtained $\text{RMSE}_{\text{attackers}}$, $\text{SE}_{\text{scorer}}$, and $\text{SE}_{\text{non-scorer}}$. In addition, we used these

Diagram showing (a) The point of maximum OBSO, (b) The grid where the defender was and the closest defender, and (c) Optimal Positioning with sample Max values.

(c) (Observed) Max: 0.133

Figure 5: The example of DRSO. In each figure, the red and blue markers represent attackers and defenders respectively. Note that the values written in Figure 5(c) are sample values for clarity of explanation and different from actual values.

values to verify EF-OBSO results. First, for RMSE<sub>attackers</sub>, we calculated the mean and standard deviation for all shoot events. Second, for SE<sub>scorer</sub> and SE<sub>non-scorer</sub>, we calculated the statistics for each team and each game.

## 2.4 Defense Response to Scoring Opportunity (DRSO)

In this section, using the EF-OBSO explained in section 2.3, we describe another core idea of our proposed method (DRSO) in the following subsection.

### 2.4.1 The algorithm of DRSO

DRSO is a method of using OBSO to identify the best position for a defender and to evaluate team defenses using counterfactuals. We showed a detailed flowchart in Figure 4. In the following, we explain the details of the DRSO algorithm. As an overview, we calculated EF-OBSO assuming that a defender was present at several points and determined the optimal position of that defender by comparing the maximum value of EF-OBSO at some candidates with the one of the original EF-OBSO. We showed the flow of the algorithm in Figure 5. First, we calculated the maximum value of observed EF-OBSO

(denoted by $\text{EF-OBSO}_{\text{max\_obs}}$) at an event that occurred in the attacking third and identified the point on the pitch where this value was taken (Figure 5(a)). Next, we extracted the defender closest to that point and the grid on which that defender was (Figure 5(b)). So far, we performed the above operations based on observed data. Third, assuming that the defender was at the coordinates of the four vertices of this grid (for efficient computing; ideally, we can consider more locations), we calculated counterfactual EF-OBSO for each of them and the maximum value (denoted by $\text{EF-OBSO}_{\text{max\_obs},k} (k = 1, \dots, 4)$). We compared $\text{EF-OBSO}_{\text{max\_obs},k}$ with $\text{EF-OBSO}_{\text{max\_obs}}$ and regarded the point where $\min(\text{EF-OBSO}_{\text{max\_cf},k}, \text{EF-OBSO}_{\text{max\_obs}})$ was taken as the optimal position for this defender (Figure 5(c)).

This method has two advantages: first, we can present the optimal position for each defender. Players continuously change their positioning during the match to the team's advantage. Hence, the defensive players must be well organized as a team, as they need to block and defend to make it difficult for the opposing team to attack. Therefore, the optimal positioning of players other than oneself can support decision-making regarding where you should be.

Second, it does not use machine learning or deep learning methods. In recent years, more and more teams have been using AI techniques to analyze sports. While this has made it possible to explore a wide range of data, one of its weaknesses is its low interpretability, and there are still many challenges for practical application. Therefore, this method will give players and coaches more helpful advice.

## 2.4.2 Evaluation of team defense with DRSO

This section describes how the DRSO can be used to evaluate team defense. First, since the player information in the data used in this study is incomplete and varied, the DRSO calculation may not be possible concerning all defenders. Therefore, it is necessary to determine the number of defenders to which this algorithm is applied. In this study, we applied this method to the three defenders closest to the point of maximum EF-OBSO. Next, we used the difference (denoted by ) between and to evaluate team defenses. For example, in Figure 5(c), Diff_opt is 0.109, and Diff_obs is 0.133. In this case, we used the difference -0.024 as. The smaller this value is, the closer the defender's position in the original data was to the optimal one. Hence, we were able to interpret that the player defended well. Next, we used the average value of this operation performed on the three defenders to evaluate the defending team at this event. We calculated this manipulation for each event in the attacking third and averaged the values per team. Using this value per team, we compared the total number of concedes per team and the differences between the 2021-22 season and the 2022-23 season.

# Results

## 3.1 Verification of EF-OBSO

We presented the results of the EF-OBSO validation in Table 1, where conditions 1 to 5 represent the conditions described in Section 2.3.3. First, for all attackers, condition 5 had the smallest values for the mean and standard deviation (Std) of the RMSE. In particular, for scorers, the mean of the RMSE for the five conditions had the smallest value for the team-by-team and game-by-game comparison. Second, for non-scorers, the mean of the RMSE was the smallest in condition 2, both for the team-by-team and game-by-game comparison. Based on this result, we calculated the DRSO method on condition 5 because the means of All attackers and scores were the smallest. Predicting scorers with fewer samples would be more complicated than that for non-scorers with more samples.

Table 1. The result of verification of EF-OBSO.

<table>
  <thead>
    <tr>
      <th></th>
      <th colspan="2">All attackers</th>
      <th colspan="4">scorers</th>
      <th colspan="4">non-scorers</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td></td>
      <td colspan="2">All attackers</td>
      <td colspan="2">teams</td>
      <td colspan="2">games</td>
      <td colspan="2">teams</td>
      <td colspan="2">games</td>
    </tr>
    <tr>
      <td>(Unit:
m/s)</td>
      <td>Mean
<sub>( 10<sup>-1</sup>)</sub></td>
      <td>Std
<sub>( 10<sup>-1</sup>)</sub></td>
      <td>Mean
<sub>( 10<sup>-1</sup>)</sub></td>
      <td>Std
<sub>( 10<sup>-2</sup>)</sub></td>
      <td>Mean
<sub>( 10<sup>-1</sup>)</sub></td>
      <td>Std
<sub>( 10<sup>-2</sup>)</sub></td>
      <td>Mean
<sub>( 10<sup>-1</sup>)</sub></td>
      <td>Std
<sub>( 10<sup>-3</sup>)</sub></td>
      <td>Mean
<sub>( 10<sup>-1</sup>)</sub></td>
      <td>Std
<sub>( 10<sup>-2</sup>)</sub></td>
    </tr>
    <tr>
      <td>Ⅰ. All
0.0</td>
      <td>1.246</td>
      <td>1.171</td>
      <td>8.456</td>
      <td>1.096</td>
      <td>8.424</td>
      <td>6.723</td>
      <td>1.214</td>
      <td>5.598</td>
      <td>1.192</td>
      <td>2.076</td>
    </tr>
    <tr>
      <td colspan="11">( Fixing speeds of actor and defensive GK, other players’ speeds were)</td>
    </tr>
    <tr>
      <td>Ⅱ. 0.0</td>
      <td>1.245</td>
      <td>1.172</td>
      <td>8.472</td>
      <td>1.232</td>
      <td>8.445</td>
      <td>5.420</td>
      <td>1.200</td>
      <td>5.420</td>
      <td>1.181</td>
      <td>1.978</td>
    </tr>
    <tr>
      <td>Ⅲ. 4.0</td>
      <td>1.196</td>
      <td>1.153</td>
      <td>8.330</td>
      <td>1.032</td>
      <td>8.300</td>
      <td>5.864</td>
      <td>1.340</td>
      <td>6.678</td>
      <td>1.320</td>
      <td>1.955</td>
    </tr>
    <tr>
      <td>Ⅳ. 4.5</td>
      <td>1.190</td>
      <td>1.150</td>
      <td>8.303</td>
      <td>1.057</td>
      <td>8.269</td>
      <td>5.990</td>
      <td>1.365</td>
      <td>6.853</td>
      <td>1.344</td>
      <td>1.985</td>
    </tr>
    <tr>
      <td>Ⅴ. 5.0</td>
      <td>1.186</td>
      <td>1.147</td>
      <td>8.276</td>
      <td>1.097</td>
      <td>8.236</td>
      <td>6.190</td>
      <td>1.389</td>
      <td>6.978</td>
      <td>1.368</td>
      <td>2.019</td>
    </tr>
  </tbody>
</table>

StatsBomb Conference 2023 logo

## 3.2 Evaluation of team defense with DRSO

<table>
  <thead>
    <tr>
        <th>Team</th>
        <th>The total number of concedes</th>
        <th>Average Diff per defense</th>
    </tr>
  </thead>
  <tbody>
    <tr>
        <td>Everton</td>
        <td>61</td>
        <td>-0.0395</td>
    </tr>
    <tr>
        <td>Arsenal</td>
        <td>62</td>
        <td>-0.0415</td>
    </tr>
    <tr>
        <td>Brentford</td>
        <td>49</td>
        <td>-0.0428</td>
    </tr>
    <tr>
        <td>Brighton &amp; Hove Albion</td>
        <td>54</td>
        <td>-0.0445</td>
    </tr>
    <tr>
        <td>Chelsea</td>
        <td>49</td>
        <td>-0.0455</td>
    </tr>
    <tr>
        <td>Manchester United</td>
        <td>64</td>
        <td>-0.0455</td>
    </tr>
    <tr>
        <td>Manchester City</td>
        <td>31</td>
        <td>-0.0482</td>
    </tr>
    <tr>
        <td>Tottenham Hotspur</td>
        <td>53</td>
        <td>-0.0478</td>
    </tr>
    <tr>
        <td>Liverpool</td>
        <td>43</td>
        <td>-0.0488</td>
    </tr>
    <tr>
        <td>Leeds United</td>
        <td>94</td>
        <td>-0.0518</td>
    </tr>
  </tbody>
</table>

Figure 6: The figure of evaluation result about the mean of $Diff_{opt-obs}$ for the total number of concedes per team. The horizontal axis represents the total number of concedes for each team and the vertical axis represents average $Diff_{opt-obs}$ per defensive event for each team.

We then presented the results of our evaluation of team defenses using the indicators calculated by the DRSO. Figure 6 shows the relationship between the total number of concedes and the average for each team when they played nine other teams. The smaller the number of concedes (horizontal axis), the better the defense; the higher the average (vertical axis), the better the defense. This figure described Everton as having the best average $Diff_{opt-obs}$ and Leeds United as the worst. Manchester City also had the lowest number of concedes in games against the other nine teams, but their $Diff_{opt-obs}$ was not high.

Figure 7 shows the differences between the 2021-22 and 2022-23 seasons for the ten teams assessed in this study against the other nine teams. The higher the value on the vertical axis, the better defense we could consider. The blue areas in the diagram also

<table>
  <thead>
    <tr>
        <th>Teams</th>
        <th>2021-22</th>
        <th>2022-23</th>
    </tr>
  </thead>
  <tbody>
    <tr>
        <td>Arsenal</td>
        <td>-0.042</td>
        <td>-0.041</td>
    </tr>
    <tr>
        <td>Brentford</td>
        <td>-0.042</td>
        <td>-0.044</td>
    </tr>
    <tr>
        <td>Brighton &amp; Hove Albion</td>
        <td>-0.044</td>
        <td>-0.044</td>
    </tr>
    <tr>
        <td>Chelsea</td>
        <td>-0.044</td>
        <td>-0.046</td>
    </tr>
    <tr>
        <td>Everton</td>
        <td>-0.040</td>
        <td>-0.037</td>
    </tr>
    <tr>
        <td>Leeds United</td>
        <td>-0.055</td>
        <td>-0.051</td>
    </tr>
    <tr>
        <td>Liverpool</td>
        <td>-0.047</td>
        <td>-0.049</td>
    </tr>
    <tr>
        <td>Manchester City</td>
        <td>-0.042</td>
        <td>-0.052</td>
    </tr>
    <tr>
        <td>Manchester United</td>
        <td>-0.046</td>
        <td>-0.045</td>
    </tr>
    <tr>
        <td>Tottenham Hotspur</td>
        <td>-0.047</td>
        <td>-0.047</td>
    </tr>
  </tbody>
</table>

Figure 7. The figure of evaluation result about the difference of the average $Diff_{opt-obs}$ between the 2021-22 season and the 2022-23 season. The horizontal axis represents 10 teams evaluated in this study and the vertical axis represents average $Diff_{opt-obs}$ per defensive event for each season.

indicated that the defense improved in the following season. It shows that Arsenal, Leeds United, and Manchester United improved their defenses against the other nine teams. Conversely, where the orange area was visible, it indicated that the defense deteriorated in the following season. It means Brentford, Chelsea, Everton, Liverpool, and Manchester City worsened their defenses. on the vertical axis, the better defense we could consider. The blue areas in the diagram also indicated that the defense improved in the following season. Results show that Arsenal, Leeds United, and Manchester United improved their defenses against the other nine teams. Conversely, where the orange area was visible, it indicated that the defense deteriorated in the following season. This means that Brentford, Chelsea, Everton, Liverpool, and Manchester City deteriorated their defenses.

## Discussion

In this section, we first discuss the verification results of EF-OBSO concerning scorers and non-scorers. Next, we discuss the relevance of the DRSO-based team defense evaluation to real-game situations with several teams as examples.

We show the result of the verification of EF-OBSO in Table 1. According to this table, for scorers, we found the means of RMSE on all conditions to have more significant errors

than those of non-scorers. This indicates that the locations of higher EF-OBSO values were few. As shown in Equation 1, OBSO is a multiple of three types of probabilities. Therefore, even if each probability is high, the value of that multiplication is small. In addition, the value of S<sub>r</sub>, the scoring probability, decreases rapidly with distance [6]. Therefore, the value of EF-OBSO for a mid-shot from outside the penalty area is minimal compared to that of EF-OBSO for a shot inside the penalty area. However, it was unclear whether we could obtain similar values in the original paper. It is, therefore, necessary to verify whether this result is also accepted for EF-OBSO using the complete data.

In addition, we found that the mean of the RMSE for non-scorers is close to zero, but the error is still present. This indicates that even with high EF-OBSO values, it is difficult to obtain the goal. Indeed, for the original data before data preprocessing, a ratio of goals to shots of 0.1156 was obtained, which is close to the average of the RMSE for non-scorers.

Next, we discuss the team defense evaluation using the DRSO. First, concerning Figure 6, which shows the relationship with concedes, Everton was judged to defend the best in the DRSO model. This may be related to the team's tactics. In both the 2021-22 and 2022-23 seasons, Everton had an average possession rate of less than 40% against nine other teams [24, 25]. This means that they often defended their half of the field. Everton is not necessarily stronger than the other nine teams, and the players on that team know that they will be on the defensive. Therefore, they place more people within their own team's defensive third to prevent them from scoring when they are defending. Hence, we regard the areas with high EF-OBSO for off-ball attackers as few in the attacking third. Therefore, the result suggests a slight difference between the EF-OBSO values in the optimum position and the original values.

On the other hand, Manchester City had an average possession rate of over 60% against nine other teams in both the 2021-22 and 2022-23 seasons [25, 26]. This means that they have more effort when attacking and less effort when defending, which increases the areas where the opponent's EF-OBSO value was higher. Hence, the EF-OBSO values when the players would have been in the optimal position had an enormous difference compared to the original values. Therefore, the result showed that the value could have been better even if the total number of concedes was the fewest. These considerations suggest improving the DRSO method weighted by the time or number of events invaded within the attacking third for the opposing side.

Concerning Figure 7, the DRSO model explains that Arsenal, Leeds United, and Manchester United defended better in the 2022-23 season than in the 2021-22 season against nine other teams. All three of these teams also conceded fewer total goals. Leeds United, in particular, had the most improved Diff value with a significant reduction from 56 to 40 points. However, in both the 2021-22 and 2022-23 seasons, Diff had the lowest values of the ten teams, suggesting that despite the improvement, they were still inferior to the other teams in defense. This could be the reason why the team was one of the teams relegated to the EFL Championship for the 2022-23 season.

Furthermore, the DRSO model explains that Brentford, Chelsea, Everton, Liverpool, and Manchester City defended worse in the 2022-23 season than in the 2021-22 season

StatsBomb Conference 2023 logo

against nine other teams. Chelsea, Liverpool, and Manchester City also conceded more. Liverpool, in particular, saw a significant increase from 16 to 27 concedes. However, Brentford and Everton saw their total number of concedes decrease. In particular, Brentford saw a significant reduction from 30 to 18 concedes. The decline of concedes of the two teams suggests that aspects other than defensive positioning may have improved. Indeed, in the case of Brentford, their save percentage against nine other teams improved from 64/93 in the 2021-22 season to 90/108 in the 2022-23 season [28, 29]. Similarly, in the case of Everton, the percentage improved from 62/92 in the 2021-22 season to 63/90 in the 2022-23 season [30, 31]. These two suggest that, for keepers, we should look at improvements in saving aspects other than positioning.

---

## Conclusion

In this study, we proposed two methodologies. The first is EF-OBSO (Event and Freeze frame based OBSO). This method makes it possible to compute OBSO even with incomplete data by limiting the range of computation. We hope that everyone will use more accessible data and evaluate off-ball players. The second is DRSO (Defense Response to Scoring Opportunities). This method makes it possible to compute a defender's optimal positioning by dividing the pitch into grids. We expect that this method helps support players in their decision-making.

Finally, we introduce the limitations of this study and future perspectives. The first is about comparison with the case of using complete location data. Since the data in this study only includes some players' information, the verification result does not perfectly describe the performance. Another issue is that we needed to verify the parameters of our method more. To calculate EF-OBSO, we set the parameters such as players' speeds. However, since these values were not necessarily correct, it is necessary to verify them more in the future. The third is improvements to the DRSO algorithm. In this study, we defined players' counterfactual positionings as vertices of the grid in which the player was present. However, we did not verify whether the player could reliably have been at the vertices. Therefore, an improvement could be to, for example, represent the range in which the player could move based on their speed as a circle and sample one point within that range.

---

## References

[1] Decroos, Tom, et al. "Actions speak louder than goals: Valuing player actions in soccer." *Proceedings of the 25th ACM SIGKDD international conference on knowledge discovery & data mining*. 2019.

[2] Bransen, Lotte, Jan Van Haaren, and Michel van de Velden. "Measuring soccer players’ contributions to chance creation by valuing their passes." *Journal of Quantitative Analysis in Sports* 15.2 (2019): 97-116.

[3] Liu, Guiliang, et al. "Deep soccer analytics: learning an action-value function for evaluating soccer players." *Data Mining and Knowledge Discovery* 34 (2020): 1531-1559.

[4] Rudd, Sarah. "A framework for tactical analysis and individual offensive production assessment in soccer using markov chains." *New England symposium on statistics in sports*. 2011.

[5] Power, Paul, et al. "Not all passes are created equal: Objectively measuring the risk and reward of passes in soccer from tracking data." *Proceedings of the 23rd ACM SIGKDD international conference on knowledge discovery and data mining*. 2017.

[6] Spearman, William. "Beyond expected goals." *Proceedings of the 12th MIT sloan sports analytics conference*. 2018.

[7] Teranishi, Masakiyo, et al. "Evaluation of creating scoring opportunities for teammates in soccer via trajectory prediction." *International Workshop on Machine Learning and Data Mining for Sports Analytics*. Cham: Springer Nature Switzerland, 2022.

[8] Llana, Sergio, et al. "The right place at the right time: Advanced off-ball metrics for exploiting an opponent’s spatial weaknesses in soccer." *Proceedings of the 14th MIT Sloan Sports Analytics Conference*. 2020.

[9] Fernandez, Javier, and Luke Bornn. "Wide Open Spaces: A statistical technique for measuring space creation in professional soccer." *Sloan sports analytics conference*. Vol. 2018. 2018.

[10] Nakahara, Hiroshi, et al. "Action valuation of on-and off-ball soccer players based on multi-agent deep reinforcement learning." *arXiv preprint arXiv:2305.17886* (2023).

[11] Toda, Kosuke, et al. "Evaluation of soccer team defense based on prediction models of ball recovery and being attacked: A pilot study." *Plos one* 17.1 (2022): e0263051.

[12] Stöckl, Michael, et al. "Making offensive play predictable-using a graph convolutional network to understand defensive performance in soccer." *Proceedings of the 15th MIT sloan sports analytics conference*. Vol. 2022. 2021.

[13] Kubayi, Alliance, and Paul Larkin. "Analysis of teams’ corner kicks defensive strategies at the FIFA World Cup 2018." *International Journal of Performance Analysis in Sport* 19.5 (2019): 809-819.

[14] Merckx, Simon, et al. "Measuring the effectiveness of pressing in soccer." *Proceedings of the Workshop on Machine Learning and Data Mining for Sports Analytics, Virtual*. Vol. 13. 2021.

[15] Van Roy, Maaike, Pieter Robberechts, and Jesse Davis. "Optimally Disrupting Opponent Build-ups." *StatsBomb Conference*. 2021.

[16] Robberechts, Pieter, Maaike Van Roy, and Jesse Davis. "un-xPass: Measuring Soccer Player's Creativity." *Proceedings of the 29th ACM SIGKDD Conference on Knowledge Discovery and Data Mining*. 2023.

[17] J. Yorke, “StatsBomb 360: Exploring Line Breaking Passes,” StatsBomb | Data Champions, Jun. 28, 2022.

StatsBomb Conference 2023 logo

https://statsbomb.com/articles/soccer/statsbomb-360-exploring-line-breaking-passes/ (accessed Sep. 4, 2023).

[18] Rahimian, Pegah, et al. "Let’s Penetrate the Defense: A Machine Learning Model for Prediction and Valuation of Penetrative Passes." *International Workshop on Machine Learning and Data Mining for Sports Analytics*. Cham: Springer Nature Switzerland, 2022.

[19] Umemoto, Rikuhei, Kazushi Tsutsui, and Keisuke Fujii. "Location analysis of players in UEFA EURO 2020 and 2022 using generalized valuation of defense by estimating probabilities." *arXiv preprint arXiv:2212.00021* (2022).

[20] Furbino Marques do Nascimento, Ricardo, and Hugo MR Rios-Neto. "Generalized Action-based Ball Recovery Model using 360$^\circ$ data." *arXiv e-prints* (2023): arXiv-2307.

[21] Spearman, William, et al. "Physics-based modeling of pass probabilities in soccer." *Proceeding of the 11th MIT Sloan Sports Analytics Conference*. 2017.

[22] Professional HQ, “How Fast Are Pro Soccer Players? We Pulled the Data (2022)”, https://professionalshq.com/how-fast-are-pro-soccer-players-we-pulled-the-data-2022/ (accessed Sep. 21, 2023).

[23] Kern Campbell, "How Fast Do Soccer Players Run?", GAMEDAY CULTURE, https://gamedayculture.com/how-fast-do-soccer-players-run/ (accessed Sep. 21, 2023).

[24] FBref.com: Football Statistics and History, https://fbref.com/en/squads/d3fd31cc/2022-2023/matchlogs/c9/possession/Everton-Match-Logs-Premier-League (accessed Sep. 25, 2023)

[25] FBref.com: Football Statistics and History, https://fbref.com/en/squads/d3fd31cc/2021-2022/matchlogs/c9/possession/Everton-Match-Logs-Premier-League (accessed Sep. 25, 2023)

[26] FBref.com: Football Statistics and History, https://fbref.com/en/squads/b8fd03ef/2022-2023/matchlogs/c9/possession/Manchester-City-Match-Logs-Premier-League (accessed Sep. 25, 2023)

[27] FBref.com: Football Statistics and History, https://fbref.com/en/squads/b8fd03ef/2021-2022/matchlogs/c9/possession/Manchester-City-Match-Logs-Premier-League (accessed Sep. 25, 2023)

[28] FBref.com: Football Statistics and History, https://fbref.com/en/squads/cd051869/2021-2022/matchlogs/c9/keeper/Brentford-Match-Logs-Premier-League (accessed Sep. 25, 2023)

[29] FBref.com: Football Statistics and History, https://fbref.com/en/squads/cd051869/2022-2023/matchlogs/c9/keeper/Brentford-Match-Logs-Premier-League (accessed Sep. 25, 2023)

[30] FBref.com: Football Statistics and History, https://fbref.com/en/squads/d3fd31cc/2021-2022/matchlogs/c9/keeper/Everton-Match-Logs-Premier-League (accessed Sep. 25, 2023)

[31] FBref.com: Football Statistics and History, https://fbref.com/en/squads/d3fd31cc/2022-2023/matchlogs/c9/keeper/Everton-Match-Logs-Premier-League (accessed Sep. 25, 2023)

StatsBomb Conference 2023 logo