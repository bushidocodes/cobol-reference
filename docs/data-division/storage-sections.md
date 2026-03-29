# Storage Sections (WORKING-STORAGE, LOCAL-STORAGE, LINKAGE)

The Data Division contains three sections that define program variables: **WORKING-STORAGE**, **LOCAL-STORAGE**, and **LINKAGE**. Each section has distinct rules for storage allocation, initialization, lifetime, and visibility. Understanding these differences is essential for writing correct COBOL programs, especially subprograms and recursive programs.

- **Standard:** WORKING-STORAGE and LINKAGE: COBOL-68. LOCAL-STORAGE: COBOL 2002.

!!! info "Historical Note"
    **WORKING-STORAGE** has existed since COBOL-60, where it was one of the original
    three Data Division sections (alongside FILE and CONSTANT). In COBOL-61,
    working-storage items were either "independent" (level-77) or "grouped" (level-01
    with subordinates); independent items had to precede grouped items. Initial values
    were set via the VALUE clause, same as today.

    The **CONSTANT Section** was a separate third section in COBOL-60 and COBOL-61,
    organized identically to WORKING-STORAGE, but items defined there could not be
    modified at runtime. It was removed in later standards, with its role absorbed by
    VALUE clauses on WORKING-STORAGE items.

    The **LINKAGE Section** did not exist in COBOL-60 or COBOL-61. It was added in
    COBOL-65/68 to support parameter passing between programs via the CALL statement.

---

## WORKING-STORAGE SECTION

The Working-Storage Section defines data items that are allocated statically and persist for the lifetime of the run unit.

```cobol
WORKING-STORAGE SECTION.
01  WS-COUNTER       PIC 9(5)    VALUE 0.
01  WS-NAME          PIC X(30)   VALUE SPACES.
77  WS-STATUS        PIC X(2)    VALUE "OK".
```

### Allocation and Lifetime

- Storage is allocated **once**, when the program is first loaded into memory.
- Storage **persists** across multiple `CALL` invocations of the program. Data items retain their last-used values between calls.
- Storage is **not released** until the program is canceled (`CANCEL` statement) or the run unit terminates (`STOP RUN`).
- For the **main program**, WORKING-STORAGE exists for the entire execution.
- For a **subprogram**, WORKING-STORAGE is allocated on the first `CALL` and retained across subsequent calls.

### Initialization

- `VALUE` clauses are applied **once**, at initial load time.
- On subsequent `CALL` invocations, data items retain their prior values -- `VALUE` clauses are **not** reapplied.
- If no `VALUE` clause is specified, the initial content is undefined on most compilers (though some compilers zero-fill or space-fill by default as an extension).

### State Persistence Example

```cobol
      *> Subprogram that counts how many times it is called
       IDENTIFICATION DIVISION.
       PROGRAM-ID. CALL-COUNTER.

       DATA DIVISION.
       WORKING-STORAGE SECTION.
       01  WS-COUNT      PIC 9(5) VALUE 0.

       LINKAGE SECTION.
       01  LS-RESULT     PIC 9(5).

       PROCEDURE DIVISION USING LS-RESULT.
           ADD 1 TO WS-COUNT
           MOVE WS-COUNT TO LS-RESULT
           GOBACK.
       END PROGRAM CALL-COUNTER.
```

Each call increments and returns an ever-increasing count because `WS-COUNT` retains its value:

```
Call 1: LS-RESULT = 00001
Call 2: LS-RESULT = 00002
Call 3: LS-RESULT = 00003
```

### Resetting WORKING-STORAGE

There are three ways to reset WORKING-STORAGE to its initial state:

1. **CANCEL the subprogram** -- the next `CALL` reloads and reinitializes:

    ```cobol
    CALL "CALL-COUNTER" USING WS-RESULT
    CANCEL "CALL-COUNTER"
    CALL "CALL-COUNTER" USING WS-RESULT
    *> WS-RESULT = 00001 (reset)
    ```

2. **Use the INITIAL clause** on `PROGRAM-ID` -- the program reinitializes on every call:

    ```cobol
    PROGRAM-ID. CALL-COUNTER IS INITIAL PROGRAM.
    ```

3. **Use INITIALIZE** explicitly in the PROCEDURE DIVISION.

### WORKING-STORAGE in Recursive Programs

When a program is declared `RECURSIVE`, WORKING-STORAGE is **shared** across all active invocations. There is only one copy, and all recursive calls read and write the same data. This makes WORKING-STORAGE unsuitable for per-invocation variables in recursive programs -- use LOCAL-STORAGE instead.

---

## LOCAL-STORAGE SECTION

The Local-Storage Section defines data items that are allocated and initialized fresh on **every invocation** of the program.

- **Standard:** COBOL 2002

```cobol
LOCAL-STORAGE SECTION.
01  LS-TEMP          PIC X(100)  VALUE SPACES.
01  LS-INDEX         PIC 9(5)    VALUE 0.
```

### Allocation and Lifetime

- Storage is allocated **each time** the program is invoked (via `CALL` or recursive entry).
- Storage is **deallocated** when the program returns control to its caller (`GOBACK`, `EXIT PROGRAM`, or `EXIT FUNCTION`).
- Each active invocation has its own **independent copy** of LOCAL-STORAGE.
- Data items **do not persist** between calls.

### Initialization

- `VALUE` clauses are applied on **every invocation**, not just the first.
- If no `VALUE` clause is specified, the initial content is undefined.
- This behavior is fundamentally different from WORKING-STORAGE, where `VALUE` clauses are applied only once.

### Per-Invocation State Example

```cobol
      *> Subprogram demonstrating LOCAL-STORAGE behavior
       IDENTIFICATION DIVISION.
       PROGRAM-ID. FORMAT-LINE.

       DATA DIVISION.
       LOCAL-STORAGE SECTION.
       01  LS-BUFFER     PIC X(80) VALUE SPACES.
       01  LS-LENGTH     PIC 9(3)  VALUE 0.

       LINKAGE SECTION.
       01  LS-INPUT      PIC X(50).
       01  LS-OUTPUT     PIC X(80).

       PROCEDURE DIVISION USING LS-INPUT LS-OUTPUT.
      *>   LS-BUFFER is guaranteed to be SPACES here
      *>   LS-LENGTH is guaranteed to be 0 here
      *>   regardless of what previous calls did
           STRING LS-INPUT DELIMITED BY SPACES
               INTO LS-BUFFER
               WITH POINTER LS-LENGTH
           MOVE LS-BUFFER TO LS-OUTPUT
           GOBACK.
       END PROGRAM FORMAT-LINE.
```

### LOCAL-STORAGE in Recursive Programs

LOCAL-STORAGE is essential for recursion. Each recursive invocation gets its own copy, making it safe for per-invocation variables like loop counters, intermediate results, and saved parameters.

```cobol
       IDENTIFICATION DIVISION.
       PROGRAM-ID. FIBONACCI IS RECURSIVE PROGRAM.

       DATA DIVISION.
       WORKING-STORAGE SECTION.
      *>   Shared across all invocations (input/output interface)
       01  WS-N           PIC 9(4).
       01  WS-RESULT      PIC 9(18).

       LOCAL-STORAGE SECTION.
      *>   Separate copy per invocation
       01  LS-MY-N        PIC 9(4).
       01  LS-FIB-1       PIC 9(18).
       01  LS-FIB-2       PIC 9(18).

       PROCEDURE DIVISION.
           MOVE WS-N TO LS-MY-N
           EVALUATE TRUE
               WHEN LS-MY-N = 0
                   MOVE 0 TO WS-RESULT
               WHEN LS-MY-N = 1
                   MOVE 1 TO WS-RESULT
               WHEN OTHER
                   SUBTRACT 1 FROM LS-MY-N GIVING WS-N
                   CALL "FIBONACCI"
                   MOVE WS-RESULT TO LS-FIB-1

                   SUBTRACT 2 FROM LS-MY-N GIVING WS-N
                   CALL "FIBONACCI"
                   MOVE WS-RESULT TO LS-FIB-2

                   ADD LS-FIB-1 LS-FIB-2 GIVING WS-RESULT
           END-EVALUATE
           GOBACK.
       END PROGRAM FIBONACCI.
```

Without LOCAL-STORAGE, the intermediate values `LS-FIB-1` and `LS-FIB-2` would be overwritten by deeper recursive calls.

---

## LINKAGE SECTION

The Linkage Section describes data items whose storage is **not allocated by the program**. Instead, the storage is owned by a caller (or by the runtime via `ALLOCATE`) and the program receives an address to it.

```cobol
LINKAGE SECTION.
01  LS-CUSTOMER-REC.
    05  LS-CUST-ID     PIC X(10).
    05  LS-CUST-NAME   PIC X(30).
    05  LS-CUST-BAL    PIC S9(9)V99.
```

### Allocation and Lifetime

- The program **does not allocate** storage for LINKAGE items. Declaring them only describes the expected layout.
- Storage is **provided by the caller** through `CALL ... USING` or by the runtime through `ALLOCATE` / `SET ADDRESS OF`.
- The lifetime of the data is controlled by the **caller**, not the called program.
- LINKAGE items are **accessible only while the program is active** (between entry and `GOBACK`/`EXIT PROGRAM`).
- After the called program returns, the caller's data items still exist in the caller's storage.

### Initialization

- `VALUE` clauses are **not permitted** on LINKAGE items (except on level-88 condition names).
- The data content is whatever the caller provides. The called program must not assume any initial state beyond what the caller guarantees.

### Parameter Passing

LINKAGE items are associated with caller arguments through the `PROCEDURE DIVISION USING` phrase:

```cobol
      *> Calling program
       CALL "CALC-TAX" USING WS-AMOUNT WS-RATE WS-TAX

      *> Called program
       LINKAGE SECTION.
       01  LS-AMOUNT    PIC 9(7)V99.
       01  LS-RATE      PIC 9V9(4).
       01  LS-TAX       PIC 9(7)V99.

       PROCEDURE DIVISION USING LS-AMOUNT LS-RATE LS-TAX.
           COMPUTE LS-TAX ROUNDED =
               LS-AMOUNT * LS-RATE
           GOBACK.
```

### Passing Modes

The `USING` phrase supports three passing modes:

| Mode | Syntax | Behavior |
|------|--------|----------|
| `BY REFERENCE` | `USING LS-ITEM` (default) | Caller's storage is shared; changes are visible to the caller |
| `BY CONTENT` | `USING BY CONTENT LS-ITEM` | A copy of the caller's data is passed; changes are **not** visible to the caller |
| `BY VALUE` | `USING BY VALUE LS-ITEM` | The value is passed (not the address); used for interoperability with C and other languages |

```cobol
       PROCEDURE DIVISION
           USING BY REFERENCE LS-RESULT
                 BY CONTENT LS-INPUT
                 BY VALUE LS-FLAG.
```

### Accessing LINKAGE Without CALL

LINKAGE items can also be associated with storage via the `SET ADDRESS OF` statement, without a `CALL`:

```cobol
       WORKING-STORAGE SECTION.
       01  WS-PTR        USAGE POINTER.

       LINKAGE SECTION.
       01  LS-RECORD BASED.
           05  LS-FIELD-A PIC X(10).
           05  LS-FIELD-B PIC 9(5).

       PROCEDURE DIVISION.
           ALLOCATE LS-RECORD RETURNING WS-PTR
           SET ADDRESS OF LS-RECORD TO WS-PTR
           MOVE "Hello" TO LS-FIELD-A
           FREE LS-RECORD
           STOP RUN.
```

See [ALLOCATE and FREE](../procedure-division/memory/allocate-free.md) for details.

---

## Lifetime and Initialization Comparison

| Aspect | WORKING-STORAGE | LOCAL-STORAGE | LINKAGE |
|--------|----------------|---------------|---------|
| **Allocated by** | Runtime (once) | Runtime (each invocation) | Caller or ALLOCATE |
| **Allocation time** | Program load | Each CALL / entry | Caller's discretion |
| **Deallocation time** | CANCEL or STOP RUN | GOBACK / EXIT PROGRAM | Caller's discretion |
| **Persists across calls** | Yes | No | Depends on caller |
| **VALUE clause applied** | Once (at load) | Every invocation | Not permitted |
| **Shared in recursion** | Yes (one copy) | No (copy per invocation) | Depends on caller |
| **Default initial value** | Undefined (without VALUE) | Undefined (without VALUE) | Caller's data |
| **Standard** | COBOL-68 | COBOL 2002 | COBOL-68 |

### Timeline Diagram

```
Main program CALLs subprogram three times:

CALL 1          CALL 2          CALL 3
┌──────────┐   ┌──────────┐   ┌──────────┐
│ WORKING- │   │ WORKING- │   │ WORKING- │
│ STORAGE  │ = │ STORAGE  │ = │ STORAGE  │  Same storage,
│ (values  │   │ (values  │   │ (values  │  values persist
│  from    │   │  from    │   │  from    │
│  CALL 1) │   │  CALL 2) │   │  CALL 3) │
├──────────┤   ├──────────┤   ├──────────┤
│ LOCAL-   │   │ LOCAL-   │   │ LOCAL-   │
│ STORAGE  │   │ STORAGE  │   │ STORAGE  │  Fresh copy,
│ (VALUE   │   │ (VALUE   │   │ (VALUE   │  VALUE clauses
│  clauses │   │  clauses │   │  clauses │  reapplied
│  applied)│   │  applied)│   │  applied)│
├──────────┤   ├──────────┤   ├──────────┤
│ LINKAGE  │   │ LINKAGE  │   │ LINKAGE  │
│ (points  │   │ (points  │   │ (points  │  Caller's data,
│  to      │   │  to      │   │  to      │  no allocation
│  caller) │   │  caller) │   │  caller) │
└──────────┘   └──────────┘   └──────────┘
   freed          freed          freed     ← LOCAL-STORAGE
```

---

## GLOBAL Clause

The `GLOBAL` clause on a WORKING-STORAGE or FILE SECTION item makes that item visible to all programs contained (nested) within the declaring program.

```cobol
       IDENTIFICATION DIVISION.
       PROGRAM-ID. OUTER-PROG.

       DATA DIVISION.
       WORKING-STORAGE SECTION.
       01  WS-SHARED-DATA  PIC X(50) IS GLOBAL.

       PROCEDURE DIVISION.
           MOVE "Visible to inner" TO WS-SHARED-DATA
           CALL "INNER-PROG"
           STOP RUN.

       IDENTIFICATION DIVISION.
       PROGRAM-ID. INNER-PROG.
       PROCEDURE DIVISION.
      *>   WS-SHARED-DATA is accessible here because
      *>   it was declared GLOBAL in the containing program
           DISPLAY WS-SHARED-DATA
           GOBACK.
       END PROGRAM INNER-PROG.

       END PROGRAM OUTER-PROG.
```

### GLOBAL Rules

- Only items in WORKING-STORAGE and the FILE SECTION can be declared GLOBAL.
- LOCAL-STORAGE and LINKAGE items **cannot** be GLOBAL.
- GLOBAL items are visible to all directly and indirectly contained programs.
- A GLOBAL item in a nested program is visible only to programs nested within it, not to sibling or parent programs.

---

## EXTERNAL Clause

The `EXTERNAL` clause declares that a data item's storage is shared across separately compiled programs within the same run unit. All programs that declare the same EXTERNAL item with the same name share a single copy of the storage.

```cobol
      *> Program A
       WORKING-STORAGE SECTION.
       01  SHARED-CONFIG  PIC X(100) IS EXTERNAL.

      *> Program B (separately compiled)
       WORKING-STORAGE SECTION.
       01  SHARED-CONFIG  PIC X(100) IS EXTERNAL.

      *> Both programs access the same storage
```

### EXTERNAL Rules

- EXTERNAL items must be level-01 in WORKING-STORAGE.
- The name, size, and structure must match across all programs sharing the item.
- EXTERNAL provides a form of global shared memory without parameter passing.
- `VALUE` clauses on EXTERNAL items are generally ignored (the runtime allocates and initializes the storage once).

---

## Choosing the Right Section

| Scenario | Section | Reason |
|----------|---------|--------|
| Program-wide variables and accumulators | WORKING-STORAGE | Persists across calls, allocated once |
| Constants and lookup tables | WORKING-STORAGE | Initialized once, never changes |
| Temporary work areas in a subprogram | LOCAL-STORAGE | Clean slate on each call |
| Variables in a recursive program | LOCAL-STORAGE | Separate copy per invocation |
| Receiving parameters from a caller | LINKAGE | Storage owned by caller |
| Dynamically allocated structures | LINKAGE (BASED) | Storage from ALLOCATE |
| Data shared across nested programs | WORKING-STORAGE (GLOBAL) | Visible to contained programs |
| Data shared across separate programs | WORKING-STORAGE (EXTERNAL) | Shared by name across run unit |

!!! tip "Rule of Thumb"
    Use WORKING-STORAGE by default. Switch to LOCAL-STORAGE when you need
    guaranteed clean state on each invocation (especially in recursive or
    multi-threaded contexts). Use LINKAGE for parameters and dynamically
    allocated memory.

---

## See Also

- [Level Numbers](level-numbers.md) -- data item level numbers
- [VALUE Clause](value.md) -- data initialization
- [REDEFINES Clause](redefines.md) -- storage overlay
- [CALL](../procedure-division/program-linkage/call.md) -- invoking subprograms
- [Program Attributes](../identification-division/program-attributes.md) -- INITIAL, COMMON, RECURSIVE
- [ALLOCATE and FREE](../procedure-division/memory/allocate-free.md) -- dynamic memory
- [User-Defined Types](typedef.md) -- TYPEDEF clause
