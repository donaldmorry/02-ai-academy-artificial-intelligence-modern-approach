# Section 24.4: The Transformer

> **Source:** Russell & Norvig, *Artificial Intelligence: A Modern Approach* (4th ed.), Chapter 24, Deep Learning for Natural Language Processing
> **Enhanced with:** source-grounded explanations, worked checks, implementation guidance, diagnostics, and mastery prompts
> **Vocabulary:** [GLOSSARY.md](../../GLOSSARY.md)
> **Math conventions:** [MATH_CONVENTIONS.md](../../MATH_CONVENTIONS.md)

---

## The Core Idea

AIMA presents the transformer as the architecture that uses self-attention to model long-distance context without the sequential dependency of RNNs. Instead of passing one hidden state forward through time, each position can attend to other positions in the same sequence.

This is the architectural hinge of modern NLP. It improves parallel training, gives every token direct access to contextual evidence, and supports both encoder-style understanding models and decoder-style generation models.

> **Readable form:** A transformer layer updates token representations by self-attention, a position-wise feedforward network, and residual connections.

---

## Self-Attention

In ordinary seq2seq attention, a target state attends to source states. In self-attention, the sequence attends to itself. Each token can gather information from other tokens in the same sentence.

AIMA introduces projections for keys, queries, and values so attention can learn different comparison spaces:

| Vector | Role |
|--------|------|
| Query | What this position is looking for |
| Key | What each position offers for matching |
| Value | The information copied into the context vector |

$$
\operatorname{Attention}(Q,K,V)=\operatorname{softmax}\left(\frac{QK^T}{\sqrt{d_k}}\right)V
$$
> **Readable form:** Compare queries with keys, normalize the scores, and average the values.

---

## Multiheaded Attention

AIMA notes that one attention matrix can blur different kinds of relationships. Multiheaded attention splits the representation into several subspaces, applies attention separately, and concatenates the results.

| Head might specialize in | Example |
|--------------------------|---------|
| Syntax | Subject-verb agreement |
| Coreference | Pronoun and antecedent |
| Local phrase structure | Adjective modifying noun |
| Long-distance relation | Dependency across a clause |

Heads are learned, so these interpretations are diagnostics rather than guarantees.

---

## Transformer Layer

A single transformer layer contains:

1. Self-attention.
2. Position-wise feedforward layers.
3. Nonlinear activation.
4. Residual connections.
5. Usually normalization in practical implementations.

The feedforward network is applied independently at each position after attention has mixed contextual information. Residual connections help gradients flow through deep stacks.

---

## Positional Embeddings

Self-attention alone is insensitive to word order. AIMA explains that transformers add positional embeddings to word embeddings so the model can distinguish the first word from the fifth word.

$$
z_t=e_{w_t}+p_t
$$
> **Readable form:** The input representation at position $t$ combines word identity with position identity.

Without positional information, the same bag of words would look too similar even when order changes meaning.

---

## Encoder and Decoder

AIMA separates the transformer encoder from the full encoder-decoder architecture:

| Component | Attention pattern | Use case |
|-----------|-------------------|----------|
| Transformer encoder | Tokens attend to the whole input sequence | Classification, tagging, contextual representations |
| Transformer decoder | Tokens attend only to previous target tokens, plus encoder output in seq2seq | Left-to-right generation and translation |

The decoder uses a causal mask so it cannot peek at future target tokens during generation.

---

## Minimal Mask Check

```python
def causal_mask(length):
    return [[j <= i for j in range(length)] for i in range(length)]

assert causal_mask(3) == [
    [True, False, False],
    [True, True, False],
    [True, True, True],
]
```

A mask bug is a serious evaluation leak: the model may appear to predict well because it saw future tokens during training.

---

## Diagnostics

| Diagnostic | Healthy signal | Warning signal |
|------------|----------------|----------------|
| Shape check | Attention scores are batch x heads x tokens x tokens | Silent broadcasting changes scores |
| Mask check | Padding and future positions are blocked when required | Model attends to padding or future text |
| Positional check | Word order changes affect outputs | Model behaves like order does not matter |
| Complexity check | Attention cost grows with sequence length | Long contexts become infeasible unexpectedly |
| Baseline check | Transformer beats simpler RNN where long context matters | Extra capacity gives no measured benefit |

---

## Reflection Questions

1. What problem does self-attention solve compared with RNN sequence processing?
2. Why do transformers need positional embeddings?
3. What is the difference between encoder and decoder self-attention?
4. Why can multiheaded attention be more expressive than one attention head?
5. What failure would a broken causal mask create?
6. How does transformer parallelism change training feasibility?

---

## References

- Russell, S., & Norvig, P. (2020). *Artificial Intelligence: A Modern Approach* (4th ed.), Ch. 24.
- AIMA Python code: [https://github.com/aimacode/aima-python](https://github.com/aimacode/aima-python)

---

**Previous:** [Section 24.3: Seq2Seq and Attention](./section-03-seq2seq-and-attention.md)  
**Next:** [Section 24.5: BERT and Pretraining](./section-05-bert-and-pretraining.md)

---

## Assessment Practice

Use the shared [Assessment Appendix](../../ASSESSMENT_APPENDIX.md) for concept audits, worked examples, implementation checks, experiment logs, oral-exam prompts, and deliverable checklists.
