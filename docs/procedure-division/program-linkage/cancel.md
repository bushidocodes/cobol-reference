# CANCEL

The `CANCEL` statement releases the resources of a previously called program and restores it to its initial state so that the next `CALL` to that program behaves as a first-time invocation.

- **Standard:** COBOL-74, COBOL-85, COBOL 2002, COBOL 2014, COBOL 2023
- **Division:** Procedure Division
- **Category:** Program Linkage

---

## Syntax

```cobol
CANCEL {identifier-1 | literal-1} ...
```

---

## Rules

### Program Identification

`literal-1` or `identifier-1` specifies the program name of the program to cancel. The name must match the `PROGRAM-ID` of the target program. Multiple programs may be cancelled in a single `CANCEL` statement.

```cobol
CANCEL "SUBPROG-A"
CANCEL "SUBPROG-A" "SUBPROG-B" "SUBPROG-C"
CANCEL WS-PROGRAM-NAME
```

### Prerequisites

- The program identified must have been previously called by the current run unit, either directly or indirectly.
- The program must not be currently active (i.e., it must have returned control via `EXIT PROGRAM`, `GOBACK`, or equivalent). Cancelling an active program produces undefined behavior.

### Effect on Program State

When a program is cancelled:

1. All Working-Storage data items are released. On the next `CALL`, they are reinitialized according to their `VALUE` clauses (or to an undefined state if no `VALUE` clause is specified).
2. All files opened by the cancelled program are closed. The effect is as if a `CLOSE` statement without optional phrases were executed for each open file.
3. Any internal state maintained by the program is discarded.
4. If the program was loaded dynamically, the runtime system may release the memory occupied by the program module (implementation-defined).

### Cancelling a Program That Was Not Called

If `CANCEL` is executed for a program that has not been called in the current run unit, the statement has no effect.

### Programs with the INITIAL Attribute

A program defined with `PROGRAM-ID. program-name IS INITIAL` is automatically reinitialized on every `CALL`. Cancelling such a program is permitted but has no additional effect beyond what already occurs on each call.

---

## Behavior

### When to Use CANCEL

`CANCEL` is useful in the following situations:

- **Memory management:** In systems with constrained memory, cancelling infrequently used programs releases the storage they occupy.
- **Reinitializing program state:** When a program must be reset to its initial state, `CANCEL` followed by `CALL` achieves this without requiring the called program to implement its own reinitialization logic.
- **Dynamic module replacement:** After cancelling a dynamically loaded program, the next `CALL` loads the module again. If the module has been replaced on disk, the new version is loaded.

### CANCEL and Static Calls

For statically linked programs, `CANCEL` reinitializes the program's state but typically does not release memory, since the program code is part of the same executable. The precise behavior is implementation-defined.

### Implicit CANCEL on STOP RUN

When `STOP RUN` executes, the runtime environment implicitly cancels all programs in the run unit. There is no need to explicitly cancel programs before `STOP RUN`.

---

## Examples

### Basic CANCEL and Re-CALL

```cobol
CALL "ACCUM" USING WS-VALUE-1
CALL "ACCUM" USING WS-VALUE-2
*> At this point, ACCUM retains accumulated state from both calls.

CANCEL "ACCUM"
*> ACCUM is now in its initial state.

CALL "ACCUM" USING WS-VALUE-3
*> ACCUM starts fresh, as if called for the first time.
```

### Cancelling Multiple Programs

```cobol
CANCEL "VALIDATE-INPUT"
       "FORMAT-OUTPUT"
       "WRITE-REPORT"
```

### Dynamic CANCEL

```cobol
01 WS-MODULE-TABLE.
   05 WS-MODULE-NAME PIC X(8) OCCURS 10 TIMES.

PERFORM VARYING WS-I FROM 1 BY 1
    UNTIL WS-I > 10
    IF WS-MODULE-NAME(WS-I) NOT = SPACES
        CANCEL WS-MODULE-NAME(WS-I)
    END-IF
END-PERFORM
```

### CANCEL for State Reset

```cobol
*> Process first batch
PERFORM VARYING WS-I FROM 1 BY 1
    UNTIL WS-I > WS-BATCH-1-COUNT
    CALL "BATCH-PROC" USING BATCH-1-REC(WS-I)
END-PERFORM

*> Reset the processor for a new batch
CANCEL "BATCH-PROC"

*> Process second batch with fresh state
PERFORM VARYING WS-I FROM 1 BY 1
    UNTIL WS-I > WS-BATCH-2-COUNT
    CALL "BATCH-PROC" USING BATCH-2-REC(WS-I)
END-PERFORM
```

### Conditional CANCEL

```cobol
IF WS-RELOAD-MODULE = "Y"
    CANCEL "DATA-LOOKUP"
    CALL "DATA-LOOKUP" USING WS-REFRESH-PARM
END-IF
```

---

## See Also

- [CALL](call.md) — transfers control to another program
- [EXIT](../control-flow/exit.md) — exits a called program (EXIT PROGRAM)
- [STOP](../control-flow/stop.md) — terminates the run unit
- GOBACK — returns to calling program or terminates run unit
- [Procedure Division Overview](../index.md)
