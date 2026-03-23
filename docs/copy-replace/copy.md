# COPY Statement

The COPY statement includes the content of a library text (copybook) into the source program at the point where the COPY statement appears, optionally replacing specified text within the included content.

**Standard:** COBOL-65, COBOL-68, COBOL-74, COBOL-85, COBOL 2002, COBOL 2014, COBOL 2023

---

## Syntax

```cobol
COPY text-name [ { OF | IN } library-name ]
    [ REPLACING { operand-1 BY operand-2 } ... ]
    .
```

Where each `operand` is one of:

- `==pseudo-text==`
- `identifier`
- `literal`
- `word`

The COPY statement must be terminated by a period, which also terminates the statement in which COPY appears.

---

## Description

The COPY statement directs the compiler to retrieve the text named by `text-name` from a library and to logically insert that text into the source program, replacing the entire COPY statement. The resulting source text is then compiled as though it had been written directly in the source program.

### Copybook Concept

A **copybook** (also called **library text** or **copy member**) is a file containing COBOL source text that is intended to be included in one or more programs via the COPY statement. Copybooks promote consistency and reduce duplication by maintaining shared record layouts, condition names, paragraph definitions, and other code fragments in a single location.

### Library Name

The optional `OF` or `IN` phrase specifies the library from which the text is retrieved. If omitted, the compiler searches a default library or set of library paths, which is implementation-defined.

```cobol
       COPY CUSTOMER-RECORD OF COPYLIB.
       COPY CUSTOMER-RECORD IN COPYLIB.
```

Both forms are equivalent. The mapping of library names to physical file locations is implementation-defined.

---

## REPLACING Phrase

The REPLACING phrase specifies text substitutions to be applied to the copied text before it is compiled.

```cobol
COPY record-layout
    REPLACING ==:PREFIX:== BY ==WS==
              ==:TAG:==    BY ==CUSTOMER==.
```

### Operand Types

| Operand Type | Syntax | Description |
|-------------|--------|-------------|
| Pseudo-text | `==text==` | A sequence of text words, literals, and separators bounded by `==` delimiters. May be empty on the replacement side (`== ==`). |
| Identifier | `data-name` | A data name or identifier |
| Literal | `"string"` or `123` | An alphanumeric or numeric literal |
| Word | `COBOL-WORD` | A single COBOL word |

### Pseudo-Text Rules

1. Pseudo-text on the left (`operand-1`) must contain at least one text word. It may not consist entirely of separator commas or separator semicolons.
2. Pseudo-text on the right (`operand-2`) may be empty (`== ==`), which effectively deletes the matched text.
3. Matching is performed on a word-by-word basis. The compiler scans the copied text from left to right and replaces each occurrence of `operand-1` with `operand-2`.
4. Each text word in the pseudo-text operand is compared as a whole word; partial word matches do not occur except within pseudo-text delimiters.

### REPLACING Examples

```cobol
      *> Copybook: ACCT-RECORD.cpy
      *>    05  :PFX:-ACCOUNT-ID    PIC X(10).
      *>    05  :PFX:-ACCOUNT-NAME  PIC X(30).
      *>    05  :PFX:-BALANCE       PIC S9(9)V99 COMP-3.

       COPY ACCT-RECORD
           REPLACING ==:PFX:== BY ==WS==.

      *> Result after COPY:
      *>    05  WS-ACCOUNT-ID    PIC X(10).
      *>    05  WS-ACCOUNT-NAME  PIC X(30).
      *>    05  WS-BALANCE       PIC S9(9)V99 COMP-3.
```

---

## Rules

### Placement

1. A COPY statement may appear anywhere in a COBOL source program where a word or separator may appear, in any division.
2. A COPY statement must be preceded by a space and terminated by a period.
3. The period that terminates the COPY statement also terminates any sentence or entry in which it appears.

### Processing Order

1. COPY statements are processed before any other compilation activity. The compiler resolves all COPY statements first, producing a complete source text that is then compiled.
2. If the copied text itself contains COPY statements (nested COPY), those are resolved recursively.

### Nested COPY

1. A copybook may contain COPY statements that reference other copybooks.
2. A copybook must not directly or indirectly copy itself (recursive COPY is not permitted).

!!! note "COBOL-85"
    COBOL-85 formally defined the nested COPY capability. Some earlier implementations supported it as an extension.

### Multiple REPLACING Pairs

Multiple REPLACING pairs are applied independently. Each pair specifies its own operand-1 and operand-2. The compiler scans the copied text from left to right; when a match is found for any operand-1, the corresponding operand-2 is substituted and scanning continues after the substituted text.

```cobol
       COPY TRANSACTION-LAYOUT
           REPLACING ==:PFX:==   BY ==IN==
                     ==COMP-3==  BY ==COMP==
                     ==PIC X==   BY ==PIC A==.
```

---

## Common Patterns

### Record Layouts

The most common use of COPY is to share record layouts across programs.

```cobol
       FD  CUSTOMER-FILE.
       01  CUSTOMER-RECORD.
           COPY CUSTOMER-FIELDS.
```

### Prefixed Copybooks

Copybooks that use a replaceable prefix allow the same layout to be copied multiple times within a single program under different names.

```cobol
       01  WS-INPUT-CUSTOMER.
           COPY CUSTOMER-FIELDS
               REPLACING ==:PFX:== BY ==WS-IN==.

       01  WS-OUTPUT-CUSTOMER.
           COPY CUSTOMER-FIELDS
               REPLACING ==:PFX:== BY ==WS-OUT==.
```

### Paragraph Includes

COPY may be used in the Procedure Division to include shared paragraph or section definitions.

```cobol
       PERFORM VALIDATE-CUSTOMER.

       COPY VALIDATION-PARAGRAPHS.
```

### Condition Name Definitions

Shared sets of condition names ensure consistent validation across programs.

```cobol
       01  WS-COUNTRY-CODE      PIC X(3).
           COPY COUNTRY-88-VALUES.
```

---

## Compiler Library Search Paths

The mechanism by which the compiler locates copybooks is implementation-defined. Common approaches include:

- A compiler option specifying one or more directories (e.g., `-I` or `--copypath`)
- An environment variable listing library directories (e.g., `COBCPY` for GnuCOBOL)
- A fixed library naming convention (e.g., PDS members on z/OS)

The file extension of copybooks is also implementation-defined. Common extensions include `.cpy`, `.cbl`, `.copy`, and `.CPY`.

---

## Examples

### Basic COPY

```cobol
       IDENTIFICATION DIVISION.
       PROGRAM-ID. REPORT-PROGRAM.

       DATA DIVISION.
       WORKING-STORAGE SECTION.
       01  WS-EMPLOYEE.
           COPY EMPLOYEE-RECORD.

       PROCEDURE DIVISION.
           DISPLAY WS-EMPLOYEE.
           STOP RUN.
```

### COPY with REPLACING

```cobol
       DATA DIVISION.
       WORKING-STORAGE SECTION.
       01  WS-OLD-ADDRESS.
           COPY ADDRESS-FIELDS
               REPLACING ==:TAG:== BY ==OLD==.

       01  WS-NEW-ADDRESS.
           COPY ADDRESS-FIELDS
               REPLACING ==:TAG:== BY ==NEW==.
```

If `ADDRESS-FIELDS` contains:

```cobol
           05  :TAG:-STREET   PIC X(30).
           05  :TAG:-CITY     PIC X(20).
           05  :TAG:-STATE    PIC X(2).
           05  :TAG:-ZIP      PIC X(10).
```

Then the result is two distinct sets of fields: `OLD-STREET`, `OLD-CITY`, etc. and `NEW-STREET`, `NEW-CITY`, etc.

### COPY Across Divisions

```cobol
       ENVIRONMENT DIVISION.
       INPUT-OUTPUT SECTION.
       FILE-CONTROL.
           COPY FILE-CONTROL-ENTRIES.

       DATA DIVISION.
       FILE SECTION.
           COPY FILE-DESCRIPTIONS.

       WORKING-STORAGE SECTION.
           COPY WORKING-FIELDS.

       PROCEDURE DIVISION.
           COPY MAINLINE-LOGIC.
```

---

## See Also

- [REPLACE Statement](replace.md)
- IDENTIFICATION DIVISION
- DATA DIVISION
- PROCEDURE DIVISION
