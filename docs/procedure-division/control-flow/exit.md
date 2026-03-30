# EXIT

The `EXIT` statement provides a common end point for a performed range, terminates execution of a paragraph, section, performed loop, or called program.

- **Standard:** COBOL-60, COBOL-61, COBOL-61 Extended, COBOL-65, COBOL-68, COBOL-74, COBOL-85, COBOL 2002, COBOL 2014, COBOL 2023
- **Division:** Procedure Division
- **Category:** Control Flow

---

## Syntax

### Format 1 — Basic EXIT

```cobol
EXIT
```

### Format 2 — EXIT PARAGRAPH

```cobol
EXIT PARAGRAPH
```

### Format 3 — EXIT SECTION

```cobol
EXIT SECTION
```

### Format 4 — EXIT PROGRAM

```cobol
EXIT PROGRAM [{WITH | GIVING} {NORMAL | ERROR} STATUS [identifier-1 | literal-1]]
```

### Format 5 — EXIT PERFORM

```cobol
EXIT PERFORM
```

### Format 6 — EXIT PERFORM CYCLE

```cobol
EXIT PERFORM CYCLE
```

---

## Rules

### Basic EXIT (Format 1)

The basic `EXIT` statement is a no-operation. It generates no executable code. Its sole purpose is to provide a paragraph name that serves as an end point for a `PERFORM THRU` range.

In COBOL-68 and COBOL-74, `EXIT` must be the only statement in its paragraph. COBOL-85 and later standards relax this requirement, but the conventional usage remains a standalone `EXIT` in a named paragraph.

```cobol
2000-PROCESS-EXIT.
    EXIT.
```

### EXIT PARAGRAPH (Format 2)

!!! note "COBOL-85"
    `EXIT PARAGRAPH` was introduced in COBOL-85.

`EXIT PARAGRAPH` causes control to transfer to the end of the current paragraph. Execution continues with the next paragraph in sequence, or if the paragraph is the last paragraph in a performed range, control returns to the `PERFORM` statement.

```cobol
2000-VALIDATE.
    IF WS-AMOUNT < 0
        EXIT PARAGRAPH
    END-IF
    PERFORM 2100-PROCESS-AMOUNT.
```

### EXIT SECTION (Format 3)

!!! note "COBOL-85"
    `EXIT SECTION` was introduced in COBOL-85.

`EXIT SECTION` causes control to transfer to the end of the current section. Execution continues with the next section in sequence, or if the section is a performed range, control returns to the `PERFORM` statement.

### EXIT PROGRAM (Format 4)

`EXIT PROGRAM` returns control from a called program to the calling program. The calling program resumes execution with the statement following the `CALL` that invoked the subprogram.

- If `EXIT PROGRAM` is executed in a program that is not a called subprogram (i.e., the main program), the statement has no effect and control passes to the next executable statement.
- All files opened by the called program remain open unless explicitly closed.

!!! note "COBOL 2002"
    The `WITH STATUS` and `GIVING STATUS` phrases were introduced in COBOL 2002. They specify a return status passed to the calling program.

```cobol
EXIT PROGRAM WITH NORMAL STATUS 0
EXIT PROGRAM WITH ERROR STATUS 8
```

### EXIT PERFORM (Format 5)

!!! note "COBOL 2002"
    `EXIT PERFORM` was introduced in COBOL 2002.

`EXIT PERFORM` causes control to transfer to the statement following the `END-PERFORM` of the innermost enclosing inline `PERFORM`. It is analogous to `break` in C-family languages.

- `EXIT PERFORM` applies only to inline `PERFORM` statements. It must not appear outside an inline `PERFORM`.

```cobol
PERFORM VARYING WS-I FROM 1 BY 1
    UNTIL WS-I > WS-TABLE-SIZE
    IF TBL-KEY(WS-I) = WS-SEARCH-KEY
        MOVE WS-I TO WS-FOUND-INDEX
        EXIT PERFORM
    END-IF
END-PERFORM
```

### EXIT PERFORM CYCLE (Format 6)

!!! note "COBOL 2002"
    `EXIT PERFORM CYCLE` was introduced in COBOL 2002.

`EXIT PERFORM CYCLE` causes control to transfer to the loop-control mechanism of the innermost enclosing inline `PERFORM`, beginning the next iteration. It is analogous to `continue` in C-family languages.

- For `PERFORM VARYING`, the varying identifier is incremented and the condition is tested.
- For `PERFORM UNTIL`, the condition is tested.
- For `PERFORM TIMES`, the iteration counter is incremented and tested.

```cobol
PERFORM VARYING WS-I FROM 1 BY 1
    UNTIL WS-I > WS-TABLE-SIZE
    IF TBL-STATUS(WS-I) = "D"
        EXIT PERFORM CYCLE
    END-IF
    PERFORM 3000-PROCESS-RECORD
END-PERFORM
```

---

## Behavior

### GOBACK as an Alternative to EXIT PROGRAM

The `GOBACK` statement provides an alternative to `EXIT PROGRAM`. When executed in a called program, `GOBACK` returns control to the caller. When executed in a main program, `GOBACK` terminates the run unit (equivalent to `STOP RUN`). `GOBACK` works consistently regardless of whether the program is a main program or subprogram, making it the preferred choice in many coding standards.

### EXIT PROGRAM in the Main Program

If `EXIT PROGRAM` is executed in a program that was not called by another program, the statement is ignored. This differs from `GOBACK`, which terminates the run unit when executed in a main program.

### Scope of EXIT PERFORM

`EXIT PERFORM` and `EXIT PERFORM CYCLE` apply only to the innermost enclosing inline `PERFORM`. To exit an outer loop, additional logic (such as a flag variable) is required.

---

## Examples

### EXIT as a PERFORM THRU End Point

```cobol
PERFORM 2000-VALIDATE THRU 2000-VALIDATE-EXIT.

2000-VALIDATE.
    IF WS-CODE NOT NUMERIC
        MOVE "Invalid code" TO WS-ERROR
        GO TO 2000-VALIDATE-EXIT
    END-IF
    IF WS-AMOUNT > WS-LIMIT
        MOVE "Over limit" TO WS-ERROR
        GO TO 2000-VALIDATE-EXIT
    END-IF
    MOVE SPACES TO WS-ERROR.

2000-VALIDATE-EXIT.
    EXIT.
```

### EXIT PARAGRAPH

```cobol
3000-CHECK-RECORD.
    IF WS-RECORD-TYPE = "H"
        EXIT PARAGRAPH
    END-IF
    IF WS-RECORD-TYPE = "T"
        EXIT PARAGRAPH
    END-IF
    ADD 1 TO WS-DETAIL-COUNT
    PERFORM 3100-PROCESS-DETAIL.
```

### EXIT PROGRAM in a Called Subprogram

```cobol
IDENTIFICATION DIVISION.
PROGRAM-ID. VALIDATE-ACCT.

DATA DIVISION.
LINKAGE SECTION.
01 LS-ACCOUNT   PIC X(10).
01 LS-RESULT    PIC X(1).

PROCEDURE DIVISION USING LS-ACCOUNT LS-RESULT.
    IF LS-ACCOUNT = SPACES
        MOVE "E" TO LS-RESULT
        EXIT PROGRAM
    END-IF
    PERFORM VALIDATE-ACCOUNT-NUMBER
    EXIT PROGRAM.
```

### EXIT PERFORM — Early Loop Exit

```cobol
MOVE "N" TO WS-FOUND
PERFORM VARYING WS-I FROM 1 BY 1
    UNTIL WS-I > 500
    IF CUST-ID(WS-I) = WS-TARGET-ID
        MOVE "Y" TO WS-FOUND
        MOVE WS-I TO WS-FOUND-INDEX
        EXIT PERFORM
    END-IF
END-PERFORM

IF WS-FOUND = "Y"
    DISPLAY "Customer found at index " WS-FOUND-INDEX
ELSE
    DISPLAY "Customer not found"
END-IF
```

### EXIT PERFORM CYCLE — Skip Iteration

```cobol
PERFORM VARYING WS-I FROM 1 BY 1
    UNTIL WS-I > WS-NUM-EMPLOYEES
    IF EMP-DEPT(WS-I) NOT = WS-TARGET-DEPT
        EXIT PERFORM CYCLE
    END-IF
    ADD EMP-SALARY(WS-I) TO WS-DEPT-TOTAL
    ADD 1 TO WS-DEPT-COUNT
END-PERFORM
```

### Nested EXIT PERFORM

```cobol
MOVE "N" TO WS-FOUND
PERFORM VARYING WS-ROW FROM 1 BY 1
    UNTIL WS-ROW > 100 OR WS-FOUND = "Y"
    PERFORM VARYING WS-COL FROM 1 BY 1
        UNTIL WS-COL > 50
        IF GRID-CELL(WS-ROW, WS-COL) = WS-TARGET
            MOVE "Y" TO WS-FOUND
            EXIT PERFORM
        END-IF
    END-PERFORM
END-PERFORM
```

---

## See Also

- [PERFORM](perform.md) — executes paragraphs or inline statements
- [GO TO](go-to.md) — unconditional transfer of control
- [STOP](stop.md) — terminates the run unit
- [CONTINUE](continue.md) — explicit no-operation statement
- [GOBACK](goback.md) -- returns to calling program or terminates run unit
- [CALL](../program-linkage/call.md) -- invoking subprograms
