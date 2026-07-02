# Section 12.6: Naive Bayes Classifier

> **Source:** Russell & Norvig, *Artificial Intelligence: A Modern Approach* (4th ed.), Chapter 12.6  
> **Previous:** [Section 12.5 - Bayes' Rule Applications](./section-05-bayes-rule-applications.md)  
> **Vocabulary:** [GLOSSARY.md](../../GLOSSARY.md)  
> **Math conventions:** [MATH_CONVENTIONS.md](../../MATH_CONVENTIONS.md)

---


## Classification with Bayes

Given features $\mathbf{F} = (F_1, \ldots, F_n)$ and class variable $C$, pick:

$$
c^* = \arg\max_c P(c \mid \mathbf{f}) = \arg\max_c P(c) \prod_i P(f_i \mid c)
$$
> **Readable form:** Choose the class that makes the observed features most probable (times prior).

We need not compute $P(\mathbf{f})$ - it is the same for all classes. The **naive** conditional independence assumption from [Section 12.4](./section-04-independence.md) makes training tractable.

---

## Training: Count and Normalize

From labeled data, estimate:

$$
P(c) = \frac{\text{count}(c)}{N}, \qquad P(f_i \mid c) = \frac{\text{count}(f_i, c)}{\text{count}(c)}
$$
> **Readable form:** The expression assigns probability to the event or value using the stated model assumptions.

For text: $f_i$ = word $i$ present or word identity in bag-of-words model. Smoothing handles unseen words.

---

## Laplace Smoothing

Without smoothing, a unseen word in class $c$ zeroes $P(c \mid \mathbf{f})$. **Add-one (Laplace) smoothing**:

$$
P(f_i \mid c) = \frac{\text{count}(f_i, c) + 1}{\text{count}(c) + |V|}
$$
> **Readable form:** The expression assigns probability to the event or value using the stated model assumptions.

where $|V|$ is vocabulary size. Other pseudocounts are possible; smoothing trades bias for robustness.

---

## Log Probabilities

Products of many small probabilities **underflow** floating point. Work in log space:

$$
\log P(c \mid \mathbf{f}) = \log P(c) + \sum_i \log P(f_i \mid c) + \text{const}
$$
> **Readable form:** The probability is obtained by summing over the hidden cases or outcomes that satisfy the query.

```python
import math
from collections import defaultdict

class NaiveBayes:
    def __init__(self, alpha=1.0):
        self.alpha = alpha
        self.class_count = defaultdict(int)
        self.feature_count = defaultdict(lambda: defaultdict(int))
        self.vocab = set()

    def train(self, docs, labels):
        for words, label in zip(docs, labels):
            self.class_count[label] += 1
            for w in words:
                self.vocab.add(w)
                self.feature_count[label][w] += 1

    def log_prob(self, words, label):
        n = sum(self.class_count.values())
        log_p = math.log(self.class_count[label] / n)
        denom = self.class_count[label] + self.alpha * len(self.vocab)
        for w in words:
            num = self.feature_count[label][w] + self.alpha
            log_p += math.log(num / denom)
        return log_p

    def predict(self, words):
        return max(self.class_count, key=lambda c: self.log_prob(words, c))
```

---

## Text Classification Workflow

1. Tokenize and lowercase email body.
2. Represent as bag-of-words or binary word features.
3. Train per-class word frequencies with smoothing.
4. Classify new email by argmax log posterior.
5. Evaluate precision/recall on held-out set.

Connects to Course 1 logistic regression - both are linear classifiers in log-feature space for naive Bayes with binary features.

---

## Feature Selection and Naive Bayes

Because naive Bayes is fast, it is often used for **baseline** text classification and feature screening. Mutual information between word and class helps rank features before training richer models.

---

## Multinomial vs. Bernoulli Event Models

| Model | Feature | Typical use |
|-------|---------|-------------|
| Bernoulli | Word present/absent | Short texts |
| Multinomial | Word counts | Long documents |

Multinomial naive Bayes uses word frequency counts; likelihoods come from per-class word probabilities normalized over corpus length.

---

## Calibration

Naive Bayes posteriors are often **overconfident** due to violated independence. **Platt scaling** or isotonic regression can calibrate probabilities post-hoc - important when probabilities drive decisions, not just labels.

---

## Lab Connection

This chapter's lab trains a spam classifier on a public email corpus. Steps: split train/test, implement counts + smoothing, report precision/recall, ablate smoothing to show zero-probability failures.

---

## Comparison Table: Generative vs. Discriminative

| | Naive Bayes | Logistic regression |
|---|-------------|---------------------|
| Type | Generative | Discriminative |
| Models | $P(X,Y)$ | $P(Y \mid X)$ |
| Training | Count features | Gradient on loss |
| Missing data | Easy | Harder |

Course 1 covers logistic regression; naive Bayes is the probabilistic generative counterpart.

---

## Key Takeaways

1. Naive Bayes picks the class maximizing prior × product of likelihoods.
2. Training is counting; prediction is argmax in log space.
3. Laplace smoothing prevents zero probabilities for unseen features.
4. Text spam filtering is the flagship application.
5. Independence is wrong but often useful - a bias-variance trade-off.



## Study Guide

Work through each equation with the readable-form blockquote, then reproduce the main formulas from memory before opening the next section.

---

## Notation Review

| Symbol | Meaning |
|--------|---------|
| $P(X)$ | Prior or marginal |
| $P(X \mid e)$ | Posterior given evidence |
| $P(X,Y)$ | Joint probability |

---

## Worked Drill

1. State the main definition.
2. Apply to a numeric toy.
3. Run the Python snippet.
4. List one common mistake.
5. Link to the **Next** section.

---

## Practice Problems

1. Define glossary terms from memory.
2. Compute a small numerical exercise.
3. Compare to [Chapter 07](../chapter-07-logical-agents/README.md) logical reasoning.
4. Preview how [Chapter 13](../chapter-13-probabilistic-reasoning/README.md) builds on this idea.

---

## Engineering Note

Calibrate probabilistic models on held-out data; base rates and independence assumptions often break silently in deployment.

---

## Deep Dive

Trace one AIMA figure or example related to this section. Annotate each arrow or table entry with the corresponding probability factor. Teaching others is the fastest way to expose gaps in your own understanding.

---


---

## Reflection Questions

1. Why do we use log probabilities in naive Bayes?
2. What happens without smoothing when a test word never appeared in class spam?
3. Compare naive Bayes to logistic regression for text classification.
4. How would you handle numeric features in naive Bayes?
5. What metrics beyond accuracy matter for spam filters?

---

**Next:** [Section 12.7 - Wumpus Revisited](./section-07-wumpus-revisited.md)

---

## References

1. Russell, S., & Norvig, P. (2020). *Artificial Intelligence: A Modern Approach* (4th ed.). [aima.cs.berkeley.edu](https://aima.cs.berkeley.edu/)
2. Murphy, K. (2022). *Probabilistic Machine Learning*. MIT Press.
3. aimacode/aima-python - logic, probability, planning chapters. [GitHub](https://github.com/aimacode/aima-python)
4. Manning, C., Raghavan, P., & Schütze, H. (2008). *Introduction to Information Retrieval*.

---

## Section Glossary

| Term | Definition |
|------|------------|
| **Naive Bayes classifier** | Classifier using P(C)∏P(Fi|C) with conditional independence of features. |
| **Laplace smoothing** | Add-one pseudocount to avoid zero probability for unseen events. |
| **Bag-of-words** | Text representation ignoring word order; each word is a feature. |
| **Log-likelihood** | Sum of log probabilities; numerically stable alternative to products. |
