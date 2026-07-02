# Section 25.6: Segmentation

> **Source:** Russell & Norvig, *Artificial Intelligence: A Modern Approach* (4th ed.), Chapter 25.7
> **Previous:** [Section 25.5 - Object Detection](./section-05-object-detection.md)
> **Next:** [Section 25.7 - 3D Vision](./section-07-3d-vision.md)
> **Vocabulary:** [../../GLOSSARY.md](../../GLOSSARY.md)
> **Math conventions:** [../../MATH_CONVENTIONS.md](../../MATH_CONVENTIONS.md)

---
## Pixel-Level Understanding

**Segmentation** assigns a label to every pixel in an image. Russell and Norvig distinguish three levels:

| Type | Output | Example |
|------|--------|---------|
| **Semantic** | Class per pixel (no instance ID) | All "road" pixels labeled |
| **Instance** | Separate mask per object instance | Car #1 vs. car #2 |
| **Panoptic** | Semantic + instance for all classes | Unified scene parsing |

> **Readable form:** Segmentation colors every pixel - "this pixel is road, this one is sky, this one is person" - like a paint-by-numbers map of the scene.

---

## Classical Segmentation

### Thresholding and Region Growing

**Otsu's method** chooses threshold $t$ maximizing between-class variance:

$$
\sigma_B^2(t) = \omega_0(t) \omega_1(t) [\mu_0(t) - \mu_1(t)]^2
$$
> **Readable form:** This worked substitution plugs concrete values into the formula so you can verify the arithmetic step by step.

**Region growing** seeds from points and merges similar neighbors by intensity/color criterion.

### Graph-Based Segmentation

**Normalized cuts** partition image graph where nodes are pixels, edges weighted by similarity. Minimizes cut cost relative to segment size.

**GrabCut** interactive segmentation: user draws bounding box; GMM models foreground/background; graph cut optimizes labeling.

---

## Fully Convolutional Networks (FCN)

CNNs for classification output fixed-size vectors. **FCN** replaces fully connected layers with convolutions to produce spatial output maps.

**Upsampling** restores resolution:
- **Transpose convolution** (deconvolution) - learnable upsampling
- **Bilinear upsampling** + convolution
- **Skip connections** from encoder to decoder

**U-Net** architecture (Ronneberger et al.):

```
Encoder: Conv → Pool → ... (contracting path)
Decoder: Upsample → Concat skip → Conv (expanding path)
```

Skip connections preserve fine spatial detail for precise boundaries.

```python
# Simplified U-Net block concept
def unet_block(x, filters):
    skip = conv_block(x, filters)
    down = max_pool(skip)
    return skip, down  # skip saved for decoder concat
```

---

## Loss Functions for Segmentation

**Pixel-wise cross-entropy:**

$$
\mathcal{L} = -\frac{1}{HW} \sum_{i,j} \sum_{c} y_{i,j,c} \log \hat{y}_{i,j,c}
$$
> **Readable form:** This loss is the scalar training signal: it is larger when predictions violate the target and smaller when they match.

**Class imbalance** (few road pixels vs. many sky): use **weighted CE** or **Dice loss**:

$$
\mathcal{L}_{\text{Dice}} = 1 - \frac{2 |P \cap G|}{|P| + |G|}
$$
> **Readable form:** This loss is the scalar training signal: it is larger when predictions violate the target and smaller when they match.

**Focal loss** and **Lovász-Softmax** address hard pixels and direct IoU optimization.

---

## Instance Segmentation

**Mask R-CNN** extends Faster R-CNN:
1. Detect bounding boxes (as before)
2. Add parallel branch predicting $m \times m$ binary mask per region

**ROI Align** (vs. ROI Pool) uses bilinear interpolation - avoids quantization misalignment.

| Model | Approach |
|-------|----------|
| Mask R-CNN | Two-stage, per-ROI masks |
| YOLACT | Parallel prototype masks + coefficients |
| SOLO / SOLOv2 | Direct instance mask prediction |

---

## Evaluation Metrics

| Metric | Use |
|--------|-----|
| Pixel accuracy | $\frac{\text{correct pixels}}{\text{total}}$ - misleading with imbalance |
| Mean IoU (mIoU) | Average IoU per class |
| Boundary F-score | Measures contour quality |
| PQ (Panoptic Quality) | Combines recognition and segmentation |

$$
\text{IoU}_c = \frac{TP_c}{TP_c + FP_c + FN_c}
$$
> **Readable form:** This definition states exactly how the left-hand quantity is computed from the variables on the right.

---

## Medical and Scientific Segmentation

High stakes - tumor volume measurement, cell counting.

Challenges:
- **Limited labeled data** - few expert annotations
- **3D volumes** - extend to 3D U-Net
- **Class imbalance** - small lesions in large scans

**Weak supervision:** train from bounding boxes or scribbles when pixel labels unavailable.

---

## Real-Time Segmentation

Autonomous driving requires 30+ FPS semantic segmentation.

| Model | Trade-off |
|-------|-----------|
| DeepLabV3+ | High accuracy, slower |
| BiSeNet | Bilateral segmentation, fast |
| SegFormer | Transformer encoder, efficient |

**Knowledge distillation** compresses teacher model into fast student.

---

## AIMA Perspective

Russell and Norvig emphasize segmentation for **scene understanding** - converting pixels to symbolic descriptions ("drivable surface," "obstacle"). This feeds high-level planning and supports **affordance reasoning** (where can the robot move?).

Segmentation bridges continuous perception and discrete logic - a theme in neuro-symbolic integration ([Chapter 28](../chapter-28-future-of-ai/README.md)).

---

## Common Failure Modes

| Failure | Cause | Fix |
|---------|-------|-----|
| Blobby boundaries | Low resolution decoder | Skip connections, higher res |
| Class confusion at edges | Ambiguous boundaries | Boundary-aware loss |
| Missing small objects | Downsampling | Multi-scale features, FPN |
| Temporal flicker (video) | Frame-independent | Temporal consistency loss |

---

## Key Takeaways

1. Semantic segmentation labels every pixel; instance segmentation separates individual objects.
2. FCN and U-Net use encoder-decoder architectures with skip connections for dense prediction.
3. Dice and weighted cross-entropy address class imbalance common in segmentation.
4. Mask R-CNN extends object detection with per-instance mask branches.
5. mIoU is the standard metric; panoptic segmentation unifies semantic and instance tasks.

## Additional Notes

This section's extra practice: choose one real image pair, state the visual representation, identify the failure mode, and write the diagnostic you would use before trusting the perception result in an agent loop. Computer vision is useful only when its output is tied to an action, belief update, or safety decision.

---


## Key Terms (Glossary)

- **semantic segmentation** - see [../../GLOSSARY.md](../../GLOSSARY.md)
- **FCN** - see [../../GLOSSARY.md](../../GLOSSARY.md)
- **U-Net** - see [../../GLOSSARY.md](../../GLOSSARY.md)

---

## Reflection Questions

1. Implement Otsu thresholding on a grayscale histology image.
2. Train a small U-Net on a binary segmentation dataset (e.g., nuclei).
3. Compare pixel accuracy vs. mIoU when 90% of pixels are background.
4. Explain ROI Align and why it improves mask quality over ROI Pool.
5. Design a panoptic labeling scheme for a street scene with 10 thing classes and 5 stuff classes.

---

**Previous:** [Section 25.5 - Object Detection](./section-05-object-detection.md) · **Next:** [Section 25.7 - 3D Vision](./section-07-3d-vision.md)

---

## References

1. Russell, S., & Norvig, P. (2020). *Artificial Intelligence: A Modern Approach* (4th ed.). [aima.cs.berkeley.edu](https://aima.cs.berkeley.edu/)
2. Russell, S., & Norvig, P. - [aima-python](https://github.com/aimacode/aima-python)

---

## Assessment Practice

Use the shared [Assessment Appendix](../../ASSESSMENT_APPENDIX.md) for concept audits, worked examples, implementation checks, experiment logs, oral-exam prompts, and deliverable checklists.
