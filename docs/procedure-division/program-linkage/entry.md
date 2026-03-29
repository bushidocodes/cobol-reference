# ENTRY

The `ENTRY` statement defines an alternate entry point in a called subprogram, allowing a single program to be called by different names with different parameter lists.

- **Standard:** COBOL-85, COBOL 2002, COBOL 2014, COBOL 2023
- **Division:** Procedure Division
- **Category:** Program Linkage

---

## Syntax

```cobol
ENTRY literal-1 [ USING { BY REFERENCE | BY CONTENT | BY VALUE }
    identifier-1 [ identifier-2 ] ... ]
    [ RETURNING identifier-3 ]
```

---

## Rules

1. `literal-1` is an alphanumeric literal specifying the entry point name. This is the name used in CALL statements from other programs.
2. When a program is called via an ENTRY point, execution begins at the **first statement following the ENTRY statement**, not at the beginning of the Procedure Division.
3. The USING clause specifies the parameters for this entry point, which may differ from the program's main PROCEDURE DIVISION USING parameters.
4. Parameters follow the same BY REFERENCE / BY CONTENT / BY VALUE rules as CALL.
5. RETURNING specifies a return value, as with PROCEDURE DIVISION RETURNING.
6. A program may have multiple ENTRY statements, each providing a different entry point.
7. ENTRY points share the program's WORKING-STORAGE and other data areas.
8. The program returns to the caller via GOBACK or EXIT PROGRAM, regardless of which entry point was used.

---

## Examples

### Multiple Entry Points

```cobol
       IDENTIFICATION DIVISION.
       PROGRAM-ID. MATH-UTILS.

       DATA DIVISION.
       WORKING-STORAGE SECTION.
       01  WS-TEMP    PIC S9(9)V99.

       LINKAGE SECTION.
       01  LS-A       PIC S9(9)V99.
       01  LS-B       PIC S9(9)V99.
       01  LS-RESULT  PIC S9(9)V99.

       PROCEDURE DIVISION.
      *>   Default entry - not normally called
           DISPLAY "Use a specific entry point"
           GOBACK.

       ENTRY "ADD-VALUES" USING LS-A LS-B LS-RESULT.
           ADD LS-A TO LS-B GIVING LS-RESULT
           GOBACK.

       ENTRY "MULTIPLY-VALUES" USING LS-A LS-B LS-RESULT.
           MULTIPLY LS-A BY LS-B GIVING LS-RESULT
           GOBACK.

       END PROGRAM MATH-UTILS.
```

### Calling Different Entry Points

```cobol
       CALL "ADD-VALUES" USING WS-X WS-Y WS-SUM
       CALL "MULTIPLY-VALUES" USING WS-X WS-Y WS-PRODUCT
```

---

## Use Cases

- **Legacy library consolidation** — combining related utility routines into a single program
- **Shared initialization** — multiple entry points share common WORKING-STORAGE that is initialized once
- **State machines** — different entry points for different processing phases

!!! tip "Modern Alternative"
    For new programs, consider using separate programs or user-defined
    functions instead of ENTRY. Multiple entry points can make programs
    harder to understand and maintain.

---

## See Also

- [CALL](call.md) -- invoking subprograms
- [CANCEL](cancel.md) -- releasing subprogram resources
- [GOBACK](../control-flow/goback.md) -- returning from a subprogram
