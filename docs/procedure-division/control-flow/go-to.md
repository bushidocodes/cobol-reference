# GO TO

The `GO TO` statement transfers control to a designated paragraph or section, bypassing the normal sequential flow of execution.

- **Standard:** COBOL-60, COBOL-61, COBOL-61 Extended, COBOL-65, COBOL-68, COBOL-74, COBOL-85, COBOL 2002, COBOL 2014, COBOL 2023
- **Division:** Procedure Division
- **Category:** Control Flow

---

## Syntax

### Format 1 — Unconditional GO TO

```cobol
GO TO procedure-name-1
```

### Format 2 — GO TO DEPENDING ON

```cobol
GO TO procedure-name-1 [procedure-name-2] ...
    DEPENDING ON identifier-1
```

### Format 3 — Altered GO TO (Obsolete)

```cobol
GO TO
```

---

## Rules

### Unconditional GO TO (Format 1)

Control transfers unconditionally to `procedure-name-1`. Any statements following the `GO TO` in the same paragraph are never executed.

```cobol
GO TO 9000-EXIT-PARAGRAPH
```

### GO TO DEPENDING ON (Format 2)

Control transfers to one of the listed procedure names based on the integer value of `identifier-1`.

- If `identifier-1` has the value 1, control transfers to `procedure-name-1`.
- If `identifier-1` has the value 2, control transfers to `procedure-name-2`.
- If `identifier-1` has the value *n*, control transfers to the *n*th procedure name.
- If the value of `identifier-1` is less than 1 or greater than the number of listed procedure names, control falls through to the next statement after the `GO TO`.

```cobol
GO TO PROCESS-ADD
         PROCESS-UPDATE
         PROCESS-DELETE
    DEPENDING ON WS-ACTION-CODE
```

In this example, if `WS-ACTION-CODE` is 1, control passes to `PROCESS-ADD`; if 2, to `PROCESS-UPDATE`; if 3, to `PROCESS-DELETE`.

### Altered GO TO (Format 3 — Obsolete)

A `GO TO` statement without a procedure name serves as the target of an `ALTER` statement. The `ALTER` statement modifies the destination of the `GO TO` at runtime.

```cobol
ALTER SWITCH-PARA TO PROCEED TO NEW-DESTINATION
```

!!! warning "Obsolete Feature"
    The `ALTER` statement and altered `GO TO` were classified as obsolete in COBOL-85 and deleted in COBOL 2002. They produce programs that are extremely difficult to understand and debug, since the transfer destination is determined at runtime. No modern COBOL programs should use this feature.

### GO TO as the Last Statement

A `GO TO` statement must be the last (or only) statement in a sentence or in an imperative statement sequence within a conditional phrase. Any statements written after a `GO TO` in the same sequence are unreachable.

---

## Behavior

### Structured Programming Considerations

The `GO TO` statement disrupts structured, top-down control flow and makes programs harder to read and maintain. The COBOL-85 standard introduced structured constructs — inline `PERFORM`, `EVALUATE`, and explicit scope terminators — that eliminate most needs for `GO TO`.

!!! note "Usage Guidance"
    Many COBOL coding standards restrict or prohibit `GO TO`. Where it is permitted, the most common acceptable use is a forward `GO TO` that branches to an exit paragraph at the end of a performed range. This pattern provides early exit from a paragraph without deeply nested `IF` statements.

    ```cobol
    PERFORM 2000-VALIDATE THRU 2000-VALIDATE-EXIT.

    2000-VALIDATE.
        IF WS-AMOUNT < 0
            MOVE "Negative" TO WS-ERROR
            GO TO 2000-VALIDATE-EXIT
        END-IF
        IF WS-ACCOUNT NOT NUMERIC
            MOVE "Bad account" TO WS-ERROR
            GO TO 2000-VALIDATE-EXIT
        END-IF
        MOVE SPACES TO WS-ERROR.

    2000-VALIDATE-EXIT.
        EXIT.
    ```

### GO TO DEPENDING ON vs EVALUATE

The `GO TO DEPENDING ON` statement transfers control based on a numeric value. The [EVALUATE](evaluate.md) statement offers a more flexible and readable alternative for multi-way branching that does not require transfer of control.

---

## Examples

### Unconditional GO TO

```cobol
PROCEDURE DIVISION.
MAIN-LOGIC.
    PERFORM OPEN-FILES
    PERFORM PROCESS-RECORDS
    PERFORM CLOSE-FILES
    GO TO PROGRAM-DONE.

PROGRAM-DONE.
    STOP RUN.
```

### GO TO DEPENDING ON

```cobol
01 WS-TRANS-TYPE   PIC 9(1).

EVALUATE-TRANSACTION.
    GO TO TRANS-ADD
             TRANS-CHANGE
             TRANS-DELETE
        DEPENDING ON WS-TRANS-TYPE

    DISPLAY "Invalid transaction type: " WS-TRANS-TYPE
    GO TO EVALUATE-TRANSACTION-EXIT.

TRANS-ADD.
    PERFORM ADD-RECORD
    GO TO EVALUATE-TRANSACTION-EXIT.

TRANS-CHANGE.
    PERFORM CHANGE-RECORD
    GO TO EVALUATE-TRANSACTION-EXIT.

TRANS-DELETE.
    PERFORM DELETE-RECORD
    GO TO EVALUATE-TRANSACTION-EXIT.

EVALUATE-TRANSACTION-EXIT.
    EXIT.
```

### GO TO in a PERFORM THRU Range

```cobol
PERFORM 3000-PROCESS THRU 3000-PROCESS-EXIT.

3000-PROCESS.
    READ INPUT-FILE INTO WS-RECORD
        AT END
            MOVE "Y" TO WS-EOF
            GO TO 3000-PROCESS-EXIT
    END-READ
    IF WS-RECORD-TYPE = "H"
        GO TO 3000-PROCESS-EXIT
    END-IF
    ADD 1 TO WS-RECORD-COUNT
    PERFORM 3100-WRITE-OUTPUT.

3000-PROCESS-EXIT.
    EXIT.
```

---

## See Also

- [IF](if.md) — conditional execution
- [EVALUATE](evaluate.md) — multi-branch conditional statement
- [PERFORM](perform.md) — executes paragraphs or inline statements
- [EXIT](exit.md) — provides end points for performed ranges
- [STOP](stop.md) — terminates the run unit
- [Procedure Division Overview](../index.md)
