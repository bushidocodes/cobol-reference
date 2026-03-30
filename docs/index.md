# COBOL Reference

A comprehensive reference for the COBOL programming language (Common Business-Oriented Language), documenting syntax, semantics, and behavior across all COBOL standards from the 1960 CODASYL specification through COBOL 2023.

---

## Overview

COBOL is a compiled, English-like programming language designed for business data processing. Created in 1959 by the CODASYL committee and first published in 1960, it is one of the oldest programming languages still in widespread production use. An estimated 200+ billion lines of COBOL are in active use today, processing $3 trillion in daily commerce, powering 80% of financial services, and handling 95% of ATM transactions.

A COBOL program is organized into four **divisions**:

| Division | Purpose |
|----------|---------|
| [Identification](identification-division/index.md) | Names and identifies the program |
| [Environment](environment-division/index.md) | Describes hardware, files, and system configuration |
| [Data](data-division/index.md) | Defines all data items, records, and files |
| [Procedure](procedure-division/index.md) | Contains executable statements |

---

## Quick Reference

### Data Description

- [PICTURE clause](data-division/picture.md) — data format and editing (`PIC 9(5)V99`, `PIC X(30)`)
- [USAGE clause](data-division/usage.md) — internal representation (DISPLAY, COMP, PACKED-DECIMAL, BINARY)
- [OCCURS clause](data-division/occurs.md) — tables and arrays
- [Level Numbers](data-division/level-numbers.md) — data hierarchy (01-49, 66, 77, 88)
- [Condition Names](data-division/condition-names.md) — level-88 boolean/enum equivalents
- [FD Entry](data-division/fd-entry.md) — file description
- [Storage Sections](data-division/storage-sections.md) — WORKING-STORAGE, LOCAL-STORAGE, LINKAGE

### Statements

- **Arithmetic** — [ADD](procedure-division/arithmetic/add.md), [SUBTRACT](procedure-division/arithmetic/subtract.md), [MULTIPLY](procedure-division/arithmetic/multiply.md), [DIVIDE](procedure-division/arithmetic/divide.md), [COMPUTE](procedure-division/arithmetic/compute.md)
- **Data movement** — [MOVE](procedure-division/data-movement/move.md), [INITIALIZE](procedure-division/data-movement/initialize.md), [SET](procedure-division/data-movement/set.md), [CORRESPONDING](procedure-division/data-movement/corresponding.md)
- **Control flow** — [IF](procedure-division/control-flow/if.md), [EVALUATE](procedure-division/control-flow/evaluate.md), [PERFORM](procedure-division/control-flow/perform.md), [GOBACK](procedure-division/control-flow/goback.md), [STOP](procedure-division/control-flow/stop.md)
- **I/O** — [OPEN](procedure-division/io/open.md), [READ](procedure-division/io/read.md), [WRITE](procedure-division/io/write.md), [CLOSE](procedure-division/io/close.md), [REWRITE](procedure-division/io/rewrite.md), [DELETE](procedure-division/io/delete.md), [START](procedure-division/io/start.md)
- **String handling** — [STRING](procedure-division/string-handling/string.md), [UNSTRING](procedure-division/string-handling/unstring.md), [INSPECT](procedure-division/string-handling/inspect.md)
- **Table handling** — [SEARCH / SEARCH ALL](procedure-division/data-movement/search.md)
- **Program linkage** — [CALL](procedure-division/program-linkage/call.md), [CANCEL](procedure-division/program-linkage/cancel.md), [ENTRY](procedure-division/program-linkage/entry.md)
- **Sort/Merge** — [SORT](procedure-division/sort-merge/sort.md), [MERGE](procedure-division/sort-merge/merge.md), [RELEASE](procedure-division/sort-merge/release.md), [RETURN](procedure-division/sort-merge/return.md)
- **Error handling** — [DECLARATIVES/USE](procedure-division/control-flow/declaratives.md), [RAISE/RESUME](procedure-division/control-flow/raise-resume.md)
- **Console I/O** — [ACCEPT](procedure-division/io/accept.md), [DISPLAY](procedure-division/io/display.md)
- **Memory** — [ALLOCATE/FREE](procedure-division/memory/allocate-free.md)

### Functions

- [Numeric Functions](intrinsic-functions/numeric.md) — ABS, SQRT, MOD, MAX, MIN, MEAN, etc.
- [String Functions](intrinsic-functions/string.md) — LENGTH, UPPER-CASE, LOWER-CASE, TRIM, REVERSE, etc.
- [Date/Time Functions](intrinsic-functions/date-time.md) — CURRENT-DATE, INTEGER-OF-DATE, etc.
- [Financial Functions](intrinsic-functions/financial.md) — ANNUITY, PRESENT-VALUE
- [User-Defined Functions](intrinsic-functions/user-defined.md) — FUNCTION-ID

### Extensions

- [Embedded SQL](extensions/embedded-sql.md) — database access via EXEC SQL
- [XML Processing](extensions/xml.md) — XML PARSE / XML GENERATE
- [JSON Processing](extensions/json.md) — JSON PARSE / JSON GENERATE
- [OO COBOL](extensions/oo-cobol.md) — classes, methods, interfaces
- [Inter-Language Interop](extensions/interop.md) — C calling COBOL, COBOL calling C

---

## Coming from Another Language?

See the [Terminology Mapping](appendices/terminology.md) page for a comprehensive table mapping concepts from C, Java, Python, and Rust to their COBOL equivalents. Search for terms like "array", "struct", "switch", "for loop", "malloc", or "try/catch" to find the COBOL feature you need.

---

## Standards Coverage

This reference covers COBOL from the **1960 CODASYL specification** through **COBOL 2023** (ISO/IEC 1989:2023). The primary baseline is **COBOL-85** (ANSI X3.23-1985), the most widely implemented standard. Features from COBOL 2002, 2014, and 2023 are noted where applicable.

See [Standards](standards.md) for a detailed history spanning 60+ years and 45+ primary sources.

---

## Resources

- [Sample Programs](appendices/sample-programs.md) — complete end-to-end examples
- [Best Practices](appendices/best-practices.md) — coding patterns and conventions
- [Glossary](appendices/glossary.md) — COBOL-specific term definitions
- [File Status Codes](appendices/file-status-codes.md) — I/O status reference
- [Reserved Words](appendices/reserved-words.md) — complete reserved word list
- [References](appendices/references.md) — 45+ primary sources consulted
