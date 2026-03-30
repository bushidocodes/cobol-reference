# VALIDATE

The `VALIDATE` statement verifies that the content of a data item conforms to the requirements specified in its data description (PICTURE, class, VALUE ranges).

- **Standard:** COBOL 2002 (defined), COBOL 2014 (refined)
- **Division:** Procedure Division
- **Category:** Data Movement / Validation

!!! warning "Limited Compiler Support"
    VALIDATE is defined in the COBOL standard but is not widely implemented.
    IBM Enterprise COBOL does not support it. Check your compiler
    documentation before using this statement.

---

## Syntax

```cobol
VALIDATE identifier-1 [ identifier-2 ] ...
```

---

## Rules

1. Each identifier is validated against the rules defined by its data description entry.
2. Validation checks include:
    - **Class validation** — numeric items contain only digits (and sign); alphabetic items contain only letters and spaces.
    - **Range validation** — the value falls within the ranges specified by level-88 condition names (if the data item has associated WHEN-SET-TO-FALSE, PRESENT-WHEN, or VARYING clauses).
    - **Relation validation** — relationships between data items are satisfied.
3. If validation fails, an **exception condition** (`EC-VALIDATE`) is raised.
4. VALIDATE does not change the content of the data item — it only checks it.
5. Multiple data items can be validated in a single statement.

---

## Example

```cobol
       01  WS-CUSTOMER-RECORD.
           05  WS-CUST-ID    PIC 9(8).
           05  WS-CUST-NAME  PIC X(30).
           05  WS-CUST-TYPE  PIC X.
               88  VALID-TYPE VALUE "R" "P" "G".
           05  WS-BALANCE    PIC S9(7)V99.

       VALIDATE WS-CUSTOMER-RECORD
       *> Checks:
       *> - WS-CUST-ID is numeric
       *> - WS-CUST-NAME is valid alphanumeric
       *> - WS-CUST-TYPE is one of "R", "P", or "G"
       *> - WS-BALANCE is valid signed numeric
```

---

## Alternative: Manual Validation

Since VALIDATE has limited compiler support, most COBOL programs perform validation manually:

```cobol
       IF WS-CUST-ID NOT NUMERIC
           DISPLAY "Invalid customer ID"
           MOVE "N" TO WS-VALID
       END-IF
       IF NOT VALID-TYPE
           DISPLAY "Invalid customer type: " WS-CUST-TYPE
           MOVE "N" TO WS-VALID
       END-IF
```

---

## See Also

- [Conditions](../../language/conditions.md) -- class conditions (NUMERIC, ALPHABETIC)
- [Condition Names](../../data-division/condition-names.md) -- level-88 value validation
- [INSPECT](../string-handling/inspect.md) -- character-level scanning
- [RAISE and RESUME](../control-flow/raise-resume.md) -- exception handling
