# RAISE and RESUME

The `RAISE` statement explicitly raises an exception condition, and the `RESUME` statement transfers control from an exception handler back to a resumption point.

- **Standard:** COBOL 2002, COBOL 2014, COBOL 2023
- **Division:** Procedure Division
- **Category:** Exception Handling

!!! note "Compiler Support"
    RAISE and RESUME are part of COBOL 2002+ but are not supported by all
    compilers. IBM Enterprise COBOL 6.x does not support them. Micro Focus
    and GnuCOBOL provide varying levels of support.

---

## RAISE

Explicitly raises an exception condition.

### Syntax

```cobol
RAISE [ EXCEPTION ] exception-name

RAISE [ EXCEPTION ] exception-object
```

### Rules

1. `exception-name` is a predefined or user-defined exception condition (e.g., `EC-SIZE-OVERFLOW`, `EC-RANGE-SEARCH-NO-MATCH`).
2. When RAISE executes, the runtime searches for an active exception handler (USE AFTER EXCEPTION CONDITION).
3. If no handler is found, the program terminates abnormally.
4. User-defined exception conditions are declared in the REPOSITORY paragraph.

### Example

```cobol
IF WS-INPUT-VALUE < 0
    RAISE EXCEPTION EC-RANGE-INSPECT-SIZE
END-IF
```

---

## RESUME

Transfers control from within an exception handler to a resumption point.

### Syntax

```cobol
RESUME [ AT ] NEXT STATEMENT

RESUME [ AT ] procedure-name
```

### Rules

1. RESUME can only appear within an exception handler (USE AFTER EXCEPTION CONDITION).
2. `RESUME AT NEXT STATEMENT` returns to the statement following the one that raised the exception.
3. `RESUME AT procedure-name` transfers control to the specified paragraph or section.
4. RESUME without any phrase is equivalent to `RESUME AT NEXT STATEMENT`.

### Example

```cobol
DECLARATIVES.
OVERFLOW-HANDLER SECTION.
    USE AFTER EXCEPTION CONDITION EC-SIZE-OVERFLOW.
OVERFLOW-PARA.
    DISPLAY "Size overflow - using maximum value"
    MOVE 999999 TO WS-RESULT
    RESUME AT NEXT STATEMENT.
END DECLARATIVES.
```

---

## Predefined Exception Conditions

Exception conditions follow the naming pattern `EC-category-detail`:

| Exception | Meaning |
|-----------|---------|
| `EC-ALL` | Any exception |
| `EC-ARGUMENT` | Invalid function argument |
| `EC-BOUND` | Subscript or reference out of bounds |
| `EC-DATA` | Data exception (incompatible data) |
| `EC-FLOW` | Flow of control exception |
| `EC-I-O` | I/O exception |
| `EC-IMP` | Implementor-defined exception |
| `EC-OO` | Object-oriented exception |
| `EC-ORDER` | Ordering exception |
| `EC-OVERFLOW` | Overflow condition |
| `EC-PROGRAM` | Program exception |
| `EC-RAISING` | Exception during RAISE |
| `EC-RANGE` | Range exception |
| `EC-SIZE` | Size error |
| `EC-SORT-MERGE` | Sort/merge exception |
| `EC-STORAGE` | Storage allocation failure |

---

## Relationship to Traditional Error Handling

| Mechanism | Scope | Standard |
|-----------|-------|----------|
| `ON SIZE ERROR` | Single arithmetic statement | COBOL-68 |
| `AT END` / `INVALID KEY` | Single I/O statement | COBOL-68 |
| `USE AFTER ERROR` | File-level error handler | COBOL-68 |
| `RAISE` / `RESUME` | Program-wide exception handling | COBOL 2002 |

RAISE/RESUME provides a more structured alternative to the traditional conditional phrases, similar to try/catch in other languages.

---

## See Also

- [DECLARATIVES and USE](declaratives.md) -- USE AFTER EXCEPTION CONDITION
- [Conditional Phrases](../../language/conditional-phrases.md) -- ON SIZE ERROR, AT END, INVALID KEY
- [Exception Handling Functions](../../intrinsic-functions/exception-handling.md) -- EXCEPTION-STATUS, etc.
