# Data Division

The Data Division describes all data items used by a COBOL program.

**Standard:** COBOL-60, COBOL-61, COBOL-61 Extended, COBOL-65, COBOL-68, COBOL-85, COBOL 2002, COBOL 2014, COBOL 2023

---

## Description

The Data Division is the third division of a COBOL source program. It defines the structure, type, and size of every data item that the program creates, receives, or manipulates. All storage allocation for a program is determined by the entries in this division.

The Data Division header is:

```cobol
DATA DIVISION.
```

If no data items are needed, the entire division may be omitted.

!!! note "Historical Note: COBOL-60 and COBOL-61"
    The Data Division was one of the original three divisions defined in the 1960 COBOL report, where it was described as "the second division of the problem definition" (the Procedure Division being considered the primary one). In COBOL-60, data fell into three categories: file data, intermediate/working-storage data, and constants/figurative constants. The division originally contained three sections: **FILE**, **WORKING-STORAGE**, and **CONSTANT**. The CONSTANT Section was a dedicated section for defining named constants and figurative constants; its concepts were later absorbed into the `VALUE` clause and the language's built-in figurative constants (such as `ZERO`, `SPACE`, and `HIGH-VALUE`), and the section itself was removed in subsequent standards. Even in 1960, records were described using the hierarchical level-number system and the `PICTURE` clause. File Description and Record Description entries could also be stored on a COBOL library tape for reuse across programs, an early precursor to the `COPY` statement.

    In **COBOL-61**, the Data Division still had these same three sections (FILE, WORKING-STORAGE, CONSTANT) with no LINKAGE Section yet — parameter passing between programs was not standardized. Record descriptions could use separate **SIZE**, **CLASS**, **POINT LOCATION**, and **SIGNED** clauses rather than a unified PICTURE clause (though PICTURE was also available as an alternative). The **COPY** clause could copy record descriptions from elsewhere in the Data Division or from a COBOL library. **Level-77** was used for independent (non-grouped) items in both the Working-Storage and Constant sections. **TALLY** was a special register — a system-defined counter used with the **EXAMINE** verb (the predecessor to INSPECT, introduced in COBOL-74).

## Sections

The Data Division contains up to six sections. Each section, if present, must appear in the order shown below.

### FILE SECTION

```cobol
FILE SECTION.
```

The File Section describes the structure of records associated with each file defined in the Environment Division's FILE-CONTROL paragraph. Each file described in a `SELECT` statement must have a corresponding `FD` (File Description) or `SD` (Sort/Merge Description) entry in this section.

- Contains `FD` and `SD` entries followed by record descriptions.
- Record descriptions define the layout of data read from or written to files.
- A file may have multiple record descriptions; they implicitly share the same storage (implicit redefinition).

See: FD entry, SD entry

### WORKING-STORAGE SECTION

```cobol
WORKING-STORAGE SECTION.
```

The Working-Storage Section defines data items that are allocated when the program is loaded and persist for the duration of the program's execution (or the run unit, for nested programs with the `GLOBAL` clause).

- Data items may be initialized with `VALUE` clauses; initialization occurs once, at program load time.
- In a called subprogram, items retain their last-used values across `CALL` invocations unless the subprogram is canceled or the `INITIAL` clause is specified on the `PROGRAM-ID`.
- Both group items (level 01) and independent elementary items (level 77) may appear.

See: [Level Numbers](level-numbers.md), VALUE clause

### LOCAL-STORAGE SECTION

```cobol
LOCAL-STORAGE SECTION.
```

The Local-Storage Section defines data items that are allocated fresh each time the program is invoked and deallocated when the program returns control to its caller.

- `VALUE` clauses are applied on every invocation, not just the first.
- Primarily useful in recursive programs and programs invoked multiple times where clean initial state is required.
- Introduced in COBOL 2002.

**Standard:** COBOL 2002 and later.

### LINKAGE SECTION

```cobol
LINKAGE SECTION.
```

The Linkage Section describes data items whose storage is provided by a calling program or by the operating environment. The program does not allocate storage for these items; it receives addresses at runtime via the `CALL ... USING` or `PROCEDURE DIVISION USING` mechanism.

- Items defined here are accessible only when the program is entered with the corresponding arguments.
- No `VALUE` clauses are permitted except on level-88 condition-name entries.
- The section is required in any subprogram that receives parameters.

See: CALL statement, PROCEDURE DIVISION USING

### REPORT SECTION

```cobol
REPORT SECTION.
```

The Report Section defines the layout and control logic for reports produced by the Report Writer module. Each report is described by an `RD` (Report Description) entry followed by report group descriptions.

- Control breaks, page headings, footings, and detail lines are defined declaratively.
- The Report Writer generates output via `INITIATE`, `GENERATE`, and `TERMINATE` statements.
- This section is only present in programs that use the Report Writer feature.

**Standard:** Defined in COBOL-85; classified as optional in COBOL 2014.

See: RD entry, GENERATE statement

### SCREEN SECTION

```cobol
SCREEN SECTION.
```

The Screen Section defines screen layouts for terminal-based input and output. Screen items describe fields, literals, attributes (color, underlining, reverse video), and their positions on a display.

- Used with the `ACCEPT` and `DISPLAY` statements for screen I/O.
- Provides a portable way to define interactive terminal screens.
- Not part of the ANSI COBOL-85 standard; adopted in COBOL 2002.

**Standard:** COBOL 2002 and later. Widely supported as a vendor extension prior to standardization.

See: ACCEPT statement, DISPLAY statement

## Rules

1. The Data Division header (`DATA DIVISION.`) must precede all sections within the division.
2. Sections, if present, must appear in the order listed above: FILE, WORKING-STORAGE, LOCAL-STORAGE, LINKAGE, REPORT, SCREEN.
3. Any section may be omitted if it is not needed.
4. Data description entries within each section must conform to the rules for [level numbers](level-numbers.md).
5. Each data item must have a unique name within its scope, or must be qualified by group names to be made unique.

## Example

```cobol
       DATA DIVISION.
       FILE SECTION.
       FD  CUSTOMER-FILE.
       01  CUSTOMER-RECORD.
           05  CUST-ID            PIC 9(8).
           05  CUST-NAME          PIC X(30).
           05  CUST-BALANCE       PIC S9(7)V99.

       WORKING-STORAGE SECTION.
       77  WS-EOF-FLAG            PIC X       VALUE 'N'.
           88  WS-EOF             VALUE 'Y'.
       01  WS-COUNTERS.
           05  WS-READ-COUNT     PIC 9(5)    VALUE ZERO.
           05  WS-ERROR-COUNT    PIC 9(5)    VALUE ZERO.

       LOCAL-STORAGE SECTION.
       01  LS-TEMP-AREA           PIC X(100).

       LINKAGE SECTION.
       01  LK-RETURN-CODE         PIC S9(4) COMP.
```

## See Also

- [Level Numbers](level-numbers.md)
- [PICTURE Clause](picture.md)
- [USAGE Clause](usage.md)
- [OCCURS Clause](occurs.md)
- [Condition Names (Level 88)](condition-names.md)
- [VALUE Clause](value.md)
- [REDEFINES Clause](redefines.md)
