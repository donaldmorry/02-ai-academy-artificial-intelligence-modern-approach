# Section 24.1: Word Embeddings

> **Source:** Russell & Norvig, *Artificial Intelligence: A Modern Approach* (4th ed.), Chapter 24.1
> **Previous:** [Chapter 23 - Natural Language Processing](../chapter-23-natural-language-processing/README.md)
> **Next:** [Section 24.2 - Neural Language Models](./section-02-neural-language-models.md)
> **Prerequisites:** [Chapters 21 & 23 - Deep Learning & NLP](../chapter-21-deep-learning/README.md)
> **Vocabulary:** [../../GLOSSARY.md](../../GLOSSARY.md)
> **Math conventions:** [../../MATH_CONVENTIONS.md](../../MATH_CONVENTIONS.md)

---
## The Limits of One-Hot Vectors

[Course 1 Chapter 04](https://github.com/Collaborative-ai/ai-academy-applied-ml-engineering/blob/main/chapters/chapter-04-text-classification/section-03-bag-of-words-and-tf-idf.md) represents words as sparse high-dimensional vectors - mostly zeros, no notion of similarity. "Cat" and "dog" are as unrelated as "cat" and "democracy."

**Word embeddings** map words to dense, low-dimensional vectors $\mathbf{v}_w \in \mathbb{R}^d$ where semantic similarity corresponds to geometric proximity.

> **Readable form:** Embeddings place words on a map where neighbors mean similar things - "king" sits near "queen," far from "carburetor."

---

## Distributional Hypothesis

Words in similar contexts have similar meanings (Firth, 1957; Harris, 1954). [N-gram models](../chapter-23-natural-language-processing/section-01-language-models.md) exploit co-occurrence counts; embeddings **compress** co-occurrence into vectors.

$$
\text{similarity}(w_1, w_2) \approx f(\text{shared contexts}(w_1, w_2))
$$
> **Readable form:** The score compares vector representations; closer or more aligned vectors receive a better match score.

Neural methods learn $f$ automatically from raw text.

---

## Word2Vec: Skip-Gram and CBOW

Mikolov et al. (2013) introduced efficient training objectives.

### Skip-Gram

Predict context words given center word $w_c$:

$$
\max \sum_{(w_c, w_o) \in \mathcal{D}} \log P(w_o \mid w_c)
$$
> **Readable form:** The probability is obtained by summing over the hidden cases or outcomes that satisfy the query.

$$
P(w_o \mid w_c) = \frac{\exp(\mathbf{v}_{w_o}^\top \mathbf{v}_{w_c}')}{\sum_{w \in V} \exp(\mathbf{v}_w^\top \mathbf{v}_{w_c}')}
$$
> **Readable form:** The probability is obtained by summing over the hidden cases or outcomes that satisfy the query.

**Negative sampling** approximates the expensive softmax denominator.

### CBOW (Continuous Bag of Words)

Predict center word from surrounding context - inverse of skip-gram.

| Model | Predicts | Best for |
|-------|----------|----------|
| Skip-gram | Context from center | Rare words |
| CBOW | Center from context | Frequent words, speed |

> **Readable form:** Skip-gram asks "given 'king,' what words appear nearby?" CBOW asks "given nearby words, what's the missing one?"

---

## Famous Vector Arithmetic

Trained embeddings capture relational structure:

$$
\mathbf{v}_{\text{king}} - \mathbf{v}_{\text{man}} + \mathbf{v}_{\text{woman}} \approx \mathbf{v}_{\text{queen}}
$$
> **Readable form:** The gender direction from man to woman applied to king lands near queen in embedding space.

$$
\mathbf{v}_{\text{Paris}} - \mathbf{v}_{\text{France}} + \mathbf{v}_{\text{Italy}} \approx \mathbf{v}_{\text{Rome}}
$$
> **Readable form:** The capital-city relation from France to Paris transfers to Italy and points near Rome.

Not magic - linear structure emerges from predicting contexts where these words substitute.

[Course 1 Chapter 13](https://github.com/Collaborative-ai/ai-academy-applied-ml-engineering/blob/main/chapters/chapter-13-natural-language-processing/README.md) visualizes this with t-SNE on learned embedding layers.

---

## GloVe: Global Vectors

Pennington et al. (2014) - **GloVe** combines global co-occurrence statistics with local context windows.

Objective minimizes:

$$
J = \sum_{i,j} f(X_{ij}) \left( \mathbf{w}_i^\top \tilde{\mathbf{w}}_j + b_i + \tilde{b}_j - \log X_{ij} \right)^2
$$
> **Readable form:** The total combines the indexed terms, so each relevant example, state, feature, or dimension contributes once.

where $X_{ij}$ counts co-occurrences, $f$ down-weights rare pairs.

| Method | Training signal |
|--------|----------------|
| Word2Vec | Local context windows |
| GloVe | Global co-occurrence matrix |

Both produce useful static embeddings; choice often matters less than dimensionality and corpus size.

---

## Embedding Properties

| Property | Typical value |
|----------|---------------|
| Dimension $d$ | 100-300 (static), 768+ (contextual) |
| Vocabulary | 50k-3M tokens |
| Training corpus | Wikipedia, news, web crawl |

**Static embeddings:** one vector per word type - "bank" has one vector for all senses.

**Contextual embeddings** ([BERT](./section-05-bert-and-pretraining.md), transformers): vector depends on sentence - resolves sense ambiguity.

---

## Subword Tokenization

Rare and unknown words break word-level embeddings. **Subword** methods ([Section 23.2](../chapter-23-natural-language-processing/section-02-text-processing.md)) split words:

| Algorithm | Used by |
|-----------|---------|
| BPE | GPT-2, RoBERTa |
| WordPiece | BERT |
| SentencePiece | T5, multilingual models |

$$
\text{embed}(\text{unhappiness}) = \text{combine}(\text{embed}(\text{un}), \text{embed}(\text{happi}), \text{embed}(\text{ness}))
$$
> **Readable form:** A subword model represents an unfamiliar word by combining embeddings of its known pieces.

Character-level models ([Course 3 Chapter 10](https://github.com/Collaborative-ai/ai-academy-deep-learning-foundations/blob/main/chapters/chapter-10-sequence-modeling/README.md)) extend this to every string - slower but zero OOV rate.

---

## Using Pretrained Embeddings

```python
import numpy as np
from tensorflow.keras.layers import Embedding

# Load GloVe matrix: words[i] -> vector
embedding_matrix = np.zeros((vocab_size, 300))
for word, i in word_index.items():
    if word in glove_vectors:
        embedding_matrix[i] = glove_vectors[word]

layer = Embedding(vocab_size, 300, weights=[embedding_matrix], trainable=False)
```

| Strategy | When |
|----------|------|
| **Frozen pretrained** | Small target data |
| **Finetune** | Domain-specific vocabulary |
| **Train from scratch** | Large domain corpus |

[Course 1 Chapter 13](https://github.com/Collaborative-ai/ai-academy-applied-ml-engineering/blob/main/chapters/chapter-13-natural-language-processing/README.md) compares learned vs GloVe-initialized embeddings on sentiment.

---

## From Embeddings to Neural LMs

Embeddings are the **input layer** of neural language models ([Section 24.2](./section-02-neural-language-models.md)):

$$
\mathbf{h}_t = f(\mathbf{W}_e \mathbf{x}_t, \mathbf{h}_{t-1})
$$
> **Readable form:** The recurrent hidden state uses the current token embedding and previous hidden state to update sequence memory.

[Chapter 21 - Deep Learning](../chapter-21-deep-learning/README.md) provides backprop foundations; embeddings are the first learned parameters.

---

## Evaluation

| Intrinsic | Extrinsic |
|-----------|-----------|
| Word similarity correlation (WordSim-353) | Downstream task accuracy |
| Analogy accuracy | Perplexity of LM using embeddings |
| Clustering quality | NER F1 with embedding features |

Intrinsic tests probe geometry; extrinsic tests measure utility - prefer extrinsic for deployment decisions.

---

## Limitations of Static Embeddings

| Issue | Example |
|-------|---------|
| Polysemy | Single vector for "bank" |
| No morphology | Rare forms poorly represented |
| Bias | Gender stereotypes in analogies |
| Out-of-vocabulary | New words need retraining |

Contextual transformers address polysemy and OOV - at compute cost. [Section 24.8](./section-08-nlp-ethics.md) addresses bias.

---

## Connection to Classical NLP

| Classical | Neural embedding |
|-----------|------------------|
| [N-gram counts](../chapter-23-natural-language-processing/section-01-language-models.md) | Learned dense vectors |
| Manual synonym lists (WordNet) | Emergent similarity |
| Sparse TF-IDF | Dense semantic space |

The Chapter 24 lab compares DistilBERT finetuning against [bag-of-words baselines](https://github.com/Collaborative-ai/ai-academy-applied-ml-engineering/blob/main/chapters/chapter-04-text-classification/section-03-bag-of-words-and-tf-idf.md) - embeddings are the bridge.

---

## Key Terms (Glossary)

- **word embedding** - see [../../GLOSSARY.md](../../GLOSSARY.md)
- **Word2Vec** - see [../../GLOSSARY.md](../../GLOSSARY.md)
- **GloVe** - see [../../GLOSSARY.md](../../GLOSSARY.md)

---

## Reflection Questions

1. Why can't one-hot vectors express "cat is closer to dog than to car"?
2. How does negative sampling make Word2Vec training tractable?
3. When would frozen GloVe embeddings outperform training from scratch?
4. How do subword tokenizations solve the OOV problem for static embeddings?

---

**Previous:** [Chapter 23 - Natural Language Processing](../chapter-23-natural-language-processing/README.md) · **Next:** [Section 24.2 - Neural Language Models](./section-02-neural-language-models.md)

---

## References

1. Russell, S., & Norvig, P. (2020). *Artificial Intelligence: A Modern Approach* (4th ed.). [aima.cs.berkeley.edu](https://aima.cs.berkeley.edu/)
2. Russell, S., & Norvig, P. - [aima-python](https://github.com/aimacode/aima-python)
