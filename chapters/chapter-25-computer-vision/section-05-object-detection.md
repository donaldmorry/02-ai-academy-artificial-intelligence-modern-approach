# Section 25.5: Object Detection

> **Source:** Russell & Norvig, *Artificial Intelligence: A Modern Approach* (4th ed.), Chapter 25.6
> **Previous:** [Section 25.4 - Image Classification](./section-04-image-classification.md)
> **Next:** [Section 25.6 - Segmentation](./section-06-segmentation.md)
> **Vocabulary:** [../../GLOSSARY.md](../../GLOSSARY.md)
> **Math conventions:** [../../MATH_CONVENTIONS.md](../../MATH_CONVENTIONS.md)

---
## Beyond Classification: Where Are Objects?

**Object detection** localizes and classifies all object instances in an image. Output: set of bounding boxes $\{(b_i, c_i, s_i)\}$ where $b_i$ is box coordinates, $c_i$ is class, $s_i$ is confidence score.

Russell and Norvig distinguish:
- **Classification** - one label per image
- **Detection** - multiple objects, each with location
- **Segmentation** - pixel-level masks (Section 25.6)

> **Readable form:** Detection answers "what is in the image AND where?" - drawing boxes around every car, person, and sign.

---

## Sliding Window and Region Proposals

Naive approach: slide fixed-size window across image at multiple scales; classify each patch.

Problems:
- **Computational cost** - millions of windows per image
- **Scale variation** - objects appear at many sizes
- **Aspect ratios** - cars vs. pedestrians vs. poles

**Region Proposal Networks** score candidate regions likely to contain objects, reducing search space.

| Method | Proposal source |
|--------|-----------------|
| Selective Search | Grouping superpixels |
| Edge Boxes | Edge-based grouping |
| RPN (Faster R-CNN) | Learned anchors |

---

## IoU and Evaluation

**Intersection over Union (IoU)** measures box overlap:

$$
\text{IoU} = \frac{\text{Area}(B_{\text{pred}} \cap B_{\text{gt}})}{\text{Area}(B_{\text{pred}} \cup B_{\text{gt}})}
$$
> **Readable form:** This definition states exactly how the left-hand quantity is computed from the variables on the right.

Typically IoU $\geq 0.5$ counts as true positive. **mAP** (mean Average Precision) averages AP across classes at multiple IoU thresholds (COCO uses 0.5:0.95).

**Non-Maximum Suppression (NMS):**
1. Sort boxes by confidence
2. Keep highest-scoring box
3. Suppress overlapping boxes with IoU $> \tau$
4. Repeat

```python
def iou(box_a, box_b):
    x1 = max(box_a[0], box_b[0])
    y1 = max(box_a[1], box_b[1])
    x2 = min(box_a[2], box_b[2])
    y2 = min(box_a[3], box_b[3])
    inter = max(0, x2 - x1) * max(0, y2 - y1)
    area_a = (box_a[2] - box_a[0]) * (box_a[3] - box_a[1])
    area_b = (box_b[2] - box_b[0]) * (box_b[3] - box_b[1])
    return inter / (area_a + area_b - inter + 1e-6)
```

---

## R-CNN Family

### R-CNN (2014)
1. Region proposals (~2000/image)
2. Warp each to fixed size
3. CNN features → SVM classifier + bbox regressor

Slow - runs CNN per proposal.

### Fast R-CNN (2015)
Single CNN on full image; ROI pooling extracts features per proposal.

### Faster R-CNN (2015)
**Region Proposal Network (RPN)** shares backbone with detector - end-to-end trainable.

**Anchor boxes:** predefined scales and aspect ratios at each spatial location. RPN predicts:
- Objectness score (foreground vs. background)
- Box refinements $(\Delta x, \Delta y, \Delta w, \Delta h)$

---

## One-Stage Detectors: YOLO and SSD

**YOLO** (You Only Look Once) divides image into $S \times S$ grid. Each cell predicts $B$ boxes and class probabilities in one forward pass.

**SSD** (Single Shot Detector) uses multi-scale feature maps for objects of different sizes.

| Property | Two-stage (Faster R-CNN) | One-stage (YOLO) |
|----------|--------------------------|------------------|
| Speed | Slower | Faster (real-time) |
| Small objects | Often better | Historically weaker |
| Training | More complex | Simpler |

Modern YOLO versions (v5-v8) close accuracy gap while maintaining speed.

> **Readable form:** Two-stage detectors "look twice" - find regions, then classify. One-stage detectors "glance once" - faster but historically missed small things.

---

## Loss Functions

Detection loss combines:

$$
\mathcal{L} = \mathcal{L}_{\text{cls}} + \lambda_{\text{box}} \mathcal{L}_{\text{box}} + \lambda_{\text{obj}} \mathcal{L}_{\text{obj}}
$$
> **Readable form:** This loss is the scalar training signal: it is larger when predictions violate the target and smaller when they match.

**Box regression** often uses Smooth L1 or IoU-based losses (GIoU, DIoU, CIoU) for better gradient behavior.

**Focal loss** down-weights easy negatives in dense detectors:

$$
\mathrm{FL}(p_t) = -\alpha (1 - p_t)^\gamma \log(p_t)
$$
> **Readable form:** The update moves an estimate toward a reward-informed target using the learning rate.

Addresses extreme foreground/background imbalance.

---

## Anchor-Free Methods

**CenterNet, FCOS** predict object centers and sizes directly without anchor boxes - simplifies design and reduces hyperparameter tuning.

Trend: unified architectures handling detection, segmentation, and keypoints (YOLO-seg, Mask R-CNN).

---

## Applications

| Domain | Requirements |
|--------|--------------|
| Autonomous driving | Real-time, robust to weather |
| Retail | Shelf monitoring, checkout |
| Security | Intrusion detection |
| Medical | Lesion detection in radiology |

Each domain imposes latency, accuracy, and explainability constraints.

---

## AIMA Perspective

Russell and Norvig connect detection to **acting agents** - a robot must know *where* obstacles and grasp targets are, not just *what* category they belong to. Detection outputs feed planning and manipulation ([Chapter 26](../chapter-26-robotics/README.md)).

The textbook notes detection is harder than classification because the **output space is variable-sized** - number of objects per image differs.

---

## Practical Deployment

| Challenge | Mitigation |
|-----------|------------|
| Small objects | Higher resolution, FPN, tiling |
| Occlusion | Context modeling, temporal fusion (video) |
| Class imbalance | Oversampling, focal loss |
| Domain shift | Fine-tune on target domain data |

Monitor per-class mAP in production; aggregate accuracy hides rare-class failures.

---

## Key Takeaways

1. Object detection outputs bounding boxes, classes, and scores for all instances in an image.
2. IoU and mAP are standard evaluation metrics; NMS removes duplicate detections.
3. Faster R-CNN uses two-stage proposal + classification; YOLO achieves real-time one-stage detection.
4. Anchor boxes handle scale/aspect variation; anchor-free methods simplify the pipeline.
5. Detection is essential for spatial reasoning in robotics and autonomous systems.

## Additional Notes

This section's extra practice: choose one real image pair, state the visual representation, identify the failure mode, and write the diagnostic you would use before trusting the perception result in an agent loop. Computer vision is useful only when its output is tied to an action, belief update, or safety decision.

## Agent-Loop Review Check

Take one detection output and trace what happens next: which planner, controller, or human decision consumes the box? If a false positive or missed object would change the action, record that as the deployment risk.

---


## Key Terms (Glossary)

- **R-CNN** - see [../../GLOSSARY.md](../../GLOSSARY.md)
- **YOLO** - see [../../GLOSSARY.md](../../GLOSSARY.md)
- **anchor box** - see [../../GLOSSARY.md](../../GLOSSARY.md)

---

## Reflection Questions

1. Implement IoU and NMS for a set of predicted boxes.
2. Compare inference speed of a two-stage vs. one-stage detector on the same GPU.
3. Explain why small objects are harder for single-scale detectors.
4. Visualize anchor boxes at one spatial location for three scales and three aspect ratios.
5. Analyze false positives on a pedestrian detector - are they background texture or partial persons?

---

**Previous:** [Section 25.4 - Image Classification](./section-04-image-classification.md) · **Next:** [Section 25.6 - Segmentation](./section-06-segmentation.md)

---

## References

1. Russell, S., & Norvig, P. (2020). *Artificial Intelligence: A Modern Approach* (4th ed.). [aima.cs.berkeley.edu](https://aima.cs.berkeley.edu/)
2. Russell, S., & Norvig, P. - [aima-python](https://github.com/aimacode/aima-python)


