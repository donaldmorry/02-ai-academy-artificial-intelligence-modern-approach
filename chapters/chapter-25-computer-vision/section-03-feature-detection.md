# Section 25.3: Feature Detection

> **Source:** Russell & Norvig, *Artificial Intelligence: A Modern Approach* (4th ed.), Chapter 25.4
> **Previous:** [Section 25.2 - Low-Level Vision](./section-02-low-level-vision.md)
> **Next:** [Section 25.4 - Image Classification](./section-04-image-classification.md)
> **Vocabulary:** [../../GLOSSARY.md](../../GLOSSARY.md)
> **Math conventions:** [../../MATH_CONVENTIONS.md](../../MATH_CONVENTIONS.md)

---
## From Edges to Distinctive Points

**Feature detection** identifies stable, repeatable locations in images - corners, blobs, and keypoints - that can be described and matched across views. Russell and Norvig position features as the bridge between low-level filtering and high-level recognition, especially for **structure from motion**, **object recognition**, and **image stitching**.

A good feature detector should find points that are:
- **Distinctive** - locally unique appearance
- **Invariant** - stable under viewpoint, scale, illumination changes
- **Localizable** - precise sub-pixel position

> **Readable form:** Features are "landmarks" in a photo - the corner of a window frame, not a blank wall - that you can find again in another picture of the same scene.

---

## Corner Detection

**Corners** are points where intensity changes in multiple directions - unlike edges (one direction) or flat regions (none).

### Harris Corner Detector

The **structure tensor** (second-moment matrix) captures local gradient distribution:

$$
\mathbf{M} = \sum_{(x,y) \in W} \begin{pmatrix} I_x^2 & I_x I_y \\ I_x I_y & I_y^2 \end{pmatrix}
$$
> **Readable form:** The total combines the indexed terms, so each relevant example, state, feature, or dimension contributes once.

**Harris response:**

$$
R = \det(\mathbf{M}) - k \, \mathrm{trace}(\mathbf{M})^2 = \lambda_1 \lambda_2 - k(\lambda_1 + \lambda_2)^2
$$
> **Readable form:** This worked substitution plugs concrete values into the formula so you can verify the arithmetic step by step.

Large $R$ indicates corners ($\lambda_1, \lambda_2$ both large). Edges have one large, one small eigenvalue; flat regions have both small.

| Region type | $\lambda_1, \lambda_2$ | Harris $R$ |
|-------------|------------------------|------------|
| Flat | Both small | Small |
| Edge | One large, one small | Negative |
| Corner | Both large | Large positive |

> **Readable form:** Harris asks "does intensity change in all directions here?" - corners score high, edges score low.

```python
import numpy as np

def harris_response(Ix, Iy, k=0.04, window=5):
    Ixx = Ix * Ix
    Iyy = Iy * Iy
    Ixy = Ix * Iy
  # Sum over window (use box filter or Gaussian weighting)
    Sxx = box_filter(Ixx, window)
    Syy = box_filter(Iyy, window)
    Sxy = box_filter(Ixy, window)
    det = Sxx * Syy - Sxy ** 2
    trace = Sxx + Syy
    return det - k * trace ** 2
```

### Shi-Tomasi (Good Features to Track)

Selects corners where $\min(\lambda_1, \lambda_2) > \tau$ - better for tracking pipelines (Lucas-Kanade optical flow).

---

## Scale-Space and Blob Detection

Objects appear at different scales in images. **Scale-space** analyzes images at increasing Gaussian blur levels.

**Laplacian of Gaussian (LoG) blob detector:** extrema of $\nabla^2 (G_\sigma * I)$ correspond to blob centers at scale $\sigma$.

**Difference of Gaussians (DoG)** approximates LoG efficiently:

$$
\mathrm{DoG}(x,y,\sigma) = G(x,y,k\sigma) * I - G(x,y,\sigma) * I
$$
> **Readable form:** This definition states exactly how the left-hand quantity is computed from the variables on the right.

**SIFT** (Scale-Invariant Feature Transform) uses DoG extrema in scale-space pyramid.

| Detector | Scale invariant | Rotation invariant |
|----------|-----------------|-------------------|
| Harris | No | No |
| SIFT | Yes | Yes (via descriptor) |
| SURF | Yes | Yes (faster approximation) |

---

## SIFT Descriptor

After detecting keypoints with position $(x,y)$, scale $\sigma$, and orientation $\theta$:

1. Take 16×16 patch around keypoint, rotated to canonical orientation
2. Divide into 4×4 subregions
3. Compute 8-bin gradient orientation histogram per subregion
4. Concatenate → 128-dimensional descriptor

Matching uses **Euclidean distance** or **ratio test** (Lowe): accept match if nearest neighbor distance $< 0.8 \times$ second-nearest - rejects ambiguous matches.

> **Readable form:** SIFT summarizes "what the neighborhood looks like" as a 128-number fingerprint robust to small viewpoint changes.

```python
def sift_match_ratio(desc1, desc2, ratio=0.8):
    """desc1: Nx128 query, desc2: Mx128 train."""
    matches = []
    for i, d in enumerate(desc1):
        dists = np.linalg.norm(desc2 - d, axis=1)
        order = np.argsort(dists)
        if dists[order[0]] < ratio * dists[order[1]]:
            matches.append((i, order[0]))
    return matches
```

---

## HOG: Histogram of Oriented Gradients

**HOG** (Dalal & Triggs) describes object shape via gradient orientation histograms in dense cells - foundational for pedestrian detection before deep learning.

Pipeline:
1. Compute gradients $G_x, G_y$
2. Divide image into cells (e.g., 8×8 pixels)
3. Build orientation histogram per cell (9 bins over 0°-180°)
4. Normalize blocks of 2×2 cells (contrast normalization)
5. Concatenate block histograms → feature vector

Used with linear SVM for sliding-window detection. CNNs largely supersede HOG for accuracy but HOG remains interpretable and fast on CPU.

---

## Feature Matching and RANSAC

Given matches between two images, estimate **homography** or **fundamental matrix** despite outliers.

**RANSAC** (Random Sample Consensus):
1. Randomly sample minimal set of matches (4 for homography)
2. Compute model, count inliers
3. Repeat; keep model with most inliers
4. Refit on all inliers

Inlier threshold based on reprojection error:

$$
e_i = \| \mathbf{x}_i' - H \mathbf{x}_i \| < \tau
$$
> **Readable form:** This definition states exactly how the left-hand quantity is computed from the variables on the right.

Applications: panorama stitching, object pose estimation, visual localization.

---

## Modern Learned Features

**SuperPoint, D2-Net, LoFTR** learn detectors and descriptors end-to-end. Advantages:
- Better performance on textureless regions
- Unified training objective

Classical features remain relevant for:
- Embedded systems without GPU
- Interpretable pipelines
- Teaching geometric computer vision

---

## AIMA Perspective

Russell and Norvig connect feature detection to **perception for action** - a robot matching landmarks for localization ([Chapter 26](../chapter-26-robotics/README.md)). Features compress high-dimensional images into sparse correspondences enabling **3D reconstruction** (Section 25.7).

The textbook emphasizes that feature design encodes **invariances** explicitly; CNNs learn invariances implicitly from data.

---

## Evaluation Metrics

| Metric | Meaning |
|--------|---------|
| Repeatability | Same feature found in two views of scene |
| Localization error | Pixel distance from ground truth |
| Matching score | Precision/recall of correspondences |
| Invariance | Performance under rotation, scale, lighting |

Benchmarks: Oxford affine covariant regions, HPatches for learned descriptors.

---

## Key Takeaways

1. Corners (Harris, Shi-Tomasi) are detected where intensity varies in multiple directions.
2. Scale-space blob detection (LoG, DoG) enables scale-invariant keypoints like SIFT.
3. SIFT descriptors provide 128-D signatures robust to viewpoint and illumination changes.
4. HOG encodes gradient orientation histograms for shape-based object detection.
5. RANSAC robustly estimates geometric transforms from noisy feature matches.

## Additional Notes

This section's extra practice: choose one real image pair, state the visual representation, identify the failure mode, and write the diagnostic you would use before trusting the perception result in an agent loop. Computer vision is useful only when its output is tied to an action, belief update, or safety decision.

---


## Key Terms (Glossary)

- **SIFT** - see [../../GLOSSARY.md](../../GLOSSARY.md)
- **HOG** - see [../../GLOSSARY.md](../../GLOSSARY.md)
- **feature matching** - see [../../GLOSSARY.md](../../GLOSSARY.md)

---

## Reflection Questions

1. Implement Harris corner detection and visualize responses on a checkerboard image.
2. Explain why edges are poor features for wide-baseline stereo matching.
3. Describe Lowe's ratio test and when it fails (repeated textures).
4. Compare SIFT vs. ORB for a real-time SLAM application on a mobile robot.
5. Use RANSAC to stitch two overlapping photos into a panorama.

---

**Previous:** [Section 25.2 - Low-Level Vision](./section-02-low-level-vision.md) · **Next:** [Section 25.4 - Image Classification](./section-04-image-classification.md)

---

## References

1. Russell, S., & Norvig, P. (2020). *Artificial Intelligence: A Modern Approach* (4th ed.). [aima.cs.berkeley.edu](https://aima.cs.berkeley.edu/)
2. Russell, S., & Norvig, P. - [aima-python](https://github.com/aimacode/aima-python)
