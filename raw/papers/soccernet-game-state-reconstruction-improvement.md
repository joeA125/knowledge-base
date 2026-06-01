# From Broadcast to Minimap: Achieving State-of-the-Art SoccerNet Game State Reconstruction

Vladimir Golovkin* Nikolay Nemtsev Vasyl Shandyba Oleg Udin
Nikita Kasatkin Pavel Kononov Anton Afanasiev Sergey Ulasen Andrei Boiarov
Constructor Tech, Sofia

## Abstract

*Game State Reconstruction (GSR), a critical task in Sports Video Understanding, involves precise tracking and localization of all individuals on the football field—players, goalkeepers, referees, and others—in real-world coordinates. This capability enables coaches and analysts to derive actionable insights into player movements, team formations, and game dynamics, ultimately optimizing training strategies and enhancing competitive advantage. Achieving accurate GSR using a single-camera setup is highly challenging due to frequent camera movements, occlusions, and dynamic scene content. In this work, we present a robust end-to-end pipeline for tracking players across an entire match using a single-camera setup. Our solution integrates a fine-tuned YOLOv5m for object detection, a SegFormer-based camera parameter estimator, and a DeepSORT-based tracking framework enhanced with re-identification, orientation prediction, and jersey number recognition. By ensuring both spatial accuracy and temporal consistency, our method delivers state-of-the-art game state reconstruction, securing first place in the SoccerNet Game State Reconstruction Challenge 2024 and significantly outperforming competing methods.*

## 1. Introduction

Game state reconstruction in football analytics represents a transformative step toward understanding and optimizing player performance, team strategies, and tactical decision-making during matches. By reconstructing the positions, roles, and identities of all individuals on the field—players, goalkeepers, referees, and others—coaches and analysts can derive actionable insights into player movements, team formations, and game dynamics. This capability is particularly critical in professional football, where precise tracking and role identification enable data-driven training strategies and  enhance competitive advantage.

At its core, the SoccerNet Game State Reconstruction (GSR) challenge [46] is a multiple object tracking (MOT) task, but with additional complexities that significantly elevate the difficulty. Beyond simply localizing athletes in 2D pitch coordinates, the task requires determining their roles (e.g., player, goalkeeper, referee), jersey numbers, and team affiliations (left or right relative to the camera viewpoint). These requirements introduce unique challenges, such as handling occlusions, distinguishing between players with similar appearances, and accurately associating identities across fragmented trajectories. Furthermore, the single-camera setup commonly used in football broadcasts introduces erratic camera movements, varying perspectives, and dynamic scene content, making accurate tracking and localization even more challenging.

Our work presents a robust end-to-end pipeline for tackling the SoccerNet GSR challenge, achieving state-of-the-art performance and securing first place in the 2024 competition. Key contributions of this paper include:

* A Modular Pipeline Design ensures high performance, flexibility, and seamless integration of new modules.
* Unique Camera Parameters Network, which enables accurate mapping of detected objects from image-space to real-world pitch coordinates.
* Pitch Localization with Keypoints Refinement: Our pipeline enhances camera parameter predictions by detecting pitch line intersections and minimizing reprojection errors, ensuring precise and consistent athlete localization.
* Innovative Post-Processing: We introduce a sophisticated post-processing stage that refines and merges short tracklets into longer, consistent trajectories, significantly reducing fragmentation and identity swaps.
* Superior Performance: Our pipeline achieved a GS-HOTA score of **63.81**, significantly outperforming the second-place solution (43.15).

# 2. Related Works

Since game state reconstruction was first introduced in 2024, few papers explicitly address the task. However, it can be decomposed into four critical subtasks: **player detection and tracking**, **pitch localization**, **team recognition**, and **jersey number detection**.

**Players detection and tracking** in sports analytics is dominated by the tracking-by-detection paradigm, which typically combines one-stage object detectors like YOLOv8 [19] or other [3–5, 44, 61][45, 50, 55, 56] with DeepSORT-like algorithms [52, 59], augmented by pre-trained re-identification (ReID) feature extractors. While these methods enable real-time performance due to their computational efficiency, they struggle with tracklet fragmentation in dynamic scenes and identity swaps in crowded scenarios. These issues stem from Kalman filter limitations in predicting non-linear motion [22] and the challenges of ReID in football, where uniform similarity, occlusions, and drastic pose variations degrade feature matching [8, 9, 17, 37, 40, 49, 51]. An alternative approach, tracking-by-attention [57, 59], integrates detection and tracking into a unified transformer-based framework, achieving state-of-the-art results on benchmarks like DanceTrack [47] and MOTChallenge [10]. However, these methods prioritize spatial accuracy over temporal consistency, often neglecting smooth tracklet association, which is critical for sports analytics. Additionally, the computational complexity of transformer-based tracking makes it impractical for real-time applications.

**Pitch localization** is crucial for game state reconstruction, enabling the conversion of player positions from image space to real-world coordinates by estimating homography parameters. The most common method relies on detecting correspondences between predicted keypoints [7, 21, 34] or extracting field lines and circles using semantic segmentation [18, 58], aligning them with a predefined field model using RANSAC [12]. While keypoint-based methods are computationally efficient, they suffer from instability in broadcast footage, where only a small subset of field markings is visible at any moment.

Alternative approaches estimate homography directly via CNN-based regression [48] or use search-based optimization, matching image features with a database of pre-computed homographies [6, 41]. While these methods offer greater stability than keypoint-based approaches, they require large-scale databases for robust feature matching and still tend to be less precise and often require post-prediction refinement.

The refinement stage improves accuracy by minimizing reprojection error [34, 42, 49], with temporal consistency enforced through smoothing techniques like Savitzky-Golay filtering [13] and outlier removal.

In **Team Recognition** task, most modern player-to-team assignment methods follow a pipeline similar to that described in [46], which consists of feature extraction and clustering. In the feature extraction stage, player appearance information is collected, often in the form of color histograms or Re-ID embeddings trained via contrastive learning [33]. Extracted features are then clustered using unsupervised methods such as DBSCAN [11] or k-means [31], grouping players into up to 2–5 clusters corresponding to referees, field players, and goalkeepers [46]. An alternative approach, **Associative Embedding (AE)**, eliminates the need for clustering by training a CNN to generate team-discriminative embeddings directly [20]. Unlike clustering-based pipelines, AE does not require explicit position-based assignment, making it more adaptable to different sports and camera views.

**Jersey Number Detection** methods are broadly categorized into two approaches: (1) adaptations of general-purpose OCR techniques [26, 27, 46] and (2) custom solutions tailored for sports scenarios [2, 14, 23, 25, 29]. The first approach typically follows a two-step pipeline: text detection on localized player regions using methods like *Differentiable Binarization* [27], followed by recognition via models such as *Show, Attend and Read* [26]. While these methods offer development simplicity, their accuracy suffers in scenarios with distant players, occlusions, or sideways orientations. Custom solutions address domain-specific challenges by integrating sports-aware designs, such as one-stage number classification [14, 25], pose-guided detectors with post-processing [29], or keyframe identification modules leveraging spatiotemporal networks to extract critical frames with visible numbers [2]. These methods boost accuracy using contextual features but need dataset-specific training, adding implementation complexity.

# 3. Methodology

Our game state reconstruction framework is a robust and modular pipeline designed to track players and reconstruct their positions on a 2D top-view (minimap) of the football pitch using single-camera video footage (Figure 1). The tracking pipeline consists of three key stages: **raw tracking**, **team detection**, and **post-processing**.

In the **raw tracking** stage, athletes are detected, pitch localization is performed to determine their coordinates on the field, initial tracking is established, and feature vectors (ReID, TeamID, and jersey number) are computed for further processing. Next, in the **team detection** stage, all athletes on the field are grouped into five clusters based on their roles. Finally, in the **post-processing** stage, the system refines and merges short tracklets into longer, continuous trajectories by leveraging all previously gathered information.

Overall pipeline diagram showing video input, raw tracking, team detection, and post-processing stages.

Figure 1. The overall pipeline is divided into three main stages. In the **raw tracking stage**, initial tracks are generated, and information about team embeddings and jersey numbers is estimated for each player. During the team detection stage, the previously collected information is used to assign player tracks to their respective teams. Finally, postprocessing is applied to reduce the number of resulting tracks by merging raw tracking results.

Detailed diagram of the Raw Tracking stage showing object detection, pitch localization, and various player attribute models.

Figure 2. Raw tracking stage performs object detection, pitch localization, collects information about players teams required on consequent stages, Re-ID embeddings, jersey numbers and then merges all collected data into preliminary object tracks using the DeepSort-based tracking

## 3.1. Raw Tracking

Raw tracking is the first stage of our pipeline, generating preliminary tracking results in real time as video frames are processed (Figure 2). Since this stage is the most computationally intensive, we implemented it using NVIDIA DeepStream SDK [35], optimizing all models as TensorRT engines [36] with FP16 precision. This approach allowed us to achieve real-time tracking at up to 80 FPS on consumer-grade hardware, such as an RTX 3080Ti laptop GPU.

Firstly, for each input frame, the system detects players by YOLO detector [53], fine-tuned for football-specific object detection, localizes athletes and the ball in the video frames. YOLOv5 architecture was chosen due to its one-stage detection strategy, relatively lightweight architecture which allows acceptable trade-off between quality and inference speed.

Then the proposed system proceed with **Pitch Localization**, which is a critical component of our pipeline, enabling the accurate mapping of detected objects from image-space to real-world pitch coordinates. This process is divided into two key stages: Initial Camera Parameter Prediction and Keypoints-Based Refinement (see Supplementary Material for a visualization of the entire process).

At **Initial Camera Parameters Prediction** a standard pinhole camera model was utilized, assuming zero distortions, and fixing the principal point to the center of the frame. The initial camera parameters for each frame are predicted using a custom CNN-Transformer encoder-decoder model. The encoder is based on the SegFormer architecture [54], while the decoder comprises multiple heads trained simultaneously. The model takes an H×W image from a football broadcast as input and predicts camera parameters.

The first head, the **“parameters head”** is trained to estimate seven camera parameters: position (x, y, z real-world coordinates), orientation (pan, roll, tilt), and field of view (FOV). This head incorporates a **Polarized Self-Attention (PSA)** layer [30] followed by multiple Conv2D layers and an average pooling layer. The second head, the **“heatmaps head”** is trained to predict heatmaps for X and Y coordinates [38]. It is used only during training to enhance the training process by adding more modalities and distilling task-specific knowledge. This head consists of a PSA layer followed by Conv2D and Pixel Shuffle layers [43] (further details about the model architecture can be found in Supplementary Material).

The proposed camera parameters model is trained with the following **loss function**:

$$ \mathcal{L} = w_1 \cdot L^2_{\text{world}} + w_2 \cdot L^2_{\text{camera}} + w_3 \cdot L^1_{\text{parameters}} + w_4 \cdot L^2_{\text{heatmap}} \quad (1) $$

where $w_1, w_2, w_3, w_4$ are the loss weights. $L^2_{\text{world}}$ is the re-projection error in world space:

$$ L^2_{\text{world}} = \sum_x \left\| P_{\text{pred}}^{\text{inv}}(P_{\text{gt}}(x_{\text{world}})) - x_{\text{world}} \right\|_2 \quad (2) $$

where $P(x_{\text{world}})$ is a function that maps a 3D point in world coordinates $x_{\text{world}}$ to 3D camera coordinates expressed in Normalized Device Coordinates (NDC), and $P^{\text{inv}}(x_{\text{camera}})$ is its inverse function that maps NDC camera coordinates $x_{\text{camera}}$ back to 3D world coordinates $x_{\text{world}}$. The subscripts ‘gt‘ and ‘pred‘ indicate whether the function uses ground-truth or predicted camera parameters, respectively.

$L^2_{\text{camera}}$ denotes the loss in NDC camera space, allowing resolution-invariant learning and smooth gradient propagation due to the absence of the perspective divide.

$$ L_{\text{camera}}^2 = \sum_{x} \| P_{\text{pred}}(x_{\text{world}}) - P_{\text{gt}}(x_{\text{world}}) \|_2 \quad (3) $$

$L_{\text{parameters}}^1$ represents the difference between predicted and real camera parameters (camera position, pitch, yaw, roll, field of view):

$$ L_{\text{parameters}}^1 = \sum_{y} \| (y_{\text{pred}} - y_{\text{gt}}) \|_1 \quad (4) $$

where $y$ is a specific camera parameter. Term of the loss function Eq. (1)

$$ L_{\text{heatmap}}^2 = \sum_{i,j} \| UV_{\text{pred}}(i, j) - UV_{\text{gt}}(i, j) \|_2 \quad (5) $$

represents the difference between the ground truth heatmap $UV_{\text{gt}}$ and the heatmap predicted by the network $UV_{\text{pred}}$, where $i, j$ are pixel coordinates on the heatmap (see details in Supplementary Material).

At **Keypoints-Based Refinement** stage the Field Keypoints Model, based on ResNet18 [16], detects 74 keypoints representing the intersections of pitch lines with grass lines. These keypoints refine the initial camera parameter predictions by minimizing the reprojection error when mapped to a canonical (two-dimensional aerial) field view.

This refinement process employs a brute-force optimization strategy, evaluating predefined combinations of camera parameter corrections and selecting the best one. A predefined set of delta values, such as $[-0.15, -0.10, \dots, 0.15]$, is used for each camera parameter. These deltas are added to the SegFormer [54] model's outputs, generating a delta matrix. Using these adjusted camera parameters, a series of homography matrices are computed to project the detected keypoints from the Field Keypoints Model onto planar coordinates (in the world space).

To ensure alignment, keypoints are converted into lines, where applicable, based on the expectation that certain keypoints lie along the same line. Outliers — keypoints deviating significantly from the expected line are discarded. Multiple sets of lines are then generated, and the L2 distance between these planar lines and the ideal pitch model is calculated. The optimal camera parameters are determined by selecting the set with the smallest L2 difference, ensuring the most accurate alignment and projection.

The final camera parameters derived from this process enable the calculation of real-world 3D coordinates for all detected athletes. These coordinates are used by the DeepSORT algorithm [52] for tracking player movements on the pitch, providing a bird's-eye view of the game.

To reduce frame-to-frame fluctuations and ensure temporal smoothness in camera parameter predictions, a Savitzky-Golay filter [13] is applied. This method efficiently smooths

predictions but is limited to correcting small errors, with a maximum adjustment range of $\pm 2$ degrees for angles and $\pm 2$ meters for positions. To enable its use, predictions are delayed by 15 frames (0.5 sec), allowing the filter to access both past and future values.

The **Jersey Number Recognition** model is based on a modified ResNet architecture designed for $32 \times 32$ input images. Instead of detecting individual digits using bounding boxes, the model employs two classification heads: the first determines whether a jersey number contains a leading digit (1–9 or none), while the second predicts the second digit (0–9). This structure eliminates the need for precise bounding box annotations, simplifying data labeling and improving robustness.

To handle class imbalance in jersey number occurrences, a hybrid loss function was used in training: BinaryFocalLoss [28] helps in determining the presence of a leading digit, while CrossEntropyLoss ensures accurate digit classification. This combination enhances performance on less frequent digit classes and improves overall robustness. The model processes each jersey crop independently and predicts two digits per image, allowing for recognition even when part of the number is occluded.

At **Feature Extraction** stage secondary networks infer additional attributes for each athlete: **ReID Embeddings** are extracted using a ResNet50-based [16] model trained on athlete crops, enabling DeepSORT [52] to maintain consistent player identities across frames. **TeamID Embeddings** are generated by an OSNet [60] model trained on 111 uniform classes, providing uniform-specific features for team detection in subsequent stages. Each class represents a unique uniform of the football players (for example red t-shirt and white shorts). OSNet[60] is designed for person re-identification (**ReID**), utilizing multi-scale residual blocks and a unified aggregation gate (**AG**) to dynamically fuse spatial features, ensuring both efficiency and strong discriminative power. **Player Orientation** is predicted using a ResNet18 [16] model trained to classify athletes' directions (left/up/right/down). **Player Anomaly** is predicted by a lightweight custom model identifies and discards crops containing multiple athletes or irrelevant content to improve tracking quality.

DeepSORT-based algorithm [52] is used for **preliminary tracking** and operates in real-world pitch coordinates derived from the camera parameters instead of pixel coordinates. To ensure accurate and consistent tracking, restrictions based on player orientation (e.g., preventing 180-degree direction changes in one frame) and TeamID Embeddings are applied.

Raw Tracking stage lays the foundation for subsequent steps, enabling efficient, real-time tracking while providing essential features and initial tracking results for further refinement.

Team Detection Process diagram showing frame clustering and goalkeeper detection.

Figure 3. Team Detection Process. (a) Frames are clustered into three main groups: the two largest clusters (left and right teams) and the referee cluster. (b) Goalkeeper detection is performed separately by identifying athletes inside the penalty area and clustering them based on embeddings.

## 3.2. Team Detection

In the previous step (raw-tracking), we gathered TeamID embeddings and pitch coordinates for every athlete across all frames. The next step is team detection, where we identify five embedding clusters: left/right team athletes, referees, and left/right goalkeepers. During post-processing, each athlete is assigned to a team based on the minimum cosine distance to the estimated clusters.

Estimating the Two Biggest Clusters (Left/Right Teams) and Referee Cluster (Figure 3a):

1. We query all athletes whose X-axis distance is less than 30 meters from the pitch center and cluster them into three groups: the two largest clusters representing the left and right teams, and a smaller cluster for the referees.

2. If the input video contains only the left or right penalty area (determined by the camera pan), or the clip is very short we instead select all athletes outside the penalty area. This strategy is not used by default, as goalkeepers occasionally leave their designated areas.

Estimating Embeddings Clusters for Left and Right Goalkeepers (Figure 3b):

1. We query athletes positioned inside the penalty area (on the left or right side).

2. Athletes near previously detected clusters are filtered out based on cosine distance to avoid overlap.

3. The remaining athletes are used to calculate an embedding cluster for the respective goalkeeper.

Defining Left vs. Right Teams. To distinguish the left team from the right team, we employ a voting mechanism across frames (or at a defined stride for computational efficiency):

1. For each processed frame, we calculate the mean X-coordinate of athletes belonging to one of the two largest clusters and collect votes indicating whether the cluster is positioned more to the left or right.

2. After collecting all votes, the team affiliation is determined based on the majority vote count.

This voting mechanism is more robust than simply calculating the mean X-coordinate across all frames, which can fail in very short clips or scenarios where only part of the football field is visible.

## 3.3. Post-Processing

The post-processing stage plays a critical role in refining and consolidating the raw tracking results generated during the preliminary tracking and team detection stages. By leveraging information such as player positions on the pitch, detected jersey numbers, re-identification (ReID) embeddings, and team labels, this stage addresses common tracking challenges, including tracklet fragmentation, identity swaps, and temporal inconsistencies. The primary objective of post-processing is to concatenate short tracklets into longer, coherent trajectories while ensuring consistency across all attributes.

The post-processing pipeline consists of four key stages: splitting tracklets by jersey numbers and team IDs, merging tracklets by jersey numbers, merging tracklets by ReID similarity, tracklet interpolation.

In the first stage, tracklets are split based on the assumption that each track should correspond to a single recognized jersey number and team label. This ensures that no tracklet contains conflicting or inconsistent attributes. After this step, all tracklets are guaranteed to contain only detections associated with a unique jersey number and team ID. This process effectively eliminates ambiguities arising from incorrect associations during earlier stages.

The second stage focuses on merging tracklets that share the same jersey number. This is performed under the following conditions:

* Temporal Non-Overlap: The tracklets must not overlap in time.

* Physical Feasibility: The distance between the end position of one tracklet and the start position of another must be physically traversable by a player within the time gap between the two tracklets.

* Consistent Team IDs: The team IDs of the tracklets being merged must match.

This stage significantly reduces fragmentation caused by temporary occlusions or misdetections.

For tracklets that remain unmerged after the jersey number-based merging stage, often due to missing or undetected jersey numbers caused by occlusions or unfavorable player orientations, we employ ReID feature similarity as an alternative criterion. The merging process adheres to the same foundational constraints outlined in Section 3.3, ensuring temporal non-overlap, physical feasibility of movement, and consistency in team IDs. To determine whether two tracklets belong to the same player, we leverage multiple ReID vectors along each tracklet. For example, con-

sider Tracklet 1 with $N$ ReID vectors and Tracklet 2 with $M$ ReID vectors. We apply two complementary methods for comparing their ReID features:

* *Mean Vector Comparison*: For each tracklet, we compute the mean ReID vector by averaging all embeddings along the tracklet. The cosine distance between the mean vectors of Tracklet 1 and Tracklet 2 is then calculated. If this distance falls below a predefined threshold, the tracklets are merged. This approach provides a computationally efficient way to assess overall similarity.
* *Cross-Multiplication Matrix (Pairwise Comparison)*: To achieve more robust matching, we construct an $N \times M$ matrix where each entry represents the cosine similarity between a pair of ReID vectors—one from Tracklet 1 and one from Tracklet 2. By identifying the maximum value in this matrix (or equivalently, the minimum cosine distance), we ensure that even noisy or inconsistent embeddings do not hinder accurate matching. This method effectively captures the most similar pair of embeddings across the two tracklets.

By combining these two strategies, our pipeline achieves higher accuracy in merging tracklets, particularly in challenging scenarios where individual ReID embeddings may be unreliable. This dual-strategy approach leverages both global (mean vector) and local (pairwise comparison) information, ensuring precise identity association across fragmented tracklets.

Finally, gaps in player trajectories are addressed using a classic linear interpolation algorithm. This step ensures smooth and continuous trajectories by filling in missing positions for brief periods when players temporarily leave the camera's field of view or are occluded.

The effectiveness of the post-processing pipeline depends heavily on the quality of the input data, including video resolution, weather conditions, and scene complexity. However, in general, the pipeline achieves a 90% reduction in the number of tracklets and significantly minimizes tracklet swaps in crowded scenes. These improvements are particularly beneficial in challenging scenarios, such as dynamic content where players frequently enter and exit the camera's field of view.

By systematically addressing fragmentation, identity swaps, and temporal inconsistencies, the post-processing stage ensures high-quality, temporally consistent player trajectories, which are essential for accurate game state reconstruction.

# 4. Datasets

All datasets used for training our models are summarized in Tab. 1. These datasets cover a wide range of tasks, including object detection, re-identification, team classification, and pitch localization. While most datasets are briefly outlined in the table, the camera parameters dataset requires a more  detailed explanation, which will be provided in a section below.

## 4.1. Camera parameters dataset

Our dataset mainly consists of two parts: real-world images (sourced from the Field Keypoints Dataset) and synthetic images (generated using a modified version of the Google Research Football Simulator [24]). The combination of these datasets ensures a balance between real-world variability and controlled synthetic diversity, enhancing the robustness of the training process. Visualized distributions of camera parameters and example images can be found in Supplementary Material.

**Real-World Dataset.** To construct our dataset, we modified the TVCalib framework [49] to work with labeled keypoints instead of segmentation masks. The dataset consists of images annotated with keypoints corresponding to the pitch lines in the image space. Originally TVCalib calculates the loss in the image space. In our implementation, the optimization process operates in the real-world space (canonical view). Specifically, labeled keypoints from the image space are projected onto the canonical view, and the loss is defined as the sum of L2 distances between the projected keypoints and the corresponding pitch lines in the real-world space. This adjustment to the framework proved to be more robust, leading to significantly more accurate camera parameter estimations compared to the original implementation. Using this process, we collected a dataset of 22,000 images with corresponding keypoint annotations.

**Synthetic Dataset.** To generate a synthetic dataset, we customized the Google Research Football Simulator [24] to introduce randomized textures and camera views. Specifically, for every other launch of the simulator, textures for the grass, pitch lines, and athlete uniforms were selected randomly from a predefined set. This ensured a diverse appearance across the generated frames. During each simulation, the game was played at 10x real-world speed, and camera parameters (e.g., position, orientation, and zoom) varied randomly as the game progressed. This approach allowed us to capture a wide range of perspectives on the football pitch efficiently. To properly sample new camera positions while ensuring the camera still views the field, we performed sampling in multiple stages: the x-coordinate was uniformly sampled (in meters) from $-60\text{m}$ to $60\text{m}$, the y-coordinate from $40\text{m}$ to $110\text{m}$, and the z-coordinate from $-40\text{m}$ to $-10\text{m}$; tilt and pan angles were adjusted to ensure the camera's center ray hit the football field, while the roll angle was fixed at 0 for all images, resulting in a dataset of 40,000 images annotated with camera parameters including position (x, y, z), orientation (pan, roll, tilt), and field of view (FoV).

<table>
  <thead>
    <tr>
        <th>Task</th>
        <th>Type</th>
        <th># Imgs</th>
        <th># Classes/kpts</th>
        <th>Comment</th>
    </tr>
  </thead>
  <tbody>
    <tr>
        <td>Detect athletes and ball</td>
        <td>Object detection</td>
        <td>66k</td>
        <td>2 (athletes, ball)</td>
        <td>YOLO-style dataset with bounding boxes</td>
    </tr>
    <tr>
        <td>Field Keypoints</td>
        <td>Keypoint detection</td>
        <td>36k</td>
        <td>74</td>
        <td>Intersections of pitch lines with grass lines</td>
    </tr>
    <tr>
        <td>Camera Parameters</td>
        <td>Regression</td>
        <td>62k</td>
        <td>7 (x, y, z, pan, roll, tilt, fov)</td>
        <td>Includes synthetic and real-world data</td>
    </tr>
    <tr>
        <td>TeamID</td>
        <td>Classification</td>
        <td>550k</td>
        <td>111</td>
        <td>Each class represents a unique team</td>
    </tr>
    <tr>
        <td>ReID</td>
        <td>Re-Identification</td>
        <td>280k</td>
        <td>370</td>
        <td>Contains crops of 370 unique players</td>
    </tr>
    <tr>
        <td>Player Orientation</td>
        <td>Classification</td>
        <td>20k</td>
        <td>4 (left, up, right, down)</td>
        <td>Athlete facing direction classification</td>
    </tr>
    <tr>
        <td>Anomaly</td>
        <td>Classification</td>
        <td>16k</td>
        <td>2 (normal, anomaly)</td>
        <td>Detects irrelevant athlete crops</td>
    </tr>
    <tr>
        <td>Jersey Number Recognition</td>
        <td>Classification</td>
        <td>70k</td>
        <td>100</td>
        <td>Upper half of athlete bbox with visible jersey number</td>
    </tr>
  </tbody>
</table>


Table 1. Datasets overview.

# 5. Experiment setting

The training configurations for all models used in our pipeline are summarized in Tab. 2, providing a comprehensive overview of the hyperparameters, loss functions, input sizes, and optimization details for each task. All models were trained using the Constructor Research Platform¹ with NVIDIA A100 GPU (40 GB). For camera parameter estimation we provide more in-depth description of the training setup. These additional details highlight the specific strategies and methodologies employed to achieve optimal performance for this task.

## 5.1. Camera parameters regression

**Training setup.** The camera parameter regression model was trained in two distinct phases to ensure robust generalization across both synthetic and real-world data:

1. The model was trained for 200k batches using a combination of real-world and synthetic datasets. This phase leveraged the diversity of synthetic data while grounding the model in real-world variability.

2. An additional 200k batches were trained exclusively on real-world images to fine-tune the model for enhanced accuracy in practical scenarios.

Training was conducted with a batch size of 8, utilizing the AdamW [32] optimizer. The learning rate was set to a maximum of $5 \times 10^{-4}$ and was dynamically adjusted using a cosine annealing scheduler with a warmup period of 20k steps to stabilize early training. The backbone weights were initialized using Kaiming initialization [15] to facilitate stable convergence.

**Hyperparameter Optimization.** To further enhance model performance, we employed the Optuna hyperparameter optimization framework [1]. Optuna allows efficient exploration of the hyperparameter space using a structured Parzen estimator (TPE) sampler. We optimized the weights $w_1, w_2, w_3, w_4$ in the loss function Eq. (1) and the number of grid steps used for keypoint generation. This process involved randomly sampling weight values and grid configurations, followed by short training runs to identify the most effective combination. The final selected values were: $w_1 = 0.048$ ($L^2_{\text{world}}$), $w_2 = 2.49$ ($L^2_{\text{camera}}$), $w_3 = 1.0$ ($L^2_{\text{params}}$, was fixed), $w_4 = 10.0$ ($L^2_{\text{heatmap}}$, introduced later and not optimized,), and `grid_steps` = 36.

**Augmentations.** To enhance robustness, we applied two types of augmentations: **pixel-space** and **geometrical**, implemented using the Kornia library [39] for GPU-accelerated transformations.

* **Pixel-Space Augmentations:** Modified image appearance without altering camera parameters, including cutouts, horizontal flipping, Gaussian noise/blur, sharpness adjustments, random color variations (contrast, brightness, hue, saturation), CLAHE, and shadow/brightness/contrast effects (e.g., RandomPlasmaShadow, RandomPlasmaBrightness). Each was applied with a probability of 0.5.

* **Geometrical Augmentations:** Affected both images and camera parameters, such as random rotations (adjusting the "roll" parameter) and horizontal flips (inverting the "pan" parameter).

These augmentations improved generalization across diverse broadcast conditions and camera setups.

# 6. Evaluation (SoccerNet GSR)

We evaluated our method in the SoccerNet Game State Reconstruction (GSR) Challenge 2024 [46], which focuses on tracking and identifying players from a single-camera setup to generate a minimap representation of the game. The dataset consists of 200 video sequences (30 seconds each) with 2.36 million annotated athlete bounding boxes and 9.37 million pitch keypoints for localization. Ground truth labels for a subset of the data were kept private for evaluation on Eval.ai².

<table>
  <thead>
    <tr>
        <th>Task</th>
        <th>Type</th>
        <th>Backbone</th>
        <th>Loss</th>
        <th>Input Size</th>
        <th>mPara</th>
        <th>Optimizer</th>
        <th>Comment</th>
    </tr>
  </thead>
  <tbody>
    <tr>
        <td>Detect athletes and ball</td>
        <td>Object detection</td>
        <td>YOLOv5</td>
        <td>YOLOv5 loss (CIoU + BCE)</td>
        <td>1920 × 1080</td>
        <td>35.5M</td>
        <td>Adam</td>
        <td>100 epochs, LambdaLR scheduler, initial learning rate: 0.01, momentum: 0.95, decay: 5 × 10⁻⁴</td>
    </tr>
    <tr>
        <td>Field Keypoints</td>
        <td>Keypoints detection</td>
        <td>ResNet-18</td>
        <td>AdaptiveWing Loss</td>
        <td>480 × 270</td>
        <td>11M</td>
        <td>AdamW</td>
        <td>150 epochs, OneCycleLR, learning rate: 0.01, learning rate factor: 0.1</td>
    </tr>
    <tr>
        <td>Camera Parameters</td>
        <td>Regression</td>
        <td>Custom SegFormer</td>
        <td>Custom Loss</td>
        <td>512 × 288</td>
        <td>5M</td>
        <td>AdamW</td>
        <td>400k batches with batch size 8, cosine annealing scheduler, learning rate: 5 × 10⁻⁴</td>
    </tr>
    <tr>
        <td>TeamID</td>
        <td>Classification</td>
        <td>OSNet</td>
        <td>TripletLoss</td>
        <td>64 × 32</td>
        <td>0.3M</td>
        <td>Adam</td>
        <td>40 epochs, learning rate: 0.001, momentum: 0.9, weight decay: 5 × 10⁻⁴, single step learning rate scheduler with γ = 0.1 and step size 1</td>
    </tr>
    <tr>
        <td>ReID</td>
        <td>Re-Identification</td>
        <td>ResNet-50</td>
        <td>TripletLoss</td>
        <td>256 × 128</td>
        <td>25.6M</td>
        <td>SGD</td>
        <td>Learning rate: 1 × 10⁻³, triplet sampler: each batch contains 28 classes and 7 samples from each class</td>
    </tr>
    <tr>
        <td>Player Orientation</td>
        <td>Classification</td>
        <td>ResNet-18</td>
        <td>CrossEntropy</td>
        <td>62 × 32</td>
        <td>11M</td>
        <td>SGD</td>
        <td>Learning rate: 1 × 10⁻⁴</td>
    </tr>
    <tr>
        <td>Anomaly</td>
        <td>Classification</td>
        <td>ResNet-18</td>
        <td>CrossEntropy</td>
        <td>62 × 32</td>
        <td>11M</td>
        <td>Adam</td>
        <td>Learning rate: 1 × 10⁻³</td>
    </tr>
    <tr>
        <td>Jersey Number Recognition</td>
        <td>Classification</td>
        <td>ResNet-18</td>
        <td>BinaryFocal + CrossEntropy</td>
        <td>32 × 32</td>
        <td>17M</td>
        <td>AdamW</td>
        <td>Learning rate: 1 × 10⁻⁴ with Reduce LR on Plateau</td>
    </tr>
  </tbody>
</table>


Table 2. Training configurations.

# 6.1. GS-HOTA Metric

Performance was evaluated using GS-HOTA, a metric from [46] tailored for game state reconstruction. GS-HOTA extends HOTA by factoring in role, team, and jersey number, ensuring practical applicability. It penalizes errors like incorrect attributes or missing labels, enforcing strict matching for high-quality results in football analytics. The similarity function includes:

* Localization Similarity (LocSim) – Measures *Euclidean distance* in pitch coordinates, smoothed by a Gaussian kernel with a 5-meter tolerance.

* Identification Similarity (IdSim) – Requires exact matches for role, team, and jersey number; mismatches are treated as false positives, enforcing strict tracking accuracy.

# 6.2. Results


<table>
  <thead>
    <tr>
        <th>Participant</th>
        <th>GS-HOTA (↑)</th>
        <th>GS-DetA (↑)</th>
        <th>GS-AssA (↑)</th>
    </tr>
  </thead>
  <tbody>
    <tr>
        <td><strong>Constructor Tech (Ours)</strong></td>
        <td><strong>63.81</strong></td>
        <td><strong>49.52</strong></td>
        <td><strong>82.23</strong></td>
    </tr>
    <tr>
        <td>UPCxMobius</td>
        <td>43.15</td>
        <td>30.46</td>
        <td>61.16</td>
    </tr>
    <tr>
        <td>JAM</td>
        <td>34.40</td>
        <td>19.38</td>
        <td>61.08</td>
    </tr>
    <tr>
        <td><em>Baseline [46]</em></td>
        <td>23.36</td>
        <td>9.80</td>
        <td>55.69</td>
    </tr>
  </tbody>
</table>


Table 3. Game state reconstruction leaderboard. The main metric (**GS-HOTA**) and best performances are in **bold**.

Our method secured **first place** in the SoccerNet GSR Challenge 2024, achieving a GS-HOTA score of **63.81**, significantly outperforming the second-place team (43.15) and third-place team (34.40). The comparison of our results with the top results of SoccerNet GSR Challenge 2024 and the baseline is given in Sec. 6.2.

The post-processing stage played a crucial role in improving association accuracy (GS-AssA) by reducing tracklet fragmentation, while our pitch localization pipeline enhanced detection accuracy (GS-DetA) by ensuring precise player positioning on the pitch. These improvements led to our significant performance margin over competing methods.

# 7. Conclusion and Future Work

We evaluated our approach on the largest football broadcast Game State Reconstruction dataset available, securing the top position in the SoccerNet Game State Reconstruction Challenge 2024. This achievement highlights the effectiveness of our method in real-world scenarios. For future improvements, we aim to unify the camera models and field keypoints model by adding an additional head to the custom SegFormer. Additionally, we plan to replace YOLOv5, remove the anomaly model due to its negligible impact, and enhance player orientation modeling by training on a uniform angle distribution (0 to 360 degrees) instead of four discrete bins. Furthermore, since our model predicts jersey numbers as separate first and second digits, we can relax the tracklet merging criteria to require only one matching digit (depending on the athlete’s orientation). When combined with the improved player orientation model, this adjustment could significantly enhance the association ability of our tracking pipeline, leading to more robust and accurate game state reconstruction.

# 8. Acknowledgments

The authors would like to thank Igor Rekun for his valuable contributions to the development of the Camera Parameter model.

# References

[1] Takuya Akiba, Shotaro Sano, Toshihiko Yanase, Takeru Ohta, and Masanori Koyama. Optuna: A next-generation

hyperparameter optimization framework. In *Proceedings of the 25th ACM SIGKDD international conference on knowledge discovery & data mining*, pages 2623–2631, 2019. 7

[2] Bavesh Balaji, Jerrin Bright, Harish Prakash, Yuhao Chen, David A Clausi, and John Zelek. Jersey number recognition using keyframe identification from low-resolution broadcast videos. In *Proceedings of the 6th International Workshop on Multimedia Content Analysis in Sports*, pages 123–130, 2023. 2

[3] Philipp Bergmann, Tim Meinhardt, and Laura Leal-Taixe. Tracking without bells and whistles. In *Proceedings of the IEEE/CVF international conference on computer vision*, pages 941–951, 2019. 2

[4] Alex Bewley, Zongyuan Ge, Lionel Ott, Fabio Ramos, and Ben Upcroft. Simple online and realtime tracking. In *2016 IEEE international conference on image processing (ICIP)*, pages 3464–3468. Ieee, 2016.

[5] Erik Bochinski, Tobias Senst, and Thomas Sikora. Extending iou based multi-object tracking by visual information. In *2018 15th IEEE International Conference on Advanced Video and Signal Based Surveillance (AVSS)*, pages 1–6. IEEE, 2018. 2

[6] Jianhui Chen and James J Little. Sports camera calibration via synthetic data. In *Proceedings of the IEEE/CVF conference on computer vision and pattern recognition workshops*, pages 0–0, 2019. 2

[7] Yen-Jui Chu, Jheng-Wei Su, Kai-Wen Hsiao, Chi-Yu Lien, Shu-Ho Fan, Min-Chun Hu, Ruen-Rone Lee, Chih-Yuan Yao, and Hung-Kuo Chu. Sports field registration via keypoints-aware label condition. In *Proceedings of the IEEE/CVF Conference on Computer Vision and Pattern Recognition*, pages 3523–3530, 2022. 2

[8] Anthony Cioppa, Adrien Deliege, Silvio Giancola, Bernard Ghanem, Marc Van Droogenbroeck, Rikke Gade, and Thomas B Moeslund. A context-aware loss function for action spotting in soccer videos. In *Proceedings of the IEEE/CVF Conference on Computer Vision and Pattern Recognition*, pages 13126–13136, 2020. 2

[9] Anthony Cioppa, Adrien Deliège, Silvio Giancola, Bernard Ghanem, and Marc Van Droogenbroeck. Scaling up soccernet with multi-view spatial localization and re-identification. *Scientific data*, 9(1):355, 2022. 2

[10] Patrick Dendorfer, Aljosa Osep, Anton Milan, Konrad Schindler, Daniel Cremers, Ian Reid, Stefan Roth, and Laura Leal-Taixé. Motchallenge: A benchmark for single-camera multiple target tracking. *International Journal of Computer Vision*, 129:845–881, 2021. 2

[11] Martin Ester, Hans-Peter Kriegel, Jörg Sander, and Xiaowei Xu. A density-based algorithm for discovering clusters in large spatial databases with noise. In *Proceedings of the Second International Conference on Knowledge Discovery and Data Mining (KDD)*, pages 226–231. AAAI Press, 1996. 2

[12] Martin A Fischler and Robert C Bolles. Random sample consensus: a paradigm for model fitting with applications to image analysis and automated cartography. *Communications of the ACM*, 24(6):381–395, 1981. 2

[13] Neal B Gallagher. Savitzky-golay smoothing and differentiation filter. *Eigenvector Research Incorporated*, 2020. 2, 4

[14] Sebastian Gerke, Karsten Muller, and Ralf Schafer. Soccer jersey number recognition using convolutional neural networks. In *Proceedings of the IEEE International Conference on Computer Vision Workshops*, pages 17–24, 2015. 2

[15] Kaiming He, Xiangyu Zhang, Shaoqing Ren, and Jian Sun. Delving deep into rectifiers: Surpassing human-level performance on imagenet classification. In *Proceedings of the IEEE International Conference on Computer Vision (ICCV)*, pages 1026–1034, 2015. 7

[16] Kaiming He, Xiangyu Zhang, Shaoqing Ren, and Jian Sun. Deep residual learning for image recognition. In *Proceedings of the IEEE conference on computer vision and pattern recognition*, pages 770–778, 2016. 4

[17] Jan Held, Hani Itani, Anthony Cioppa, Silvio Giancola, Bernard Ghanem, and Marc Van Droogenbroeck. X-vars: Introducing explainability in football refereeing with multi-modal large language models. In *Proceedings of the IEEE/CVF Conference on Computer Vision and Pattern Recognition*, pages 3267–3279, 2024. 2

[18] Namdar Homayounfar, Sanja Fidler, and Raquel Urtasun. Sports field localization via deep structured models. In *Proceedings of the IEEE Conference on Computer Vision and Pattern Recognition*, pages 5212–5220, 2017. 2

[19] Muhammad Hussain. Yolo-v1 to yolo-v8, the rise of yolo and its complementary nature toward digital manufacturing and industrial defect detection. *Machines*, 11(7):677, 2023. 2

[20] Maxime Istasse, Julien Moreau, and Christophe De Vleeschouwer. Associative embedding for team discrimination. In *Proceedings of the IEEE/CVF Conference on Computer Vision and Pattern Recognition Workshops*, pages 0–0, 2019. 2

[21] Nicolas Jacquelin, Romain Vuillemot, and Stefan Duffner. Efficient one-shot sports field image registration with arbitrary keypoint segmentation. In *2022 IEEE International Conference on Image Processing (ICIP)*, pages 1771–1775. IEEE, 2022. 2

[22] Masoud Khodarahmi and Vafa Maihami. A review on kalman filter models. *Archives of Computational Methods in Engineering*, 30(1):727–747, 2023. 2

[23] Maria Koshkina and James H Elder. A general framework for jersey number recognition in sports video. In *Proceedings of the IEEE/CVF Conference on Computer Vision and Pattern Recognition*, pages 3235–3244, 2024. 2

[24] Karol Kurach, Anton Raichuk, Piotr Stańczyk, Michał Zajac, Olivier Bachem, Lasse Espeholt, Carlos Riquelme, Damien Vincent, Marcin Michalski, Olivier Bousquet, et al. Google research football: A novel reinforcement learning environment. In *Proceedings of the AAAI conference on artificial intelligence*, pages 4501–4510, 2020. 6

[25] Gen Li, Shikun Xu, Xiang Liu, Lei Li, and Changhu Wang. Jersey number recognition with semi-supervised partial transformer network. In *Proceedings of the IEEE conference on computer vision and pattern recognition workshops*, pages 1783–1790, 2018. 2

[26] Hui Li, Peng Wang, Chunhua Shen, and Guyu Zhang. Show, attend and read: A simple and strong baseline for irregular text recognition. In *Proceedings of the AAAI conference on artificial intelligence*, pages 8610–8617, 2019. 2

[27] Minghui Liao, Zhaoyi Wan, Cong Yao, Kai Chen, and Xiang Bai. Real-time scene text detection with differentiable binarization. In *Proceedings of the AAAI conference on artificial intelligence*, pages 11474–11481, 2020. 2

[28] Tsung-Yi Lin, Priya Goyal, Ross Girshick, Kaiming He, and Piotr Dollár. Focal loss for dense object detection. In *Proceedings of the IEEE international conference on computer vision*, pages 2980–2988, 2017. 4

[29] Hengyue Liu and Bir Bhanu. Pose-guided r-cnn for jersey number recognition in sports. In *Proceedings of the IEEE/CVF Conference on Computer Vision and Pattern Recognition Workshops*, pages 0–0, 2019. 2

[30] Huajun Liu, Fuqiang Liu, Xinyi Fan, and Dong Huang. Polarized self-attention: Towards high-quality pixel-wise regression. *arXiv preprint arXiv:2107.00782*, 2021. 3

[31] Stuart Lloyd. Least squares quantization in pcm. *IEEE transactions on information theory*, 28(2):129–137, 1982. 2

[32] Ilya Loshchilov and Frank Hutter. Decoupled weight decay regularization. *arXiv preprint arXiv:1711.05101*, 2017. 7

[33] Amir M Mansourian, Vladimir Somers, Christophe De Vleeschouwer, and Shohreh Kasaei. Multi-task learning for joint re-identification, team affiliation, and role classification for sports visual tracking. In *Proceedings of the 6th International Workshop on Multimedia Content Analysis in Sports*, pages 103–112, 2023. 2

[34] Xiaohan Nie, Shixing Chen, and Raffay Hamid. A robust and efficient framework for sports-field registration. In *Proceedings of the IEEE/CVF Winter Conference on Applications of Computer Vision*, pages 1936–1944, 2021. 2

[35] NVIDIA Corporation. *NVIDIA DeepStream SDK Developer Guide*, 2025. https://docs.nvidia.com/metropolis/deepstream/dev-guide/. 3

[36] NVIDIA Corporation. *NVIDIA TensorRT: High Performance Deep Learning Inference Platform*, 2025. https://developer.nvidia.com/tensorrt. 3

[37] Luca Pappalardo, Paolo Cintia, Alessio Rossi, Emanuele Massucco, Paolo Ferragina, Dino Pedreschi, and Fosca Giannotti. A public data set of spatio-temporal match events in soccer competitions. *Scientific data*, 6(1):236, 2019. 2

[38] Roi Poranne, Marco Tarini, Sandro Huber, Daniele Panozzo, and Olga Sorkine-Hornung. Autocuts: simultaneous distortion and cut optimization for uv mapping. *ACM Transactions on Graphics (TOG)*, 36(6):1–11, 2017. 3

[39] Edgar Riba, Dmytro Mishkin, Daniel Ponsa, Ethan Rublee, and Gary Bradski. Kornia: an open source differentiable computer vision library for pytorch. In *Proceedings of the IEEE/CVF Winter Conference on Applications of Computer Vision*, pages 3674–3683, 2020. 7

[40] Miguel Santos Marques, Ricardo Gomes Faria, and José Henrique Brito. Hierarchical line extremity segmentation u-net for the soccernet 2022 calibration challenge-pitch localization. In *Iberian Conference on Pattern Recognition and Image Analysis*, pages 442–453. Springer, 2023. 2

[41] Rahul Anand Sharma, Bharath Bhat, Vineet Gandhi, and CV Jawahar. Automated top view registration of broadcast football videos. In *2018 IEEE Winter Conference on Applications of Computer Vision (WACV)*, pages 305–313. IEEE, 2018. 2

[42] Feng Shi, Paul Marchwica, Juan Camilo Gamboa Higuera, Michael Jamieson, Mehrsan Javan, and Parthipan Siva. Self-supervised shape alignment for sports field registration. In *Proceedings of the IEEE/CVF Winter Conference on Applications of Computer Vision*, pages 287–296, 2022. 2

[43] Wenzhe Shi, Jose Caballero, Ferenc Huszár, Johannes Totz, Andrew P Aitken, Rob Bishop, Daniel Rueckert, and Zehan Wang. Real-time single image and video super-resolution using an efficient sub-pixel convolutional neural network. In *Proceedings of the IEEE conference on computer vision and pattern recognition*, pages 1874–1883, 2016. 3

[44] Gal Shitrit, Ishay Be’ery, and Ido Yerhushalmy. Soccernet 2023 tracking challenge–3rd place mot4mot team technical report. *arXiv preprint arXiv:2308.16651*, 2023. 2

[45] Vladimir Somers, Christophe De Vleeschouwer, and Alexandre Alahi. Body part-based representation learning for occluded person re-identification. In *Proceedings of the IEEE/CVF winter conference on applications of computer vision*, pages 1613–1623, 2023. 2

[46] Vladimir Somers, Victor Joos, Anthony Cioppa, Silvio Giancola, Seyed Abolfazl Ghasemzadeh, Floriane Magera, Baptiste Standaert, Amir M Mansourian, Xin Zhou, Shohreh Kasaei, et al. Soccernet game state reconstruction: End-to-end athlete tracking and identification on a minimap. In *Proceedings of the IEEE/CVF Conference on Computer Vision and Pattern Recognition*, pages 3293–3305, 2024. 1, 2, 7, 8

[47] Peize Sun, Jinkun Cao, Yi Jiang, Zehuan Yuan, Song Bai, Kris Kitani, and Ping Luo. Dancetrack: Multi-object tracking in uniform appearance and diverse motion. In *Proceedings of the IEEE/CVF conference on computer vision and pattern recognition*, pages 20993–21002, 2022. 2

[48] Shuhei Tarashima. Sflnet: direct sports field localization via cnn-based regression. In *Pattern Recognition: 5th Asian Conference, ACPR 2019, Auckland, New Zealand, November 26–29, 2019, Revised Selected Papers, Part I 5*, pages 677–690. Springer, 2020. 2

[49] Jonas Theiner and Ralph Ewerth. Tvcalib: Camera calibration for sports field registration in soccer. In *Proceedings of the IEEE/CVF Winter Conference on Applications of Computer Vision*, pages 1166–1175, 2023. 2, 6

[50] Graham Thomas, Rikke Gade, Thomas B Moeslund, Peter Carr, and Adrian Hilton. Computer vision for sports: Current applications and research topics. *Computer Vision and Image Understanding*, 159:3–18, 2017. 2

[51] Kanav Vats, Mehrnaz Fani, David A Clausi, and John Zelek. Multi-task learning for jersey number recognition in ice hockey. In *Proceedings of the 4th International Workshop on Multimedia Content Analysis in Sports*, pages 11–15, 2021. 2

[52] Nicolai Wojke, Alex Bewley, and Dietrich Paulus. Simple online and realtime tracking with a deep association metric. In *2017 IEEE international conference on image processing (ICIP)*, pages 3645–3649. IEEE, 2017. 2, 4

[53] Wentong Wu, Han Liu, Lingling Li, Yilin Long, Xiaodong Wang, Zhuohua Wang, Jinglun Li, and Yi Chang. Application of local fully convolutional neural network combined with yolo v5 algorithm in small target detection of remote sensing image. *PloS one*, 16(10):e0259283, 2021. 3

[54] Enze Xie, Wenhai Wang, Zhiding Yu, Anima Anandkumar, Jose M Alvarez, and Ping Luo. Segformer: Simple and efficient design for semantic segmentation with transformers. *Advances in neural information processing systems*, 34: 12077–12090, 2021. 3, 4

[55] Qixiang Ye, Qingming Huang, Shuqiang Jiang, Yang Liu, and Wen Gao. Jersey number detection in sports video for athlete identification. In *Visual Communications and Image Processing 2005*, pages 1599–1606. SPIE, 2005. 2

[56] Junqing Yu, Aiping Lei, Zikai Song, Tingting Wang, Hengyou Cai, and Na Feng. Comprehensive dataset of broadcast soccer videos. In *2018 IEEE Conference on Multimedia Information Processing and Retrieval (MIPR)*, pages 418–423. IEEE, 2018. 2

[57] Fangao Zeng, Bin Dong, Yuang Zhang, Tiancai Wang, Xiangyu Zhang, and Yichen Wei. Motr: End-to-end multiple-object tracking with transformer. In *European conference on computer vision*, pages 659–675. Springer, 2022. 2

[58] Neng Zhang and Ebroul Izquierdo. A high accuracy camera calibration method for sport videos. In *2021 International Conference on Visual Communications and Image Processing (VCIP)*, pages 1–5. IEEE, 2021. 2

[59] Yifu Zhang, Peize Sun, Yi Jiang, Dongdong Yu, Fucheng Weng, Zehuan Yuan, Ping Luo, Wenyu Liu, and Xinggang Wang. Bytetrack: Multi-object tracking by associating every detection box. In *European conference on computer vision*, pages 1–21. Springer, 2022. 2

[60] Kaiyang Zhou, Yongxin Yang, Andrea Cavallaro, and Tao Xiang. Omni-scale feature learning for person re-identification. In *Proceedings of the IEEE/CVF international conference on computer vision*, pages 3702–3712, 2019. 4

[61] Xingyi Zhou, Dequan Wang, and Philipp Krähenbühl. Objects as points. *arXiv preprint arXiv:1904.07850*, 2019. 2

# From Broadcast to Minimap: Achieving State-of-the-Art SoccerNet Game State Reconstruction

## Supplementary Material

## A. Camera Parameters Network Architecture

See Figure 4 for a detailed diagram of the camera parameters prediction model architecture.

## B. Pitch Localization

Accurate pitch localization is essential for mapping athletes on the image to their 3D positions on the pitch. Our method employs a multi-stage approach that first leverages a custom SegFormer model to generate an initial estimate of the camera parameters. Following this, a ResNet50-based segmentation network detects keypoints on the field, such as intersections of pitch lines with grass lines. These keypoints are then used in an optimization process to refine the estimated parameters, ensuring a more precise camera alignment with the real-world pitch.

You can find visualization of the pitch localization pipeline in Figure 5.

## C. Camera Parameters Dataset

The histograms in Figure 6 compare key distributions, such as camera position coordinates (X, Y, Z), field of view, pan, and tilt angles. The real dataset is naturally constrained by physical camera placements, while the synthetic dataset spans a wider range of configurations, enabling the model to learn robust representations. You can see examples of synthetic images in Figure 7.

## D. Camera Parameters Loss

Given the camera parameters params = $\{I, R, t\}$, we define a mapping function $P = F(\text{params})$ that transforms 3D world coordinates $X$ into 3D camera coordinates in Normalized Device Coordinates (NDC) space:

$$x_{\text{camera}} = P(X) = IR(X - t),$$

where:

* $X$ is the 3D world coordinates $[X, Y, Z]^T$.

* $I$ is the intrinsic matrix, encoding focal length and principal point (which is set to zero in the case of NDC coordinates).

* $R$ is the rotation matrix representing the camera's orientation.

* $t$ is the translation vector representing the camera's position.

* $x_{\text{camera}}$ is the resulting 3D point $[x_c, y_c, z_c]^T$ in NDC space.

This 3D-to-3D transformation (from world coordinates to Normalized Device Coordinates, or NDC) offers three key advantages: (1) it ensures resolution invariance by decoupling the loss from the input image size, (2) it eliminates the need for a perspective divide, thereby maintaining a smooth and stable gradient flow during optimization, and (3) it retains invertibility, enabling consistent reconstruction of 3D world coordinates from camera coordinates.

The inverse mapping is derived as follows. Starting from the forward transformation:

$$x_{\text{camera}} = IR(X - t),$$

we derive the inverse as follows:

* Multiply both sides by $(IR)^{-1}$ to isolate $X - t$:

$$(IR)^{-1}x_{\text{camera}} = X - t.$$

* Solve for $X$ by adding $t$ to both sides:

$$X = (IR)^{-1}x_{\text{camera}} + t.$$

* Since $R$ is an orthogonal matrix, $R^{-1} = R^T$, giving the final expression:

$$X = R^T I^{-1}x_{\text{camera}} + t.$$

Thus, our inverse mapping is:

$$P^{\text{inv}}(x_{\text{camera}}) = R^T I^{-1}x_{\text{camera}} + t.$$

This formulation plays a critical role in our training loss, allowing symmetric penalization of both forward and inverse transformations between world and NDC camera coordinates.

## E. Camera parameters data preparation

Figure 8 illustrates the projection of keypoints and coordinate heatmaps into image space using a homography matrix computed from the camera parameters.

```mermaid
graph TD
    subgraph backbone_encoder [backbone (encoder)]
        input_images[input images] --> EB1[Encoder Block]
        EB1 -- "H/4 x W/4 x C₁" --> EB2[Encoder Block]
        EB2 -- "H/8 x W/8 x C₂" --> EB3[Encoder Block]
        EB3 -- "H/16 x W/16 x C₃" --> EB4[Encoder Block]
        EB4 -- "H/32 x W/32 x C₄" --> LF[LinearFuse]
    end

    subgraph decoders [decoders]
        LF -- "H/4 x W/4 x C" --> CP_head[Camera parameters head]
        LF -- "H/4 x W/4 x C" --> UV_head[UV heatmaps head]
    end

    subgraph transformer_details [Transformer blocks details]
        ESA[Efficient Self-Attention] --> MLP_MixFFN[MLP Segformer MixFFN]
        OPE[Overlap Patch Embedding] --> TB[Transformer blocks] --> LN[nn.LayerNorm]
    end

    subgraph linear_fuse_details [LinearFuse details]
        MLP_LF[MLP] --> BU[bilinear upsample] --> MF[MLP Fuse]
    end

    subgraph cp_head_details [Camera parameters head details]
        SPSA1[Sequential Polarized Self-Attention] --> CM1[Conv Module]
    end

    subgraph uv_head_details [UV heatmaps head details]
        SPSA2[Sequential Polarized Self-Attention] --> CM2[Conv Module] --> PS[Pixel Shuffle]
    end

    CP_head -- "1 x 1 x 7" --> SPSA1
    UV_head -- "H x W x 2" --> SPSA2
```

Figure 4. Camera Parameters Model. This figure illustrates the architecture of our custom SegFormer-based camera parameter estimator. The model consists of an encoder-decoder structure, where the encoder is based on the SegFormer architecture and the decoder includes two heads: one for predicting camera parameters (position, orientation, and field of view) and another for generating UV heatmaps.

Diagram showing the pipeline for camera parameter estimation, including a Custom Segformer, ResNet50 Segmentation, and keypoint alignment to produce final camera parameters.

Figure 5. The pipeline estimates camera parameters by combining a custom SegFormer model for initial predictions and a ResNet50-based segmentation for keypoint detection. The parameters are refined using keypoint alignment to obtain the final camera pose.

Six histograms comparing fov, X_coordinate, Y_coordinate, Z_coordinate, pan, and tilt distributions for Synthetic Data (blue) and Real Data (orange).

Figure 6. Real and synthetic data statistics. The histograms compare the distributions of key camera parameters and coordinate values between real and synthetic datasets. The X, Y, and Z coordinates represent camera positions with respect to the center of the football field, which serves as the origin (0,0,0). The FIFA standard field dimensions are 105 meters (length) × 68 meters (width). The field of view (FoV), pan, and tilt angles illustrate differences in camera configurations across datasets, while roll is fixed at 0 for all images. The synthetic data (blue) shows a broader and more uniform distribution, while the real data (orange) exhibits a more concentrated range of values, indicating the constrained nature of real-world camera placements.

Synthetic soccer stadium view from the stands

(a) FoV: 0.86, $c_x$: -48.19, $c_y$: 72.27, $c_z$: -13.31, Pan: -0.06, Tilt: 1.35, Roll: 0.0

Overhead synthetic view of a soccer field with players

(b) FoV: 0.47, $c_x$: -11.75, $c_y$: 74.62, $c_z$: -34.18, Pan: -0.12, Tilt: 1.16, Roll: 0.0

High-angle synthetic view of a soccer match in progress

(c) FoV: 0.78, $c_x$: 26.77, $c_y$: 44.59, $c_z$: -34.42, Pan: -0.71, Tilt: 1.23, Roll: 0.0

Wide-angle synthetic view of the entire soccer stadium

(d) FoV: 1.29, $c_x$: 57.28, $c_y$: 94.18, $c_z$: -39.87, Pan: -0.49, Tilt: 1.25, Roll: 0.0

Figure 7. Examples from the synthetic dataset with corresponding camera parameters.

Diagram showing the projection of keypoints, Y-coordinate heatmaps, and X-coordinate heatmaps from a bird's-eye view to image space using a homography matrix H. The top row shows keypoints on a football pitch diagram being mapped to a real match image. The middle row shows a Y-coordinate gradient heatmap being warped. The bottom row shows an X-coordinate gradient heatmap being warped.

Figure 8. Keypoints, Y-coordinate heatmap (bird’s-eye view), and X-coordinate heatmap are projected into image space using a homography matrix derived from the camera parameters.