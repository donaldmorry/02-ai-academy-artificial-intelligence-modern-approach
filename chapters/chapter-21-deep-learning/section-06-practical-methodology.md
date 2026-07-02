# Section 21.6: Practical Methodology

> **Source:** Russell & Norvig, *Artificial Intelligence: A Modern Approach* (4th ed.), Chapter 21.6  
> **Previous:** [Section 21.5 - Recurrent Networks](./section-05-recurrent-networks.md)  
> **Vocabulary:** [GLOSSARY.md](../../GLOSSARY.md)  
> **Math conventions:** [MATH_CONVENTIONS.md](../../MATH_CONVENTIONS.md)

---

## Theory Meets Engineering

Sections 21.1-21.5 covered architectures and training algorithms. Chapter 21.6 addresses what separates **reproducible deep learning projects** from frustrating trial-and-error: systematic methodology for data, baselines, debugging, and evaluation.

Russell and Norvig frame methodology as part of **rational agent design** - a learning agent must not only have a hypothesis class but a **disciplined process** for improving performance under time and compute constraints. This parallels [Chapter 19](../chapter-19-learning-from-examples/README.md) cross-validation but adds deep-learning-specific pitfalls (GPU memory, days-long training runs, opaque failures).

> **Readable form:** good methodology means knowing what to measure, what to try first, and how to tell whether a change actually helped.

Humorous analogy: training without methodology is cooking by throwing random spices until something tastes okay once - you cannot reproduce dinner, and guests (your test set) get food poisoning from overfitting.

---

## End-to-End Workflow

```
Problem definition → Data collection → Baseline → Model → Tune → Error analysis → Deploy
```

| Stage | Key questions |
|-------|---------------|
| Problem | What metric reflects user value? |
| Data | Train/val/test split? Label quality? |
| Baseline | Simplest model that could work? |
| Model | Architecture matched to data type? |
| Tune | One change at a time? |
| Deploy | Latency, memory, monitoring? |

[Course 1 ML workflow](https://github.com/Collaborative-ai/ai-academy-applied-ml-engineering/blob/main/chapters/chapter-01-machine-learning/section-06-the-ml-workflow-and-data-hygiene.md) established split discipline; deep learning adds **epoch-level monitoring** and **checkpointing**.

---

## Data Pipelines

Deep models consume data continuously during training. A robust pipeline:

1. **Load** - read from disk, database, or stream
2. **Preprocess** - normalize, tokenize, resize
3. **Augment** (training only) - random transforms ([Section 21.3](./section-03-training-deep-networks.md))
4. **Batch** - group examples for GPU efficiency
5. **Prefetch** - overlap I/O with compute

```python
# TensorFlow data pipeline sketch
ds = tf.data.Dataset.from_tensor_slices((x_train, y_train))
ds = ds.shuffle(10000).batch(64).prefetch(tf.data.AUTOTUNE)
```

**Data leakage** - validation statistics computed on full dataset, or duplicate near-duplicates across splits - inflates reported performance. Always fit normalization on **training data only**.

> **Readable form:** the pipeline feeds the GPU without starving it; leakage is accidentally teaching the exam answers.

---

## Establishing Baselines

Before a custom CNN or LSTM, implement:

| Data type | Baseline |
|-----------|----------|
| Tabular | Logistic regression / small MLP |
| Images | Linear classifier on flattened pixels, or small conv net |
| Text | Bag-of-words + logistic regression ([Course 1 Chapter 04](https://github.com/Collaborative-ai/ai-academy-applied-ml-engineering/blob/main/chapters/chapter-04-text-classification/README.md)) |
| Sequences | Markov n-gram |

If your deep model barely beats the baseline, the problem may need **more/better data** rather than more layers. AIMA stresses **incremental complexity** - justified by measured gains.

---

## Choosing Metrics

Match metrics to **deployment goals**:

| Task | Common metrics | Caveats |
|------|----------------|---------|
| Classification | Accuracy, F1, AUC-ROC | Accuracy misleading on imbalance |
| Regression | MSE, MAE, $R^2$ | Outliers dominate MSE |
| Generation | BLEU, perplexity | Imperfect proxies for quality |
| Detection | mAP | IoU thresholds matter |

Report **confidence intervals** or multiple seeds - deep training has variance from initialization and batch order.

---

## Hyperparameter Search Strategy

With limited budget:

1. **Manual tuning** on small subset - learning rate, architecture scale
2. **Random search** over log-uniform ranges ([Section 21.3](./section-03-training-deep-networks.md))
3. **Coarse-to-fine** - narrow around best region
4. **Automated tools** - Optuna, Ray Tune (optional)

**One change at a time** (ablation discipline): if you change learning rate, optimizer, and architecture simultaneously, you cannot attribute improvement.

$$
\Delta \text{val\_acc} = f(\eta, \text{arch}, \text{reg}) \quad \Rightarrow \quad \text{isolate one variable}
$$
> **Readable form:** change one knob per experiment so you know what worked.

---

## Debugging Training

Systematic checklist when training fails:

### Step 1: Overfit a Single Batch

Can the model memorize 10 examples? If not - bug in labels, loss, or backward pass ([Section 21.2](./section-02-backpropagation.md)).

### Step 2: Monitor Scalars

Log training loss, validation loss, gradient norms, weight norms, activation histograms (TensorBoard, Weights & Biases).

### Step 3: Visualize

- Misclassified examples
- CNN filters ([Section 21.4](./section-04-convolutional-networks.md))
- Attention weights (later chapters)

### Step 4: Sanitize Inputs

Check shapes, NaNs, label encoding (0-indexed vs one-hot).

| Symptom | Diagnosis |
|---------|-----------|
| Val loss ↑ while train ↓ | Overfitting → regularize |
| Both losses flat | LR too low or dead units |
| Loss NaN | LR too high, divide-by-zero |
| OOM GPU | Reduce batch size, mixed precision |

---

## Reproducibility

Document for every experiment:

- Random seeds (Python, NumPy, framework)
- Library versions
- Data version / hash
- Full hyperparameter config
- Hardware (GPU model)

```python
import random, numpy as np, tensorflow as tf
seed = 42
random.seed(seed); np.random.seed(seed); tf.random.set_seed(seed)
```

Non-determinism on GPUs (atomic ops) may still cause small drift - report **mean ± std** over 3-5 runs for important claims.

---

## Compute and Memory Management

| Technique | Purpose |
|-----------|---------|
| Mixed precision (FP16) | Faster training, less memory |
| Gradient accumulation | Simulate large batch on small GPU |
| Checkpointing | Resume after crash; keep best val model |
| Distributed training | Scale to multiple GPUs |

Estimate memory: parameters + activations + optimizer states (Adam stores 2× parameter count). A model with 10M params in FP32 uses ~40 MB weights but **>100 MB** with Adam and activations for typical batch sizes.

---

## When to Collect More Data vs Change Model

| Signal | Action |
|--------|--------|
| High train error | More capacity or train longer |
| Low train, high val error | Regularize or get more data |
| Both high | Better features, cleaner labels, check baselines |
| Plateau on large data | Architecture change or pretraining |

**Data quality** often beats architecture novelty - mislabeled examples hurt more than a missing ResNet block.

---

## Ablation Studies

Before publishing or deploying, remove components to measure contribution:

- No dropout
- No batch norm
- No data augmentation
- Shallower network

Table format AIMA recommends:

| Configuration | Val accuracy |
|---------------|--------------|
| Full model | 92.1% |
| − dropout | 89.4% |
| − augmentation | 90.2% |
| − BN | 88.7% |

> **Readable form:** ablation tells you which ingredients actually matter in the recipe.

---

## From Notebook to Production Preview

[Course 1 Chapter 07](https://github.com/Collaborative-ai/ai-academy-applied-ml-engineering/blob/main/chapters/chapter-07-operationalizing-models/README.md) covers serialization and serving. Deep models add:

- **Latency budgets** - batch inference vs real-time
- **Model compression** - quantization, pruning, distillation
- **Monitoring** - data drift, prediction distribution shift
- **Fallback** - simpler model when OOD detected

---

## Connection to Course 1

| Course 1 | AIMA link |
|----------|-----------|
| ML workflow | [06-ml-workflow.md](https://github.com/Collaborative-ai/ai-academy-applied-ml-engineering/blob/main/chapters/chapter-01-machine-learning/section-06-the-ml-workflow-and-data-hygiene.md): splits and baselines |
| Operationalizing models | [Chapter 07 README](https://github.com/Collaborative-ai/ai-academy-applied-ml-engineering/blob/main/chapters/chapter-07-operationalizing-models/README.md): deployment path |
| Hyperparameter tuning (SVM) | [03-hyperparameter-tuning.md](https://github.com/Collaborative-ai/ai-academy-applied-ml-engineering/blob/main/chapters/chapter-05-support-vector-machines/section-03-hyperparameter-tuning-c-gamma-and-gridsearchcv.md): same search discipline |
| Model comparison | [07-model-comparison-selection.md](https://github.com/Collaborative-ai/ai-academy-applied-ml-engineering/blob/main/chapters/chapter-02-regression-models/section-07-model-comparison-and-cross-validation.md): evaluation mindset |

---

## Connection to Course 3

| AIMA topic | Goodfellow et al. |
|------------|-------------------|
| Practical methodology | [Chapter 11](https://github.com/Collaborative-ai/ai-academy-deep-learning-foundations/blob/main/chapters/chapter-11-practical-methodology/README.md) - full treatment |
| Debugging optimization | [Chapter 08](https://github.com/Collaborative-ai/ai-academy-deep-learning-foundations/blob/main/chapters/chapter-08-optimization/README.md) §8.2 |
| Regularization choices | [Chapter 07](https://github.com/Collaborative-ai/ai-academy-deep-learning-foundations/blob/main/chapters/chapter-07-regularization/README.md) |
| Large-scale training | [Chapter 12](https://github.com/Collaborative-ai/ai-academy-deep-learning-foundations/blob/main/chapters/chapter-12-applications/README.md) |

---

## Reflection Questions

1. Why should you overfit a single batch before training on full data?
2. What is data leakage, and give one example in image classification.
3. When is random search preferable to grid search for hyperparameters?
4. How do you decide between collecting more data and adding model capacity?
5. What should every experiment log for reproducibility?

---

**Previous:** [Section 21.5 - Recurrent Networks](./section-05-recurrent-networks.md) · **Next:** [Section 21.7 - Applications](./section-07-applications.md)

---

## References

1. Russell, S., & Norvig, P. (2020). *Artificial Intelligence: A Modern Approach* (4th ed.), Ch. 21. [AIMA](https://aima.cs.berkeley.edu/)
2. Russell, S., & Norvig, P. - [aima-python](https://github.com/aimacode/aima-python)
3. Goodfellow, I., Bengio, Y., & Courville, A. (2016). *Deep Learning*. Ch. 11. [deeplearningbook.org](https://www.deeplearningbook.org/)
4. Sculley, D. et al. (2015). "Hidden technical debt in machine learning systems." *NeurIPS*.
5. Karpathy, A. - "A Recipe for Training Neural Networks." [karpathy.github.io](https://karpathy.github.io/2019/04/25/recipe/)



