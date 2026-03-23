# Program Attributes (INITIAL, COMMON, RECURSIVE)

The `PROGRAM-ID` paragraph accepts optional clauses that control a program's initialization behavior, visibility within nested program structures, and support for recursive invocation.

- **Standard:** INITIAL and COMMON: COBOL-85. RECURSIVE: COBOL 2002.

---

## Syntax

```cobol
PROGRAM-ID. program-name [ IS { INITIAL | COMMON | RECURSIVE } PROGRAM ].
```

Multiple attributes may be combined (where supported by the compiler):

```cobol
PROGRAM-ID. program-name IS INITIAL COMMON PROGRAM.
```

---

## INITIAL

When `INITIAL` is specified, the program is placed in its initial state each time it is called. This means:

- All `WORKING-STORAGE SECTION` data items are reinitialized to their `VALUE` clauses (or to their default values if no VALUE is specified).
- All internal files are closed.
- All `PERFORM` ranges are reset.
- All `ALTER` references are restored to their initial states.

Without `INITIAL`, a program retains the state of its data items, open files, and PERFORM ranges across successive invocations (i.e., it behaves as a persistent subprogram).

### When to Use INITIAL

- Programs that must not carry state between calls (e.g., initialization routines, one-shot utilities).
- Programs that risk data corruption if stale values persist from a prior call.
- Programs called from multiple callers that expect a clean slate each time.

### Example

```cobol
      *> ============================================
      *> Accumulator WITHOUT INITIAL -- retains state
      *> ============================================
       IDENTIFICATION DIVISION.
       PROGRAM-ID. ACCUMULATE.

       DATA DIVISION.
       WORKING-STORAGE SECTION.
       01  WS-TOTAL     PIC 9(9)V99 VALUE 0.

       LINKAGE SECTION.
       01  LS-AMOUNT    PIC 9(7)V99.
       01  LS-RESULT    PIC 9(9)V99.

       PROCEDURE DIVISION USING LS-AMOUNT LS-RESULT.
           ADD LS-AMOUNT TO WS-TOTAL
           MOVE WS-TOTAL TO LS-RESULT
           GOBACK.
       END PROGRAM ACCUMULATE.
```

Each call adds to the running total because `WS-TOTAL` retains its value.

```cobol
      *> ============================================
      *> Formatter WITH INITIAL -- fresh state each call
      *> ============================================
       IDENTIFICATION DIVISION.
       PROGRAM-ID. FORMAT-RECORD IS INITIAL PROGRAM.

       DATA DIVISION.
       WORKING-STORAGE SECTION.
       01  WS-LINE-COUNT    PIC 9(5)  VALUE 0.
       01  WS-ERROR-FLAG    PIC 9     VALUE 0.
       01  WS-BUFFER        PIC X(200) VALUE SPACES.

       LINKAGE SECTION.
       01  LS-INPUT         PIC X(100).

       PROCEDURE DIVISION USING LS-INPUT.
      *>   WS-LINE-COUNT, WS-ERROR-FLAG, and WS-BUFFER
      *>   are guaranteed to be at their VALUE clause
      *>   values at this point
           PERFORM PROCESS-RECORD
           GOBACK.

       PROCESS-RECORD.
           ADD 1 TO WS-LINE-COUNT
           DISPLAY "Processing line " WS-LINE-COUNT.

       END PROGRAM FORMAT-RECORD.
```

### INITIAL vs LOCAL-STORAGE

`LOCAL-STORAGE SECTION` data is also reinitialized on each invocation, even without `INITIAL`. The difference:

| Aspect | INITIAL program | LOCAL-STORAGE |
|--------|----------------|---------------|
| Scope | Entire program state | Only LOCAL-STORAGE items |
| WORKING-STORAGE | Reinitialized each call | Retains values between calls |
| Files | Closed on each entry | Not affected |
| PERFORM ranges | Reset | Reset |
| Available since | COBOL-85 | COBOL 2002 |

For COBOL 2002 and later, `LOCAL-STORAGE` is often preferred over `INITIAL` because it gives finer control -- you can have some data persist (in WORKING-STORAGE) while other data reinitializes (in LOCAL-STORAGE).

---

## COMMON

The `COMMON` clause is meaningful only for programs that are nested (contained) within another program. A common program is visible to all programs contained within the same outermost containing program, not just its directly containing program.

Without `COMMON`, a nested program can only be called by the program that directly contains it.

### Visibility Rules

```
OUTER-PROG
├── PROG-A
│   └── HELPER (COMMON)
├── PROG-B
└── PROG-C
```

- Without `COMMON` on HELPER: only PROG-A can call HELPER.
- With `COMMON` on HELPER: PROG-A, PROG-B, and PROG-C can all call HELPER.

### Example

```cobol
       IDENTIFICATION DIVISION.
       PROGRAM-ID. OUTER-PROG.

       PROCEDURE DIVISION.
           CALL "CALC-A"
           CALL "CALC-B"
           STOP RUN.

      *> --- CALC-A can call LOG-MSG ---
       IDENTIFICATION DIVISION.
       PROGRAM-ID. CALC-A.
       PROCEDURE DIVISION.
           DISPLAY "In CALC-A"
           CALL "LOG-MSG"
           GOBACK.
       END PROGRAM CALC-A.

      *> --- CALC-B can also call LOG-MSG ---
      *> --- because LOG-MSG is COMMON ---
       IDENTIFICATION DIVISION.
       PROGRAM-ID. CALC-B.
       PROCEDURE DIVISION.
           DISPLAY "In CALC-B"
           CALL "LOG-MSG"
           GOBACK.
       END PROGRAM CALC-B.

      *> --- Shared utility, visible to siblings ---
       IDENTIFICATION DIVISION.
       PROGRAM-ID. LOG-MSG IS COMMON PROGRAM.
       PROCEDURE DIVISION.
           DISPLAY "LOG: message logged"
           GOBACK.
       END PROGRAM LOG-MSG.

       END PROGRAM OUTER-PROG.
```

### When to Use COMMON

- Utility routines shared by multiple sibling nested programs (logging, validation, formatting).
- Reducing code duplication within a nested program structure without moving the utility to a separate compilation unit.

### COMMON and INITIAL Together

A program may be both COMMON and INITIAL:

```cobol
PROGRAM-ID. SHARED-INIT IS COMMON INITIAL PROGRAM.
```

This creates a shared utility that also reinitializes its state on every call -- useful for stateless helper routines.

---

## RECURSIVE

The `RECURSIVE` clause permits a program to call itself, either directly or through a chain of intermediate calls. Without this clause, a recursive invocation causes undefined behavior on most compilers.

- **Standard:** COBOL 2002

### How Recursion Works in COBOL

When `RECURSIVE` is specified:

- Each invocation receives its own copy of `LOCAL-STORAGE SECTION` data.
- `WORKING-STORAGE SECTION` data is **shared** across all invocations (it exists once for the program).
- The return point is saved so that control returns correctly when each invocation completes.

This means that local variables for recursion should be declared in `LOCAL-STORAGE`, not `WORKING-STORAGE`.

### Factorial Example

```cobol
       IDENTIFICATION DIVISION.
       PROGRAM-ID. FACTORIAL IS RECURSIVE PROGRAM.

       DATA DIVISION.
       WORKING-STORAGE SECTION.
      *>   Shared across all invocations
       01  WS-INPUT       PIC 9(4).
       01  WS-RESULT      PIC 9(18).

       LOCAL-STORAGE SECTION.
      *>   Separate copy per invocation
       01  LS-N           PIC 9(4).
       01  LS-PREV        PIC 9(18).

       PROCEDURE DIVISION.
           MOVE WS-INPUT TO LS-N
           IF LS-N <= 1
               MOVE 1 TO WS-RESULT
           ELSE
               SUBTRACT 1 FROM LS-N GIVING WS-INPUT
               CALL "FACTORIAL"
               MOVE WS-RESULT TO LS-PREV
               COMPUTE WS-RESULT = LS-N * LS-PREV
           END-IF
           GOBACK.
       END PROGRAM FACTORIAL.
```

**Calling program:**

```cobol
       IDENTIFICATION DIVISION.
       PROGRAM-ID. MAIN-PROG.

       DATA DIVISION.
       WORKING-STORAGE SECTION.
       01  WS-NUM          PIC 9(4).
       01  WS-ANSWER       PIC 9(18).

       PROCEDURE DIVISION.
           MOVE 10 TO WS-NUM
           CALL "FACTORIAL" USING WS-NUM WS-ANSWER
           DISPLAY "10! = " WS-ANSWER
           STOP RUN.
       END PROGRAM MAIN-PROG.
```

### Binary Search Example

```cobol
       IDENTIFICATION DIVISION.
       PROGRAM-ID. BIN-SEARCH IS RECURSIVE PROGRAM.

       DATA DIVISION.
       WORKING-STORAGE SECTION.
       01  WS-TABLE.
           05  WS-ELEM    PIC 9(5) OCCURS 100 TIMES.
       01  WS-TABLE-SIZE   PIC 9(3).
       01  WS-TARGET       PIC 9(5).
       01  WS-FOUND-IDX    PIC 9(3) VALUE 0.

       LOCAL-STORAGE SECTION.
       01  LS-LOW          PIC 9(3).
       01  LS-HIGH         PIC 9(3).
       01  LS-MID          PIC 9(3).

       LINKAGE SECTION.
       01  LS-LO           PIC 9(3).
       01  LS-HI           PIC 9(3).

       PROCEDURE DIVISION USING LS-LO LS-HI.
           MOVE LS-LO TO LS-LOW
           MOVE LS-HI TO LS-HIGH

           IF LS-LOW > LS-HIGH
               MOVE 0 TO WS-FOUND-IDX
               GOBACK
           END-IF

           COMPUTE LS-MID = (LS-LOW + LS-HIGH) / 2

           EVALUATE TRUE
               WHEN WS-ELEM(LS-MID) = WS-TARGET
                   MOVE LS-MID TO WS-FOUND-IDX
               WHEN WS-ELEM(LS-MID) > WS-TARGET
                   SUBTRACT 1 FROM LS-MID GIVING LS-HIGH
                   CALL "BIN-SEARCH" USING LS-LOW LS-HIGH
               WHEN WS-ELEM(LS-MID) < WS-TARGET
                   ADD 1 TO LS-MID GIVING LS-LOW
                   CALL "BIN-SEARCH" USING LS-LOW LS-HIGH
           END-EVALUATE

           GOBACK.
       END PROGRAM BIN-SEARCH.
```

### Recursion Guidelines

1. **Always use `LOCAL-STORAGE`** for variables that must be unique per invocation (loop counters, intermediate results, parameters saved across recursive calls).

2. **Use `WORKING-STORAGE`** for data shared across all invocations (the input/output interface, lookup tables, accumulators for the final result).

3. **Ensure a base case** to prevent infinite recursion. COBOL has no built-in stack overflow detection on most platforms.

4. **Be mindful of stack depth.** Each recursive call allocates a new LOCAL-STORAGE frame. Deep recursion can exhaust system resources.

5. **Consider iteration** as an alternative. Most COBOL programs favor iterative solutions with `PERFORM VARYING` over recursion, as they are more efficient and idiomatic.

---

## Comparison

| Attribute | Effect | Scope | Standard |
|-----------|--------|-------|----------|
| INITIAL | Reinitializes program state on each call | Any called program | COBOL-85 |
| COMMON | Makes nested program visible to sibling programs | Nested programs only | COBOL-85 |
| RECURSIVE | Permits self-invocation with per-call LOCAL-STORAGE | Any program | COBOL 2002 |

---

## See Also

- [Identification Division](index.md) -- division overview
- [CALL](../procedure-division/program-linkage/call.md) -- invoking subprograms
- [CANCEL](../procedure-division/program-linkage/cancel.md) -- releasing subprogram resources
- [User-Defined Functions](../intrinsic-functions/user-defined.md) -- FUNCTION-ID definitions
