# STOP

The `STOP` statement terminates the execution of a run unit or temporarily suspends it.

- **Standard:** COBOL-60, COBOL-61, COBOL-61 Extended, COBOL-65, COBOL-68, COBOL-74, COBOL-85, COBOL 2002, COBOL 2014, COBOL 2023
- **Division:** Procedure Division
- **Category:** Control Flow

---

## Syntax

### Format 1 — STOP RUN

```cobol
STOP RUN
```

### Format 2 — STOP RUN WITH STATUS

```cobol
STOP RUN WITH {NORMAL | ERROR} STATUS [identifier-1 | literal-1]
```

### Format 3 — STOP Literal (Obsolete)

```cobol
STOP literal-1
```

---

## Rules

### STOP RUN (Format 1)

`STOP RUN` terminates the current run unit. All files opened by the run unit are closed, and control returns to the operating system or calling environment.

- If the program is the main program, `STOP RUN` ends the process.
- If the program is a subprogram invoked by `CALL`, `STOP RUN` terminates the entire run unit, including the calling program. To return control to the calling program instead, the `EXIT PROGRAM` or `GOBACK` statement is used.

### STOP RUN WITH STATUS (Format 2)

!!! note "COBOL 2002"
    The `WITH STATUS` phrase was introduced in COBOL 2002.

The `WITH STATUS` phrase specifies a return status to the operating environment upon termination.

- `WITH NORMAL STATUS` indicates successful completion. If `identifier-1` or `literal-1` is specified, its value is passed as the return code.
- `WITH ERROR STATUS` indicates abnormal completion. If `identifier-1` or `literal-1` is specified, its value is passed as the return code.

```cobol
STOP RUN WITH NORMAL STATUS 0
STOP RUN WITH ERROR STATUS 12
STOP RUN WITH NORMAL STATUS WS-RETURN-CODE
```

The behavior when no value is specified is implementation-defined, but typically a value of zero is returned for `NORMAL STATUS` and a nonzero value for `ERROR STATUS`.

### STOP Literal (Format 3 — Obsolete)

`STOP literal-1` suspends execution and displays `literal-1` to the operator. Execution resumes only after the operator acknowledges the message.

!!! warning "Obsolete Feature"
    `STOP literal` was classified as obsolete in COBOL-85 and deleted in COBOL 2002. The `DISPLAY` and `ACCEPT` statements provide a more flexible alternative for operator communication.

```cobol
*> Obsolete — do not use in new programs
STOP "Check printer alignment"
```

---

## Behavior

### Effect on Program State

When `STOP RUN` executes:

1. All files opened by the run unit are closed. The effect is as if a `CLOSE` statement without any optional phrases were executed for each open file.
2. All resources held by the run unit are released.
3. Control returns to the operating system or the environment that initiated the run unit.

### Difference from GOBACK

`STOP RUN` always terminates the entire run unit, regardless of the call depth. `GOBACK`, when executed in a subprogram, returns control to the calling program, leaving the calling program's run unit active. When executed in the main program, `GOBACK` behaves identically to `STOP RUN`.

| Context | `STOP RUN` | `GOBACK` |
|---------|-----------|----------|
| Main program | Terminates run unit | Terminates run unit |
| Subprogram | Terminates entire run unit | Returns to caller |

### Statements After STOP RUN

Any statements written after `STOP RUN` in the same paragraph are unreachable. The compiler may issue a warning but is not required to do so.

---

## Examples

### Basic STOP RUN

```cobol
PROCEDURE DIVISION.
MAIN-LOGIC.
    PERFORM INITIALIZE-PROGRAM
    PERFORM PROCESS-DATA
    PERFORM FINALIZE-PROGRAM
    STOP RUN.
```

### STOP RUN with Return Code

```cobol
PROCEDURE DIVISION.
MAIN-LOGIC.
    PERFORM PROCESS-INPUT
    IF WS-ERROR-FLAG = "Y"
        DISPLAY "Processing failed"
        STOP RUN WITH ERROR STATUS 8
    END-IF
    DISPLAY "Processing complete"
    STOP RUN WITH NORMAL STATUS 0.
```

### STOP RUN in a Called Subprogram

```cobol
*> --- Main program ---
PROCEDURE DIVISION.
    CALL "SUBPROG1" USING WS-DATA
    DISPLAY "This line is reached if SUBPROG1 uses GOBACK"
    DISPLAY "This line is NOT reached if SUBPROG1 uses STOP RUN"
    STOP RUN.

*> --- Subprogram SUBPROG1 ---
IDENTIFICATION DIVISION.
PROGRAM-ID. SUBPROG1.

PROCEDURE DIVISION USING LS-DATA.
    IF LS-DATA = SPACES
        STOP RUN             *> Terminates the entire run unit
    END-IF
    PERFORM PROCESS-DATA
    GOBACK.                  *> Returns control to the caller
```

### Conditional Termination

```cobol
PERFORM VALIDATE-INPUT
IF WS-VALID = "N"
    DISPLAY "Invalid input — program terminated"
    STOP RUN WITH ERROR STATUS 4
END-IF
```

---

## See Also

- [EXIT](exit.md) — exits a paragraph, section, or called program
- [GO TO](go-to.md) — unconditional transfer of control
- [PERFORM](perform.md) — executes paragraphs or inline statements
- GOBACK — returns to calling program or terminates run unit
- [Procedure Division Overview](../index.md)
