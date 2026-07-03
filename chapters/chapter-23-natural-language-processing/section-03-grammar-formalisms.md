# Section 23.3: Grammar Formalisms

> **Source:** Russell & Norvig, *Artificial Intelligence: A Modern Approach* (4th ed.), Chapter 23.3
> **Previous:** [Section 23.2 - Text Processing](./section-02-text-processing.md)
> **Next:** [Section 23.4 - Parsing](./section-04-parsing.md)
> **Vocabulary:** [../../GLOSSARY.md](../../GLOSSARY.md)
> **Math conventions:** [../../MATH_CONVENTIONS.md](../../MATH_CONVENTIONS.md)

---
## Why Grammars?

[Language models](./section-01-language-models.md) score word sequences without explicit structure. **Grammars** describe which sequences are **well-formed** - capturing syntax independent of probability.

A grammar $G$ defines a language $L(G)$ - the set of strings it generates. Parsers ([Section 23.4](./section-04-parsing.md)) use grammars to build **parse trees** showing how words combine into phrases.

> **Readable form:** A grammar is the rulebook for合法 sentence construction - nouns here, verbs there, matching parentheses in language form.

---

## Context-Free Grammars (CFGs)

A **context-free grammar** is a 4-tuple $G = (N, \Sigma, R, S)$:

| Symbol | Meaning |
|--------|---------|
| $N$ | Nonterminal symbols (syntactic categories) |
| $\Sigma$ | Terminal symbols (words) |
| $R$ | Rewrite rules $\alpha \rightarrow \beta$ |
| $S$ | Start symbol |

**Example grammar (simplified English):**

```
S  → NP VP
NP → Det N | Det Adj N | N
VP → V NP | V
Det → the | a
N  → cat | dog | mat
Adj → big | small
V  → sat | chased
```

This generates: "the big cat sat", "a dog chased the cat", etc.

---

## Parse Trees

A **parse tree** shows how rules expand $S$ to terminals:

```
        S
       / \
     NP   VP
    / \    \
  Det  N     V
  |    |     |
 the  cat   sat
```

Each internal node is a nonterminal; leaves are terminals (words). The tree encodes **syntactic structure** - subject NP, predicate VP.

[Course 1 Chapter 04](https://github.com/donaldmorry/01-ai-academy-applied-ml-engineering/blob/main/chapters/chapter-04-text-classification/README.md) discards this structure in bag-of-words models. Grammars recover it.

> **Readable form:** A parse tree is a family tree for words - showing which words group together as phrases.

---

## Ambiguity

Many sentences have **multiple valid parse trees**:

*"I saw the man with the telescope"*

1. I saw [the man [with the telescope]] - man has telescope
2. I [saw [the man]] [with the telescope] - I used telescope

Ambiguity is structural, not probabilistic - both trees may be valid. [PCFGs](./section-05-statistical-parsing.md) assign probabilities to prefer one reading.

Other classic examples:

- *"Flying planes can be dangerous"*
- *"Time flies like an arrow"*

---

## Chomsky Hierarchy

Noam Chomsky classified formal languages by grammar power:

| Type | Grammar | Recognizer | Natural language? |
|------|---------|------------|-------------------|
| 3 | Regular | Finite automaton | Fragments (morphology) |
| 2 | Context-free | Pushdown automaton | Syntax (approx.) |
| 1 | Context-sensitive | Linear bounded automaton | Some constraints |
| 0 | Unrestricted | Turing machine | Full language? |

**Context-free** grammars suffice for much of English phrase structure. Some phenomena (cross-serial dependencies in Dutch, agreement across long distances) may need mild context-sensitivity.

$$
\text{Regular} \subset \text{Context-Free} \subset \text{Context-Sensitive} \subset \text{Recursively Enumerable}
$$
> **Readable form:** Regular grammars handle "ababab..." patterns; context-free grammars handle nested parentheses - and most sentence syntax.

---

## CFG Formal Notation

Rules have the form $A \rightarrow \gamma$ where $A \in N$ and $\gamma \in (N \cup \Sigma)^*$.

**Left-recursive** (problematic for top-down parsers):

$$
NP \rightarrow NP \ PP
$$
> **Readable form:** The arrows list an ordered path or transformation sequence; each step follows from the previous one.

**Right-recursive:**

$$
NP \rightarrow N \ PP
$$
> **Readable form:** The arrows list an ordered path or transformation sequence; each step follows from the previous one.

**Chomsky Normal Form (CNF)** - required for [CYK parsing](./section-04-parsing.md): every rule is either $A \rightarrow BC$ (two nonterminals) or $A \rightarrow a$ (one terminal).

Conversion to CNF introduces dummy nonterminals but preserves the generated language.

---

## Dependency Grammar

An alternative to phrase-structure grammars: **dependency grammar** links words directly.

```
    sat
   / | \
  I  on mat
      |
     the
```

- One word is **head** of each dependency
- Arcs labeled with grammatical relations (subject, object, modifier)
- No explicit phrase nodes

| Phrase-structure (CFG) | Dependency |
|------------------------|------------|
| Tree of constituents | Tree/graph of word relations |
| Standard in Penn Treebank | Standard in Universal Dependencies |
| CYK, chart parsers | Transition-based, graph parsers |

Modern parsers often predict dependencies - especially in multilingual NLP. AIMA emphasizes CFGs; industrial systems frequently use both.

---

## Lexicalized Grammars

Pure CFGs separate syntax from words awkwardly - every noun must be listed under $N$.

**Lexicalized CFGs** attach head words to categories:

$$
\text{NP}(\text{cat}) \rightarrow \text{Det}(\text{the}) \ \text{N}(\text{cat})
$$
> **Readable form:** The arrows list an ordered path or transformation sequence; each step follows from the previous one.

**Tree-adjoining grammars (TAG)** and **combinatory categorial grammar (CCG)** offer more lexical control. These formalisms underpin some statistical parsers but are beyond our core CYK focus.

---

## Grammar Engineering

Hand-writing grammars for full natural language is labor-intensive. Approaches:

| Approach | Description |
|----------|-------------|
| **Linguist-crafted** | Expert-written rules (1980s-90s) |
| **Treebank-induced** | Learn rules from annotated corpora |
| **Neural** | Implicit grammar in latent representations |

The Penn Treebank (~1M words of parsed Wall Street Journal text) enabled **statistical parsing** ([Section 23.5](./section-05-statistical-parsing.md)). [Course 1 Chapter 13](https://github.com/donaldmorry/01-ai-academy-applied-ml-engineering/blob/main/chapters/chapter-13-natural-language-processing/README.md) skips explicit grammars - neural models learn structure from data.

---

## Mini-Grammar for CYK Lab

Your Chapter 23 lab uses a toy grammar in CNF:

```
S  → NP VP
NP → Det N
VP → V NP
VP → V
Det → the | a
N  → cat | dog
V  → chased | sat
```

Convert to CNF if any rule has length > 2 on RHS. Then run CYK ([Section 23.4](./section-04-parsing.md)).

```python
GRAMMAR_CNF = {
    ("S", "NP", "VP"),
    ("NP", "Det", "N"),
    ("VP", "V", "NP"),
    ("VP", "V"),
    ("Det", "the"), ("Det", "a"),
    ("N", "cat"), ("N", "dog"),
    ("V", "chased"), ("V", "sat"),
}
```

---

## Grammars vs Language Models

| Aspect | CFG | N-gram LM |
|--------|-----|-----------|
| Question answered | Is it grammatical? | How likely is it? |
| Output | Parse tree | Probability |
| Ungrammatical strings | Rejected | May get low P |
| Grammatical nonsense | Accepted | May get high P |

*"Colorless green ideas sleep furiously"* - grammatical (Chomsky 1957) but semantically absurd. Grammars and LMs are **complementary**.

[Chapter 24](../chapter-24-deep-learning-nlp/README.md) neural models blur the line - transformers learn both fluency and partial syntax without explicit CFGs.

---

## Connection to Semantics

Parse trees feed [semantic analysis](./section-06-semantics.md). **Compositional semantics** builds meaning bottom-up:

$$
\text{meaning}(S) = \text{combine}(\text{meaning}(NP), \text{meaning}(VP))
$$
> **Readable form:** This definition states exactly how the left-hand quantity is computed from the variables on the right.

Without a parse tree, composition order is ambiguous - another reason grammars matter in classical NLP.

---

## Key Terms (Glossary)

- **CFG** - see [../../GLOSSARY.md](../../GLOSSARY.md)
- **dependency grammar** - see [../../GLOSSARY.md](../../GLOSSARY.md)
- **Chomsky hierarchy** - see [../../GLOSSARY.md](../../GLOSSARY.md)

---

## Reflection Questions

1. Give an example of structural ambiguity not resolved by an n-gram model alone.
2. Why must grammars be converted to CNF before CYK parsing?
3. How does dependency grammar represent the subject of a sentence differently from CFG?
4. What information does a parse tree provide that [bag-of-words](https://github.com/donaldmorry/01-ai-academy-applied-ml-engineering/blob/main/chapters/chapter-04-text-classification/section-03-bag-of-words-and-tf-idf.md) discards?

---

**Previous:** [Section 23.2 - Text Processing](./section-02-text-processing.md) · **Next:** [Section 23.4 - Parsing](./section-04-parsing.md)

---

## References

1. Russell, S., & Norvig, P. (2020). *Artificial Intelligence: A Modern Approach* (4th ed.). [aima.cs.berkeley.edu](https://aima.cs.berkeley.edu/)
2. Russell, S., & Norvig, P. - [aima-python](https://github.com/aimacode/aima-python)



