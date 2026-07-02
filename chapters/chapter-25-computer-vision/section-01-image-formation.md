# Section 25.1: Image Formation

> **Source:** Russell & Norvig, *Artificial Intelligence: A Modern Approach* (4th ed.), Chapter 25.1-25.2
> **Previous:** [Chapter 24 - Deep Learning NLP](../chapter-24-deep-learning-nlp/README.md)
> **Next:** [Section 25.2 - Low-Level Vision](./section-02-low-level-vision.md)
> **Prerequisites:** [Chapter 21 - Deep Learning](../chapter-21-deep-learning/README.md)
> **Vocabulary:** [../../GLOSSARY.md](../../GLOSSARY.md)
> **Math conventions:** [../../MATH_CONVENTIONS.md](../../MATH_CONVENTIONS.md)

---
## From Light to Pixels

Every computer vision pipeline begins with a physical question: **how does light from the world become numbers in memory?** Russell and Norvig treat vision as inverse graphics - we observe an image and infer properties of the scene (shape, material, motion) that produced it.

A digital image is a **2D array of measurements**. Each pixel stores intensity or color channels. Understanding formation is prerequisite to every downstream task: classification, detection, segmentation, and 3D reconstruction.

> **Readable form:** A camera is a light bucket with rules. Photons hit a sensor; electronics convert them to numbers arranged in a grid. Everything else in vision is interpretation of that grid.

---

## The Pinhole Camera Model

The simplest geometric model is the **pinhole camera**. Light rays pass through a single point (the optical center) and project onto an image plane.

For a 3D point $\mathbf{X} = (X, Y, Z)^\top$ in camera coordinates, the projected image point $(x, y)$ satisfies:

$$
x = f \frac{X}{Z}, \quad y = f \frac{Y}{Z}
$$
> **Readable form:** This definition states exactly how the left-hand quantity is computed from the variables on the right.

where $f$ is the **focal length** (distance from pinhole to image plane).

In homogeneous coordinates, projection is a linear map followed by division:

$$
\lambda \begin{pmatrix} x \\ y \\ 1 \end{pmatrix} =
\begin{pmatrix} f & 0 & 0 & 0 \\ 0 & f & 0 & 0 \\ 0 & 0 & 1 & 0 \end{pmatrix}
\begin{pmatrix} X \\ Y \\ Z \\ 1 \end{pmatrix}
$$

> **Readable form:** Things farther away ($Z$ larger) appear smaller on the image - that's perspective. Double the distance, halve the size on the sensor.

### Real Cameras vs. Pinhole

Real lenses introduce **distortion** (barrel/pincushion), **chromatic aberration**, and **depth of field**. Calibration estimates intrinsic parameters:

$$
\mathbf{K} = \begin{pmatrix} f_x & s & c_x \\ 0 & f_y & c_y \\ 0 & 0 & 1 \end{pmatrix}
$$
> **Readable form:** This worked substitution plugs concrete values into the formula so you can verify the arithmetic step by step.

- $f_x, f_y$ - focal lengths in pixels (may differ if pixels are non-square)
- $(c_x, c_y)$ - principal point (optical axis intersection)
- $s$ - skew (usually 0)

Extrinsic parameters $(\mathbf{R}, \mathbf{t})$ map world coordinates to camera frame: $\mathbf{X}_c = \mathbf{R}\mathbf{X}_w + \mathbf{t}$.

```python
import numpy as np

def project_point(X, K, R, t):
    """Project 3D world point to 2D pixel coordinates."""
    Xc = R @ X + t
    if Xc[2] <= 0:
        return None  # behind camera
    x_hom = K @ Xc
    return x_hom[:2] / x_hom[2]
```

---

## Sampling and Quantization

**Sampling** chooses discrete locations on the image plane; **quantization** maps continuous intensity to finite bit depths.

| Stage | What happens | Failure mode |
|-------|--------------|--------------|
| Sampling | Grid of photodetectors | Aliasing (moiré, jaggies) |
| Quantization | ADC maps voltage to levels | Banding, posterization |
| Compression | JPEG, HEVC discard information | Blocking artifacts |

The **Nyquist theorem** requires sampling rate at least twice the highest spatial frequency. Undersampling high-frequency textures (fine fabric, distant fences) causes **aliasing**.

**Anti-aliasing** blurs before downsampling - trading sharpness for correctness:

$$
I_{\text{down}}(x, y) = \sum_{u,v} I(u, v) \, h(x-u, y-v)
$$
> **Readable form:** Downsampled pixel intensity is a weighted blend of nearby original pixels using a low-pass filter kernel.

where $h$ is a low-pass filter kernel.

> **Readable form:** If your pixel grid is too coarse for the detail in the scene, the camera invents fake patterns - like stripes on a TV when someone wears a tight herringbone suit.

---

## Color Spaces

Human color perception is **trichromatic** - three cone types (L, M, S). Digital cameras mimic this with RGB filters.

| Space | Channels | Use case |
|-------|----------|----------|
| RGB | Red, Green, Blue | Display, neural nets |
| sRGB | Gamma-corrected RGB | Web images |
| HSV/HSL | Hue, Saturation, Value | Segmentation heuristics |
| CIELAB | L*, a*, b* | Perceptually uniform distance |
| YCbCr | Luma + chroma | Video compression |

RGB is **not perceptually uniform**: equal Euclidean distance in RGB does not match perceived color difference. CIELAB approximates human judgment better.

Conversion RGB $\to$ grayscale (luminance):

$$
Y = 0.299 R + 0.587 G + 0.114 B
$$
> **Readable form:** This worked substitution plugs concrete values into the formula so you can verify the arithmetic step by step.

**White balance** corrects illumination color temperature - critical for outdoor/indoor mixed scenes.

```python
def rgb_to_gray(img):
    """img: HxWx3 float array in [0,1]."""
    w = np.array([0.299, 0.587, 0.114])
    return img @ w
```

---

## Image as Function

Formally, a continuous image is $I(x, y): \mathbb{R}^2 \to \mathbb{R}^k$ where $k=1$ (gray) or $k=3$ (color).

Discrete image: $I[i, j]$ for $i \in \{0,\ldots,H-1\}$, $j \in \{0,\ldots,W-1\}$.

**Spatial resolution** - pixels per unit length (DPI, PPI).  
**Dynamic range** - ratio of brightest to darkest distinguishable intensity. HDR imaging captures wide range; tone mapping compresses for display.

### Radiometry Preview

**Irradiance** $E$ (W/m²) at a surface depends on scene radiance $L$, geometry, and optics. The rendering equation (simplified):

$$
L_o(\mathbf{x}, \omega_o) = L_e(\mathbf{x}, \omega_o) + \int_\Omega f_r(\mathbf{x}, \omega_i, \omega_o) L_i(\mathbf{x}, \omega_i) (\mathbf{n} \cdot \omega_i) \, d\omega_i
$$
> **Readable form:** This definition states exactly how the left-hand quantity is computed from the variables on the right.

Vision inverts this: from $L_o$ measurements, infer shape, albedo, lighting.

---

## Field of View and Resolution Trade-offs

Horizontal field of view (FOV):

$$
\text{FOV}_h = 2 \arctan\left(\frac{w}{2f}\right)
$$
> **Readable form:** This worked substitution plugs concrete values into the formula so you can verify the arithmetic step by step.

Wide FOV captures more scene but reduces angular resolution per pixel. Telephoto lenses narrow FOV but magnify distant objects.

| Application | Typical FOV | Resolution |
|-------------|-------------|------------|
| Smartphone camera | 60-80° | 12 MP |
| Automotive front cam | 50-120° | 2-8 MP |
| Surveillance | 90-110° | 2-4 MP |
| Microscopy | <1° | Specialized sensors |

---

## Noise Models

Sensor noise combines **photon shot noise** (Poisson), **read noise** (Gaussian), and **fixed pattern noise**.

A common approximation:

$$
I_{\text{obs}} = I_{\text{true}} + \mathcal{N}(0, \sigma^2)
$$
> **Readable form:** This worked substitution plugs concrete values into the formula so you can verify the arithmetic step by step.

Poisson noise variance equals signal mean - brighter regions are noisier in relative terms but often cleaner in absolute SNR after sqrt scaling.

**Signal-to-noise ratio (SNR):**

$$
\text{SNR} = 10 \log_{10} \frac{P_{\text{signal}}}{P_{\text{noise}}} \text{ dB}
$$
> **Readable form:** Signal-to-noise ratio in decibels is ten times the log of signal power divided by noise power.

Low-light vision (night driving, astronomy) requires denoising, longer exposure, or larger apertures.

> **Readable form:** Dark photos look grainy because each pixel gets fewer photons - counting statistics get rougher, like guessing crowd size from a blurry photo.

---

## Connecting to Deep Learning

Modern CNNs learn features directly from RGB pixels, but formation physics still matters:

- **Lens distortion** uncorrected → systematic localization errors
- **Color space mismatch** train sRGB, deploy RAW → performance drop
- **Resolution mismatch** → scale-sensitive detectors fail
- **Rolling shutter** on moving scenes → geometric warping

Preprocessing pipelines often include: debayering, undistortion, normalization $\mu=0, \sigma=1$ per channel.

---

## AIMA Perspective

Chapter 25 positions vision within the **perceiving agent** architecture: sensors produce percepts; vision algorithms extract symbolic or geometric descriptions for planning and action ([Chapter 26](../chapter-26-robotics/README.md)). Image formation is the **interface between physics and computation**.

Russell and Norvig emphasize that vision is **underconstrained** - infinitely many 3D scenes could produce the same 2D image. Priors (smooth surfaces, familiar objects) and multiple views resolve ambiguity.

---

## Key Takeaways

1. The pinhole model relates 3D points to 2D via perspective division by depth $Z$.
2. Sampling and quantization introduce aliasing and banding - understand before blaming the neural net.
3. Color spaces serve different purposes; RGB is convenient but not perceptually uniform.
4. Camera calibration (intrinsics/extrinsics) is essential for metric 3D vision.
5. Noise and illumination dominate failure modes in deployment, not just model architecture.

## Key Terms (Glossary)

- **pinhole camera** - see [../../GLOSSARY.md](../../GLOSSARY.md)
- **pixel** - see [../../GLOSSARY.md](../../GLOSSARY.md)
- **color space** - see [../../GLOSSARY.md](../../GLOSSARY.md)

---

## Reflection Questions

1. Derive the image coordinates for a 3D point at $(1, 2, 5)$ with focal length $f=50$ mm.
2. Explain why downsampling without low-pass filtering causes aliasing.
3. Compare when HSV vs. CIELAB is preferable for a color-based segmentation task.
4. Estimate horizontal FOV for a sensor 36 mm wide with $f = 24$ mm.
5. List three preprocessing steps you would apply before feeding dashcam footage to a detector.

---

**Previous:** [Chapter 24 - Deep Learning NLP](../chapter-24-deep-learning-nlp/README.md) · **Next:** [Section 25.2 - Low-Level Vision](./section-02-low-level-vision.md)

---

## References

1. Russell, S., & Norvig, P. (2020). *Artificial Intelligence: A Modern Approach* (4th ed.). [aima.cs.berkeley.edu](https://aima.cs.berkeley.edu/)
2. Russell, S., & Norvig, P. - [aima-python](https://github.com/aimacode/aima-python)
