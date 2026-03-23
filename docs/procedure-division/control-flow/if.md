# IF

The `IF` statement evaluates a condition and executes one of two sets of statements based on the result.

- **Standard:** COBOL-60, COBOL-61, COBOL-61 Extended, COBOL-65, COBOL-68, COBOL-74, COBOL-85, COBOL 2002, COBOL 2014, COBOL 2023
- **Division:** Procedure Division
- **Category:** Control Flow

---

## Syntax

```cobol
IF condition-1 [THEN]
    {statement-1 ... | NEXT SENTENCE}
[ELSE
    {statement-2 ... | NEXT SENTENCE}]
[END-IF]
```

---

## Rules

### General

- If `condition-1` evaluates to true, `statement-1` is executed. If `condition-1` evaluates to false and the `ELSE` phrase is present, `statement-2` is executed.
- If `condition-1` evaluates to false and no `ELSE` phrase is present, control passes to the next executable statement after the `IF` statement (i.e., after `END-IF` or the next period).
- The `THEN` keyword is optional and has no effect on execution.

### END-IF vs Period Scope

The `END-IF` explicit scope terminator marks the end of the `IF` statement. Alternatively, a period (`.`) implicitly terminates the `IF` statement and all other open statements within the sentence.

!!! note "COBOL-85"
    The `END-IF` scope terminator was introduced in COBOL-85. Programs conforming to COBOL-68 or COBOL-74 must use the period to terminate `IF` statements.

!!! warning "Pitfall: Misplaced Period"
    A stray period inside a nested `IF` terminates **all** open `IF` statements in the sentence, not just the innermost one. This is a common source of logic errors. Always use `END-IF` in modern programs.

    ```cobol
    *> BUG: The period after PERFORM PROCESS-A terminates both IFs.
    IF WS-TYPE = "X"
        IF WS-SUB-TYPE = "1"
            PERFORM PROCESS-A.       *> <-- period ends BOTH IFs
        ELSE                          *> <-- this ELSE is orphaned
            PERFORM PROCESS-B
        END-IF
    END-IF
    ```

### NEXT SENTENCE

`NEXT SENTENCE` transfers control to the first statement following the next period (`.`), bypassing any remaining statements in the current sentence.

!!! warning "NEXT SENTENCE vs CONTINUE"
    `NEXT SENTENCE` transfers control past the next period, which may skip statements outside the `IF` construct. `CONTINUE` is a no-operation that passes control to the next statement within the normal flow. In modern programs, `CONTINUE` is preferred over `NEXT SENTENCE`.

    ```cobol
    *> NEXT SENTENCE skips to after the period, past PERFORM ALWAYS-RUN
    IF WS-SKIP = "Y"
        NEXT SENTENCE
    ELSE
        PERFORM SOME-ROUTINE
    END-IF
    PERFORM ALWAYS-RUN.

    *> CONTINUE allows normal flow into PERFORM ALWAYS-RUN
    IF WS-SKIP = "Y"
        CONTINUE
    ELSE
        PERFORM SOME-ROUTINE
    END-IF
    PERFORM ALWAYS-RUN.
    ```

---

## Condition Types

The condition in an `IF` statement may be any of the following:

### Relation Condition

Compares two operands using a relational operator.

```cobol
IF WS-AMOUNT > 1000.00
    PERFORM HIGH-VALUE-PROCESS
END-IF

IF WS-NAME = "SMITH"
    PERFORM SMITH-PROCESS
END-IF
```

Relational operators:

| Symbol | Keyword Form |
|--------|-------------|
| `=`  | `IS EQUAL TO` |
| `>`  | `IS GREATER THAN` |
| `<`  | `IS LESS THAN` |
| `>=` | `IS GREATER THAN OR EQUAL TO` |
| `<=` | `IS LESS THAN OR EQUAL TO` |
| `<>` | `IS NOT EQUAL TO` |

### Class Condition

Tests whether a data item's content belongs to a particular class.

```cobol
IF WS-INPUT IS NUMERIC
    COMPUTE WS-RESULT = WS-INPUT * 2
END-IF

IF WS-FIELD IS ALPHABETIC
    PERFORM ALPHA-PROCESS
END-IF
```

Available class tests: `NUMERIC`, `ALPHABETIC`, `ALPHABETIC-LOWER`, `ALPHABETIC-UPPER`.

!!! note "COBOL-85"
    `ALPHABETIC-LOWER` and `ALPHABETIC-UPPER` were introduced in COBOL-85.

### Sign Condition

Tests whether a numeric value is positive, negative, or zero.

```cobol
IF WS-BALANCE IS NEGATIVE
    PERFORM OVERDRAWN-PROCESS
END-IF
```

Sign tests: `POSITIVE` (> 0), `NEGATIVE` (< 0), `ZERO` (= 0).

### Condition-Name (Level 88) Condition

Tests whether the value of a data item matches one of the values associated with a level-88 condition name.

```cobol
01 WS-STATUS    PIC X(1).
   88 STATUS-ACTIVE   VALUE "A".
   88 STATUS-INACTIVE VALUE "I".
   88 STATUS-VALID    VALUE "A" "I" "P".

IF STATUS-ACTIVE
    PERFORM ACTIVE-PROCESS
END-IF

IF STATUS-VALID
    PERFORM VALID-PROCESS
END-IF
```

### Complex Conditions

Conditions can be combined using `AND`, `OR`, and `NOT`.

```cobol
IF WS-AGE >= 18 AND WS-AGE <= 65
    PERFORM WORKING-AGE-PROCESS
END-IF

IF WS-CODE = "A" OR WS-CODE = "B"
    PERFORM AB-PROCESS
END-IF

IF NOT STATUS-ACTIVE
    PERFORM INACTIVE-PROCESS
END-IF
```

### Abbreviated Combined Relation Conditions

When multiple relation conditions share the same subject or relational operator, the common parts may be omitted.

```cobol
*> Full form
IF WS-CODE = "A" OR WS-CODE = "B" OR WS-CODE = "C"

*> Abbreviated form
IF WS-CODE = "A" OR "B" OR "C"

*> Full form
IF WS-X > 10 AND WS-X < 100

*> Abbreviated form
IF WS-X > 10 AND < 100
```

!!! warning "Pitfall: Abbreviated Condition Ambiguity"
    Abbreviated conditions can be misread. `IF A = B OR C` means `IF A = B OR A = C`, **not** `IF A = B OR C IS TRUE`. Parentheses should be used to clarify intent in complex expressions.

---

## Nested IF

`IF` statements may be nested within either the `THEN` branch or the `ELSE` branch of another `IF` statement. Each `END-IF` pairs with the nearest preceding unpaired `IF`.

```cobol
IF WS-TYPE = "A"
    IF WS-PRIORITY = 1
        PERFORM HIGH-PRIORITY-A
    ELSE
        PERFORM NORMAL-A
    END-IF
ELSE
    IF WS-TYPE = "B"
        PERFORM PROCESS-B
    ELSE
        PERFORM PROCESS-DEFAULT
    END-IF
END-IF
```

!!! note "Readability"
    Deeply nested `IF` statements quickly become difficult to read. When testing multiple values of a single item, the [EVALUATE](evaluate.md) statement is often clearer.

---

## Examples

### Simple IF-ELSE

```cobol
IF WS-GENDER = "M"
    MOVE "Male" TO WS-GENDER-TEXT
ELSE
    MOVE "Female" TO WS-GENDER-TEXT
END-IF
```

### IF with No ELSE

```cobol
IF WS-DISCOUNT-ELIGIBLE = "Y"
    COMPUTE WS-TOTAL = WS-TOTAL * 0.90
END-IF
```

### Compound Condition

```cobol
IF WS-AGE >= 18
    AND WS-CITIZENSHIP = "US"
    AND NOT WS-FELON
    MOVE "Y" TO WS-ELIGIBLE
ELSE
    MOVE "N" TO WS-ELIGIBLE
END-IF
```

### Nested IF with Condition Names

```cobol
01 WS-ACCOUNT-TYPE  PIC X(1).
   88 SAVINGS       VALUE "S".
   88 CHECKING      VALUE "C".
   88 INVESTMENT    VALUE "I".

01 WS-BALANCE       PIC S9(9)V99.

IF SAVINGS
    IF WS-BALANCE < 500.00
        PERFORM APPLY-SAVINGS-FEE
    END-IF
ELSE IF CHECKING
    IF WS-BALANCE < 100.00
        PERFORM APPLY-CHECKING-FEE
    END-IF
ELSE IF INVESTMENT
    PERFORM COMPUTE-INVESTMENT-RETURNS
END-IF
```

### Class Test with Arithmetic

```cobol
ACCEPT WS-INPUT FROM CONSOLE

IF WS-INPUT IS NUMERIC
    COMPUTE WS-RESULT = FUNCTION NUMVAL(WS-INPUT) + 100
    DISPLAY "Result: " WS-RESULT
ELSE
    DISPLAY "Error: non-numeric input"
END-IF
```

---

## See Also

- [EVALUATE](evaluate.md) — multi-branch conditional statement
- [PERFORM](perform.md) — executes paragraphs or inline statements
- CONTINUE — no-operation placeholder
- [Procedure Division Overview](../index.md)
