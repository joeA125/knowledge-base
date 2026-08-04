A catalog of all wiki pages, organised by type.

## Navigation

- [[overview]] — high-level map of the domains covered by this wiki
- Dashboards: [[health|Health]] · [[reinforcement|Reinforcement]] · [[sources|Source Tracking]]

## Entities

- [[ashish-vaswani]] · [[noam-shazeer]] · [[niki-parmar]] · [[jakob-uszkoreit]] · [[llion-jones]] · [[aidan-gomez]] · [[lukasz-kaiser]] · [[illia-polosukhin]] — co-authors of "Attention Is All You Need"
- [[ralf-herbrich]] · [[thore-graepel]] — co-authors of TrueSkill
- [[tom-minka]] — creator of Expectation Propagation, co-author of TrueSkill
- [[mark-glickman]] — Statistician, creator of the Glicko and Glicko-2 rating systems
- [[dzmitry-bahdanau]] — First author of the Bahdanau attention paper
- [[kyunghyun-cho]] — Developer of GRU, co-author of Bahdanau attention
- [[yoshua-bengio]] — Deep learning pioneer, co-author of Bahdanau attention
- [[oriol-vinyals]] — Lead author of Pointer Networks and Order Matters
- [[meire-fortunato]] · [[navdeep-jaitly]] — co-authors of Pointer Networks
- [[wojciech-zaremba]] — First author of RNN Regularization
- [[ilya-sutskever]] — Co-author of RNN Regularization, seq2seq, VLAE, and GPT
- [[kaiming-he]] · [[xiangyu-zhang]] · [[shaoqing-ren]] · [[jian-sun]] — ResNet authors
- [[fisher-yu]] · [[vladlen-koltun]] — dilated convolutions
- [[alex-graves]] · [[greg-wayne]] · [[ivo-danihelka]] — Neural Turing Machines
- [[jared-kaplan]] · [[sam-mccandlish]] · [[dario-amodei]] — Scaling Laws
- [[diederik-kingma]] — Co-creator of VAE, co-author of VLAE
- [[alec-radford]] — Lead author of GPT (2018)
- [[jacob-devlin]] — Lead author of BERT (2019)
- [[jianhui-chen]] · [[james-little]] — Sports Camera Calibration via Synthetic Data
- [[floriane-magera]] · [[marc-van-droogenbroeck]] — ProCC benchmarking; SoccerNet-v2
- [[tom-decroos]] · [[jesse-davis]] — SPADL/VAEP
- [[maaike-van-roy]] · [[pieter-robberechts]] — the xT vs VAEP critical comparison
- [[karun-singh]] — Creator of expected threat (xT)
- [[garry-gelade]] — Practitioner; Bradley-Terry model of 1v1 duel ability
- [[william-spearman]] — Hudl; pass probability model, PPCF and OBSO
- [[daniel-cervone]] · [[alex-damour]] · [[kirk-goldsberry]] — the NBA martingale EPV paper
- [[luke-bornn]] — NBA EPV, pitch control, and the soccer EPV framework
- [[javier-fernandez]] — Soccer EPV framework, SoccerMap, and pitch control
- [[keisuke-fujii]] — Senior author of VDEP, GVDEP, DRSO, C-OBSO, NMSTPP and the SPC framework
- [[calvin-yeung]] — Lead author of NMSTPP and the game-theoretic SPC framework
- [[kosuke-toda]] — Lead author of VDEP
- [[rikuhei-umemoto]] — Lead author of GVDEP and DRSO; the defensive-positioning line
- [[masakiyo-teranishi]] — Lead author of C-OBSO; the trajectory-modelling line
- [[kazushi-tsutsui]] — Co-author of C-OBSO and GVDEP
- [[kazuya-takeda]] — Co-author of C-OBSO and the 2020 trajectory work
- [[keisuke-kushiro]] — Co-author of VDEP (supervision)
- [[tony-sit]] — Statistician, co-author of NMSTPP
- [[ian-simpson]] — Seq2Event and the poss-util metric
- [[david-hirnschall]] · [[robert-bajons]] — the path-signature possession paper
- [[koffi-amezouwui]] · [[brigitte-gelein]] · [[matthieu-marbac]] · [[anthony-sorel]] — possession mixture-model clustering
- [[sandeep-narayanan]] — Bayesian point-process model of football events
- [[miru-hong]] · [[minho-lee]] · [[sang-ki-ko]] · [[geonhee-jo]] · [[jae-hee-so]] — EventGPT and ScoutGPT
- [[pascal-bauer]] — Chair for Sports Analytics, Saarland
- [[tiago-mendes-neves]] · [[luis-meireles]] · [[joao-mendes-moreira]] — Valuing Players Over Time; Large Event Models
- [[andrei-shelopugin]] — Independent researcher; EPV of control and duel actions, PCR
- [[alexander-sirotkin]] — Glicko-2 duel-rating and league-rating papers
- [[openai]] · [[google-brain]] · [[google-deepmind]] · [[google-research]] · [[microsoft-research]] — research organisations
- [[university-of-toronto]] · [[jacobs-university-bremen]] · [[universite-de-montreal]] — universities
- [[universidade-do-porto]] · [[inesc-tec]] — the Porto football-analytics group
- [[nagoya-university]] · [[kyoto-university]] — the Fujii group and the VDEP lead author
- [[fc-porto]] · [[fc-barcelona]] — clubs with co-author affiliations
- [[stats-perform]] — Sports data provider (STATS LLC / Opta)
- [[data-stadium]] — Japanese sports data provider; J-League data

## Concepts

### Architectures and Deep Learning
- [[transformer]] — Attention-only sequence transduction architecture
- [[attention-mechanism]] · [[additive-attention]] · [[scaled-dot-product-attention]] · [[multi-head-attention]]
- [[pointer-network]] · [[read-process-write]] · [[neural-turing-machine]]
- [[positional-encoding]] · [[encoder-decoder]] · [[encoder-decoder-bottleneck]] · [[feed-forward-network]]
- [[recurrence]] — Sequential hidden-state processing underlying RNNs
- [[lstm]] · [[gated-recurrent-unit]] · [[bidirectional-rnn]]
- [[graph-neural-network]] — Message passing over graph-structured data; permutation equivariance
- [[message-passing]] — The pattern shared by graphical-model inference and GNNs
- [[convolution]] · [[dilated-convolution]] · [[fully-convolutional-network]] · [[feature-pyramid-network]]
- [[residual-connections]] · [[pre-activation-resnet]] · [[batch-normalization]] · [[layer-normalization]]
- [[dropout]] · [[dropout-for-rnns]] · [[recurrent-dropout]] · [[label-smoothing]] · [[regularization]]
- [[adam-optimizer]] · [[teacher-forcing]] · [[multi-task-learning]] · [[constrained-decoding]]
- [[representation-learning]] — Learning what to feed a model rather than hand-specifying it
- [[theory-based-modelling]] — Encoding domain structure as a model, often to feed a learned one
- [[feature-engineering]] · [[tokenization]] · [[player-embedding]] · [[pre-train-then-fine-tune]]

### Generative and Sequence Models
- [[generative-model]] — Models of the data distribution itself; families, uses, and the causal caveat
- [[autoregressive-model]] · [[variational-autoencoder]] · [[variational-lossy-autoencoder]] · [[conditional-gan]]
- [[trajectory-prediction]] — Positions of interacting agents; VRNN, GVRNN, prediction-as-reference
- [[event-prediction]] — Forecasting the next event; how forecasting yields valuation metrics
- [[imitation-learning]] — Learning a policy by mimicking; imitation as a measuring instrument
- [[reinforcement-learning]] — What sports valuation borrows from RL, and what it does not
- [[rlhf]] · [[gpt]] · [[bert]] · [[masked-language-model]] · [[chain-of-thought]] · [[react]]
- [[scaling-laws]] · [[retrieval-augmented-generation]] · [[ai-agent]] · [[tool-use]] · [[agent-memory]]

### Statistics, Processes and Inference
- [[stochastic-process]] — Umbrella for the process family and what the process view buys
- [[martingale]] · [[point-process]] · [[neural-temporal-point-process]] · [[gaussian-process]]
- [[poisson-binomial]] — Aggregating Bernoulli trials with differing probabilities; expected-versus-actual
- [[absorbing-markov-chain]] · [[markov-game]] · [[value-iteration]] · [[multiresolution-modelling]]
- [[game-theory]] — Strategic interaction, Nash equilibrium, and learned payoffs
- [[survival-analysis]] — Time-to-event modelling via hazards; censoring
- [[competing-risks]] · [[car-prior]] · [[inla]] · [[bayesian-inference]] · [[bayes-theorem]]
- [[factor-graph]] · [[approximate-message-passing]] · [[expectation-propagation]] · [[gaussian-density-filtering]]
- [[mixture-model]] · [[expectation-maximization]] · [[identifiability]] · [[clustering]]
- [[model-selection]] — Choosing complexity; the vault's asserted free parameters, and what GVDEP superseded
- [[policy-modelling]] · [[counterfactual-baseline]] · [[counterfactual-simulation]] · [[temporal-discounting]]
- [[kl-divergence]] · [[non-negative-matrix-factorization]] · [[eigenvector]] · [[path-signature]] · [[smoothing]]

### Machine Learning Practice and Evaluation
- [[probabilistic-classification]] — Predicting probabilities rather than labels, and why it matters here
- [[probability-calibration]] · [[uncertainty-quantification]] · [[class-imbalance-evaluation]]
- [[rare-event-proxy-targets]] — Predicting a frequent correlate when the target is too rare
- [[sample-weighting]] · [[selection-bias]] · [[positive-unlabeled-learning]]
- [[gradient-boosting]] · [[random-forest]] · [[shap]] · [[interpretability]]
- [[split-half-reliability]] · [[predictive-validity]] · [[adjusted-rand-index]] · [[jaccard-index]]

### Football and Sports Analytics
- [[action-valuation]] — Valuing individual actions via change in game-state quality
- [[expected-possession-value]] — Umbrella: a possession's worth, and the four things the term means
- [[expected-goals]] · [[expected-threat]] · [[vaep]] · [[martingale-epv]] · [[pass-carry-reward]] · [[on-ball-value]]
- [[xsot]] — Expected shot on target, and its off-ball counterpart; game-theoretic payoffs
- [[defensive-valuation]] · [[vdep]] · [[gvdep]] · [[drso]] · [[duel-skill-rating]] · [[symmetrical-duel-valuation]]
- [[off-ball-value]] · [[space-creation]] · [[obso]] · [[c-obso]] · [[pitch-control]]
- [[pass-probability-model]] — Who receives a pass, from intercept and control times; root of the PPCF/OBSO chain
- [[receiving-efficiency]] — Receptions and interceptions against a model's expectation
- [[possession-risk]] · [[effective-playing-time]] · [[intent-vs-outcome-valuation]]
- [[dynamic-pressure-lines]] · [[tactical-analysis]] · [[probability-surface]] · [[soccermap]] · [[single-pixel-supervision]]
- [[structured-model-decomposition]] — Estimating a quantity by recombining subcomponents
- [[large-event-model]] · [[nmstpp]] · [[sig-model]] · [[seq2event]] · [[eventgpt]] · [[scoutgpt]] · [[hpus]] · [[lpv]]
- [[recruitment]] · [[transfer-performance-prediction]] · [[league-strength-rating]]
- [[player-rating-time-series]] · [[performance-volatility]] · [[player-development-curve]]
- [[spadl]] · [[event-stream-data]] · [[optical-tracking-data]]
- [[bradley-terry-model]] · [[elo-rating-system]] · [[glicko-rating-system]] · [[trueskill]]

### Computer Vision
- [[multi-object-tracking]] — Following objects across frames with consistent identities
- [[object-detection]] · [[game-state-reconstruction]] · [[camera-calibration]] · [[jac-metric]]
- [[homography]] · [[radial-distortion]] · [[image-alignment]] · [[enhanced-correlation-coefficient]]
- [[semantic-segmentation]] · [[optical-flow]] · [[siamese-network]] · [[combinatorial-optimisation]]

## Syntheses

- [[action-valuation-frameworks-compared]] — **Football Modelling Tasks Compared**: the six distinct tasks (valuation, forecasting, clustering, counterfactual/transfer, tactical, prescription), how each is validated, why possession metrics outpredict goals, the seven axes of valuation design, and why prescription turns out to be the oldest task rather than the newest

## Questions

Open investigations — a question, what can be settled from held sources, and what would settle the rest. Grouped by why the question exists, which determines who could answer it.

### Component-level benchmarking gaps
- [[pitch-control-traditions-compared]] — **Do the two pitch-control traditions agree?** They answer different questions (pass reception vs spatial dominance) and are unequally validated — PPCF against 5,471 held-out pass receivers, the Gaussian model against nothing directly.
- [[shot-value-formulations-compared]] — **Are the four shot-value formulations interchangeable?** One of six pairwise comparisons exists.
- [[tracking-error-propagation]] — **Does tracking error propagate into value estimates?** Partially answered for *incomplete observation*; positional error and identity switches remain open.

### Untested assumptions in held work
- [[free-parameters-load-bearing]] — **Are the free parameters load-bearing?** $\gamma$, $\epsilon$, $k$, 4 s remain asserted; $C$ superseded by [[gvdep]].
- [[vaep-conceding-classifier]] — **Is VAEP's conceding classifier broken, or just unthresholdable?** F1 = 0.000 is near-guaranteed at a 0.23% base rate; Spearman et al. demonstrate the threshold mechanism directly.

### Claims this vault generated
- [[within-season-variation-noise-or-signal]] — **Is within-season variation noise or signal?** Shown here to be the *same quantity* under two names.
- [[observed-versus-optimal-decisions]] — **Do players decide suboptimally, or do the models only think so?** Three ways the gap could be artefactual, none tested.
- [[handcrafted-features-rule]] — **Is the handcrafted-features rule right?** A reconciliation invented here, never tested against a fourth case.

## Conversations

## Source Summaries

- [[attention-is-all-you-need]] — "Attention Is All You Need" (Vaswani et al., 2017)
- [[bayesian-true-skill-rating]] — "TrueSkill: A Bayesian Skill Rating System" (Herbrich et al., 2006)
- [[neural-machine-translation]] — "Neural Machine Translation by Jointly Learning to Align and Translate" (Bahdanau et al., 2014)
- [[pointer-networks]] — "Pointer Networks" (Vinyals et al., 2015)
- [[rnn-regularisation]] — "Recurrent Neural Network Regularization" (Zaremba et al., 2014)
- [[identity-mapping-residual-networks]] — "Identity Mappings in Deep Residual Networks" (He et al., 2016)
- [[sequence-to-sequence-sets]] — "Order Matters: Sequence to Sequence for Sets" (Vinyals et al., 2016)
- [[context-aggregation-dilated-convolutions]] — "Multi-Scale Context Aggregation by Dilated Convolutions" (Yu & Koltun, 2016)
- [[neural-turing-machines]] — "Neural Turing Machines" (Graves, Wayne & Danihelka, 2014)
- [[scaling-neural-language-models]] — "Scaling Laws for Neural Language Models" (Kaplan et al., 2020)
- [[variational-lossy-autoencoders]] — "Variational Lossy Autoencoder" (Chen et al., 2017)
- [[universal-prompt-retrieval-zero-shot-eval]] — "UPRISE" (Cheng et al., 2023)
- [[autogressive-language-model-retrieval]] — "Shall We Pretrain Autoregressive LMs with Retrieval?" (Wang et al., 2023)
- [[autogressive-language-model-retrieval-iterative]] — "ITER-RETGEN" (Shao et al., 2023)
- [[augmented-llms-parametric-guiding]] — "PKG: Parametric Knowledge Guiding" (Luo et al., 2023)
- [[agi-definition]] — "A Definition of AGI" (Hendrycks et al., 2025)
- [[llm-factcheck-consistency-certainty]] — "PCC: Fact-Checking via Probabilistic Certainty and Consistency" (Wang et al., 2026)
- [[soccernet-game-state-reconstruction]] — "SoccerNet Game State Reconstruction" (Somers et al., 2024)
- [[soccernet-game-state-reconstruction-improvement]] — "From Broadcast to Minimap: SOTA GSR" (Golovkin et al., 2024)
- [[soccernet-v2-action-spotting]] — "Camera Calibration and Player Localization in SoccerNet-v2" (Cioppa et al., 2021)
- [[detection-tracking-football-broadcast-footage]] — "Multi-Class Detection and Tracking in Soccer Broadcast" (Tshiani, 2025)
- [[computer-vision-football-review]] — "CV Technology for Football Videos" (Zheng et al., 2025)
- [[chain-of-thought-reasoning-llms]] — "Chain-of-Thought Prompting Elicits Reasoning in LLMs" (Wei et al., 2022)
- [[synergising-reasoning-acting-llms]] — "ReAct: Synergizing Reasoning and Acting in LLMs" (Yao et al., 2023)
- [[training-lm-follow-instructions-with-human-feedback]] — "InstructGPT" (Ouyang et al., 2022)
- [[rag-intense-nlp-tasks]] — "RAG for Knowledge-Intensive NLP Tasks" (Lewis et al., 2020)
- [[tvcalib-camera-calibration-football]] — "TVCalib" (Theiner & Ewerth, 2023)
- [[sports-camera-calibration-synthetic-data]] — "Sports Camera Calibration via Synthetic Data" (Chen & Little, 2019)
- [[amateur-football-analytics-computer-vision]] — "Amateur Football Analytics Using Computer Vision" (Mavrogiannis, 2021)
- [[language-understanding-gpt]] — "Improving Language Understanding by Generative Pre-Training" (Radford et al., 2018)
- [[bert-bidirectional-transformers]] — "BERT" (Devlin et al., 2019)
- [[camera-calibration-benchmarking]] — "ProCC" (Magera et al., 2025)
- [[ai-agent-architecture-breakdown]] — "AI Agent Architecture Breakdown" (technical article, 2026)
- [[eigenvectors-explained]] — "Eigenvectors Explained" (tutorial article, 2026)
- [[physics-based-pass-probabilities]] — "Physics-Based Modeling of Pass Probabilities in Soccer" (Spearman et al., MIT Sloan 2017) — the intercept/control model, PPCF, and the vault's earliest prescriptive method
- [[evaluating-football-player-actions]] — "Actions Speak Louder than Goals" (Decroos et al., 2019)
- [[multiresolution-stochastic-process-nba-possessions]] — "A Multiresolution Stochastic Process Model for Predicting Basketball Possession Outcomes" (Cervone et al., 2016)
- [[on-ball-actions-football-xt-vs-vaep]] — "Valuing On-the-Ball Actions in Soccer: xT vs VAEP" (Van Roy et al., 2020)
- [[transformer-point-process-football-event-modelling]] — "Transformer-Based NMSTPP for Football Match Events" (Yeung et al., 2023)
- [[understanding-football-possessions-path-signatures]] — "The Path to a Goal: Understanding Soccer Possessions via Path Signatures" (Hirnschall & Bajons, 2025)
- [[football-event-sequences-point-process-mixture]] — "Model-Based Clustering of Football Event Sequences" (Amezouwui et al., 2025)
- [[eventgpt-player-impact-team-action-sequences]] — "EventGPT" (Lee, Hong et al., 2025)
- [[scoutgpt-counterfactual-player-valuation]] — "Modeling Matches as Language" (Hong et al., 2026)
- [[football-performance-time-series]] — "Valuing Players Over Time" (Mendes-Neves, Meireles & Mendes-Moreira)
- [[epv-control-duel-skills-football]] — "Expected Possession Value of Control and Duel Actions" (Shelopugin, preprint)
- [[expected-value-possession-framework]] — "A Framework for the Fine-Grained Evaluation of the Instantaneous Expected Value of Soccer Possessions" (Fernández, Bornn & Cervone, 2020)
- [[football-defence-evaluation-vdep]] — "Evaluation of Soccer Team Defense Based on Prediction Models of Ball Recovery and Being Attacked" (Toda et al., PLOS ONE 2022)
- [[generalized-vdep-euro-location-analysis]] — "Location Analysis of Players in UEFA EURO 2020 and 2022 using Generalized VDEP" (Umemoto, Tsutsui & Fujii, 2022) — GVDEP
- [[team-defense-positioning-counterfactuals]] — "Evaluation of Team Defense Positioning by Computing Counterfactuals using StatsBomb 360 data" (Umemoto & Fujii, StatsBomb 2023) — EF-OBSO and DRSO
- [[creating-scoring-opportunities-trajectory-prediction]] — "Evaluation of Creating Scoring Opportunities for Teammates in Soccer via Trajectory Prediction" (Teranishi et al., MLSA 2022/23)
- [[beyond-expected-goals]] — "Beyond Expected Goals" (Spearman, MIT Sloan 2018) — OBSO
- [[optimal-decisions-shot-taking-situations]] — "A Strategic Framework for Optimal Decisions in Football 1-vs-1 Shot-Taking Situations" (Yeung & Fujii, Complex & Intelligent Systems 2024) — game theory, xSOT
