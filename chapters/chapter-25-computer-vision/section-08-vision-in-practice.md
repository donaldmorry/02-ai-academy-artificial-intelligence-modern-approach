# Section 25.8: Vision in Practice

> **Source:** Russell & Norvig, *Artificial Intelligence: A Modern Approach* (4th ed.), Chapter 25
> **Previous:** [Section 25.7 - 3D Vision](./section-07-3d-vision.md)
> **Vocabulary:** [../../GLOSSARY.md](../../GLOSSARY.md)
> **Math conventions:** [../../MATH_CONVENTIONS.md](../../MATH_CONVENTIONS.md)

---
## From Benchmarks to Production

Academic vision benchmarks (ImageNet, COCO, KITTI) measure progress but **production systems** face distribution shift, latency budgets, adversarial inputs, and regulatory requirements. Russell and Norvig close Chapter 25 by emphasizing **engineering discipline** alongside algorithmic advances.

> **Readable form:** Winning a benchmark is like acing a practice exam - deployment is the real test with different lighting, cameras, and users every day.

---

## Datasets and Benchmarks

| Dataset | Task | Size | Notes |
|---------|------|------|-------|
| ImageNet | Classification | 1.2M images | 1000 classes |
| COCO | Detection, seg | 330K images | 80 classes, diverse |
| Open Images | Multi-task | 9M images | Long-tail classes |
| KITTI | Autonomous driving | Stereo + LiDAR | Outdoor |
| Medical (CheXpert) | Chest X-ray | 224K | Clinical labels |

**Dataset documentation** (Datasheets for Datasets): provenance, collection method, known biases, intended use.

### Label Quality

Noisy labels hurt more than model capacity. Practices:
- Multiple annotators + adjudication
- Active learning for uncertain samples
- Consistency checks (IoU between annotators)

---

## Data Augmentation

Synthetic diversity at training time reduces overfitting:

| Augmentation | Effect |
|--------------|--------|
| Random crop/flip | Translation invariance |
| Color jitter | Illumination robustness |
| Cutout / CutMix | Occlusion robustness |
| MixUp | Smoother decision boundaries |

**Test-time augmentation (TTA):** average predictions over flipped/cropped variants.

**Domain-specific:** simulate rain/fog for driving; elastic deformations for medical.

```python
import torchvision.transforms as T

train_transform = T.Compose([
    T.RandomResizedCrop(224, scale=(0.8, 1.0)),
    T.RandomHorizontalFlip(),
    T.ColorJitter(0.4, 0.4, 0.4, 0.1),
    T.ToTensor(),
    T.Normalize(mean=[0.485, 0.456, 0.406],
                std=[0.229, 0.224, 0.225]),
])
```

---

## Training at Scale

| Technique | Purpose |
|-----------|---------|
| Mixed precision (FP16) | Faster training, less memory |
| Distributed data parallel | Multi-GPU scaling |
| Gradient accumulation | Large effective batch on limited GPU |
| Learning rate warmup + cosine decay | Stable convergence |
| EMA weights | Smoother inference model |

**Reproducibility:** fix seeds, log hyperparameters, version datasets (DVC, W&B).

---

## Model Optimization for Deployment

| Method | Speedup | Accuracy trade-off |
|--------|---------|-------------------|
| Quantization (INT8) | 2-4× | Small if calibrated |
| Pruning | Variable | Retrain to recover |
| Knowledge distillation | Student smaller | Depends on teacher |
| TensorRT / ONNX | Kernel fusion | Platform-specific |

**Edge deployment:** MobileNet, EfficientNet-Lite; neural processing units (NPUs) on phones.

Latency budget example: 30 FPS driving → 33 ms total pipeline including camera capture, inference, post-processing.

---

## Monitoring and MLOps

Production vision systems need:

1. **Data drift detection** - input distribution changes (new camera, season)
2. **Prediction monitoring** - confidence histograms, per-class rates
3. **Human-in-the-loop** - review low-confidence or flagged cases
4. **Rollback** - revert model version on metric degradation

**Shadow deployment:** run new model parallel to production without affecting users.

---

## Adversarial Examples and Robustness

Small perturbations $\delta$ can fool classifiers:

$$
\mathbf{x}' = \mathbf{x} + \delta, \quad \|\delta\|_\infty < \epsilon
$$
> **Readable form:** This definition states exactly how the left-hand quantity is computed from the variables on the right.

such that $f(\mathbf{x}') \neq f(\mathbf{x})$.

**Defenses:** adversarial training, input preprocessing, certified defenses (limited).

Russell and Norvig link to AI safety ([Chapter 27](../chapter-27-philosophy-ethics-safety/README.md)) - vision in safety-critical systems must handle worst cases, not just average accuracy.

---

## Fairness and Bias in Vision

Documented issues:
- Face recognition accuracy gaps across demographics
- Geographic bias in scene understanding datasets
- Occupation stereotypes in image search

**Mitigation:**
- Representative training and test data
- Fairness metrics per subgroup
- External audits before deployment

---

## System Integration

Vision rarely stands alone:

```
Camera → ISP → Undistort → Detect/Segment → Track → Planner → Actuator
```

**Temporal consistency:** Kalman filters, optical flow across frames stabilize detections.

**Multi-camera:** surround view stitching, cross-camera tracking.

Connection to [Chapter 26](../chapter-26-robotics/README.md) for embodied pipelines.

---

## AIMA Perspective

Chapter 25 concludes that computer vision is a **mature but incomplete** field - superhuman on curated benchmarks, fragile in the open world. The rational agent framework demands:

- **Uncertainty quantification** in percepts
- **Graceful degradation** when vision fails
- **Fusion** with other sensors and reasoning

Vision provides percepts; the full agent integrates them with knowledge and goals.

---

## Career and Research Directions

| Direction | Skills |
|-----------|--------|
| Applied CV engineer | PyTorch, deployment, MLOps |
| Research scientist | Papers, novel architectures |
| ML platform | Infrastructure, scaling |
| Domain specialist | Medical, robotics, agriculture |

Stay current: arXiv cs.CV, major conferences (CVPR, ICCV, ECCV).

---

## Key Takeaways

1. Production vision requires handling distribution shift, latency, and label quality beyond benchmark accuracy.
2. Data augmentation and large diverse datasets improve robustness to real-world variation.
3. Quantization and efficient architectures enable edge deployment within latency budgets.
4. MLOps practices - monitoring, versioning, rollback - sustain production systems.
5. Adversarial examples and demographic bias demand proactive testing and mitigation.

## Additional Notes

This section's extra practice: choose one real image pair, state the visual representation, identify the failure mode, and write the diagnostic you would use before trusting the perception result in an agent loop. Computer vision is useful only when its output is tied to an action, belief update, or safety decision.

---


## Key Terms (Glossary)

- **data augmentation** - see [../../GLOSSARY.md](../../GLOSSARY.md)
- **dataset bias** - see [../../GLOSSARY.md](../../GLOSSARY.md)
- **deployment** - see [../../GLOSSARY.md](../../GLOSSARY.md)

---

## Reflection Questions

1. Design a data augmentation pipeline for a factory defect detector with 500 labeled images.
2. Estimate end-to-end latency for a 30 FPS detection pipeline with 15 ms inference.
3. Propose three monitoring metrics for a deployed face-verification system.
4. Conduct a failure analysis on 20 misclassified images - categorize root causes.
5. Write a model card section covering intended use, limitations, and fairness considerations.

---

**Previous:** [Section 25.7 - 3D Vision](./section-07-3d-vision.md)

---

## References

1. Russell, S., & Norvig, P. (2020). *Artificial Intelligence: A Modern Approach* (4th ed.). [aima.cs.berkeley.edu](https://aima.cs.berkeley.edu/)
2. Russell, S., & Norvig, P. - [aima-python](https://github.com/aimacode/aima-python)



