# Chapter 25: Computer Vision

> **Source:** *Artificial Intelligence: A Modern Approach* (4th ed.), Chapter 25
> **Part:** Part VI: Communicating, Perceiving & Acting
> **Estimated time:** 12–14 hours
> **Prerequisites:** Chapter 21: Deep Learning; Course 1: Chapter 10 (Convolutional Neural Networks)

---

## Chapter Overview

Study image formation, low-level processing, feature extraction, classification, detection, segmentation, and 3D vision. Connect classical pipelines to CNN-based methods and relate to Course 1 CNN chapters with greater theoretical depth.

---

## Learning Objectives

By the end of this chapter, you will be able to:

1. Explain image formation, color spaces, and pinhole camera model
2. Apply filtering, edge detection, and feature descriptors (SIFT/HOG concepts)
3. Train and evaluate image classifiers with CNNs
4. Describe object detection architectures (R-CNN family, YOLO concepts)
5. Explain semantic segmentation with FCN/U-Net overview
6. Introduce stereo vision and structure from motion
7. Analyze vision system failures and dataset bias

---

## Sections

| # | Section | Topics |
|---|--------|--------|
| 1 | [Image Formation](./section-01-image-formation.md) | Cameras, pixels, color, sampling |
| 2 | [Low-Level Vision](./section-02-low-level-vision.md) | Filtering, edges, texture, morphology |
| 3 | [Feature Detection](./section-03-feature-detection.md) | Corners, SIFT, HOG, feature matching |
| 4 | [Image Classification](./section-04-image-classification.md) | CNN architectures, transfer learning |
| 5 | [Object Detection](./section-05-object-detection.md) | Sliding window, R-CNN, anchors, YOLO |
| 6 | [Segmentation](./section-06-segmentation.md) | Semantic vs. instance, FCN, U-Net |
| 7 | [3D Vision](./section-07-3d-vision.md) | Stereo, depth estimation, point clouds overview |
| 8 | [Vision in Practice](./section-08-vision-in-practice.md) | Datasets, augmentation, deployment concerns |

---

## Lab / Project

Fine-tune a pretrained CNN for image classification. Implement a simple edge-detection pipeline and compare features with CNN activations.

---

## Connections to Other Courses

| Topic in this chapter | Where it deepens |
|---------------------|------------------|
| CNN practice | Course 1 Chapter 10: image classification projects |
| Object detection | Course 1 Chapter 12: detection pipelines |
| Generative vision | Course 4 Chapters 02–05: GANs and diffusion for images |

---

## Prerequisites

- Chapter 21: Deep Learning
- Course 1: Chapter 10 (Convolutional Neural Networks)

---

## Self-Assessment

1. How does convolution achieve translation equivariance?
2. Contrast classification, detection, and segmentation tasks.
3. What is the role of anchor boxes in two-stage detectors?
4. Why are pretrained ImageNet weights widely used?
5. Explain parallax in stereo depth estimation.
6. What vision biases can ImageNet-trained models exhibit?

---

**Previous:** [Chapter 24 — Deep Learning for Natural Language Processing](../chapter-24-deep-learning-nlp/README.md)
**Next:** [Chapter 26 — Robotics](../chapter-26-robotics/README.md)
