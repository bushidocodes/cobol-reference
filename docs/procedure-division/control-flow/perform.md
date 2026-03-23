# PERFORM

The `PERFORM` statement transfers control to one or more paragraphs or sections and returns control upon completion, or executes a set of inline statements. It supports counted iteration, condition-controlled looping, and multi-level varying with nested loop control.

- **Standard:** COBOL-60, COBOL-61, COBOL-61 Extended, COBOL-65, COBOL-68, COBOL-74, COBOL-85, COBOL 2002, COBOL 2014, COBOL 2023
- **Division:** Procedure Division
- **Category:** Control Flow

---

## Syntax

### Format 1 — Basic Out-of-Line PERFORM

```cobol
PERFORM procedure-name-1 [{THROUGH | THRU} procedure-name-2]
```

### Format 2 — PERFORM TIMES

```cobol
PERFORM procedure-name-1 [{THROUGH | THRU} procedure-name-2]
    {identifier-1 | integer-1} TIMES
```

### Format 3 — PERFORM UNTIL

```cobol
PERFORM procedure-name-1 [{THROUGH | THRU} procedure-name-2]
    [WITH TEST {BEFORE | AFTER}]
    UNTIL condition-1
```

### Format 4 — PERFORM VARYING

```cobol
PERFORM procedure-name-1 [{THROUGH | THRU} procedure-name-2]
    [WITH TEST {BEFORE | AFTER}]
    VARYING {identifier-2 | index-name-1}
        FROM {identifier-3 | index-name-2 | literal-1}
        BY {identifier-4 | literal-2}
        UNTIL condition-1
    [AFTER {identifier-5 | index-name-3}
        FROM {identifier-6 | index-name-4 | literal-3}
        BY {identifier-7 | literal-4}
        UNTIL condition-2] ...
```

### Format 5 — Inline PERFORM

```cobol
PERFORM [WITH TEST {BEFORE | AFTER}]
    [VARYING ... FROM ... BY ... UNTIL ...
        [AFTER ... FROM ... BY ... UNTIL ...] ...]
    [{identifier-1 | integer-1} TIMES]
    [UNTIL condition-1]
    statement-1 ...
END-PERFORM
```

!!! note "COBOL-85"
    Inline `PERFORM` (Format 5 with `END-PERFORM`) was introduced in COBOL-85. COBOL-68 and COBOL-74 support only out-of-line `PERFORM`.

---

## Rules

### Out-of-Line vs Inline

- **Out-of-line PERFORM** (Formats 1–4) transfers control to the named paragraph or section. After the last statement of the performed range executes, control returns to the statement following the `PERFORM`.
- **Inline PERFORM** (Format 5) executes the statements between `PERFORM` and `END-PERFORM` directly. No separate paragraph is needed.

### PERFORM THRU

When `THRU` is specified, the performed range includes all paragraphs (or sections) from `procedure-name-1` through `procedure-name-2`, inclusive, in their physical order in the source.

```cobol
PERFORM 1000-INIT THRU 1000-INIT-EXIT.
```

!!! warning "Pitfall: PERFORM THRU"
    `PERFORM THRU` relies on the physical ordering of paragraphs. Inserting a new paragraph between the start and end paragraphs inadvertently includes it in the performed range. Many coding standards discourage `PERFORM THRU` in favor of `PERFORM` of a single paragraph that uses explicit `GO TO` for its exit structure.

### PERFORM TIMES

The statements in the performed range are executed `identifier-1` or `integer-1` times. The value of the operand is evaluated once, before the first execution. If the value is zero or negative, the performed range is not executed.

```cobol
PERFORM PRINT-HEADER 3 TIMES
```

### PERFORM UNTIL

The performed range executes repeatedly until `condition-1` becomes true.

- **TEST BEFORE** (default): The condition is tested before each execution. If the condition is true initially, the performed range is not executed at all.
- **TEST AFTER**: The condition is tested after each execution. The performed range is always executed at least once.

```cobol
PERFORM READ-RECORD
    WITH TEST BEFORE
    UNTIL WS-EOF = "Y"
```

!!! note "COBOL-85"
    The `WITH TEST BEFORE` and `WITH TEST AFTER` phrases were introduced in COBOL-85. In COBOL-68 and COBOL-74, the test is always performed before execution (equivalent to `TEST BEFORE`).

### PERFORM VARYING

The `VARYING` phrase provides loop-counter control. Before and during execution, the varying identifier is modified:

1. The varying identifier is set to its `FROM` value.
2. The `UNTIL` condition is tested (if `TEST BEFORE`).
3. The performed range is executed.
4. The varying identifier is incremented by its `BY` value.
5. Steps 2–4 repeat until the condition is true.

With `TEST AFTER`, steps 2 and 3 are swapped (execute first, then test).

```cobol
PERFORM PROCESS-ELEMENT
    VARYING WS-INDEX FROM 1 BY 1
    UNTIL WS-INDEX > 100
```

### Nested Loops with AFTER

The `AFTER` phrase introduces inner loops. When the inner loop's condition becomes true, the outer varying identifier is incremented and the inner identifier is reset to its `FROM` value.

```cobol
PERFORM PROCESS-CELL
    VARYING WS-ROW FROM 1 BY 1 UNTIL WS-ROW > 10
    AFTER WS-COL FROM 1 BY 1 UNTIL WS-COL > 5
```

This is equivalent to:

```cobol
MOVE 1 TO WS-ROW
PERFORM UNTIL WS-ROW > 10
    MOVE 1 TO WS-COL
    PERFORM UNTIL WS-COL > 5
        PERFORM PROCESS-CELL
        ADD 1 TO WS-COL
    END-PERFORM
    ADD 1 TO WS-ROW
END-PERFORM
```

Multiple `AFTER` phrases may be specified for three or more levels of nesting.

---

## Behavior

### Execution Flow for Out-of-Line PERFORM

1. The current execution point is saved.
2. Control transfers to the first statement of the performed range.
3. Statements execute in sequence through the range.
4. After the last statement of the range executes, control returns to the statement following the `PERFORM`.

### Recursion and Active PERFORMs

An out-of-line `PERFORM` sets up a return point at the end of the performed range. If the same range is performed recursively (a `PERFORM` of a paragraph that itself performs the same paragraph), behavior is undefined in COBOL-85. Some compilers support recursive `PERFORM` as an extension.

!!! warning "Overlapping PERFORM Ranges"
    If two active `PERFORM` statements have overlapping ranges, the results are undefined. For example, if paragraph A performs paragraphs B THRU D, and paragraph C within that range performs paragraphs D THRU E, the ranges overlap at paragraph D. This produces unpredictable behavior.

### State of Varying Identifier After Loop

After a `PERFORM VARYING` loop completes:

- With `TEST BEFORE`, the varying identifier holds the first value that caused the condition to be true.
- With `TEST AFTER`, the varying identifier holds the value it had when the condition was last tested and found true (after the last increment).

---

## Examples

### Basic Out-of-Line PERFORM

```cobol
PROCEDURE DIVISION.
MAIN-LOGIC.
    PERFORM INITIALIZE-DATA
    PERFORM PROCESS-DATA
    PERFORM FINALIZE-DATA
    STOP RUN.

INITIALIZE-DATA.
    OPEN INPUT IN-FILE OUTPUT OUT-FILE.

PROCESS-DATA.
    READ IN-FILE INTO WS-RECORD
        AT END MOVE "Y" TO WS-EOF
    END-READ
    PERFORM UNTIL WS-EOF = "Y"
        PERFORM PROCESS-RECORD
        READ IN-FILE INTO WS-RECORD
            AT END MOVE "Y" TO WS-EOF
        END-READ
    END-PERFORM.

PROCESS-RECORD.
    WRITE OUT-RECORD FROM WS-RECORD.

FINALIZE-DATA.
    CLOSE IN-FILE OUT-FILE.
```

### Inline PERFORM with UNTIL

```cobol
MOVE "N" TO WS-EOF
PERFORM UNTIL WS-EOF = "Y"
    READ TRANS-FILE INTO WS-TRANS
        AT END
            MOVE "Y" TO WS-EOF
        NOT AT END
            PERFORM PROCESS-TRANSACTION
    END-READ
END-PERFORM
```

### PERFORM TIMES

```cobol
MOVE SPACES TO WS-SEPARATOR-LINE
PERFORM 3 TIMES
    DISPLAY WS-SEPARATOR-LINE
END-PERFORM
```

### PERFORM VARYING (Simple Counter)

```cobol
PERFORM VARYING WS-I FROM 1 BY 1
    UNTIL WS-I > WS-TABLE-SIZE
    IF TBL-ITEM(WS-I) = WS-SEARCH-KEY
        MOVE WS-I TO WS-FOUND-INDEX
        MOVE "Y" TO WS-FOUND
    END-IF
END-PERFORM
```

### PERFORM VARYING with AFTER (Nested Loop)

```cobol
01 WS-MATRIX.
   05 WS-ROW OCCURS 10 TIMES.
      10 WS-CELL OCCURS 5 TIMES  PIC 9(4).

01 WS-R   PIC 99.
01 WS-C   PIC 99.
01 WS-SUM PIC 9(8) VALUE 0.

PERFORM VARYING WS-R FROM 1 BY 1 UNTIL WS-R > 10
    AFTER WS-C FROM 1 BY 1 UNTIL WS-C > 5
    ADD WS-CELL(WS-R, WS-C) TO WS-SUM
END-PERFORM

DISPLAY "Matrix sum: " WS-SUM
```

### PERFORM with TEST AFTER

```cobol
*> Prompt until valid input is received (always prompts at least once)
PERFORM WITH TEST AFTER
    UNTIL WS-INPUT-VALID = "Y"
    DISPLAY "Enter Y or N: "
    ACCEPT WS-USER-INPUT FROM CONSOLE
    EVALUATE WS-USER-INPUT
        WHEN "Y"
        WHEN "N"
            MOVE "Y" TO WS-INPUT-VALID
        WHEN OTHER
            DISPLAY "Invalid input. Try again."
            MOVE "N" TO WS-INPUT-VALID
    END-EVALUATE
END-PERFORM
```

### PERFORM THRU with EXIT Paragraph

A common pattern uses `PERFORM THRU` with a dedicated exit paragraph:

```cobol
PERFORM 2000-VALIDATE THRU 2000-VALIDATE-EXIT.

2000-VALIDATE.
    IF WS-AMOUNT < 0
        MOVE "Negative amount" TO WS-ERROR-MSG
        GO TO 2000-VALIDATE-EXIT
    END-IF
    IF WS-AMOUNT > WS-CREDIT-LIMIT
        MOVE "Over limit" TO WS-ERROR-MSG
        GO TO 2000-VALIDATE-EXIT
    END-IF
    MOVE SPACES TO WS-ERROR-MSG.

2000-VALIDATE-EXIT.
    EXIT.
```

### Decrementing Loop

```cobol
PERFORM VARYING WS-I FROM 100 BY -1
    UNTIL WS-I < 1
    DISPLAY "Countdown: " WS-I
END-PERFORM
```

### Three-Level Nested Loop

```cobol
PERFORM VARYING WS-X FROM 1 BY 1 UNTIL WS-X > 3
    AFTER WS-Y FROM 1 BY 1 UNTIL WS-Y > 4
    AFTER WS-Z FROM 1 BY 1 UNTIL WS-Z > 5
    MOVE WS-VALUE TO WS-CUBE(WS-X, WS-Y, WS-Z)
END-PERFORM
```

---

## See Also

- [IF](if.md) — conditional execution
- [EVALUATE](evaluate.md) — multi-branch conditional
- GO TO — unconditional transfer of control
- EXIT — provides a common end point or exits a loop
- [Procedure Division Overview](../index.md)
