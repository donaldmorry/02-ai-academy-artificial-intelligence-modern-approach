# Section 21.7: Applications

> **Source:** Russell & Norvig, *Artificial Intelligence: A Modern Approach* (4th ed.), Chapter 21.7  
> **Previous:** [Section 21.6 - Practical Methodology](./section-06-practical-methodology.md)  
> **Vocabulary:** [GLOSSARY.md](../../GLOSSARY.md)  
> **Math conventions:** [MATH_CONVENTIONS.md](../../MATH_CONVENTIONS.md)

---

## Deep Learning in the Real World

Chapter 21.7 surveys **where deep networks deliver state-of-the-art results** - connecting the architectures from Sections 21.1-21.6 to deployed systems. Russell and Norvig treat applications not as a catalog of buzzwords but as **evidence that learned representations scale** across perception, language, and decision-making when data and compute are available.

This section maps AIMA's application overview to your hands-on experience in [Course 1](https://github.com/donaldmorry/01-ai-academy-applied-ml-engineering/blob/main/chapters/chapter-08-deep-learning/README.md) and forward to specialized chapters: [Chapter 25 - Computer Vision](../chapter-25-computer-vision/README.md), [Chapter 23 - NLP](../chapter-23-natural-language-processing/README.md), [Chapter 22 - Reinforcement Learning](../chapter-22-reinforcement-learning/README.md).

> **Readable form:** the same recipe - big data, GPU training, layered representations - keeps winning across domains that look unrelated on the surface.

Humorous analogy: deep learning is a Swiss Army knife that keeps sprouting new tools - each application is a different blade, but the hinge (gradient-based representation learning) is the same.

---

## Image Classification

**Task:** assign one label per image (e.g., "cat," "dog," "airplane").

| Milestone | Result | Significance |
|-----------|--------|--------------|
| AlexNet (2012) | 15.3% top-5 error on ImageNet | CNNs beat hand-crafted features |
| VGG, GoogLeNet (2014) | Deeper, modular designs | Architecture search begins |
| ResNet (2015) | 3.6% top-5 error | 152-layer trains successfully |
| EfficientNet (2019+) | Accuracy vs FLOPs tradeoff | Mobile deployment |

Architecture: CNN backbone ([Section 21.4](./section-04-convolutional-networks.md)) + global pooling + softmax classifier. Training uses augmentation, dropout, and often **transfer learning** from ImageNet pretraining.

```python
# Typical inference pipeline
# image → resize/normalize → CNN → softmax probabilities → argmax label
```

**Deployment contexts:** medical screening (diabetic retinopathy), agriculture (crop disease), manufacturing (defect detection). [Chapter 25](../chapter-25-computer-vision/README.md) extends to geometry, 3-D, and video.

> **Readable form:** show the network a photo; it outputs a probability for each category; pick the highest.

---

## Object Detection and Segmentation (Preview)

Classification assigns one label per image; **detection** localizes multiple objects with bounding boxes; **segmentation** labels every pixel.

| Task | Output | Example architecture family |
|------|--------|----------------------------|
| Detection | Boxes + classes | Faster R-CNN, YOLO |
| Instance seg | Per-object masks | Mask R-CNN |
| Semantic seg | Pixel classes | U-Net, DeepLab |

These build on CNN feature maps with **specialized heads** - covered formally in [Chapter 25](../chapter-25-computer-vision/README.md). AIMA Chapter 21.7 introduces them as natural extensions of conv nets.

---

## Speech Recognition

**Task:** map acoustic signal to text transcript.

Pipeline historically: **feature extraction** (MFCCs) → **acoustic model** → **language model**. Deep learning replaced Gaussian mixtures with **deep neural networks** (2012), then **end-to-end** models:

| Era | Approach |
|-----|----------|
| Classical | HMM + GMM |
| Hybrid | DNN-HMM |
| End-to-end | RNN/LSTM, CTC loss |
| Modern | Transformer (Whisper) |

**CTC (Connectionist Temporal Classification)** aligns variable-length audio with text without frame-level alignment - loss sums over valid alignments.

LSTM and later Transformers ([Chapter 24](../chapter-24-deep-learning-nlp/README.md)) model **temporal dependencies** in spectrograms or raw waveforms ([Section 21.5](./section-05-recurrent-networks.md)).

> **Readable form:** the network listens to a sequence of sound frames and outputs a sequence of words or characters.

---

## Natural Language Processing

Deep NLP progressed: **word embeddings** (Word2Vec, GloVe) → **RNN/LSTM seq2seq** → **Transformers** (2017) → **large language models** (GPT, BERT).

| Task | Deep approach |
|------|---------------|
| Sentiment | Embedding + LSTM or fine-tuned BERT |
| Translation | Encoder-decoder, then Transformer |
| Question answering | Attention over context |
| Summarization | Seq2seq, then pretrained LLMs |

[Course 1 Chapter 04](https://github.com/donaldmorry/01-ai-academy-applied-ml-engineering/blob/main/chapters/chapter-04-text-classification/README.md) used bag-of-words; deep models learn **contextual representations**. [Chapter 23](../chapter-23-natural-language-processing/README.md) covers parsing and semantics; [Chapter 24](../chapter-24-deep-learning-nlp/README.md) focuses on neural NLP.

Russell and Norvig note **hallucination** and **lack of grounding** in pure language models - connecting to [Section 1.4](../chapter-01-introduction/section-04-state-of-the-art.md) on the understanding gap.

---

## Game Playing: AlphaZero Overview

Board games exemplify **planning + learning** integration:

**AlphaGo (2016):** CNN evaluates board positions + Monte Carlo tree search (MCTS) selects moves - defeated Lee Sedol at Go.

**AlphaZero (2017):** Self-play reinforcement learning without human games. A single algorithm mastered chess, shogi, and Go from scratch:

$$
\pi(a \mid s) \leftarrow \text{MCTS guided by neural network } (p, v) = f_\theta(s)
$$
> **Readable form:** The neural network predicts move priors and state value, and MCTS uses them to improve the action policy.

- $p$ - policy (move probabilities)
- $v$ - value (win probability from state $s$)

Network trained to match MCTS improved policy and actual game outcome:

$$
\mathcal{L} = (v - z)^2 - \pi^\top \log p + c \|\theta\|^2
$$
> **Readable form:** the network learns both how good a position is and which moves look promising; MCTS refines both by simulating games.

This connects [Chapter 22 - Reinforcement Learning](../chapter-22-reinforcement-learning/README.md) and [state of the art](../chapter-01-introduction/section-04-state-of-the-art.md) game benchmarks. Not pure supervised deep learning - **RL with deep function approximation**.

---

## Autonomous Systems and Robotics

Self-driving stacks combine:

1. **Perception** - CNNs/LiDAR nets for objects, lanes
2. **Prediction** - RNNs/Transformers for agent trajectories
3. **Planning** - search and optimization ([Chapters 03-05](../chapter-03-solving-problems-by-searching/README.md))
4. **Control** - reinforcement learning or classical controllers

[Chapter 26 - Robotics](../chapter-26-robotics/README.md) integrates these. Deep learning handles messy sensory input; symbolic and probabilistic methods handle safety-critical reasoning.

---

## Scientific Applications

| Domain | Deep learning role |
|--------|-------------------|
| Biology | AlphaFold - protein structure from sequence |
| Chemistry | Property prediction, molecule generation |
| Physics | Surrogate models for simulations |
| Astronomy | Galaxy classification, exoplanet detection |

AlphaFold uses attention over amino acid sequences - **structure prediction as learned spatial reasoning**. Impact: accelerates drug discovery and structural biology.

---

## Recommendation and Tabular Data

Deep learning is not always the winner:

| Data | Often best approach |
|------|---------------------|
| Large-scale user behavior | Deep recommender nets, embeddings |
| Small tabular | Gradient boosting ([Chapter 19](../chapter-19-learning-from-examples/README.md)) |
| Mixed modalities | Multimodal fusion (vision + text) |

AIMA cautions against **defaulting to deep nets** when data is limited and features are already meaningful - rational agents choose tools by expected utility, not hype.

---

## Chapter Lab Connection

The Chapter 21 lab trains:

1. **CNN on CIFAR-10** - 10-class color images ($32 \times 32$)
2. **LSTM on character-level text** - next-character prediction

Compare training curves with/without dropout ([Section 21.3](./section-03-training-deep-networks.md)). Report final accuracy/perplexity and describe **one error analysis insight** per task ([Section 21.6](./section-06-practical-methodology.md)).

---

## Societal Context

Applications raise issues from [Section 1.5](../chapter-01-introduction/section-05-risks-and-benefits-of-ai.md):

- **Bias** in face recognition across demographics
- **Energy** cost of large-scale training
- **Labor** displacement vs augmentation
- **Dual use** - deepfakes, autonomous weapons

Technical capability without [Chapter 27 - Ethics](../chapter-27-philosophy-ethics-safety/README.md) oversight is incomplete agent design.

---

## Connection to Course 1

| Course 1 | AIMA link |
|----------|-----------|
| MNIST digits | [08-digit-recognition-mnist.md](https://github.com/donaldmorry/01-ai-academy-applied-ml-engineering/blob/main/chapters/chapter-03-classification-models/section-08-case-study-mnist-digit-recognition.md): classification precursor |
| CNN chapter | [Chapter 10](https://github.com/donaldmorry/01-ai-academy-applied-ml-engineering/blob/main/chapters/chapter-10-convolutional-neural-networks/README.md): CIFAR-scale extension |
| Text classification | [Chapter 04](https://github.com/donaldmorry/01-ai-academy-applied-ml-engineering/blob/main/chapters/chapter-04-text-classification/README.md): NLP before LSTM |
| Why deep learning | [01-why-deep-learning.md](https://github.com/donaldmorry/01-ai-academy-applied-ml-engineering/blob/main/chapters/chapter-08-deep-learning/section-01-why-deep-learning.md): application motivation |

---

## Connection to Course 3

| AIMA topic | Goodfellow et al. |
|------------|-------------------|
| Large-scale applications | [Chapter 12](https://github.com/donaldmorry/03-ai-academy-deep-learning-foundations/blob/main/chapters/chapter-12-applications/README.md) |
| Computer vision depth | [Chapter 09](https://github.com/donaldmorry/03-ai-academy-deep-learning-foundations/blob/main/chapters/chapter-09-convolutional-networks/README.md) |
| Sequence applications | [Chapter 10](https://github.com/donaldmorry/03-ai-academy-deep-learning-foundations/blob/main/chapters/chapter-10-sequence-modeling/README.md) |
| Deployment at scale | [Chapter 12](https://github.com/donaldmorry/03-ai-academy-deep-learning-foundations/blob/main/chapters/chapter-12-applications/section-01-large-scale-deep-learning.md) |

---

## Reflection Questions

1. What architectural components from this chapter appear in ImageNet-classification systems?
2. How does AlphaZero differ from supervised CNN training on labeled games?
3. When might classical ML outperform deep learning on a business tabular dataset?
4. Name one societal risk specific to a vision application and one to an NLP application.
5. What would you measure to compare CNN vs LSTM lab models fairly?

---

**Previous:** [Section 21.6 - Practical Methodology](./section-06-practical-methodology.md) · **Next:** [Section 21.8 - Unsupervised Deep Learning Preview](./section-08-unsupervised-deep-learning-preview.md)

---

## References

1. Russell, S., & Norvig, P. (2020). *Artificial Intelligence: A Modern Approach* (4th ed.), Ch. 21. [AIMA](https://aima.cs.berkeley.edu/)
2. Russell, S., & Norvig, P. - [aima-python](https://github.com/aimacode/aima-python)
3. Goodfellow, I., Bengio, Y., & Courville, A. (2016). *Deep Learning*. Ch. 12. [deeplearningbook.org](https://www.deeplearningbook.org/)
4. Silver, D. et al. (2017). "Mastering chess and shogi by self-play with a general reinforcement learning algorithm." *arXiv:1712.01815*.
5. Jumper, J. et al. (2021). "Highly accurate protein structure prediction with AlphaFold." *Nature*.
