# ENTER (Removed)

The `ENTER` statement allowed a COBOL program to switch to another programming language (such as assembly language or ALGOL) within the Procedure Division.

- **Introduced:** COBOL-60
- **Obsolete:** COBOL-85
- **Removed:** COBOL 2002
- **Status:** Removed

!!! warning "Removed"
    `ENTER` was classified as obsolete in COBOL-85 and removed in COBOL 2002.
    It had unique status in COBOL-85: implementation was entirely optional
    for the compiler vendor — the only statement with this distinction.
    Use `CALL` for inter-language communication instead.

---

## Syntax

```cobol
ENTER language-name [ routine-name ]
```

---

## Rules

1. `language-name` identifies the other programming language to be used (e.g., `ENTER MACRO-ASSEMBLER`).
2. `routine-name`, if specified, identifies the entry point in the other language.
3. The statements following `ENTER` are written in the specified language until another `ENTER COBOL` statement returns control to COBOL.
4. The exact language-names accepted and the mechanism for embedding foreign code were entirely implementor-defined.

---

## Example

```cobol
       PROCEDURE DIVISION.
       MAIN-PROCESS.
           MOVE INPUT-VALUE TO WORK-AREA
           ENTER MACRO-ASSEMBLER FAST-CALC
           MOVE WORK-AREA TO OUTPUT-VALUE
           STOP RUN.
```

The intent was that `FAST-CALC` would be an assembly language routine processed inline. In practice, the mechanism was unreliable across compilers and was identified as a fundamental portability barrier as early as 1962 (Lippitt, "COBOL and Compatibility").

---

## Why It Was Removed

1. **No portable semantics** — every compiler implemented ENTER differently (or not at all). A program using ENTER could not be compiled on a different system.
2. **The CALL statement** provided a cleaner, standardized mechanism for inter-language communication without embedding foreign source code in the COBOL program.
3. **Security and reliability** — inline assembly or other low-level code bypassed COBOL's data description model and could corrupt the program's data areas.

---

## Modern Alternative

Use `CALL` with `BY VALUE` or `BY REFERENCE` to invoke routines written in other languages:

```cobol
       CALL "fast_calc" USING BY REFERENCE WORK-AREA
```

The called routine is compiled separately in its own language and linked with the COBOL program.

---

## See Also

- [CALL](../program-linkage/call.md) -- inter-program and inter-language communication
