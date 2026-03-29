# NEXT SENTENCE

The `NEXT SENTENCE` phrase transfers control to the first statement following the next period (sentence terminator). It has been part of COBOL since COBOL-60 and predates scope terminators.

- **Standard:** COBOL-60, COBOL-68, COBOL-85, COBOL 2002, COBOL 2014, COBOL 2023
- **Division:** Procedure Division
- **Category:** Control Flow

!!! warning "Prefer CONTINUE"
    `NEXT SENTENCE` is considered obsolete in practice. Use `CONTINUE` instead,
    which respects scope terminators (`END-IF`, `END-EVALUATE`, etc.) and
    behaves predictably in structured code. `NEXT SENTENCE` remains in the
    standard for backward compatibility but should not be used in new programs.

---

## Syntax

```cobol
NEXT SENTENCE
```

---

## Rules

1. `NEXT SENTENCE` transfers control to the implicit scope terminator — the **next period** in the source code. All statements between `NEXT SENTENCE` and the next period are skipped.

2. `NEXT SENTENCE` may appear wherever an imperative statement is allowed, but it is most commonly used in `IF` and `EVALUATE` statements.

3. `NEXT SENTENCE` is **not the same as `CONTINUE`**. The critical difference is what defines the target:
    - `NEXT SENTENCE` jumps to the statement **after the next period**.
    - `CONTINUE` passes control to the **next statement in sequence** (respecting scope terminators).

4. `NEXT SENTENCE` must not appear within an explicitly terminated statement (one using `END-IF`, `END-EVALUATE`, etc.) if the intent is to exit only that statement. The jump will bypass the scope terminator and continue to the next period.

---

## The NEXT SENTENCE vs CONTINUE Problem

The distinction between `NEXT SENTENCE` and `CONTINUE` is one of the most common sources of bugs in COBOL programs, particularly when migrating from period-delimited to scope-terminated code.

### With Period-Delimited Code (Safe)

In traditional period-delimited COBOL, `NEXT SENTENCE` works as expected because the period is both the scope terminator and the sentence terminator:

```cobol
       IF AMOUNT < 0
           NEXT SENTENCE
       ELSE
           PERFORM PROCESS-AMOUNT.
       PERFORM NEXT-STEP.
```

`NEXT SENTENCE` skips to the statement after the period — `PERFORM NEXT-STEP`. This is the intended behavior and works correctly.

### With Scope Terminators (Dangerous)

When scope terminators are used, `NEXT SENTENCE` **ignores** `END-IF` and jumps to the next period, which may be far away:

```cobol
       IF RECORD-TYPE = "A"
           IF AMOUNT < 0
               NEXT SENTENCE
           ELSE
               PERFORM PROCESS-CREDIT
           END-IF
           PERFORM LOG-TRANSACTION
       END-IF
       PERFORM NEXT-RECORD.
```

The programmer likely intended `NEXT SENTENCE` to skip to `PERFORM LOG-TRANSACTION` (after the inner `END-IF`). Instead, it jumps to the statement after the next **period** — `PERFORM NEXT-RECORD` — skipping both `PERFORM LOG-TRANSACTION` and the outer `END-IF` entirely.

### The Correct Fix

Replace `NEXT SENTENCE` with `CONTINUE`:

```cobol
       IF RECORD-TYPE = "A"
           IF AMOUNT < 0
               CONTINUE
           ELSE
               PERFORM PROCESS-CREDIT
           END-IF
           PERFORM LOG-TRANSACTION
       END-IF
       PERFORM NEXT-RECORD.
```

Now `CONTINUE` passes control to the next statement within the inner `IF` scope — `PERFORM LOG-TRANSACTION` — which is the correct behavior.

---

## Common Usage Patterns

### Null THEN Clause

The most common use of `NEXT SENTENCE` was to express "do nothing in this case":

```cobol
      *> Old style (period-delimited)
       IF ERROR-CODE = 0
           NEXT SENTENCE
       ELSE
           PERFORM ERROR-HANDLER.

      *> Modern equivalent
       IF ERROR-CODE NOT = 0
           PERFORM ERROR-HANDLER
       END-IF
```

### Skipping to the End of a Sentence

```cobol
      *> Old style — skip remaining statements in sentence
       IF STATUS-CODE = "OK"
           NEXT SENTENCE.
       DISPLAY "Error: " STATUS-CODE.
       PERFORM ERROR-ROUTINE.
       PERFORM CLEANUP.

      *> Modern equivalent
       IF STATUS-CODE NOT = "OK"
           DISPLAY "Error: " STATUS-CODE
           PERFORM ERROR-ROUTINE
           PERFORM CLEANUP
       END-IF
```

---

## Migration Guide

When modernizing COBOL code, replace `NEXT SENTENCE` according to the context:

| Pattern | Replace With |
|---------|-------------|
| `IF cond NEXT SENTENCE ELSE action.` | `IF NOT cond action END-IF` |
| `IF cond action ELSE NEXT SENTENCE.` | `IF cond action END-IF` |
| `IF cond NEXT SENTENCE.` (skip rest of sentence) | Restructure with `IF NOT cond ... END-IF` |
| `NEXT SENTENCE` in nested IF with `END-IF` | `CONTINUE` |

!!! tip "Compiler Warning"
    Many modern compilers issue a warning when `NEXT SENTENCE` is used within
    an explicitly terminated statement. Enable compiler warnings to catch
    these cases during modernization.

---

## Historical Context

`NEXT SENTENCE` was the original mechanism for handling null branches in COBOL-60, which had no scope terminators — every `IF` statement was terminated by a period. When COBOL-85 introduced explicit scope terminators (`END-IF`, `END-EVALUATE`, etc.), `CONTINUE` was added as the structured alternative. However, `NEXT SENTENCE` was not removed from the standard to preserve backward compatibility with the vast body of existing COBOL programs.

---

## See Also

- [CONTINUE](continue.md) -- no-operation statement (preferred alternative)
- [IF](if.md) -- conditional execution
- [EVALUATE](evaluate.md) -- multi-way branching
- [Scope Terminators](../../language/scope-terminators.md) -- explicit scope termination
