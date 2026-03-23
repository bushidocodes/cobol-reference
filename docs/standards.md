# COBOL Standards

COBOL has evolved through a series of CODASYL specifications and ANSI/ISO standard revisions since 1960. Each revision builds on the previous one, adding new features while maintaining backward compatibility with existing programs.

## COBOL-60

**CODASYL specification (pre-standard)**

The original COBOL specification, produced by the Conference on Data Systems Languages (CODASYL) Short-Range Committee and published in April 1960. It established the fundamental English-like syntax and four-division program structure. COBOL-60 was a specification rather than a formal standard, created by a committee convened at the Pentagon in May 1959.

## COBOL-61

**CODASYL specification (pre-standard)**

A cleanup of COBOL-60, addressing numerous logical flaws found in the original specification. Published in 1961, it replaced COBOL-60 and became the first version widely implemented by compiler vendors.

## COBOL-61 EXTENDED

**CODASYL specification (pre-standard)**

Published in 1963, this revision added the SORT and REPORT WRITER features to the language.

## COBOL-65

**CODASYL specification (pre-standard)**

Published in 1965, this revision introduced facilities for mass storage file handling and tables (OCCURS clause with DEPENDING ON). COBOL-65 also brought improvements to the SORT feature and added the ability to define library text for inclusion (the precursor to the COPY statement).

## COBOL-68

**ANSI X3.23-1968**

The first ANSI standard for COBOL. It formalized the language as defined by the Conference on Data Systems Languages (CODASYL) and established the four-division program structure, the PICTURE clause, level-number data hierarchy, and the core set of Procedure Division verbs (MOVE, ADD, SUBTRACT, MULTIPLY, DIVIDE, IF, PERFORM, GO TO, READ, WRITE, OPEN, CLOSE).

COBOL-68 defined two source formats (fixed-form with 80-column layout) and introduced the concept of functional processing modules at different implementation levels.

## COBOL-74

**ANSI X3.23-1974**

The second ANSI revision. Notable additions include:

- **INSPECT** statement for tallying and replacing characters
- **STRING** and **UNSTRING** statements for string concatenation and splitting
- **Communication module** for message handling
- Improvements to the SORT and MERGE features
- Extended WRITE ADVANCING capabilities

COBOL-74 also refined the module system and clarified many areas left ambiguous by COBOL-68.

## COBOL-85

**ANSI X3.23-1985 / ISO 1989:1985**

The most widely implemented COBOL standard and the de facto baseline for production COBOL code worldwide. COBOL-85 introduced substantial structural improvements:

- **Scope terminators** (END-IF, END-PERFORM, END-READ, END-EVALUATE, etc.)
- **Inline PERFORM** — PERFORM with imperative statements between PERFORM and END-PERFORM
- **EVALUATE** statement — multi-branch conditional (analogous to switch/case)
- **Reference modification** — substring access via `identifier(start:length)`
- **CONTINUE** statement — explicit no-operation
- **Nested programs** — programs defined within other programs, with shared data via GLOBAL and EXTERNAL clauses
- **NOT ON SIZE ERROR**, **NOT AT END**, **NOT INVALID KEY** — negative condition phrases
- **INITIALIZE** statement — structured data initialization
- **Day-of-week intrinsic** via ACCEPT FROM DAY-OF-WEEK
- Improvements to file handling, including LINAGE for report formatting

Two amendments followed: Amendment 1 (1989) added **intrinsic functions** (FUNCTION LENGTH, FUNCTION CURRENT-DATE, FUNCTION UPPER-CASE, etc.), and Amendment 2 (1993) added corrections and clarifications.

## COBOL 2002

**ISO/IEC 1989:2002**

A major revision that modernized the language:

- **Object-oriented programming** — classes, methods, interfaces, INVOKE statement, object references
- **Free-form source format** — removal of the fixed-column constraints
- **User-defined functions** — FUNCTION-ID paragraph, RETURNING clause
- **LOCALE support** — locale-sensitive collating sequences and formatting
- **Unicode support** — national (UTF-16) data items via USAGE NATIONAL and PIC N
- **Binary and boolean data types** — USAGE BIT
- **TYPEDEF and SAME AS** — user-defined data types
- **Multiple currency signs** — CURRENCY SIGN clause with PICTURE SYMBOL
- **Expanded CALL conventions** — BY VALUE parameter passing
- **Conditional compilation** — >>DEFINE, >>IF, >>ELSE, >>END-IF directives
- **Exception handling** — RAISE and RESUME statements
- **Pointer arithmetic** and expanded pointer support

## COBOL 2014

**ISO/IEC 1989:2014**

A focused revision building on COBOL 2002:

- **Dynamic-capacity tables** — OCCURS DYNAMIC, ALLOCATE/FREE statements for variable-length tables
- **VALIDATE statement** — data validation against class conditions and ranges
- **JSON and XML extensions** — JSON GENERATE, JSON PARSE, XML GENERATE, XML PARSE
- **Enhanced arithmetic** — IEEE 754 floating-point via USAGE FLOAT-BINARY, FLOAT-DECIMAL, FLOAT-EXTENDED
- **FUNCTION FORMATTED-CURRENT-DATE** and related date/time functions
- **Interface enhancements** — factory methods, stronger typing for object references
- **COMMIT and ROLLBACK** — transaction-oriented I/O verbs
- **Enhanced DISPLAY and ACCEPT** — screen handling improvements

## COBOL 2023

**ISO/IEC 1989:2023**

The latest revision of the COBOL standard:

- **Asynchronous messaging** — SEND and RECEIVE statements for message-based communication
- **Transaction processing** — COMMIT and ROLLBACK facility (moved from COBOL 2014 draft)
- **XOR logical operator** — exclusive-or in conditional expressions
- **CONTINUE with duration** — pause program execution for a specified time
- **DELETE FILE** statement — delete a file from external storage
- **LINE SEQUENTIAL** file organization — standardization of a widely-used vendor extension
- **PERFORM UNTIL EXIT** — defined infinite looping construct
- **SUBSTITUTE** intrinsic function — substring substitution with different-length replacement
- **CONVERT** intrinsic function — base conversion
- **Boolean shifting operators** — logical shift operations on boolean data

GCC 15.1 introduced `gcobol`, a COBOL front-end based on this standard.

## Feature Summary by Standard

| Feature | Version |
|---|---|
| Four-division structure, PICTURE, level numbers | COBOL-60 |
| SORT, REPORT WRITER | COBOL-61 EXT |
| OCCURS DEPENDING ON, mass storage files | COBOL-65 |
| First ANSI standard, module system | COBOL-68 |
| INSPECT, STRING, UNSTRING | COBOL-74 |
| DELETE statement, file organizations | COBOL-74 |
| Scope terminators (END-IF, etc.) | COBOL-85 |
| Inline PERFORM | COBOL-85 |
| EVALUATE | COBOL-85 |
| Reference modification | COBOL-85 |
| Nested programs | COBOL-85 |
| CONTINUE, INITIALIZE | COBOL-85 |
| Intrinsic functions | COBOL-85 Amdt. 1 |
| Object-oriented programming | COBOL 2002 |
| Free-form source format | COBOL 2002 |
| User-defined functions | COBOL 2002 |
| Unicode / USAGE NATIONAL | COBOL 2002 |
| Conditional compilation | COBOL 2002 |
| RAISE / RESUME exception handling | COBOL 2002 |
| Dynamic-capacity tables | COBOL 2014 |
| JSON GENERATE / JSON PARSE | COBOL 2014 |
| XML GENERATE / XML PARSE | COBOL 2014 |
| VALIDATE statement | COBOL 2014 |
| IEEE 754 arithmetic types | COBOL 2014 |
| Asynchronous messaging (SEND/RECEIVE) | COBOL 2023 |
| COMMIT / ROLLBACK | COBOL 2023 |
| XOR operator, boolean shifting | COBOL 2023 |
| LINE SEQUENTIAL organization | COBOL 2023 |
| PERFORM UNTIL EXIT | COBOL 2023 |

## Compiler Support

Most production COBOL code targets **COBOL-85**. All major compilers fully support this standard:

- **IBM Enterprise COBOL** (z/OS) — full COBOL-85 support; selective COBOL 2002/2014 features (free-form source, XML/JSON statements, some OO support)
- **Micro Focus Visual COBOL / COBOL Server** — full COBOL-85 support; extensive COBOL 2002/2014 coverage including OO, managed types, and free-form source
- **GnuCOBOL** — full COBOL-85 support; partial COBOL 2002 support (free-form source, some intrinsic functions, binary data types); limited COBOL 2014 features
- **gcobol (GCC 15.1+)** — a COBOL front-end for GCC based on the COBOL 2023 standard

No compiler currently implements the full COBOL 2002 or COBOL 2014 specification. Feature coverage varies significantly between vendors, and many implementations include proprietary extensions beyond the standard.
