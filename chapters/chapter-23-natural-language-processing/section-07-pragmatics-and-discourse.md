# Section 23.7: Pragmatics and Discourse

> **Source:** Russell & Norvig, *Artificial Intelligence: A Modern Approach* (4th ed.), Chapter 23.7
> **Previous:** [Section 23.6 - Semantics](./section-06-semantics.md)
> **Next:** [Section 23.8 - NLP Pipeline Integration](./section-08-nlp-pipeline-integration.md)
> **Vocabulary:** [../../GLOSSARY.md](../../GLOSSARY.md)
> **Math conventions:** [../../MATH_CONVENTIONS.md](../../MATH_CONVENTIONS.md)

---
## Meaning in Context

[Semantics](./section-06-semantics.md) assigns literal, sentence-internal meaning. **Pragmatics** studies how context shapes interpretation - who speaks, to whom, when, and why.

**Discourse** extends analysis across sentences - tracking topics, references, and rhetorical structure in documents and dialogues.

> **Readable form:** Semantics tells you what "I" refers to in a dictionary; pragmatics tells you it means the speaker, not the reader.

---

## Speech Acts

Austin and Searle: utterances **do things**, not just describe.

| Speech act | Example | Literal vs intended |
|------------|---------|---------------------|
| **Assertive** | "It's cold in here" | Statement (may imply request) |
| **Directive** | "Close the door" | Command |
| **Commissive** | "I'll finish by Friday" | Promise |
| **Expressive** | "Congratulations!" | Express feeling |
| **Declarative** | "I now pronounce you married" | Changes world if conditions met |

**Indirect speech acts:** "Can you pass the salt?" - yes/no question form, **request** intent.

Dialogue systems ([Chapter 24](../chapter-24-deep-learning-nlp/section-07-llm-capabilities.md)) must infer intent beyond literal semantics - modern LLMs learn this from data but still fail on subtle implicature.

---

## Grice's Maxims

Paul Grice proposed cooperative communication follows maxims:

| Maxim | Guideline |
|-------|-----------|
| **Quantity** | Say enough, not too much |
| **Quality** | Be truthful |
| **Relation** | Be relevant |
| **Manner** | Be clear, brief, orderly |

**Implicature:** What is suggested but not said. "Some students passed" implicates not all passed.

These principles help explain why humans resolve ambiguity without explicit rules - and why NLP struggles without shared context.

---

## Coreference Resolution

**Coreference** links expressions referring to the same entity:

*"John walked in. **He** sat down. **The professor** opened **his** laptop."*

| Mention | Antecedent |
|---------|------------|
| He | John |
| his | the professor |

**Pronoun resolution** is a classic NLP task - features include gender, number, syntactic position, and discourse salience.

Algorithms:

- **Rule-based** - Hobbs algorithm walks parse tree
- **Statistical** - classifiers on mention pairs
- **Neural** - end-to-end coreference resolvers (e.g., SpanBERT)

[Course 1 Chapter 04](https://github.com/Collaborative-ai/ai-academy-applied-ml-engineering/blob/main/chapters/chapter-04-text-classification/README.md) treats each sentence independently; discourse models maintain **entity registers** across sentences.

$$
P(\text{antecedent}(m_i) = m_j \mid \text{document})
$$
> **Readable form:** The expression assigns probability to the event or value using the stated model assumptions.

---

## Anaphora and Cataphora

| Type | Order | Example |
|------|-------|---------|
| **Anaphora** | Antecedent first | "John arrived. **He** waved." |
| **Cataphora** | Referent later | "**When he arrived**, John waved." |

**Bridging inference:** "I entered the room. The ceiling was high." - infer room has ceiling (world knowledge).

---

## Discourse Structure

Beyond sentence boundaries, documents have structure:

### Rhetorical Structure Theory (RST)

Relations between text spans: **Elaboration**, **Contrast**, **Cause**, **Evidence**.

```
[Claim: AI will transform healthcare]
    └── [Evidence: diagnostic accuracy studies]
```

### Topic Segmentation

Split documents into coherent sections - useful for summarization and retrieval.

### Dialogue Acts

In conversations, label each turn: **question**, **answer**, **acknowledgment**, **offer**.

Chatbots and voice assistants use dialogue act classification in pipeline architectures.

---

## Coherence and Cohesion

**Cohesion** - linguistic ties (pronouns, conjunctions, lexical repetition).

**Coherence** - logical flow of ideas (may exist without explicit ties).

*"The cat sat on the mat. It was fluffy."* - cohesive (it → cat).

*"The cat sat. Mathematics is abstract."* - no cohesion, incoherent.

Discourse models score coherence for essay grading, summarization quality, and LLM output filtering.

---

## Pragmatics in Information Extraction

Extracting structured facts from text requires pragmatic inference:

*"Apple announced record revenue"* - Apple = company, not fruit (disambiguation + world knowledge).

*"The CEO resigned after the scandal"* - who is CEO? When? Coreference + temporal reasoning ([Chapter 14](../chapter-14-probabilistic-reasoning-time/README.md)).

[Knowledge representation](../chapter-10-knowledge-representation/README.md) stores extracted facts; pragmatics governs what to extract.

---

## Question Answering and Discourse

Multi-hop QA requires discourse understanding:

*"Where was the author of *1984* born?"*

1. Resolve *1984* → George Orwell
2. Lookup birthplace → India (Motihari)

No single sentence contains the answer - **discourse + knowledge** bridge required.

[Chapter 24](../chapter-24-deep-learning-nlp/section-07-llm-capabilities.md) LLMs perform multi-hop reasoning imperfectly - often confabulating links.

---

## Evaluation

| Task | Benchmark | Metric |
|------|-----------|--------|
| Coreference | OntoNotes | MUC, B³, CEAF |
| Dialogue acts | Switchboard | Accuracy |
| Discourse parsing | RST-DT | F1 on relations |
| NLI (pragmatic) | IMPACT, DNC | Accuracy |

---

## Classical vs Neural Approaches

| Component | Classical | Neural (2020s) |
|-----------|-----------|----------------|
| Coreference | Mention-pair classifiers | Transformer end-to-end |
| Dialogue | POMDP planners + NLU | LLM agents |
| Discourse | RST parsers | Implicit in pretraining |

[Course 3 Chapter 10](https://github.com/Collaborative-ai/ai-academy-deep-learning-foundations/blob/main/chapters/chapter-10-sequence-modeling/README.md) sequence models process token streams; long-document transformers (Longformer, etc.) extend context for discourse.

[Course 1 Chapter 13](https://github.com/Collaborative-ai/ai-academy-applied-ml-engineering/blob/main/chapters/chapter-13-natural-language-processing/README.md) focuses on sentence-level tasks - this section explains what happens when you scale to documents.

---

## Connection to LLM Limitations

LLMs lack persistent **situated context** unless provided in the prompt:

- Who is speaking?
- What was agreed three meetings ago?
- What is the user's goal?

**Retrieval-augmented generation (RAG)** and **memory chapters** address discourse gaps - pragmatic engineering atop [language models](./section-01-language-models.md).

[Chapter 24, Section 8](../chapter-24-deep-learning-nlp/section-08-nlp-ethics.md) covers hallucination - often a failure of grounding and pragmatic faithfulness.

---

## Temporal and Event Semantics

Discourse often requires **ordering events** across sentences:

*"Before the merger was announced, insiders sold shares. The SEC opened an investigation."*

Understanding requires:

- Event timeline reconstruction
- Causal vs temporal "before"
- Entity persistence (the SEC, the merger)

**TimeML** and **event coreference** extend coreference to verbs and events - active research areas bridging [semantics](./section-06-semantics.md) and pragmatics.

---

## Key Terms (Glossary)

- **coreference** - see [../../GLOSSARY.md](../../GLOSSARY.md)
- **speech act** - see [../../GLOSSARY.md](../../GLOSSARY.md)
- **discourse** - see [../../GLOSSARY.md](../../GLOSSARY.md)

---

## Reflection Questions

1. Why is "Can you open the window?" not a yes/no question in most contexts?
2. What features help resolve "he" in a two-sentence paragraph about two men?
3. How does discourse structure differ from [parse tree](./section-03-grammar-formalisms.md) structure?
4. When would sentence-level [sentiment analysis](https://github.com/Collaborative-ai/ai-academy-applied-ml-engineering/blob/main/chapters/chapter-04-text-classification/section-05-sentiment-analysis.md) fail without discourse context?

---

**Previous:** [Section 23.6 - Semantics](./section-06-semantics.md) · **Next:** [Section 23.8 - NLP Pipeline Integration](./section-08-nlp-pipeline-integration.md)

---

## References

1. Russell, S., & Norvig, P. (2020). *Artificial Intelligence: A Modern Approach* (4th ed.). [aima.cs.berkeley.edu](https://aima.cs.berkeley.edu/)
2. Russell, S., & Norvig, P. - [aima-python](https://github.com/aimacode/aima-python)



