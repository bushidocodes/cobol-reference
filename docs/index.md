# COBOL Reference

This site is a comprehensive reference for the COBOL programming language (Common Business-Oriented Language). It documents the syntax, semantics, and behavior of COBOL as defined by the ANSI and ISO standards, with practical notes on compiler-specific behavior where relevant.

## Overview

COBOL is a compiled, English-like programming language designed for business data processing. First standardized in 1968, it remains one of the most widely deployed languages in production, particularly in banking, insurance, government, and mainframe environments. Billions of lines of COBOL are in active use today.

A COBOL program is organized into up to four **divisions**, each serving a distinct purpose. The divisions appear in a fixed order: Identification, Environment, Data, and Procedure.

## Reference Sections

### [Language](language/index.md)

General topics including program structure, source format (fixed-form and free-form), character sets, and language fundamentals such as words, literals, and separators.

### [Identification Division](identification-division/index.md)

The required first division of every COBOL program. Specifies the program name and optional informational paragraphs (AUTHOR, DATE-WRITTEN, etc.).

### [Environment Division](environment-division/index.md)

Describes the computing environment in which the program operates. Contains the Configuration Section (SOURCE-COMPUTER, OBJECT-COMPUTER, SPECIAL-NAMES) and the Input-Output Section (FILE-CONTROL, I-O-CONTROL).

### [Data Division](data-division/index.md)

Defines all data items used by the program. Contains the File Section, Working-Storage Section, Local-Storage Section, and Linkage Section.

Key reference pages:

- [PICTURE clause](data-division/picture.md) — data item editing and category
- [USAGE clause](data-division/usage.md) — internal representation (DISPLAY, COMP, PACKED-DECIMAL, etc.)
- [OCCURS clause](data-division/occurs.md) — tables and arrays

### [Procedure Division](procedure-division/index.md)

Contains the executable statements of the program, organized into sections and paragraphs.

Statement categories:

- **Arithmetic** — ADD, SUBTRACT, MULTIPLY, DIVIDE, COMPUTE
- **Data movement** — MOVE, INITIALIZE, SET
- **Control flow** — IF, EVALUATE, PERFORM, GO TO, STOP RUN
- **I/O** — OPEN, CLOSE, READ, WRITE, REWRITE, DELETE
- **String handling** — STRING, UNSTRING, INSPECT
- **Table handling** — SEARCH, SEARCH ALL
- **Program invocation** — CALL, CANCEL

### Intrinsic Functions

*Coming soon.* Reference for built-in functions such as FUNCTION LENGTH, FUNCTION CURRENT-DATE, FUNCTION NUMVAL, and others.

### COPY and REPLACE

*Coming soon.* Reference for the COPY statement (copybook inclusion) and the REPLACE statement (source text manipulation).

## Standards Coverage

This reference primarily documents **COBOL-85** (ANSI X3.23-1985 / ISO 1989:1985), which is the most widely implemented standard and the baseline for virtually all production COBOL code. Features introduced in COBOL 2002 and COBOL 2014 are noted where applicable and marked with the standard that introduced them.

See [Standards](standards.md) for a detailed history of COBOL standard revisions.
