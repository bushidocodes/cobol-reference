# LINAGE Clause

The `LINAGE` clause in a File Description (FD) entry specifies the logical page structure for a print file, controlling the number of lines per page, top and bottom margins, and the footing area.

- **Standard:** COBOL-74, COBOL-85, COBOL 2002, COBOL 2014, COBOL 2023
- **Division:** Data Division (File Section)

---

## Syntax

```cobol
FD  file-name
    LINAGE IS { integer-1 | data-name-1 } LINES
    [ WITH FOOTING AT { integer-2 | data-name-2 } ]
    [ LINES AT TOP { integer-3 | data-name-3 } ]
    [ LINES AT BOTTOM { integer-4 | data-name-4 } ].
```

---

## Rules

1. **LINAGE IS n LINES** specifies the number of usable lines in the page body. This is required.
2. **WITH FOOTING AT n** specifies the line number within the page body at which the footing area begins. Must be less than or equal to the LINAGE value.
3. **LINES AT TOP n** specifies the number of blank lines at the top of each page (top margin). Default is 0.
4. **LINES AT BOTTOM n** specifies the number of blank lines at the bottom of each page (bottom margin). Default is 0.
5. The total logical page size is: LINES AT TOP + LINAGE + LINES AT BOTTOM.
6. Values may be integer literals or data names, allowing dynamic page sizing at runtime.
7. When data names are used, the values are evaluated at the time of each OPEN and at each page advance.

---

## LINAGE-COUNTER Special Register

When a LINAGE clause is specified, the compiler automatically creates a special register called **LINAGE-COUNTER** for the file. This register tracks the current line position within the page body.

- LINAGE-COUNTER is set to 1 at OPEN time and after each page advance.
- LINAGE-COUNTER is incremented by the number of lines advanced on each WRITE ADVANCING.
- The programmer can read LINAGE-COUNTER but should not modify it directly.
- When LINAGE-COUNTER exceeds the FOOTING value, the END-OF-PAGE condition is raised.

---

## Page Structure

```
┌─────────────────────────┐
│   LINES AT TOP (margin) │ ← Not addressable
├─────────────────────────┤
│   Line 1                │ ← LINAGE-COUNTER = 1
│   Line 2                │
│   ...                   │
│   FOOTING line          │ ← WITH FOOTING AT n
│   ...                   │
│   Last LINAGE line      │ ← LINAGE-COUNTER = LINAGE value
├─────────────────────────┤
│   LINES AT BOTTOM       │ ← Not addressable
└─────────────────────────┘
```

---

## Examples

### Basic Page Layout

```cobol
       FD  PRINT-FILE
           LINAGE IS 60 LINES
           WITH FOOTING AT 55
           LINES AT TOP 3
           LINES AT BOTTOM 3.
       01  PRINT-RECORD    PIC X(132).
```

This defines a page with:
- 3 blank lines at the top (margin)
- 60 usable lines in the body
- Footing area starts at line 55 (last 5 body lines are for footers)
- 3 blank lines at the bottom
- Total: 66 lines per page

### Writing with Page Control

```cobol
       WRITE PRINT-RECORD AFTER ADVANCING 1 LINE
           AT END-OF-PAGE
               PERFORM WRITE-PAGE-FOOTER
               PERFORM WRITE-PAGE-HEADER
       END-WRITE
```

The `AT END-OF-PAGE` (or `AT EOP`) phrase is triggered when LINAGE-COUNTER equals or exceeds the FOOTING value after the WRITE.

### Dynamic Page Size

```cobol
       WORKING-STORAGE SECTION.
       01  WS-PAGE-SIZE    PIC 99 VALUE 60.
       01  WS-FOOTING      PIC 99 VALUE 55.

       FD  PRINT-FILE
           LINAGE IS WS-PAGE-SIZE LINES
           WITH FOOTING AT WS-FOOTING
           LINES AT TOP 2
           LINES AT BOTTOM 2.
```

### Complete Report Example

```cobol
       PROCEDURE DIVISION.
       MAIN-PARA.
           OPEN OUTPUT PRINT-FILE
           PERFORM WRITE-PAGE-HEADER
           PERFORM PROCESS-RECORDS UNTIL WS-EOF
           CLOSE PRINT-FILE
           STOP RUN.

       PROCESS-RECORDS.
           READ INPUT-FILE
               AT END SET WS-EOF TO TRUE
           END-READ
           IF NOT WS-EOF
               MOVE INPUT-DATA TO PRINT-RECORD
               WRITE PRINT-RECORD
                   AFTER ADVANCING 1 LINE
                   AT END-OF-PAGE
                       PERFORM WRITE-PAGE-FOOTER
                       PERFORM WRITE-PAGE-HEADER
               END-WRITE
           END-IF.

       WRITE-PAGE-HEADER.
           MOVE PAGE-HDR TO PRINT-RECORD
           WRITE PRINT-RECORD
               AFTER ADVANCING PAGE.

       WRITE-PAGE-FOOTER.
           MOVE SPACES TO PRINT-RECORD
           STRING "Page " LINAGE-COUNTER
               DELIMITED BY SIZE
               INTO PRINT-RECORD
           WRITE PRINT-RECORD
               AFTER ADVANCING 1 LINE.
```

---

## LINAGE vs Report Writer

| Aspect | LINAGE | Report Writer |
|--------|--------|---------------|
| Complexity | Simple, procedural | Declarative, complex |
| Page control | Manual (AT END-OF-PAGE) | Automatic |
| Control breaks | Manual coding | Automatic |
| Subtotals/totals | Manual coding | SUM clause |
| Flexibility | Full control | Constrained by framework |
| Standard | COBOL-74 | COBOL-85 (optional module) |

LINAGE is often preferred over Report Writer for its simplicity and wider compiler support.

---

## See Also

- [WRITE](../procedure-division/io/write.md) -- writing records with ADVANCING
- [Report Section](report-section.md) -- Report Writer (declarative alternative)
- [SELECT](../environment-division/select.md) -- file-control entries
- [OPEN](../procedure-division/io/open.md) -- file opening
- [Special Registers](../language/special-registers.md) -- LINAGE-COUNTER
- [File Status Codes](../appendices/file-status-codes.md) -- I/O status codes
