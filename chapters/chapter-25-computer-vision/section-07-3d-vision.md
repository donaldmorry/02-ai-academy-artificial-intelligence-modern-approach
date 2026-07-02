# Section 25.7: 3D Vision

> **Source:** Russell & Norvig, *Artificial Intelligence: A Modern Approach* (4th ed.), Chapter 25.8
> **Previous:** [Section 25.6 - Segmentation](./section-06-segmentation.md)
> **Next:** [Section 25.8 - Vision in Practice](./section-08-vision-in-practice.md)
> **Vocabulary:** [../../GLOSSARY.md](../../GLOSSARY.md)
> **Math conventions:** [../../MATH_CONVENTIONS.md](../../MATH_CONVENTIONS.md)

---
## Recovering the Third Dimension

Images are 2D projections of 3D scenes. **3D vision** infers depth, shape, and pose from visual data - the inverse of image formation (Section 25.1). Russell and Norvig treat this as essential for robotics, AR/VR, and autonomous navigation.

The problem is **ill-posed**: infinitely many 3D configurations produce the same image. Additional constraints - stereo geometry, motion parallax, known object models, learned priors - resolve ambiguity.

> **Readable form:** 3D vision is guessing how far away things are and what shape they have - your brain does it constantly; computers need math and extra sensors.

---

## Stereo Vision

Two cameras separated by **baseline** $B$ observe the same point. **Disparity** $d = x_l - x_r$ relates to depth $Z$:

$$
Z = \frac{f \cdot B}{d}
$$
> **Readable form:** This definition states exactly how the left-hand quantity is computed from the variables on the right.

where $f$ is focal length. Closer objects have larger disparity.

**Stereo matching** finds corresponding pixels between left and right images:

| Algorithm | Approach |
|-----------|----------|
| Block matching | Sum of squared differences in window |
| Semi-Global Matching (SGM) | Global optimization with smoothness |
| Learned (RAFT-Stereo) | Neural network end-to-end |

**Epipolar constraint:** corresponding point lies on epipolar line - reduces search from 2D to 1D.

```python
def depth_from_disparity(disparity, f, baseline):
    """disparity in pixels; returns depth map."""
    with np.errstate(divide='ignore'):
        Z = np.where(disparity > 0, f * baseline / disparity, np.inf)
    return Z
```

---

## Structure from Motion (SfM)

**SfM** reconstructs 3D structure and camera poses from multiple images of a static scene.

Pipeline:
1. Detect and match features across images
2. Estimate fundamental/essential matrix (two views)
3. Triangulate 3D points
4. Bundle adjustment - jointly optimize 3D points and camera parameters

**Bundle adjustment** minimizes reprojection error:

$$
\min_{\{P_i\}, \{R_j, t_j\}} \sum_{i,j} \| \mathbf{x}_{ij} - \pi(R_j P_i + t_j) \|^2
$$
> **Readable form:** SfM is like building a 3D model from vacation photos - match landmarks, figure out where you stood, triangulate positions.

---

## Depth from Single Image

Monocular depth estimation uses learned priors - humans infer depth from perspective, occlusion, familiar object sizes.

**MiDaS, DPT** predict relative depth maps from single RGB images. Training on diverse datasets with ground truth from LiDAR or stereo.

Limitations: scale ambiguity (absolute meters unknown without calibration).

---

## Point Clouds and Representations

**Point cloud:** set of 3D points $\{(x_i, y_i, z_i)\}$, optionally with color/normal.

| Representation | Properties |
|----------------|------------|
| Point cloud | Unordered, sparse |
| Voxel grid | Regular 3D grid, memory heavy |
| Mesh | Surface connectivity |
| Implicit (NeRF, SDF) | Continuous function |

**PointNet** processes point clouds directly via permutation-invariant operations:

$$
f(\{x_1, \ldots, x_n\}) = \gamma\left(\max_{i} h(x_i)\right)
$$
> **Readable form:** The update moves an estimate toward a reward-informed target using the learning rate.

---

## LiDAR and Sensor Fusion

**LiDAR** measures time-of-flight to surfaces - direct metric depth, sparse (e.g., 64 beams).

Autonomous vehicles fuse:
- **Camera** - rich semantics, no direct depth
- **LiDAR** - accurate 3D geometry
- **Radar** - velocity, weather robust

**Calibration** aligns sensor coordinate frames. **Sensor fusion** networks (BEVFormer, PointPillars) combine modalities in bird's-eye view.

---

## NeRF and Neural Rendering

**Neural Radiance Fields (NeRF)** represent scene as MLP:

$$
F_\theta(\mathbf{x}, \mathbf{d}) \to (\mathbf{c}, \sigma)
$$
> **Readable form:** This formal sentence is the symbolic version of the relationship stated in the surrounding explanation.

Color $\mathbf{c}$ and density $\sigma$ for point $\mathbf{x}$ viewed from direction $\mathbf{d}$. Volume rendering integrates along rays.

Applications: novel view synthesis, virtual tours, digital twins.

---

## Pose Estimation

**6-DoF pose** - 3D rotation + 3D translation of known object model relative to camera.

Methods:
- **PnP** (Perspective-n-Point) - given 3D-2D correspondences
- **Deep learning** - direct regression or render-and-compare

Used in AR (overlay graphics), robotics (grasp planning).

---

## AIMA Perspective

Russell and Norvig connect 3D vision to the **physical agent** loop: perception $\to$ world model $\to$ planning $\to$ action. Metric 3D maps enable collision checking and manipulation ([Chapter 26](../chapter-26-robotics/README.md)).

The textbook stresses **multi-view geometry** as the principled foundation; learning augments but does not replace geometric constraints in safety-critical systems.

---

## Evaluation

| Task | Metric |
|------|--------|
| Depth estimation | AbsRel, RMSE, $\delta < 1.25$ |
| Stereo | Bad pixel %, EPE |
| 3D reconstruction | Chamfer distance, F-score |
| Pose | ADD, 5cm 5° criterion |

---

## Key Takeaways

1. Stereo disparity $Z = fB/d$ recovers depth from calibrated camera pairs.
2. SfM and bundle adjustment reconstruct 3D structure from feature matches across views.
3. Monocular depth uses learned priors but lacks absolute scale without extra cues.
4. Point clouds, meshes, and neural fields represent 3D data for different applications.
5. Sensor fusion combines cameras, LiDAR, and radar for robust 3D perception in robotics.

## Additional Notes

This section's extra practice: choose one real image pair, state the visual representation, identify the failure mode, and write the diagnostic you would use before trusting the perception result in an agent loop. Computer vision is useful only when its output is tied to an action, belief update, or safety decision.

---


## Key Terms (Glossary)

- **stereo** - see [../../GLOSSARY.md](../../GLOSSARY.md)
- **depth estimation** - see [../../GLOSSARY.md](../../GLOSSARY.md)
- **structure from motion** - see [../../GLOSSARY.md](../../GLOSSARY.md)

---

## Reflection Questions

1. Given $f = 500$ px, $B = 0.12$ m, disparity $d = 40$ px, compute depth $Z$.
2. Explain the epipolar constraint and why it speeds up stereo matching.
3. Triangulate a 3D point from two calibrated camera views with known pose.
4. Compare point cloud vs. voxel representation for memory on a 100m × 100m outdoor scene.
5. Describe how NeRF differs from traditional mesh-based 3D reconstruction.

---

**Previous:** [Section 25.6 - Segmentation](./section-06-segmentation.md) · **Next:** [Section 25.8 - Vision in Practice](./section-08-vision-in-practice.md)

---

## References

1. Russell, S., & Norvig, P. (2020). *Artificial Intelligence: A Modern Approach* (4th ed.). [aima.cs.berkeley.edu](https://aima.cs.berkeley.edu/)
2. Russell, S., & Norvig, P. - [aima-python](https://github.com/aimacode/aima-python)

---

## Assessment Practice

Use the shared [Assessment Appendix](../../ASSESSMENT_APPENDIX.md) for concept audits, worked examples, implementation checks, experiment logs, oral-exam prompts, and deliverable checklists.
