# GOBACK

The `GOBACK` statement returns control from any program to its caller, or from a main program to the operating system. It is the preferred way to end execution of a subprogram.

- **Standard:** COBOL-85, COBOL 2002, COBOL 2014, COBOL 2023
- **Division:** Procedure Division
- **Category:** Control Flow

---

## Syntax

```cobol
GOBACK
```

---

## Rules

1. In a **main program**, GOBACK terminates the run unit (equivalent to STOP RUN).
2. In a **called subprogram**, GOBACK returns control to the calling program (equivalent to EXIT PROGRAM).
3. In a **user-defined function**, GOBACK returns control to the invoking expression (equivalent to EXIT FUNCTION).
4. After GOBACK from a subprogram, the subprogram's state is preserved — WORKING-STORAGE retains its values for the next CALL (unlike CANCEL, which resets state).
5. GOBACK must be the last statement executed in its sentence. Statements following GOBACK in the same sentence are unreachable.

---

## GOBACK vs STOP RUN vs EXIT PROGRAM

| Statement | Main Program | Subprogram | User-Defined Function |
|-----------|-------------|------------|----------------------|
| `STOP RUN` | Terminates run unit | Terminates **entire** run unit | Not permitted |
| `EXIT PROGRAM` | No effect (continues) | Returns to caller | Not permitted |
| `EXIT FUNCTION` | Not permitted | Not permitted | Returns to caller |
| `GOBACK` | Terminates run unit | Returns to caller | Returns to caller |

**GOBACK is preferred** because it behaves correctly regardless of whether the program is a main program, subprogram, or function. Using STOP RUN in a subprogram is a common bug — it terminates the entire run unit instead of just returning to the caller.

!!! warning "STOP RUN in Subprograms"
    Using `STOP RUN` in a called subprogram terminates all programs in the
    run unit, including the calling program. This is almost never the intended
    behavior. Use `GOBACK` instead.

---

## Examples

### Subprogram

```cobol
       IDENTIFICATION DIVISION.
       PROGRAM-ID. CALC-TAX.

       DATA DIVISION.
       LINKAGE SECTION.
       01  LS-AMOUNT     PIC 9(7)V99.
       01  LS-TAX        PIC 9(7)V99.

       PROCEDURE DIVISION USING LS-AMOUNT LS-TAX.
           COMPUTE LS-TAX = LS-AMOUNT * 0.0825
           GOBACK.
       END PROGRAM CALC-TAX.
```

### Main Program

```cobol
       IDENTIFICATION DIVISION.
       PROGRAM-ID. MAIN-PROG.

       PROCEDURE DIVISION.
           CALL "CALC-TAX" USING WS-AMOUNT WS-TAX
           DISPLAY "Tax: " WS-TAX
           GOBACK.
      *>   In a main program, GOBACK terminates the run unit
       END PROGRAM MAIN-PROG.
```

### Conditional Return

```cobol
       PROCEDURE DIVISION USING LS-INPUT LS-RESULT.
           IF LS-INPUT = SPACES
               MOVE ZEROS TO LS-RESULT
               GOBACK
           END-IF
           PERFORM PROCESS-INPUT
           GOBACK.
```

---

## See Also

- [STOP](stop.md) -- run unit termination
- [EXIT](exit.md) -- EXIT PROGRAM and EXIT FUNCTION
- [CALL](../program-linkage/call.md) -- invoking subprograms
- [Program Attributes](../../identification-division/program-attributes.md) -- INITIAL, COMMON, RECURSIVE
