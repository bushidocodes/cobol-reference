# NOTE (Removed)

The `NOTE` statement allowed the programmer to write explanatory text in the Procedure Division that would appear on the program listing but not be compiled.

- **Introduced:** COBOL-60
- **Replaced by:** Comment indicator (`*` in column 7) and inline comments (`*>` in COBOL 2002)
- **Status:** Removed

---

## Syntax

```cobol
NOTE any text until the next period.
```

---

## Rules

1. Everything from the word `NOTE` to the next period was treated as a comment.
2. If `NOTE` appeared as the first word in a paragraph, the entire paragraph (all sentences until the next paragraph name) was treated as a comment.
3. NOTE was a compiler directing verb — it generated no object code.

---

## Examples

### Inline Comment

```cobol
NOTE THIS ROUTINE HANDLES END-OF-FILE PROCESSING.
```

### Comment Paragraph

```cobol
EXPLANATION-PARA.
    NOTE THIS PARAGRAPH IS ENTIRELY EXPLANATORY.
    IT DESCRIBES THE FOLLOWING PROCESSING LOGIC.
    NO CODE IS GENERATED FROM THIS TEXT.

ACTUAL-PROCESSING.
    MOVE A TO B.
```

When NOTE was the first word in a paragraph, everything up to the next paragraph name was a comment.

---

## Why It Was Replaced

NOTE had a subtle and dangerous interaction with the period:

```cobol
NOTE COMPUTE THE TAX AMOUNT.
MULTIPLY RATE BY AMOUNT GIVING TAX.
```

Because NOTE consumed everything to the next period, the `MULTIPLY` statement was **also treated as a comment** — it was part of the NOTE sentence (no period appeared between NOTE and MULTIPLY). This was a common source of bugs.

The comment indicator (`*` in column 7 of fixed-format source) provided a safer mechanism because each line was independently marked as a comment. Inline comments (`*>`) introduced in COBOL 2002 provided the same capability without the period-scoping danger.

---

## Modern Alternatives

### Comment Lines (COBOL-68+)

```cobol
      * This routine handles end-of-file processing
      * Multiple lines, each independently marked
```

### Inline Comments (COBOL 2002+)

```cobol
       MULTIPLY RATE BY AMOUNT GIVING TAX  *> Calculate tax
```

---

## See Also

- [Source Format](../../language/source-format.md) -- comment indicators and source layout
