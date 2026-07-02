# Section 23.1: Language Models

> **Source:** Russell & Norvig, *Artificial Intelligence: A Modern Approach* (4th ed.), Chapter 23.1
> **Previous:** [Chapter 22 - Reinforcement Learning](../chapter-22-reinforcement-learning/README.md)
> **Next:** [Section 23.2 - Text Processing](./section-02-text-processing.md)
> **Prerequisites:** [Chapter 12 - Quantifying Uncertainty](../chapter-12-quantifying-uncertainty/README.md)
> **Vocabulary:** [../../GLOSSARY.md](../../GLOSSARY.md)
> **Math conventions:** [../../MATH_CONVENTIONS.md](../../MATH_CONVENTIONS.md)

---
## What Is a Language Model?

A **language model** (LM) assigns probabilities to sequences of words. Given a sentence $w_1, w_2, \ldots, w_n$, we want:

$$
P(w_1, w_2, \ldots, w_n)
$$
> **Readable form:** The expression assigns probability to the event or value using the stated model assumptions.

Why does this matter? LMs power speech recognition (choosing among acoustic hypotheses), machine translation (scoring fluent outputs), spelling correction, autocomplete, and - in modern form - chatbots that generate entire paragraphs.

[Course 1 Chapter 04](https://github.com/Collaborative-ai/ai-academy-applied-ml-engineering/blob/main/chapters/chapter-04-text-classification/README.md) treated text as bags of words for classification. Language models care about **order**: "dog bites man" and "man bites dog" have radically different probabilities even with identical word counts.

> **Readable form:** A language model answers "how likely is this string of words?" - the statistical backbone of everything from your phone's next-word suggestion to GPT.

---

## The Chain Rule of Probability

By the chain rule:

$$
P(w_1, \ldots, w_n) = \prod_{i=1}^{n} P(w_i \mid w_1, \ldots, w_{i-1})
$$
> **Readable form:** The joint probability is built by multiplying the conditional factors specified by the model.

Each factor asks: given everything said so far, what is the probability of the next word?

For English, conditioning on the **entire** history is intractable - the context grows without bound and most histories never appear in training data.

---

## The Markov Assumption

An **n-gram model** approximates each word's probability using only the previous $n-1$ words:

$$
P(w_i \mid w_1, \ldots, w_{i-1}) \approx P(w_i \mid w_{i-n+1}, \ldots, w_{i-1})
$$
> **Readable form:** The expression assigns probability to the event or value using the stated model assumptions.

| Model | Context used | Example for $P(\text{sat} \mid \ldots)$ |
|-------|--------------|------------------------------------------|
| Unigram | None | $P(\text{sat})$ |
| Bigram | 1 word | $P(\text{sat} \mid \text{the})$ |
| Trigram | 2 words | $P(\text{sat} \mid \text{cat, the})$ |

The **Markov assumption** says the distant past does not directly influence the present - only a fixed window matters.

> **Readable form:** An n-gram model is a short-memory guesser. A trigram only looks at the last two words before predicting the next one.

---

## Estimating N-Gram Probabilities

With maximum likelihood on a corpus of $N$ tokens:

$$
P(w_n \mid w_{n-1}) = \frac{C(w_{n-1}, w_n)}{C(w_{n-1})}
$$
> **Readable form:** The expression assigns probability to the event or value using the stated model assumptions.

where $C(\cdot)$ counts occurrences.

**Example corpus:**

```
the cat sat on the mat
the dog sat on the log
```

Bigram counts for "the → ?":

| Next word | Count | Probability |
|-----------|-------|-------------|
| cat | 1 | 1/3 |
| dog | 1 | 1/3 |
| mat | 1 | 1/3 |

$$
P(\text{cat} \mid \text{the}) = \frac{C(\text{the, cat})}{C(\text{the})} = \frac{1}{3}
$$
> **Readable form:** Count how often each word pair appears, then divide by how often the first word appears alone.

---

## The Zero-Probability Problem

If trigram $(w_{i-2}, w_{i-1}, w_i)$ never appeared in training, its probability is **zero**. Multiply many zeros and entire sentences get probability zero - even plausible ones.

**Smoothing** fixes this. Common techniques:

| Method | Idea |
|--------|------|
| **Laplace (add-one)** | Add 1 to every count: $P = \frac{C+1}{N+V}$ |
| **Add-k** | Add smaller constant $k < 1$ |
| **Backoff** | If trigram missing, fall back to bigram, then unigram |
| **Kneser-Ney** | Discount observed counts; redistribute to unseen n-grams |

[Chapter 12 - Quantifying Uncertainty](../chapter-12-quantifying-uncertainty/README.md) provides the probabilistic foundations; smoothing is applied statistics on sparse counts.

```python
from collections import defaultdict

def laplace_bigrams(tokens, vocab_size):
    counts = defaultdict(int)
    context_counts = defaultdict(int)
    for i in range(1, len(tokens)):
        bigram = (tokens[i-1], tokens[i])
        counts[bigram] += 1
        context_counts[tokens[i-1]] += 1
  # P(w2|w1) = (C(w1,w2)+1) / (C(w1)+V)
    return counts, context_counts, vocab_size
```

---

## Perplexity: Evaluating Language Models

**Perplexity** measures how "surprised" the model is by a test corpus. Lower is better.

\text{PP}(W) = P(w_1, \ldots, w_N)^{-1/N} = \sqrt[N]{\prod_{i=1}^{N} \frac{1}{P(w_i \mid \text{context}_i)}}
$$

Equivalently, perplexity is $2^{H}$ where $H$ is cross-entropy in bits (or $e^H$ in nats).

$$
> **Readable form:** Perplexity is the average number of equally likely choices the model considers at each step. PP = 100 means the model is as uncertain as picking uniformly among 100 words.

| Perplexity | Interpretation |
|------------|----------------|
| 1 | Perfect prediction (impossible on real language) |
| |V| | Random guessing over vocabulary |
| 50-200 | Typical trigram LM on news text |
| 10-30 | Strong neural LMs |

**Important:** Perplexity is only comparable across models with the **same vocabulary and test set**.

---

## Interpolation and Backoff

**Linear interpolation** blends n-gram orders:

$$
\hat{P}(w_i \mid w_{i-2}, w_{i-1}) = \lambda_3 P_3 + \lambda_2 P_2 + \lambda_1 P_1
$$
> **Readable form:** The expression assigns probability to the event or value using the stated model assumptions.

where $\lambda_3 + \lambda_2 + \lambda_1 = 1$ and each $P_k$ is the k-gram estimate.

**Backoff** (e.g., Katz smoothing): if the trigram count is zero, use the bigram; if that fails, use the unigram - with discount factors so probabilities still sum to 1.

These techniques dominated NLP from the 1990s through the early 2010s. [Course 1 Chapter 04, Section 4.3](https://github.com/Collaborative-ai/ai-academy-applied-ml-engineering/blob/main/chapters/chapter-04-text-classification/section-03-bag-of-words-and-tf-idf.md) uses word counts for classification; n-grams extend counts to **sequences**.

---

## N-Gram Limitations

| Limitation | Consequence |
|------------|-------------|
| Fixed context window | Cannot capture long-range dependencies ("The keys that I left on the table yesterday are...") |
| Data sparsity | High-order n-grams rarely observed |
| No generalization | "cat" and "dog" treated as unrelated symbols |
| Storage | Large vocabularies × many n-grams = huge tables |

Neural language models ([Chapter 24](../chapter-24-deep-learning-nlp/README.md)) address these with distributed representations and unbounded effective context - but n-grams remain useful baselines, fast fallbacks, and components in hybrid systems.

> **Humorous analogy:** An n-gram model is a goldfish linguist - brilliant at finishing "peanut butter and ___" but hopeless at remembering what you said three paragraphs ago.

---

## Generating Text from N-Gram Models

To sample a sentence:

1. Start with a start symbol or seed context
2. Sample next word from $P(w \mid \text{context})$
3. Shift context window; repeat until end-of-sentence token

```python
import random

def sample_bigram(start_word, bigram_probs, max_len=20):
    sentence = [start_word]
    for _ in range(max_len - 1):
        context = sentence[-1]
        choices = bigram_probs.get(context, {})
        if not choices:
            break
        words, probs = zip(*choices.items())
        nxt = random.choices(words, weights=probs, k=1)[0]
        if nxt == "<EOS>":
            break
        sentence.append(nxt)
    return " ".join(sentence)
```

Output is locally fluent but globally incoherent - a preview of why longer context matters.

---

## Connection to Modern LLMs

Today's large language models still optimize the same objective - likelihood of text - but with neural networks instead of count tables. The **perplexity metric** carries over: researchers report PP on held-out corpora when comparing architectures.

[Course 3 Chapter 10](https://github.com/Collaborative-ai/ai-academy-deep-learning-foundations/blob/main/chapters/chapter-10-sequence-modeling/README.md) develops RNN-based sequence models that learn $P(w_i \mid w_1, \ldots, w_{i-1})$ without explicit n-gram tables. [Chapter 24, Section 2](../chapter-24-deep-learning-nlp/section-02-neural-language-models.md) applies this directly to language modeling.

| Era | Representation | Context |
|-----|--------------|---------|
| 1990s-2000s | N-gram counts | Fixed $n-1$ words |
| 2010s | RNN/LSTM | Hidden state (limited memory) |
| 2017+ | Transformer | Self-attention (long range) |

---

## Building a Trigram LM: Checklist

1. **Tokenize** corpus ([Section 23.2](./section-02-text-processing.md))
2. **Count** unigrams, bigrams, trigrams
3. **Smooth** (Kneser-Ney recommended for production baselines)
4. **Evaluate** perplexity on held-out text
5. **Compare** to unigram/bigram to justify complexity

The Chapter 23 lab asks you to implement steps 2-4 on a real corpus.

---

## Key Terms (Glossary)

- **language model** - see [../../GLOSSARY.md](../../GLOSSARY.md)
- **n-gram** - see [../../GLOSSARY.md](../../GLOSSARY.md)
- **perplexity** - see [../../GLOSSARY.md](../../GLOSSARY.md)

---

## Reflection Questions

1. Why does a trigram model assign probability zero to a grammatically correct sentence never seen in training?
2. If perplexity drops from 150 to 75, did the model improve? What else would you check?
3. How does an n-gram LM differ from the bag-of-words model in [Course 1 Chapter 04](https://github.com/Collaborative-ai/ai-academy-applied-ml-engineering/blob/main/chapters/chapter-04-text-classification/section-03-bag-of-words-and-tf-idf.md)?
4. When might you still deploy an n-gram model in 2026?

---

**Previous:** [Chapter 22 - Reinforcement Learning](../chapter-22-reinforcement-learning/README.md) · **Next:** [Section 23.2 - Text Processing](./section-02-text-processing.md)

---

## References

1. Russell, S., & Norvig, P. (2020). *Artificial Intelligence: A Modern Approach* (4th ed.). [aima.cs.berkeley.edu](https://aima.cs.berkeley.edu/)
2. Russell, S., & Norvig, P. - [aima-python](https://github.com/aimacode/aima-python)


