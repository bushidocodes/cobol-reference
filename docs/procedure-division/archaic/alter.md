# ALTER (Removed)

The `ALTER` statement modified the destination of a `GO TO` statement at runtime, allowing the program to change its own control flow dynamically.

- **Introduced:** COBOL-60
- **Obsolete:** COBOL-85
- **Removed:** COBOL 2002
- **Status:** Removed

!!! warning "Removed"
    `ALTER` was classified as obsolete in COBOL-85 and removed in COBOL 2002.
    It produces programs that are extremely difficult to understand, debug,
    and maintain because transfer destinations are determined at runtime
    and may change unpredictably. Use `EVALUATE`, `PERFORM`, or condition
    variables instead.

---

## Syntax

```cobol
ALTER procedure-name-1 TO PROCEED TO procedure-name-2
    [ procedure-name-3 TO PROCEED TO procedure-name-4 ] ...
```

---

## Rules

1. `procedure-name-1` (and `procedure-name-3`, etc.) must be a paragraph containing a single `GO TO` statement — the "alterable GO TO."
2. `procedure-name-2` (and `procedure-name-4`, etc.) is the new destination that the GO TO will branch to after the ALTER is executed.
3. Multiple GO TO targets can be altered in a single ALTER statement.
4. The alterable GO TO must be written as `GO TO` with no procedure-name (the "optional GO TO"), or with an initial procedure-name that serves as the default destination.

---

## How It Worked

An alterable paragraph contained a single GO TO statement:

```cobol
SWITCH-PARA.
    GO TO DEFAULT-ROUTINE.
```

An ALTER statement elsewhere in the program could change where that GO TO branched:

```cobol
ALTER SWITCH-PARA TO PROCEED TO SPECIAL-ROUTINE
```

After this ALTER executed, when control reached SWITCH-PARA, the GO TO would branch to SPECIAL-ROUTINE instead of DEFAULT-ROUTINE.

---

## Examples

### Simple Switch

```cobol
       PROCEDURE DIVISION.
       MAIN-PROCESS.
           PERFORM READ-RECORD
           GO TO PROCESS-SWITCH.

       PROCESS-SWITCH.
           GO TO FIRST-TIME-PROCESS.

       FIRST-TIME-PROCESS.
           DISPLAY "Processing first record"
           ALTER PROCESS-SWITCH TO PROCEED TO
               NORMAL-PROCESS
           GO TO MAIN-PROCESS.

       NORMAL-PROCESS.
           DISPLAY "Processing subsequent record"
           GO TO MAIN-PROCESS.
```

After the first record, PROCESS-SWITCH is altered to branch to NORMAL-PROCESS instead of FIRST-TIME-PROCESS.

### Multiple Alterations

```cobol
ALTER CLOSING-SWITCH TO PROCEED TO CLOSING-ROUTINE-1
       STATE-SWITCH TO PROCEED TO NEW-YORK
       TABLE-COMPUTATION TO PROCEED TO
           GENERAL-COMPUTATION
```

---

## Why It Was Removed

ALTER created what are sometimes called "tracing problems" — the actual flow of the program could only be determined by tracing every possible ALTER statement that might have executed before reaching the alterable GO TO. This made programs:

- **Unreadable** — the destination of a GO TO was not apparent from reading the code at the GO TO location
- **Unpredictable** — multiple ALTER statements could change the same GO TO, and the effective destination depended on execution history
- **Unmaintainable** — modifying one part of a program could silently change the behavior of a distant paragraph

The "COBOL Minefield Detection" study (Veerman & Verhoeven, 2006) identified ALTER as one of the most hazardous constructs in legacy COBOL code.

---

## Modern Alternatives

### Using EVALUATE (preferred)

```cobol
       WORKING-STORAGE SECTION.
       01  WS-PROCESS-STATE  PIC X VALUE "F".
           88  FIRST-TIME    VALUE "F".
           88  NORMAL        VALUE "N".

       PROCEDURE DIVISION.
           EVALUATE TRUE
               WHEN FIRST-TIME
                   PERFORM FIRST-TIME-PROCESS
                   SET NORMAL TO TRUE
               WHEN NORMAL
                   PERFORM NORMAL-PROCESS
           END-EVALUATE
```

### Using a Condition Variable

```cobol
       IF FIRST-TIME
           PERFORM FIRST-TIME-PROCESS
           SET NORMAL TO TRUE
       ELSE
           PERFORM NORMAL-PROCESS
       END-IF
```

---

## See Also

- [GO TO](../control-flow/go-to.md) -- unconditional branching
- [EVALUATE](../control-flow/evaluate.md) -- multi-way conditional (modern replacement)
- [PERFORM](../control-flow/perform.md) -- structured procedure invocation
