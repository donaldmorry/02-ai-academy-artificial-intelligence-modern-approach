# Section 23.4: Parsing

> **Source:** Russell & Norvig, *Artificial Intelligence: A Modern Approach* (4th ed.), Chapter 23.4
> **Previous:** [Section 23.3 - Grammar Formalisms](./section-03-grammar-formalisms.md)
> **Next:** [Section 23.5 - Statistical Parsing](./section-05-statistical-parsing.md)
> **Vocabulary:** [../../GLOSSARY.md](../../GLOSSARY.md)
> **Math conventions:** [../../MATH_CONVENTIONS.md](../../MATH_CONVENTIONS.md)

---
## Parsing: Recognition and Structure

**Parsing** assigns syntactic structure to a token sequence. Two related problems:

| Problem | Input | Output |
|---------|-------|--------|
| **Recognition** | String $w$, grammar $G$ | Is $w \in L(G)$? |
| **Parsing** | String $w$, grammar $G$ | Parse tree(s) for $w$ |

[Section 23.3](./section-03-grammar-formalisms.md) defined CFGs; this section covers algorithms that use them. [PCFGs](./section-05-statistical-parsing.md) add probabilities to disambiguate multiple trees.

> **Readable form:** Parsing is diagramming sentences with an algorithm instead of a colored pencil.

---

## Top-Down Parsing

**Top-down** parsers start at the start symbol $S$ and try to derive the input left-to-right.

**Recursive descent:** each nonterminal has a function that expands it according to grammar rules.

```
parse_S():
    parse_NP()
    parse_VP()
```

**Problems:**

- **Left recursion** causes infinite loops ($NP \rightarrow NP \ PP$)
- **Backtracking** explodes on ambiguous grammars
- Predictive parsers need **FIRST/FOLLOW** sets to choose rules without backtracking

Top-down suits well-designed grammars; fragile on natural language scale.

---

## Bottom-Up Parsing

**Bottom-up** parsers start at words and combine them into larger constituents until $S$ spans the sentence.

**Shift-reduce parsing:**

| Action | Effect |
|--------|--------|
| **Shift** | Push next token onto stack |
| **Reduce** | Pop RHS of rule; push LHS nonterminal |

Example trace for "the cat sat":

```
Stack          Input         Action
               the cat sat   shift
the            cat sat       reduce Det→the
Det            cat sat       shift
Det cat        sat           reduce N→cat
Det N          sat           reduce NP→Det N
NP             sat           shift
NP sat                       reduce V→sat
NP V                           reduce VP→V
NP VP                          reduce S→NP VP
S                              accept
```

LR parsers (used in compiler design) systematize bottom-up parsing with parse tables.

---

## Chart Parsing

**Chart parsers** fill a data structure (chart) with partial parses - avoiding redundant re-analysis of substrings.

For substring $w_{i:j}$ (words $i$ through $j-1$), store all constituents spanning that range.

**Earley parser** - $O(n^3)$ for CFGs in general, $O(n)$ for unambiguous grammars - interleaves top-down prediction with bottom-up confirmation.

Chart parsing is the backbone of many practical CFG parsers before neural dominance.

---

## The CYK Algorithm

The **Cocke-Younger-Kasami (CYK)** algorithm parses strings using grammars in **Chomsky Normal Form (CNF)**.

**Data structure:** table $T[i, j]$ = set of nonterminals that can generate substring $w_{i:j}$ (0-indexed, $j$ exclusive).

**Algorithm:**

1. **Base case:** For each position $i$, if rule $A \rightarrow w_i$ exists, add $A$ to $T[i, i+1]$
2. **Fill by length:** For span length $\ell = 2$ to $n$:
   - For each split $k$ with $i < k < j$:
     - If $B \in T[i,k]$ and $C \in T[k,j]$ and $A \rightarrow BC \in R$, add $A$ to $T[i,j]$
3. **Accept** if $S \in T[0, n]$

**Time complexity:** $O(n^3 \cdot |G|)$

> **Readable form:** CYK fills a triangular table - each cell lists what grammatical category could produce that substring. Bottom row is single words; top cell is the whole sentence.

---

## CYK Walkthrough

Grammar (CNF):

```
S → NP VP    NP → Det N    VP → V
Det → the    N → cat       V → sat
```

Input: `["the", "cat", "sat"]`

| Span | Cell | Contents |
|------|------|----------|
| [0:1] | the | {Det} |
| [1:2] | cat | {N} |
| [2:3] | sat | {V} |
| [0:2] | the cat | {NP} (Det+N) |
| [1:3] | cat sat | {VP}? only V - no VP→N V rule |
| [2:3] | sat | {V} |
| [0:3] | full | {S} via NP[0:2] + VP[2:3] if VP→V |

With $VP \rightarrow V$, we get $VP$ at [2:3], $NP$ at [0:2], hence $S$ at [0:3].

```python
def cyk(tokens, grammar, start="S"):
    n = len(tokens)
    table = [[set() for _ in range(n + 1)] for _ in range(n)]
    # terminals
    for i, tok in enumerate(tokens):
        for (lhs, rhs) in grammar:
            if len(rhs) == 1 and rhs[0] == tok:
                table[i][i + 1].add(lhs)
    # spans
    for length in range(2, n + 1):
        for i in range(n - length + 1):
            j = i + length
            for k in range(i + 1, j):
                for (lhs, b, c) in grammar:
                    if (b, c) == tuple(sorted([b, c])):  # simplified
                        if b in table[i][k] and c in table[k][j]:
                            table[i][j].add(lhs)
    return start in table[0][n], table
```

---

## Recovering Parse Trees

CYK recognition tells us *if* a parse exists. To recover trees, store **backpointers** during table fill: which split $(k)$ and rule $(A \rightarrow BC)$ produced each entry.

Backtrace from $S \in T[0,n]$ to leaves - yields one or more parse trees for ambiguous inputs.

Visualization libraries (NLTK `Tree`, Graphviz) render trees for debugging.

---

## Complexity Comparison

| Algorithm | Time | Space | Notes |
|-----------|------|-------|-------|
| CYK | $O(n^3)$ | $O(n^2)$ | Requires CNF |
| Earley | $O(n^3)$ worst | $O(n^2)$ | Handles any CFG |
| Shift-reduce | $O(n)$ best | $O(n)$ | Grammar-dependent |
| Neural parser | $O(n)$ amortized | Model size | Learned from treebanks |

For sentences under 50 words, $n^3$ is trivial. For megabyte inputs, parsing is chunked by [sentence segmentation](./section-02-text-processing.md).

---

## Parsing Failures

| Failure mode | Cause | Response |
|--------------|-------|----------|
| No parse | Ungrammatical input or grammar gap | Robust parsing, error recovery |
| Too many parses | Ambiguity | PCFG ranking ([Section 23.5](./section-05-statistical-parsing.md)) |
| Garden path | Human-like late disambiguation | Incremental parsers |

Real pipelines tolerate noise - social media breaks hand-crafted grammars. Statistical and neural parsers generalize better.

---

## Connection to Machine Learning

Classical parsing is **search** over structures defined by [grammar rules](./section-03-grammar-formalisms.md) - connecting to AIMA's search chapters (Chapters 03-04).

[Course 3 Chapter 10](https://github.com/donaldmorry/03-ai-academy-deep-learning-foundations/blob/main/chapters/chapter-10-sequence-modeling/README.md) covers sequence models that implicitly learn parse-like structure. [Chapter 24](../chapter-24-deep-learning-nlp/README.md) transformers attend over tokens without explicit CYK tables - but CYK remains the clearest introduction to **structured prediction**.

[Course 1 Chapter 04](https://github.com/donaldmorry/01-ai-academy-applied-ml-engineering/blob/main/chapters/chapter-04-text-classification/README.md) never parses - classification operates on flat vectors. Adding a parse tree as features (production rules as dimensions) was a 1990s-2000s technique before embeddings.

---

## Lab Connection

The Chapter 23 lab implements CYK on a toy grammar and visualizes parse trees. Steps:

1. Convert grammar to CNF
2. Run CYK on tokenized sentence
3. Backtrace to tree
4. Compare with NLTK `ChartParser` on same input

---

## Key Terms (Glossary)

- **CYK** - see [../../GLOSSARY.md](../../GLOSSARY.md)
- **chart parsing** - see [../../GLOSSARY.md](../../GLOSSARY.md)
- **parse tree** - see [../../GLOSSARY.md](../../GLOSSARY.md)

---

## Reflection Questions

1. Why does CYK require Chomsky Normal Form?
2. What is the time complexity of CYK for a sentence of length 20? Length 100?
3. How does chart parsing avoid redundant work compared to naive backtracking?
4. When would a neural parser replace CYK in production - and when would CYK still matter?

---

**Previous:** [Section 23.3 - Grammar Formalisms](./section-03-grammar-formalisms.md) · **Next:** [Section 23.5 - Statistical Parsing](./section-05-statistical-parsing.md)

---

## References

1. Russell, S., & Norvig, P. (2020). *Artificial Intelligence: A Modern Approach* (4th ed.). [aima.cs.berkeley.edu](https://aima.cs.berkeley.edu/)
2. Russell, S., & Norvig, P. - [aima-python](https://github.com/aimacode/aima-python)