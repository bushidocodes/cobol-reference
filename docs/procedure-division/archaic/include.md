# INCLUDE (Removed)

The `INCLUDE` statement incorporated procedures from a COBOL library tape into the source program, with optional parameter substitution via a REPLACING clause. It was the predecessor to the modern `COPY` statement.

- **Introduced:** COBOL-60 (Elective)
- **Replaced by:** `COPY` statement
- **Status:** Removed

!!! warning "Removed"
    `INCLUDE` was replaced by the `COPY` statement, which provides a more
    general text inclusion facility that works across all divisions, not just
    the Procedure Division. Use [COPY](../../copy-replace/copy.md) in all
    new programs.

---

## Syntax

### Inline Form

```cobol
INCLUDE procedure-name
    [ REPLACING routine-word-1 BY { data-name-1 | word-1 }
              [ routine-word-2 BY { data-name-2 | word-2 } ] ... ]
```

### Declarative Form

```cobol
INCLUDE procedure-name FROM LIBRARY
```

The declarative form was used to incorporate USE and DEFINE declarations from the library.

---

## Rules

1. `procedure-name` identifies a procedure stored in the COBOL library.
2. The REPLACING clause allowed parameter substitution — each occurrence of `routine-word-n` in the library procedure was replaced by the corresponding `data-name-n` or `word-n`.
3. INCLUDE was a compiler directing verb — it appeared inline in the Procedure Division where the included code should be inserted.
4. The library procedure could itself contain INCLUDE statements, but only to reference other library entries.
5. INCLUDE could only bring in Procedure Division content. Data Division and Environment Division content were included via the COPY clause (which existed separately in those divisions).

---

## Examples

```cobol
INCLUDE SPECIAL-ROUTINE

INCLUDE STOCK-STATUS-REPORT REPLACING CHAIRS BY TABLES

INCLUDE PAYROLL-CALCULATION
    REPLACING HOURLY-RATE BY WS-RATE
              HOURS-WORKED BY WS-HOURS
```

In the second example, every occurrence of `CHAIRS` in the library's STOCK-STATUS-REPORT procedure would be replaced by `TABLES` when included.

---

## Why It Was Replaced

1. **Limited scope** — INCLUDE only worked for Procedure Division content. A separate COPY mechanism existed in the Data and Environment Divisions. Unifying them into a single `COPY` statement simplified the language.
2. **COPY...REPLACING** subsumed INCLUDE's parameter substitution capability while adding text-level (not just word-level) replacement.
3. **COPY** works at the source text level, making it division-independent and more flexible.

---

## Modern Alternative

```cobol
COPY STOCK-STATUS-REPORT REPLACING ==CHAIRS== BY ==TABLES==
```

The `COPY...REPLACING` mechanism uses pseudo-text delimiters (`==`) to identify replacement targets, allowing multi-word replacements that INCLUDE could not handle.

---

## See Also

- [COPY](../../copy-replace/copy.md) -- modern source text inclusion
- [REPLACE](../../copy-replace/replace.md) -- source text replacement
- [DEFINE](define.md) -- macro-like verb definitions (also removed)
