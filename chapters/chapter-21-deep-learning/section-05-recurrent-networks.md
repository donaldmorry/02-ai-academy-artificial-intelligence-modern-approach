# Section 21.5: Recurrent Networks

> **Source:** Russell & Norvig, *Artificial Intelligence: A Modern Approach* (4th ed.), Chapter 21.5  
> **Previous:** [Section 21.4 - Convolutional Networks](./section-04-convolutional-networks.md)  
> **Vocabulary:** [GLOSSARY.md](../../GLOSSARY.md)  
> **Math conventions:** [MATH_CONVENTIONS.md](../../MATH_CONVENTIONS.md)

---

## Sequences in Intelligent Agents

Many agent tasks involve **ordered data**: speech waveforms, text, robot sensor streams, stock prices, patient vital signs over time. Feedforward networks ([Section 21.1](./section-01-feedforward-networks.md)) map fixed-size inputs to outputs - they cannot naturally handle variable-length sequences or temporal dependencies without awkward fixed windows.

**Recurrent neural networks (RNNs)** maintain a **hidden state** updated at each time step, giving the network memory of past inputs. Russell and Norvig introduce RNNs alongside CNNs as core deep architectures, with **LSTM** as the principal solution to training difficulties on long sequences.

> **Readable form:** at each time step, the network reads a new input, updates its internal memory, and optionally produces an output - same weights applied at every step.

Humorous analogy: a feedforward net is a photograph; an RNN is a flipbook where each page remembers what happened on previous pages.

---

## Basic RNN Formulation

For input sequence $\mathbf{x}_1, \ldots, \mathbf{x}_T$:

$$
\mathbf{h}_t = g\!\left(\mathbf{W}_{xh}\,\mathbf{x}_t + \mathbf{W}_{hh}\,\mathbf{h}_{t-1} + \mathbf{b}_h\right)
$$

$$
\mathbf{y}_t = g_y\!\left(\mathbf{W}_{hy}\,\mathbf{h}_t + \mathbf{b}_y\right)
$$
> **Readable form:** new hidden state combines current input with previous hidden state through the same weight matrices.

**Weight sharing across time** - $\mathbf{W}_{xh}, \mathbf{W}_{hh}, \mathbf{W}_{hy}$ reused at every $t$ - mirrors spatial weight sharing in CNNs ([Section 21.4](./section-04-convolutional-networks.md)).

```python
def rnn_step(x_t, h_prev, W_xh, W_hh, b):
    h_t = np.tanh(W_xh @ x_t + W_hh @ h_prev + b)
    return h_t
```

Initial state $\mathbf{h}_0$ often zeros or learned. The unfolded graph over $T$ steps is a deep network with shared parameters - backprop through it is **BPTT** (backpropagation through time), extending [Section 21.2](./section-02-backpropagation.md).

---

## RNN Application Patterns

| Pattern | Input → Output | Example |
|---------|----------------|---------|
| **Many-to-one** | Full sequence → single label | Sentiment classification |
| **One-to-many** | Single input → sequence | Image captioning |
| **Many-to-many (sync)** | Aligned sequences | Video frame labeling |
| **Many-to-many (seq2seq)** | Different lengths | Machine translation |

[Chapter 23](../chapter-23-natural-language-processing/README.md) and [Chapter 24](../chapter-24-deep-learning-nlp/README.md) build on these patterns with Transformers; RNNs remain the pedagogical foundation AIMA presents.

---

## The Vanishing Gradient Problem

BPTT backpropagates through $T$ time steps. Gradient w.r.t. early hidden states involves products of repeated Jacobian $\mathbf{W}_{hh}$ and $g'(\cdot)$:

$$
\frac{\partial \mathcal{L}}{\partial \mathbf{h}_1} \propto \prod_{t=2}^{T} \frac{\partial \mathbf{h}_t}{\partial \mathbf{h}_{t-1}}
$$
> **Readable form:** The gradient reaching the first hidden state is a product of many timestep Jacobians, so repeated small factors shrink it rapidly.

If $\|\mathbf{W}_{hh}\| < 1$ and $g' \leq 1$ (tanh/sigmoid), the product **exponentially vanishes** - early time steps receive negligible learning signal. **Exploding** gradients (product $> 1$) cause unstable updates.

> **Readable form:** credit for a mistake at the end of a long sentence barely reaches the beginning - early words never learn.

Mitigations:

1. **Gradient clipping** - cap gradient norm
2. **Better activations and init** - partial help only
3. **Gated architectures** - LSTM, GRU (primary solution)
4. **Truncated BPTT** - backprop through last $k$ steps only

---

## LSTM: Long Short-Term Memory

**LSTM** (Hochreiter & Schmidhuber, 1997) introduces a **cell state** $\mathbf{c}_t$ (long-term memory) and **gates** that control information flow:

### Forget Gate

$$
\mathbf{f}_t = \sigma\!\left(\mathbf{W}_f [\mathbf{h}_{t-1}, \mathbf{x}_t] + \mathbf{b}_f\right)
$$
> **Readable form:** decide what fraction of old cell state to erase (0 = forget all, 1 = keep all).

### Input Gate and Candidate

$$
\mathbf{i}_t = \sigma\!\left(\mathbf{W}_i [\mathbf{h}_{t-1}, \mathbf{x}_t] + \mathbf{b}_i\right)
$$
> **Readable form:** The recurrent update combines current input with previous hidden state so sequence history can influence the next representation.

$$
\tilde{\mathbf{c}}_t = \tanh\!\left(\mathbf{W}_c [\mathbf{h}_{t-1}, \mathbf{x}_t] + \mathbf{b}_c\right)
$$
> **Readable form:** The recurrent update combines current input with previous hidden state so sequence history can influence the next representation.

### Cell Update

$$
\mathbf{c}_t = \mathbf{f}_t \odot \mathbf{c}_{t-1} + \mathbf{i}_t \odot \tilde{\mathbf{c}}_t
$$
> **Readable form:** new cell state equals filtered old memory plus filtered new candidate content.

### Output Gate

\mathbf{o}_t = \sigma\!\left(\mathbf{W}_o [\mathbf{h}_{t-1}, \mathbf{x}_t] + \mathbf{b}_o\right), \quad
\mathbf{h}_t = \mathbf{o}_t \odot \tanh(\mathbf{c}_t)

The **additive cell update** creates a **gradient highway** - errors can flow backward through $\mathbf{c}_t$ with less repeated multiplication by $\mathbf{W}_{hh}$, addressing vanishing gradients over long dependencies.

```python
# Simplified LSTM cell (one step)
def lstm_step(x_t, h_prev, c_prev, params):
    concat = np.concatenate([h_prev, x_t])
    f = sigmoid(params['Wf'] @ concat + params['bf'])
    i = sigmoid(params['Wi'] @ concat + params['bi'])
    c_tilde = np.tanh(params['Wc'] @ concat + params['bc'])
    c_t = f * c_prev + i * c_tilde
    o = sigmoid(params['Wo'] @ concat + params['bo'])
    h_t = o * np.tanh(c_t)
    return h_t, c_t
```

---

## GRU: Gated Recurrent Unit

**GRU** (Cho et al., 2014) simplifies LSTM with two gates - **reset** and **update** - merging cell and hidden state:

\mathbf{z}_t = \sigma(\mathbf{W}_z [\mathbf{h}_{t-1}, \mathbf{x}_t]), \quad
\mathbf{r}_t = \sigma(\mathbf{W}_r [\mathbf{h}_{t-1}, \mathbf{x}_t])

\tilde{\mathbf{h}}_t = \tanh(\mathbf{W} [\mathbf{r}_t \odot \mathbf{h}_{t-1}, \mathbf{x}_t]), \quad
\mathbf{h}_t = (1 - \mathbf{z}_t) \odot \mathbf{h}_{t-1} + \mathbf{z}_t \odot \tilde{\mathbf{h}}_t
> **Readable form:** update gate blends previous hidden state with new candidate - fewer parameters than LSTM, often comparable performance.

| Model | Gates | Parameters | Typical use |
|-------|-------|------------|-------------|
| Vanilla RNN | 0 | Fewest | Short sequences only |
| GRU | 2 | Moderate | Faster training |
| LSTM | 3 | Most | Long dependencies, speech, text |

[Course 3 Chapter 10](https://github.com/Collaborative-ai/ai-academy-deep-learning-foundations/blob/main/chapters/chapter-10-sequence-modeling/README.md) derives BPTT and compares cells in depth.

---

## Encoder-Decoder and Seq2seq Preview

**Machine translation** maps variable-length source to variable-length target. An **encoder** RNN compresses the source into a **context vector** $\mathbf{h}_T$; a **decoder** RNN generates target tokens autoregressively:

$$
P(y_1, \ldots, y_{T'} \mid x_1, \ldots, x_T) = \prod_{t=1}^{T'} P(y_t \mid y_{<t}, \mathbf{h}_T)
$$
> **Readable form:** The joint probability is built by multiplying the conditional factors specified by the model.

**Teacher forcing** feeds ground-truth previous tokens during training; at inference, the model feeds its own predictions - **exposure bias** can hurt. Attention mechanisms ([Chapter 24](../chapter-24-deep-learning-nlp/README.md)) replace fixed context vectors with dynamic alignment - the Transformer era builds on this insight.

---

## Bidirectional RNNs

Process sequence **forward and backward**, concatenate hidden states:

\overrightarrow{\mathbf{h}}_t = \text{RNN}(\mathbf{x}_t, \overrightarrow{\mathbf{h}}_{t-1}), \quad
\overleftarrow{\mathbf{h}}_t = \text{RNN}(\mathbf{x}_t, \overleftarrow{\mathbf{h}}_{t+1})

$$
\mathbf{h}_t = [\overrightarrow{\mathbf{h}}_t ; \overleftarrow{\mathbf{h}}_t]
$$
> **Readable form:** The recurrent update combines current input with previous hidden state so sequence history can influence the next representation.

Useful for **tagging** (POS, NER) where each position needs full-sentence context. Not used for autoregressive generation (would leak future information).

---

## Training RNNs in Practice

Apply [Section 21.3](./section-03-training-deep-networks.md) techniques:

- **Gradient clipping** (essential): $\mathbf{g} \leftarrow \mathbf{g} \cdot \min(1, \tau/\|\mathbf{g}\|)$
- **Dropout** on non-recurrent connections (variational dropout for recurrent)
- **Truncated BPTT** with unroll length 20-35 for language modeling
- **Adam** or SGD with momentum; learning rate warmup

Chapter lab: train LSTM on character-level text - compare vanilla RNN (fails on long range) vs LSTM (captures phrases).

---

## RNNs vs Other Sequence Models

| Approach | Strength | Weakness |
|----------|----------|----------|
| Vanilla RNN | Simple | Vanishing gradients |
| LSTM/GRU | Long-range deps | Sequential - hard to parallelize |
| 1-D CNN | Parallelizable | Limited receptive field |
| Transformer | Parallel + long range | Quadratic memory in length |

AIMA presents RNNs as historically central; [Chapter 24](../chapter-24-deep-learning-nlp/README.md) covers the Transformer shift. Understanding LSTM gates remains essential for reading pre-2018 literature and small-sequence deployments.

---

## Connection to Course 1

| Course 1 | AIMA link |
|----------|-----------|
| Chapter 09 - Keras RNN layers | [README.md](https://github.com/Collaborative-ai/ai-academy-applied-ml-engineering/blob/main/chapters/chapter-09-neural-networks/README.md): LSTM layer API |
| Text as data | [Chapter 04](https://github.com/Collaborative-ai/ai-academy-applied-ml-engineering/blob/main/chapters/chapter-04-text-classification/README.md): sequences before deep NLP |
| ML workflow | [06-ml-workflow.md](https://github.com/Collaborative-ai/ai-academy-applied-ml-engineering/blob/main/chapters/chapter-01-machine-learning/section-06-the-ml-workflow-and-data-hygiene.md): train/val split for sequences |
| Chapter 08 | [01-why-deep-learning.md](https://github.com/Collaborative-ai/ai-academy-applied-ml-engineering/blob/main/chapters/chapter-08-deep-learning/section-01-why-deep-learning.md): sequence tasks motivation |

---

## Connection to Course 3

| AIMA topic | Goodfellow et al. |
|------------|-------------------|
| Unfolding computational graphs | [Chapter 10](https://github.com/Collaborative-ai/ai-academy-deep-learning-foundations/blob/main/chapters/chapter-10-sequence-modeling/README.md) §10.1 |
| Vanishing/exploding gradients | [Chapter 10](https://github.com/Collaborative-ai/ai-academy-deep-learning-foundations/blob/main/chapters/chapter-10-sequence-modeling/README.md) §10.2 |
| LSTM architecture | [Chapter 10](https://github.com/Collaborative-ai/ai-academy-deep-learning-foundations/blob/main/chapters/chapter-10-sequence-modeling/README.md) §10.3 |
| GRU | [Chapter 10](https://github.com/Collaborative-ai/ai-academy-deep-learning-foundations/blob/main/chapters/chapter-10-sequence-modeling/README.md) §10.4 |
| Encoder-decoder & attention preview | [Chapter 10](https://github.com/Collaborative-ai/ai-academy-deep-learning-foundations/blob/main/chapters/chapter-10-sequence-modeling/README.md) §10.6-10.7 |

---

## Reflection Questions

1. What problem do LSTM forget and input gates solve that vanilla RNNs cannot?
2. Why does the additive cell update $\mathbf{c}_t = \mathbf{f}_t \odot \mathbf{c}_{t-1} + \ldots$ help gradient flow?
3. When would bidirectional RNNs be inappropriate?
4. Explain teacher forcing and one risk it introduces at inference time.
5. How is weight sharing across time analogous to weight sharing in CNNs?

---

**Previous:** [Section 21.4 - Convolutional Networks](./section-04-convolutional-networks.md) · **Next:** [Section 21.6 - Practical Methodology](./section-06-practical-methodology.md)

---

## References

1. Russell, S., & Norvig, P. (2020). *Artificial Intelligence: A Modern Approach* (4th ed.), Ch. 21. [AIMA](https://aima.cs.berkeley.edu/)
2. Russell, S., & Norvig, P. - [aima-python](https://github.com/aimacode/aima-python)
3. Goodfellow, I., Bengio, Y., & Courville, A. (2016). *Deep Learning*. Ch. 10. [deeplearningbook.org](https://www.deeplearningbook.org/)
4. Hochreiter, S., & Schmidhuber, J. (1997). "Long short-term memory." *Neural Computation*.
5. Cho, K. et al. (2014). "Learning phrase representations using RNN encoder-decoder." *EMNLP*.
