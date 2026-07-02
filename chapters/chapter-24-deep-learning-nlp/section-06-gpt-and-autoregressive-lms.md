# Section 24.6: GPT and Autoregressive LMs

> **Source:** Russell & Norvig, *Artificial Intelligence: A Modern Approach* (4th ed.), Chapter 24, Deep Learning for Natural Language Processing
> **Enhanced with:** source-grounded explanations, worked checks, implementation guidance, diagnostics, and mastery prompts
> **Vocabulary:** [GLOSSARY.md](../../GLOSSARY.md)
> **Math conventions:** [MATH_CONVENTIONS.md](../../MATH_CONVENTIONS.md)

---

## The Core Idea

GPT-style models are autoregressive transformer language models: they predict the next token from previous tokens, then feed generated tokens back into the context to continue. AIMA discusses GPT-2 as an example of a large transformer-like language model that can produce fluent local text and perform surprising tasks from prompts.

Autoregression is simple and powerful, but it gives the model a one-directional view during generation. Unlike BERT, a GPT model cannot condition a next-token prediction on future text that has not yet been generated.

> **Readable form:** GPT models factor a sequence probability into left-to-right next-token probabilities.

$$
P(w_1,\ldots,w_T)=\prod_{t=1}^{T}P(w_t\mid w_{<t})
$$
> **Readable form:** The probability of a generated text is the product of each next-token probability conditioned on earlier tokens.

---

## Autoregressive Transformer Mechanics

A decoder-only transformer uses causal self-attention. Each position may attend to earlier positions, but not later ones.

| Component | Role |
|-----------|------|
| Token embedding | Represents the current token identity |
| Positional embedding | Represents order in the context window |
| Causal self-attention | Lets each token use prior context only |
| Feedforward layers | Transform contextual token vectors |
| Softmax head | Predicts the next-token distribution |

The same mechanism supports both training and generation. During training, the target next tokens are known. During generation, sampled or selected tokens become part of the next context.

---

## Prompted Generation

AIMA's GPT-2 examples illustrate both the strength and weakness of autoregressive generation. The text can be fluent and locally coherent, but it may drift, repeat, contradict itself, or collapse into boilerplate.

| Sampling choice | Effect |
|-----------------|--------|
| Greedy decoding | Deterministic but can become repetitive |
| Temperature | Controls randomness in token sampling |
| Top-k or nucleus sampling | Restricts sampling to plausible tokens |
| Stop sequences | Prevent runaway continuation |
| Prompt design | Sets the task, style, and context available to the model |

Autoregressive fluency is not the same as truth, planning, or grounded reasoning.

---

## Transfer Learning and State of the Art

AIMA emphasizes the 2018-era shift: new NLP projects increasingly start with pretrained transformer models. GPT-2, RoBERTa, and T5 demonstrate that large pretrained models can be adapted or prompted for many tasks.

| Model family | Training or use pattern |
|--------------|------------------------|
| BERT/RoBERTa | Encoder-style contextual representations and fine-tuning |
| GPT-2 | Autoregressive prompt continuation, sometimes without task-specific fine-tuning |
| T5 | Text-to-text encoder-decoder transfer learning |
| ARISTO ensemble | Combines retrieval, reasoning, and transformer language models |

The source chapter also warns that benchmark gains do not eliminate limitations. Models may handle multiple-choice questions better than essays, lack diagram understanding, and rely on limited context windows.

---

## BERT versus GPT

| Dimension | BERT-style MLM | GPT-style autoregression |
|-----------|----------------|--------------------------|
| Direction | Bidirectional context for masked tokens | Left-to-right context for next token |
| Architecture | Transformer encoder | Transformer decoder or decoder-only stack |
| Pretraining target | Recover masked words | Predict next word/token |
| Best fit | Understanding and classification tasks | Generation and continuation tasks |
| Main risk | Pretraining/fine-tuning mismatch or data leakage | Fluent hallucination, repetition, and exposure bias |

Both are transfer-learning systems, but they package language knowledge for different inference patterns.

---

## Diagnostics

| Diagnostic | Healthy signal | Warning signal |
|------------|----------------|----------------|
| Perplexity or next-token loss | Improves on held-out text | Improves only on training data |
| Prompt robustness | Small wording changes preserve task behavior | Output changes unpredictably |
| Repetition check | Long generation remains diverse | Loops or repeated phrases appear |
| Factuality check | Claims can be verified externally | Fluent unsupported assertions |
| Context-limit check | Important evidence stays within window | Model ignores earlier required facts |
| Refusal or uncertainty behavior | Admits missing evidence where appropriate | Answers every prompt confidently |

---

## Minimal Causal Mask Check

```python
def allowed_attention(length):
    return [[j <= i for j in range(length)] for i in range(length)]

for row in allowed_attention(4):
    print(row)
```

If a GPT-style model can attend to future tokens during training, the evaluation is invalid. Causal masking is therefore part of the model definition, not an implementation detail.

---

## Reflection Questions

1. How does autoregressive factorization define the training objective?
2. Why does GPT need a causal attention mask?
3. How can locally fluent text still fail globally?
4. What is the difference between prompting and fine-tuning?
5. Which AIMA limitations still apply to modern large language models?
6. When should you prefer an encoder model, a decoder model, or an encoder-decoder model?

---

## References

- Russell, S., & Norvig, P. (2020). *Artificial Intelligence: A Modern Approach* (4th ed.), Ch. 24.
- AIMA Python code: [https://github.com/aimacode/aima-python](https://github.com/aimacode/aima-python)

---

**Previous:** [Section 24.5: BERT and Pretraining](./section-05-bert-and-pretraining.md)  
**Next:** [Section 24.7: LLM Capabilities](./section-07-llm-capabilities.md)

---

## Assessment Practice

Use the shared [Assessment Appendix](../../ASSESSMENT_APPENDIX.md) for concept audits, worked examples, implementation checks, experiment logs, oral-exam prompts, and deliverable checklists.
