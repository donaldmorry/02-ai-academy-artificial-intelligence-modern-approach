# Section 24.5: BERT and Pretraining

> **Source:** Russell & Norvig, *Artificial Intelligence: A Modern Approach* (4th ed.), Chapter 24, Deep Learning for Natural Language Processing
> **Enhanced with:** source-grounded explanations, worked checks, implementation guidance, diagnostics, and mastery prompts
> **Vocabulary:** [GLOSSARY.md](../../GLOSSARY.md)
> **Math conventions:** [MATH_CONVENTIONS.md](../../MATH_CONVENTIONS.md)

---

## The Core Idea

AIMA frames pretraining as transfer learning for NLP. Instead of hand-labeling a new corpus for every task, train a general language model on abundant unlabeled text, then adapt it to a smaller target task.

BERT is the canonical masked-language-model example: a transformer encoder is trained to recover hidden tokens using both left and right context. The result is a contextual representation that can be fine-tuned for tasks such as question answering, named entity recognition, sentiment analysis, text classification, and natural language inference.

> **Readable form:** BERT-style pretraining learns contextual token representations by predicting masked words from surrounding context, then reuses those representations for downstream tasks.

---

## Why Pretraining Matters

AIMA contrasts labeled NLP data with unlabeled text. POS tags, parse trees, and other annotations require expert effort, while raw text is abundant. Pretraining turns raw text into reusable linguistic representations.

| Data source | What it enables |
|-------------|-----------------|
| Unlabeled running text | Language-model pretraining |
| Question-answer pairs | Reading comprehension or QA adaptation |
| Parallel translations | Machine translation training |
| Reviews with ratings | Sentiment or preference modeling |
| Domain documents | Specialized vocabulary and idioms |

The practical workflow is pretrain broadly, then fine-tune narrowly.

---

## From Word Embeddings to Contextual Representations

Static word embeddings give one vector per word type. AIMA points out the problem with polysemy: the same surface word can have different meanings in different contexts.

| Representation | Limitation or strength |
|----------------|------------------------|
| Atomic word token | No similarity structure |
| Static word embedding | Captures general similarity but gives one vector per word |
| Contextual representation | Maps a word plus surrounding words to a context-specific vector |

A contextual model should represent a word like “rose” differently when it refers to a flower versus when it describes rising water.

---

## Masked Language Modeling

Masked language modeling hides selected input words and trains the model to predict them. Unlike left-to-right language modeling, MLM can use evidence from both sides of the missing word.

$$
\max_\theta \sum_{m\in M}\log P_\theta(w_m \mid x_{\setminus M})
$$
> **Readable form:** Train the model to predict masked tokens from the unmasked context.

This objective creates its own labels from raw text: the original hidden word is the answer. That is why it scales to large corpora without manual annotation.

---

## BERT as a Transformer Encoder

BERT uses a transformer encoder, so every unmasked token can attend bidirectionally across the input sequence. The pretrained encoder produces contextual vectors that can be reused with small task-specific heads.

| Downstream task | Fine-tuning signal |
|-----------------|-------------------|
| Classification | Label for a sentence or document |
| Named entity recognition | Token-level entity tags |
| Question answering | Start and end span positions |
| Natural language inference | Entailment, contradiction, or neutral label |
| Sentiment analysis | Positive, negative, or graded sentiment |

The section is not that BERT removes evaluation work. It shifts effort from feature engineering to data curation, fine-tuning, and error analysis.

---

## Diagnostics

| Diagnostic | Healthy signal | Warning signal |
|------------|----------------|----------------|
| Masked-token sanity check | Model uses both left and right context | Predictions ignore one side of context |
| Domain adaptation | Fine-tuning improves target validation | Model overfits small labeled data |
| Tokenization audit | Important domain terms are represented sensibly | Rare terms fragment into awkward subwords |
| Error slices | Improvements hold across linguistic phenomena | Gains only occur on easy examples |
| Calibration | Confidence tracks accuracy | Model is confidently wrong |

---

## Minimal Fine-Tuning Plan

1. Choose a downstream task and baseline.
2. Freeze or fine-tune the pretrained encoder deliberately.
3. Add the smallest task head that matches the output type.
4. Track validation metrics by error slice, not only aggregate score.
5. Compare against a non-contextual embedding baseline.

---

## Reflection Questions

1. Why is unlabeled text especially valuable for NLP pretraining?
2. What problem do contextual representations solve that static embeddings cannot?
3. Why can masked language modeling use right context?
4. What changes when BERT is fine-tuned for question answering rather than classification?
5. What evaluation leak can occur with small task data?
6. When would a smaller domain-specific model beat a larger generic one?

---

## References

- Russell, S., & Norvig, P. (2020). *Artificial Intelligence: A Modern Approach* (4th ed.), Ch. 24.
- AIMA Python code: [https://github.com/aimacode/aima-python](https://github.com/aimacode/aima-python)

---

**Previous:** [Section 24.4: The Transformer](./section-04-the-transformer.md)  
**Next:** [Section 24.6: GPT and Autoregressive LMs](./section-06-gpt-and-autoregressive-lms.md)

---

## Assessment Practice

Use the shared [Assessment Appendix](../../ASSESSMENT_APPENDIX.md) for concept audits, worked examples, implementation checks, experiment logs, oral-exam prompts, and deliverable checklists.
