# A Multiresolution Stochastic Process Model for Predicting Basketball Possession Outcomes

Daniel Cervone<sup>1</sup>, Alex D’Amour<sup>2</sup>, Luke Bornn<sup>3</sup>, and Kirk Goldsberry<sup>4</sup>

<sup>1</sup>Center for Data Science, New York University, New York, NY 10003

<sup>2</sup>Department of Statistics, Harvard University, Cambridge, MA 02138

<sup>3</sup>Department of Statistics and Actuarial Science, Simon Fraser University, Burnaby, BC, Canada

<sup>4</sup>Institute for Quantitative Social Science, Harvard University, Cambridge, MA 02138

**Author’s Footnote:**

Daniel Cervone (email: dcervone@nyu.edu) is Moore-Sloan Data Science Fellow at New York University. Alex D’Amour (damour@fas.harvard.edu) is PhD candidate, Statistics Department, Harvard University. Luke Bornn (lbornn@sfu.ca) is Assistant Professor, Department of Statistics and Actuarial Science, Simon Fraser University. Kirk Goldsberry (kgoldsberry@fas.harvard.edu) is Visiting Scholar, Center for Geographic Analysis, Harvard University. Daniel Cervone is funded by a fellowship from the Alfred P. Sloan and Betty Moore foundations. Luke Bornn is funded by DARPA Award FA8750-14-2-0117, ARO Award W911NF-15-1-0172, an Amazon AWS Research Grant, and the Natural Sciences and Engineering Research Council of Canada.

The authors would like to thank Alex Franks, Andrew Miller, Carl Morris, Natesh Pillai, and Edoardo Airoldi for helpful comments, as well as STATS LLC in partnership with the NBA for providing the optical tracking data. The computations in this paper were run on the Odyssey cluster supported by the FAS Division of Science, Research Computing Group at Harvard University.

# Abstract

Basketball games evolve continuously in space and time as players constantly interact with their teammates, the opposing team, and the ball. However, current analyses of basketball outcomes rely on discretized summaries of the game that reduce such interactions to tallies of points, assists, and similar events. In this paper, we propose a framework for using optical player tracking data to estimate, in real time, the expected number of points obtained by the end of a possession. This quantity, called *expected possession value* (EPV), derives from a stochastic process model for the evolution of a basketball possession. We model this process at multiple levels of resolution, differentiating between continuous, infinitesimal movements of players, and discrete events such as shot attempts and turnovers. Transition kernels are estimated using hierarchical spatiotemporal models that share information across players while remaining computationally tractable on very large data sets. In addition to estimating EPV, these models reveal novel insights on players’ decision-making tendencies as a function of their spatial strategy. A data sample and R code for further exploration of our model/results are available in the repository [https://github.com/dcervone/EPVDemo](https://github.com/dcervone/EPVDemo).

KEYWORDS: Optical tracking data, spatiotemporal model, competing risks, Gaussian process.

# 1. INTRODUCTION

Basketball is a fast-paced sport, free-flowing in both space and time, in which players’ actions and decisions continuously impact their teams’ prospective game outcomes. Team owners, general managers, coaches, and fans all seek to quantify and evaluate players’ contributions to their team’s success. However, current statistical models for player evaluation such as “Player Efficiency Rating” (Hollinger 2005) and “Adjusted Plus/Minus” (Omidiran 2011) rely on highly reductive summary statistics of basketball games such as points scored, rebounds, steals, assists—the so-called “box score” summary of the game. Such models reflect the fact that up until very recently, data on basketball games were only available in this low level of resolution. Thus previous statistical analyses of basketball performance have overlooked many of the high-resolution motifs—events not measurable by such aggregate statistics—that characterize basketball strategy. For instance, traditional analyses cannot estimate the value of a clever move that fools the defender, or the regret of skipping an open shot in favor of passing to a heavily defended teammate. The advent of player tracking data in the NBA has provided an opportunity to fill this gap.

## 1.1 Player-Tracking Data

In 2013 the National Basketball Association (NBA), in partnership with data provider STATS LLC, installed optical tracking systems in the arenas of all 30 teams in the league. The systems track the exact two-dimensional locations of every player on the court (as well as the three-dimensional location of the ball) at a resolution of 25Hz, yielding over 1 billion space-time observations over the course of a full season.

Consider, for example, the following possession recorded using this player tracking system. This is a specific Miami Heat possession against the Brooklyn Nets from the second quarter of a game on November 1, 2013, chosen arbitrarily among those during which LeBron James (widely considered the best NBA player as of 2014) handles the ball. In this particular

Diagram showing eight panels (A-H) of a basketball possession with player movements and legends for Offense (Norris Cole, Ray Allen, Rashard Lewis, LeBron James, Chris Bosh) and Defense (Deron Williams, Jason Terry, Joe Johnson, Andray Blatche, Brook Lopez).

Figure 1: Miami Heat possession against Brooklyn Nets. Norris Cole wanders into the perimeter (A) before driving toward the basket (B). Instead of taking the shot, he runs underneath the basket (C) and eventually passes to Rashard Lewis (D), who promptly passes to LeBron James (E). After entering the perimeter (F), James slips behind the defense (G) and scores an easy layup (H).

possession, diagrammed in Figure 1, point guard Norris Cole begins with possession of the ball crossing the halfcourt line (panel A). After waiting for his teammates to arrive in the offensive half of the court, Cole wanders gradually into the perimeter (inside the three point line), before attacking the basket through the left post. He draws two defenders, and while he appears to beat them to the basket (B), instead of attempting a layup he runs underneath the basket through to the right post (C). He is still being double teamed and at this point passes to Rashard Lewis (D), who is standing in the right wing three position. As defender Joe Johnson closes, Lewis passes to LeBron James, who is standing about 6 feet beyond the three point line and drawing the attention of Andray Blatche (E). James wanders slowly into the perimeter (F), until just behind the free throw line, at which point he breaks towards the basket. His rapid acceleration (G) splits the defense and gains him a clear lane to the basket. He successfully finishes with a layup (H), providing the Heat two points.

## 1.2 Expected Possession Value

Such detailed data hold both limitless analytical potential for basketball spectators and new methodological challenges to statisticians. Of the dizzying array of questions that could be asked of such data, we choose to focus this paper on one particularly compelling quantity of interest, which we call *expected possession value* (EPV), defined as the expected number of points the offense will score on a particular possession conditional on that possession’s evolution up to time $t$.

For illustration, we plot the EPV curve corresponding to the example Heat possession in Figure 2, with EPV estimated using the methodology in this paper. We see several moments when the expected point yield of the possession, given its history, changes dramatically. For the first 2 seconds of the possession, EPV remains around 1. When Cole drives toward the basket, EPV rises until peaking at around 1.34 when Cole is right in front of the basket. As Cole dribbles past the basket (and his defenders continue pursuit), however, EPV falls rapidly, bottoming out at 0.77 before “resetting” to 1.00 with the pass to Rashard Lewis.

The EPV increases slightly to 1.03 when the ball is then passed to James. As EPV is sensitive to small changes in players’ exact locations, we see EPV rise slightly as James approaches the three point line and then dip slightly as he crosses it. Shortly afterwards, EPV rises suddenly as James breaks towards the basket, eluding the defense, and continues rising until he is beneath the basket, when an attempted layup boosts the EPV from 1.52 to 1.62.

<table>
  <thead>
    <tr>
        <th>time (sec)</th>
        <th>EPV</th>
        <th>Possession</th>
        <th>Annotation</th>
    </tr>
  </thead>
  <tbody>
    <tr>
        <td>0</td>
        <td>1.0</td>
        <td>NORRIS COLE POSSESSION</td>
        <td> </td>
    </tr>
    <tr>
        <td>0.8</td>
        <td>1.0</td>
        <td>NORRIS COLE POSSESSION</td>
        <td>Slight dip in EPV after crossing 3 point line</td>
    </tr>
    <tr>
        <td>1.2</td>
        <td>1.0</td>
        <td>NORRIS COLE POSSESSION</td>
        <td> </td>
    </tr>
    <tr>
        <td>2.0</td>
        <td>1.1</td>
        <td>NORRIS COLE POSSESSION</td>
        <td>Accelerates towards basket</td>
    </tr>
    <tr>
        <td>2.4</td>
        <td>1.3</td>
        <td>NORRIS COLE POSSESSION</td>
        <td> </td>
    </tr>
    <tr>
        <td>3.0</td>
        <td>1.0</td>
        <td>NORRIS COLE POSSESSION</td>
        <td>Runs behind basket and defenders close</td>
    </tr>
    <tr>
        <td>3.2</td>
        <td>0.8</td>
        <td>NORRIS COLE POSSESSION</td>
        <td>Pass</td>
    </tr>
    <tr>
        <td>3.4</td>
        <td>1.0</td>
        <td>RASHARD LEWIS POSS.</td>
        <td>EPV constant while pass is en route</td>
    </tr>
    <tr>
        <td>4.2</td>
        <td>1.0</td>
        <td>RASHARD LEWIS POSS.</td>
        <td>Pass</td>
    </tr>
    <tr>
        <td>4.8</td>
        <td>1.0</td>
        <td>LEBRON JAMES POSSESSION</td>
        <td>Slight dip in EPV after crossing 3 point line</td>
    </tr>
    <tr>
        <td>5.6</td>
        <td>1.1</td>
        <td>LEBRON JAMES POSSESSION</td>
        <td>Accelerates into the paint</td>
    </tr>
    <tr>
        <td>6.0</td>
        <td>1.2</td>
        <td>LEBRON JAMES POSSESSION</td>
        <td>Splits the defense; clear path to basket</td>
    </tr>
    <tr>
        <td>7.2</td>
        <td>1.6</td>
        <td>LEBRON JAMES POSSESSION</td>
        <td>James shot</td>
    </tr>
  </tbody>
</table>

Figure 2: Estimated EPV over time for the possession shown in Figure 1. Changes in EPV are induced by changes in players’ locations and dynamics of motion; macrotransitions such as passes and shot attempts produce immediate, sometimes rapid changes in EPV. The black line slightly smooths EPV evaluations at each time point (gray dots), which are subject to Monte Carlo error.

In this way, EPV corresponds naturally to a coach’s or spectator’s sense of how the actions that basketball players take in continuous time help or hurt their team’s cause to score in the current possession, and quantifies this in units of expected points. EPV acts like a stock ticker, providing an instantaneous summary of the possession’s eventual point value given all available information, much like a stock price values an asset based on speculation of future expectations.

## 1.3 Related Work and Contributions

Concepts similar to EPV, where final outcomes are modeled conditional on observed progress, have had statistical treatment in other sports, such as in-game win probability in baseball (Bukiet, Harold & Palacios 1997; Yang & Swartz 2004) and football (Lock & Nettleton 2014), as well as in-possession point totals in football (Burke 2010; Goldner 2012). These previous efforts can be categorized into either marginal regression/classification approaches, where features of the current game state are mapped directly to expected outcomes, or process-based models that use a homogeneous Markov chain representation of the game to derive outcome distributions. Neither of these approaches is ideal for application to basketball. Marginal regression methodologies ignore the natural martingale structure of EPV, which is essential to its “stock ticker” interpretation. On the other hand, while Markov chain methodologies do maintain this “stock ticker” structure, applying them to basketball requires discretizing the data, introducing an onerous bias-variance-computation tradeoff that is not present for sports like baseball that are naturally discrete in time.

To estimate EPV effectively, we introduce a novel multiresolution approach in which we model basketball possessions at two separate levels of resolution, one fully continuous and one highly coarsened. By coherently combining these models we are able to obtain EPV estimates that are reliable, sensitive, stochastically consistent, and computationally feasible (albeit intensive). While our methodology is motivated by basketball, we believe that this research can serve as an informative case study for analysts working in other application areas where continuous monitoring data are becoming widespread, including traffic monitoring (Ihler, Hutchins & Smyth 2006), surveillance, and digital marketing (Shao & Li 2011), as well as other sports such as soccer and hockey (Thomas, Ventura, Jensen & Ma 2013).

Section 2 formally defines EPV within the context of a stochastic process for basketball, introducing the multiresolution modeling approach that make EPV calculations tractable as averages over future paths of a stochastic process. Parameters for these models, which represent players’ decision-making tendencies in various spatial and situational circumstances, are discussed in Section 3. Section 4 discusses inference for these parameters using hierchical models that share information between players and across space. We highlight results from actual NBA possessions in Section 5, and show how EPV provides useful new quantifications of player ability and skill. Section 6 concludes with directions for further work.

## 2. MULTIRESOLUTION MODELING

To present our process model for a basketball possession, we require some formalism. Let $\Omega$ represent the space of all possible basketball possessions in full detail, with $\omega \in \Omega$ describing the full path of a particular possession. For simplicity, we restrict our focus to possessions that do not include fouls<sup>1</sup>, so that each possession we consider results in 0, 2, or 3 points scored for the offense, denoted $X(\omega) \in \{0, 2, 3\}$. For any possession path $\omega$, we denote by $Z(\omega)$ the optical tracking time series generated by this possession so that $Z_t(\omega) \in \mathcal{Z}$, $t > 0$, is a "snapshot" of the tracking data exactly $t$ seconds from the start of the possession ($t = 0$). $\mathcal{Z}$ is a high dimensional space that includes $(x, y)$ coordinates for all 10 players on the court, $(x, y, z)$ coordinates for the ball, summary information such as which players are on the court and what the game situation is (game location, score, time remaining, etc.), and event annotations that are observable in real time, such as a turnover occurring, a pass, or a shot being attempted and the result of that attempt.

Taking the intuitive view of $\Omega$ as a sample space of possession paths, we model $Z(\omega)$ as a stochastic process, and likewise, define $Z_t(\omega)$ for each $t > 0$ as a random variable in $\mathcal{Z}$. $Z(\omega)$ provides the natural filtration $\mathcal{F}_t^{(Z)} = \sigma(\{Z_s^{-1} : 0 \leq s \leq t\})$, which represents all information available from the optical tracking data for the first $t$ seconds of a possession. EPV is the expected value of the number of points scored for the possession ($X$) given all available data up to time $t$ ($\mathcal{F}_t^{(Z)}$):

**Definition** The *expected possession value*, or EPV, at time $t \geq 0$ during a possession is $\nu_t = \mathbb{E}[X | \mathcal{F}_t^{(Z)}]$.

<sup>1</sup>Our data include foul events, but do not specify the type or circumstances of the foul. There are several types of fouls and game situations for which fouls lead to free throws—for instance, shooting fouls, technical/flagrant fouls, clear path fouls, and fouls during the fouling team’s “bonus” period; thus, modeling fouls presents additional complications relative to the other events we model in our EPV model. While drawing fouls can be a valuable and important part of team strategy, we omit modeling such behavior in this paper.

The expectation $\mathbb{E}[X|\mathcal{F}_t^{(Z)}]$ is an integral over the distribution of future paths the current possession can take. Letting $T(\omega)$ denote the time at which a possession following path $\omega$ ends<sup>2</sup>, the possession’s point total is a deterministic function of the full resolution data at this time, $X(\omega) = h(Z_{T(\omega)}(\omega))$. Thus, evaluating EPV amounts to integrating over the joint distribution of $(T, Z_T)$:

$$
\begin{aligned}
\nu_t = \mathbb{E}[X|\mathcal{F}_t^{(Z)}] &= \int_{\Omega} X(\omega)\mathbb{P}(d\omega|\mathcal{F}_t^{(Z)}) \\
&= \int_t^\infty \int_{\mathcal{Z}} h(z)\mathbb{P}(Z_s = z | T = s, \mathcal{F}_t^{(Z)})\mathbb{P}(T = s | \mathcal{F}_t^{(Z)})dzds.
\end{aligned}
\eqno(1)
$$

Note that we use probability notation $\mathbb{P}(\cdot)$ somewhat heuristically, as $\mathbb{P}(T = s|\mathcal{F}_t^{(Z)})$ is a density with respect to Lebesgue measure, while $Z_s$ mixes both discrete (annotations) and continuous (locations) components. We will also generally omit the dependence on $\omega$ when writing random variables, e.g., $Z_t$ instead of $Z_t(\omega)$.

## 2.1 Estimator Criteria

We have defined EPV in (1) as an unobserved, theoretical quantity; one could thus imagine many different EPV estimators based on different models and/or information in the data. However, we believe that in order for EPV to achieve its full potential as a basis for high-resolution player and strategy evaluation, an EPV estimator should meet several criteria.

First, we require that the EPV estimator be stochastically consistent. Recognizing that EPV is simply a conditional expectation, it is tempting to estimate EPV using a regression or classification approach that maps features from $\mathcal{F}_t^{(Z)}$ to an outcome space, $[0, 3]$ or $\{0, 2, 3\}$. Setting aside the fact that our data associate each possession outcome $X$ with process-valued inputs $Z$, and thus do not conform naturally to input/output structure of such models, such an approach cannot guarantee the estimator will have the (Kolmogorov) stochastic consistency inherent to theoretical EPV, which is essential to its “stock ticker” interpretation. Using a stochastically consistent EPV estimator guarantees that changes in the resulting EPV curve derive from players’ on-court actions, rather than artifacts or inefficiencies of the data analysis. A stochastic process model for the evolution of a basketball possession guarantees such consistency.

The second criterion that we require is that the estimator be sensitive to the fine-grained details of the data without incurring undue variance or computatonal complexity. Applying a Markov chain-based estimation approach would require discretizing the data by mapping the observed spatial configuration $Z_t$ into a simplified summary $C_t$, violating this criterion by trading potentially useful information in the player tracking data for computational tractability.

To develop methodology that meet both criteria, we note the information-computation tradeoff in current process modeling strategies results from choosing a single level of resolution at which to model the possession and compute all expectations. In contrast, our method for estimating EPV combines models for the possession at two distinct levels of resolution, namely, a fully continuous model of player movement and actions, and a Markov chain model for a highly coarsened view of the possession. This multiresolution approach

<sup>2</sup>The time of a possession is bounded, even for pathological examples, by the 12-minute length of a quarter; yet we do not leverage this fact and simply assume that possession lenghts are almost surely finite.

leverages the computational simplicity of a discrete Markov chain model while conditioning on exact spatial locations and high-resolution data features.

## 2.2 A Coarsened Process

The Markov chain portion of our method requires a coarsened view of the data. For all time $0 < t \leq T$ during a possession, let $C(\cdot)$ be a coarsening that maps $\mathcal{Z}$ to a finite set $\mathcal{C}$, and call $C_t = C(Z_t)$ the "state" of the possession. To make the Markovian assumption plausible, we populate the coarsened state space $\mathcal{C}$ with summaries of the full resolution data so that transitions between these states represent meaningful events in a basketball possession—see Figure 3 for an illustration.

Schematic of the coarsened possession process C, showing states for players, transitions like passes, turnovers, shots, and rebounds, and end states like made shots or end of possession.

Figure 3: *Schematic of the coarsened possession process $\mathcal{C}$, with states (rectangles) and possible state transitions (arrows) shown. The unshaded states in the first row compose $\mathcal{C}_{poss}$. Here, states corresponding to distinct ballhandlers are grouped together (Player 1 through 5), and the discretized court in each group represents the player's coarsened position and defended state. The gray shaded rectangles are transition states, $\mathcal{C}_{trans}$, while the rectangles in the third row represent the end states, $\mathcal{C}_{end}$. Blue arrows represent possible macrotransition entrances (and red arrows, macrotransition exits) when Player 1 has the ball; these terms are introduced in Section 3.*

First, there are 3 "bookkeeping" states, denoted $\mathcal{C}_{end}$, that categorize the end of the possession, so that $C_T \in \mathcal{C}_{end}$ and for all $t < T, C_t \notin \mathcal{C}_{end}$ (shown in the bottom row of Figure 3). These are $\mathcal{C}_{end} = \{\text{made 2 pt, made 3 pt, end of possession}\}$. These three states have associated point values of 2, 3, and 0, respectively (the generic possession end state can be reached by turnovers and defensive rebounds, which yield no points). This makes the possession point value $X$ a function of the final coarsened state $C_T$.

Next, whenever a player possesses the ball at time $t$, we assume $C_t = (\text{ballcarrier ID at } t) \times (\text{court region at } t) \times (\text{defended at } t)$, having defined seven disjoint regions of the court and classifying a player as defended at time $t$ by whether there is a defender within 5 feet of him. The possible values of $C_t$, if a player possesses the ball at time $t$, thus live in $\mathcal{C}_{poss} = \{\text{player ID}\} \times \{\text{region ID}\} \times \{\mathbf{1}[\text{defended}]\}$. These states are represented by the unshaded portion of the top row of Figure 3, where the differently colored regions of the court

diagrams reveal the court space discretization.

Finally, we define a set of states to indicate that an annotated basketball action from the full resolution data $Z$ is currently in progress. These "transition" states encapsulate constrained motifs in a possession, for example, when the ball is in the air traveling between players in a pass attempt. Explicitly, denote $\mathcal{C}_{\text{trans}} = \{\text{shot attempt from } c \in \mathcal{C}_{\text{poss}}, \text{ pass to } c' \in \mathcal{C}_{\text{poss}} \text{ from } c \in \mathcal{C}_{\text{poss}}, \text{ turnover in progress, rebound in progress}\}$ (listed in the gray shaded portions of Figure 3). These transition states carry information about the possession path, such as the most recent ballcarrier, and the target of the pass, while the ball is in the air during shot attempts and passes<sup>3</sup>. Note that, by design, a possession must pass through a state in $\mathcal{C}_{\text{trans}}$ in order to reach a state in $\mathcal{C}_{\text{end}}$. For simplicity and due to limitations of the data, this construction of $\mathcal{C} = \mathcal{C}_{\text{poss}} \cup \mathcal{C}_{\text{trans}} \cup \mathcal{C}_{\text{end}}$ excludes several notable basketball events (such as fouls, violations, and other stoppages in play) and aggregates others (the data, for example, does not discriminate among steals, intercepted passes, or lost balls out of bounds).

## 2.3 Combining Resolutions

We make several modeling assumptions about the processes $Z$ and $C$, which allow them to be combined into a coherent EPV estimator.

(A1) $C$ is marginally semi-Markov.

The semi-Markov assumption (A1) guarantees that the embedded sequence of disjoint possession states $C^{(0)}, C^{(1)}, \dots, C^{(K)}$ is a Markov chain, which ensures that it is straightforward to compute $E[X|C_t]$ using the associated transition probability matrix (Kemeny & Snell 1976).

Next, we specify the relationship between coarsened and full-resolution conditioning. This first requires defining two additional time points which mark changes in the future evolution of the possession:

$$ \tau_t = \begin{cases} \min\{s : s > t, C_s \in \mathcal{C}_{\text{trans}}\} & \text{if } C_t \in \mathcal{C}_{\text{poss}} \\ t & \text{if } C_t \notin \mathcal{C}_{\text{poss}} \end{cases} \quad (2) $$

$$ \delta_t = \min\{s : s \geq \tau_t, C_s \notin \mathcal{C}_{\text{trans}}\}. \quad (3) $$

Thus, assuming a player possesses the ball at time $t$, $\tau_t$ is the first time after $t$ he attempts a shot/pass or turns the ball over (entering a state in $\mathcal{C}_{\text{trans}}$), and $\delta_t$ is the endpoint of this shot/pass/turnover (leaving a state in $\mathcal{C}_{\text{trans}}$). We assume that passing through these transition states, $\mathcal{C}_{\text{trans}}$, *decouples* the future of the possession after time $\delta_t$ with its history up to time $t$:

(A2) For all $s > \delta_t$ and $c \in \mathcal{C}$, $P(C_s = c|C_{\delta_t}, \mathcal{F}_t^{(Z)}) = P(C_s = c|C_{\delta_t})$.

Intuitively, assumption (A2) states that for predicting coarsened states beyond some point in the future $\delta_t$, all information in the possession history up to time $t$ is summarized by the distribution of $C_{\delta_t}$. The dynamics of basketball make this assumption reasonable; when a player passes the ball or attempts a shot, this represents a structural transition in the basketball possession to which all players react. Their actions prior to this transition are

not likely to influence their actions after this transition. Given $C_{\delta_t}$—which, for a pass at $\tau_t$ includes the pass recipient, his court region, and defensive pressure, and for a shot attempt at $\tau_t$ includes the shot outcome—data prior to the pass/shot attempt are not informative of the possession’s future evolution.

Together, these assumptions yield a simplified expression for (1), which combines contributions from full-resolution and coarsened views of the process.

**Theorem 2.1** *Under assumptions (A1)–(A2), the full-resolution EPV $\nu_t$ can be rewritten:*

$$ \nu_t = \sum_{c \in \mathcal{C}} \mathbb{E}[X | C_{\delta_t} = c] \mathbb{P}(C_{\delta_t} = c | \mathcal{F}_t^{(Z)}). \tag{4} $$

**Remark** Although we have specified this result in terms of the specific coarsening defined in Section 2.2, we could substitute any coarsening for which (A1)–(A2) are well-defined and reasonably hold. We briefly discuss potential alternative coarsenings in Section 6.

The proof of Theorem 2.1, follows immediately from (A1)–(A2), and is therefore omitted. Heuristically, (4) expresses $\nu_t$ as the expectation given by a homogeneous Markov chain on $\mathcal{C}$ with a random starting point $C_{\delta_t}$, where only the starting point depends on the full-resolution information $\mathcal{F}_t^{(Z)}$. This result illustrates the multiresolution conditioning scheme that makes our EPV approach computationally feasible: the term $\mathbb{E}[X | C_{\delta_t} = c]$ is easy to calculate using properties of Markov chains, and $\mathbb{P}(C_{\delta_t} | \mathcal{F}_t^{(Z)})$ only requires forecasting the full-resolution data for a short period of time relative to (1), as $\delta_t \leq T$.

# 3. TRANSITION MODEL SPECIFICATION

The representation in Theorem (2.1) shows that estimating EPV does not require a full-blown model for entire basketball possessions at high resolution. Instead, the priority is to accurately predict the next major “decoupling” action in the possession, which we have denoted $C_{\delta_t}$. At this point, Equation (4) switches resolutions: $C_{\delta_t}$ depends on the full-resolution possession history $\mathcal{F}_t^{(Z)}$, after which our EPV estimate only depends on the coarsened state $C_{\delta_t}$. This section presents models that operate on these two distinct levels of resolution, using parameterizations that reflect players’ reactions to the situational, spatiotemporal predicaments they face on the basketball court.

First, using the full-resolution data, we need to predict $C_{\delta_t}$. We achieve this using three models; heuristically speaking, one predicts player movement in space while the ballcarrier remains constant, one predicts the occurrence of events (passes/shots/turnovers) that change the ballcarrier, and one predicts the outcome state ($C_{\delta_t}$) of such events.

Writing these models requires some additional notation. Fix $\epsilon > 0$, and for any $t \geq 0$ during the possession (here, we use 1/25 second since this is the temporal resolution of our data), and let $M(t)$ be the event $\{\tau_t \leq t + \epsilon\}$; for this, we say that a *macrotransition occurs* during $(t, t + \epsilon]$. Recalling the definition of $\tau_t$ (2), $M(t)$ is realized when the possession moves from $\mathcal{C}_{\text{poss}}$ to $\mathcal{C}_{\text{trans}}$, which represents the start of a pass, shot attempt, or turnover (and $M(t)$ is continuously realized throughout the duration of this action). We now define:

(M1) The *microtransition model*, $\mathbb{P}(Z_{t+\epsilon} | M(t)^c, \mathcal{F}_t^{(Z)})$, which describes infinitesimal player movement assuming that a major ball movement does *not* occur.

(M2) The *macrotransition entry model*, $\mathbb{P}(M(t)|\mathcal{F}_t^{(Z)})$, which describes the occurrence of a macrotransition (pass/shot/turnover) within the next $\epsilon$ time.

(M3) The *macrotransition exit model* $\mathbb{P}(C_{\delta_t}|M(t), \mathcal{F}_t^{(Z)})$, which gives the outcome of this macrotransition in $\mathcal{C}$.

(M1)–(M3) together allow us to sample from $\mathbb{P}(C_{\delta_t}|\mathcal{F}_t^{(Z)})$, as we alternate draws from (M1) and (M2) until a macrotransition occurs, and then use (M3) to sample the outcome state of this macrotransition. Thus, while models (M1)–(M3) condition on the full-resolution possession history $\mathcal{F}_t^{(Z)}$ in order to predict $C_{\delta_t}$, calculating EPV given $C_{\delta_t}$ only requires a model for transitions between coarsened states $\mathcal{C}$. Due to the Markov assumption (A1), this is easily summarized by a transition probability matrix:

(M4) The *Markov transition probability matrix* **P**, with $P_{qr} = \mathbb{P}(C^{(n+1)} = c_r|C^{(n)} = c_q)$.

Thus, (M1)–(M4) are sufficient to compute EPV using our multiresolution framework of Theorem 2.1. In the following subsections, we discuss each of these models in greater detail.

## 3.1 Microtransition Model

The microtransition model describes player movement with the ballcarrier held constant. In the periods between transfers of ball possession (including passes, shots, and turnovers), all players on the court move in order to influence the character of the next ball movement (macrotransition). For instance, the ballcarrier might drive toward the basket to attempt a shot, or move laterally to gain separation from a defender, while his teammates move to position themselves for passes or rebounds, or to set screens and picks. The defense moves correspondingly, attempting to deter easy shot attempts or passes to certain players while simultaneously anticipating a possible turnover. Separate models are assumed for offensive and defensive players, as we shall describe.

Predicting the motion of offensive players over a short time window is driven by the players’ dynamics (velocity, acceleration, etc.). Let the location of offensive player $\ell$ ($\ell \in \{1, \dots, L = 461\}$) at time $t$ be $\mathbf{z}^\ell(t) = (x^\ell(t), y^\ell(t))$. We then model movement in each of the $x$ and $y$ coordinates using

$$x^\ell(t + \epsilon) = x^\ell(t) + \alpha_x^\ell[x^\ell(t) - x^\ell(t - \epsilon)] + \eta_x^\ell(t) \eqno(5)$$

(and analogously for $y^\ell(t)$). This expression derives from a Taylor series expansion to the ballcarrier’s position for each coordinate, such that $\alpha_x^\ell[x^\ell(t) - x^\ell(t - \epsilon)] \approx \epsilon x^{\ell \prime}(t)$, and $\eta_x^\ell(t)$ provides stochastic innovations representing the contribution of higher-order derivatives (acceleration, jerk, etc.). Because they are driven to score, players’ dynamics on offense are nonstationary. When possessing the ball, most players accelerate toward the basket when beyond the three-point line, and decelerate when very close to the basket in order to attempt a shot. Also, players will accelerate away from the edges of the court as they approach these, in order to stay in bounds. To capture such behavior, we assume spatial structure for the innovations, $\eta_x^\ell(t) \sim \mathcal{N}(\mu_x^\ell(\mathbf{z}^\ell(t)), (\sigma_x^\ell)^2)$, where $\mu_x^\ell$ maps player $\ell$’s location on the court to an additive effect in (5), which has the interpretation of an acceleration effect; see Figure 4 for an example.

The defensive components of $\mathbb{P}(Z_{t+\epsilon}|M(t)^c, \mathcal{F}_t^{(Z)})$, corresponding to the positions of the five defenders, are easier to model conditional on the evolution of the offense’s positions.

Acceleration fields for Tony Parker and Dwight Howard with and without ball possession.

*Figure 4: Acceleration fields $(\mu_x(\mathbf{z}(t)), \mu_y(\mathbf{z}(t)))$ for Tony Paker (a)–(b) and Dwight Howard (c)–(d) with and without ball possession. The arrows point in the direction of the acceleration at each point on the court’s surface, and the size and color of the arrows are proportional to the magnitude of the acceleration. Comparing (a) and (c) for instance, we see that when both players possess the ball, Parker more frequently attacks the basket from outside the perimeter. Howard does not accelerate to the basket from beyond the perimeter, and only tends to attack the basket inside the paint.*

Following Franks, Miller, Bornn & Goldsberry (2015), we assume each defender’s position is centered on a linear combination of the basket’s location, the ball’s location, and the location of the offensive player he is guarding. Franks et al. (2015) use a hidden Markov model (HMM) based on this assumption to learn which offensive players each defender is guarding, such that conditional on defender $\ell$ guarding offender $k$ his location $\mathbf{z}^\ell(t) = (x^\ell(t), y^\ell(t))$ should be normally distributed with mean $\mathbf{m}_{\text{opt}}^k(t) = 0.62\mathbf{z}^k(t) + 0.11\mathbf{z}_{\text{bask}} + 0.27\mathbf{z}_{\text{ball}}(t)$.

Of course, the dynamics (velocity, etc.) of defensive players’ are still hugely informative for predicting their locations within a small time window. Thus our microtransition model for defender $\ell$ balances these dynamics with the mean path induced by the player he is guarding:

$$
\begin{aligned}
x^\ell(t + \epsilon) | m_{\text{opt},x}^k(t) \sim \mathcal{N} \bigg( &x^\ell(t) + a_x^\ell [x^\ell(t) - x^\ell(t - \epsilon)] \\
+ \: &b_x^\ell [m_{\text{opt},x}^k(t + \epsilon) - m_{\text{opt},x}^k(t)] + c_x^\ell [x^\ell(t) - m_{\text{opt},x}^k(t + \epsilon)], (\tau_x^\ell)^2 \bigg)
\end{aligned} \eqno(6)
$$

and symmetrically in $y$. Rather than implement the HMM procedure used in Franks et al. (2015), we simply assume each defender is guarding at time $t$ whichever offensive player $j$ yields the smallest residual $||\mathbf{z}^\ell(t) - \mathbf{m}_{\text{opt}}^j(t)||$, noting that more than one defender may be guarding the same offender (as in a “double team”). Thus, conditional on the locations of the offense at time $t + \epsilon$, (6) provides a distribution over the locations of the defense at $t + \epsilon$.

## 3.2 Macrotransition Entry Model

The macrotransition entry model $\mathbb{P}(M(t)|\mathcal{F}_t^{(Z)})$ predicts ball movements that instantaneously shift the course of the possession—passes, shot attempts, and turnovers. As such, we consider a family of macrotransition entry models $\mathbb{P}(M_j(t)|\mathcal{F}_t^{(Z)})$, where $j$ indexes the type of macrotransition corresponding to $M(t)$. There are six such types: four pass options (indexed, without loss of generality, $j \leq 4$), a shot attempt ($j = 5$), or a turnover ($j = 6$). Thus, $M_j(t)$ is the event that a macrotransition of type $j$ begins in the time window

$(t, t + \epsilon]$, and $M(t) = \bigcup_{j=1}^6 M_j(t)$. Since macrotransition types are disjoint, we also know $\mathbb{P}(M(t)|\mathcal{F}_t^{(Z)}) = \sum_{j=1}^6 \mathbb{P}(M_j(t)|\mathcal{F}_t^{(Z)})$.

We parameterize the macrotransition entry models as competing risks (Prentice, Kalbfleisch, Peterson Jr, Flournoy, Farewell & Breslow 1978): assuming player $\ell$ possesses the ball at time $t > 0$ during a possession, denote

$$ \lambda_j^\ell(t) = \lim_{\epsilon \to 0} \frac{\mathbb{P}(M_j(t)|\mathcal{F}_t^{(Z)})}{\epsilon} \eqno(7) $$

as the hazard for macrotransition $j$ at time $t$. We assume these are log-linear,

$$ \log(\lambda_j^\ell(t)) = [\mathbf{W}_j^\ell(t)]' \boldsymbol{\beta}_j^\ell + \xi_j^\ell \left( \mathbf{z}^\ell(t) \right) + \left( \tilde{\xi}_j^\ell \left( \mathbf{z}_j(t) \right) \mathbf{1}[j \leq 4] \right), \eqno(8) $$

where $\mathbf{W}_j^\ell(t)$ is a $p_j \times 1$ vector of time-varying covariates, $\boldsymbol{\beta}_j^\ell$ a $p_j \times 1$ vector of coefficients, $\mathbf{z}^\ell(t)$ is the ballcarrier's 2D location on the court (denote the court space $\mathbb{S}$) at time $t$, and $\xi_j^\ell : \mathbb{S} \to \mathbb{R}$ is a mapping of the player's court location to an additive effect on the log-hazard, providing spatial variation. The last term in (8) only appears for pass events ($j \leq 4$) to incorporate the location of the receiving player for the corresponding pass: $\mathbf{z}_j(t)$ (which slightly abuses notation) provides his location on the court at time $t$, and $\tilde{\xi}_j^\ell$, analogously to $\xi_j^\ell$, maps this location to an additive effect on the log-hazard. The four different passing options are identified by the (basketball) position of the potential pass recipient: point guard (PG), shooting guard (SG), small forward (SF), power forward (PF), and center (C).

The macrotransition model (7)–(8) represents the ballcarrier's decision-making process as an interpretable function of the unique basketball predicaments he faces. For example, in considering the hazard of a shot attempt, the time-varying covariates ($\mathbf{W}_j^\ell(t)$) we use are the distance between the ballcarrier and his nearest defender (transformed as $\log(1 + d)$ to moderate the influence of extremely large or small observed distances), an indicator for whether the ballcarrier has dribbled since gaining possession, and a constant representing a baseline shooting rate (this is not time-varying)<sup>4</sup>. The spatial effects $\xi_j^\ell$ reveal locations where player $\ell$ is more/less likely to attempt a shot in a small time window, holding fixed the time-varying covariates $\mathbf{W}_j^\ell(t)$. Such spatial effects (illustrated in Figure 5) are well-known to be nonlinear in distance from the basket and asymmetric about the angle to the basket (Miller, Bornn, Adams & Goldsberry 2013).

All model components—the time-varying covariates, their coefficients, and the spatial effects $\xi, \tilde{\xi}$ differ across macrotransition types $j$ for the same ballcarrier $\ell$, as well as across all $L = 461$ ballcarriers in the league during the 2013-14 season. This reflects the fact that players' decision-making tendencies and skills are unique; a player such as Dwight Howard will very rarely attempt a three point shot even if he is completely undefended, while someone like Stephen Curry will attempt a three point shot even when closely defended.

<sup>4</sup>Full details on all covariates used for all macrotransition types are included in Appendix A.1

Plots of estimated spatial effects $\xi$ for LeBron James as the ballcarrier, showing pass to PG, pass to SG, pass to PF, pass to C, shot-taking, and turnover.

Figure 5: Plots of estimated spatial effects $\xi$ for LeBron James as the ball carrier. For instance, plot (c) reveals the largest effect on James’ shot-taking hazard occurs near the basket, with noticeable positive effects also around the three point line (particularly in the “corner 3” shot areas). Plot (a) shows that he is more likely (per unit time) to pass to the point guard when at the top of the arc—more so when the point guard is positioned in the post area.

## 3.3 Macrotransition Exit Model

Using the six macrotransition types introduced in the previous subsection, we can express the macrotransition exit model (M3) when player $\ell$ has possession as

$$
\begin{aligned}
\mathbb{P}(C_{\delta_t} | M(t), \mathcal{F}_t^{(Z)}) &= \sum_{j=1}^6 \mathbb{P}(C_{\delta_t} | M_j(t), \mathcal{F}_t^{(Z)}) \mathbb{P}(M_j(t) | M(t), \mathcal{F}_t^{(Z)}) \\
&= \sum_{j=1}^6 \mathbb{P}(C_{\delta_t} | M_j(t), \mathcal{F}_t^{(Z)}) \frac{\lambda_j^\ell(t)}{\sum_{k=1}^6 \lambda_k^\ell(t)},
\end{aligned}
\eqno(9)
$$

using the competing risks model for $M_j(t)$ given by (7)–(8). As terms $\lambda_j^\ell(t)$ are supplied by (8), we focus on the macrotransition exit model conditional on the macrotransition type, $\mathbb{P}(C_{\delta_t} | M_j(t), \mathcal{F}_t^{(Z)})$.

For each $j = 1, \dots, 4$, $M_j(t)$ represents a pass-type macrotransition, therefore $C_{\delta_t}$ is a possession state $c' \in \mathcal{C}_{\text{poss}}$ for the player corresponding to pass option $j$. Thus, a model for $\mathbb{P}(C_{\delta_t} | M_j(t), \mathcal{F}_t^{(Z)})$ requires us to predict the state $c' \in \mathcal{C}_{\text{poss}}$ the $j$th pass target will occupy upon receiving the ball. Our approach is to simply assume $c'$ is given by the pass target’s location at the time the pass begins. While this is naive and could be improved by further modeling, it is a reasonable approximation in practice, because with only seven court regions and two defensive spacings comprising $\mathcal{C}_{\text{poss}}$, the pass recipient’s position in this space is unlikely to change during the time the pass is traveling en route, $\delta_t - t$ (a noteable exception is the alley-oop pass, which leads the pass recipient from well outside the basket to a dunk or layup within the restricted area). Our approach thus collapses $\mathbb{P}(C_{\delta_t} | M_j(t), \mathcal{F}_t^{(Z)})$ to a single state in $\mathcal{C}_{\text{poss}}$, which corresponds to pass target $j$’s location at time $t$.

When $j = 5$, and a shot attempt occurs in $(t, t + \epsilon]$, $C_{\delta_t}$ is either a made/missed 2 point shot, or made/missed three point shot. For sufficiently small $\epsilon$, we observe at $Z_t$ whether the shot attempt in $(t, t + \epsilon]$ is a two- or three-point shot, therefore our task in providing

$\mathbb{P}(C_{\delta_t} | M_j(t), \mathcal{F}_t^{(Z)})$ is modeling the shot's probability of success. We provide a parametric shot probability model, which shares the same form as the macrotransition entry model (7)–(8), though we use a logit link function as we are modeling a probability instead of a hazard. Specifically, for player $\ell$ attempting a shot at time $t$, let $p^\ell(t)$ represent the probability of the shot attempt being successful (resulting in a basket). We assume

$$ \text{logit}(p^\ell(t)) = [\mathbf{W}_s^\ell(t)]' \boldsymbol{\beta}_s^\ell + \xi_s^\ell(\mathbf{z}^\ell(t)) \tag{10} $$

with components in (10) having the same interpretation as their $j$-indexed counterparts in the competing risks model (8); that is, $\mathbf{W}_s^\ell$ is a vector of time-varying covariates (we use distance to the nearest defender—transformed as $\log(1 + d)$—an indicator for whether the player has dribbled, and a constant to capture baseline shooting efficiency) with $\boldsymbol{\beta}_s^\ell$ a corresponding vector of coefficients, and $\xi_s^\ell$ a smooth spatial effect, as in (8).

Lastly, when $j = 6$ and $M_j(t)$ represents a turnover, $C_{\delta_t}$ is equal to the turnover state in $\mathcal{C}_{end}$ with probability 1.

Note that the macrotransition exit model is mostly trivial when no player has ball possession at time $t$, since this implies $C_t \in \mathcal{C}_{trans} \cup \mathcal{C}_{end}$ and $\tau_t = t$. If $C_t \in \mathcal{C}_{end}$, then the possession is over and $C_{\delta_t} = C_t$. Otherwise, if $C_t \in \mathcal{C}_{trans}$ represents a pass attempt or turnover in progress, the following state $C_{\delta_t}$ is deterministic given $C_t$ (recall that the pass recipient and his location are encoded in the definition of pass attempt states in $\mathcal{C}_{trans}$). When $C_t$ represents a shot attempt in progress, the macrotransition exit model reduces to the shot probability model (10). Finally, when $C_t$ is a rebound in progress, we ignore full-resolution information and simply use the Markov transition probabilities from $\mathbf{P}^5$.

## 3.4 Transition Probability Matrix for Coarsened Process

The last model necessary for calculating EPV is (M4), the transition probability matrix for the embedded Markov chain corresponding to the coarsened process $C^{(0)}, C^{(2)}, \dots, C^{(K)}$. This transition probability matrix is used to compute the term $\mathbb{E}[X | C_{\delta_t} = c]$ that appears in Theorem 2.1. Recall that we denote the transition probability matrix as $\mathbf{P}$, where $P_{qr} = \mathbb{P}(C^{(i+1)} = c_r | C^{(i)} = c_q)$ for any $c_q, c_r \in \mathcal{C}$.

Without any other probabilistic structure assumed for $C^{(i)}$ other than Markov, for all $q, r$, the maximum likelihood estimator of $P_{qr}$ is the observed transition frequency, $\hat{P}_{qr} = \frac{N_{qr}}{\sum_{r'} N_{qr'}}$, where $N_{qr}$ counts the number of transitions $c_q \to c_r$. Of course, this estimator has undesirable performance if the number of visits to any particular state $c_q$ is small (for instance, Dwight Howard closely defended in the corner 3 region), as the estimated transition probabilities from that state may be degenerate.

Under our multiresolution model for basketball possessions, however, expected transition counts between many coarsened states $C^{(i)}$ can be computed as summaries of our macro-transition models (M2)–(M3). To show this, for any arbitrary $t > 0$ let $M_j^r(t)$ be the event

$$ M_j^r(t) = \{ \mathbb{P}(M_j(t) \text{ and } C_{t+\epsilon} = c_r | \mathcal{F}_t^{(Z)}) > 0 \}. $$

Thus $M_j^r(t)$ occurs if it is possible for a macrotransition of type $j$ into state $c_r$ to occur in

<sup>5</sup>Our rebounding model could be improved by using full-resolution spatiotemporal information, as players' reactions to the missed shot event are informative of who obtains the rebound.

$(t, t + \epsilon]$. When applicable, we can use this to get the expected number of $c_q \rightarrow c_r$ transitions:

$$ \tilde{N}_{qr} = \epsilon \sum_{t:C_t=c_q} \lambda_j^\ell(t) \mathbf{1}[M_j^r(t)]. \tag{11} $$

When $c_q$ is a shot attempt state from $c_{q'} \in \mathcal{C}_{\text{poss}}$, (11) is adjusted using the shot probability model (10): $\tilde{N}_{qr} = \epsilon \sum_{t:C_t=c_{q'}} \lambda_j^\ell(t)p(t)\mathbf{1}[M_j^r(t)]$ when $c_r$ represents an eventual made shot and $\tilde{N}_{qr} = \epsilon \sum_{t:C_t=c_{q'}} \lambda_j^\ell(t)(1 - p(t))\mathbf{1}[M_j^r(t)]$ when $c_r$ represents an eventual miss.

By replacing raw counts with their expectations conditional on higher-resolution data, leveraging the hazards (11) provides a Rao-Blackwellized (unbiased, lower variance) alternative to counting observed transitions. Furthermore, due to the hierarchical parameterization of hazards $\lambda_j^\ell(t)$ (discussed in Section 4), information is shared across space and player combinations so that estimated hazards are well-behaved even in situations without significant observed data. Thus, when $c_q \rightarrow c_r$ represents a macrotransition, we use $\tilde{N}_{qr}$ in place of $N_{qr}$ when calculating $\hat{P}_{qr}$.

# 4. HIERARCHICAL MODELING AND INFERENCE

A critical aspect of the micro- and macrotransition models defined in the previous section is that they are parameterized to capture the variations between actions, players, and court space that play a central role in basketball strategy. This section outlines the procedure for reliably estimating this rich set of model parameters using likelihood-based methods.

Hierarchical models are essential for our problem because by implicitly averaging over all possible future possession paths, calculating EPV requires transition probabilities for situations for which there is no data. For instance, DeAndre Jordan did not attempt a three-point shot in the 2013-14 season, yet any EPV estimate for a possession with him on the court requires an estimate of his shooting ability from everywhere on the court, even though for some of these regions it is unlikely he would attempt a shot. Hierarchical models combine information both across space and across different players to estimate such probabilities.

## 4.1 Conditional Autoregressive Prior for Player-Specific Coefficients

Sharing information between players is critical for our estimation problem, but standard hierarchical models encode an assumption of exchangeability between units that is too strong for NBA players, even between those who are classified by the league as playing the same position. For instance, LeBron James is listed at the same position (small forward) as Steve Novak, despite the fact that James is one of the NBA's most prolific short-range scorers whereas Novak has not scored a layup since 2012. To model between-player variation more realistically, our hierarchical model shares information across players based on a localized notion of player similarity that we represent as an $L \times L$ binary adjacency matrix **H**: $H_{\ell k} = 1$ if players $\ell$ and $k$ are similar to each other and $H_{\ell k} = 0$ otherwise. We determine similarity in a pre-processing step that compares the spatial distribution of where players spend time on the offensive half-court; see Appendix A.2 for exact details on specifying **H**.

Now let $\beta_{ji}^\ell$ be the $i$th component of $\boldsymbol{\beta}_j^\ell$, the vector of coefficients for the time-referenced covariates for player $\ell$'s hazard $j$ (8). Also let $\boldsymbol{\beta}_{ji}$ be the vector representing this component across all $L = 461$ players, $(\beta_{ji}^1, \beta_{ji}^2, \dots, \beta_{ji}^L)'$. We assume independent conditional autoregressive

(CAR) priors (Besag 1974) for $\beta_{ji}$:

$$
\begin{aligned}
\beta_{ji}^\ell | \beta_{ji}^{(-\ell)}, \tau_{\beta_{ji}}^2 &\sim \mathcal{N} \left( \frac{1}{n_\ell} \sum_{k: H_{\ell k}=1} \beta_{ji}^k, \frac{\tau_{\beta_{ji}}^2}{n_\ell} \right) \\
\tau_{\beta_{ji}}^2 &\sim \text{InvGam}(1, 1)
\end{aligned}
\eqno(12)
$$

where $n_\ell = \sum_k H_{\ell k}$. Similarly, let $\boldsymbol{\beta}_{si} = (\beta_{si}^1 \ \beta_{si}^2 \ \dots \ \beta_{si}^L)$ be the vector of the $i$th component of the shot probability model (10) across players $1, \dots, L$. We assume the same CAR prior (12) independently for each component $i$. While the inverse gamma prior for $\tau_*^2$ terms seems very informative, we want to avoid very large or small values of $\tau_*^2$, corresponding to 0 or full shrinkage (respectively), which we know are inappropriate for our model. Predictive performance for the 0 shrinkage model ($\tau_*^2$ very large) is shown in Table 1, whereas the full shrinkage model ($\tau_*^2 = 0$) doesn't allow parameters to differ by player identity, which precludes many of the inferences EPV was designed for.

## 4.2 Spatial Effects $\xi$

Player-tracking data is a breakthrough because it allows us to model the fundamental spatial component of basketball. In our models, we incorporate the properties of court space in spatial effects $\xi_j^\ell, \tilde{\xi}_j^\ell, \xi_s^\ell$, which are unknown real-valued functions on $\mathbb{S}$, and therefore infinite dimensional. We represent such spatial effects using Gaussian processes (see Rasmussen (2006) for an overview of modeling aspects of Gaussian processes). Gaussian processes are usually specified by a mean function and covariance function; this approach is computationally intractable for large data sets, as the computation cost of inference and interpolating the surface at unobserved locations is $\mathcal{O}(n^3)$, where $n$ is the number of different points at which $\xi_j^\ell$ is observed (for many spatial effects $\xi_j^\ell$, the corresponding $n$ would be in the hundreds of thousands). We instead provide $\xi$ with a low-dimensional representation using functional bases (Higdon 2002; Quiñonero-Candela & Rasmussen 2005), which offers three important advantages. First, this representation is more computationally efficient for large data sets such as ours. Second, functional bases allow for a non-stationary covariance structure that reflects unique spatial dependence patterns on the basketball court. Finally, the finite basis representation allows us to apply the same between-player CAR prior to estimate each player's spatial effects.

Our functional basis representation of a Gaussian process $\xi_j^\ell$ relies on $d$ deterministic basis functions $\phi_{j1}, \dots, \phi_{jd} : \mathbb{S} \to \mathbb{R}$ such that for any $\mathbf{z} \in \mathbb{S}$,

$$
\xi_j^\ell(\mathbf{z}) = \sum_{i=1}^d w_{ji}^\ell \phi_{ji}(\mathbf{z}), \eqno(13)
$$

where $\mathbf{w}_j^\ell = (w_{j1}^\ell \ \dots \ w_{jd}^\ell)'$ is a random vector of loadings, $\mathbf{w}_j^\ell \sim \mathcal{N}(\boldsymbol{\omega}_j^\ell, \boldsymbol{\Sigma}_j^\ell)$. Letting $\Phi_j(\mathbf{z}) = (\phi_{j1}(\mathbf{z}) \ \dots \ \phi_{jd}(\mathbf{z}))'$, we can see that $\xi_j^\ell$ given by (13) is a Gaussian process with mean function $\Phi_j(\mathbf{z})' \boldsymbol{\omega}_j^\ell$ and covariance function $\text{Cov}[\xi_j^\ell(\mathbf{z}_1), \xi_j^\ell(\mathbf{z}_2)] = \Phi_j(\mathbf{z}_1)' \boldsymbol{\Sigma}_j^\ell \Phi_j(\mathbf{z}_2)$. However, since bases $\phi_{ji}$ are deterministic, each $\xi_j^\ell$ is represented as a $d$-dimensional parameter. Note that we also use (13) for pass receiver spatial effects and the spatial effect term in the shot probability model, $\tilde{\xi}_j^\ell$ and $\xi_s^\ell$, respectively. For these terms we have associated bases $\tilde{\phi}_{ji}, \phi_{si}$ and weights, $\tilde{w}_{ji}^\ell, w_{si}^\ell$. As our notation indicates, bases functions $\Phi_j(\mathbf{z})$ differ for each

Heatmaps showing functional bases for shot-taking macro-transitions on a basketball court.

*Figure 6: The functional bases $\phi_{ji}$ for $i = 1, \dots, 10$ and $j$ corresponding to the shot-taking macro-transition, $j = 5$. There is no statistical interpretation of the ordering of the bases; we have displayed them in rough order of the shot types represented, from close-range to long-range.*

macrotransition type but are constant across players; whereas weight vectors $\mathbf{w}_j^\ell$ vary across both macrotransition types and players.

Using $d = 10$, we determine the functional bases in a pre-processing step, discussed in Appendix A.3. These basis functions are interpretable as patterns/motifs that constitute players’ decision-making tendencies as a function of space; please see Figure 6 for an example, or Miller et al. (2013) for related work in a basketball context. Furthermore, we use a CAR model (12) to supply the prior mean and covariance matrix $(\boldsymbol{\omega}_j^\ell, \boldsymbol{\Sigma}_j^\ell)$ for the weights:

$$
\begin{aligned}
\mathbf{w}_j^\ell | \mathbf{w}_j^{-(\ell)}, \tau_{\mathbf{w}_j}^2 &\sim \mathcal{N} \left( \frac{1}{n_\ell} \sum_{k: H_{\ell k}=1} \mathbf{w}_j^k, \frac{\tau_{\mathbf{w}_j}^2}{n_\ell} \mathbf{I}_d \right) \\
\tau_{\mathbf{w}_j}^2 &\sim \text{InvGam}(1, 1).
\end{aligned}
\eqno(14)
$$

As with (12), we also use (14) for terms $\tilde{\mathbf{w}}_j$ and $\mathbf{w}_s$. Combining the functional basis representation (13) with the between-player CAR prior (14) for the weights, we get a prior representation for spatial effects $\xi_j^\ell, \tilde{\xi}_j^\ell, \tilde{\xi}^\ell$ that is low-dimensional and shares information both across space and between different players.

## 4.3 Parameter Estimation

As discussed in Section 3, calculating EPV requires the parameters that define the multiresolution transition models (M1)–(M4)—specifically, the hazards $\lambda_j^\ell$, shot probabilities $p^\ell$, and all parameters of the microtransition model (M1). We estimate these parameters in a Bayesian fashion, combining the likelihood of the observed optical tracking data with the prior structure discussed earlier in this section. Using our multiresolution models, we can

write the likelihood for the full optical tracking data, indexed arbitrarily by $t$:

$$
\begin{aligned}
\prod_{t} \mathbb{P}(Z_{t+\epsilon} | \mathcal{F}_{t}^{(Z)}) = & \overbrace{\left( \prod_{t} \mathbb{P}(Z_{t+\epsilon} | M(t)^{c}, \mathcal{F}_{t}^{(Z)})^{\mathbf{1}[M(t)^{c}]} \right)}^{L_{\text{mic}}} \overbrace{\left( \prod_{t} \prod_{j=1}^{6} \mathbb{P}(Z_{t+\epsilon} | M_{j}(t), C_{\delta_{t}}, \mathcal{F}_{t}^{(Z)})^{\mathbf{1}[M_{j}(t)]} \right)}^{L_{\text{rem}}} \\
& \times \underbrace{\left( \prod_{t} \mathbb{P}(M(t)^{c} | \mathcal{F}_{t}^{(Z)})^{\mathbf{1}[M(t)^{c}]} \prod_{j=1}^{6} \mathbb{P}(M_{j}(t) | \mathcal{F}_{t}^{(Z)})^{\mathbf{1}[M_{j}(t)]} \right)}_{L_{\text{entry}}} \underbrace{\left( \prod_{t} \prod_{j=1}^{6} \mathbb{P}(C_{\delta_{t}} | M_{j}(t), \mathcal{F}_{t}^{(Z)})^{\mathbf{1}[M_{j}(t)]} \right)}_{L_{\text{exit}}},
\end{aligned}
\eqno (15)
$$

The factorization used in (15) highlights data features that inform different parameter groups: $L_{\text{mic}}$ is the likelihood term corresponding to the microtransition model (M1), $L_{\text{entry}}$ the macrotransition entry model (M2), and $L_{\text{exit}}$ the macrotransition exit model (M3). The remaining term $L_{\text{rem}}$ is left unspecified, and ignored during inference. Thus, $L_{\text{mic}}$, $L_{\text{entry}}$, and $L_{\text{exit}}$ can be thought of as partial likelihoods (Cox 1975*b*), which under mild conditions leads to consistent and asymptotically well-behaved estimators (Wong 1986). When parameters in these partial likelihood terms are given prior distributions, as is the case for those comprising the hazards in the macrotransition entry model, as well as those in the shot probability model, the resulting inference is partially Bayesian (Cox 1975*a*).

The microtransition partial likelihood term $L_{\text{mic}}$ factors by player:

$$
L_{\text{mic}} \propto \prod_{t} \prod_{\ell=1}^{L} \mathbb{P}(\mathbf{z}_{\ell}(t + \epsilon) | M(t)^{c}, \mathcal{F}_{t}^{(Z)})^{\mathbf{1}[M(t)^{c} \text{ and } \ell \text{ on court at time } t]}. \eqno (16)
$$

Depending on whether or not player $\ell$ is on offense (handling the ball or not) or defense, $\mathbb{P}(\mathbf{z}_{\ell}(t+\epsilon) | M(t)^{c}, \mathcal{F}_{t}^{(Z)})$ is supplied by the offensive (5) or defensive (6) microtransition models. Parameters for these models (5)–(6) are estimated using R-INLA, where spatial acceleration effects $\mu_{x}^{\ell}, \mu_{y}^{\ell}$ are represented using a Gaussian Markov random field approximation to a Gaussian process with Matérn covariance (Lindgren, Rue & Lindström 2011). Appendix A.3 provides the details on this approximation. We do perform any hierarchical modeling for the parameters of the microtransition model—because this model only describes movement (not decision-making), the data for every player is informative enough to provide precise inference. Thus, microtransition models are fit in parallel using each player's data separately; this requires $L = 461$ processors, each taking at most 18 hours at 2.50Ghz clock speed, using 32GB of RAM.

For the macrotransition entry term, we can write

$$
L_{\text{entry}} \propto \prod_{l=1}^{L} \prod_{j=1}^{6} L_{\text{entry}_{j}}^{\ell}(\lambda_{j}^{\ell}(\cdot)), \eqno (17)
$$

recognizing that (for small $\epsilon$),

$$L_{\text{entry}_j}^\ell(\lambda_j^\ell(\cdot)) \propto \left( \prod_{\substack{t \colon M_j(t) \\ t \in \mathcal{T}^\ell}} \lambda_j^\ell(t) \right) \exp \left( -\sum_{t \in \mathcal{T}^\ell} \lambda_j^\ell(t) \right)$$
$$\text{where} \quad \log(\lambda_j^\ell(t)) = [\mathbf{W}_j^\ell(t)]' \boldsymbol{\beta}_j^\ell + \boldsymbol{\phi}_j(\mathbf{z}_\ell(t))' \mathbf{w}_j^\ell + \left( \tilde{\boldsymbol{\phi}}_j(\mathbf{z}_j(t))' \tilde{\mathbf{w}}_j^\ell \mathbf{1}[j \leq 4] \right) \quad (18)$$

and $\mathcal{T}^\ell$ is the set of time $t$ for which player $\ell$ possesses the ball. Expression (18) is the likelihood for a Poisson regression; combined with prior distributions (12)–(14), inference for $\boldsymbol{\beta}_j^\ell, \mathbf{w}_j^\ell, \tilde{\mathbf{w}}_j^\ell$ is thus given by a hierarchical Poisson regression. However, the size of our data makes implementing such a regression model computationally difficult as the design matrix would have 30.4 million rows and a minimum of $L(p_j + d) \geq 5993$ columns, depending on macrotransition type. We perform this regression through the use of integrated nested Laplace approximations (INLA) (Rue, Martino & Chopin 2009). Each macrotransition type can be fit separately, and requires approximately 24 hours using a single 2.50GHz processor with 120GB of RAM.

Recalling Section 3.3, the macrotransition exit model (M3) is deterministic for all macrotransitions except shot attempts ($j = 5$). Thus, $L_{\text{exit}}$ only provides information on the parameters of our shot probability model (10). Analogous to the Poisson model in (18), $L_{\text{exit}}$ is the likelihood of a logistic regression, which factors by player. We also use INLA to fit this hierarchical logistic regression model, though fewer computational resources are required as this likelihood only depends on time points where a shot is attempted, which is a much smaller subset of our data.

# 5. RESULTS

After obtaining parameter estimates for the multiresolution transition models, we can calculate EPV using Theorem 2.1 and plot $\nu_t$ throughout the course of any possession in our data. We view such (estimated) EPV curves as the main contribution of our work, and their behavior and potential inferential value has been introduced in Section 1. We illustrate this value by revisiting the possession highlighted in Figure 1 through the lens of EPV. Analysts may also find meaningful aggregations of EPV curves that summarize players' behavior over a possession, game, or season in terms of EPV. We offer two such aggregations in this section.

## 5.1 Predictive Performance of EPV

Before analyzing EPV estimates, it is essential to check that such estimates are properly calibrated (Gneiting, Balabdaoui & Raftery 2007) and accurate enough to be useful to basketball analysts. Our paper introduces EPV, and as such there are no existing results to benchmark the predictive performance of our estimates. We can, however, compare the proposed implementation for estimating EPV with simpler models, based on lower resolution information, to verify whether our multiresolution model captures meaningful features of our data. Assessing the predictive performance of an EPV estimator is difficult because the estimand is a curve whose length varies by possession. Moreover, we never observe any portion of this curve; we only know its endpoint. Therefore, rather than comparing estimated EPV curves between our method and alternative methods, we compare estimated transition prob-

abilities. For any EPV estimator method that is stochastically consistent, if the predicted transitions are properly calibrated, then the derived EPV estimates should be as well.

For the inference procedure in Section 4, we use only 90% of our data set for parameter inference, with the remaining 10% used to evaluate the out-of-sample performance of our model. We also evaluated out-of-sample performance of simpler macrotransition entry/exit models, which use varying amounts of information from the data. Table 1 provides the out-of-sample log-likelihood for the macrotransition models applied to the 10% of the data not used in model fitting for various simplified models. In particular, we start with the simple model employing constant hazards for each player/event type, then successively add situational covariates, spatial information, then full hierarchical priors. Without any shrinkage, our full model performs in some cases worse than a model with no spatial effects included, but with shrinkage, it consistently performs the best (highest log-likelihood) of the configurations compared. This behavior justifies the prior structure introduced in Section 4.

<table>
  <thead>
    <tr>
        <th colspan="5">Model Terms</th>
    </tr>
    <tr>
        <th>Macro. type</th>
        <th>Player</th>
        <th>Covariates</th>
        <th>Covariates + Spatial</th>
        <th>Full</th>
    </tr>
  </thead>
  <tbody>
    <tr>
        <td>Pass1</td>
        <td>-29.4</td>
        <td>-27.7</td>
        <td>-27.2</td>
        <td>-26.4</td>
    </tr>
    <tr>
        <td>Pass2</td>
        <td>-24.5</td>
        <td>-23.7</td>
        <td>-23.2</td>
        <td>-22.2</td>
    </tr>
    <tr>
        <td>Pass3</td>
        <td>-26.3</td>
        <td>-25.2</td>
        <td>-25.3</td>
        <td>-23.9</td>
    </tr>
    <tr>
        <td>Pass4</td>
        <td>-20.4</td>
        <td>-20.4</td>
        <td>-24.5</td>
        <td>-18.9</td>
    </tr>
    <tr>
        <td>Shot Attempt</td>
        <td>-48.9</td>
        <td>-46.4</td>
        <td>-40.9</td>
        <td>-40.7</td>
    </tr>
    <tr>
        <td>Made Basket</td>
        <td>-6.6</td>
        <td>-6.6</td>
        <td>-5.6</td>
        <td>-5.2</td>
    </tr>
    <tr>
        <td>Turnover</td>
        <td>-9.3</td>
        <td>-9.1</td>
        <td>-9.0</td>
        <td>-8.4</td>
    </tr>
  </tbody>
</table>

Table 1: *Out of sample log-likelihood (in thousands) for macrotransition entry/exit models under various model specifications. "Player" assumes constant hazards for each player/event type combination. "Covariates" augments this model with situational covariates, $\mathbf{W}_j^\ell(t)$ as given in (8). "Covariates + Spatial" adds a spatial effect, yielding (8) in its entirety. Lastly, "Full" implements this model with the full hierarchical model discussed in Section 4.*

## 5.2 Possession Inference from Multiresolution Transitions

Understanding the calculation of EPV in terms of multiresolution transitions is a valuable exercise for a basketball analyst, as these model components reveal precisely how the EPV estimate derives from the spatiotemporal circumstances of the time point considered. Figure 7 diagrams four moments during our example possession (introduced originally in Figures 1 and 2) in terms of multiresolution transition probabilities. These diagrams illustrate Theorem 2.1 by showing EPV as a weighted average of the value of the next macrotransition. Potential ball movements representing macrotransitions are shown as arrows, with their respective values and probabilities graphically illustrated by color and line thickness (this information is also annotated explicitly). Microtransition distributions are also shown, indicating distributions of players' movement over the next two seconds. Note that the possession diagrammed here was omitted from the data used for parameter estimation.

Analyzing Figure 7, we see that our model estimates largely agree with basketball intuition. For example, players are quite likely to take a shot when they are near to and/or moving towards the basket, as shown in panels A and D. Additionally, because LeBron James

**LEGEND**

**MOVEMENT**
$\leftarrow$ Macro $\quad \dots$ History $\quad \blacktriangleright$ Micro

**MACROTRANSITION VALUE (V)**
<span style="color:red">▬</span> High $\quad$ <span style="color:grey">▬</span> Medium $\quad$ <span style="color:blue">▬</span> Low

**MACROTRANSITION PROBABILITY (P)**
▬ High $\quad$ ▬ Medium $\quad$ ▬ Low

<table>
  <thead>
    <tr>
        <th>OFFENSE</th>
        <th>DEFENSE</th>
    </tr>
  </thead>
  <tbody>
    <tr>
        <td>❶ Norris Cole</td>
        <td>Deron Williams ❶</td>
    </tr>
    <tr>
        <td>❷ Ray Allen</td>
        <td>Jason Terry ❷</td>
    </tr>
    <tr>
        <td>❸ Rashard Lewis</td>
        <td>Joe Johnson ❸</td>
    </tr>
    <tr>
        <td>❹ LeBron James</td>
        <td>Andray Blatche ❹</td>
    </tr>
    <tr>
        <td>❺ Chris Bosh</td>
        <td>Brook Lopez ❺</td>
    </tr>
  </tbody>
</table>

Visualization of EPV and player transitions

*Figure 7: Detailed diagram of EPV as a function of multiresolution transition probabilities for four time points (labeled A, B, C, D) of the possession featured in Figures 1–2. Two seconds of microtransitions are shaded (with forecasted positions for short time horizons darker) while macrotransitions are represented by arrows, using color and line thickness to encode the value (V) and probability (P) of such macrotransitions. The value and probability of the “other” category represents the case that no macrotransition occurs during the next two seconds.*

is a better shooter than Norris Cole, the value of his shot attempt is higher, even though in the snapshot in panel D he is much farther from the basket than Cole is in panel A. While the value of the shot attempt averages over future microtransitions, which may move the player closer to the basket, when macrotransition hazards are high this average is dominated by microtransitions on very short time scales.

We also see Ray Allen, in the right corner 3, as consistently one of the most valuable pass options during this possession, particularly when he is being less closely defended as in panels A and D. In these panels, though, we never see an estimated probability of him receiving a pass above 0.05, most likely because he is being fairly closely defended for someone so far from the ball, and because there are always closer passing options for the ballcarrier. Similarly, while Chris Bosh does not move much during this possession, he is most valuable

20

as a passing option in panel C where he is closest to the basket and without any defenders in his lane. From this, we see that the estimated probabilities and values of the macrotransitions highlighted in Figure 7 match well with basketball intuition.

The analysis presented here could be repeated on any of hundreds of thousands of possessions available in a season of optical tracking data. EPV plots as in Figure 2 and diagrams as in Figure 7 provide powerful insight as to how players’ movements and decisions contribute value to their team’s offense. With this insight, coaches and analysts can formulate strategies and offensive schemes that make optimal use of their players’ ability—or, defensive strategies that best suppress the motifs and situations that generate value for the opposing offense.

## 5.3 EPV-Added

Aggregations of EPV estimates across possessions can yield useful summaries for player evaluation. For example, *EPV-Added* (EPVA) quantifies a player’s overall offensive value through his movements and decisions while handling the ball, relative to the estimated value contributed by a league-average player receiving ball possession in the same situations. The notion of *relative* value is important because the martingale structure of EPV ($\nu_t$) prevents any meaningful aggregation of EPV across a specific player’s possessions. $\mathbb{E}[\nu_t - \nu_{t+\epsilon}] = 0$ for all $t$, meaning that *on average* EPV does not change during any specific player’s ball handling. Thus, while we see the EPV skyrocket after LeBron James receives the ball and eventually attack the basket in Figure 2, the definition of EPV prevents such increases being observed on average.

If player $\ell$ has possession of the ball starting at time $t_s$ and ending at $t_e$, the quantity $\nu_{t_e} - \nu_{t_s}^{r(\ell)}$ estimates the value contributed player by $\ell$ relative to the hypothetical league-average player during his ball possession (represented by $\nu_{t_s}^{r(\ell)}$). We calculate EPVA for player $\ell$ ($\text{EPVA}(\ell)$) by summing such differences over all a player’s touches (and dividing by the number of games played by player $\ell$ to provide standardization):

$$ \text{EPVA}(\ell) = \frac{1}{\# \text{ games for } \ell} \sum_{\{t_s, t_e\} \in \mathcal{T}^\ell} \nu_{t_e} - \nu_{t_s}^{r(\ell)} \text{ (19)} $$

where $\mathcal{T}^\ell$ contains all intervals of form $[t_s, t_e]$ that span player $\ell$’s ball possession. Specific details on calculating $\nu_t^{r(\ell)}$ are included in Appendix A.4.

Averaging over games implicitly rewards players who have high usage, even if their value added per touch might be low. Often, one-dimensional offensive players accrue the most EPVA per touch since they only handle the ball when they are uniquely suited to scoring; for instance, some centers (such as the Clippers’ DeAndre Jordan) only receive the ball right next to the basket, where their height offers a considerable advantage for scoring over other players in the league. Thus, averaging by game—not touch—balances players’ efficiency per touch with their usage and importance in the offense. Depending on the context of the analysis, EPVA can also be adjusted to account for team pace (by normalizing by 100 possession) or individual usage (by normalizing by player-touches).

Table 2 provides a list of the top and bottom 10 ranked players by EPVA using our 2013-14 data. Generally, players with high EPVA effectively adapt their decision-making process to the spatiotemporal circumstances they inherit when gaining possession. They receive the ball in situations that are uniquely suited to their abilities, so that on average the rest of the league is less successful in these circumstances. Players with lower EPVA are not necessarily

“bad” players in any conventional sense; their actions simply tend to lead to fewer points than other players given the same options. Of course, EPVA provides a limited view of a player’s overall contributions since it does not quantify players’ actions on defense, or other ways that a player may impact EPV while not possessing the ball (though EPVA could be extended to include these aspects).

<table>
  <thead>
    <tr>
        <th>Rank</th>
        <th>Player</th>
        <th>EPVA</th>
    </tr>
  </thead>
  <tbody>
    <tr>
        <td>1</td>
        <td>Kevin Durant</td>
        <td>3.26</td>
    </tr>
    <tr>
        <td>2</td>
        <td>LeBron James</td>
        <td>2.96</td>
    </tr>
    <tr>
        <td>3</td>
        <td>Jose Calderon</td>
        <td>2.79</td>
    </tr>
    <tr>
        <td>4</td>
        <td>Dirk Nowitzki</td>
        <td>2.69</td>
    </tr>
    <tr>
        <td>5</td>
        <td>Stephen Curry</td>
        <td>2.50</td>
    </tr>
    <tr>
        <td>6</td>
        <td>Kyle Korver</td>
        <td>2.01</td>
    </tr>
    <tr>
        <td>7</td>
        <td>Serge Ibaka</td>
        <td>1.70</td>
    </tr>
    <tr>
        <td>8</td>
        <td>Channing Frye</td>
        <td>1.65</td>
    </tr>
    <tr>
        <td>9</td>
        <td>Al Horford</td>
        <td>1.55</td>
    </tr>
    <tr>
        <td>10</td>
        <td>Goran Dragic</td>
        <td>1.54</td>
    </tr>
  </tbody>
</table>
<table>
  <thead>
    <tr>
        <th>Rank</th>
        <th>Player</th>
        <th>EPVA</th>
    </tr>
  </thead>
  <tbody>
    <tr>
        <td>277</td>
        <td>Zaza Pachulia</td>
        <td>-1.55</td>
    </tr>
    <tr>
        <td>278</td>
        <td>DeMarcus Cousins</td>
        <td>-1.59</td>
    </tr>
    <tr>
        <td>279</td>
        <td>Gordon Hayward</td>
        <td>-1.61</td>
    </tr>
    <tr>
        <td>280</td>
        <td>Jimmy Butler</td>
        <td>-1.61</td>
    </tr>
    <tr>
        <td>281</td>
        <td>Rodney Stuckey</td>
        <td>-1.63</td>
    </tr>
    <tr>
        <td>282</td>
        <td>Ersan Ilyasova</td>
        <td>-1.89</td>
    </tr>
    <tr>
        <td>283</td>
        <td>DeMar DeRozan</td>
        <td>-2.03</td>
    </tr>
    <tr>
        <td>284</td>
        <td>Rajon Rondo</td>
        <td>-2.27</td>
    </tr>
    <tr>
        <td>285</td>
        <td>Ricky Rubio</td>
        <td>-2.36</td>
    </tr>
    <tr>
        <td>286</td>
        <td>Rudy Gay</td>
        <td>-2.59</td>
    </tr>
  </tbody>
</table>

Table 2: Top/bottom 10 players by EPVA per game in 2013-14, minimum 500 touches in season.

As such, we stress the idea that EPVA is not a best/worst players in the NBA ranking. Analysts should also be aware that the league-average player being used as a baseline is completely hypothetical, and we heavily extrapolate our model output by considering value calculations assuming this nonexistant player possessing the ball in all the situations encountered by an actual NBA player. The extent to which such an extrapolation is valid is a judgment a basketball expert can make. Alternatively, one can consider EPV-added over *specific* players (assuming player $\ell_2$ receives the ball in the same situations as player $\ell_1$), using the same framework developed for EPVA. Such a quantity may actually be more useful, particularly if the players being compared play similar roles on their teams and face similar situations and the degree of extrapolation is minimized.

## 5.4 Shot Satisfaction

Aggregations of the individual components of our multiresolution transition models can also provide useful insights. For example, another player metric we consider is called *shot satisfaction*. For each shot attempt a player takes, we wonder how satisfied the player is with his decision to shoot—what was the expected point value of his most reasonable passing option at the time of the shot? If for a particular player, the EPV measured at his shot attempts is higher than the EPV conditioned on his possible passes at the same time points, then by shooting the player is usually making the best decision for his team. On the other hand, players with pass options at least as valuable as shots should regret their shot attempts (we term “satisfaction” as the opposite of regret) as passes in these situations have higher expected value.

Specifically, we calculate

$$ \text{SATIS}(\ell) = \frac{1}{|T_{\text{shot}}^{\ell}|} \sum_{t \in T_{\text{shot}}^{\ell}} \nu_t - \mathbb{E} \left[ X \mid \bigcup_{j=1}^{4} M_j(t), \mathcal{F}_t^{(Z)} \right] \tag{20} $$

<table>
  <thead>
    <tr>
        <th>Rank</th>
        <th>Player</th>
        <th>Shot Satis.</th>
    </tr>
  </thead>
  <tbody>
    <tr>
        <td>1</td>
        <td>Mason Plumlee</td>
        <td>0.35</td>
    </tr>
    <tr>
        <td>2</td>
        <td>Pablo Prigioni</td>
        <td>0.31</td>
    </tr>
    <tr>
        <td>3</td>
        <td>Mike Miller</td>
        <td>0.27</td>
    </tr>
    <tr>
        <td>4</td>
        <td>Andre Drummond</td>
        <td>0.26</td>
    </tr>
    <tr>
        <td>5</td>
        <td>Brandan Wright</td>
        <td>0.24</td>
    </tr>
    <tr>
        <td>6</td>
        <td>DeAndre Jordan</td>
        <td>0.24</td>
    </tr>
    <tr>
        <td>7</td>
        <td>Kyle Korver</td>
        <td>0.24</td>
    </tr>
    <tr>
        <td>8</td>
        <td>Jose Calderon</td>
        <td>0.22</td>
    </tr>
    <tr>
        <td>9</td>
        <td>Jodie Meeks</td>
        <td>0.22</td>
    </tr>
    <tr>
        <td>10</td>
        <td>Anthony Tolliver</td>
        <td>0.22</td>
    </tr>
  </tbody>
</table>
<table>
  <thead>
    <tr>
        <th>Rank</th>
        <th>Player</th>
        <th>Shot Satis.</th>
    </tr>
  </thead>
  <tbody>
    <tr>
        <td>277</td>
        <td>Garrett Temple</td>
        <td>-0.02</td>
    </tr>
    <tr>
        <td>278</td>
        <td>Kevin Garnett</td>
        <td>-0.02</td>
    </tr>
    <tr>
        <td>279</td>
        <td>Shane Larkin</td>
        <td>-0.02</td>
    </tr>
    <tr>
        <td>280</td>
        <td>Tayshaun Prince</td>
        <td>-0.03</td>
    </tr>
    <tr>
        <td>281</td>
        <td>Dennis Schroder</td>
        <td>-0.04</td>
    </tr>
    <tr>
        <td>282</td>
        <td>LaMarcus Aldridge</td>
        <td>-0.04</td>
    </tr>
    <tr>
        <td>283</td>
        <td>Ricky Rubio</td>
        <td>-0.04</td>
    </tr>
    <tr>
        <td>284</td>
        <td>Roy Hibbert</td>
        <td>-0.05</td>
    </tr>
    <tr>
        <td>285</td>
        <td>Will Bynum</td>
        <td>-0.05</td>
    </tr>
    <tr>
        <td>286</td>
        <td>Darrell Arthur</td>
        <td>-0.05</td>
    </tr>
  </tbody>
</table>

Table 3: Top/bottom 10 players by shot satisfaction in 2013-14, minimum 500 touches in season.

where $\mathcal{T}_{\text{shot}}^{\ell}$ indexes times a shot attempt occurs, $\{t : M_5(t)\}$, for player $\ell$. Recalling that macrotransitions $j = 1, \dots, 4$ correspond to pass events (and $j = 5$ a shot attempt), $\bigcup_{j=1}^4 M_j(t)$ is equivalent to a pass happening in $(t, t + \epsilon]$. Unlike EPVA, shot satisfaction $\text{SATIS}(\ell)$ is expressed as an average per shot (not per game), which favors player such as three point specialists, who often take fewer shots than their teammates, but do so in situations where their shot attempt is extremely valuable. Table 3 provides the top/bottom 10 players in shot satisfaction for our 2013-14 data. While players who mainly attempt three-pointers (e.g. Miller, Korver) and/or shots near the basket (e.g. Plumlee, Jordan) have the most shot satisfaction, players who primarily take mid-range or long-range two-pointers (e.g. Aldridge, Garnett) or poor shooters (e.g. Rubio, Prince) have the least. However, because shot satisfaction numbers are mostly positive league-wide, players still shoot relatively efficiently—almost every player generally helps his team by shooting rather than passing in the same situations, though some players do so more than others.

We stress that the two derived metrics given in this paper, EPVA and shot satisfaction, are simply examples of the kinds of analyses enabled by EPV. Convential metrics currently used in basketball analysis do measure shot selection and efficiency, as well as passing rates and assists, yet EPVA and shot satisfaction are novel in analyzing these events in their spatiotemporal contexts.

# 6. DISCUSSION

This paper introduces a new quantity, EPV, which represents a paradigm shift in the possibilities for statistical inferences about basketball. Using high resolution, optical tracking data, EPV reveals the value in many of the schemes and motifs that characterize basketball offenses but are omitted in the box score. For instance, as diagrammed in Figures 2 and 7, we see that EPV may rise as a player attacks the basket (more so for a strong scorer like LeBron James than for a bench player like Norris Cole), passes to a well-positioned teammate, or gains separation from the defense. Aside from simply tracking changes in EPV, analysts can understand why EPV changes by expressing its value as a weighted average of transition values (as done in Figure 7). Doing so reveals that the source of a high (or low) EPV estimate may come from alternate paths of the possession that were never realized, but were probable enough to have influenced the EPV estimate—an open teammate in a good

shooting location, for instance. These insights, which can be reproduced for any valid NBA possession in our data set, have the potential to reshape the way we quantify players’ actions and decisions.

We make a number of assumptions—mostly to streamline and simplify our modeling and analysis pipeline—that could be relaxed and yield a more precise model. The largest assumption is that the particular coarsened view of a basketball possession that we propose here is marginally semi-Markov. While this serves as a workable first-order approximation, there are cases that clearly violate this assumption, for example, pre-set plays that string together sequences of runs and passes. Future refinements of the model could define a wider set of macrotransitions and coarsened states to encapsulate these motifs, effectively encoding this additional possession structure from the coach’s playbook.

A number of smaller details could also be addressed. For instance, it seems desirable to model rebound outcomes conditional on high resolution information, such as the identities and motion dynamics of potential rebounders; we do not do this, however, and use a constant probability for each team of a rebound going to either the offense or defense. We also do not distinguish between different types of turnovers (steals, bad passes, ball out of bounds, etc.), though this is due to a technical feature of our data set. Indeed, regardless of the complexity and refinement of an EPV model, we stress that the full resolution data still omits key information, such as the positioning of players’ hands and feet, their heights when jumping, and other variables that impact basketball outcomes. As such, analyses based on EPV are best accompanied by actual game film and the insight of a basketball expert.

The computational requirements of estimating EPV curves (and the parameters that generate them) likely limit EPV discussions to academic circles and professional basketball teams with access to the appropriate resources. Our model nevertheless offers a case study whose influence extends beyond basketball. High resolution spatiotemporal data sets are an emerging inferential topic in a number of scientific or business areas, such as climate, security and surveillance, advertising, and gesture recognition. Many of the core methodological approaches in our work, such as using multiresolution transitions and hierarchical spatial models, provide insight beyond the scope of basketball to other spatiotemporal domains.

# APPENDIX A. ADDITIONAL TECHNICAL DETAILS

In this appendix we provide additional details on steps used in fitting multiresolution models and deriving basketball metrics from EPV estimates.

## A.1 Time-Varying Covariates in Macrotransition Entry Model

As revealed in (8), the hazards $\lambda_j^\ell(t)$ are parameterized by spatial effects ($\xi_j^\ell$ and $\tilde{\xi}_j^\ell$ for pass events), as well as coefficients for situation covariates, $\beta_j^\ell$. The covariates used may be different for each macrotransition $j$, but we assume for each macrotransition type the same covariates are used across players $\ell$.

Among the covariates we consider, `dribble` is an indicator of whether the ballcarrier has started dribbling after receiving possession. `ndef` is the distance between the ballcarrier and his nearest defender (transformed to $\log(1+d)$). `ball_lastsec` records the distance traveled by the ball in the previous one second. `closeness` is a categorical variable giving the rank of the ballcarrier's teammates' distance to the ballcarrier. Lastly, `open` is a measure of how open a potential pass receiver is using a simple formula relating the positions of the defensive players to the vector connecting the ballcarrier with the potential pass recipient.

For $j \leq 4$, the pass event macrotransitions, we use `dribble`, `ndef`, `closeness`, and `open`. For shot-taking and turnover events, `dribble`, `ndef`, and `ball_lastsec` are included. Lastly, the shot probability model (which, from (10) has the same parameterization as the macrotransition model) uses `dribble` and `ndef` only. All models also include an intercept term. As discussed in Section 4.1, independent CAR priors are assumed for each coefficient in each macrotransition hazard model.

## A.2 Player Similarity Matrix **H** for CAR Prior

The hierarchical models used for parameters of the macrotransition entry model, discussed in Section 4.1, are based on the idea that players who share similar roles for their respective teams should behave similarly in the situations they face. Indeed, players' positions (point guard, power forward, etc.) encode their offensive responsibilities: point guards move and distribute the ball, small forwards penetrate and attack the basket, and shooting guards get open for three-point shots. Such responsibilities reflect spatiotemporal decision-making tendencies, and therefore informative for our macrotransition entry model (7)–(8).

Rather than use the labeled positions in our data, we define position as a distribution of a player's location during his time on the court. Specifically, we divide the offensive half of the court into 4-square-foot bins (575 total) and count, for each player, the number of data points for which he appears in each bin. Then we stack these counts together into a $L \times 575$ matrix (there are $L = 461$ players in our data), denoted **G**, and take the square root of all entries in **G** for normalization. We then perform non-negative matrix factorization (NMF) on **G** in order to obtain a low-dimensional representation of players' court occupancy that still reflects variation across players (Miller et al. 2013). Specifically, this involves solving:

$$ \hat{\mathbf{G}} = \underset{\mathbf{G}^*}{\text{argmin}} \{D(\mathbf{G}, \mathbf{G}^*)\}, \text{ subject to } \mathbf{G}^* = \left( \underset{L \times r}{\mathbf{U}} \right) \left( \underset{r \times 575}{\mathbf{V}} \right) \text{ and } U_{ij}, V_{ij} \geq 0 \text{ for all } i, j, $$

(A.1)

where $r$ is the rank of the approximation $\hat{\mathbf{G}}$ to **G** (we use $r = 5$), and $D$ is some distance

function; we use a Kullback-Liebler type

$$D(\mathbf{G}, \mathbf{G}^*) = \sum_{i,j} G_{ij} \log (G_{ij}/G_{ij}^*) - G_{ij} + G_{ij}^*.$$

The rows of **V** are non-negative basis vectors for players’ court occupancy distributions (plotted in Figure 8) and the rows of **U** give the loadings for each player. With this factorization, **U**<sub>$i$</sub> (the $i$th row of **U**) provides player $i$’s “position”—a $r$-dimensional summary of where he spends his time on the court. Moreover, the smaller the difference between two players’ positions, $||\mathbf{U}_i - \mathbf{U}_j||$, the more alike are their roles on their respective teams, and the more similar we expect the parameters of their macrotransition models to be a priori.

Heatmaps showing five basis vectors for player court occupancy distributions on a basketball court.

Figure 8: The rows of **V** (plotted above for $r = 5$) are bases for the players’ court occupancy distribution. There is no interpretation to the ordering.

Formalizing this, we take the $L \times L$ matrix **H** to consist of 0s, then set $H_{ij} = 1$ if player $j$ is one of the eight closest players in our data to player $i$ using the distance $||\mathbf{U}_i - \mathbf{U}_j||$ (the cutoff of choosing the closest eight players is arbitrary). This construction of **H** does not guarantee symmetry, which is required for the CAR prior we use, thus we set $H_{ji} = 1$ if $H_{ij} = 1$. For instance, LeBron James’ “neighbors” are (in no order): Andre Iguodala, Harrison Barnes, Paul George, Kobe Bryant, Evan Turner, Carmelo Anthony, Rodney Stuckey, Will Barton, and Rudy Gay.

## A.3 Basis Functions for Spatial Effects $\xi$

Recalling (13), for each player $\ell$ and macrotransition type $j$, we have $\xi_j^\ell(\mathbf{z}) = \sum_{i=1}^d w_{ji}^\ell \phi_{ji}(\mathbf{z})$, where $\{\phi_{ji}, i = 1, \dots, d\}$ are the basis functions for macrotransition $j$. During the inference discussed in Section 4, these basis functions are assumed known. They are derived from a pre-processing step. Heuristically, they are constructed by approximately fitting a simplified macrotransition entry model with stationary spatial effect for each player, then performing NMF to find a low-dimensional subspace (in this function space of spatial effects) that accurately captures the spatial dependence of players’ macrotransition behavior. We now describe this process in greater detail.

Each basis function $\phi_{ji}$ is itself represented as a linear combination of basis functions,

$$\phi_{ji}(\mathbf{z}) = \sum_{k=1}^{d_0} v_{jik} \psi_k(\mathbf{z}), \tag{A.2}$$

where $\{\psi_k, k = 1, \dots, d_0\}$ are basis functions (as the notation suggests, the same basis is used for all $j, i$). The basis functions $\{\psi_k, k = 1, \dots, d_0\}$ are induced by a triangular mesh of $d_0$ vertices (we use $d_0 = 383$) on the court space $\mathbb{S}$. In practice, the triangulation is defined on

a larger region that includes $\mathbb{S}$, due to boundary effects. The mesh is formed by partitioning $\mathbb{S}$ into triangles, where any two triangles share at most one edge or corner; see Figure 9 for an illustration. With some arbitrary ordering of the vertices of this mesh, $\psi_k : \mathbb{S} \to \mathbb{R}$ is the unique function taking value 0 at all vertices $\tilde{k} \neq k$, 1 at vertex $k$, and linearly interpolating between any two points within the same triangle used in the mesh construction. Thus, with this basis, $\phi_{ji}$ (and consequently, $\xi_j^\ell$) are piecewise linear within the triangles of the mesh.

Triangulation of S used to build the functional basis

*Figure 9: Triangulation of $\mathbb{S}$ used to build the functional basis $\{\psi_k, k = 1, \dots, d_0\}$. Here, $d_0 = 383$.*

This functional basis $\{\psi_k, k = 1, \dots, d_0\}$ is used by Lindgren et al. (2011), who show that it can approximate a Gaussian random field with Matérn covariance. Specifically, let $x(\mathbf{z}) = \sum_{k=1}^{d_0} v_k \psi_k(\mathbf{z})$ and assume $(v_1 \dots v_k)' = \mathbf{v} \sim \mathcal{N}(0, \boldsymbol{\Sigma}_{\nu, \kappa, \sigma^2})$. The form of $\boldsymbol{\Sigma}_{\nu, \kappa, \sigma^2}$ is such that the covariance function of $x$ approximates a Matérn covariance:

$$ \text{Cov}[x(\mathbf{z}_1), x(\mathbf{z}_2)] = \boldsymbol{\psi}(\mathbf{z}_1)' \boldsymbol{\Sigma}_{\nu, \kappa, \sigma^2} \boldsymbol{\psi}(\mathbf{z}_2) \approx \frac{\sigma^2}{\Gamma(\nu) 2^{\nu-1}} (\kappa ||\mathbf{z}_1 - \mathbf{z}_2||)^\nu K_\nu (\kappa ||\mathbf{z}_1 - \mathbf{z}_2||), \quad (\text{A.3}) $$

where $\boldsymbol{\psi}(\mathbf{z}) = (\psi_1(\mathbf{z}) \dots \psi_{d_0}(\mathbf{z}))'$. As discussed in Section 4.2, the functional basis representation of a Gaussian process offers computational advantages in that the infinite dimensional field $x$ is given a $d_0$-dimensional representation, as $x$ is completely determined by $\mathbf{v}$. Furthermore, as discussed in Lindgren et al. (2011), $\boldsymbol{\Sigma}_{\nu, \kappa, \sigma^2}^{-1}$ is sparse ((A.3) is actually a Gaussian Markov random field (GMRF) approximation to $x$), offering additional computational savings (Rue 2001).

The GMRF approximation given by (A.2)–(A.3) is actually used in fitting the micro-transition models for offensive players (5). We give the spatial innovation terms $\mu_x^\ell, \mu_y^\ell$ representations using the $\psi$ basis. Then, as mentioned in Section 4.3, (5) is fit independently for each player in our dataset using the software R-INLA.

We also fit simplified versions of the macrotransition entry model, using the $\psi$ basis, in order to determine $\{v_{jik}, k = 1, \dots, d_0\}$, the loadings of the basis representation for $\phi$, (A.2). This simplified model replaces the macrotransition hazards (8) with

$$ \log(\lambda_j^\ell(t)) = c_j^\ell + \sum_{k=1}^{d_0} u_{jk}^\ell \psi_k(\mathbf{z}^\ell(t)) + \mathbf{1}[j \leq 4] \sum_{k=1}^{d_0} \tilde{u}_{jk}^\ell \psi_k(\mathbf{z}_j(t)), \quad (\text{A.4}) $$

thus omitting situational covariates ($\boldsymbol{\beta}_j^\ell$ in (8)) and using the $\psi$ basis representation in place

of $\xi_j^\ell$. Note that for pass events, like (8), we have an additional term based on the pass recipient's location, parameterized by $\{\tilde{u}_{jk}^\ell, k = 1, \dots, d_0\}$. As discussed in Section 4.3, parameters in (A.4) can be estimated by running a Poisson regression. We perform this independently for all players $\ell$ and macrotransition types $j$ using the R-INLA software. Like the microtransition model, we fit (A.4) separately for each player across $L = 461$ processors (each hazard type $j$ is run in serial), each requiring at most 32GB RAM and taking no more than 16 hours.

For each macrotransition type $j$, point estimates $\hat{u}_{jk}^\ell$ are exponentiated<sup>6</sup>, so that $[\mathbf{U}_j]_{\ell k} = \exp(\hat{u}_{jk}^\ell)$. We then perform NMF (A.1) on $\mathbf{U}_j$:

$$ \mathbf{U}_j \approx \left( \underset{L \times d}{\mathbf{Q}_j} \right) \left( \underset{d \times d_0}{\mathbf{V}_j} \right). \eqno(A.5) $$

Following the NMF example in Section A.2, the rows of $\mathbf{V}_j$ are bases for the variation in coefficients $\{u_{jk}^\ell, k = 1, \dots, d_0\}$ across players $\ell$. As $1 \le k \le d_0$ indexes points on our court triangulation (Figure 9), such bases reflect structured variation across space. We furthermore use these terms as the coefficients for (A.2), the functional basis representation of $\phi_{ji}$, setting $v_{jik} = [\mathbf{V}_j]_{ik}$. Equivalently, we can summarize our spatial basis model as:

$$ \xi_j^\ell(\mathbf{z}) = [\mathbf{w}_j^\ell]' \phi_j(\mathbf{z}) = [\mathbf{w}_j^\ell]' \mathbf{V}_j \psi(\mathbf{z}). \eqno(A.6) $$

The preprocessing steps described in this section—fitting a simplified macrotransition entry model (A.4) and performing NMF on the coefficient estimates (A.5)—provide us with basis functions $\phi_{ji}(\mathbf{z})$ that we treat as fixed and known during the modeling and inference discussed in Section 4.

Note that an analogous expression for (A.6) is used for $\tilde{\xi}_j^\ell$ in terms of $\tilde{\mathbf{w}}_j^\ell$ and $\tilde{\mathbf{V}}_j$ for pass events; however, for the spatial effect $\xi_s^\ell$ in the shot probability model, we simply use $\mathbf{V}_5$. Thus, the basis functions for the shot probability model are the same as those for the shot-taking hazard model.

## A.4 Calculating EPVA: Baseline EPV for League-Average Player

To calculate the baseline EPV for a league-average player possessing the ball in player $\ell$'s shoes, denoted $\nu_t^{r(\ell)}$ in (19), we start by considering an alternate version of the transition probability matrix between coarsened states $\mathbf{P}$. For each player $\ell_1, \dots, \ell_5$ on offense, there is a disjoint subset of rows of $\mathbf{P}$, denoted $\mathbf{P}_{\ell_i}$, that correspond to possession states for player $\ell_i$. Each row of $\mathbf{P}_{\ell_i}$ is a probability distribution over transitions in $\mathcal{C}$ given possession in a particular state. Technically, since states in $\mathcal{C}_{\text{poss}}$ encode player identities, players on different teams do not share all states which they have a nonzero probability of transitioning to individually. To get around this, we remove the columns from each $\mathbf{P}_{\ell_i}$ corresponding to passes to players not on player $\ell_i$'s team, and reorder the remaining columns according to the position (guard, center, etc.) of the associated pass recipient. Thus, the interpretation of transition distributions $\mathbf{P}_{\ell_i}$ across players $\ell_i$ is as consistent as possible.

<sup>6</sup>The reason for exponentiation is because estimates $\hat{u}_{jk}^\ell$ inform the log hazard, so exponentiation converts these estimates to a more natural scale of interest. Strong negative signals among the $\hat{u}_{jk}^\ell$ will move to 0 in the entries of $\mathbf{U}_j$ and not be very influential in the matrix factorization (A.5), which is desirable for our purposes.

We create a baseline transition profile of a hypothetical league-average player by averaging these transition probabilities across all players: (with slight abuse of notation) let $\mathbf{P}_r = \sum_{\ell=1}^L \mathbf{P}_\ell / L$. Using this, we create a new transition probability matrix $\mathbf{P}_r(\ell)$ by replacing player $\ell$'s transition probabilities ($\mathbf{P}_\ell$) with the league-average player's ($\mathbf{P}_r$). The baseline (league-average) EPV at time $t$ is then found by evaluating $\nu_t^{r(\ell)} = \mathbb{E}_{\mathbf{P}_r(\ell)}[X | C_t]$. Note that $\nu_t^{r(\ell)}$ depends only on the coarsened state $C_t$ at time $t$, rather than the full history of the possession, $\mathcal{F}_t^{(Z)}$, as in $\nu_t$ (4). This "coarsened" baseline $\nu_t^{r(\ell)}$ exploits the fact that, when averaging possessions over the entire season, the results are (in expectation) identical to using a full-resolution baseline EPV that assumes the corresponding multiresolution transition probability models for this hypothetical league-average player.

## APPENDIX B. DATA AND CODE

The Git repository [https://github.com/dcervone/EPVDemo](https://github.com/dcervone/EPVDemo) contains a one game sample of optical tracking data (csv), along with R code for visualizing model results and reproducing EPV calculations. Pre-computed results of computationally-intensive steps are also included as Rdata files, and can be loaded to save time and resources. A reproducible knitr tutorial, `EPV_demo.Rnw`, introduces the data and demonstrates core code functionality.

## REFERENCES

Besag, J. (1974), "Spatial Interaction and the Statistical Analysis of Lattice Systems," *Journal of the Royal Statistical Society: Series B (Methodological)*, 36(2), 192–236.

Bukiet, B., Harold, E. R., & Palacios, J. L. (1997), "A Markov Chain Approach to Baseball," *Operations Research*, 45(1), 14–23.

Burke, B. (2010), "Win Probability Added (WPA) Explained," www.advancedfootballanalytics.com, (website).

Cox, D. R. (1975*a*), "A Note on Partially Bayes Inference and the Linear Model," *Biometrika*, 62(3), 651–654.

Cox, D. R. (1975*b*), "Partial Likelihood," *Biometrika*, 62(2), 269–276.

Franks, A., Miller, A., Bornn, L., & Goldsberry, K. (2015), "Characterizing the Spatial Structure of Defensive Skill in Professional Basketball," *Annals of Applied Statistics*, .

Gneiting, T., Balabdaoui, F., & Raftery, A. E. (2007), "Probabilistic forecasts, calibration and sharpness," *Journal of the Royal Statistical Society: Series B (Statistical Methodology)*, 69(2), 243–268.

Goldner, K. (2012), "A Markov Model of Football: Using Stochastic Processes to Model a Football Drive," *Journal of Quantitative Analysis in Sports [online]*, 8(1).

Higdon, D. (2002), "Space and Space-Time Modeling Using Process Convolutions," in *Quantitative Methods for Current Environmental Issues*, New York, NY: Springer, pp. 37–56.

Hollinger, J. (2005), *Pro Basketball Forecast, 2005-06*, Washington, D.C: Potomac Books.

Ihler, A., Hutchins, J., & Smyth, P. (2006), “Adaptive Event Detection with Time-Varying Poisson Processes,” in *Proceedings of the 12th ACM SIGKDD International Conference on Knowledge Discovery and Data Mining*, New York, NY: ACM, pp. 207–216.

Kemeny, J. G., & Snell, J. L. (1976), *Finite Markov chains: with a new appendix “Generalization of a fundamental matrix”* Springer.

Lindgren, F., Rue, H., & Lindström, J. (2011), “An Explicit Link Between Gaussian Fields and Gaussian Markov Random Fields: the Stochastic Partial Differential Equation Approach,” *Journal of the Royal Statistical Society: Series B (Methodological)*, 73(4), 423–498.

Lock, D., & Nettleton, D. (2014), “Using random forests to estimate win probability before each play of an NFL game,” *Journal of Quantitative Analysis in Sports*, 10(2), 197–205.

Miller, A., Bornn, L., Adams, R., & Goldsberry, K. (2013), “Factorized Point Process Intensities: A Spatial Analysis of Professional Basketball,” in *Proceedings of the 31st International Conference on Machine Learning*, pp. 235–243.

Omidiran, D. (2011), “A New Look at Adjusted Plus/Minus for Basketball Analysis,” *MIT Sloan Sports Analytics Conference [online]*, 2011.

Prentice, R. L., Kalbfleisch, J. D., Peterson Jr, A. V., Flournoy, N., Farewell, V., & Breslow, N. (1978), “The Analysis of Failure Times in the Presence of Competing Risks,” *Biometrics*, pp. 541–554.

Quiñonero-Candela, J., & Rasmussen, C. E. (2005), “A Unifying View of Sparse Approximate Gaussian Process Regression,” *The Journal of Machine Learning Research*, 6, 1939–1959.

Rasmussen, C. E. (2006), *Gaussian Processes for Machine Learning*, Cambridge, MA: MIT Press.

Rue, H. (2001), “Fast sampling of Gaussian Markov random fields,” *Journal of the Royal Statistical Society: Series B (Statistical Methodology)*, 63(2), 325–338.

Rue, H., Martino, S., & Chopin, N. (2009), “Approximate Bayesian Inference for Latent Gaussian Models by Using Integrated Nested Laplace Approximations,” *Journal of the Royal Statistical Society: Series B (Methodological)*, 71(2), 319–392.

Shao, X., & Li, L. (2011), “Data-Driven Multi-Touch Attribution Models,” in *Proceedings of the 17th ACM SIGKDD International Conference on Knowledge Discovery and Data Mining*, New York, NY: ACM, pp. 258–264.

Thomas, A., Ventura, S. L., Jensen, S. T., & Ma, S. (2013), “Competing Process Hazard Function Models for Player Ratings in Ice Hockey,” *The Annals of Applied Statistics*, 7(3), 1497–1524.

Wong, W. H. (1986), “Theory of Partial Likelihood,” *The Annals of Statistics*, pp. 88–123.

Yang, T. Y., & Swartz, T. (2004), “A Two-Stage Bayesian Model for Predicting Winners in Major League Baseball,” *Journal of Data Science*, 2(1), 61–73.