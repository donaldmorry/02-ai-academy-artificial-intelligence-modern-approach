# Section 24.7: LLM Capabilities

> **Source:** Russell & Norvig, *Artificial Intelligence: A Modern Approach* (4th ed.), Chapter 24.6
> **Previous:** [Section 24.6 - GPT and Autoregressive LMs](./section-06-gpt-and-autoregressive-lms.md)
> **Next:** [Section 24.8 - NLP Ethics](./section-08-nlp-ethics.md)
> **Vocabulary:** [../../GLOSSARY.md](../../GLOSSARY.md)
> **Math conventions:** [../../MATH_CONVENTIONS.md](../../MATH_CONVENTIONS.md)

---
## From Models to Applications

[GPT](./section-06-gpt-and-autoregressive-lms.md) and [BERT](./section-05-bert-and-pretraining.md) are **foundations**. This section surveys how pretrained models become **products** - through prompting, finetuning, retrieval, and orchestration.

[Section 1.4 - State of the Art](../chapter-01-introduction/section-04-state-of-the-art.md) listed NLP capabilities visible to general users; here we explain **how** those capabilities emerge from transformer LMs.

> **Readable form:** An LLM is a engine; prompting, RAG, and tools are the steering wheel, GPS, and pedals that make it drivable.

---

## Prompt Engineering

A **prompt** is the input text conditioning model behavior. Small wording changes produce large output shifts.

### Zero-Shot

No examples - instruction only:

```
Classify the sentiment: "The movie was surprisingly good."
Answer:
```

### Few-Shot

Include labeled examples in context ([GPT-3](./section-06-gpt-and-autoregressive-lms.md) specialty):

```
Review: "Loved it!" → Positive
Review: "Terrible waste." → Negative
Review: "The movie was surprisingly good." →
```

### System Prompts

APIs separate **system** (behavior rules) from **user** (query):

```
System: You are a concise technical tutor. Use examples.
User: Explain backpropagation.
```

[Course 1 Chapter 04](https://github.com/Collaborative-ai/ai-academy-applied-ml-engineering/blob/main/chapters/chapter-04-text-classification/section-05-sentiment-analysis.md) trained classifiers on thousands of labels; few-shot prompting can match rough performance with dozens of examples in context - trading training compute for inference context length.

---

## Chain-of-Thought (CoT)

Wei et al. (2022): prompting models to **reason step by step** improves math and logic performance.

```
Q: Roger has 5 tennis balls. He buys 2 cans of 3 balls each. How many now?
A: Let's think step by step.
   Roger starts with 5.
   2 cans × 3 = 6 new balls.
   5 + 6 = 11. The answer is 11.
```

**Zero-shot CoT:** append "Let's think step by step" without examples.

CoT increases tokens generated - higher latency and cost - but reduces arithmetic errors.

---

## Instruction Tuning

**Supervised finetuning (SFT)** on instruction-response pairs teaches models to follow user intent - foundation of ChatGPT-style assistants.

| Stage | Data | Result |
|-------|------|--------|
| Pretraining | Raw web text | Base LM |
| SFT | (instruction, response) pairs | Helpful format |
| RLHF | Human preferences | Aligned behavior |

Open models (Llama, Mistral, Phi) release instruction-tuned variants (`-Instruct`, `-Chat`).

---

## Retrieval-Augmented Generation (RAG)

LLMs lack access to private or recent data. **RAG** retrieves relevant documents, injects them into the prompt:

```
Context: [retrieved chunks from company wiki]
Question: What is our refund policy for EU customers?
Answer:
```

| Component | Technology |
|-----------|------------|
| Retriever | Dense embeddings, BM25, hybrid |
| Index | Vector DB (FAISS, Pinecone, etc.) |
| Generator | LLM |

RAG reduces hallucination on factual queries - connects to [pragmatics](../chapter-23-natural-language-processing/section-07-pragmatics-and-discourse.md) (grounding in context).

---

## Tool Use and Agents

Modern systems let LLMs call **tools**:

- Web search
- Code execution (Python sandbox)
- Database queries
- API calls (calendar, email)

**ReAct** pattern: Reason → Act → Observe loop - LLM generates thought, tool call, reads result, continues.

This revives the [agent architecture](../chapter-02-intelligent-agents/README.md) - LLM as planner/controller.

---

## Task Landscape

| Task | Typical approach |
|------|-----------------|
| Classification | BERT finetune or few-shot GPT |
| Summarization | GPT with length prompt |
| Translation | Dedicated MT model or GPT |
| Code | Code-pretrained models (Codex, StarCoder) |
| QA | RAG + GPT |
| Dialogue | Instruction-tuned chat models |

[Course 1 Chapter 13](https://github.com/Collaborative-ai/ai-academy-applied-ml-engineering/blob/main/chapters/chapter-13-natural-language-processing/README.md) lab progression (LSTM → BERT) maps to production decision trees.

---

## Evaluation Benchmarks

| Benchmark | Measures |
|-----------|----------|
| MMLU | Multitask knowledge (57 subjects) |
| HumanEval | Code generation |
| GSM8K | Grade-school math |
| TruthfulQA | Factual accuracy |
| HELM | Holistic evaluation |

**Leaderboard chasing** risks overfitting - models train on benchmark contamination. Always evaluate on **your** domain data.

---

## Latency and Cost Trade-offs

| Model tier | Params | Use case |
|------------|--------|----------|
| Small (1-7B) | Edge, high QPS | Classification, routing |
| Medium (7-70B) | Balanced | Enterprise chat |
| Large (70B+) | Best quality | Research, hard reasoning |

**Distillation** - train small model to mimic large - and **quantization** (INT8, INT4) reduce deployment cost.

Compare to [bag-of-words latency](https://github.com/Collaborative-ai/ai-academy-applied-ml-engineering/blob/main/chapters/chapter-04-text-classification/section-08-pipelines-and-production-text-ml.md) - classical methods win on speed for simple tasks.

---

## Multimodal LLMs

GPT-4V, Gemini, Claude 3 accept **images + text** - vision encoder projects images into token space shared with language [transformer](./section-04-the-transformer.md).

Applications: document understanding, chart analysis, visual QA - bridges to [Chapter 25 - Computer Vision](../chapter-25-computer-vision/README.md).

---

## Limitations (Capabilities Perspective)

| Claim | Reality |
|-------|---------|
| "Understands" language | Predicts plausible continuations |
| "Knows" facts | Memorized patterns; may confabulate |
| "Reasons" | Pattern-matches reasoning templates |
| "Plans" | Fragile without tool feedback |

[Section 24.8](./section-08-nlp-ethics.md) covers societal implications; technically, **capability ≠ reliability**.

---

## Building with LLMs: Practical Checklist

1. **Define task** - generation vs classification vs extraction
2. **Choose model** - size, license, hosting (API vs self-hosted)
3. **Baseline** - [Course 1 Chapter 04](https://github.com/Collaborative-ai/ai-academy-applied-ml-engineering/blob/main/chapters/chapter-04-text-classification/README.md) simple model first
4. **Prototype** - prompt engineering before finetuning
5. **Evaluate** - domain-specific test set with human review
6. **Guardrails** - content filters, citation requirements, abstention
7. **Monitor** - drift, abuse, failure modes in production

---

## Connection to Sequence Modeling

[Course 3 Chapter 10](https://github.com/Collaborative-ai/ai-academy-deep-learning-foundations/blob/main/chapters/chapter-10-sequence-modeling/README.md) taught RNNs for sequences; transformers extended capabilities but **sequence thinking** remains essential - context windows, streaming output, temporal dialogue state.

[Classical pipelines](../chapter-23-natural-language-processing/section-08-nlp-pipeline-integration.md) decomposed tasks explicitly; LLMs **unify** tasks behind one interface - with trade-offs in control and interpretability.

---

## Key Terms (Glossary)

- **prompting** - see [../../GLOSSARY.md](../../GLOSSARY.md)
- **few-shot** - see [../../GLOSSARY.md](../../GLOSSARY.md)
- **chain-of-thought** - see [../../GLOSSARY.md](../../GLOSSARY.md)

---

## Reflection Questions

1. When does few-shot prompting outperform BERT finetuning?
2. How does RAG address knowledge cutoff and hallucination?
3. Why does chain-of-thought help arithmetic but increase cost?
4. What task from [Course 1 Chapter 13](https://github.com/Collaborative-ai/ai-academy-applied-ml-engineering/blob/main/chapters/chapter-13-natural-language-processing/README.md) would you still not trust to a zero-shot LLM?

---

**Previous:** [Section 24.6 - GPT and Autoregressive LMs](./section-06-gpt-and-autoregressive-lms.md) · **Next:** [Section 24.8 - NLP Ethics](./section-08-nlp-ethics.md)

---

## References

1. Russell, S., & Norvig, P. (2020). *Artificial Intelligence: A Modern Approach* (4th ed.). [aima.cs.berkeley.edu](https://aima.cs.berkeley.edu/)
2. Russell, S., & Norvig, P. - [aima-python](https://github.com/aimacode/aima-python)