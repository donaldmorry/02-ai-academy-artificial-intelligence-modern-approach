# Section 25.4: Image Classification

> **Source:** Russell & Norvig, *Artificial Intelligence: A Modern Approach* (4th ed.), Chapter 25.5
> **Previous:** [Section 25.3 - Feature Detection](./section-03-feature-detection.md)
> **Next:** [Section 25.5 - Object Detection](./section-05-object-detection.md)
> **Vocabulary:** [../../GLOSSARY.md](../../GLOSSARY.md)
> **Math conventions:** [../../MATH_CONVENTIONS.md](../../MATH_CONVENTIONS.md)

---
## The Classification Task

**Image classification** assigns a single label $y \in \{1, \ldots, C\}$ to an input image $\mathbf{x}$. Russell and Norvig frame it as supervised learning: learn function $f: \mathcal{X} \to \mathcal{Y}$ from labeled training set $\{(\mathbf{x}_i, y_i)\}_{i=1}^N$.

Modern classifiers output class probabilities via softmax:

$$
P(y=c \mid \mathbf{x}) = \frac{\exp(z_c)}{\sum_{j=1}^{C} \exp(z_j)}
$$
> **Readable form:** Softmax converts class logit $z_c$ into a normalized probability by exponentiating it and dividing by the sum over all classes.

where $z = f_\theta(\mathbf{x})$ are **logits** from a neural network with parameters $\theta$.

> **Readable form:** Classification answers "what is in this picture?" with one label - cat, dog, or traffic sign type - and a confidence score.

---

## Historical Pipeline: Features + Classifier

Before deep learning dominated:

1. **Hand-crafted features** - SIFT bags-of-words, HOG
2. **Classifier** - SVM, random forest on feature vectors

**Bag of Visual Words (BoVW):**
1. Extract local descriptors from training images
2. Cluster into visual vocabulary (k-means)
3. Represent image as histogram of word counts
4. Train linear SVM

| Era | Representation | Classifier | ImageNet top-5 error |
|-----|----------------|------------|----------------------|
| Pre-2012 | BoVW + SIFT | SVM | ~26% |
| 2012+ | CNN features | Softmax | <3% (ResNet) |

---

## Convolutional Neural Networks

CNNs exploit image structure through three key ideas:

1. **Local connectivity** - neurons connect to small patches
2. **Weight sharing** - same filter slides across image (**translation equivariance**)
3. **Pooling** - subsample for translation tolerance

2D convolution at layer $\ell$:

$$
h_{i,j}^{(\ell)} = \sigma\left( \sum_{m,n} w_{m,n}^{(\ell)} \, h_{i+m,j+n}^{(\ell-1)} + b^{(\ell)} \right)
$$
> **Readable form:** CNNs learn small pattern detectors (edges, textures, parts) and stack them into whole-object recognizers.

### Landmark Architectures

| Model | Year | Key innovation |
|-------|------|----------------|
| LeNet-5 | 1998 | First successful CNN (digits) |
| AlexNet | 2012 | ReLU, dropout, GPU training |
| VGG | 2014 | Deep 3×3 stacks |
| ResNet | 2015 | Skip connections, 152 layers |
| EfficientNet | 2019 | Compound scaling |

**ResNet skip connection:**

$$
\mathbf{h}^{(\ell+1)} = \mathcal{F}(\mathbf{h}^{(\ell)}) + \mathbf{h}^{(\ell)}
$$
> **Readable form:** This loss is the scalar training signal: it is larger when predictions violate the target and smaller when they match.

Enables training very deep networks by easing gradient flow.

```python
import torch
import torch.nn as nn

class SimpleCNN(nn.Chapter):
    def __init__(self, num_classes=10):
        super().__init__()
        self.features = nn.Sequential(
            nn.Conv2d(3, 32, 3, padding=1), nn.ReLU(),
            nn.MaxPool2d(2),
            nn.Conv2d(32, 64, 3, padding=1), nn.ReLU(),
            nn.MaxPool2d(2),
        )
        self.classifier = nn.Linear(64 * 8 * 8, num_classes)

    def forward(self, x):
        x = self.features(x)
        x = x.view(x.size(0), -1)
        return self.classifier(x)
```

---

## Training

**Loss:** cross-entropy for multi-class classification

$$
\mathcal{L} = -\frac{1}{N} \sum_{i=1}^{N} \sum_{c=1}^{C} \mathbb{1}[y_i = c] \log P(y=c \mid \mathbf{x}_i)
$$
> **Readable form:** This loss is the scalar training signal: it is larger when predictions violate the target and smaller when they match.

**Optimization:** SGD with momentum or Adam. **Data augmentation** - random crops, flips, color jitter - reduces overfitting.

| Hyperparameter | Typical range |
|----------------|---------------|
| Learning rate | $10^{-4}$ to $10^{-3}$ |
| Batch size | 32-256 |
| Weight decay | $10^{-4}$ |
| Epochs | 50-200 (with early stopping) |

---

## Transfer Learning

Pretraining on **ImageNet** (1.2M images, 1000 classes) learns general visual features transferable to new tasks with limited data.

Strategies:
1. **Feature extraction** - freeze backbone, train new head
2. **Fine-tuning** - unfreeze some/all layers with small learning rate

When to fine-tune more layers:
- Target domain similar to ImageNet → train classifier only
- Target domain different (medical X-rays) → fine-tune deeper layers

> **Readable form:** Transfer learning is hiring someone who already knows what edges and textures look like - you only teach them your specific categories.

---

## Evaluation Metrics

| Metric | Formula / meaning |
|--------|-------------------|
| Top-1 accuracy | Correct class is highest probability |
| Top-5 accuracy | Correct class in top 5 predictions |
| Confusion matrix | Per-class error patterns |
| F1 score | Harmonic mean of precision and recall (imbalanced classes) |

**Calibration:** predicted probabilities should match empirical accuracy - important for downstream decision systems.

---

## Failure Modes and Dataset Bias

ImageNet-trained models exhibit:
- **Texture bias** - classify by texture not shape (Elephant vs. Cat experiments)
- **Background correlation** - "water" co-occurs with "boat"
- **Demographic bias** - uneven performance across skin tones in face-related tasks

**Mitigation:** diverse training data, bias audits, test sets with controlled distribution shifts.

---

## AIMA Perspective

Russell and Norvig place classification within the **learning agent** architecture. Vision percepts feed into belief states and planning. Classification alone is insufficient for embodied agents - detection and segmentation provide spatial grounding.

Connection to [Chapter 21](../chapter-21-deep-learning/README.md): CNN theory and training dynamics are developed there; this section focuses on vision-specific application.

---

## Deployment Checklist

| Concern | Practice |
|---------|----------|
| Input resolution | Match training size or use multi-scale |
| Normalization | Same mean/std as training |
| Latency | Quantization, TensorRT, MobileNet |
| Monitoring | Track confidence distribution drift |
| Fallback | Human review for low-confidence predictions |

---

## Key Takeaways

1. Image classification maps images to single labels via learned features and softmax probabilities.
2. CNNs use local connectivity, weight sharing, and pooling for efficient visual representation.
3. ResNet skip connections enabled training very deep networks that dominate benchmarks.
4. Transfer learning from ImageNet reduces data needs for new classification tasks.
5. Dataset bias and texture shortcuts cause failures - evaluate on diverse, targeted test sets.

## Additional Notes

This section's extra practice: choose one real image pair, state the visual representation, identify the failure mode, and write the diagnostic you would use before trusting the perception result in an agent loop. Computer vision is useful only when its output is tied to an action, belief update, or safety decision.

---


## Key Terms (Glossary)

- **CNN** - see [../../GLOSSARY.md](../../GLOSSARY.md)
- **ImageNet** - see [../../GLOSSARY.md](../../GLOSSARY.md)
- **transfer learning** - see [../../GLOSSARY.md](../../GLOSSARY.md)

---

## Reflection Questions

1. Train a small CNN on CIFAR-10 and report top-1 accuracy with and without data augmentation.
2. Fine-tune a pretrained ResNet on a 5-class custom dataset with 50 images per class.
3. Plot a confusion matrix and identify the most confused class pairs.
4. Explain why global average pooling reduces parameters vs. flattening conv features.
5. Design a test set to detect background-correlation bias in an animal classifier.

---

**Previous:** [Section 25.3 - Feature Detection](./section-03-feature-detection.md) · **Next:** [Section 25.5 - Object Detection](./section-05-object-detection.md)

---

## References

1. Russell, S., & Norvig, P. (2020). *Artificial Intelligence: A Modern Approach* (4th ed.). [aima.cs.berkeley.edu](https://aima.cs.berkeley.edu/)
2. Russell, S., & Norvig, P. - [aima-python](https://github.com/aimacode/aima-python)
