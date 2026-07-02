# Chapter 21: Deep Learning

> **Source:** *Artificial Intelligence: A Modern Approach* (4th ed.), Chapter 21
> **Part:** Part V: Machine Learning
> **Estimated time:** 14–15 hours
> **Prerequisites:** Chapter 19: Learning from Examples; Course 1: Chapters 08–09 (Neural Networks)

---

## Chapter Overview

Study neural networks from the AIMA perspective: feedforward networks, backpropagation, CNNs for vision, RNNs for sequences, training challenges, and applications. Bridge Course 1 practical experience with formal treatment preparing for Course 3.

---

## Learning Objectives

By the end of this chapter, you will be able to:

1. Represent functions with multilayer feedforward networks and activation functions
2. Derive backpropagation for computing gradients efficiently
3. Design CNN architectures with conv, pooling, and normalization layers
4. Explain RNNs, LSTM gates, and vanishing gradient mitigations
5. Apply regularization: dropout, weight decay, data augmentation
6. Choose optimizers (SGD, momentum, Adam) and learning-rate schedules
7. Evaluate deep models on vision and sequence benchmarks

---

## Sections

| # | Section | Topics |
|---|--------|--------|
| 1 | [Feedforward Networks](./section-01-feedforward-networks.md) | Perceptrons, MLPs, universal approximation |
| 2 | [Backpropagation](./section-02-backpropagation.md) | Computational graphs, chain rule, implementations |
| 3 | [Training Deep Networks](./section-03-training-deep-networks.md) | Initialization, normalization, optimization |
| 4 | [Convolutional Networks](./section-04-convolutional-networks.md) | Convs, pools, ResNet concepts, transfer learning |
| 5 | [Recurrent Networks](./section-05-recurrent-networks.md) | RNN, LSTM, GRU, sequence-to-sequence preview |
| 6 | [Practical Methodology](./section-06-practical-methodology.md) | Data pipelines, GPUs, debugging training |
| 7 | [Applications](./section-07-applications.md) | Image classification, speech, game playing (AlphaZero overview) |
| 8 | [Unsupervised Deep Learning Preview](./section-08-unsupervised-deep-learning-preview.md) | Autoencoders, pretraining history |

---

## Lab / Project

Train a CNN on CIFAR-10 and an LSTM on a character-level text task. Compare training curves with and without dropout; report final metrics.

---

## Connections to Other Courses

| Topic in this chapter | Where it deepens |
|---------------------|------------------|
| Keras/TensorFlow practice | [Course 1 Chapters 08–10](https://github.com/Collaborative-ai/ai-academy-applied-ml-engineering/blob/main/chapters/chapter-08-deep-learning/README.md): applied deep learning |
| Mathematical foundations | [Course 3 Chapters 06–09](https://github.com/Collaborative-ai/ai-academy-deep-learning-foundations/blob/main/chapters/chapter-06-deep-feedforward-networks/README.md): rigorous DL treatment |
| CNN architectures | [Course 3 Chapter 09](https://github.com/Collaborative-ai/ai-academy-deep-learning-foundations/blob/main/chapters/chapter-09-convolutional-networks/README.md): conv nets in depth |
| Sequence modeling | [Course 3 Chapter 10](https://github.com/Collaborative-ai/ai-academy-deep-learning-foundations/blob/main/chapters/chapter-10-sequence-modeling/README.md): RNNs and attention |

---

## Prerequisites

- Chapter 19: Learning from Examples
- Course 1: Chapters 08–09 (Neural Networks)

---

## Self-Assessment

1. Why are nonlinear activation functions necessary in hidden layers?
2. What problem do LSTMs address in standard RNNs?
3. How does dropout act as regularization?
4. Explain the role of convolution weight sharing.
5. What is the vanishing gradient problem?
6. When is transfer learning most beneficial?

---

**Previous:** [Chapter 20 — Learning Probabilistic Models](../chapter-20-learning-probabilistic-models/README.md)
**Next:** [Chapter 22 — Reinforcement Learning](../chapter-22-reinforcement-learning/README.md)
