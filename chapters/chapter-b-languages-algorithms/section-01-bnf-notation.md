# Section B.1: BNF Notation

> **Source:** Russell & Norvig, *Artificial Intelligence: A Modern Approach* (4th ed.), Appendix B.1  
> **Vocabulary:** [GLOSSARY.md](../../GLOSSARY.md)  
> **Math conventions:** [MATH_CONVENTIONS.md](../../MATH_CONVENTIONS.md)

---

## Formal Grammars for Languages

**Backus-Naur Form (BNF)** specifies the syntax of formal languages - programming languages, logical formulas, natural language grammars. AIMA uses BNF throughout [Chapters 07-08](../chapter-07-logical-agents/README.md) and [23](../chapter-23-natural-language-processing/README.md). Appendix B establishes notation for reading and writing grammars.

> **Readable form:** BNF is a recipe for valid sentences - it lists what pieces can combine and in what order.

---

## Basic Components

| Symbol | Name | Example |
|--------|------|---------|
| `terminal` | Literal token | `+`, `if`, `dog` |
| `<nonterminal>` | Syntactic category | `<expr>`, `<noun>` |
| `::=` | Production definition | "is defined as" |
| `\|` | Alternative | OR choice |
| `*` / `+` | Repetition (EBNF) | Zero or more / one or more |

**Example grammar:**

```
<digit> ::= 0 | 1 | 2 | 3 | 4 | 5 | 6 | 7 | 8 | 9
<number> ::= <digit> | <digit> <number>
```

`<number>` derives `123` via repeated application of productions.

---

## Derivation Example

Grammar for simple arithmetic:

```
<expr>   ::= <expr> + <term> | <term>
<term>   ::= <term> * <factor> | <factor>
<factor> ::= ( <expr> ) | <number>
<number> ::= <digit> | <digit> <number>
<digit>  ::= 0 | 1 | 2 | 3 | 4 | 5 | 6 | 7 | 8 | 9
```

Derive `2 + 3 * 4$:

```
<expr> ⇒ <expr> + <term>
       ⇒ <term> + <term>
       ⇒ <factor> + <term>
       ⇒ <number> + <term>
       ⇒ ... ⇒ 2 + 3 * 4
```

Precedence encoded in grammar structure - `*` lower in hierarchy than `+` in this example (actually here `*` binds tighter via `<term>`).

> **Readable form:** Each line is a rewrite rule - replace nonterminals until only terminals remain.

---

## EBNF Extensions

**Extended BNF** adds syntactic sugar:

```
<list> ::= <item> { , <item> }    // comma-separated list
<id>   ::= <letter> { <letter> | <digit> }
```

`{ }` - zero or more repetitions  
`[ ]` - optional  
`( )` - grouping

---

## Parse Trees

Derivation corresponds to **parse tree** - root is start symbol, leaves are terminals.

```
       <expr>
      /  |  \
  <expr> + <term>
    |       / | \
  <term>  <term> * <factor>
    |       |      |
  <factor> <factor> 4
    |       |
    2       3
```

Used in compilers, NLP parsers, semantic interpretation.

---

## Ambiguity

Grammar is **ambiguous** if multiple parse trees exist for same string.

Classic: `if E then S else S` - dangling else.

**Resolution:** precedence rules, associativity declarations, or grammar refactoring.

---

## AI Applications

| Application | Grammar role |
|-------------|--------------|
| Propositional logic | Well-formed formulas |
| First-order logic | Terms, sentences |
| NLP | Syntactic parsing (CFG) |
| Program synthesis | Valid program generation |
| LLM constraints | Grammar-guided decoding |

---

## Context-Free Grammars

BNF defines **context-free grammars (CFG)** - production left side is single nonterminal.

More powerful formalisms exist (context-sensitive, unrestricted) but CFG suffices for most programming and natural language syntax (with limitations).

**Chomsky hierarchy** - CFG is Type 2.

---

## Writing Grammars

| Tip | Reason |
|-----|--------|
| Factor common patterns | Reduce redundancy |
| Encode precedence in levels | Avoid ambiguity |
| Use EBNF for lists | Readability |
| Test with derivations | Verify coverage |

---

---

## Study Notes

Russell and Norvig present this material as foundational for multiple AI subfields. When working through exercises:

- Re-derive key formulas before looking up answers
- Implement at least one algorithm or calculation in Python
- Connect concepts to the relevant course chapter
- Test edge cases - algorithms often fail on boundaries first

> **Readable form:** Mastery means you can explain it, compute it, and code it - not just recognize the definition on a flashcard.

### Cross-Chapter Connections

| Related chapter | Connection |
|----------------|------------|
| Earlier foundations | Provides prerequisites for formal derivations |
| Applied chapters | Techniques appear in agents, learning, and perception |
| Ethics and safety | Deployment context shapes how methods are used |

### Practice Checklist

| Step | Action |
|------|--------|
| 1 | Read the AIMA section alongside this section |
| 2 | Complete all five exercises |
| 3 | Implement one code example from scratch |
| 4 | Explain one key formula in plain English |
| 5 | Identify one real-world application |

---

## Additional Examples

### Worked Example

Consider applying the concepts from this section to a concrete AI problem. Identify inputs, outputs, constraints, and evaluation criteria before implementing. Start with the smallest instance that exercises the core idea.

### Implementation Reminder

```python
# Template: verify understanding with executable code
def verify_understanding():
    """Replace with section-specific test."""
    assert True  # placeholder - implement real check
    return "ok"
```

### Summary Table

| Concept | Definition | AI application |
|---------|------------|----------------|
| Core idea | See section sections above | Chapter-specific |
| Key formula | See math blocks above | Computation |
| Pitfall | Common student error | Debugging |
| Extension | Advanced topic | Further reading |

> **Readable form:** If you can fill in this table from memory, you understand the section well enough to move on.

## Key Takeaways

1. BNF defines formal languages via terminal/nonterminal production rules.
2. Derivations rewrite start symbol into terminal strings using productions.
3. EBNF extensions (`{ }`, `[ ]`) compactly express repetition and optionality.
4. Parse trees represent syntactic structure; ambiguous grammars yield multiple trees.
5. CFGs underpin logic syntax, NLP parsing, and grammar-guided generation.

## Exercises

1. Write BNF for balanced parentheses: `()`, `(())`, `()()`.
2. Derive string `a + b * c` using the arithmetic grammar above.
3. Show ambiguity in grammar `<stmt> ::= if <expr> then <stmt> | if <expr> then <stmt> else <stmt>`.
4. Convert EBNF `<args> ::= <expr> { , <expr> }` to pure BNF.
5. Write CFG for propositional logic formulas with `and`, `or`, `not`, and atoms.

---

**Next:** [Section B.2 - Pseudocode Conventions](./section-02-pseudocode-conventions.md)