# Section 25.2: Low-Level Vision

> **Source:** Russell & Norvig, *Artificial Intelligence: A Modern Approach* (4th ed.), Chapter 25.3
> **Previous:** [Section 25.1 - Image Formation](./section-01-image-formation.md)
> **Next:** [Section 25.3 - Feature Detection](./section-03-feature-detection.md)
> **Vocabulary:** [../../GLOSSARY.md](../../GLOSSARY.md)
> **Math conventions:** [../../MATH_CONVENTIONS.md](../../MATH_CONVENTIONS.md)

---
## From Pixels to Structure

Low-level vision transforms raw pixel arrays into **structured representations**: smoothed intensities, edges, corners, and texture patterns. These operations are local - each output depends on a neighborhood of inputs - and form the foundation for both classical pipelines and modern CNN early layers.

Russell and Norvig present low-level vision as **front-end processing** before recognition. Even when end-to-end deep networks learn filters, understanding classical operators clarifies what networks might approximate and where hand-crafted priors still help.

> **Readable form:** Low-level vision is like sharpening your glasses before reading a sign - you clean up noise, highlight boundaries, and make structure visible.

---

## Linear Filtering and Convolution

A **linear filter** computes a weighted sum of neighboring pixels. For kernel $h$ of size $(2k+1) \times (2k+1)$:

$$
I'(x, y) = \sum_{u=-k}^{k} \sum_{v=-k}^{k} h(u, v) \, I(x+u, y+v)
$$
> **Readable form:** The total combines the indexed terms, so each relevant example, state, feature, or dimension contributes once.

**Convolution** flips the kernel; **correlation** does not. In deep learning, "convolution" layers typically perform correlation.

| Kernel | Effect |
|--------|--------|
| Box $\frac{1}{9}\mathbf{1}_{3\times3}$ | Blur / noise reduction |
| Gaussian | Smooth with weight falloff |
| Laplacian | Second derivative, edge emphasis |
| Sobel $G_x, G_y$ | First derivative, gradient magnitude |

Gradient magnitude:

$$
|\nabla I| = \sqrt{G_x^2 + G_y^2}
$$
> **Readable form:** Sliding a small stencil over the image - each new pixel is a weighted vote from its neighbors.

```python
import numpy as np

def convolve2d(img, kernel):
    """Naive 2D convolution with edge padding."""
    kh, kw = kernel.shape
    pad_h, pad_w = kh // 2, kw // 2
    padded = np.pad(img, ((pad_h, pad_h), (pad_w, pad_w)), mode='edge')
    out = np.zeros_like(img, dtype=float)
    for i in range(img.shape[0]):
        for j in range(img.shape[1]):
            region = padded[i:i+kh, j:j+kw]
            out[i, j] = np.sum(region * kernel)
    return out

SOBEL_X = np.array([[-1, 0, 1], [-2, 0, 2], [-1, 0, 1]], dtype=float)
SOBEL_Y = SOBEL_X.T
```

---

## Edge Detection

An **edge** is a rapid intensity change, often modeling object boundaries or surface orientation discontinuities.

**Canny edge detector** (multi-stage):
1. Gaussian smoothing
2. Gradient computation
3. Non-maximum suppression along gradient direction
4. Hysteresis thresholding (high/low thresholds)

Marr-Hildreth **LoG** (Laplacian of Gaussian) finds zero-crossings of $\nabla^2 (G_\sigma * I)$.

| Method | Strength | Weakness |
|--------|----------|----------|
| Sobel | Fast, simple | Thick edges, noise-sensitive |
| Canny | Thin, well-localized | Parameter tuning |
| LoG | Theoretically motivated | Scale selection |

> **Readable form:** Edges are where brightness jumps - Canny finds those jumps cleanly by suppressing non-maxima and linking strong candidates.

---

## Nonlinear Filtering

**Median filter** replaces each pixel with the median of its neighborhood - excellent for **salt-and-pepper noise** while preserving edges better than linear blur.

**Bilateral filter** weights by spatial distance and intensity difference:

$$
I'(p) = \frac{1}{W_p} \sum_{q \in \Omega} G_{\sigma_s}(\|p-q\|) \, G_{\sigma_r}(|I(p)-I(q)|) \, I(q)
$$
> **Readable form:** The total combines the indexed terms, so each relevant example, state, feature, or dimension contributes once.

Preserves edges while smoothing flat regions - used in denoising and tone mapping.

---

## Morphological Operations

On binary images, structuring element $B$ defines neighborhood shape.

| Operation | Definition | Effect |
|-----------|------------|--------|
| Erosion | $A \ominus B$ | Shrink foreground |
| Dilation | $A \oplus B$ | Expand foreground |
| Opening | $(A \ominus B) \oplus B$ | Remove small blobs |
| Closing | $(A \oplus B) \ominus B$ | Fill small holes |

**Grayscale morphology** extends min/max over structuring element. Applications: noise removal, skeletonization, boundary extraction.

```python
def binary_erosion(img, se):
    kh, kw = se.shape
    ph, pw = kh // 2, kw // 2
    padded = np.pad(img, ph, mode='constant', constant_values=0)
    out = np.zeros_like(img)
    for i in range(img.shape[0]):
        for j in range(img.shape[1]):
            region = padded[i:i+kh, j:j+kw]
            out[i, j] = 1 if np.all(region[se == 1] >= 1) else 0
    return out
```

---

## Texture Analysis

**Texture** describes spatial repetition of intensity patterns - grass, brick, fabric.

Classical approaches:
- **Co-occurrence matrices** $P_{d,\theta}(i,j)$ - frequency of intensity pairs at offset $d$, angle $\theta$
- **Gabor filters** - oriented bandpass; model simple-cell responses
- **Local Binary Patterns (LBP)** - compare center pixel to neighbors

Gabor filter in spatial domain:

g(x,y;\lambda,\theta) = \exp\left(-\frac{x'^2 + \gamma^2 y'^2}{2\sigma^2}\right) \cos\left(2\pi \frac{x'}{\lambda}\right)
$$

where $x' = x \cos\theta + y \sin\theta$.

$$
> **Readable form:** Texture is "what the neighborhood pattern looks like" - repeating dots, lines, or random grain.

---

## Frequency Domain View

Convolution theorem: $\mathcal{F}(h * I) = \mathcal{F}(h) \cdot \mathcal{F}(I)$.

Large kernels convolve faster via FFT for big images. **High-pass** filters emphasize edges; **low-pass** remove fine detail. Understanding frequency helps explain **aliasing** (from Section 25.1) and design anti-aliasing filters.

---

## Histogram and Intensity Transforms

**Histogram equalization** spreads intensity distribution for better contrast:

$$
s = T(r) = (L-1) \int_0^r p_r(w) \, dw
$$
> **Readable form:** This worked substitution plugs concrete values into the formula so you can verify the arithmetic step by step.

**Gamma correction** $V_{\text{out}} = V_{\text{in}}^\gamma$ compensates for display nonlinearity. **CLAHE** (contrast-limited adaptive histogram equalization) applies equalization locally with clipping to avoid noise amplification.

| Transform | Purpose |
|-----------|---------|
| Linear stretch | Map min/max to full range |
| Log transform | Compress bright dynamic range |
| Thresholding | Binarize for morphology |

---

## Connection to CNNs

First layers of CNNs often learn edge-like and blob-like filters resembling Gabor and Sobel. Differences:
- CNN filters are **learned** from data
- Multiple channels capture diverse orientations/scales
- Stacking builds hierarchical features

Transfer learning assumes low-level features (edges, textures) transfer across domains.

---

## AIMA Perspective

Chapter 25 treats low-level vision as building **invariance to nuisance factors** - illumination changes should not destroy edge structure. Retinex theory and normalization address lighting variation.

Russell and Norvig note that no single low-level representation suffices for all tasks - pipelines chain operations based on downstream needs. Medical imaging may emphasize contrast enhancement; robotics may prioritize real-time edge maps for obstacle detection.

---

## Deployment Considerations

| Issue | Low-level symptom | Mitigation |
|-------|-------------------|------------|
| Motion blur | Smeared edges | Shorter exposure, deconvolution |
| Low light | Noisy gradients | Bilateral/median pre-filter |
| JPEG artifacts | Blocking at 8×8 blocks | Deblocking filters, train on compressed data |
| HDR scenes | Clipped highlights | Tone mapping before edge detection |

Preprocessing choices affect all downstream vision chapters - document them in model cards.

---

## Key Takeaways

1. Linear filtering computes weighted sums; convolution vs. correlation differs by kernel flip.
2. Edge detectors (Canny, Sobel) locate intensity discontinuities; parameters affect noise sensitivity.
3. Median and bilateral filters handle noise while preserving edges better than Gaussian blur alone.
4. Morphological operations manipulate binary/shape structure via erosion, dilation, opening, closing.
5. Texture descriptors and CNN early layers extract local patterns for higher-level recognition.

## Key Terms (Glossary)

- **filtering** - see [../../GLOSSARY.md](../../GLOSSARY.md)
- **edge detection** - see [../../GLOSSARY.md](../../GLOSSARY.md)
- **morphology** - see [../../GLOSSARY.md](../../GLOSSARY.md)

---

## Reflection Questions

1. Apply Sobel filters to a grayscale image and visualize gradient magnitude.
2. Explain why median filtering removes salt-and-pepper noise but blurs less than a mean filter.
3. Design a morphological opening to remove pepper noise from a binary fingerprint image.
4. Compare co-occurrence matrix features vs. LBP for brick vs. grass texture classification.
5. Show that convolution in spatial domain equals multiplication in frequency domain for a 1D example.

---

**Previous:** [Section 25.1 - Image Formation](./section-01-image-formation.md) · **Next:** [Section 25.3 - Feature Detection](./section-03-feature-detection.md)

---

## References

1. Russell, S., & Norvig, P. (2020). *Artificial Intelligence: A Modern Approach* (4th ed.). [aima.cs.berkeley.edu](https://aima.cs.berkeley.edu/)
2. Russell, S., & Norvig, P. - [aima-python](https://github.com/aimacode/aima-python)


