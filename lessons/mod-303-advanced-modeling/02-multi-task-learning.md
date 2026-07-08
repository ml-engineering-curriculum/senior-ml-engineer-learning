# Multi-task learning in practice

## Motivation

Multi-task learning (MTL) is the regime senior ML engineers reach for most often, because most production systems predict more than one thing. A ranker jointly predicts click, dwell, purchase, and long-term retention. A content-safety classifier predicts a dozen policy categories on the same text. A medical-imaging model predicts disease presence, severity, and location. A pricing model predicts demand at several price tiers. In all of those, a naive design would train N separate models; an MTL design trains one backbone with N heads and lets the heads share a learned representation.

The bet — first named in Caruana's [*Multitask Learning*](https://link.springer.com/article/10.1023/A:1007379606734) — is that jointly-learned representations generalise better than independently-learned ones, especially when labels are scarce for some of the tasks. The tax — call it **negative transfer** — is that tasks can also fight each other and end up worse than they would have been alone.

This chapter is how you make the bet consciously. You will meet the four architectural shapes, the three ways loss balancing goes wrong, the diagnostics that let you spot negative transfer before it ships, and the operational envelope that MTL costs you at serving time.

## Four architectural shapes to have in your pocket

The literature has more names than shapes. At L30 you should be able to reach for these four and know when each is the right fit.

### 1. Hard parameter sharing — one backbone, N heads

The classical shape. One shared trunk (encoder, embedding stack, transformer, whatever the modality demands) feeds N small task-specific heads, each with its own loss. Gradients from each task's loss update the shared trunk. This is what "MTL" means in most production systems when nobody adds a qualifier.

Why it dominates in practice: it is the cheapest to serve (one forward pass through the trunk, N cheap heads), the easiest to reason about, and — when the tasks genuinely share representation — often the best-performing shape. Ruder's [*An Overview of Multi-Task Learning in Deep Neural Networks*](https://arxiv.org/abs/1706.05098) is the standard survey.

When it breaks: when tasks are only weakly related, the shared trunk becomes a compromise trunk — none of the heads gets the representation it would have wanted. That is negative transfer, and it is the failure mode you are watching for.

### 2. Soft parameter sharing — N backbones with a similarity penalty

Each task keeps its own private trunk, but a regulariser penalises divergence between corresponding layers across tasks. It is more expressive than hard sharing (each task's trunk can specialise) but almost always more expensive to train and to serve, and the regulariser hyperparameter (how strong the similarity penalty is) becomes yet another knob. In production ML at senior altitude, soft sharing shows up in research and in cross-domain adaptation, less in day-to-day ranking systems. Know the name; reach for hard sharing first.

### 3. Mixture-of-experts and gating — shared parameters, task-conditioned routing

The Multi-gate Mixture-of-Experts (MMoE) shape from Ma et al.'s [*Modeling Task Relationships in Multi-task Learning with Multi-gate Mixture-of-Experts*](https://dl.acm.org/doi/10.1145/3219819.3220007) (KDD 2018) is the reference for large-scale industrial rankers with weakly related tasks. The trunk is a bank of expert subnetworks; per-task gates learn how much each expert contributes to each task. It gives you task specialisation without the full serving-cost tax of N private backbones.

Google's Ads and YouTube systems have described MMoE-shaped rankers publicly (e.g., YouTube's [*Recommending What Video to Watch Next: A Multitask Ranking System*](https://dl.acm.org/doi/10.1145/3298689.3346997), RecSys 2019). This is the shape to reach for when hard sharing hits a negative-transfer wall on a genuinely diverse task set.

### 4. Cross-stitch and multi-gate variants — learned per-layer sharing

Cross-stitch networks (Misra et al., CVPR 2016, [*Cross-stitch Networks for Multi-task Learning*](https://arxiv.org/abs/1604.03539)) learn per-layer combinations of task-specific activations. Conceptually a compromise between hard and soft sharing, learned rather than hyperparameter-configured. Rarely worth the operational complexity in production; useful vocabulary for research reviews.

### The default at L30

Reach for **hard parameter sharing** first. Escalate to **MMoE** when hard sharing measurably underperforms per-task models on a subset of tasks and the diagnostics in the next section confirm negative transfer, not underfitting. Everything else is research territory.

## Loss balancing — where MTL usually goes wrong

The heads have different losses. Cross-entropy on a rare category and cross-entropy on a common one produce gradient magnitudes that differ by orders of magnitude. Mean-squared error on a raw-scale regression head and log-loss on a click head are not even in the same units. If you sum them naively, the largest-magnitude loss dominates the shared trunk's updates and the small-magnitude tasks are effectively ignored.

Three families of fixes, listed by how much of the loss-balancing decision they leave to the model.

### Fixed weights — the honest baseline

```python
loss = w_click * loss_click + w_dwell * loss_dwell + w_purchase * loss_purchase
```

The weights are hyperparameters set by hand or by search. This is what almost every production MTL system ships as v1. It is honest about the weighting decision (a reviewer can see and challenge each weight), it is trivial to debug, and if you have a small number of tasks it usually works. The tuning cost grows quickly with N — a Pareto front over four weights is manageable, over ten is not.

### Uncertainty weighting — Kendall, Gal, and Cipolla

Kendall et al.'s [*Multi-Task Learning Using Uncertainty to Weigh Losses for Scene Geometry and Semantics*](https://arxiv.org/abs/1705.07115) (CVPR 2018) introduced per-task learned log-variances that automatically down-weight noisy tasks. Concretely, each task gets a learned scalar `s_i` and its contribution becomes `exp(-s_i) * loss_i + s_i`. The tasks with higher inherent noise learn a larger `s_i`, contributing less to the shared gradient. In practice this reduces the loss-weight hyperparameter search from N knobs to zero, at the cost of a few extra parameters per task and one more thing that can go wrong when a task's loss numerics are pathological.

### Gradient-based balancing — GradNorm and PCGrad

GradNorm (Chen et al., ICML 2018, [*GradNorm*](https://arxiv.org/abs/1711.02257)) rescales each task's loss weight so that the gradient magnitudes at the shared trunk are roughly equal, adapted per-step. PCGrad (Yu et al., NeurIPS 2020, [*Gradient Surgery for Multi-Task Learning*](https://arxiv.org/abs/2001.06782)) projects each task's gradient onto the orthogonal complement of any conflicting task's gradient — literally cancelling the "fighting" component. Both are effective; both add training-loop complexity.

At L30 you should know these names. Reach for fixed weights first, uncertainty weighting when your task list gets to five or more, gradient-based methods when you have measured negative transfer on specific task pairs and cannot find a shared representation that works for all of them.

## Diagnosing negative transfer

You do not know you have negative transfer until you compare against the honest baseline. The baseline is: **train each task as its own single-task model with the same backbone architecture** (frozen from the MTL initialisation or freshly initialised — do both), on the same data. Compare each task's MTL metric to its single-task baseline.

- If every task's MTL metric ≥ single-task metric, the MTL bet paid off. Ship.
- If some tasks are strictly better under MTL and none are strictly worse, ship.
- If some tasks are strictly worse under MTL, you have negative transfer on those tasks. Options: drop the losing task from MTL (train it separately), split into two MTL groups (task clustering — cluster tasks whose gradients agree; e.g., Fifty et al., [*Efficiently Identifying Task Groupings for Multi-Task Learning*](https://arxiv.org/abs/2109.04617), NeurIPS 2021), or move to a gated architecture like MMoE.

The single-task baseline table is the artefact that makes an MTL launch defensible in a review. Do not launch an MTL model whose per-task metrics you cannot compare against a single-task baseline for at least the top-priority tasks.

## Serving-time economics

The reason MTL is worth the complexity in production is almost always the **serving budget**. N separate models means N forward passes, N memory allocations, N sets of parameters loaded to the accelerator. An MTL model means one backbone forward pass and N cheap head passes.

At Google-scale ranking, the MMoE paper measured trunk sharing as the load-bearing efficiency. At small scale, the same math applies: one 200 M-parameter trunk plus five 1 M-parameter heads is a very different serving cost from five 200 M-parameter models. When you write the MTL RFC (chapter 06 of [mod-302] for the RFC skeleton), the serving-cost delta versus the single-task baseline is a required section.

The other operational cost is **evaluation surface** — every task has its own evaluation slice, its own metric threshold, and its own regression-guard. Multi-task evaluation infrastructure is one of the load-bearing hand-offs to [mod-305] (evaluation): if your eval harness cannot report per-task metrics against a per-task baseline on every run, the MTL system is unshippable in the L30 sense. You will not know when negative transfer arrives.

## A worked pattern — a two-stage ranker

The most common shape a senior ML engineer will build in the MTL regime is a **two-stage ranker**: retrieval + ranking. The ranking stage is the MTL model. Its heads typically include:

- `p_click` — probability of a click given exposure.
- `p_dwell_gt_k` — probability the user stays past threshold k, conditional on click.
- `p_conversion` — probability of a downstream conversion event.
- Sometimes: `p_negative` (report, hide, dislike) as a suppression signal.

The final score is a weighted combination (the "weighted logistic" or "value-model" shape): `score = a * logit(p_click) + b * logit(p_dwell) + c * logit(p_conv) - d * logit(p_negative)`. The head weights are set by the product objective (revenue, retention, satisfaction), not learned end-to-end.

Concretely — the shape of the code that owns this in production (PyTorch pseudocode, not runnable):

```python
class MultiTaskRanker(nn.Module):
    def __init__(self, backbone, task_heads: dict[str, nn.Module]):
        super().__init__()
        self.backbone = backbone
        self.heads = nn.ModuleDict(task_heads)

    def forward(self, batch):
        shared = self.backbone(batch["features"])
        return {name: head(shared) for name, head in self.heads.items()}

def multitask_loss(preds, labels, weights):
    return sum(
        weights[name] * losses[name](preds[name], labels[name])
        for name in preds
    )
```

Notable properties of this shape:

- The backbone is shared and does one forward pass.
- Each head is small (one to three linear layers).
- The loss weights are named data, not hyperparameters buried in a training script. Reviewers can see and challenge them.
- When product asks for a new objective (say, `p_satisfaction`), the delta is one new head plus one weight; the backbone does not have to be retrained from scratch.

That last property — extensibility — is one of the main reasons MTL wins at production scale. Building a new single-task model per objective ossifies the roadmap; adding a head to an MTL model is a two-week feature.

## Three failure modes to catch in review

- **No per-task baseline table.** The launch review claims the MTL model wins, but there is no per-task comparison against the single-task baseline. You cannot detect negative transfer. Send back for the table.
- **All losses summed with `weight = 1`.** The reviewer asks how the weights were chosen; the answer is "we didn't." One task's loss is 30x the others in magnitude and dominates the trunk. Ask for the loss-magnitude distribution and either fixed weights or uncertainty weighting.
- **Per-task eval metrics not wired into the release harness.** The model can regress on one task and pass the launch check because the aggregate metric is fine. This is a mod-305 hand-off failure. The release harness must report each task's metric against its per-task threshold, and any per-task regression is a launch block.

## When to reach for MMoE instead of hard sharing

Escalate from hard sharing to MMoE (or another gated / task-cluster shape) when:

- Your per-task baseline table shows measurable negative transfer on ≥1 task and you cannot find a fixed-weight or uncertainty-weighted setup that fixes it.
- The number of tasks has grown to a point where task pairs are demonstrably unrelated (the ranker predicts click, purchase, and returns-fraud all in one model, and returns-fraud gradients wreck the click head).
- You have production-scale traffic to justify the serving-cost increase (MMoE is meaningfully more expensive than hard sharing).

Below that bar, stay on hard sharing. MMoE is not free — the gating adds parameters, training complexity, and one more thing that can go subtly wrong under distribution shift.

## Summary

MTL is the "N correlated predictions" answer from chapter 01, and it lives on hard parameter sharing until the diagnostics say otherwise. Loss balancing is where naive implementations go wrong — start with fixed weights, escalate to uncertainty weighting or GradNorm/PCGrad if the task list grows. Negative transfer is a specific measurable failure detected against a single-task baseline; that baseline table is the artefact that makes the MTL launch defensible in review. Serving-cost economics — one shared backbone, N cheap heads — is why MTL wins at production scale. Escalate to MMoE only when hard sharing measurably fails. Exercise-02 in this module puts you through a real MTL launch memo including the loss-weighting decision and the single-task baseline comparison.
