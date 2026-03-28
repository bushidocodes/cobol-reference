# Environment Division

The Environment Division specifies the computing environment in which the program operates and defines the mapping between logical file references in the program and the physical files of the operating environment. It is the second of the four COBOL divisions and is optional.

- **Standard:** COBOL-68, COBOL-85, COBOL 2002, COBOL 2014, COBOL 2023

---

## Syntax

```cobol
[ ENVIRONMENT DIVISION.

  [ CONFIGURATION SECTION.
    [ SOURCE-COMPUTER.  compilation-computer-specification.  ]
    [ OBJECT-COMPUTER.  execution-computer-specification.    ]
    [ REPOSITORY.       prototype-specification...        .  ]
    [ SPECIAL-NAMES.    program-configuration-specification. ] ]

  [ INPUT-OUTPUT SECTION.
    [ FILE-CONTROL.     general-file-description...       .  ]
    [ I-O-CONTROL.      file-buffering-specification...   .  ] ] ]
```

The Environment Division contains two sections: the **Configuration Section** and the **Input-Output Section**. Both are optional, as is the division itself. If neither section is needed, the entire `ENVIRONMENT DIVISION.` header may be omitted.

### Evolution Across Standards

| Element | COBOL-68 | COBOL-85 | COBOL 2002 | COBOL 2014 |
|---------|----------|----------|------------|------------|
| ENVIRONMENT DIVISION | Required | Required | Optional | Optional |
| CONFIGURATION SECTION | Required | Optional | Optional | Optional |
| SOURCE-COMPUTER | Required | Optional | Obsolete | Removed |
| OBJECT-COMPUTER | Required | Optional | Optional | Optional |
| SPECIAL-NAMES | Optional | Optional | Optional | Optional |
| REPOSITORY | -- | -- | Added | Optional |
| INPUT-OUTPUT SECTION | Optional | Optional | Optional | Optional |

---

## Configuration Section

The Configuration Section describes the computer environment in which the program is compiled and executed. It may also define special names, custom alphabets, currency symbols, and references to classes and functions.

```cobol
CONFIGURATION SECTION.
[ SOURCE-COMPUTER. computer-name [ WITH DEBUGGING MODE ]. ]
[ OBJECT-COMPUTER. computer-name
    [ MEMORY SIZE integer { WORDS | CHARACTERS | MODULES } ]
    [ PROGRAM COLLATING SEQUENCE IS alphabet-name ]
    [ SEGMENT-LIMIT IS segment-number ]. ]
[ REPOSITORY. ... ]
[ SPECIAL-NAMES. ... ]
```

!!! note "Ordering"
    When present, the paragraphs within the Configuration Section must appear
    in the order shown: SOURCE-COMPUTER, OBJECT-COMPUTER, REPOSITORY (COBOL 2002+),
    SPECIAL-NAMES.

---

### SOURCE-COMPUTER

The `SOURCE-COMPUTER` paragraph identifies the computer on which the program is compiled.

!!! warning "Removed"
    The SOURCE-COMPUTER paragraph was classified as **obsolete** in COBOL 2002
    and **removed** in COBOL 2014. Modern compilers still accept it for
    backward compatibility but ignore its content. The only functional clause,
    `WITH DEBUGGING MODE`, has been superseded by compiler options.

```cobol
SOURCE-COMPUTER. computer-name [ WITH DEBUGGING MODE ].
```

- **computer-name** -- a user-defined word. Treated as a comment by all modern compilers; its value has no effect.
- **WITH DEBUGGING MODE** -- when specified, all debugging lines (lines with `D` in the indicator area, column 7) are compiled as executable code, and all `USE FOR DEBUGGING` declaratives are activated. When omitted, debugging lines are treated as comments.

```cobol
SOURCE-COMPUTER. IBM-370.
SOURCE-COMPUTER. IBM-370 WITH DEBUGGING MODE.
SOURCE-COMPUTER. X86-64 WITH DEBUGGING MODE.
```

**Modern replacement:** Most compilers provide a command-line option or compiler directive to enable debugging mode. For example:

| Compiler | Debugging Option |
|----------|-----------------|
| IBM Enterprise COBOL | `TEST` / `NOTEST` compiler option |
| Micro Focus | `DEBUGGING` directive |
| GnuCOBOL | `-fdebugging-line` flag |

---

### OBJECT-COMPUTER

The `OBJECT-COMPUTER` paragraph identifies the computer on which the program is intended to run and specifies certain execution characteristics.

```cobol
OBJECT-COMPUTER. computer-name
    [ MEMORY SIZE integer { WORDS | CHARACTERS | MODULES } ]
    [ PROGRAM COLLATING SEQUENCE IS alphabet-name ]
    [ SEGMENT-LIMIT IS segment-number ]
    [ CHARACTER CLASSIFICATION IS locale-name ].
```

- **computer-name** -- a user-defined word. Treated as a comment by modern compilers.

#### MEMORY SIZE (removed)

!!! warning "Removed"
    The MEMORY SIZE clause was classified as **obsolete** in COBOL-85 and
    **removed** in COBOL 2002. It is universally ignored by modern compilers.
    Memory management is handled by the operating system.

```cobol
*> Obsolete -- do not use in new programs
OBJECT-COMPUTER. IBM-370
    MEMORY SIZE 65536 CHARACTERS.
```

#### PROGRAM COLLATING SEQUENCE

Specifies the alphabet-name that defines the character comparison order used in nonnumeric comparisons and in `SORT`/`MERGE` statements when no `COLLATING SEQUENCE` phrase is specified on those statements. The alphabet-name must be defined in the `SPECIAL-NAMES` paragraph.

```cobol
OBJECT-COMPUTER. X86-64
    PROGRAM COLLATING SEQUENCE IS MY-SORT-ORDER.
```

If this clause is omitted, the native collating sequence of the system is used (EBCDIC on IBM mainframes, ASCII on most other platforms).

This is the **only clause in OBJECT-COMPUTER that is still functionally relevant** in modern COBOL.

#### SEGMENT-LIMIT (removed)

!!! warning "Removed"
    The SEGMENT-LIMIT clause was classified as **obsolete** in COBOL 2002 and
    **removed** in COBOL 2014. Segmentation was a memory overlay mechanism
    from the era of constrained address spaces. It is irrelevant on modern
    systems with virtual memory.

```cobol
*> Obsolete -- do not use in new programs
OBJECT-COMPUTER. IBM-370
    SEGMENT-LIMIT IS 49.
```

#### CHARACTER CLASSIFICATION (COBOL 2002)

!!! note "COBOL 2002"
    The CHARACTER CLASSIFICATION clause was added in COBOL 2002.

Specifies the locale that determines the classification of characters (which characters are alphabetic, uppercase, lowercase, etc.).

```cobol
OBJECT-COMPUTER. X86-64
    CHARACTER CLASSIFICATION IS locale-name.
```

---

### REPOSITORY

*Added in COBOL 2002.*

The `REPOSITORY` paragraph specifies references to external classes, interfaces, and user-defined functions. It also controls the availability of intrinsic functions.

```cobol
REPOSITORY.
    [ CLASS class-name AS "external-name" ] ...
    [ INTERFACE interface-name AS "external-name" ] ...
    [ FUNCTION function-name ] ...
    [ FUNCTION { ALL INTRINSIC | intrinsic-name } ] ...
```

#### Intrinsic Function References

```cobol
*> Make ALL intrinsic functions available without FUNCTION keyword
REPOSITORY.
    FUNCTION ALL INTRINSIC.

*> After this, you can write:
MOVE UPPER-CASE(WS-INPUT) TO WS-OUTPUT
*> Instead of:
MOVE FUNCTION UPPER-CASE(WS-INPUT) TO WS-OUTPUT
```

#### User-Defined Function References

```cobol
*> Register user-defined functions for invocation
REPOSITORY.
    FUNCTION calc-sales-tax
    FUNCTION is-valid-date
    FUNCTION ALL INTRINSIC.
```

See [User-Defined Functions](../intrinsic-functions/user-defined.md) for full details.

#### Class and Interface References

```cobol
*> OO COBOL class references
REPOSITORY.
    CLASS CustomerAccount AS "customer-account"
    CLASS BaseObject AS "base"
    INTERFACE Printable AS "printable".
```

---

### SPECIAL-NAMES

The `SPECIAL-NAMES` paragraph associates implementor-names with user-defined names, defines the currency symbol, overrides the decimal point character, establishes alphabet names, defines user class conditions, and assigns symbolic names to characters.

This paragraph has extensive functionality documented on its own page. A summary of the available clauses:

| Clause | Purpose | Notes |
|--------|---------|-------|
| Implementor-names | Associates system devices with mnemonic names | `CONSOLE IS TTY` |
| `ALPHABET` | Defines a named collating sequence | `STANDARD-1`, `STANDARD-2`, `NATIVE`, or custom |
| `SYMBOLIC CHARACTERS` | Assigns names to character positions | `SYM-TAB IS 10` |
| `CLASS` | Defines user class conditions | `CLASS VALID-ID IS "A" THRU "Z"` |
| `CURRENCY SIGN` | Defines the currency symbol | Multi-character in COBOL 2002+ |
| `DECIMAL-POINT IS COMMA` | Swaps period and comma roles | Common in European locales |
| `CURSOR` | Cursor position for screen I/O | Used with Screen Section |
| `CRT STATUS` | Screen I/O termination status | Used with Screen Section |
| `LOCALE` | Associates a locale name | COBOL 2002+ |
| Switches | Maps external switches to conditions | `SWITCH-1 ON STATUS IS DEBUG-ON` |

See [SPECIAL-NAMES](special-names.md) for full syntax, rules, and examples.

---

## Input-Output Section

The Input-Output Section defines the files used by the program and specifies information needed for their transmission and handling.

```cobol
INPUT-OUTPUT SECTION.
[ FILE-CONTROL.
    SELECT ... ]
[ I-O-CONTROL.
    ... ]
```

---

### FILE-CONTROL

The `FILE-CONTROL` paragraph contains one or more `SELECT` statements. Each `SELECT` statement associates a logical file name with a physical file and specifies the file's organization, access mode, and status variable.

```cobol
FILE-CONTROL.
    SELECT file-name
        ASSIGN TO assignment-name
        [ ORGANIZATION IS { SEQUENTIAL | RELATIVE | INDEXED | LINE SEQUENTIAL } ]
        [ ACCESS MODE IS { SEQUENTIAL | RANDOM | DYNAMIC } ]
        [ RECORD KEY IS key-name ]
        [ ALTERNATE RECORD KEY IS alt-key
            [ WITH DUPLICATES ] ]
        [ RELATIVE KEY IS relative-key ]
        [ FILE STATUS IS status-field ]
        [ LOCK MODE IS { MANUAL | AUTOMATIC }
            [ WITH LOCK ON { RECORD | MULTIPLE RECORDS } ] ].
```

#### File Organization Summary

| Organization | Description | Access Modes | Key Required |
|-------------|-------------|--------------|-------------|
| `SEQUENTIAL` | Records accessed in order of physical position | Sequential only | No |
| `LINE SEQUENTIAL` | Text file with line delimiters (extension) | Sequential only | No |
| `RELATIVE` | Records accessed by relative record number | Sequential, Random, Dynamic | `RELATIVE KEY` |
| `INDEXED` | Records accessed by one or more key fields | Sequential, Random, Dynamic | `RECORD KEY` |

#### Access Mode Summary

| Mode | Description |
|------|-------------|
| `SEQUENTIAL` | Records are read/written in order |
| `RANDOM` | Records are read/written by key or record number |
| `DYNAMIC` | Both sequential and random access within the same OPEN |

#### Examples

```cobol
FILE-CONTROL.
    *> Sequential text file
    SELECT REPORT-FILE
        ASSIGN TO "REPORT.TXT"
        ORGANIZATION IS LINE SEQUENTIAL
        FILE STATUS IS WS-RPT-STATUS.

    *> Indexed file with primary and alternate keys
    SELECT CUSTOMER-FILE
        ASSIGN TO "CUSTFILE"
        ORGANIZATION IS INDEXED
        ACCESS MODE IS DYNAMIC
        RECORD KEY IS CUST-ID
        ALTERNATE RECORD KEY IS CUST-NAME
            WITH DUPLICATES
        FILE STATUS IS WS-CUST-STATUS.

    *> Relative file
    SELECT HASH-FILE
        ASSIGN TO "HASHFILE"
        ORGANIZATION IS RELATIVE
        ACCESS MODE IS RANDOM
        RELATIVE KEY IS WS-SLOT-NUMBER
        FILE STATUS IS WS-HASH-STATUS.
```

See [SELECT](select.md) for full syntax and details.

---

### I-O-CONTROL

The `I-O-CONTROL` paragraph specifies information about sharing memory areas among files and about special I/O techniques.

!!! warning "Largely Obsolete"
    Most clauses in I-O-CONTROL are obsolete relics from the era of tape-based
    storage and constrained memory. The `SAME RECORD AREA` clause is the only
    one with occasional modern relevance.

```cobol
I-O-CONTROL.
    [ SAME { AREA | RECORD AREA | SORT AREA | SORT-MERGE AREA }
        FOR file-name-1 file-name-2 [ file-name-3 ] ... ] ...
    [ APPLY WRITE-ONLY ON file-name-1 [ file-name-2 ] ... ]
    [ MULTIPLE FILE TAPE CONTAINS
        file-name-1 [ POSITION integer-1 ]
      [ file-name-2 [ POSITION integer-2 ] ] ... ].
```

| Clause | Purpose | Status |
|--------|---------|--------|
| `SAME AREA` | Files share the same memory buffer for processing. Only one of the named files may be open at a time. | Obsolete (COBOL 2002) |
| `SAME RECORD AREA` | Files share the same record area. When a record is read from any named file, it is available in the record area of all. Useful for file-to-file copying without an intermediate MOVE. | Still functional |
| `SAME SORT AREA` | Sort work files share the same memory area. | Obsolete (COBOL 2002) |
| `SAME SORT-MERGE AREA` | Sort/merge work files share the same memory area. | Obsolete (COBOL 2002) |
| `APPLY WRITE-ONLY` | Optimizes buffer handling for sequential output files (IBM extension). | Vendor extension |
| `MULTIPLE FILE TAPE CONTAINS` | Associates multiple logical files with a single physical tape reel. | Removed (COBOL 2002) |

#### SAME RECORD AREA Example

```cobol
I-O-CONTROL.
    SAME RECORD AREA FOR INPUT-FILE OUTPUT-FILE.

*> In the PROCEDURE DIVISION, after reading:
READ INPUT-FILE
    AT END SET WS-EOF TO TRUE
END-READ
*> The record is immediately available as OUTPUT-FILE's record
*> without requiring a MOVE statement
WRITE OUTPUT-RECORD
```

---

## Deprecation and Removal Summary

The following table summarizes elements that have been deprecated, made obsolete, or removed across COBOL standards:

| Element | Introduced | Obsolete | Removed | Replacement |
|---------|-----------|----------|---------|-------------|
| `SOURCE-COMPUTER` paragraph | COBOL-68 | COBOL 2002 | COBOL 2014 | Compiler options for debugging |
| `WITH DEBUGGING MODE` | COBOL-68 | COBOL 2002 | COBOL 2014 | Compiler options (`-fdebugging-line`, `TEST`, etc.) |
| `MEMORY SIZE` clause | COBOL-68 | COBOL-85 | COBOL 2002 | OS memory management |
| `SEGMENT-LIMIT` clause | COBOL-68 | COBOL 2002 | COBOL 2014 | Virtual memory (OS-level) |
| `SAME AREA` | COBOL-68 | COBOL 2002 | -- | No longer needed with modern memory |
| `SAME SORT AREA` | COBOL-68 | COBOL 2002 | -- | Compiler manages sort buffers |
| `SAME SORT-MERGE AREA` | COBOL-68 | COBOL 2002 | -- | Compiler manages sort/merge buffers |
| `MULTIPLE FILE TAPE` | COBOL-68 | COBOL-85 | COBOL 2002 | Not applicable (tape is rare) |
| `RERUN` clause | COBOL-68 | COBOL-85 | COBOL 2002 | Transaction/checkpoint systems |

!!! tip "Legacy Code"
    When maintaining older programs, you will encounter these obsolete elements
    frequently. Most compilers still accept them silently. There is no need to
    remove them unless you are modernizing the codebase, but you should not use
    them in new programs.

---

## Complete Example

```cobol
       ENVIRONMENT DIVISION.

       CONFIGURATION SECTION.
      *> SOURCE-COMPUTER is obsolete but still accepted
       SOURCE-COMPUTER. X86-64 WITH DEBUGGING MODE.
       OBJECT-COMPUTER. X86-64
           PROGRAM COLLATING SEQUENCE IS ASCII-ORDER.

       REPOSITORY.
           FUNCTION calc-sales-tax
           FUNCTION ALL INTRINSIC.

       SPECIAL-NAMES.
           ALPHABET ASCII-ORDER IS STANDARD-1
           CURRENCY SIGN IS "$"
           DECIMAL-POINT IS PERIOD
           CLASS ALPHA-NUM IS "A" THRU "Z", "a" THRU "z",
                               "0" THRU "9"
           SYMBOLIC CHARACTERS SYM-TAB IS 10
           CONSOLE IS TTY
           ENVIRONMENT-NAME IS ENV-NAME
           ENVIRONMENT-VALUE IS ENV-VALUE.

       INPUT-OUTPUT SECTION.
       FILE-CONTROL.
           SELECT MASTER-FILE
               ASSIGN TO "MASTER.DAT"
               ORGANIZATION IS INDEXED
               ACCESS MODE IS DYNAMIC
               RECORD KEY IS MF-ACCOUNT-NO
               ALTERNATE RECORD KEY IS MF-CUSTOMER-NAME
                   WITH DUPLICATES
               FILE STATUS IS WS-MASTER-STATUS.

           SELECT TRANS-FILE
               ASSIGN TO "TRANS.DAT"
               ORGANIZATION IS SEQUENTIAL
               FILE STATUS IS WS-TRANS-STATUS.

           SELECT REPORT-FILE
               ASSIGN TO "REPORT.RPT"
               ORGANIZATION IS LINE SEQUENTIAL
               FILE STATUS IS WS-RPT-STATUS.

       I-O-CONTROL.
           SAME RECORD AREA FOR TRANS-FILE REPORT-FILE.
```

---

## Minimal Environment Division

Many programs need only a FILE-CONTROL entry:

```cobol
       ENVIRONMENT DIVISION.
       INPUT-OUTPUT SECTION.
       FILE-CONTROL.
           SELECT INPUT-FILE
               ASSIGN TO "DATA.TXT"
               ORGANIZATION IS LINE SEQUENTIAL
               FILE STATUS IS WS-STATUS.
```

Programs that perform no file I/O can omit the entire division:

```cobol
       IDENTIFICATION DIVISION.
       PROGRAM-ID. HELLO.
       PROCEDURE DIVISION.
           DISPLAY "Hello, World!"
           STOP RUN.
```

---

## See Also

- [SPECIAL-NAMES](special-names.md) -- full reference for the SPECIAL-NAMES paragraph
- [SELECT](select.md) -- full reference for file-control entries
- [Identification Division](../identification-division/index.md) -- program identification
- [Data Division](../data-division/index.md) -- data definitions
- [Procedure Division](../procedure-division/index.md) -- executable statements
- [Collating Sequence](../appendices/collating-sequence.md) -- character ordering reference
