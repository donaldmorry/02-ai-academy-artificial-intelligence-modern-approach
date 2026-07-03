# Section 23.5: Statistical Parsing

> **Source:** Russell & Norvig, *Artificial Intelligence: A Modern Approach* (4th ed.), Chapter 23.5
> **Previous:** [Section 23.4 - Parsing](./section-04-parsing.md)
> **Next:** [Section 23.6 - Semantics](./section-06-semantics.md)
> **Vocabulary:** [../../GLOSSARY.md](../../GLOSSARY.md)
> **Math conventions:** [../../MATH_CONVENTIONS.md](../../MATH_CONVENTIONS.md)

---
## From Recognition to Ranking

[CYK parsing](./section-04-parsing.md) returns all parse trees consistent with a CFG. Ambiguous sentences may yield dozens of trees. **Statistical parsing** assigns probabilities to trees and selects the most likely one.

A **probabilistic context-free grammar (PCFG)** augments each rule with a probability:

$$
A \rightarrow \beta \quad [p]
$$
> **Readable form:** Nonterminal $A$ rewrites as sequence $\beta$ with probability $p$.

where $p = P(\beta \mid A)$ and $\sum_{\beta} P(\beta \mid A) = 1$ for each nonterminal $A$.

> **Readable form:** A PCFG is a grammar where each rewrite rule comes with a likelihood - like weighted recipe steps.

---

## Probability of a Parse Tree

For tree $T$ with rules $r_1, r_2, \ldots, r_k$:

$$
P(T) = \prod_{i=1}^{k} P(r_i)
$$
> **Readable form:** The joint probability is built by multiplying the conditional factors specified by the model.

**PCFG assumption:** rule probabilities depend only on the parent nonterminal (context-free), not on siblings or distant nodes.

For sentence $w$ with parse $T$:

$$
P(w, T) = P(T) \cdot \mathbb{1}[T \text{ yields } w]
$$
> **Readable form:** The expression assigns probability to the event or value using the stated model assumptions.

Marginal over trees:

$$
P(w) = \sum_{T \in \text{parse}(w)} P(T)
$$
> **Readable form:** The probability is obtained by summing over the hidden cases or outcomes that satisfy the query.

---

## Estimating PCFG Parameters

From a **treebank** (corpus of parse trees, e.g., Penn Treebank):

$$
P(\beta \mid A) = \frac{\text{count}(A \rightarrow \beta)}{\text{count}(A)}
$$
> **Readable form:** The arrows list an ordered path or transformation sequence; each step follows from the previous one.

**Example:**

| Rule | Count |
|------|-------|
| VP → V NP | 40 |
| VP → V | 10 |
| VP → ... | ... |

$$
P(VP \rightarrow V \ NP) = \frac{40}{40 + 10 + \cdots}
$$
> **Readable form:** The arrows list an ordered path or transformation sequence; each step follows from the previous one.

Smoothing handles unseen rules - same sparsity issues as [n-gram models](./section-01-language-models.md).

---

## Disambiguation Example

*"I saw the man with the telescope"*

| Parse | Rough probability factors |
|-------|---------------------------|
| VP → V NP (NP has PP) | $P(VP \rightarrow V \ NP) \cdot P(NP \rightarrow NP \ PP) \cdot \cdots$ |
| VP → VP PP (saw with telescope) | $P(VP \rightarrow VP \ PP) \cdot \cdots$ |

The tree with higher product of rule probabilities wins. PP attachment preferences are **learned from data** - not hand-coded.

> **Readable form:** Statistical parsing picks the parse tree that multiplies to the highest probability - each grammar rule contributes its learned weight.

---

## Parsing as Search for Max Tree

**Goal:** Find $\arg\max_T P(T)$ among trees yielding input $w$.

Algorithms:

| Method | Approach |
|--------|----------|
| **CYK + Viterbi-style** | Store best probability per cell |
| **A\*** parsing | Heuristic-guided search |
| **CKY inside-outside** | Compute $P(w)$ exactly |

**Viterbi CYK** modification: in each cell, keep the best probability and backpointer (not just membership).

$$
T[i,j,A] = \max_{A \rightarrow B \ C,\, k} T[i,k,B] \cdot T[k,j,C] \cdot P(A \rightarrow B \ C)
$$
> **Readable form:** The arrows list an ordered path or transformation sequence; each step follows from the previous one.

Complexity remains $O(n^3)$.

---

## Lexicalized PCFGs

Pure PCFGs struggle with lexical dependencies - *"ate"* prefers an edible object.

**Lexicalized PCFGs** tag nonterminals with head words:

$$
P(VP(\text{ate}) \rightarrow V(\text{ate}) \ NP(\text{pizza}))
$$
> **Readable form:** The arrows list an ordered path or transformation sequence; each step follows from the previous one.

Parameters explode but **parent annotation** and **head lexicalization** dramatically improve accuracy.

Modern neural parsers largely supersede lexicalized PCFGs, but the probabilistic framework persists in understanding parser outputs.

---

## Evaluation Metrics

Compare system parses to gold treebank trees:

| Metric | Measures |
|--------|----------|
| **PARSEVAL** labeled precision/recall | Correct constituents with label |
| **F1** | Harmonic mean of precision and recall |
| **Attachment score** | Correct dependency links |

**Labeled Attachment Score (LAS)** - standard for dependency parsing.

A PCFG baseline on Penn Treebank might achieve ~75% F1; state-of-the-art neural parsers exceed 95%.

---

## Generative Story

PCFGs are **generative models**:

1. Start at $S$
2. Recursively rewrite nonterminals according to rule probabilities
3. Emit terminals when rules produce them

This mirrors [n-gram language models](./section-01-language-models.md) - PCFGs generate both structure and words. **Discriminative parsers** (neural) directly model $P(\text{tree} \mid \text{sentence})$ without a generative story.

[Chapter 19 - Learning from Examples](../chapter-19-learning-from-examples/README.md) contrasts generative vs discriminative learning - PCFGs are the NLP instance.

---

## Connection to Neural Parsing

| Era | Model | Features |
|-----|-------|----------|
| 1990s | PCFG | Manual grammar + EM |
| 2000s | Lexicalized PCFG | Treebank statistics |
| 2010s | RNN parsers | Distributed word vectors |
| 2020s | Transformer parsers | Self-attention over tokens |

[Course 3 Chapter 10](https://github.com/donaldmorry/03-ai-academy-deep-learning-foundations/blob/main/chapters/chapter-10-sequence-modeling/README.md) builds sequence models; [Chapter 24](../chapter-24-deep-learning-nlp/README.md) applies transformers to parsing via finetuning (BERT on constituency or dependency tasks).

[Course 1 Chapter 13](https://github.com/donaldmorry/01-ai-academy-applied-ml-engineering/blob/main/chapters/chapter-13-natural-language-processing/README.md) skips PCFGs - jumping to neural end-to-end models. Understanding PCFGs explains **what neural parsers learned to approximate**.

---

## Inside-Outside Algorithm

The **inside-outside** algorithm computes $P(w)$ for a PCFG in $O(n^3)$ - analogous to forward-backward in HMMs ([Chapter 14](../chapter-14-probabilistic-reasoning-time/README.md)).

- **Inside** $\beta_{i,j}(A)$: probability of generating $w_{i:j}$ from $A$
- **Outside** $\alpha_{i,j}(A)$: probability of generating outside spans given $A$ generates $w_{i:j}$

Used for EM training when trees are partially observed - foundational but rarely implemented from scratch today.

---

## PCFG Limitations

| Limitation | Example |
|------------|---------|
| Context-free independence | Agreement: "The cats *is*" vs "The cat *is*" |
| Markovian rules | Long-distance dependencies |
| Treebank bias | News text grammar ≠ social media |
| No semantics | Syntactically valid, semantically odd |

These motivated richer models - but PCFGs remain pedagogically essential and appear inside speech recognition pipelines.

---

## Code Sketch: Rule Probability Lookup

```python
from collections import defaultdict

def estimate_pcfg_rules(trees):
    counts = defaultdict(int)
    lhs_counts = defaultdict(int)
    for tree in trees:
        for rule in tree.productions():
            lhs, rhs = rule.lhs(), tuple(rule.rhs())
            counts[(lhs, rhs)] += 1
            lhs_counts[lhs] += 1
    return {
        (lhs, rhs): counts[(lhs, rhs)] / lhs_counts[lhs]
        for (lhs, rhs) in counts
    }

def tree_probability(tree, rule_probs):
    p = 1.0
    for rule in tree.productions():
        p *= rule_probs.get((rule.lhs(), tuple(rule.rhs())), 1e-10)
    return p
```

---

## Key Terms (Glossary)

- **PCFG** - see [../../GLOSSARY.md](../../GLOSSARY.md)
- **constituency** - see [../../GLOSSARY.md](../../GLOSSARY.md)
- **parse probability** - see [../../GLOSSARY.md](../../GLOSSARY.md)

---

## Reflection Questions

1. Why must PCFG rule probabilities for a given nonterminal sum to 1?
2. How does a PCFG resolve PP attachment ambiguity?
3. What data do you need to estimate $P(NP \rightarrow Det \ N)$?
4. How does discriminative neural parsing differ from the PCFG generative story?

---

**Previous:** [Section 23.4 - Parsing](./section-04-parsing.md) · **Next:** [Section 23.6 - Semantics](./section-06-semantics.md)

---

## References

1. Russell, S., & Norvig, P. (2020). *Artificial Intelligence: A Modern Approach* (4th ed.). [aima.cs.berkeley.edu](https://aima.cs.berkeley.edu/)
2. Russell, S., & Norvig, P. - [aima-python](https://github.com/aimacode/aima-python)
