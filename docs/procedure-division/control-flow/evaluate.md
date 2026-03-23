# EVALUATE

The `EVALUATE` statement provides multi-branch conditional execution analogous to `switch` or `case` statements in other languages. It selects one of several sets of statements for execution based on the evaluation of one or more subjects against corresponding sets of values.

- **Standard:** COBOL-85, COBOL 2002, COBOL 2014, COBOL 2023
- **Division:** Procedure Division
- **Category:** Control Flow

---

## Syntax

```cobol
EVALUATE {identifier-1 | literal-1 | expression-1 | TRUE | FALSE}
    [ALSO {identifier-2 | literal-2 | expression-2 | TRUE | FALSE}] ...

    {WHEN
        {ANY
        | condition-1
        | TRUE
        | FALSE
        | [NOT] {identifier-3 | literal-3 | arithmetic-expression-1}
            [{THROUGH | THRU} {identifier-4 | literal-4 | arithmetic-expression-2}]
        }
        [ALSO
            {ANY
            | condition-2
            | TRUE
            | FALSE
            | [NOT] {identifier-5 | literal-5 | arithmetic-expression-3}
                [{THROUGH | THRU} {identifier-6 | literal-6 | arithmetic-expression-4}]
            }
        ] ...
        statement-1 ...} ...

    [WHEN OTHER
        statement-2 ...]

[END-EVALUATE]
```

---

## Rules

### Subjects and Objects

- The operands before the first `WHEN` phrase are the **selection subjects**.
- The operands within each `WHEN` phrase are the **selection objects**.
- If there are multiple subjects (separated by `ALSO`), each `WHEN` phrase must have a corresponding number of objects (also separated by `ALSO`).

### Evaluation Process

1. Each selection subject is evaluated once.
2. Each `WHEN` phrase is examined in order, from top to bottom.
3. For each `WHEN` phrase, each selection object is compared to its corresponding selection subject.
4. If all objects in a `WHEN` phrase satisfy their respective subjects, the statements following that `WHEN` phrase are executed, and control passes to the end of the `EVALUATE` statement.
5. If no `WHEN` phrase is satisfied and a `WHEN OTHER` phrase is present, the statements following `WHEN OTHER` are executed.
6. If no `WHEN` phrase is satisfied and no `WHEN OTHER` is present, control passes to the end of the `EVALUATE` statement.

### Comparison Rules

| Subject | Object | Match Condition |
|---------|--------|----------------|
| Identifier/literal/expression | Identifier/literal/expression | Subject equals object |
| Identifier/literal/expression | Object-1 THRU object-2 | Subject is within the range (inclusive) |
| TRUE | Condition | Condition is true |
| FALSE | Condition | Condition is false |
| Any subject | ANY | Always matches |

### WHEN OTHER

The `WHEN OTHER` phrase acts as a default branch. It is executed when none of the preceding `WHEN` phrases are satisfied. It must be the last `WHEN` phrase.

### Multiple WHEN Phrases (Fall-Through)

Multiple `WHEN` phrases may precede a single set of statements. All such `WHEN` phrases share the same execution path. This is analogous to multiple `case` labels in C without intervening `break` statements.

```cobol
EVALUATE WS-DAY
    WHEN "MON"
    WHEN "TUE"
    WHEN "WED"
    WHEN "THU"
    WHEN "FRI"
        MOVE "WEEKDAY" TO WS-DAY-TYPE
    WHEN "SAT"
    WHEN "SUN"
        MOVE "WEEKEND" TO WS-DAY-TYPE
    WHEN OTHER
        MOVE "UNKNOWN" TO WS-DAY-TYPE
END-EVALUATE
```

!!! note "COBOL-85"
    The `EVALUATE` statement was introduced in COBOL-85. Prior COBOL programs must use nested `IF` statements or `GO TO ... DEPENDING ON` for multi-way branching.

---

## Behavior

- Only the first matching `WHEN` branch is executed. There is no fall-through from one set of statements to the next (unlike C `switch` without `break`).
- Evaluation of subjects occurs exactly once, before any `WHEN` phrases are examined.
- If a `WHEN` phrase contains a condition (used with a `TRUE`/`FALSE` subject), the condition is evaluated during the matching process.

---

## Examples

### Simple Value-Based EVALUATE

```cobol
EVALUATE WS-REGION-CODE
    WHEN "NE"
        PERFORM PROCESS-NORTHEAST
    WHEN "SE"
        PERFORM PROCESS-SOUTHEAST
    WHEN "MW"
        PERFORM PROCESS-MIDWEST
    WHEN "SW"
        PERFORM PROCESS-SOUTHWEST
    WHEN "NW"
        PERFORM PROCESS-NORTHWEST
    WHEN OTHER
        DISPLAY "Unknown region: " WS-REGION-CODE
END-EVALUATE
```

### EVALUATE TRUE (Condition-Based)

When the subject is `TRUE`, each `WHEN` phrase contains a condition. The first `WHEN` whose condition is true is selected.

```cobol
EVALUATE TRUE
    WHEN WS-SCORE >= 90
        MOVE "A" TO WS-GRADE
    WHEN WS-SCORE >= 80
        MOVE "B" TO WS-GRADE
    WHEN WS-SCORE >= 70
        MOVE "C" TO WS-GRADE
    WHEN WS-SCORE >= 60
        MOVE "D" TO WS-GRADE
    WHEN OTHER
        MOVE "F" TO WS-GRADE
END-EVALUATE
```

This is equivalent to a chain of `IF`/`ELSE IF` statements but is more readable when testing multiple independent conditions.

### EVALUATE with THRU (Range)

```cobol
EVALUATE WS-TEMPERATURE
    WHEN 0 THRU 32
        MOVE "FREEZING" TO WS-TEMP-DESC
    WHEN 33 THRU 50
        MOVE "COLD" TO WS-TEMP-DESC
    WHEN 51 THRU 70
        MOVE "MILD" TO WS-TEMP-DESC
    WHEN 71 THRU 90
        MOVE "WARM" TO WS-TEMP-DESC
    WHEN 91 THRU 999
        MOVE "HOT" TO WS-TEMP-DESC
    WHEN OTHER
        MOVE "UNKNOWN" TO WS-TEMP-DESC
END-EVALUATE
```

### Multiple Subjects with ALSO

When multiple subjects are specified, each `WHEN` phrase must supply a corresponding value for each subject using `ALSO`.

```cobol
EVALUATE WS-GENDER ALSO WS-AGE-GROUP
    WHEN "M" ALSO "CHILD"
        PERFORM PROCESS-BOY
    WHEN "M" ALSO "ADULT"
        PERFORM PROCESS-MAN
    WHEN "F" ALSO "CHILD"
        PERFORM PROCESS-GIRL
    WHEN "F" ALSO "ADULT"
        PERFORM PROCESS-WOMAN
    WHEN OTHER
        PERFORM PROCESS-DEFAULT
END-EVALUATE
```

### EVALUATE TRUE ALSO TRUE

Both subjects are `TRUE`, so all objects are conditions.

```cobol
EVALUATE TRUE ALSO TRUE
    WHEN WS-ACCOUNT-TYPE = "S" ALSO WS-BALANCE > 10000
        COMPUTE WS-INTEREST = WS-BALANCE * 0.05
    WHEN WS-ACCOUNT-TYPE = "S" ALSO WS-BALANCE > 1000
        COMPUTE WS-INTEREST = WS-BALANCE * 0.03
    WHEN WS-ACCOUNT-TYPE = "S" ALSO OTHER
        COMPUTE WS-INTEREST = WS-BALANCE * 0.01
    WHEN WS-ACCOUNT-TYPE = "C" ALSO ANY
        MOVE 0 TO WS-INTEREST
    WHEN OTHER
        PERFORM HANDLE-UNKNOWN-ACCOUNT
END-EVALUATE
```

### Using ANY

`ANY` matches any value for the corresponding subject.

```cobol
EVALUATE WS-TRANSACTION-TYPE ALSO WS-AMOUNT
    WHEN "D" ALSO ANY
        PERFORM PROCESS-DEPOSIT
    WHEN "W" ALSO 0 THRU 500
        PERFORM PROCESS-SMALL-WITHDRAWAL
    WHEN "W" ALSO 501 THRU 99999
        PERFORM PROCESS-LARGE-WITHDRAWAL
    WHEN OTHER
        PERFORM PROCESS-ERROR
END-EVALUATE
```

### Comparison with Nested IF

The following `EVALUATE` and nested `IF` are functionally equivalent:

```cobol
*> EVALUATE form
EVALUATE WS-STATUS
    WHEN "A"
        PERFORM PROCESS-ACTIVE
    WHEN "I"
        PERFORM PROCESS-INACTIVE
    WHEN "C"
        PERFORM PROCESS-CLOSED
    WHEN OTHER
        PERFORM PROCESS-UNKNOWN
END-EVALUATE

*> Equivalent nested IF form
IF WS-STATUS = "A"
    PERFORM PROCESS-ACTIVE
ELSE IF WS-STATUS = "I"
    PERFORM PROCESS-INACTIVE
ELSE IF WS-STATUS = "C"
    PERFORM PROCESS-CLOSED
ELSE
    PERFORM PROCESS-UNKNOWN
END-IF
```

The `EVALUATE` form is generally preferred for readability when testing multiple discrete values of a single subject.

---

## See Also

- [IF](if.md) — two-branch conditional statement
- [PERFORM](perform.md) — executes paragraphs or inline statements
- GO TO — unconditional transfer of control
- [Procedure Division Overview](../index.md)
