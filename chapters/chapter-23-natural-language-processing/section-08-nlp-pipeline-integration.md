# Section 23.8: NLP Pipeline Integration

> **Source:** Russell & Norvig, *Artificial Intelligence: A Modern Approach* (4th ed.), Chapter 23.8
> **Previous:** [Section 23.7 - Pragmatics and Discourse](./section-07-pragmatics-and-discourse.md)
> **Vocabulary:** [../../GLOSSARY.md](../../GLOSSARY.md)
> **Math conventions:** [../../MATH_CONVENTIONS.md](../../MATH_CONVENTIONS.md)

---
## Assembling the Classical Stack

Chapters 23.1-23.7 built individual components. This section wires them into **end-to-end classical NLP pipelines** - the architecture that powered machine translation, speech recognition, and information retrieval before the transformer era.

[Course 1 Chapter 13](https://github.com/Collaborative-ai/ai-academy-applied-ml-engineering/blob/main/chapters/chapter-13-natural-language-processing/README.md) implements neural pipelines in Keras. Understanding classical integration clarifies what those neural models replaced - and what they still depend on (tokenization, evaluation metrics).

> **Readable form:** A classical NLP pipeline is an assembly line - each station transforms text until you get structure, meaning, or a probability score.

---

## Canonical Pipeline Architecture

```
                    ┌─────────────────────────────────────┐
  Raw Text ────────►│ 1. Text Processing                  │
                    │    tokenize, normalize, segment     │
                    └──────────────┬──────────────────────┘
                                   ▼
                    ┌─────────────────────────────────────┐
                    │ 2. Morphology / POS Tagging         │
                    └──────────────┬──────────────────────┘
                                   ▼
          ┌────────────────────────┼────────────────────────┐
          ▼                        ▼                        ▼
   ┌─────────────┐         ┌─────────────┐         ┌─────────────┐
   │ 3a. N-gram  │         │ 3b. Parser  │         │ 3c. WSD /   │
   │     LM      │         │  (CYK/PCFG) │         │   NER       │
   └──────┬──────┘         └──────┬──────┘         └──────┬──────┘
          │                       ▼                        │
          │              ┌─────────────┐                   │
          │              │ 4. Semantics│◄──────────────────┘
          │              └──────┬──────┘
          │                       ▼
          │              ┌─────────────┐
          └─────────────►│ 5. Pragmatics│
                         │  / Discourse │
                         └──────┬──────┘
                                ▼
                         Application Layer
```

Not every application runs all stages - [spam filters](https://github.com/Collaborative-ai/ai-academy-applied-ml-engineering/blob/main/chapters/chapter-04-text-classification/section-06-spam-filtering.md) skip parsing; speech recognizers lean heavily on [language models](./section-01-language-models.md).

---

## Pipeline by Application

| Application | Key stages | Classical technique |
|-------------|--------------|---------------------|
| **Spell correction** | LM + edit distance | Noisy channel model |
| **Speech recognition** | Acoustic model + LM | N-gram rescoring |
| **Machine translation** | Parse + transfer + generate | Rule-based / statistical |
| **Information extraction** | NER + coreference + RE | Hand-built patterns |
| **Question answering** | IR + parse + semantics | Pipeline QA |
| **Sentiment** | Tokenize + classify | Often bag-of-words |

[Course 1 Chapter 04](https://github.com/Collaborative-ai/ai-academy-applied-ml-engineering/blob/main/chapters/chapter-04-text-classification/README.md) covers the sentiment row with scikit-learn - the simplest viable pipeline.

---

## The Noisy Channel Model

A unifying probabilistic framework:

$$
\hat{w} = \arg\max_w P(w \mid o) = \arg\max_w P(o \mid w) \cdot P(w)
$$
> **Readable form:** The expression assigns probability to the event or value using the stated model assumptions.

| Term | Role | Example (spell check) |
|------|------|---------------------|
| $o$ | Observation | Typed text "teh" |
| $w$ | Hypothesis | Intended word "the" |
| $P(w)$ | Prior | [Language model](./section-01-language-models.md) |
| $P(o \mid w)$ | Channel model | Keyboard edit probability |

**Speech recognition:** $o$ = acoustic signal features, $w$ = word sequence.

**Machine translation (IBM models):** $o$ = source sentence, $w$ = target sentence.

[Chapter 12 - Quantifying Uncertainty](../chapter-12-quantifying-uncertainty/README.md) provides the Bayesian framing.

> **Readable form:** Noisy channel says: pick the most likely original message given what you observed corrupted by noise.

---

## Error Propagation

Pipelines are **sequential** - early mistakes cascade:

```
Wrong tokenization → wrong POS → wrong parse → wrong semantics
```

| Mitigation | Description |
|------------|-------------|
| **Joint models** | Single model predicts multiple layers |
| **k-best lists** | Pass top-k hypotheses downstream |
| **Confidence thresholds** | Abstain when uncertain |
| **End-to-end neural** | Learned representations skip explicit stages |

[Chapter 24](../chapter-24-deep-learning-nlp/README.md) end-to-end models reduce explicit error propagation but introduce new failure modes (hallucination).

---

## Feature-Based ML Pipeline (Bridge to Course 1)

Before deep learning, many tasks used **engineered features** from classical NLP:

```python
def extract_features(sentence, parser, tagger):
    tokens = tokenize(sentence)
    pos = tagger.tag(tokens)
    tree = parser.parse(tokens)
    return {
        "bow": bag_of_words(tokens),
        "root_verb": tree.root_label(),
        "np_count": count_constituents(tree, "NP"),
        "avg_word_len": sum(len(t) for t in tokens) / len(tokens),
    }
```

Feed features to logistic regression or SVM - exactly the [Course 1](https://github.com/Collaborative-ai/ai-academy-applied-ml-engineering/blob/main/chapters/chapter-04-text-classification/README.md) pattern extended with linguistic structure.

---

## Evaluation End-to-End

| Level | Metric | Chapter reference |
|-------|--------|------------------|
| LM | [Perplexity](./section-01-language-models.md) | 23.1 |
| POS | Accuracy | 23.2 |
| Parse | F1 (PARSEVAL) | 23.4-23.5 |
| WSD | Accuracy | 23.6 |
| Coreference | CEAF F1 | 23.7 |
| Application | Task-specific (BLEU, F1, accuracy) | - |

**Regression testing:** Maintain a golden set of sentences with expected outputs at each stage.

---

## Tooling Landscape

| Tool | Role |
|------|------|
| **NLTK** | Teaching, parsers, corpora |
| **spaCy** | Production pipelines, pretrained models |
| **Stanford CoreNLP** | Research-grade annotators |
| **OpenNLP** | Java Apache stack |

```python
import spacy
nlp = spacy.load("en_core_web_sm")
doc = nlp("The cat sat on the mat.")
for token in doc:
    print(token.text, token.pos_, token.dep_)
```

Modern spaCy pipelines are neural - but expose the same stages conceptually.

---

## Classical MT Pipeline (Historical)

**Statistical machine translation (SMT)** circa 2000-2015:

1. Sentence alignment (parallel corpus)
2. Word alignment (IBM Models 1-5)
3. Phrase table extraction
4. [N-gram language model](./section-01-language-models.md) on target
5. Decoder search for highest $P(\text{target} \mid \text{source})$

Replaced by neural MT ([Chapter 24, Section 3](../chapter-24-deep-learning-nlp/section-03-seq2seq-and-attention.md)) - but LM rescoring and phrase tables persisted in hybrid systems for years.

---

## Information Extraction Pipeline

```
Documents → sentence split → NER → coreference → relation extraction → KB
```

Output populates [knowledge bases](../chapter-10-knowledge-representation/README.md). Classical IE used pattern rules + statistical NER; modern systems finetune transformers.

---

## Transition to Neural NLP

| Classical strength | Classical weakness | Neural response |
|-------------------|-------------------|-----------------|
| Interpretable stages | Error propagation | End-to-end learning |
| Data-efficient (some tasks) | Feature engineering | Automatic features |
| Small models | Limited generalization | Scale + pretraining |

[Course 3 Chapter 10](https://github.com/Collaborative-ai/ai-academy-deep-learning-foundations/blob/main/chapters/chapter-10-sequence-modeling/README.md) introduces RNNs for sequences; [Chapter 24](../chapter-24-deep-learning-nlp/README.md) completes the neural story with transformers.

**Recommended learning path:**

1. Build classical trigram LM + CYK parser (this chapter's lab)
2. Replicate sentiment with [Course 1 Chapter 04](https://github.com/Collaborative-ai/ai-academy-applied-ml-engineering/blob/main/chapters/chapter-04-text-classification/section-05-sentiment-analysis.md) baseline
3. Beat baseline with [Course 1 Chapter 13](https://github.com/Collaborative-ai/ai-academy-applied-ml-engineering/blob/main/chapters/chapter-13-natural-language-processing/README.md) BERT finetuning
4. Understand *why* perplexity improved at each step

---

## Production Considerations

| Concern | Classical approach | Modern approach |
|---------|---------------------|-----------------|
| Latency | Fast n-grams, small parsers | GPU inference, distillation |
| Memory | Count tables | Model weights (GBs) |
| Updates | Retrain counts | Finetune / RAG |
| Monitoring | Per-stage accuracy | End-task metrics + drift |

Document preprocessing contracts between training and serving - critical when mixing classical and neural components.

---

## Chapter 23 Lab Recap

Integrate:

1. **Trigram LM** with Kneser-Ney smoothing → report perplexity
2. **CYK parser** on toy CFG → visualize parse trees
3. **Compare** LM probabilities for grammatical vs ungrammatical sentences
4. **Optional:** spaCy pipeline on same sentences - map outputs to section concepts

---

## Key Terms (Glossary)

- **pipeline** - see [../../GLOSSARY.md](../../GLOSSARY.md)
- **POS tagging** - see [../../GLOSSARY.md](../../GLOSSARY.md)
- **information extraction** - see [../../GLOSSARY.md](../../GLOSSARY.md)

---

## Reflection Questions

1. Draw the pipeline stages needed for a voice-controlled calendar assistant.
2. How does the noisy channel model unify spell checking and speech recognition?
3. Why did statistical MT need both a translation model and a language model?
4. Which classical components would you still run before a transformer in production?

---

**Previous:** [Section 23.7 - Pragmatics and Discourse](./section-07-pragmatics-and-discourse.md)

---

## References

1. Russell, S., & Norvig, P. (2020). *Artificial Intelligence: A Modern Approach* (4th ed.). [aima.cs.berkeley.edu](https://aima.cs.berkeley.edu/)
2. Russell, S., & Norvig, P. - [aima-python](https://github.com/aimacode/aima-python)



