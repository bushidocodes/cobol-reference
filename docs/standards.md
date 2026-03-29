# COBOL Standards

COBOL has evolved through two distinct phases: the early CODASYL specifications (1960-1965), which were industry reports produced by voluntary committees, and the formal ANSI/ISO standards (1968 onward), which carried the weight of national and international standardization bodies. Each revision builds on the previous one, adding new features while maintaining backward compatibility with existing programs.

---

## CODASYL Specifications (1960-1965)

The pre-standard era of COBOL was governed by the Conference on Data Systems Languages (CODASYL), a voluntary industry group that produced successive specification reports. These were not formal standards but served as the de facto definition of the language for early implementors.

### COBOL-60

**CODASYL Initial Specifications, April 1960** — *"COBOL - Initial Specifications for a COmmon Business Oriented Language"*

The very first COBOL specification and the origin of the language.

#### Origin

On May 28-29, 1959, a meeting was called at the Pentagon to consider creating a common language for business-type data processing. Three committees were formed to carry out the work: a **Short Range Committee**, an **Intermediate Range Committee**, and a **Long Range Committee**. The Short Range Committee was composed of six manufacturers and three Government representatives.

The Short Range Committee held its first meeting on June 23, 1959, and organized four working groups: Data Description, Procedural Statements, Application Survey, and Usage and Experience. During this work, the name **COBOL** was adopted, suggesting a **CO**mmon **B**usiness **O**riented **L**anguage.

The COBOL System was reviewed and approved by the Short Range Committee during November 16-20, 1959. The final report was delivered to the Executive Committee on December 17, 1959. The Executive Committee accepted and approved the report on January 7-8, 1960, and it was published in April 1960.

#### Contributing organizations

The Short Range Committee drew members from the following organizations:

- Air Materiel Command, U.S. Air Force
- Bureau of Standards, Department of Commerce
- Computer Science Corporation
- Datamatic Division, Minneapolis-Honeywell Corporation
- David Taylor Model Basin, Bureau of Ships, U.S. Navy
- ElectroData Division, Burroughs Corporation
- International Business Machines Corporation
- Radio Corporation of America
- Remington-Rand Division of Sperry-Rand, Inc.
- Sylvania Electric Products, Inc.

#### Source languages

COBOL drew on three existing languages:

- **FLOW-MATIC** (Sperry-Rand, 1958) — Grace Hopper's language, the primary influence on COBOL's English-like syntax
- **Commercial Translator** (IBM, 1959)
- **AIMACO** (Air Materiel Command and Sperry-Rand)

#### Phasing

The 1960 report defined three implementation levels:

- **Phase I (Basic COBOL)** — the minimum language that all compilers were expected to support
- **Phase II (COBOL)** — the present level, as defined in the full specification
- **Phase III (Extended COBOL)** — a more ideal system, reserved for future development

Basic COBOL (Phase I) excluded the following features: algebraic formulas, the COMPUTE verb, segmentation, multi-way IF, the REVERSE option in OPEN, the VARY option in PERFORM, the UNTIL option in PERFORM, and the INCLUDE, USE, DEFINE, and ENTER verbs.

#### Original COBOL-60 structure

COBOL-60 defined only **three divisions** (not four):

- **Environment Division**
- **Data Division**
- **Procedure Division**

There was no Identification Division; it was added later in COBOL-61.

The Data Division contained three sections: the **FILE Section**, the **WORKING-STORAGE Section**, and a **CONSTANT Section**.

The Procedure Division defined **23 verbs** in six categories:

| Category | Verbs |
|---|---|
| Arithmetic | ADD, SUBTRACT, MULTIPLY, DIVIDE, COMPUTE |
| Input-Output | READ, WRITE, OPEN, CLOSE, ACCEPT, DISPLAY |
| Procedure Branching | GO, ALTER, PERFORM |
| Data Movement | MOVE, EXAMINE |
| Ending | STOP |
| Compiler Directing | DEFINE, ENTER, EXIT, INCLUDE, NOTE, USE |

#### Maintenance and future direction

Two standing committees were established to maintain COBOL: a **Technical Committee** (composed of manufacturers) and an **Executive Committee** (with broader membership). The original report noted:

> "The prospective user of COBOL is reminded that a source program language such as COBOL must be dynamic to be effective."

Features explicitly planned for future versions included Table Handling Functions, External Format and Media Translation, Report Writing Extension, and Sort-Merge Functions.

### COBOL-61

**CODASYL specification (pre-standard)**

A cleanup of COBOL-60, addressing numerous logical flaws found in the original specification. Published in 1961, it replaced COBOL-60 and became the first version widely implemented by compiler vendors.

#### Structure

COBOL-61 was split into two compliance levels:

- **REQUIRED COBOL-1961** — the minimum subset that every conforming implementation must support
- **ELECTIVE COBOL-1961** — optional features that implementations could choose to include

#### Maintenance Group

Beyond the original ten organizations on the Short Range Committee, additional organizations joined the COBOL Maintenance Group, including **Allstate Insurance Company** and others from both the insurance and manufacturing sectors.

#### Identification Division

COBOL-61 introduced the **Identification Division**, establishing the four-division program structure that would become a defining characteristic of the language. The division defined seven paragraphs:

- **PROGRAM-ID** (required, must be the first paragraph)
- **AUTHOR** (optional)
- **INSTALLATION** (optional)
- **DATE-WRITTEN** (optional)
- **DATE-COMPILED** (optional)
- **SECURITY** (optional)
- **REMARKS** (optional)

#### Data Division

The Data Division retained three sections from COBOL-60: **FILE**, **WORKING-STORAGE**, and **CONSTANT**. There was no LINKAGE Section; parameter passing between programs was not yet standardized.

Record descriptions used separate **SIZE**, **CLASS**, **POINT LOCATION**, and **SIGNED** clauses for defining data item characteristics, though the **PICTURE** clause was also available as a more concise alternative. These separate clauses were later unified into PICTURE in subsequent standards.

**Level-77** items were used for independent (non-grouped) work areas and constants in both the Working-Storage and Constant sections.

**TALLY** was a special register — a system-defined counter used with the **EXAMINE** verb (the predecessor to INSPECT).

#### Procedure Division

- **EXAMINE** was the string-scanning verb, later replaced by **INSPECT** in COBOL-74.
- **OTHERWISE** was used in IF statements where **ELSE** would later be used.

#### COPY clause

The COPY clause was pervasive in COBOL-61, available for copying library text into **FD entries**, **Record Description entries**, **OBJECT-COMPUTER**, **SPECIAL-NAMES**, **FILE-CONTROL**, and **I-O-CONTROL** paragraphs.

#### FILE-CONTROL RENAMING

FILE-CONTROL included a **RENAMING** option that allowed one file to share another file's File Description, avoiding the need to duplicate record layouts.

#### Reserved words

The COBOL-61 reserved word list included words that were later removed or replaced in subsequent standards: **CONSTANT**, **RENAMING**, **TALLY**, **EXAMINE**, **OTHERWISE**, **PROTECT**, **CHECK**, **PLACE**, **FLOAT**, **DOLLAR**, **DIGIT**, **LEAVING**, **SUPPRESS**, **LOCATION**, **ADDRESS**.

### COBOL-61 EXTENDED

**CODASYL specification (pre-standard)**

Published in 1963, this revision added the SORT and REPORT WRITER features to the language.

### COBOL-65

**CODASYL specification (pre-standard)**

Published in 1965, this revision introduced facilities for mass storage file handling and tables (OCCURS clause with DEPENDING ON). COBOL-65 also brought improvements to the SORT feature and added the ability to define library text for inclusion (the precursor to the COPY statement).

---

## ANSI/ISO Standards (1968-present)

Beginning in 1968, COBOL became a formally standardized language under the American National Standards Institute (ANSI) and later the International Organization for Standardization (ISO). These standards carry formal authority and define conformance requirements for compiler vendors.

### COBOL-68

**ANSI X3.23-1968**

The first formal ANSI standard for COBOL. Building on the CODASYL specifications, it codified the four-division program structure (as established by COBOL-61), the PICTURE clause, level-number data hierarchy, and the core set of Procedure Division verbs (MOVE, ADD, SUBTRACT, MULTIPLY, DIVIDE, IF, PERFORM, GO TO, READ, WRITE, OPEN, CLOSE).

COBOL-68 defined two source formats (fixed-form with 80-column layout) and introduced the concept of functional processing modules at different implementation levels.

### COBOL-74

**ANSI X3.23-1974**

The second ANSI revision. Notable additions include:

- **INSPECT** statement for tallying and replacing characters
- **STRING** and **UNSTRING** statements for string concatenation and splitting
- **Communication module** for message handling
- Improvements to the SORT and MERGE features
- Extended WRITE ADVANCING capabilities

COBOL-74 also refined the module system and clarified many areas left ambiguous by COBOL-68.

### COBOL-85

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

### COBOL 2002

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

### COBOL 2014

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

### COBOL 2023

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
| Three-division structure, PICTURE, level numbers, 23 verbs | COBOL-60 |
| Identification Division (four-division structure) | COBOL-61 |
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
