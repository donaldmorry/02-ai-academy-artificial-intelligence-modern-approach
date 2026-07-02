# Section 23.2: Text Processing

> **Source:** Russell & Norvig, *Artificial Intelligence: A Modern Approach* (4th ed.), Chapter 23.2
> **Previous:** [Section 23.1 - Language Models](./section-01-language-models.md)
> **Next:** [Section 23.3 - Grammar Formalisms](./section-03-grammar-formalisms.md)
> **Vocabulary:** [../../GLOSSARY.md](../../GLOSSARY.md)
> **Math conventions:** [../../MATH_CONVENTIONS.md](../../MATH_CONVENTIONS.md)

---
## The First Step in Every NLP Pipeline

Before grammars, parsers, or neural networks can operate on text, raw strings must become **structured tokens**. Text processing - also called **normalization** - converts heterogeneous input (HTML, tweets, clinical notes) into a consistent symbolic representation.

[Course 1 Chapter 04, Section 4.2](https://github.com/Collaborative-ai/ai-academy-applied-ml-engineering/blob/main/chapters/chapter-04-text-classification/section-02-text-preprocessing.md) introduced practical preprocessing for classification: lowercasing, stop-word removal, stemming. This section formalizes those steps in the AIMA tradition and connects them to [n-gram language models](./section-01-language-models.md) and downstream parsing.

> **Readable form:** Text processing is dishwashing before cooking - skip it and every later step tastes wrong.

---

## Levels of Linguistic Analysis

NLP pipelines stack abstractions from characters to discourse:

| Level | Unit | Example task |
|-------|------|--------------|
| **Morphology** | Subword | Pluralization, tense |
| **Lexicon** | Word | Lemmatization, POS tags |
| **Syntax** | Phrase/sentence | Parsing ([Section 23.4](./section-04-parsing.md)) |
| **Semantics** | Meaning | Word sense ([Section 23.6](./section-06-semantics.md)) |
| **Pragmatics** | Context | Coreference ([Section 23.7](./section-07-pragmatics-and-discourse.md)) |

Text processing spans morphology and lexicon - the foundation layers.

---

## Tokenization

**Tokenization** splits text into tokens (usually words or subwords).

Challenges:

- **Punctuation:** `"don't"` → `["do", "n't"]` or `["don't"]`?
- **Hyphens:** `"state-of-the-art"` → one token or four?
- **Languages without spaces:** Chinese, Japanese require segmentation models
- **URLs, emails, hashtags:** domain-specific rules

```python
import re

def simple_tokenize(text):
    text = text.lower()
    text = re.sub(r"([.!?])", r" \1 ", text)
    return [t for t in re.split(r"\s+", text) if t]

simple_tokenize("Dr. Smith arrived at 3:00 p.m.")
# ['dr.', 'smith', 'arrived', 'at', '3:00', 'p.m.']
```

Production systems use libraries (spaCy, NLTK, Hugging Face tokenizers) with language-specific rules and statistical models.

> **Readable form:** Tokenization decides what counts as a word. There is no universal answer - only conventions your pipeline commits to.

---

## Normalization

Common normalization steps:

| Operation | Effect | Trade-off |
|-----------|--------|-----------|
| Lowercasing | `Apple` → `apple` | Loses named-entity casing |
| Accent stripping | `café` → `cafe` | May lose meaning in some languages |
| Number replacement | `1999` → `<NUM>` | Reduces vocabulary size |
| URL replacement | `http://...` → `<URL>` | Privacy + generalization |

For [text classification](https://github.com/Collaborative-ai/ai-academy-applied-ml-engineering/blob/main/chapters/chapter-04-text-classification/README.md), aggressive normalization often **helps** - sentiment rarely depends on capitalization. For named-entity recognition, it **hurts**.

---

## Stemming and Lemmatization

**Stemming** heuristically chops suffixes:

$$
\text{stem}(\text{running}) \rightarrow \text{run}, \quad \text{stem}(\text{studies}) \rightarrow \text{studi}
$$
> **Readable form:** The arrows list an ordered path or transformation sequence; each step follows from the previous one.

Algorithms: Porter stemmer, Snowball stemmer - fast but irregular.

**Lemmatization** maps to dictionary forms using morphology + POS:

$$
\text{lemma}(\text{running}, \text{VERB}) \rightarrow \text{run}, \quad \text{lemma}(\text{better}, \text{ADJ}) \rightarrow \text{good}
$$
> **Readable form:** The arrows list an ordered path or transformation sequence; each step follows from the previous one.

| Approach | Speed | Accuracy | Needs POS? |
|----------|-------|----------|------------|
| Stemming | Fast | Moderate | No |
| Lemmatization | Slower | Higher | Usually yes |

For n-gram models, stemming reduces sparsity:

$$
C(\text{run}, \text{fast}) \approx C(\text{running}, \text{fast}) + C(\text{runs}, \text{fast}) + \cdots
$$
> **Readable form:** This formal sentence is the symbolic version of the relationship stated in the surrounding explanation.

---

## Stop Words

**Stop words** (the, a, is, of) carry little discriminative content for classification but matter for syntax. Removing them:

- Shrinks vocabulary
- Speeds up bag-of-words models
- **Destroys** grammatical structure needed for parsing

AIMA's classical NLP pipeline typically **keeps** stop words for parsing and n-grams; [Course 1 TF-IDF pipelines](https://github.com/Collaborative-ai/ai-academy-applied-ml-engineering/blob/main/chapters/chapter-04-text-classification/section-03-bag-of-words-and-tf-idf.md) often remove them.

> **Readable form:** Stop-word removal is a classification trick, not a universal law of language.

---

## Sentence Segmentation

Long documents must be split into sentences for parsers and summarizers.

Rule-based: split on `.!?` with exceptions for `Dr.`, `U.S.A.`, `3.14`.

Statistical: classify punctuation as sentence boundary or not (common in spaCy).

$$
\text{Doc} \rightarrow \{s_1, s_2, \ldots, s_m\}
$$
> **Readable form:** The arrows list an ordered path or transformation sequence; each step follows from the previous one.

Each sentence feeds the [CYK parser](./section-04-parsing.md) independently or with cross-sentence discourse links ([Section 23.7](./section-07-pragmatics-and-discourse.md)).

---

## Subword Tokenization Preview

Neural models ([Chapter 24](../chapter-24-deep-learning-nlp/README.md)) rarely use whole words. **Byte Pair Encoding (BPE)** iteratively merges frequent character pairs:

```
Corpus: low, lower, newest, widest
Merge 'e'+'s' → 'es', 'es'+'t' → 'est', ...
Vocabulary: {low, er, new, est, wid, ...}
```

**WordPiece** and **SentencePiece** (used by BERT, GPT) apply similar ideas with different merge criteria.

Benefits:

- Open vocabulary - rare words decompose into known pieces
- Smaller |V| than word-level models
- Cross-lingual transfer (shared subwords)

[Course 1 Chapter 13](https://github.com/Collaborative-ai/ai-academy-applied-ml-engineering/blob/main/chapters/chapter-13-natural-language-processing/README.md) uses Keras `TextVectorization`; Hugging Face tokenizers implement BPE/WordPiece for transformers.

$$
\text{tokenize}(\text{unhappiness}) \rightarrow [\text{un}, \text{##happi}, \text{##ness}]
$$
> **Readable form:** Subword tokenization breaks unfamiliar long words into familiar chunks - like sounding out "un-happi-ness."

---

## Part-of-Speech Tagging Preview

**POS tagging** assigns grammatical categories:

```
The/DT cat/NN sat/VBD on/IN the/DT mat/NN
```

Tags follow conventions (Penn Treebank: NN=noun, VB=verb, DT=determiner).

Algorithms:

- **HMM / MEMM** - sequence models with tag transition probabilities
- **CRF** - global sequence optimization
- **Neural taggers** - bidirectional LSTMs, now often transformer-based

POS tags enable lemmatization, parsing, and feature engineering. [Section 23.8](./section-08-nlp-pipeline-integration.md) places tagging between tokenization and parsing.

---

## Vocabulary Construction

For count-based models, define vocabulary $V$:

1. Count token frequencies in training corpus
2. Keep tokens with count $\geq$ threshold $\theta$
3. Map rare tokens to `<UNK>`

$$
|V| \text{ affects perplexity baseline and memory}
$$
> **Readable form:** This formal sentence is the symbolic version of the relationship stated in the surrounding explanation.

|V| = 50,000 is typical for English news; multilingual models may exceed 250,000 **subword** tokens.

---

## Encoding and Unicode

Real text is Unicode. Normalization forms (NFC, NFD) matter when matching tokens:

- `é` as single codepoint vs. `e` + combining accent
- Emoji, zero-width joiners, right-to-left scripts

Production pipelines apply `unicodedata.normalize('NFC', text)` before tokenization.

---

## Pipeline Sketch

```
Raw text
  → Unicode normalize
  → Sentence split
  → Tokenize
  → (Optional) POS tag
  → (Optional) Lemmatize
  → Vocabulary / subword model
  → Downstream: LM, parser, classifier
```

[Course 1 Chapter 13](https://github.com/Collaborative-ai/ai-academy-applied-ml-engineering/blob/main/chapters/chapter-13-natural-language-processing/README.md) implements this in Keras; [Section 23.8](./section-08-nlp-pipeline-integration.md) wires classical components together.

---

## Evaluation and Consistency

**Golden rule:** Apply identical preprocessing at train and test time.

| Mistake | Consequence |
|---------|-------------|
| Fit vocabulary on test data | Data leakage |
| Different tokenization train vs deploy | Vocabulary mismatch |
| Stem train, no stem test | Broken features |

Document your preprocessing spec - future you (and Chapter 24 finetuning) will thank you.

---

## Key Terms (Glossary)

- **tokenization** - see [../../GLOSSARY.md](../../GLOSSARY.md)
- **lemmatization** - see [../../GLOSSARY.md](../../GLOSSARY.md)
- **BPE** - see [../../GLOSSARY.md](../../GLOSSARY.md)

---

## Reflection Questions

1. Why should a parsing pipeline keep stop words that a spam filter removes?
2. When does lemmatization outperform stemming for n-gram language models?
3. How does BPE handle a word never seen during tokenizer training?
4. What preprocessing from [Course 1 Chapter 04](https://github.com/Collaborative-ai/ai-academy-applied-ml-engineering/blob/main/chapters/chapter-04-text-classification/section-02-text-preprocessing.md) would you reuse unchanged for a CYK parser?

---

**Previous:** [Section 23.1 - Language Models](./section-01-language-models.md) · **Next:** [Section 23.3 - Grammar Formalisms](./section-03-grammar-formalisms.md)

---

## References

1. Russell, S., & Norvig, P. (2020). *Artificial Intelligence: A Modern Approach* (4th ed.). [aima.cs.berkeley.edu](https://aima.cs.berkeley.edu/)
2. Russell, S., & Norvig, P. - [aima-python](https://github.com/aimacode/aima-python)
