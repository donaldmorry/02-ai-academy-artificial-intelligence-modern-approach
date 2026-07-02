# Section 24.8: NLP Ethics

> **Source:** Russell & Norvig, *Artificial Intelligence: A Modern Approach* (4th ed.), Chapter 24.6
> **Previous:** [Section 24.7 - LLM Capabilities](./section-07-llm-capabilities.md)
> **Vocabulary:** [../../GLOSSARY.md](../../GLOSSARY.md)
> **Math conventions:** [../../MATH_CONVENTIONS.md](../../MATH_CONVENTIONS.md)

---
## Power Requires Responsibility

[LLMs](./section-07-llm-capabilities.md) influence hiring, healthcare, education, law, and public discourse. Russell and Norvig connect NLP ethics to the broader [risks and benefits](../chapter-01-introduction/section-05-risks-and-benefits-of-ai.md) framework - technical capability outpaces governance.

This section covers **bias**, **hallucination**, **privacy**, **misuse**, and **mitigation** - essential for anyone deploying models trained in [Chapter 24](./README.md) or [Course 1 Chapter 13](https://github.com/Collaborative-ai/ai-academy-applied-ml-engineering/blob/main/chapters/chapter-13-natural-language-processing/README.md).

> **Readable form:** A fluent model that confidently lies, stereotypes, or leaks private data is worse than no model at all in high-stakes settings.

---

## Bias in Language Models

LLMs learn from **internet text** - reflecting historical and cultural biases.

### Manifestations

| Bias type | Example |
|-----------|---------|
| **Gender** | "Doctor" → he; "nurse" → she |
| **Racial** | Sentiment skew on AAE dialect |
| **Cultural** | Western-centric history narratives |
| **Socioeconomic** | Stigma in mental health, poverty contexts |

[Word2Vec](./section-01-word-embeddings.md) analogies exposed gender bias ("man : computer programmer :: woman : homemaker"). Larger LMs mask but do not eliminate these patterns.

### Measurement

- **CrowS-Pairs** - stereotype vs anti-stereotype preference
- **BBQ** - bias benchmarks across identities
- **Discrimination testing** - paired prompts differing only in demographic attribute

---

## Sources of Bias

```
Training data demographics
        ↓
Tokenization & preprocessing choices
        ↓
Objective (likelihood ≠ fairness)
        ↓
Finetuning data (annotator bias)
        ↓
Deployment context (who uses it, for what)
```

[Course 1 Chapter 04](https://github.com/Collaborative-ai/ai-academy-applied-ml-engineering/blob/main/chapters/chapter-04-text-classification/section-05-sentiment-analysis.md) sentiment models on product reviews may fail on dialects underrepresented in training - same mechanism at scale.

---

## Mitigating Bias

| Strategy | Description |
|----------|-------------|
| **Data curation** | Balance demographics, filter toxic text |
| **Counterfactual augmentation** | Swap gendered names in training pairs |
| **Adversarial debiasing** | Penalize protected-attribute predictability |
| **Instruction tuning** | Explicit fairness guidelines in system prompts |
| **Human review** | Audit outputs on sensitive queries |
| **Diverse teams** | Annotation, evaluation, policy |

No technical fix alone suffices - **process and accountability** matter.

---

## Hallucination

**Hallucination:** model generates fluent, confident, **false** content.

| Type | Example |
|------|---------|
| **Factual** | Invented citations, fake statistics |
| **Faithfulness** | Summary adds claims not in source |
| **Intrinsic** | Contradicts prompt context |

### Causes

- Objective rewards **plausible** text, not **true** text
- No grounded knowledge base ([RAG](./section-07-llm-capabilities.md) partially addresses)
- Compression of training data into finite parameters

### Mitigations

| Approach | Mechanism |
|----------|-----------|
| **RAG** | Ground in retrieved documents |
| **Citation requirements** | Force quotes from provided context |
| **Confidence calibration** | Abstain when uncertain |
| **Fact-checking pipelines** | External verification |
| **Structured outputs** | JSON with schema validation |

[Chapter 23 semantics](../chapter-23-natural-language-processing/section-06-semantics.md) distinguished truth conditions; LLMs optimize linguistic form without truth guarantee.

> **Readable form:** Hallucination is the model writing a convincing Wikipedia article about a historical figure who never existed.

---

## Privacy and Data Leakage

Pretraining on web text risks **memorizing** personal information:

- Names, addresses, phone numbers
- Medical records (if in training leak)
- Proprietary code (GitHub scraping controversies)

**Membership inference** attacks detect if a sample was in training data.

### Protections

- PII scrubbing in training corpora
- Differential privacy (noisy training - trade accuracy)
- Output filters for SSN, credit card patterns
- Enterprise deployments with data residency controls

[Course 1 operationalization](https://github.com/Collaborative-ai/ai-academy-applied-ml-engineering/blob/main/chapters/chapter-07-operationalizing-models/README.md) principles apply - never log user prompts with PII without consent.

---

## Misuse and Safety

| Misuse | Concern |
|--------|---------|
| **Disinformation** | Scale of fake news, deepfake text |
| **Spam/phishing** | Personalized attacks at scale |
| **Malware assistance** | Code generation for exploits |
| **Harassment** | Automated abuse |
| **Academic dishonesty** | Essay generation |

**Red teaming** - adversarial testing before release - identifies jailbreaks (prompts bypassing safety filters).

**RLHF** ([Section 24.6](./section-06-gpt-and-autoregressive-lms.md)) reduces but does not eliminate harmful outputs.

---

## Environmental and Labor Ethics

| Issue | Detail |
|-------|--------|
| **Carbon footprint** | Large model training consumes significant energy |
| **Annotator labor** | RLHF relies on human labelers - fair compensation, trauma exposure |
| **Concentration of power** | Few organizations control frontier models |

Efficient models (DistilBERT, small LLMs), green datacenters, and open-weight models address subsets of these concerns.

---

## Regulation and Governance

Emerging frameworks:

- **EU AI Act** - risk tiers for AI systems
- **NIST AI RMF** - risk management framework
- **Sector rules** - HIPAA (health), FERPA (education)

Deployers must document:

- Intended use and limitations
- Evaluation on representative populations
- Human oversight for high-stakes decisions
- Appeal mechanisms when AI aids consequential decisions

[Chapter 27 - Philosophy, Ethics, Safety](../chapter-27-philosophy-ethics-safety/README.md) expands the AIMA treatment.

---

## Responsible Deployment Checklist

1. **Risk assessment** - What harm if wrong?
2. **Baseline comparison** - Is LLM better than [simpler methods](https://github.com/Collaborative-ai/ai-academy-applied-ml-engineering/blob/main/chapters/chapter-04-text-classification/README.md)?
3. **Disclosure** - Users know they interact with AI
4. **Human-in-the-loop** - Review for medical, legal, financial outputs
5. **Monitoring** - Track failure modes, bias metrics, hallucination rate
6. **Incident response** - Plan for model failures in production
7. **Accessibility** - Not everyone benefits equally from English-centric LMs

---

## Fairness Formalizations

Multiple incompatible definitions:

| Criterion | Statement |
|-----------|-----------|
| **Demographic parity** | $P(\hat{Y} \mid A=a) = P(\hat{Y} \mid A=b)$ |
| **Equalized odds** | Equal TPR and FPR across groups |
| **Calibration** | $P(Y=1 \mid \text{score}, A) = \text{score}$ |

Impossibility theorems show these cannot all hold simultaneously - **ethical choice**, not purely technical.

---

## Transparency and Explainability

LLMs are **black boxes** - attention weights are not faithful explanations.

| Alternative | Value |
|-------------|-------|
| **Provenance** | Show retrieved documents (RAG) |
| **Uncertainty** | Express low confidence |
| **Classical hybrid** | Parser + LM for auditable components |
| **Model cards** | Document training data, limitations |

[PCFG parse trees](../chapter-23-natural-language-processing/section-05-statistical-parsing.md) were interpretable; modern accuracy trades interpretability - compensate with process controls.

---

## Connection to Course 1 and 3

[Course 1 Chapter 04](https://github.com/Collaborative-ai/ai-academy-applied-ml-engineering/blob/main/chapters/chapter-04-text-classification/section-06-spam-filtering.md) - false positives block legitimate email; LLM false outputs scale similarly.

[Course 3 Chapter 10](https://github.com/Collaborative-ai/ai-academy-deep-learning-foundations/blob/main/chapters/chapter-10-sequence-modeling/README.md) - sequence models amplify training data patterns; ethics starts at data collection.

The Chapter 24 lab compares DistilBERT to bag-of-words - include **error analysis** on demographic slices, not just accuracy.

---

## Key Terms (Glossary)

- **bias** - see [../../GLOSSARY.md](../../GLOSSARY.md)
- **hallucination** - see [../../GLOSSARY.md](../../GLOSSARY.md)
- **responsible NLP** - see [../../GLOSSARY.md](../../GLOSSARY.md)

---

## Reflection Questions

1. Why does maximizing likelihood not produce truthful outputs?
2. Give an example where demographic parity and equalized odds conflict.
3. When is RAG insufficient to prevent hallucination?
4. Should frontier LLM weights be open-source? What are arguments both ways?

---

**Previous:** [Section 24.7 - LLM Capabilities](./section-07-llm-capabilities.md)

---

## References

1. Russell, S., & Norvig, P. (2020). *Artificial Intelligence: A Modern Approach* (4th ed.). [aima.cs.berkeley.edu](https://aima.cs.berkeley.edu/)
2. Russell, S., & Norvig, P. - [aima-python](https://github.com/aimacode/aima-python)