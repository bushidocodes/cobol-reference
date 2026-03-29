# Procedure Division

The Procedure Division contains the executable statements of a COBOL program. It specifies the operations to be performed on the data described in the Data Division.

- **Standard:** COBOL-60, COBOL-61, COBOL-61 Extended, COBOL-65, COBOL-68, COBOL-85, COBOL 2002, COBOL 2014, COBOL 2023
- **Division:** Procedure Division

---

## Historical Note: COBOL-60

The Procedure Division was described in the 1960 COBOL report as "commonly called the program" and "only one of the three parts of the program," reflecting its central role. Operations were divided into **sections**, each subdivided into **paragraphs** — only these could be named and transferred to, a structural convention that persists today.

The original specification defined **23 verbs** across six categories:

| Category | Verbs |
|---|---|
| Arithmetic | ADD, SUBTRACT, MULTIPLY, DIVIDE, COMPUTE |
| Input-Output | READ, WRITE, OPEN, CLOSE, ACCEPT, DISPLAY |
| Procedure Branching | GO, ALTER, PERFORM |
| Data Movement | MOVE, EXAMINE |
| Ending | STOP |
| Compiler Directing | DEFINE, ENTER, EXIT, INCLUDE, NOTE, USE |

Six arithmetic operators were defined: `=` (EQUALS), `+` (PLUS), `-` (MINUS), `*` (MULTIPLIED BY), `/` (DIVIDED BY), and `**` (EXPONENTIATED BY).

Notably, `IF` was not classified as a verb but was acknowledged as having "the most important characteristics of one." Several original verbs evolved significantly or were replaced in later standards:

- **EXAMINE** was the original string inspection verb, replaced by `INSPECT` in COBOL-85.
- **INCLUDE** was the predecessor to the `COPY` statement.
- **NOTE** allowed inline comments within the Procedure Division, predating the `*>` comment syntax introduced in COBOL 2002.
- **DEFINE** allowed defining compile-time constants, a concept that would not return to the standard until much later.

---

## Syntax

```cobol
PROCEDURE DIVISION
    [USING {BY REFERENCE | BY VALUE} {identifier-1} ...]
    [RETURNING identifier-2].

[DECLARATIVES.
    {section-name SECTION.
        USE statement.
        [paragraph-name. sentence ...] ...} ...
END DECLARATIVES.]

{section-name SECTION [segment-number].
    [paragraph-name. sentence ...] ...} ...
```

### USING Phrase

The `USING` phrase specifies formal parameters for a program or method invoked by a `CALL` statement. Each parameter is passed either by reference or by value.

```cobol
PROCEDURE DIVISION USING BY REFERENCE WS-INPUT-REC
                        BY VALUE      WS-FLAG.
```

- **BY REFERENCE** (default): The called program receives the address of the argument. Changes to the parameter in the called program are visible to the caller.
- **BY VALUE**: The called program receives a copy of the argument's value. Changes to the parameter do not affect the caller's argument.

!!! note "COBOL 2002"
    The `BY VALUE` phrase was standardized in COBOL 2002. Many compilers supported it as an extension prior to that standard.

### RETURNING Phrase

The `RETURNING` phrase specifies a data item to receive the return value of a called program or method.

```cobol
PROCEDURE DIVISION USING BY REFERENCE WS-PARAM
                   RETURNING WS-RESULT.
```

!!! note "COBOL 2002"
    The `RETURNING` phrase was introduced in COBOL 2002.

---

## Declaratives

Declaratives are a set of one or more special-purpose sections at the beginning of the Procedure Division. Each declarative section begins with a `USE` statement that identifies its purpose. Declaratives are enclosed between `DECLARATIVES.` and `END DECLARATIVES.` headers.

Common uses of declaratives include:

- **USE AFTER EXCEPTION/ERROR** — specifies procedures to execute when an I/O error occurs on a file.
- **USE BEFORE REPORTING** — specifies procedures to execute before a report group is produced (Report Writer).
- **USE FOR DEBUGGING** — specifies procedures to execute when a debug event occurs (debugging mode only).

```cobol
DECLARATIVES.
INPUT-ERROR SECTION.
    USE AFTER STANDARD ERROR PROCEDURE ON INPUT-FILE.
INPUT-ERROR-PARA.
    DISPLAY "Error reading INPUT-FILE, status: " WS-FILE-STATUS.
END DECLARATIVES.
```

---

## Sections and Paragraphs

The Procedure Division is organized into **sections** and **paragraphs**.

- A **section** begins with a section header (`section-name SECTION.`) and ends where the next section header or the end of the program appears.
- A **paragraph** begins with a paragraph name followed by a period and ends where the next paragraph name, section header, or end of program appears.

Sections and paragraphs serve as targets for `PERFORM` and `GO TO` statements.

```cobol
MAIN-LOGIC SECTION.
INIT-PARA.
    PERFORM SETUP-ROUTINE.
    PERFORM PROCESS-RECORDS UNTIL WS-EOF = "Y".
    PERFORM CLEANUP-ROUTINE.
    STOP RUN.

SETUP-ROUTINE.
    OPEN INPUT IN-FILE OUTPUT OUT-FILE.

PROCESS-RECORDS.
    READ IN-FILE INTO WS-RECORD
        AT END MOVE "Y" TO WS-EOF
    END-READ.
```

---

## Statement Categories

### Arithmetic Statements

Statements that perform arithmetic operations on numeric data items.

| Statement | Description |
|-----------|-------------|
| ADD | Adds numeric operands |
| SUBTRACT | Subtracts numeric operands |
| MULTIPLY | Multiplies numeric operands |
| DIVIDE | Divides numeric operands |
| [COMPUTE](arithmetic/compute.md) | Evaluates an arithmetic expression |

### Data Movement Statements

Statements that move or transform data between data items.

| Statement | Description |
|-----------|-------------|
| [MOVE](data-movement/move.md) | Transfers data from one item to another |
| INITIALIZE | Sets data items to predetermined values |
| SET | Sets condition names, indexes, or switches |
| STRING | Concatenates partial or complete contents of data items |
| UNSTRING | Separates a data item into multiple receiving items |
| INSPECT | Tallies, replaces, or converts characters (replaced EXAMINE in COBOL-85) |

### Control Flow Statements

Statements that control the order of execution.

| Statement | Description |
|-----------|-------------|
| [IF](control-flow/if.md) | Conditional execution |
| [EVALUATE](control-flow/evaluate.md) | Multi-branch conditional (case/switch) |
| [PERFORM](control-flow/perform.md) | Executes a paragraph/section or inline statements, optionally with looping |
| GO TO | Transfers control to another paragraph or section |
| STOP | Halts or pauses execution |
| EXIT | Provides a common end point or exits a loop |
| CONTINUE | No operation; placeholder |
| GOBACK | Returns control to the calling program or operating system |

### Input/Output Statements

Statements that perform file and screen I/O operations.

| Statement | Description |
|-----------|-------------|
| OPEN | Opens a file for processing |
| CLOSE | Closes a file |
| [READ](io/read.md) | Retrieves a record from a file |
| WRITE | Writes a record to a file |
| REWRITE | Replaces an existing record in a file |
| DELETE | Removes a record from an indexed or relative file |
| START | Positions within an indexed or relative file |
| ACCEPT | Retrieves data from an external source |
| DISPLAY | Sends data to an external destination |

### Program Linkage Statements

Statements that manage inter-program communication.

| Statement | Description |
|-----------|-------------|
| CALL | Transfers control to another program |
| CANCEL | Releases resources associated with a called program |

### Table Handling Statements

Statements that operate on tables (arrays).

| Statement | Description |
|-----------|-------------|
| SEARCH | Performs a serial or binary search on a table |

### Exception Handling Statements

| Statement | Description |
|-----------|-------------|
| RAISE | Raises an exception |
| RESUME | Resumes execution after an exception |

!!! note "COBOL 2002"
    The `RAISE` and `RESUME` statements were introduced in COBOL 2002.

---

## Scope Terminators

Scope terminators mark the end of certain Procedure Division statements. There are two types:

1. **Explicit scope terminators** — keywords such as `END-IF`, `END-READ`, `END-PERFORM`, etc.
2. **Implicit scope terminator** — the period (`.`) that ends a sentence, which terminates all open statements within that sentence.

```cobol
*> Explicit scope terminators (recommended)
IF WS-CODE = "A"
    PERFORM PROCESS-A
ELSE
    IF WS-CODE = "B"
        PERFORM PROCESS-B
    ELSE
        PERFORM PROCESS-DEFAULT
    END-IF
END-IF

*> Implicit scope terminator (period-delimited, older style)
IF WS-CODE = "A"
    PERFORM PROCESS-A
ELSE
    PERFORM PROCESS-DEFAULT.
```

!!! warning "Pitfall"
    Mixing explicit scope terminators with period-delimited scope can cause subtle bugs. A misplaced period terminates all open statements in the sentence, not just the innermost one. Modern COBOL programs should use explicit scope terminators exclusively, with a period only at the end of each paragraph's last sentence.

!!! note "COBOL-85"
    Explicit scope terminators (`END-IF`, `END-READ`, `END-PERFORM`, etc.) were introduced in COBOL-85. Programs conforming to COBOL-68 or COBOL-74 rely on the period as the sole scope terminator.

---

## See Also

- [Identification Division](../identification-division/index.md)
- [Environment Division](../environment-division/index.md)
- [Data Division](../data-division/index.md)
