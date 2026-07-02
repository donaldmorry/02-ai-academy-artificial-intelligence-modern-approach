# Section 24.3: Seq2Seq and Attention

> **Source:** Russell & Norvig, *Artificial Intelligence: A Modern Approach* (4th ed.), Chapter 24, Deep Learning for Natural Language Processing
> **Enhanced with:** source-grounded explanations, worked checks, implementation guidance, diagnostics, and mastery prompts
> **Vocabulary:** [GLOSSARY.md](../../GLOSSARY.md)
> **Math conventions:** [MATH_CONVENTIONS.md](../../MATH_CONVENTIONS.md)

---

## The Core Idea

Sequence-to-sequence models turn one sequence into another. AIMA introduces them through machine translation: given a source-language sentence, generate a target-language sentence that preserves meaning even when words do not correspond one-to-one and word order changes.

A basic seq2seq model uses one recurrent network to encode the source sentence and another recurrent network to decode the target sentence. The final hidden state of the source encoder initializes the target decoder. This was a breakthrough, but it forces the whole source sentence through one fixed-size vector.

Attention fixes that bottleneck by letting the decoder look back at all source hidden states at each target step.

> **Readable form:** Seq2seq models generate target tokens conditioned on a source sequence and previously generated target tokens; attention gives the decoder a learned view of the most relevant source positions.

---

## Source Grounding

AIMA names three shortcomings of basic recurrent seq2seq models:

| Shortcoming | Why it hurts translation |
|-------------|--------------------------|
| Nearby context bias | RNN hidden states often preserve recent words better than distant words |
| Fixed context size | A long source sentence is compressed into one vector |
| Sequential processing | RNNs process tokens one step at a time, limiting parallelism |

Attention was introduced as a way to reduce the first two problems. Instead of relying only on the final source state, the target decoder forms a context vector from all source states.

---

## Attention Mechanics

At target step $i$, compare the previous target hidden state $h_{i-1}$ with every source hidden state $s_j$.

$$
r_{ij}=h_{i-1}\cdot s_j
$$
> **Readable form:** The raw attention score measures how relevant source position $j$ is for the next target word.

$$
a_{ij}=\frac{e^{r_{ij}}}{\sum_k e^{r_{ik}}}
$$
> **Readable form:** Softmax turns raw scores into attention probabilities over source positions.

$$
c_i=\sum_j a_{ij}s_j
$$
> **Readable form:** The context vector is a weighted average of source states.

The decoder then uses both the current input and the context vector:

$$
h_i=\operatorname{RNN}(h_{i-1},[x_i;c_i])
$$
> **Readable form:** The decoder updates its state using the current target input plus the source summary chosen by attention.

---

## Why Attention Helps

AIMA emphasizes three benefits:

| Benefit | Explanation |
|---------|-------------|
| Differentiability | Softmax attention lets gradients flow through the alignment decision |
| Long-distance access | The decoder can consider any source position directly |
| Uncertainty | Probability mass can be spread across several plausible source words |

Attention weights can also be interpretable. In translation, high probabilities often correspond to word alignments a human might draw, although they should still be treated as model internals rather than guaranteed explanations.

---

## Decoding

Training maximizes the probability of target words given the source and previous target words. At inference time, the model must search for a whole target sequence.

| Decoder | Behavior | Risk |
|---------|----------|------|
| Greedy decoding | Choose the most probable next word at each step | Early local mistakes cannot be repaired |
| Beam search | Keep the top $k$ partial hypotheses at each step | More compute, but better sequence-level search |

AIMA connects this directly back to search: decoding is a search problem over possible output sequences.

---

## Minimal Implementation Check

```python
def attention_context(target_state, source_states):
    scores = [sum(t * s for t, s in zip(target_state, source)) for source in source_states]
    m = max(scores)
    weights = [pow(2.718281828, score - m) for score in scores]
    total = sum(weights)
    probs = [w / total for w in weights]
    context = [sum(p * source[d] for p, source in zip(probs, source_states)) for d in range(len(target_state))]
    return probs, context
```

Use a two-word source sentence and verify that the attention probabilities sum to 1. Then change the target state and confirm that the context vector changes.

---

## Diagnostics

| Diagnostic | Healthy signal | Warning signal |
|------------|----------------|----------------|
| Attention probabilities | Sum to 1 over source positions | NaNs or mass on padding tokens |
| Translation output | Preserves meaning and order where needed | Repetition, omissions, or wrong word order |
| Beam search | Improves sequence quality over greedy decoding | Larger beam produces generic or length-biased output |
| Alignment inspection | Plausible source-target correspondences | Attention ignores crucial source words |
| Edge cases | Handles empty, short, and long inputs gracefully | Crashes or emits unbounded repetition |

---

## Reflection Questions

1. Why is machine translation not a simple word-tagging task?
2. What information is lost when a source sentence is compressed into one hidden state?
3. How does attention create a context vector?
4. Why does softmax attention help back-propagation?
5. When can greedy decoding fail even if the model assigns good local probabilities?
6. How does beam search connect NLP generation to AIMA search?

---

## References

- Russell, S., & Norvig, P. (2020). *Artificial Intelligence: A Modern Approach* (4th ed.), Ch. 24.
- AIMA Python code: [https://github.com/aimacode/aima-python](https://github.com/aimacode/aima-python)

---

**Previous:** [Section 24.2: Neural Language Models](./section-02-neural-language-models.md)  
**Next:** [Section 24.4: The Transformer](./section-04-the-transformer.md)

---

## Assessment Practice

Use the shared [Assessment Appendix](../../ASSESSMENT_APPENDIX.md) for concept audits, worked examples, implementation checks, experiment logs, oral-exam prompts, and deliverable checklists.
