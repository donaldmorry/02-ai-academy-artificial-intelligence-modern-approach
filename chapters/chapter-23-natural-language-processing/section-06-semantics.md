# Section 23.6: Semantics

> **Source:** Russell & Norvig, *Artificial Intelligence: A Modern Approach* (4th ed.), Chapter 23.6
> **Previous:** [Section 23.5 - Statistical Parsing](./section-05-statistical-parsing.md)
> **Next:** [Section 23.7 - Pragmatics and Discourse](./section-07-pragmatics-and-discourse.md)
> **Vocabulary:** [../../GLOSSARY.md](../../GLOSSARY.md)
> **Math conventions:** [../../MATH_CONVENTIONS.md](../../MATH_CONVENTIONS.md)

---
## Beyond Syntax

[Parsing](./section-04-parsing.md) answers *how* words combine. **Semantics** answers *what the combination means*. Without semantics, we accept Chomsky's famous sentence:

*"Colorless green ideas sleep furiously"* - grammatical, meaningless.

> **Readable form:** Syntax is sentence anatomy; semantics is what the sentence is actually about.

---

## Lexical Semantics: Word Meanings

**Lexical semantics** studies word-level meaning.

### Word Sense Ambiguity

| Word | Senses |
|------|--------|
| bank | financial institution / river edge |
| bat | animal / sports equipment |
| file | document / tool / verb |

**Word sense disambiguation (WSD)** picks the intended sense from context. Classical approaches:

- **Dictionary methods** - Lesk algorithm overlaps definitions with context
- **Supervised classifiers** - features from surrounding words ([Course 1 Chapter 04](https://github.com/donaldmorry/01-ai-academy-applied-ml-engineering/blob/main/chapters/chapter-04-text-classification/README.md))
- **Embeddings** - nearest sense in vector space ([Chapter 24](../chapter-24-deep-learning-nlp/section-01-word-embeddings.md))

$$
\text{sense}(w) = \arg\max_s P(s \mid \text{context}(w))
$$
> **Readable form:** The expression assigns probability to the event or value using the stated model assumptions.

---

## Compositional Semantics

**Compositionality** (Frege's principle): the meaning of a whole is determined by the meanings of parts and how they combine.

For parse tree with rule $S \rightarrow NP \ VP$:

$$
\llbracket S \rrbracket = \text{combine}(\llbracket NP \rrbracket, \llbracket VP \rrbracket)
$$
> **Readable form:** This definition states exactly how the left-hand quantity is computed from the variables on the right.

where $\llbracket \cdot \rrbracket$ denotes semantic interpretation.

| Constituent | Semantic type (simplified) |
|-------------|---------------------------|
| NP "the cat" | Entity $e_1$ |
| VP "chased the dog" | Relation / event involving $e_1$ |

Without [parse trees](./section-03-grammar-formalisms.md), composition order is undefined - "dog bites man" vs "man bites dog" share bag-of-words features in [Course 1](https://github.com/donaldmorry/01-ai-academy-applied-ml-engineering/blob/main/chapters/chapter-04-text-classification/section-03-bag-of-words-and-tf-idf.md).

---

## Model-Theoretic Semantics

Assign denotations in a **model** $\mathcal{M} = (D, I)$:

- $D$ - domain of entities
- $I$ - interpretation function mapping constants to entities, predicates to relations

Example:

- $I(\text{cat}) = c_1 \in D$
- $I(\text{chased}) = \{ (a,b) : a \text{ chased } b \} \subseteq D \times D$

Sentence "the cat chased the dog" is true iff $(c_1, d_1) \in I(\text{chased})$.

> **Readable form:** Model-theoretic semantics asks whether the world described by facts makes the sentence true.

---

## Lambda Calculus Preview

**Lambda calculus** provides a formal language for meaning composition - bridging [logic](../chapter-07-logical-agents/README.md) and NLP.

| Expression | Meaning |
|------------|---------|
| $\lambda x.\, \text{cat}(x)$ | Property of being a cat |
| $\lambda x.\, \exists y.\, \text{chased}(x, y)$ | Property of having chased something |

**Function application** combines modifiers:

$$
(\lambda x.\, P(x))(a) \rightarrow P(a)
$$
> **Readable form:** The arrows list an ordered path or transformation sequence; each step follows from the previous one.

**Example:** Adjective-noun composition

$$
\llbracket \text{big cat} \rrbracket = \llbracket \text{big} \rrbracket(\llbracket \text{cat} \rrbracket)
$$
> **Readable form:** This definition states exactly how the left-hand quantity is computed from the variables on the right.

Full Montague grammar maps syntactic rules to lambda terms - elegant but heavy. Neural models learn implicit semantic composition without explicit lambda terms.

---

## Semantic Roles and Frames

**Semantic role labeling (SRL)** identifies who did what to whom:

| Role | Example in "John gave Mary a book" |
|------|-------------------------------------|
| Agent | John |
| Recipient | Mary |
| Theme | a book |

**Frame semantics** (FrameNet) groups related predicates and roles - "commercial transaction" frame includes buyer, seller, goods, money.

These representations power question answering and information extraction.

---

## Entailment and Contradiction

**Textual entailment:** Does sentence $A$ imply sentence $B$?

- $A$: "The cat is on the mat"
- $B$: "An animal is on the mat" - **entailed**
- $C$: "The cat is under the mat" - **contradicted**

Natural Language Inference (NLI) datasets (SNLI, MultiNLI) train classifiers - [supervised learning](../chapter-19-learning-from-examples/README.md) on sentence pairs.

[Chapter 24, Section 5](../chapter-24-deep-learning-nlp/section-05-bert-and-pretraining.md) finetunes BERT on NLI for semantic understanding.

---

## Knowledge Representation Link

Semantics connects NLP to [knowledge representation](../chapter-10-knowledge-representation/README.md):

```
chased(John, Mary) → Event(e1), agent(e1, John), theme(e1, Mary)
```

First-order logic ([Chapter 08](../chapter-08-first-order-logic/README.md)) can express facts extracted from text - enabling [logical agents](../chapter-07-logical-agents/README.md) to reason over language-derived knowledge.

---

## Distributional Semantics Bridge

**"You shall know a word by the company it keeps"** - Firth (1957).

Words appearing in similar contexts have similar meanings. This idea underpins:

- [N-gram co-occurrence](./section-01-language-models.md)
- [Word2Vec, GloVe](../chapter-24-deep-learning-nlp/section-01-word-embeddings.md)
- Transformer contextual embeddings

[Course 1 Chapter 13](https://github.com/donaldmorry/01-ai-academy-applied-ml-engineering/blob/main/chapters/chapter-13-natural-language-processing/README.md) demonstrates that "king" - "man" + "woman" ≈ "queen" in embedding space - a soft form of lexical semantics.

---

## Semantic Parsing

**Semantic parsing** maps sentences directly to logical forms or executable programs:

| Input | Output |
|-------|--------|
| "How many cats are in Boston?" | `count(x. cat(x) ∧ location(x, Boston))` |
| "Book a table for two at 7pm" | API call with structured slots |

Used in virtual assistants, database query interfaces, and robot command languages.

---

## Evaluation

| Task | Metric |
|------|--------|
| WSD | Accuracy vs gold senses |
| SRL | F1 on role spans |
| NLI | Accuracy on entailment labels |
| Semantic parsing | Exact match on logical form |

Semantic tasks are **harder to annotate** than syntax - inter-annotator agreement drops. Neural models improved benchmarks but **grounding** (connecting symbols to world) remains open.

---

## Limitations of Classical Semantics

| Challenge | Example |
|-----------|---------|
| Figurative language | "It's raining cats and dogs" |
| Implicit meaning | "Can you pass the salt?" (request, not question) |
| World knowledge | "The trophy didn't fit in the suitcase because it was too big" - what is "it"? |
| Cultural context | Idioms, sarcasm |

These push us toward [pragmatics](./section-07-pragmatics-and-discourse.md) and world-grounded models.

> **Humorous analogy:** Syntax is the grammar checker; semantics is the fact checker; pragmatics is knowing your friend said "nice haircut" with sarcasm.

---

## Additional Notes

This section's extra practice: take one sentence with ambiguity, write two possible meanings, and state what evidence would disambiguate it. Good semantic systems connect tokens to entities, relations, events, and user intent instead of stopping at surface form.

---


## Key Terms (Glossary)

- **word sense** - see [../../GLOSSARY.md](../../GLOSSARY.md)
- **compositionality** - see [../../GLOSSARY.md](../../GLOSSARY.md)
- **lambda calculus** - see [../../GLOSSARY.md](../../GLOSSARY.md)

---

## Reflection Questions

1. Why does compositional semantics require a parse tree?
2. How does word sense disambiguation relate to [text classification](https://github.com/donaldmorry/01-ai-academy-applied-ml-engineering/blob/main/chapters/chapter-04-text-classification/README.md)?
3. What is the difference between truth-conditional semantics and distributional semantics?
4. Give an example where syntactic parses match but semantic interpretations differ.

---

**Previous:** [Section 23.5 - Statistical Parsing](./section-05-statistical-parsing.md) · **Next:** [Section 23.7 - Pragmatics and Discourse](./section-07-pragmatics-and-discourse.md)

---

## References

1. Russell, S., & Norvig, P. (2020). *Artificial Intelligence: A Modern Approach* (4th ed.). [aima.cs.berkeley.edu](https://aima.cs.berkeley.edu/)
2. Russell, S., & Norvig, P. - [aima-python](https://github.com/aimacode/aima-python)
