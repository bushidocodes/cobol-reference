# CONTINUE

The `CONTINUE` statement is an explicit no-operation that causes no action and passes control to the next executable statement.

- **Standard:** COBOL-85, COBOL 2002, COBOL 2014, COBOL 2023
- **Division:** Procedure Division
- **Category:** Control Flow

---

## Syntax

### Format 1 — No-Operation

```cobol
CONTINUE
```

### Format 2 — Timed Wait

```cobol
CONTINUE AFTER {identifier-1 | arithmetic-expression-1} SECONDS
```

---

## Rules

### No-Operation (Format 1)

`CONTINUE` performs no action. Control passes to the next executable statement in normal sequence.

`CONTINUE` may be used wherever a statement is permitted. Its primary purpose is to serve as an explicit placeholder in conditional constructs where one branch requires no action.

### Timed Wait (Format 2)

!!! note "COBOL 2023"
    The `CONTINUE AFTER ... SECONDS` form was introduced in COBOL 2023.

`CONTINUE AFTER` suspends execution for the number of seconds specified by `identifier-1` or `arithmetic-expression-1`.

- If the value is zero, no pause occurs.
- If the value is negative, the result is implementation-defined; many compilers treat it as zero.
- The value may be non-integer; fractional seconds are supported if the implementation allows.

```cobol
CONTINUE AFTER 5 SECONDS
CONTINUE AFTER WS-DELAY-TIME SECONDS
CONTINUE AFTER WS-MINUTES * 60 SECONDS
```

---

## Behavior

### Use as a Placeholder in IF

When one branch of an [IF](if.md) statement requires no action, `CONTINUE` makes the intent explicit.

```cobol
IF WS-STATUS = "A"
    CONTINUE
ELSE
    PERFORM ERROR-ROUTINE
END-IF
```

Without `CONTINUE`, the `IF` branch would be empty, which some compilers reject and which obscures intent.

### Use as a Placeholder in EVALUATE

In an [EVALUATE](evaluate.md) statement, `CONTINUE` serves as a placeholder for `WHEN` branches that require no processing.

```cobol
EVALUATE WS-CODE
    WHEN "A"
        PERFORM PROCESS-A
    WHEN "B"
        CONTINUE
    WHEN "C"
        PERFORM PROCESS-C
    WHEN OTHER
        PERFORM PROCESS-DEFAULT
END-EVALUATE
```

### Difference from NEXT SENTENCE

`CONTINUE` and `NEXT SENTENCE` both cause no action, but they differ in where control passes:

| Statement | Control passes to |
|-----------|------------------|
| `CONTINUE` | The next executable statement in normal sequence |
| `NEXT SENTENCE` | The first statement following the next period (`.`) |

This distinction is significant when `CONTINUE` or `NEXT SENTENCE` appears inside a nested construct that is not terminated by a period.

```cobol
IF WS-A = 1
    IF WS-B = 2
        NEXT SENTENCE          *> Skips to after the period
    ELSE
        PERFORM PROCESS-B
    END-IF
    PERFORM ALWAYS-RUN         *> Skipped by NEXT SENTENCE
END-IF.

IF WS-A = 1
    IF WS-B = 2
        CONTINUE               *> Falls through normally
    ELSE
        PERFORM PROCESS-B
    END-IF
    PERFORM ALWAYS-RUN         *> Executed by CONTINUE path
END-IF.
```

!!! warning "Avoid NEXT SENTENCE"
    `NEXT SENTENCE` predates structured COBOL and interacts poorly with `END-IF`, `END-EVALUATE`, and other scope terminators. `CONTINUE` is the preferred replacement in all modern programs.

---

## Examples

### Placeholder in an IF-ELSE

```cobol
IF WS-RECORD-TYPE = "H"
    CONTINUE
ELSE
    ADD 1 TO WS-DETAIL-COUNT
    PERFORM PROCESS-DETAIL
END-IF
```

### Placeholder in EVALUATE

```cobol
EVALUATE TRUE
    WHEN WS-AGE < 18
        MOVE "Minor" TO WS-CATEGORY
    WHEN WS-AGE < 65
        CONTINUE
    WHEN WS-AGE >= 65
        MOVE "Senior" TO WS-CATEGORY
END-EVALUATE
```

### Timed Delay Between Retries

```cobol
MOVE 0 TO WS-RETRY-COUNT
PERFORM UNTIL WS-SUCCESS = "Y"
    OR WS-RETRY-COUNT >= 3
    PERFORM ATTEMPT-CONNECTION
    IF WS-SUCCESS NOT = "Y"
        ADD 1 TO WS-RETRY-COUNT
        CONTINUE AFTER 2 SECONDS
    END-IF
END-PERFORM
```

### Timed Wait with Computed Duration

```cobol
CONTINUE AFTER WS-RETRY-COUNT * 2 SECONDS
```

---

## See Also

- [IF](if.md) — conditional execution
- [EVALUATE](evaluate.md) — multi-branch conditional statement
- [EXIT](exit.md) — exits a paragraph, section, or performed loop
- [PERFORM](perform.md) — executes paragraphs or inline statements
- [Procedure Division Overview](../index.md)
