# Program Structure

A COBOL program is organized into a strict hierarchy of **divisions**, **sections**, **paragraphs**, **sentences**, and **statements**. Every COBOL source program contains up to four divisions, which must appear in a fixed order.

**Standard:** COBOL-60, COBOL-61, COBOL-61 Extended, COBOL-65, COBOL-68, COBOL-85, COBOL 2002, COBOL 2014, COBOL 2023

---

## Divisions

A COBOL program consists of the following four divisions, in order:

| Division | Required | Purpose |
|---|---|---|
| [IDENTIFICATION DIVISION](../identification-division/index.md) | Yes | Names the program and provides optional metadata (author, date, remarks). |
| [ENVIRONMENT DIVISION](../environment-division/index.md) | No | Describes the computing environment: file assignments, special names, and repository information. |
| [DATA DIVISION](../data-division/index.md) | No | Defines all variables, records, files, and other data items used by the program. |
| [PROCEDURE DIVISION](../procedure-division/index.md) | Yes | Contains the executable statements that form the program's logic. |

The IDENTIFICATION DIVISION and PROCEDURE DIVISION are required in every program. The ENVIRONMENT DIVISION and DATA DIVISION are technically optional, but nearly all programs of practical use include both.

Each division begins with a division header, which is the division name followed by the keyword `DIVISION` and a period:

```cobol
IDENTIFICATION DIVISION.
ENVIRONMENT DIVISION.
DATA DIVISION.
PROCEDURE DIVISION.
```

## Structural Hierarchy

The complete structural hierarchy within a COBOL program is:

```
Program
  Division
    Section
      Paragraph
        Sentence
          Statement
```

### Divisions

A division is the highest-level structural unit. Each division groups a specific category of program information or logic.

### Sections

A section is a named subdivision within a division. In the ENVIRONMENT DIVISION and DATA DIVISION, sections have predefined names (e.g., `FILE SECTION`, `WORKING-STORAGE SECTION`). In the PROCEDURE DIVISION, sections are user-defined and serve as targets for `PERFORM` and `GO TO` statements.

A section header consists of the section name followed by the keyword `SECTION` and a period:

```cobol
WORKING-STORAGE SECTION.

INPUT-OUTPUT SECTION.
```

### Paragraphs

A paragraph is a named subdivision within a section (or directly within the PROCEDURE DIVISION if sections are not used). In the IDENTIFICATION DIVISION, paragraphs have predefined names (e.g., `PROGRAM-ID`, `AUTHOR`). In the PROCEDURE DIVISION, paragraphs are user-defined.

A paragraph header consists of the paragraph name followed by a period:

```cobol
PROGRAM-ID. HELLO-WORLD.

MAIN-LOGIC.
```

### Sentences

A sentence is one or more statements terminated by a period. Sentences exist only in the PROCEDURE DIVISION.

```cobol
MOVE 0 TO WS-COUNTER ADD 1 TO WS-COUNTER.
```

The above is a single sentence containing two statements. In modern COBOL practice, each sentence typically contains a single statement.

### Statements

A statement is a single executable instruction beginning with a COBOL verb (e.g., `MOVE`, `ADD`, `PERFORM`, `IF`, `DISPLAY`). A statement is the smallest executable unit.

## Sections Within Each Division

### IDENTIFICATION DIVISION

The IDENTIFICATION DIVISION contains the following paragraphs (only PROGRAM-ID is required):

- `PROGRAM-ID` -- specifies the program name (required)
- `AUTHOR` -- names the author (obsolete)
- `INSTALLATION` -- names the installation site (obsolete)
- `DATE-WRITTEN` -- records the date the program was written (obsolete)
- `DATE-COMPILED` -- records the compilation date (obsolete)
- `SECURITY` -- describes security restrictions (obsolete)

### ENVIRONMENT DIVISION

The ENVIRONMENT DIVISION contains two sections:

- `CONFIGURATION SECTION` -- specifies the source and object computers, special names, and the repository
- `INPUT-OUTPUT SECTION` -- defines file assignments and I/O control

### DATA DIVISION

The DATA DIVISION contains the following sections:

- `FILE SECTION` -- describes the structure of files
- `WORKING-STORAGE SECTION` -- defines variables that persist for the life of the program
- `LOCAL-STORAGE SECTION` -- defines variables that are allocated and initialized on each invocation (COBOL 2002+)
- `LINKAGE SECTION` -- describes data passed from a calling program
- `REPORT SECTION` -- defines report layouts (Report Writer)
- `SCREEN SECTION` -- defines screen layouts (extension, widely supported)

### PROCEDURE DIVISION

The PROCEDURE DIVISION contains user-defined sections and paragraphs. It may also include a `DECLARATIVES` region at the beginning, which defines exception-handling procedures:

```cobol
PROCEDURE DIVISION.
DECLARATIVES.
FILE-ERROR SECTION.
    USE AFTER STANDARD ERROR PROCEDURE ON INPUT.
FILE-ERROR-HANDLER.
    DISPLAY "File error occurred".
END DECLARATIVES.

MAIN-SECTION SECTION.
MAIN-LOGIC.
    PERFORM PROCESS-DATA
    STOP RUN.
```

## Scope Terminators

Conditional and inline statements may be terminated by explicit **scope terminators**. A scope terminator is a keyword beginning with `END-` that corresponds to the opening statement verb:

| Statement | Scope Terminator |
|---|---|
| `IF` | `END-IF` |
| `PERFORM` | `END-PERFORM` |
| `EVALUATE` | `END-EVALUATE` |
| `READ` | `END-READ` |
| `WRITE` | `END-WRITE` |
| `SEARCH` | `END-SEARCH` |
| `STRING` | `END-STRING` |
| `UNSTRING` | `END-UNSTRING` |
| `COMPUTE` | `END-COMPUTE` |
| `ADD` | `END-ADD` |
| `SUBTRACT` | `END-SUBTRACT` |
| `MULTIPLY` | `END-MULTIPLY` |
| `DIVIDE` | `END-DIVIDE` |
| `CALL` | `END-CALL` |
| `ACCEPT` | `END-ACCEPT` |
| `DISPLAY` | `END-DISPLAY` |
| `RETURN` | `END-RETURN` |
| `START` | `END-START` |
| `DELETE` | `END-DELETE` |
| `REWRITE` | `END-REWRITE` |

Explicit scope terminators are preferred over the implicit termination by period, because they make the program structure unambiguous and allow statements to be nested:

```cobol
IF WS-A > WS-B
    IF WS-A > WS-C
        DISPLAY "A is greatest"
    ELSE
        DISPLAY "C is greatest"
    END-IF
ELSE
    DISPLAY "A is not greater than B"
END-IF
```

Without explicit scope terminators, a period terminates all open statements simultaneously, which can produce unintended logic when statements are nested.

## Minimal Complete Program

The smallest valid COBOL program requires only the IDENTIFICATION DIVISION (with a `PROGRAM-ID` paragraph) and the PROCEDURE DIVISION:

```cobol
       IDENTIFICATION DIVISION.
       PROGRAM-ID. MINIMAL.
       PROCEDURE DIVISION.
           STOP RUN.
```

A more typical minimal program includes all four divisions:

```cobol
       IDENTIFICATION DIVISION.
       PROGRAM-ID. HELLO-WORLD.

       ENVIRONMENT DIVISION.

       DATA DIVISION.
       WORKING-STORAGE SECTION.
       01 WS-MESSAGE PIC X(13) VALUE "Hello, world!".

       PROCEDURE DIVISION.
       MAIN-LOGIC.
           DISPLAY WS-MESSAGE
           STOP RUN.
```

## See Also

- [Source Format](source-format.md) -- fixed-form and free-form source layouts
- [IDENTIFICATION DIVISION](../identification-division/index.md)
- [ENVIRONMENT DIVISION](../environment-division/index.md)
- [DATA DIVISION](../data-division/index.md)
- [PROCEDURE DIVISION](../procedure-division/index.md)
