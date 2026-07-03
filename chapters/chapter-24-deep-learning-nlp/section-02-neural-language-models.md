# Section 24.2: Neural Language Models

> **Source:** Russell & Norvig, *Artificial Intelligence: A Modern Approach* (4th ed.), Chapter 24.2
> **Previous:** [Section 24.1 - Word Embeddings](./section-01-word-embeddings.md)
> **Next:** [Section 24.3 - Seq2Seq and Attention](./section-03-seq2seq-and-attention.md)
> **Vocabulary:** [../../GLOSSARY.md](../../GLOSSARY.md)
> **Math conventions:** [../../MATH_CONVENTIONS.md](../../MATH_CONVENTIONS.md)

---
## Beyond N-Gram Tables

[Chapter 23, Section 1](../chapter-23-natural-language-processing/section-01-language-models.md) estimated $P(w_i \mid w_{i-n+1}, \ldots, w_{i-1})$ from counts. **Neural language models** use continuous [embeddings](./section-01-word-embeddings.md) and learned functions to generalize across contexts.

Bengio et al. (2003) pioneered feedforward neural LMs; RNNs ([Course 3 Chapter 10](https://github.com/donaldmorry/03-ai-academy-deep-learning-foundations/blob/main/chapters/chapter-10-sequence-modeling/README.md)) became the workhorse until transformers.

> **Readable form:** A neural LM learns a flexible function that predicts the next word from context - sharing statistical strength across similar words and phrases.

---

## Objective: Same as Always

Maximize log-likelihood of training corpus:

$$
\mathcal{L} = \sum_{t=1}^{N} \log P(w_t \mid w_1, \ldots, w_{t-1}; \theta)
$$
> **Readable form:** This loss is the scalar training signal: it is larger when predictions violate the target and smaller when they match.

Evaluate with **perplexity** ([Chapter 23](../chapter-23-natural-language-processing/section-01-language-models.md)):

$$
\text{PP} = \exp\left(-\frac{1}{N} \sum_{t=1}^{N} \log P(w_t \mid \text{context}_t)\right)
$$
> **Readable form:** The probability is obtained by summing over the hidden cases or outcomes that satisfy the query.

Neural LMs dramatically reduced perplexity on standard benchmarks - evidence they capture more than n-grams.

---

## Feedforward Neural LM

Fixed window of $n-1$ previous words → embedding vectors → concat → hidden layer → softmax over vocabulary.

$$
P(w_t \mid w_{t-n+1}, \ldots, w_{t-1}) = \text{softmax}(\mathbf{W}_o \mathbf{h} + \mathbf{b})
$$
> **Readable form:** Softmax turns raw scores into a normalized distribution, so larger logits receive more probability mass.

**Limitation:** Fixed window - same as high-order n-grams in effective context.

---

## RNN Language Models

**Recurrent** hidden state summarizes unbounded history (in theory):

$$
\mathbf{h}_t = \tanh(\mathbf{W}_{hh} \mathbf{h}_{t-1} + \mathbf{W}_{eh} \mathbf{e}_t)
$$
> **Readable form:** The hidden state blends prior context with the current token embedding, then squashes it through tanh.

$$
P(w_{t+1} \mid w_1, \ldots, w_t) = \text{softmax}(\mathbf{W}_{ho} \mathbf{h}_t)
$$
> **Readable form:** Softmax turns raw scores into a normalized distribution, so larger logits receive more probability mass.

| Advantage | Disadvantage |
|-----------|--------------|
| Variable-length context | Sequential training (slow) |
| Shared parameters across time | Vanishing gradients |
| Compact model | Hard to parallelize |

[Course 3 Chapter 10, Sections 10.1-10.3](https://github.com/donaldmorry/03-ai-academy-deep-learning-foundations/blob/main/chapters/chapter-10-sequence-modeling/README.md) derive BPTT and LSTM solutions.

---

## LSTM and GRU Language Models

**LSTM** cell preserves gradient flow through gated memory:

$$
\mathbf{f}_t = \sigma(\mathbf{W}_f [\mathbf{h}_{t-1}, \mathbf{x}_t] + \mathbf{b}_f) \quad \text{(forget gate)}
$$
> **Readable form:** The recurrent update combines current input with previous hidden state so sequence history can influence the next representation.

$$
\mathbf{c}_t = \mathbf{f}_t \odot \mathbf{c}_{t-1} + \mathbf{i}_t \odot \tilde{\mathbf{c}}_t
$$
> **Readable form:** The recurrent update combines current input with previous hidden state so sequence history can influence the next representation.

LSTM LMs set state-of-the-art perplexity through the mid-2010s - used in speech recognition rescoring and keyboard prediction.

[Course 1 Chapter 13](https://github.com/donaldmorry/01-ai-academy-applied-ml-engineering/blob/main/chapters/chapter-13-natural-language-processing/README.md) builds LSTM sentiment classifiers - classification head instead of next-word softmax, same backbone.

---

## Perplexity Improvements (Illustrative)

| Model | PTB Perplexity (approx.) |
|-------|--------------------------|
| Trigram + Kneser-Ney | ~140 |
| Feedforward neural LM | ~120 |
| LSTM LM | ~80 |
| Transformer LM | ~20-30 |

Exact numbers vary by corpus and preprocessing - trend matters more than single figures.

> **Readable form:** Each architecture generation cut perplexity substantially - the model was "less surprised" by held-out text.

---

## Training Details

| Technique | Purpose |
|-----------|---------|
| **Truncated BPTT** | Limit backprop depth for memory |
| **Gradient clipping** | Prevent exploding gradients |
| **Dropout** | Regularize recurrent connections |
| **Learning rate schedule** | Stable convergence |
| **Weight tying** | Share input embedding and output projection weights |

```python
# Simplified character-level LM training step (PyTorch-style pseudocode)
for batch in loader:
    x, y = batch  # x: input tokens, y: shifted targets
    h = init_hidden()
  for t in range(seq_len):
        logits, h = model(x[t], h)
        loss += cross_entropy(logits, y[t])
    loss.backward()
    clip_grad_norm_(model.parameters(), 1.0)
    optimizer.step()
```

---

## Sampling and Temperature

Generate text autoregressively ([Section 24.6](./section-06-gpt-and-autoregressive-lms.md)):

$$
w_{t+1} \sim P(w \mid w_1, \ldots, w_t)
$$
> **Readable form:** The expression assigns probability to the event or value using the stated model assumptions.

**Temperature** $\tau$ scales logits before softmax:

$$
P(w_i) = \frac{\exp(z_i / \tau)}{\sum_j \exp(z_j / \tau)}
$$
> **Readable form:** The probability is obtained by summing over the hidden cases or outcomes that satisfy the query.

| $\tau$ | Effect |
|--------|--------|
| Low (0.2) | Deterministic, repetitive |
| 1.0 | Training distribution |
| High (1.5) | Creative, risky |

---

## Neural vs N-Gram Comparison

| Aspect | N-gram | Neural LM |
|--------|--------|-----------|
| Sparsity | Severe | Mitigated by embeddings |
| Generalization | None for unseen n-grams | Similar words share predictions |
| Memory | Grows with $n$ and \|V\| | Fixed parameter count |
| Inference speed | Very fast | Slower (GPU helps) |
| Interpretability | Transparent counts | Black box |

Hybrid systems used n-gram **cache** + neural LM for years in production speech.

---

## Connection to Downstream Tasks

Pretrained LMs provide **contextual features** for:

- Classification (concatenate final hidden states)
- Sequence labeling (POS, NER)
- Generation (decoding)

**Pretrain then finetune** - the paradigm BERT and GPT scaled ([Sections 24.5-24.6](./section-05-bert-and-pretraining.md)).

---

## Vanishing Gradients and the Transformer Motivation

LSTMs extend effective context but struggle with very long dependencies and parallel training. **Self-attention** ([Section 24.4](./section-04-the-transformer.md)) directly connects any two positions - motivating the architecture that dominates today.

[Course 3 Chapter 10, Section 10.7](https://github.com/donaldmorry/03-ai-academy-deep-learning-foundations/blob/main/chapters/chapter-10-sequence-modeling/README.md) previews attention as the bridge from RNNs to transformers.

---

## Evaluation Beyond Perplexity

| Metric | When |
|--------|------|
| Perplexity | LM intrinsic quality |
| BLEU / ROUGE | Generation quality |
| WER reduction | Speech rescoring contribution |
| Downstream F1 | Finetuned task performance |

Low perplexity does not guarantee useful generation - always evaluate on target tasks.

---

## Stacking Neural LMs in Production

Production speech and keyboard systems often **ensemble** models:

$$
P_{\text{final}}(w \mid \text{context}) \propto P_{\text{neural}}^\lambda \cdot P_{\text{n-gram}}^{1-\lambda}
$$
> **Readable form:** The expression assigns probability to the event or value using the stated model assumptions.

Neural LMs capture long-range patterns; [n-grams](../chapter-23-natural-language-processing/section-01-language-models.md) provide calibrated local probabilities. Interpolation $\lambda \in [0.3, 0.7]$ is tuned on dev data.

---

## Key Terms (Glossary)

- **RNN** - see [../../GLOSSARY.md](../../GLOSSARY.md)
- **LSTM** - see [../../GLOSSARY.md](../../GLOSSARY.md)
- **neural LM** - see [../../GLOSSARY.md](../../GLOSSARY.md)

---

## Reflection Questions

1. How do embeddings help neural LMs generalize beyond training n-grams?
2. Why does perplexity decrease when moving from trigrams to LSTMs?
3. What is the computational bottleneck of RNN language model training?
4. How does an LSTM sentiment classifier in [Course 1 Chapter 13](https://github.com/donaldmorry/01-ai-academy-applied-ml-engineering/blob/main/chapters/chapter-13-natural-language-processing/README.md) differ from an LSTM LM?

---

**Previous:** [Section 24.1 - Word Embeddings](./section-01-word-embeddings.md) · **Next:** [Section 24.3 - Seq2Seq and Attention](./section-03-seq2seq-and-attention.md)

---

## References

1. Russell, S., & Norvig, P. (2020). *Artificial Intelligence: A Modern Approach* (4th ed.). [aima.cs.berkeley.edu](https://aima.cs.berkeley.edu/)
2. Russell, S., & Norvig, P. - [aima-python](https://github.com/aimacode/aima-python)
