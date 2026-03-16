---
layout: post
title:  "Contrastive Learning methods with Negative Examples"
description: ""
date:   2026-02-10 15:47:45 +0100
# updated:   2022-10-14 10:51:24 +0100
categories: machine-learning
tags: ssl moco simclr
disqus_comments: true
---

Self-supervised learning (SSL) is a method of machine learning that learns useful representations from unlabeled data by creating a supervised signal from the data itself. Concretely, it (i) defines a auxiliary task that produces targets automatically, (ii) trains an encoder to solve this task using a loss function, and (iii) reuses the learned representation for downstream tasks with limited labels.

Initially, it  was conceived as an approach that employs an architecture that can learn in natural environments featuring diverse modalities. It later advanced into the learning that involves generating output labels "intrinsically" from data examples by revealing the relationships between data components like predicting any part of the input from any other part; predicting the future from the past; predicting the invisible form the visible; or predicting any masked, or corrupted part from all available parts. It has been expanded to encompass methods that operate without human-annotated labels. More commonly the fundamental concept of SSL conceives the term "pretext task". "Pretext" denotes that the task being solved is not the primary objective but serves as a means to generated a robust model.

A common misunderstanding in literature is the difference between self-supervised learning and unsupervised learning. Self-supervised learning falls under unsupervised learning because apparently they both learn with unlabelled data. But while unsupervised learning aims to find inherent structures (patterns, groups) directly, SSL uses clever data transformations to generate pseudo-labels, effectively teaching itself to solve problems similar to supervised tasks without human annotation.

Contrastive learning (CL) methods have two aspects in common; (i) enforce view-consistency (positive similarity) (ii) prevent representation collapse by maintaining global diversity in the embedding space [[wang][wang]{:target="_blank"}]. In representation learning, a trained network learns a geometric mapping of each input to a point (or vector) positioned in a space often called *embedding or latent space*. In contrastive learning, the learning objective learns two aspects; position two vectors of the same sample as close together as possible and dissimilar ones as far apart as possible. The agreement is maximized for two augmented views of the same sample (alignment) while also enforcing enough diversity of this relationships in the embedding space to avoid trivial representation (often referred to as collapse). 

The pretext task in negative examples-based CL is *instance discrimination* where the model treats each training example as its own class and learns an embedding space that maximizes the proximity of augmented views of the same example while maximizing the distance with views of other examples. The definition of positive and negative examples varies depending on factors such as modality being considered and specific requirements. For instance, in vision-based video understanding, performance often depends on temporal and spatial consistency and on the co-occurrence of modalities. Primarily, augmentation plays a huge role in the learning objective of these methods. It influences the learning objective by specifying which features maintain the inherent nature of the data. By generating multiple, semantically consistent views of the same instance, augmentation creates the positive pairs that the model must pull together, while all other instances act as negatives to be pushed apart. This forces the encoder to learn features that are invariant to nuisance factors introduced by the augmentations rather than memorizing superficial cues. MoCo [[mocov2][mocov2]{:target="_blank"}] framed the CL objective as a dictionary look-up task. In the framework, a query $q$ exist and a set of encoded examples $\lbrace k_0, k_1, k_2, \ldots\rbrace$ serve as the keys in a dictionary. A single key in the set is the positive key (matches the query) and the contrastive loss is employed as

$$
\ell_q = -\log\frac{\exp(q\cdot k_+/\tau)}{\sum_{i=1}^K\exp(q\cdot k_i/\tau)}
$$

where $\tau$ is the temperature hyperparameter. SimCLR [[simclr][simclr]{:target="_blank"}] applies the same contrastive loss objective but applied at a batch level such that each sample has one positive pair and 2N-1 negatives from a batch of N samples with 2N augmented instances. The loss objective for a pair of augmented instances, $i,j$, is

$$
\ell_{i,j} = - \log \frac{\exp(\operatorname{sim}(\mathbf{z}_i, \mathbf{z}_j)/\tau)}{\sum_{k=1}^{2N}\mathbb{1}_{[k \neq i]}\exp(\operatorname{sim}(\mathbf{z}_i, \mathbf{z}_k)/\tau)}
$$

where $\mathbb{1}_{[k\neq i]}\in\lbrace0,1\rbrace$ is an indicator function equal to 1 iff $k\neq i$.
<div class="row">
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.liquid loading="eager" path="assets/img/projects/moco-vs-simclr.png" title="moco-vs-simclr" class="img-fluid rounded z-depth-1" %}
    </div>
</div>
<div class="caption">
    SimCLR vs MoCo framework for contrastive learning. Adapted from [<a href="https://arxiv.org/pdf/2003.04297">mocov2</a>]
</div>

Another major difference between both methods, besides how they obtain a larger set of negatives, is how the key representations are produced and updated. In MoCo, the keys are generated with a momentum encoder which is a slowly updated version of the trained encoder. Whereas in SimCLR, there is no momentum encoder, all representations are produced by the same end-to-end network. The figure above nicely captures the differences between both methods. In an end-to-end mechanism like SimCLR, the negative keys are from the same batch and updated end to-end by back-propagation. MoCo instead maintains negatives in a queue (memory bank), so each batch only needs to encode the current queries and their corresponding positive keys, while the remaining negatives are reused from previous batches. Consequently, SimCLR typically relies on large batch sizes to provide a large set of negatives for effective instance discrimination which can introduce significant compute and memory pressure. MoCo, on the other hand, decouples the batch size from the number of negatives, making it generally more practical  under limited compute or memory budgets.

## Goal

In this post, we compare these methods to make a practical statement about when each one is the better choice for contrastive learning with negative examples. We focus on the two parts of the comparison that are supported by our experiments;

1. Test of performance -- linear probing and clustering quality
2. Test of sensitivity to augmentations

The performance test gives insight into representation quality in our controlled setup, while the augmentation sensitivity test shows how dependent each method is on the chosen view generation strategy. We study both methods under the same training environment with the same dataset, GPU, and shared hyperparameters. The details of the experiment are given by the config shown below.
```yaml
# shared between simclr and moco
model: shufflenet_v2_05
output_dim: 256 # dimension of output projections used in the loss computation
temperature: 0.07
num_epochs: 400
batch_size: 256
learning_rate: 0.01 # moco uses 0.03 base_lr in the formular (base_lr * batch_size / 256)
weight_decay: 1e-4
pretraining_dataset: AoAM
augmentation: MTC@(32, 64, 128, 256, 512) and AD@0.2

# additional hp for moco
hidden_dim: 1024
moco_momentum: 0.996
moco_cosine: true
```

The shared config shows the model structure, training hyperparameters, dataset and augmentation strategy. The dataset, AoAM is a 4-channel MIMO dataset used in the project [iqssl][iqssl]{:target="_blank"} (please check it out for details). Similarly, the augmentation strategies, MTC and AD are discussed in the project (`(32, 64, 128, 256, 512)` is the set of crop choices used).

## Results and Analysis

In the following analysis, we freeze the pretrained encoder when evaluating a task and only train the head (linear probing)--see detailed linear probing discussion in the project.

### Performance Test

First, we compare both methods on the two downstream tasks associated with the pretraining dataset. The figure below presents the linear probing results (frozen pretrained encoder with a trainable lightweight MLP head) of the two methods compared with a supervised end-to-end trained model on each sample budget: 1, 10, 100, 200, and 500. The results show that both pretrained models are competitive with the supervised baseline. At high sample budgets, supervised training surpasses the pretrained models on the AoA task, but the pretrained methods remain competitive on AMC. Overall, SimCLR gives the stronger downstream performance across the comparison, although MoCo may provide slightly stronger linear separability on AMC at the extreme low-data regime, as suggested by the 1-shot result.

<div class="row">
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.liquid loading="eager" path="assets/img/projects/moco-vs-simclr-lp.png" title="lp result between moco and simclr" class="img-fluid rounded z-depth-1" %}
    </div>
</div>
<div class="caption">
    Linear probing accuracy of <strong>shared encoder</strong> pretrained using SimCLR or MoCo and a supervised end-to-end trained network on each sample budget.
</div>

In addition to the linear probing results, we also evaluate the clustering quality of the pretrained encoders on each task. A metric used for clustering quality measurement is the **silhouette score**. Given a dataset, $\lbrace x_i\rbrace_{i=1}^N$ and the clusters, $C(i) = C_k$ for $k\in\lbrace1,\ldots,K\rbrace$ for all data points $i$ in $N$. We define a distance function $d(\cdot,\cdot)$ such that the silhouette score is given as

$$
s(i) = \frac{b(i)-a(i)}{\max\lbrace a(i), b(i)\rbrace}
$$

where $a(i)$ is the average distance from $x_i$ to other points in its own cluster, that is, $a(i) = \frac{1}{\lvert C(i)\rvert-1}\sum_{\mathbb{1}[j\neq i]x_j\in C(i)}d(x_i,x_j)$ and $b(i)$ is the separation between nearest clusters, that is, $b(i) = \min_{C\neq C(i)}\frac{1}{\lvert C\rvert}\sum_{x_j\in C}d(x_i,x_j)$.

The silhouette score, $s(i)$ then satisfies $-1\leq s(i)\leq1$ where higher is better; closer to its own cluster and farther from other clusters. The table below presents the silhouette score on each task for each model. We produce embeddings on each task and compute the silhouette score on 5850 data points. These results support the linear probing observations. SimCLR produces stronger clustering on AoA, while MoCo is more balanced across both tasks and performs better on AMC. We attribute this difference to how strongly each method couples to the augmentation policy. SimCLR is more aligned with the chosen augmentations and therefore exposes their effect more directly in the learned representation. MoCo, through the slowly updated momentum encoder, introduces a stabilizing bias that makes the representation less tightly tied to the exact augmentation recipe.

|| Task ||
| --- | --- | --- |
| **Method** | **AoA** (225) | **AMC** (7) |
| SimCLR | 0.235 | 0.069 |
| MoCo | 0.158 | 0.104 |

The clustering quality is calculated without any tuning and mostly reflects the behaviour at 1-shot finetuning shown in the linear probing results.

### Augmentation Sensitivity Test

In this test, we measure how sensitive each method is to the augmentation strategy used. Here, the goal is to observe whether the model falls into shortcut learning or weak representations: the training loss can decrease quickly while the downstream quality fails to improve. The figure below shows the linear probing results on the shared encoders when no augmentation is applied during pretraining. In this setting, the MoCo shared encoder outperforms the SimCLR shared encoder on both tasks, which reflects the regularizing effect of the momentum encoder when the view generation strategy is weak or missing.

<div class="row">
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.liquid loading="eager" path="assets/img/projects/augmentation-comparison.png" title="augmentation sensitivity test" class="img-fluid rounded z-depth-1" %}
    </div>
</div>
<div class="caption">
    Linear probing accuracy of <strong>shared encoder</strong> with no augmentation between SimCLR and MoCo.
</div>

However, compared to the results with augmentations, we observe a significant performance drop, especially on the AoA task. This confirms that SimCLR benefits more from a well-designed augmentation pipeline, but it also becomes more vulnerable when the augmentation choice is poor. MoCo is more robust in this setting. Its absolute peak performance is slightly lower, yet its degradation is smaller, which makes it the safer option when augmentation bias is a concern.

## Conclusion

With these results, SimCLR is the better option when the objective is to push performance and actively study the effect of augmentations. It gives the stronger overall downstream results and its representations reveal the consequences of the augmentation strategy more clearly. That makes it a useful choice when augmentation design is part of the research question.

MoCo, however, is the more stable option when robustness matters more than extracting the last bit of performance from a carefully tuned augmentation pipeline. Its performance remains comparable, and the momentum encoder reduces sensitivity to poor or missing augmentations. In that sense, MoCo is useful for handling augmentation bias: it provides a representation that is less tightly coupled to the exact transformation policy and therefore degrades more gracefully when the augmentation setup is imperfect.

[wang]: https://arxiv.org/pdf/2005.10242
[mocov2]: https://arxiv.org/pdf/2003.04297
[simclr]: https://proceedings.mlr.press/v119/chen20j/chen20j.pdf
[iqssl]: /projects/4_iqssl/
