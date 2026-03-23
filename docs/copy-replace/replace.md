# REPLACE Statement

The REPLACE statement specifies text substitutions to be applied to all subsequent source text until the next REPLACE statement or REPLACE OFF is encountered.

**Standard:** COBOL-85, COBOL 2002, COBOL 2014, COBOL 2023

---

## Syntax

### REPLACE (Activate)

```cobol
REPLACE ==pseudo-text-1== BY ==pseudo-text-2==
    [ ==pseudo-text-3== BY ==pseudo-text-4== ] ... .
```

### REPLACE OFF (Deactivate)

```cobol
REPLACE OFF.
```

Both forms must be terminated by a period.

---

## Description

The REPLACE statement provides a source-text substitution mechanism that operates on the program source after all COPY statements have been resolved. When the compiler encounters a REPLACE statement, it applies the specified pseudo-text substitutions to all subsequent source text, continuing until the next REPLACE statement or a REPLACE OFF statement is encountered.

A REPLACE OFF statement deactivates all current replacements, causing subsequent source text to be compiled without substitution.

---

## Rules

### Scope

1. A REPLACE statement takes effect at the point where it appears and remains in effect until the next REPLACE statement or REPLACE OFF statement.
2. A REPLACE statement completely supersedes any previous REPLACE statement. The new set of replacements entirely replaces the old set; the sets are not merged.
3. REPLACE OFF terminates all active replacements.

### Placement

1. A REPLACE statement may appear anywhere a COBOL statement may appear, provided it is preceded by a period (or is at the beginning of the source).
2. A REPLACE statement must be terminated by a period.
3. A REPLACE statement must not appear within a copybook.

### Pseudo-Text Rules

1. Pseudo-text on the left (`pseudo-text-1`) must contain at least one text word.
2. Pseudo-text on the right (`pseudo-text-2`) may be empty (`== ==`), which deletes the matched text.
3. Matching proceeds word by word through the source text. When a sequence of source words matches `pseudo-text-1`, it is replaced by `pseudo-text-2`, and scanning continues after the replacement text.
4. Comparisons are made on whole text words. Partial word matches do not occur.

### Interaction with COPY REPLACING

The COPY REPLACING phrase and the REPLACE statement are applied at different stages of source text processing:

1. **First**, all COPY statements are resolved. COPY REPLACING substitutions are applied to the copied text at this stage.
2. **Then**, REPLACE substitutions are applied to the complete source text (including text that was inserted by COPY).

This two-phase processing means that REPLACE operates on text that has already undergone COPY REPLACING substitutions.

---

## Common Uses

### Prefixing Data Names

REPLACE can add a consistent prefix to data names throughout a section of code, particularly when the same copybook is used in multiple contexts.

```cobol
       REPLACE ==:PFX:== BY ==WS-CUST==.

       01  :PFX:-RECORD.
           05  :PFX:-ID         PIC X(10).
           05  :PFX:-NAME       PIC X(30).
           05  :PFX:-BALANCE    PIC S9(9)V99.

       REPLACE OFF.
```

After REPLACE processing, the data names become `WS-CUST-RECORD`, `WS-CUST-ID`, `WS-CUST-NAME`, and `WS-CUST-BALANCE`.

### Conditional Compilation (Pre-COBOL 2002)

Before COBOL 2002 introduced the `>>DEFINE` / `>>IF` compiler directives, REPLACE was sometimes used for conditional compilation by replacing code fragments with empty text.

```cobol
      *> To include debug displays:
       REPLACE ==(*DEBUG*)== BY ====.

       (*DEBUG*) DISPLAY "Entering PROCESS-RECORD".
       PERFORM PROCESS-RECORD.
       (*DEBUG*) DISPLAY "Exiting PROCESS-RECORD".

      *> To suppress debug displays:
       REPLACE ==(*DEBUG*) DISPLAY== BY ====.
```

!!! note "COBOL 2002 and later"
    COBOL 2002 introduced compiler directive statements (`>>DEFINE`, `>>IF`, `>>ELSE`, `>>END-IF`) for conditional compilation, which provide a cleaner mechanism than REPLACE for this purpose.

### Text Deletion

Replacing with empty pseudo-text effectively deletes matched text from the source.

```cobol
       REPLACE ==OPTIONAL-CLAUSE== BY == ==.
```

All occurrences of `OPTIONAL-CLAUSE` in subsequent source text are removed.

---

## Examples

### Basic REPLACE

```cobol
       REPLACE ==CUSTOMER== BY ==VENDOR==.

       01  CUSTOMER-RECORD.
           05  CUSTOMER-ID      PIC X(10).
           05  CUSTOMER-NAME    PIC X(30).

       REPLACE OFF.

       01  CUSTOMER-MASTER.
           05  CUSTOMER-KEY     PIC X(10).
```

After processing, the first record becomes `VENDOR-RECORD` with `VENDOR-ID` and `VENDOR-NAME`. The second record (after REPLACE OFF) retains its original names: `CUSTOMER-MASTER` and `CUSTOMER-KEY`.

### Multiple Replacement Pairs

```cobol
       REPLACE ==:TYPE:==   BY ==SAVINGS==
               ==:SIZE:==   BY ==50==
               ==:AMOUNT:== BY ==S9(9)V99==.

       01  :TYPE:-ACCOUNT-TABLE.
           05  :TYPE:-ENTRY  OCCURS :SIZE: TIMES.
               10  :TYPE:-ACCT-ID   PIC X(10).
               10  :TYPE:-BALANCE   PIC :AMOUNT: COMP-3.

       REPLACE OFF.
```

The result is:

```cobol
       01  SAVINGS-ACCOUNT-TABLE.
           05  SAVINGS-ENTRY  OCCURS 50 TIMES.
               10  SAVINGS-ACCT-ID   PIC X(10).
               10  SAVINGS-BALANCE   PIC S9(9)V99 COMP-3.
```

### REPLACE after COPY REPLACING

```cobol
       REPLACE ==:SECT:== BY ==INPUT==.

       COPY RECORD-LAYOUT
           REPLACING ==:PFX:== BY ==:SECT:-REC==.

       REPLACE OFF.
```

Processing order:

1. COPY inserts the text from `RECORD-LAYOUT` and applies COPY REPLACING, changing `:PFX:` to `:SECT:-REC`.
2. REPLACE then processes the resulting text, changing `:SECT:` to `INPUT`, producing names like `INPUT-REC-...`.

### Superseding a Previous REPLACE

```cobol
       REPLACE ==TAG== BY ==FIRST==.

       01  TAG-RECORD  PIC X(10).
      *>   Becomes: FIRST-RECORD

       REPLACE ==TAG== BY ==SECOND==.

       01  TAG-RECORD  PIC X(10).
      *>   Becomes: SECOND-RECORD

       REPLACE OFF.

       01  TAG-RECORD  PIC X(10).
      *>   Remains: TAG-RECORD
```

Each REPLACE statement completely replaces the previous set of substitutions.

---

## See Also

- [COPY Statement](copy.md)
- IDENTIFICATION DIVISION
- DATA DIVISION
- PROCEDURE DIVISION
