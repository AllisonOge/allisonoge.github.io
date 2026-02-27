---
layout: page
title: IQSSL
description: demonstrating multi-task learning capabilities with self-supervised learning
img: assets/img/projects/moco-framework.png
importance: 1
category: work
related_publications: true
disqus_comments: false
toc:
  sidebar: left
---

Following the widespread deployment of 5G networks, research attention has shifted toward 6G, where ultra-low latency, higher data rates, and ubiquitous reliability are expected to be paired with increasingly *AI-enabled* network functionalities. A recurring bottleneck, however, is that many high-performing wireless learning pipelines still depend on large, carefully labeled datasets, which are expensive (and sometimes impractical) to obtain at scale. In contrast, self-supervised learning (SSL) has recently enabled strong representation learning in other domains by exploiting abundant unlabeled data to learn reusable features that transfer across tasks. This same premise is particularly attractive for wireless systems because raw IQ recordings are plentiful, while task-specific annotations are costly and deployment-dependent.

Despite emerging interest in wireless SSL for problems such as localization/sensing, channel estimation, and RF fingerprinting, most existing studies are still framed around a *single* downstream task. In this project, we therefore ask a more general question: *can SSL learn radio representations that transfer effectively across multiple wireless tasks?* To explore this, we draw inspiration from contrastive learning, a contrastive framework that has matured into a strong general-purpose representation learner in vision and early works in the radio domain, and adapt it to *generic* radio representation learning. The central hypothesis is that, if we can construct radio-specific "views" of the same underlying signal instance (via augmentations that preserve semantic structure), then contrastive pretraining can produce invariant latent representations that are useful beyond any single task.

Our goal in this project is two-fold: 1) we devise a robust SSL scheme based on momentum contrast to learn wireless representations directly from received IQ streams using RF-specific augmentations; 2) we assess the transferability of the learned representations across two downstream tasks: angle of arrival (AoA) estimation (a regression task) and automatic modulation classification (AMC) (a classification task). We study transfer using *frozen* feature extraction, where strong performance indicates that the learned representation produced by the pretrained model is linearly separable and not merely adaptable after end-to-end retraining and compare against fully supervised baselines.

## Problem formulation

**Overview.** We consider a MIMO setup with $N_t$ transmit antennas and  $N_r$ receive antennas. The transmitter sends a variety of modulation signals and the receiver antenna array captures the transmitted signals. The received signals are perturbed by propagation effects such as multipath and noise, at angles relative to the transmitter. The angles are the angle of elevation, $\theta$ and the azimuth angle, $\phi$, which are controlled by a servo motor at the receiver. The dataset is also generated with a set of different modulations, $m$. These are 16-QAM, 64-QAM, BPSK, QPSK, PAM4, and continuous wave (CW) AM modulation.

Let $D=\{(x_i, \theta_i, \phi_i, m_i)\}_{i=1}^{N}$ denote the dataset, where $x_i \in \mathbb{C}^{N_r \times L}$ is the received IQ data across $N_r$ antennas and $L$ time samples. We derived the unlabelled pool from the dataset by discarding the labels

$$
\mathcal{U} =\{x_i\}_{i=1}^{N}.
$$

**Multi-task learning goal.** Our goal is to learn a *single* transferable representation from $x$ that supports *two tasks jointly*:
(i) angle-of-arrival (AoA) prediction, that is, estimating $(\theta,\phi)$, and
(ii) modulation classification (AMC), that is, predicting $m$. Concretely, we learn a shared encoder $f(x;\Theta_b)$ and two task heads $h_{\text{AoA}}$ and $h_{\text{AMC}}$. This head are lightweight linear classifiers that maps the features of shared encoder to the assigned tasks.

We learn the shared encoder on the unlabelled pool, $\mathcal{U}$ and the task heads on the dataset, $D$ containing their labels and a subset of it, $D_{\text{train}} \subset D$.

**Learning objective.** Self-supervised learning constructs training signals directly from the unlabeled dataset $\mathcal{U}$ by applying stochastic augmentations. Under the contrastive learning paradigm, two augmented views of the same IQ sample are generated and passed through an encoder to produce embeddings $z^{(1)}$ and $z^{(2)}$. The model is trained to align embeddings of matching views while separating them from embeddings of other samples (negatives) in $\mathcal{U}$. We write the resulting contrastive objective as

$$
\mathcal{L}_{\text{SSL}} = -\log \frac{\exp(\operatorname{sim}(z^{(1)},z^{(2)})/\tau)}{\exp(\operatorname{sim}(z^{(1)},z^{(2)})/\tau) + \sum_j\exp(\operatorname{sim}(z^{(1)},z_j)/\tau)}
$$

where $\operatorname{sim}(\cdot)$ denotes a similarity measure and $\tau$ is a temperature parameter. This objective increases similarity between the two views of the same instance and decreases similarity to embeddings $z_j$ from other instances in $\mathcal{U}$.

## The MoCo framework

<div class="row">
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.liquid loading="eager" path="assets/img/projects/moco-framework.png" title="moco-framework" class="img-fluid rounded z-depth-1" %}
    </div>
</div>
<div class="caption">
    Momentum Contrast SSL framework
</div>

We train the shared encoder using momentum contrast (MoCo) [[mocov2][mocov2]{:target="_blank"}], as shown in Figure  above. In the field of self-supervised learning, several contrastive methods have emerged and we have briefly discussed them in the previous chapter. One notable method is momentum contrast (MoCo), which has evolved significantly since its introduction in 2020, achieving state-of-the-art performance in the vision domain. Motivated by its success, we investigate MoCo's potential for developing multi-task few-shot and zero-shot learners. The MoCo framework has two branches of the encoder networks used to translate the input signal to a low-dimensional vector. The objective of the network is to pull the two low-dimensional vectors of different augmented inputs (views), of an instance close together and away from all other instances in the dataset. Concretely, let $x$ and $x_j'$ be two augmented views of the network. The views are encoded by two neural networks, $f(\cdot;\Theta)$ and $g(\cdot;\Theta)$ such that $q = f(x;\Theta)$ and $k_j = g(x'_j;\Theta)$, assuming they are parameterized by weights, $\Theta$. One network is trained while the other is slowly updated with an exponential moving average (EMA).

MoCo aims to minimize the contrastive loss function and maximize invariant latent representations that are transferrable to downstream (related) tasks. 
The contrastive loss function [[mocov3][mocov3]{:target="_blank"}] is given as follows

$$
\mathcal{L}_{c} = \frac{1}{B}\sum_{i=1}^{B}\left(-\log\frac{\exp(q_i\cdot k_i/\tau)}{\sum_{j=1}^M\exp(q_i\cdot k_j/\tau)} \right) \times 2\tau.
$$

The loss function is computed across a batch size $B$ and memory bank $M$ to maximize the agreement between the positive pair of query and key $q_i\cdot k_i$ associated with the views of an input while also maximizing the disagreement with other keys in the memory bank $q_i\cdot k_j$. The temperature parameter $\tau$ is a hyper-parameter that scales the loss.

For the i-th example in a batch, B, the query $q$ of the input in the equation, has a positive key, $k^+$, and M - 1 negatives, $k^-$. With augmentations, the network learns encodings such that the dot product $q_i\cdot k_j$ is maximized for different augmented views of the same example ($i=j$) and minimizes the dot product with all other keys.

## Data Augmentation Methods

RF IQ signals contain task-relevant structure in both time (waveform dynamics) and space (multi-antenna channels). We therefore design augmentations that perturb the input while preserving label-relevant signal characteristics. For example, modulation can often be inferred from a short window spanning at least one symbol period. Prior work has surveyed effective augmentations for radio signals [[davis][davis]{:target="_blank"}], and recent evidence highlights the importance of core augmentation for contrastive learning [[iqfm][iqfm]{:target="_blank"}]. Based on these insights, we use two augmentations: *multi-time cropping* (MTC) and *antenna dropout* (AD) {% cite kanu2025self %}.

**Multi-Time Cropping (MTC).** Multi-Time Cropping randomly crops a contiguous time segment from an IQ sample, using a crop length drawn from a predefined set.

**Definition.** Let $x\in\mathbb{R}^{4\times 2\times L}$ and let $\mathcal{C}\subseteq\{1,\ldots,L\}$ denote a set of admissible crop lengths.
Sample a crop length $\ell_c \sim Uniform(\mathcal{C})$. If $\ell_c < L$, sample a start index
$s_k \sim Uniform\big(\{0,\ldots,L-\ell_c\}\big)$ and define

$$
\operatorname{MTC}(x, \ell_c)=
\begin{cases}
x_{:,:,\,s_k:s_k+\ell_c}, & \text{if } \ell_c < L,\\
x, & \text{otherwise}
\end{cases}
$$

Thus, for a sample, $x$, the two stochastic augmentations $\operatorname{MTC}^{(1)}$, $\operatorname{MTC}^{(2)}$ produce views of different crop lengths sampled at different locations.

**Antenna Dropout (AD) {% cite kanu2025self %}.** Antenna Dropout randomly removes entire antenna channels by multiplying the input with a Bernoulli mask, where each antenna is retained with probability $1-p$.

**Definition.** Let $x\in\mathbb{R}^{4\times 2\times L}$ and let $m\in{0,1}^{4\times 1\times 1}$ be a channel mask with independent entries
$m_{i,1,1} \sim \mathrm{Bernoulli}(1-p)$ for $i\in{1,\dots,4}$.
Broadcasting $m$ across the last two dimensions, we define

$$
\operatorname{AD}(x, p) = m \odot x
$$

Similarly, for the contrastive objective, the two stochastic augmentations $\operatorname{AD}^{(1)}$ and $\operatorname{AD}^{(2)}$ produce views that expose different subsets of antenna channels.

## Performance Metrics and Tasks

### Finetuning and Downstream Tasks
In our analysis, we freeze the pretrained encoder, $f(;\Theta)$, extract an embedding $e_i\in\mathbb{R}^{d_h}$ for each sample, and train a task-specific MLP head, $h_t(\cdot;\psi_t)$ on top of these fixed embeddings. It is refered to as *linear probing* and evaluates how linearly separable the embeddings of the pretrained encoder is towards a task. We consider two downstream tasks: angle of arrival (AoA) estimation and Automatic Modulation Classification (AMC).

### Metrics
We evaluate our methods by the performance of the encoder $f(\cdot;\Theta)$ on the downstream tasks with frozen and fine-tuned weights. For the angle of arrival task, the metric of evaluation is the mean absolute error (MAE) across all angles in the evaluation dataset, while the metrics of evaluation for the automatic modulation classification are accuracy, precision, and recall. The use of MAE is for ease of interpretation, another similar metric used in literature is the root mean square error (RMSE). These performances are compared with those of the fully supervised counterparts.

### Data Efficiency
Additionally, we investigate the data efficiency of the latent representation. Data efficiency refers to the amount of labels required to build a downstream task using $f(\cdot;\Theta)$. This is evaluated for both tasks and the metrics are evaluated against their fully supervised counterparts.

## Results and Analysis

In our analysis we present the results of adapting only a lightweight MLP on the embeddings produced with our pretrained encoder.


### Angle of Arrival task

We present the mean-absolute-error (MAE) scores of three experiments; linear probing with randomly initialized encoder and our pretrained (shared) encoder as well as the fully supervised end-to-end network on the AoA task. In the linear probing test, we adapt a lightweight MLP head on the embedding produced. This is shown in the table below. We observe that our pretrained encoder still underperforms the supervised baseline but compared to the randomly initialized encoder we measure up to 5x (37.64/7.78) reduction in the error.

| Xavier Init. | Shared Encoder | Supervised | 
| --- | --- | --- |
| 37.64 | 7.78 | 0.83 |

The linear probing test measures linear separability of the embeddings on the task and a high performance indicates a good separation of the classes in the task. Without using labelled data in the pretraining and using the SSL objective, we have derived a single shared encoder that learns transferable representation for the task that can be adapted on a lightweight head. We further analyzed the performance of our shared encoder on the angles in this task. We present the MAE of each unique angle pair in the dataset as a heat map where the darker tiles are the low errors and the bright tiles are the high errors. We observe that the outer regions with higher angles have weaker separability in the shared encoder and suggest a more non-linear adaptation for better performance. 

<div class="row">
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.liquid loading="eager" path="assets/img/projects/per_angle_mae.png" title="per angle mae" class="img-fluid w-75 rounded z-depth-1" %}
    </div>
</div>
<div class="caption">
    Linear probing mean absolute error results of <strong>shared encoder</strong> analysed by angle
</div>

### Doubling the Linear Head

By doubling, the head (linear-act-linear) we improve the performance from **7.78** degree error to **1.97** degree error. Consequently we see a better performance on the outer regions of the heatmap as shown below which supports our intuition.

<div class="row">
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.liquid loading="eager" path="assets/img/projects/per_angle_mae_2_layer.png" title="per angle mae" class="img-fluid w-75 rounded z-depth-1" %}
    </div>
</div>
<div class="caption">
    Doubling the adaptable head on the <strong>shared encoder</strong>
</div>

### Automatic Modulation Classification task

Similarly, we evaluate the modulation task with three experiments showing the accuracy in the following table. The linear probing score on the shared encoder reaches the same performance as the supervised model with both having 100% accuracy. 

| Xavier Init. | Shared Encoder | Supervised | 
| --- | --- | --- |
| 0.25 | 1.0 | 1.0 |

In comparison to the randomly initialized encoder, we measure up to 4x improvement and indicates the shared encoder successfully learned both task and produced generic features that can be linearly adapted on both task.

### Data Efficiency

In the data-efficiency test, we measure the usefulness of the shared encoder on low-labelled samples. In this test we test the few shot performance between our pretrained encoder and the fully supervised end-to-end model at 1-shot, 10-shot, 100-shot and 500-shot for both task as shown in the following table. We show the MAE performance for the AoA task and the accuracy for the AMC task. In the AoA task, we use two layer head with an activation sandwich on the embeddings produced by the shared encoder whereas we use a single layer (or linear probing) for the AMC task.

| Model | Task | 1-shot | 10-shot | 100-shot | 500-shot |
| --- | --- | --- | --- | --- | --- |
| Shared Encoder| AoA | **8.73** | **3.35** | **2.32** | 2.13 |
| Supervised |  | 37.37 |  9.21 | 2.69 | **1.64** |
| Shared Encoder | AMC  | **0.34** | **0.74** | 1.0 | 1.0 |
| Supervised |  | 0.20 | 0.44 | 1.0 | 1.0 |

As expected, the performance at low-labels regions of the shared encoder outperforms the supervised baselines. We measure up to 4.3x improvement in the AoA task and 1.7x improvement in the AMC task at 1-shot sample budget. The gap tightens as the sample budget increases in the AoA task and then supervised model exceeds the shared model with two-layer task head but with a marginal gap in performance.

### Training Efficiency

In addition, we measure the training efficiency the proposed multi-task shared model contributes compared to end-to-end fully supervised training pipeline. We train a shared model and use across tasks. The savings is measured in the single pretrained cost + finetuning adaptation vs fully supervised counterparts. We present our observations on the full dataset evaluation results. The pretrained encoder is trained for 3.99 hours and the adapted heads are trained for 0.19 and 0.17 hours respectively. On the other hand, the end-to-end fully supervised learning models for both tasks are trained for 1.37 and 1.36 hours respectively. By comparison the cost in training hours for the shared encoder is 3.99+0.19+0.17=4.35 hours and the cost for the fully supervised models is 1.37+1.36=2.72 hours. As the number of downstream tasks increases, the initial training cost is conserved and this is demonstrated with the figure below. The figure is an approximation and assumes each task cost the same both for the adaptation cost and fully supervised model.

<div class="row">
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.liquid loading="eager" path="assets/img/projects/training_cost_vs_tasks.svg" title="training cost vs tasks" class="img-fluid w-75 rounded z-depth-1" %}
    </div>
</div>
<div class="caption">
    Training Efficiency of proposed solution
</div>

This project is an extension of the findings presented in the paper titled [*Self-supervised Radio Representation Learning: Can we Learn Multiple Tasks?*](https://arxiv.org/abs/2509.03077){:target="_blank"}. Here, we improve the augmentation strategy used to produce better representation in the SSL pretraining and thus eliminate the need for end-to-end finetuning described in the paper. This was motivated by the paper titled [*IQFM--A Wireless Foundation Model for I/Q Streams in AI-Native 6G*](https://ieeexplore.ieee.org/document/11372716){:target="_blank"} that uses a SimCLR contrastive approach for the SSL pretraining. We thus also compare both solutions in this [blog post][blog-post]{:target="_blank"}.

## Conclusion

We have proposed and successfully demonstrated the applicability of momentum contrastive self-supervised learning in the wireless radio domain for two tasks: angle of arrival estimation and automatic modulation classification. Our results indicate that with carefully designed augmentations and diverse data, contrastive learning can yield quality, invariant latent representations. We anticipate that more diverse datasets will further enhance performance and applicability to even more than two tasks. In summary, our findings highlight the potential of self-supervised learning to transform wireless communication tasks by reducing dependence on labelled data and improving model generalization, paving the way for scalable foundational 6G AI-native models. 


[mocov2]: https://arxiv.org/pdf/2003.04297
[mocov3]: https://arxiv.org/pdf/2104.02057
[simclr]: https://proceedings.mlr.press/v119/chen20j/chen20j.pdf
[davis]: https://ieeexplore.ieee.org/stamp/stamp.jsp?arnumber=9930826
[iqfm]: https://ieeexplore.ieee.org/document/11372716
[blog-post]: /blog/2026/moco-vs-simclr/
