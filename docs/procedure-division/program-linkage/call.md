# CALL

The `CALL` statement transfers control from one program to another within a run unit, optionally passing data through parameters and receiving a return value.

- **Standard:** COBOL-68, COBOL-74, COBOL-85, COBOL 2002, COBOL 2014, COBOL 2023
- **Division:** Procedure Division
- **Category:** Program Linkage

---

## Syntax

```cobol
CALL {identifier-1 | literal-1}
    [USING
        [{BY REFERENCE} {identifier-2 | OMITTED}] ...
        [{BY CONTENT}   {identifier-3 | literal-2 | OMITTED}] ...
        [{BY VALUE}     {identifier-4 | literal-3}] ...
    ]
    [RETURNING identifier-5]
    [ON OVERFLOW imperative-statement-1]
    [ON EXCEPTION imperative-statement-2]
    [NOT ON EXCEPTION imperative-statement-3]
[END-CALL]
```

---

## Rules

### Program Name

The called program is identified by `literal-1` or `identifier-1`.

- **Literal:** The program name is known at compile time. The compiler or linker resolves the reference.
- **Identifier:** The program name is determined at runtime from the value of the identifier. This enables dynamic dispatch.

```cobol
CALL "CUSTVAL"  USING WS-CUSTOMER-REC
CALL WS-PROGRAM-NAME USING WS-DATA
```

### USING Phrase

The `USING` phrase specifies the parameters passed to the called program. The called program receives these parameters through its `PROCEDURE DIVISION USING` clause and defines them in its Linkage Section.

Parameters are matched positionally. The first parameter in the `CALL USING` corresponds to the first parameter in the called program's `PROCEDURE DIVISION USING`, and so forth.

### BY REFERENCE

The address of the data item is passed. The called program operates directly on the calling program's data. Any changes made by the called program are visible to the caller upon return.

`BY REFERENCE` is the default passing convention if no `BY` phrase is specified.

```cobol
CALL "SUBPROG" USING BY REFERENCE WS-RECORD
CALL "SUBPROG" USING WS-RECORD              *> Same as BY REFERENCE
```

### BY CONTENT

A copy of the data item's value is passed. The called program receives a temporary copy and may modify it, but changes do not affect the calling program's original data.

```cobol
CALL "SUBPROG" USING BY CONTENT WS-AMOUNT
CALL "SUBPROG" USING BY CONTENT "FIXED-VALUE"
```

!!! note "COBOL-85"
    `BY CONTENT` was introduced in COBOL-85. In COBOL-68 and COBOL-74, all parameters are passed by reference.

### BY VALUE

The value of the data item is passed as a binary value on the call stack, following the platform's calling convention for value parameters. This is primarily used for interfacing with programs written in C or other languages.

!!! note "COBOL 2002"
    `BY VALUE` was introduced in COBOL 2002.

```cobol
CALL "C-FUNCTION" USING BY VALUE WS-INT-PARAM
```

### OMITTED

The keyword `OMITTED` may appear in place of a parameter to indicate that no value is passed for that position. The called program must not reference the omitted parameter.

```cobol
CALL "SUBPROG" USING WS-PARAM-1
                     OMITTED
                     WS-PARAM-3
```

### RETURNING Phrase

The `RETURNING` phrase specifies a data item to receive the return value from the called program. The called program provides this value through the `RETURNING` phrase in its `PROCEDURE DIVISION` header.

!!! note "COBOL 2002"
    The `RETURNING` phrase on `CALL` was introduced in COBOL 2002.

```cobol
CALL "COMPUTE-TAX" USING WS-INCOME
    RETURNING WS-TAX-AMOUNT
```

### ON OVERFLOW / ON EXCEPTION

The `ON OVERFLOW` and `ON EXCEPTION` phrases specify statements to execute if the called program cannot be found or loaded.

- `ON OVERFLOW` is the pre-COBOL-85 syntax.
- `ON EXCEPTION` and `NOT ON EXCEPTION` were introduced in COBOL-85.

If the call fails and no `ON EXCEPTION` or `ON OVERFLOW` phrase is specified, behavior is implementation-defined (many compilers terminate the run unit with an error).

```cobol
CALL "OPTMODULE" USING WS-DATA
    ON EXCEPTION
        DISPLAY "Module OPTMODULE not available"
        MOVE "N" TO WS-MODULE-LOADED
    NOT ON EXCEPTION
        MOVE "Y" TO WS-MODULE-LOADED
END-CALL
```

### END-CALL

!!! note "COBOL-85"
    The `END-CALL` scope terminator was introduced in COBOL-85.

`END-CALL` marks the explicit end of the `CALL` statement. It is required when `ON EXCEPTION` or `NOT ON EXCEPTION` is used in a context where an implicit scope terminator (period) is not desired.

---

## Behavior

### Static vs Dynamic CALL

- **Static CALL:** The called program is linked into the same executable at link time. This is typically the result of `CALL literal` when the compiler and linker resolve the reference statically. Static calls have lower overhead but require relinking to change the called program.
- **Dynamic CALL:** The called program is loaded at runtime from a separate module or shared library. `CALL identifier` always produces a dynamic call. `CALL literal` may be static or dynamic depending on compiler options.

The distinction between static and dynamic calls is implementation-defined and controlled by compiler directives or options.

### Initial State of a Called Program

When a program is called for the first time, or for the first time after being cancelled by a `CANCEL` statement:

- Working-Storage items with `VALUE` clauses are initialized to their defined values.
- Working-Storage items without `VALUE` clauses are initialized to an undefined state (implementation-defined, often spaces for alphanumeric and zeros for numeric items).
- Files are in a closed state.

### Subsequent Calls

On subsequent calls (without an intervening `CANCEL`), the called program is in its **last-used state**:

- Working-Storage items retain the values they had when the program last returned control.
- Files remain in whatever state (open or closed) they were left in.

This behavior changes if the program has the `INITIAL` attribute, which causes the program to be reinitialized on every call.

### CANCEL Interaction

The `CANCEL` statement releases the resources of a called program and returns it to its initial state. The next `CALL` to a cancelled program behaves as a first-time call.

```cobol
CALL "SUBPROG" USING WS-DATA
*> ... later ...
CANCEL "SUBPROG"
CALL "SUBPROG" USING WS-DATA   *> Program is re-initialized
```

### Nested Program Calls

A program may contain other programs (nested programs). A nested program can be called directly by its containing program. A nested program with the `COMMON` attribute may also be called by other programs nested within the same containing program.

```cobol
IDENTIFICATION DIVISION.
PROGRAM-ID. OUTER-PROG.
PROCEDURE DIVISION.
    CALL "INNER-PROG" USING WS-DATA
    STOP RUN.

IDENTIFICATION DIVISION.
PROGRAM-ID. INNER-PROG.
PROCEDURE DIVISION USING LS-DATA.
    PERFORM PROCESS-DATA
    GOBACK.
END PROGRAM INNER-PROG.
END PROGRAM OUTER-PROG.
```

---

## Examples

### Basic CALL with BY REFERENCE

```cobol
01 WS-INPUT-REC   PIC X(100).
01 WS-OUTPUT-REC  PIC X(100).

CALL "TRANSFORM" USING BY REFERENCE WS-INPUT-REC
                                     WS-OUTPUT-REC
END-CALL
```

### CALL with BY CONTENT and BY REFERENCE

```cobol
01 WS-FUNCTION    PIC X(8).
01 WS-REQUEST     PIC X(200).
01 WS-RESPONSE    PIC X(500).

MOVE "VALIDATE" TO WS-FUNCTION
CALL "SERVICE-HANDLER" USING BY CONTENT WS-FUNCTION
                              BY REFERENCE WS-REQUEST
                              BY REFERENCE WS-RESPONSE
END-CALL
```

### Dynamic CALL

```cobol
01 WS-MODULE-NAME PIC X(8).

MOVE "MODPAY" TO WS-MODULE-NAME
CALL WS-MODULE-NAME USING WS-PAY-RECORD
    ON EXCEPTION
        DISPLAY "Module " WS-MODULE-NAME " not found"
    NOT ON EXCEPTION
        DISPLAY "Module " WS-MODULE-NAME " executed"
END-CALL
```

### CALL with RETURNING

```cobol
01 WS-EMPLOYEE-ID  PIC 9(6).
01 WS-SALARY       PIC S9(9)V99.

CALL "GET-SALARY" USING BY CONTENT WS-EMPLOYEE-ID
    RETURNING WS-SALARY
END-CALL

DISPLAY "Salary: " WS-SALARY
```

### Calling a C Function with BY VALUE

```cobol
01 WS-EXIT-CODE   PIC S9(9) COMP-5.

MOVE 0 TO WS-EXIT-CODE
CALL "exit" USING BY VALUE WS-EXIT-CODE
END-CALL
```

### Error Handling with ON EXCEPTION

```cobol
CALL "OPTIONAL-MODULE" USING WS-DATA
    ON EXCEPTION
        PERFORM FALLBACK-PROCESSING
    NOT ON EXCEPTION
        PERFORM POST-MODULE-PROCESSING
END-CALL
```

---

## See Also

- [CANCEL](cancel.md) — releases a called program's resources
- [EXIT](../control-flow/exit.md) — exits a called program (EXIT PROGRAM)
- [STOP](../control-flow/stop.md) — terminates the run unit
- GOBACK — returns to calling program or terminates run unit
- [Procedure Division Overview](../index.md)
