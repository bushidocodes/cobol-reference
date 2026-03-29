# DEFINE (Removed)

The `DEFINE` statement allowed programmers to create new verbs defined in terms of existing COBOL verbs, functioning as a macro facility with parameter substitution.

- **Introduced:** COBOL-60 (Elective)
- **Removed:** Before COBOL-68
- **Status:** Removed

---

## Syntax

```cobol
DEFINE verb-name WITH FORMAT verb-name ...
    model-paragraph.
    first sentence of model ...
```

---

## Rules

1. `verb-name` is the name of the new verb being defined.
2. The `WITH FORMAT` clause specified the syntax pattern for the new verb.
3. A model paragraph provided the expansion template, written in terms of existing COBOL verbs.
4. When the defined verb was used in the program, the compiler expanded it by substituting arguments into the model paragraph.
5. DEFINE was a compiler directing declarative — it appeared at the beginning of the Procedure Division.

---

## Example

### Defining a "Find Maximum" Verb

```cobol
DEFINE YEAR-TO-DATE-FICA-COMPUTATION WITH
    FORMAT YEAR-TO-DATE-FICA-COMPUTATION.
    MODEL-PARAGRAPH.
    IF YTD-FICA IS LESS THAN 150
        THEN COMPUTE YTD-FICA = .03125 *
        GROSS-PAY + YTD-FICA.
    IF YTD-FICA EXCEEDS 150
        THEN MOVE 150 TO YTD-FICA.

DEFINE FIND-TRIPLET-MAXIMUM WITH
    FORMAT FIND-TRIPLET-MAXIMUM FROM A, B, AND C
    GIVING TRIPLET-MAX.
    FIND-MAX.
    IF A > B THEN IF A > C MOVE A TO
        TRIPLET-MAX ELSE MOVE C TO TRIPLET-MAX;
    OTHERWISE IF B > C MOVE B TO TRIPLET-MAX
        ELSE MOVE C TO TRIPLET-MAX.
```

### Using the Defined Verb

```cobol
YEAR-TO-DATE-FICA-COMPUTATION
FIND-TRIPLET-MAXIMUM FROM X, Y, AND Z GIVING RESULT
```

---

## Design Intent

DEFINE was intended to make COBOL "open-ended" — special-purpose verbs could be added to individual compilers while preserving compatibility across computers. It served dual purposes:

1. **Subroutine-like reuse** — encapsulate common operations as named verbs
2. **Language extension** — allow domain-specific verbs without modifying the compiler

Jean E. Sammet described it as enabling "both compatibility and efficiency" by allowing new verbs to be defined in terms of standard COBOL verbs.

---

## Why It Was Removed

DEFINE was part of Elective (not Required) COBOL-60 and was never widely implemented. The PERFORM statement provided a simpler mechanism for procedure reuse, and the COPY...REPLACING mechanism (evolved from INCLUDE) provided text substitution. DEFINE's macro semantics were considered too complex for the intended audience of business programmers.

---

## Modern Alternatives

- **PERFORM** — invoke a named paragraph or section
- **CALL** — invoke a separately compiled subprogram
- **COPY ... REPLACING** — text inclusion with substitution
- **User-defined functions** (COBOL 2002) — FUNCTION-ID with RETURNING

---

## See Also

- [PERFORM](../control-flow/perform.md) -- structured procedure invocation
- [COPY](../../copy-replace/copy.md) -- source text inclusion with REPLACING
- [User-Defined Functions](../../intrinsic-functions/user-defined.md) -- FUNCTION-ID definitions
