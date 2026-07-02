# Section 21.4: Convolutional Networks

> **Source:** Russell & Norvig, *Artificial Intelligence: A Modern Approach* (4th ed.), Chapter 21.4  
> **Previous:** [Section 21.3 - Training Deep Networks](./section-03-training-deep-networks.md)  
> **Vocabulary:** [GLOSSARY.md](../../GLOSSARY.md)  
> **Math conventions:** [MATH_CONVENTIONS.md](../../MATH_CONVENTIONS.md)

---

## Why Convolutions for Vision

Fully connected layers on $224 \times 224 \times 3$ RGB images require **millions of weights per layer** and ignore spatial structure - pixel $(i,j)$ is treated independently of its neighbors. **Convolutional neural networks (CNNs)** exploit two inductive biases of natural images:

1. **Locality** - nearby pixels are more correlated than distant ones.
2. **Stationarity** - the same feature detectors (edges, textures) appear across the image.

Russell and Norvig present CNNs as the architecture that made deep learning dominate [computer vision](../chapter-25-computer-vision/README.md) after AlexNet (2012). You may have trained CNNs in [Course 1 Chapter 10](https://github.com/Collaborative-ai/ai-academy-applied-ml-engineering/blob/main/chapters/chapter-10-convolutional-neural-networks/README.md); this section explains **why** they work from the AIMA perspective.

> **Readable form:** instead of connecting every pixel to every neuron, slide small filters across the image and reuse the same filter everywhere.

Humorous analogy: a fully connected layer is a security guard who memorizes every square inch of a stadium separately; a conv layer is one guard with a magnifying glass who scans row by row - same skill, every seat.

---

## The Convolution Operation

A **2-D convolution** (technically cross-correlation in DL frameworks) applies a learnable **kernel** (filter) $\mathbf{K}$ of size $k \times k$ across input feature map $\mathbf{X}$:

$$
(\mathbf{K} * \mathbf{X})_{i,j} = \sum_{u=0}^{k-1} \sum_{v=0}^{k-1} K_{u,v} \cdot X_{i+u,\, j+v}
$$
> **Readable form:** at each output position, multiply overlapping kernel weights with input patch and sum.

**Hyperparameters:**

| Parameter | Effect |
|-----------|--------|
| **Stride** $s$ | Step size between filter positions; downsamples output |
| **Padding** $p$ | Zero-pad border to control output spatial size |
| **Kernel size** $k$ | Receptive field per output pixel |

Output spatial dimensions (single channel):

H_{\text{out}} = \left\lfloor \frac{H_{\text{in}} + 2p - k}{s} \right\rfloor + 1, \quad
W_{\text{out}} = \left\lfloor \frac{W_{\text{in}} + 2p - k}{s} \right\rfloor + 1
> **Readable form:** output height equals input plus padding minus kernel, divided by stride, plus one.

A conv layer with $C_{\text{in}}$ input channels and $C_{\text{out}}$ filters has kernel tensor shape $(k, k, C_{\text{in}}, C_{\text{out}})$ plus biases.

```python
# Conceptual: 3x3 edge detector (Sobel-like, fixed - learned in practice)
edge_kernel = np.array([[-1, -1, -1],
                        [ 0,  0,  0],
                        [ 1,  1,  1]])
```

---

## Weight Sharing and Parameter Efficiency

In a dense layer connecting $224 \times 224 \times 3$ inputs to 256 units:

$$
|\theta| = 224 \times 224 \times 3 \times 256 \approx 38\text{M}
$$
> **Readable form:** The operation slides a local window over the input and computes a feature value for each output location.

A $3 \times 3$ conv layer with 256 filters:

|\theta| = 3 \times 3 \times 3 \times 256 + 256 \approx 7\text{K}
$$

**Weight sharing** - the same kernel applied at every spatial location - is the key. It encodes **translation equivariance**: if the input shifts, the feature map shifts correspondingly (before pooling).

$$
> **Readable form:** one small filter scanned everywhere replaces millions of independent pixel-to-neuron weights.

This connects to AIMA's theme of **inductive bias**: the architecture constrains the hypothesis space toward solutions that generalize on spatial data.

---

## Pooling Layers

**Pooling** downsamples feature maps, reducing spatial resolution and computation:

| Type | Operation | Typical use |
|------|-----------|-------------|
| **Max pool** | $\max$ over $p \times p$ window | Most common; sharp features |
| **Average pool** | Mean over window | Smoother downsampling |
| **Global average pool** | Average entire map | Replace large FC layers at end |

Max pooling with stride 2 halves spatial dimensions. It provides **translation invariance** - small shifts do not change pooled output drastically.

$$
y_{i,j} = \max_{(u,v) \in \mathcal{R}_{i,j}} x_{u,v}
$$
> **Readable form:** each output cell keeps the strongest activation in its neighborhood.

Modern architectures sometimes **replace pooling with strided convolutions** - learnable downsampling. ResNet and EfficientNet variants use this pattern.

---

## Typical CNN Block

A canonical building block stacks:

```
Conv → BatchNorm → ReLU → [Pool or strided Conv]
```

Stacking blocks increases **receptive field** - the region of input influencing each output neuron. Three $3 \times 3$ conv layers equal one $7 \times 7$ receptive field but with **fewer parameters** and **more nonlinearities**.

| Layer depth | Receptive field grows |
|-------------|----------------------|
| Early | Edges, colors, textures |
| Middle | Parts (eyes, wheels) |
| Late | Whole objects, scenes |

[Course 3 Chapter 09](https://github.com/Collaborative-ai/ai-academy-deep-learning-foundations/blob/main/chapters/chapter-09-convolutional-networks/README.md) derives receptive field arithmetic and classic architectures in full detail.

---

## Classic Architectures (AIMA Overview)

| Network | Year | Key idea |
|---------|------|----------|
| **LeNet-5** | 1998 | Conv + pool + FC; MNIST digits |
| **AlexNet** | 2012 | ReLU, dropout, GPU training; ImageNet breakthrough |
| **VGG** | 2014 | Deep stacks of $3 \times 3$ convs |
| **ResNet** | 2015 | Skip connections; train 100+ layers |

### AlexNet

Eight layers (5 conv, 3 FC), ReLU activations, dropout, data augmentation, trained on two GPUs. Demonstrated that **depth + scale** beats hand-crafted features on ImageNet.

### VGG

Showed that **depth with small filters** (3×3 throughout) outperforms larger kernels. Simple, uniform design - easy to reason about but parameter-heavy in FC layers.

---

## ResNet and Skip Connections

Very deep plain CNNs suffer **degradation** - training error *increases* with depth beyond a point, not merely overfitting. **Residual networks** (He et al., 2015) learn **residual mappings**:

$$
\mathbf{h}^{(\ell+1)} = \mathbf{h}^{(\ell)} + \mathcal{F}^{(\ell)}\!\left(\mathbf{h}^{(\ell)}\right)
$$
> **Readable form:** layer output equals input plus a learned correction - skip connection carries identity forward.

Benefits:

1. **Gradient highway** - backprop can flow directly through additions, mitigating vanishing gradients ([Section 21.2](./section-02-backpropagation.md)).
2. **Easier optimization** - learning small residuals is simpler than full mappings.
3. **Ensemble-like behavior** - effective paths vary during training.

```python
# Residual block (conceptual)
def res_block(x, conv_fn):
    residual = conv_fn(x)  # Conv-BN-ReLU-Conv-BN
    return tf.nn.relu(x + residual)
```

ResNet-50, ResNet-101 remain backbone architectures for [Chapter 25](../chapter-25-computer-vision/README.md) and transfer learning.

---

## Normalization in CNNs

**Batch normalization** ([Section 21.3](./section-03-training-deep-networks.md)) after conv layers stabilizes training. **Layer normalization** appears in later architectures (Transformers). For CNNs, per-channel normalization variants (Group Norm) help small batch sizes common in detection/segmentation.

---

## Transfer Learning

Training ImageNet-scale CNNs requires massive compute. **Transfer learning** reuses pretrained weights:

1. **Feature extraction** - freeze conv backbone, train new classifier head.
2. **Fine-tuning** - unfreeze top layers (or all) with small learning rate.

Beneficial when target dataset is **small** and **similar** to pretraining domain (natural photos). [Course 1 Chapter 10](https://github.com/Collaborative-ai/ai-academy-applied-ml-engineering/blob/main/chapters/chapter-10-convolutional-neural-networks/section-01-why-cnns-for-images.md) motivates this practically; AIMA connects it to **knowledge reuse in learning agents**.

> **Readable form:** borrow eyes trained on millions of photos, then teach a new task-specific decision on top.

---

## Equivariance and Invariance Summary

| Property | Mechanism | Benefit |
|----------|-----------|---------|
| Translation equivariance | Weight sharing in conv | Detect features anywhere |
| Translation invariance | Pooling / global pool | Classify regardless of position |
| Scale invariance (partial) | Multi-scale training, augmentation | Robustness to object size |

Fully achieving scale and rotation invariance requires **data augmentation** or **architecture extensions** (dilated conv, spatial transformer networks - beyond AIMA core).

---

## CNN vs Fully Connected on Images

| Aspect | Dense MLP | CNN |
|--------|-----------|-----|
| Parameters | Enormous | Compact via sharing |
| Spatial structure | Ignored | Exploited |
| Data needed | More | Less (with good bias) |
| Interpretability | Low | Filter visualizations possible |

For MNIST ($28 \times 28$), a small MLP can reach high accuracy - CNN advantages grow with image resolution and task complexity.

---

## Pitfalls

1. **Wrong input shape** - channels-last vs channels-first conventions.
2. **Forgetting to normalize inputs** - ImageNet mean/std when using pretrained weights.
3. **Excessive pooling** - destroys spatial information needed for localization.
4. **Training from scratch on tiny data** - use transfer learning instead.

---

## Connection to Course 1

| Course 1 | AIMA link |
|----------|-----------|
| Why CNNs for images | [01-why-cnns-for-images.md](https://github.com/Collaborative-ai/ai-academy-applied-ml-engineering/blob/main/chapters/chapter-10-convolutional-neural-networks/section-01-why-cnns-for-images.md): practical motivation |
| Chapter 10 overview | [README.md](https://github.com/Collaborative-ai/ai-academy-applied-ml-engineering/blob/main/chapters/chapter-10-convolutional-neural-networks/README.md): Keras Conv2D, MaxPooling |
| MNIST baseline | [08-digit-recognition-mnist.md](https://github.com/Collaborative-ai/ai-academy-applied-ml-engineering/blob/main/chapters/chapter-03-classification-models/section-08-case-study-mnist-digit-recognition.md): MLP vs conv comparison |
| Chapter 08 deep learning | [01-why-deep-learning.md](https://github.com/Collaborative-ai/ai-academy-applied-ml-engineering/blob/main/chapters/chapter-08-deep-learning/section-01-why-deep-learning.md): ImageNet narrative |

---

## Connection to Course 3

| AIMA topic | Goodfellow et al. |
|------------|-------------------|
| Convolution operation | [Chapter 09](https://github.com/Collaborative-ai/ai-academy-deep-learning-foundations/blob/main/chapters/chapter-09-convolutional-networks/README.md) §9.1 |
| Pooling | [Chapter 09](https://github.com/Collaborative-ai/ai-academy-deep-learning-foundations/blob/main/chapters/chapter-09-convolutional-networks/README.md) §9.2 |
| Weight sharing & equivariance | [Chapter 09](https://github.com/Collaborative-ai/ai-academy-deep-learning-foundations/blob/main/chapters/chapter-09-convolutional-networks/README.md) §9.4 |
| ResNet & classic architectures | [Chapter 09](https://github.com/Collaborative-ai/ai-academy-deep-learning-foundations/blob/main/chapters/chapter-09-convolutional-networks/README.md) §9.6 |
| Transfer learning | [Chapter 09](https://github.com/Collaborative-ai/ai-academy-deep-learning-foundations/blob/main/chapters/chapter-09-convolutional-networks/README.md) §9.9 |

---

## Reflection Questions

1. How does weight sharing reduce parameters and encode an inductive bias?
2. Derive output size for a $32 \times 32$ input, $5 \times 5$ kernel, stride 1, padding 2.
3. Why do ResNet skip connections ease training of very deep networks?
4. When is transfer learning most beneficial, and when might it hurt?
5. What is the difference between translation equivariance and translation invariance?

---

**Previous:** [Section 21.3 - Training Deep Networks](./section-03-training-deep-networks.md) · **Next:** [Section 21.5 - Recurrent Networks](./section-05-recurrent-networks.md)

---

## References

1. Russell, S., & Norvig, P. (2020). *Artificial Intelligence: A Modern Approach* (4th ed.), Ch. 21. [AIMA](https://aima.cs.berkeley.edu/)
2. Russell, S., & Norvig, P. - [aima-python](https://github.com/aimacode/aima-python)
3. Goodfellow, I., Bengio, Y., & Courville, A. (2016). *Deep Learning*. Ch. 9. [deeplearningbook.org](https://www.deeplearningbook.org/)
4. Krizhevsky, A., Sutskever, I., & Hinton, G. (2012). "ImageNet classification with deep convolutional neural networks." *NeurIPS*.
5. He, K. et al. (2016). "Deep residual learning for image recognition." *CVPR*.

