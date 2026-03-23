# Environment Division

The Environment Division specifies the computing environment in which the program operates and defines the mapping between logical file references in the program and the physical files of the operating environment. It is optional.

**Standard:** COBOL-60, COBOL-61, COBOL-61 Extended, COBOL-65, COBOL-68, COBOL-85, COBOL 2002, COBOL 2014, COBOL 2023

---

## Syntax

```cobol
ENVIRONMENT DIVISION.

[CONFIGURATION SECTION.
    [SOURCE-COMPUTER. ...]
    [OBJECT-COMPUTER. ...]
    [SPECIAL-NAMES. ...]
    [REPOSITORY. ...]]

[INPUT-OUTPUT SECTION.
    [FILE-CONTROL. ...]
    [I-O-CONTROL. ...]]
```

The Environment Division contains two sections: the Configuration Section and the Input-Output Section. Both are optional.

---

## Configuration Section

The Configuration Section describes the computer environment in which the program is compiled and executed. It may also define special names and environment-specific behavior.

```cobol
CONFIGURATION SECTION.
[SOURCE-COMPUTER. computer-name [WITH DEBUGGING MODE].]
[OBJECT-COMPUTER. computer-name
    [MEMORY SIZE integer {WORDS | CHARACTERS | MODULES}]
    [PROGRAM COLLATING SEQUENCE IS alphabet-name]
    [SEGMENT-LIMIT IS segment-number].]
[SPECIAL-NAMES. ...]
[REPOSITORY. ...]
```

### SOURCE-COMPUTER

The `SOURCE-COMPUTER` paragraph identifies the computer on which the program is compiled.

```cobol
SOURCE-COMPUTER. IBM-370.
SOURCE-COMPUTER. IBM-370 WITH DEBUGGING MODE.
```

When `WITH DEBUGGING MODE` is specified, all debugging lines (lines with `D` in the indicator area) and all `USE FOR DEBUGGING` declaratives are compiled. When this clause is omitted, debugging lines are treated as comments and debugging declaratives are suppressed.

The computer-name entry is treated as a comment by most compilers. The `DEBUGGING MODE` clause is the operative element of this paragraph.

### OBJECT-COMPUTER

The `OBJECT-COMPUTER` paragraph identifies the computer on which the program is intended to run.

```cobol
OBJECT-COMPUTER. IBM-370
    PROGRAM COLLATING SEQUENCE IS EBCDIC-ORDER.
```

| Clause | Purpose |
|---|---|
| `MEMORY SIZE` | Specifies the amount of memory available. Obsolete in COBOL 2002; ignored by modern compilers. |
| `PROGRAM COLLATING SEQUENCE` | Specifies the alphabet-name that defines the character comparison order used in nonnumeric comparisons and the `SORT`/`MERGE` statements when no `COLLATING SEQUENCE` phrase is specified on those statements. |
| `SEGMENT-LIMIT` | Specifies the segment classification for fixed and independent segments. Obsolete in COBOL 2002. |

### SPECIAL-NAMES

The `SPECIAL-NAMES` paragraph associates implementor-names with user-defined names, defines the currency symbol, overrides the decimal point character, and establishes alphabet names and class definitions.

```cobol
SPECIAL-NAMES.
    CURRENCY SIGN IS "EUR"
    DECIMAL-POINT IS COMMA
    CLASS VALID-ID IS "A" THRU "Z", "0" THRU "9", "-"
    ALPHABET EBCDIC-ORDER IS NATIVE
    SYMBOLIC CHARACTERS TAB IS 10
    CONSOLE IS CRT
    ENVIRONMENT-NAME IS ENVIRONMENT-VALUE.
```

| Clause | Purpose |
|---|---|
| `CURRENCY SIGN IS literal` | Defines the currency symbol used in `PICTURE` strings. Defaults to `$`. COBOL 2002 permits multi-character currency symbols. |
| `DECIMAL-POINT IS COMMA` | Exchanges the roles of the period and comma in `PICTURE` character strings and numeric literals. |
| `CLASS class-name IS literal THRU literal` | Defines a user-specified class condition. The class-name may then appear in conditional expressions (e.g., `IF field IS VALID-ID`). |
| `ALPHABET alphabet-name IS ...` | Defines a collating sequence. Values may be `STANDARD-1` (ASCII), `STANDARD-2` (ISO 646), `NATIVE` (system default), or an explicit character ordering. |
| `SYMBOLIC CHARACTERS symbol IS integer` | Assigns a name to an ordinal character position within the program collating sequence. |
| Implementor-names | Associates system devices or environment entities (e.g., `CONSOLE`, `SYSIN`, `SYSOUT`) with user-defined names. |

### REPOSITORY

*COBOL 2002 and later.*

The `REPOSITORY` paragraph specifies the external names of classes and interfaces referenced in the program. It is also used to declare intrinsic function references.

```cobol
REPOSITORY.
    CLASS CustomerAccount AS "customer-account"
    FUNCTION ALL INTRINSIC.
```

The `FUNCTION ALL INTRINSIC` clause makes all intrinsic functions available without the `FUNCTION` keyword prefix. Without this clause, intrinsic functions are invoked as `FUNCTION UPPER-CASE(...)`.

---

## Input-Output Section

The Input-Output Section defines the files used by the program and specifies information needed for their transmission and handling.

```cobol
INPUT-OUTPUT SECTION.
FILE-CONTROL.
    SELECT ...
[I-O-CONTROL.
    ...]
```

### FILE-CONTROL

The `FILE-CONTROL` paragraph contains one or more [SELECT](select.md) statements. Each `SELECT` statement associates a logical file name (a file-control entry) with a physical file on the system and specifies the file's organization, access mode, and status variable.

```cobol
FILE-CONTROL.
    SELECT CUSTOMER-FILE
        ASSIGN TO "CUSTFILE"
        ORGANIZATION IS INDEXED
        ACCESS MODE IS DYNAMIC
        RECORD KEY IS CUST-ID
        FILE STATUS IS WS-FILE-STATUS.

    SELECT REPORT-FILE
        ASSIGN TO "REPORT.DAT"
        ORGANIZATION IS SEQUENTIAL
        FILE STATUS IS WS-RPT-STATUS.
```

See [SELECT statement](select.md) for full syntax and details.

### I-O-CONTROL

The `I-O-CONTROL` paragraph specifies information about sharing memory areas among files and about special I/O techniques. Most clauses in this paragraph are obsolete or have very limited modern relevance.

```cobol
I-O-CONTROL.
    SAME RECORD AREA FOR FILE-A FILE-B.
```

| Clause | Purpose |
|---|---|
| `SAME AREA` | Specifies that two or more files share the same memory area for processing. |
| `SAME RECORD AREA` | Specifies that two or more files share the same record area. When a record is read from any of the named files, it is available in the record area of all of them. |
| `SAME SORT AREA` | Specifies that the same memory area is used for sort work files. |
| `SAME SORT-MERGE AREA` | Specifies that the same memory area is used for sort/merge work files. |
| `MULTIPLE FILE TAPE CONTAINS` | Associates multiple files with a single physical tape reel. Obsolete. |

---

## Example

```cobol
       ENVIRONMENT DIVISION.

       CONFIGURATION SECTION.
       SOURCE-COMPUTER. X86-64 WITH DEBUGGING MODE.
       OBJECT-COMPUTER. X86-64
           PROGRAM COLLATING SEQUENCE IS ASCII-ORDER.
       SPECIAL-NAMES.
           ALPHABET ASCII-ORDER IS STANDARD-1
           CURRENCY SIGN IS "$"
           DECIMAL-POINT IS PERIOD
           CLASS ALPHA-NUM IS "A" THRU "Z", "a" THRU "z",
                               "0" THRU "9".

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

           SELECT PRINT-FILE
               ASSIGN TO "REPORT.RPT"
               ORGANIZATION IS SEQUENTIAL
               FILE STATUS IS WS-PRINT-STATUS.
```

## See also

- [Identification Division](../identification-division/index.md)
- [SELECT statement](select.md)
- [Data Division](../data-division/index.md)
- [Procedure Division](../procedure-division/index.md)
