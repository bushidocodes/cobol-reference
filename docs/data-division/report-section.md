---
search:
  boost: 2
tags:
  - report writer
  - report section
---

# Report Section — Report Writer

The Report Section defines the layout and control structure of printed reports using the COBOL Report Writer facility. It specifies report descriptions (RD entries) and report group descriptions that define headings, detail lines, footings, and subtotals, with automatic handling of page breaks and control breaks.

**Standard:** COBOL-61 Extended, COBOL-65, COBOL-68, COBOL-74, COBOL-85, COBOL 2002, COBOL 2014 (optional), COBOL 2023

---

## Syntax

### Report Section Declaration

```cobol
DATA DIVISION.
REPORT SECTION.
RD  report-name
    [CONTROL IS    { data-name-1 [data-name-2 ...] }]
    [              { FINAL [data-name-1 ...] }       ]
    [PAGE LIMIT IS integer-1 LINE[S]
        [HEADING integer-2]
        [FIRST DETAIL integer-3]
        [LAST DETAIL integer-4]
        [FOOTING integer-5]].

{report-group-description-entry} ...
```

### Report Group Description Entry

```cobol
level-number  [data-name]
    [TYPE IS { REPORT HEADING | RH }]
    [        { PAGE HEADING   | PH }]
    [        { CONTROL HEADING | CH } { data-name | FINAL }]
    [        { DETAIL          | DE }]
    [        { CONTROL FOOTING | CF } { data-name | FINAL }]
    [        { PAGE FOOTING    | PF }]
    [        { REPORT FOOTING  | RF }]
    [LINE NUMBER IS { integer [ON NEXT PAGE] }]
    [               { PLUS integer }           ]
    [COLUMN NUMBER IS integer]
    [SOURCE IS identifier]
    [SUM identifier-1 [identifier-2 ...] [UPON detail-name [detail-name ...]]
        [RESET ON { data-name | FINAL }]]
    [VALUE IS literal]
    [GROUP INDICATE]
    [NEXT GROUP IS { integer }]
    [              { PLUS integer }]
    [              { NEXT PAGE }]
    [picture-and-other-clauses].
```

### Procedure Division Statements

```cobol
INITIATE report-name-1 [report-name-2 ...].

GENERATE { report-name | detail-report-group-name }.

TERMINATE report-name-1 [report-name-2 ...].
```

### USE BEFORE REPORTING Declarative

```cobol
DECLARATIVES.
section-name SECTION.
    USE BEFORE REPORTING report-group-name.
    [procedure-body]
END DECLARATIVES.
```

---

## Description

The Report Writer is a declarative facility that automates the production of formatted reports. The programmer defines the layout of the report in the Report Section, and the Report Writer runtime generates page headings, control-break headings and footings, subtotals, and page breaks automatically. The three Procedure Division statements -- `INITIATE`, `GENERATE`, and `TERMINATE` -- drive the entire report cycle.

!!! note "Optional Module"
    The Report Writer is classified as an optional module in COBOL 2014 and COBOL 2023. Not all compilers provide it. GnuCOBOL and Micro Focus COBOL support the Report Writer; IBM Enterprise COBOL does not.

---

## RD (Report Description) Entry

The RD entry names the report and specifies its page layout and control hierarchy. The report name must match a report name specified in a file's FD entry via the `REPORT IS` clause (in the FILE SECTION).

### PAGE LIMIT Clause

The PAGE LIMIT clause defines the vertical structure of the report page.

| Parameter | Meaning |
|-----------|---------|
| `PAGE LIMIT IS integer-1` | Total number of lines per page |
| `HEADING integer-2` | First line on which a heading may appear (default: 1) |
| `FIRST DETAIL integer-3` | First line on which a detail or control heading may appear |
| `LAST DETAIL integer-4` | Last line on which a detail or control heading may appear |
| `FOOTING integer-5` | Last line on which a control footing may appear |

The relationship `integer-2 <= integer-3 <= integer-4 <= integer-5 <= integer-1` must hold.

### CONTROL Clause

The CONTROL clause names the data items that define the control hierarchy for control breaks. Data items are listed from major to minor. The keyword `FINAL` designates the highest-level control break, which occurs only at the beginning and end of the report.

```cobol
RD  SALES-REPORT
    CONTROL IS FINAL REGION-CODE BRANCH-CODE
    PAGE LIMIT IS 60
        HEADING 1
        FIRST DETAIL 5
        LAST DETAIL 55
        FOOTING 58.
```

When the value of a control data item changes between successive `GENERATE` statements, the Report Writer automatically produces the appropriate control footings (from minor to major) and control headings (from major to minor).

---

## Report Group Types

Each report group description entry specifies a TYPE clause that determines when the Report Writer presents the group.

| Type | Abbreviation | Description |
|------|--------------|-------------|
| REPORT HEADING | RH | Produced once, at the beginning of the report |
| PAGE HEADING | PH | Produced at the top of each page |
| CONTROL HEADING | CH | Produced when a control break occurs, before detail lines |
| DETAIL | DE | Produced by each `GENERATE` statement (the main data line) |
| CONTROL FOOTING | CF | Produced when a control break occurs, after detail lines |
| PAGE FOOTING | PF | Produced at the bottom of each page |
| REPORT FOOTING | RF | Produced once, at the end of the report |

### Rules

1. A report may contain at most one RH, one RF, one PH, and one PF group.
2. A report may contain multiple DE groups, multiple CH groups, and multiple CF groups.
3. Each CH and CF group is associated with a control data item or FINAL. A report may have at most one CH and one CF for each control data item and for FINAL.
4. If a report has no DETAIL groups, the `GENERATE` statement specifies the report name, producing a **summary report**.

---

## LINE Clause

The LINE clause specifies the vertical position of a report group or a line within a report group.

```cobol
LINE NUMBER IS integer.
LINE NUMBER IS PLUS integer.
LINE NUMBER IS integer ON NEXT PAGE.
```

- An absolute line number positions the line at the specified line on the page.
- `PLUS integer` positions the line relative to the previous line.
- `ON NEXT PAGE` forces a page break before printing the line.

### Rules

1. The first entry in a report group that specifies LINE determines the starting line of the group.
2. Within a group, absolute line numbers must be in ascending order.
3. Subordinate items without a LINE clause appear on the same line as the preceding item that has a LINE clause.

---

## COLUMN Clause

The COLUMN clause specifies the horizontal position of a report item.

```cobol
COLUMN NUMBER IS integer.
```

The integer specifies the leftmost character position (1-based) for the item on the print line. If the COLUMN clause is omitted, the item is not presented (it may still participate in SUM operations).

---

## SOURCE Clause

The SOURCE clause specifies a data item or expression whose value is moved into the report item at presentation time.

```cobol
SOURCE IS identifier.
```

The source identifier is evaluated each time the Report Writer presents the group containing this item. The value is moved according to the [PICTURE clause](picture.md) of the report item.

---

## SUM Clause

The SUM clause specifies automatic accumulation (summation) of values in a CONTROL FOOTING group.

```cobol
SUM identifier-1 [identifier-2 ...]
    [UPON detail-name-1 [detail-name-2 ...]]
    [RESET ON { data-name | FINAL }].
```

### Rules

1. The SUM clause is permitted only in CONTROL FOOTING groups.
2. Each identifier is typically a SOURCE item from a DETAIL group. The Report Writer adds the value of each identifier to the SUM counter each time the associated DETAIL group is generated.
3. The `UPON` phrase restricts accumulation to specific DETAIL groups when multiple DETAIL groups exist.
4. The SUM counter is automatically reset to zero after the control footing is presented, unless the `RESET ON` phrase specifies a higher-level control item.
5. SUM counters in a higher-level CF group may reference SUM counters in a lower-level CF group, producing **rolling totals** (subtotals of subtotals).

---

## VALUE Clause in Report Groups

The VALUE clause in a report group specifies a literal to be presented. It serves a different purpose from the [VALUE clause](value.md) in WORKING-STORAGE.

```cobol
05  LINE 1  COLUMN 20  PIC X(15)  VALUE "SALES REPORT".
```

The literal is presented each time the report group is produced. The VALUE clause and the SOURCE clause are mutually exclusive on the same item.

---

## GROUP INDICATE Clause

The GROUP INDICATE clause causes the report item to be presented only on the first detail line after a control break or page break.

```cobol
05  COLUMN 1  PIC X(20)  SOURCE IS REGION-NAME
    GROUP INDICATE.
```

On subsequent detail lines within the same control group and page, the item is suppressed (spaces are printed). This avoids repetitive printing of group identifiers.

---

## NEXT GROUP Clause

The NEXT GROUP clause specifies the vertical positioning after the last line of a report group.

```cobol
NEXT GROUP IS integer.
NEXT GROUP IS PLUS integer.
NEXT GROUP IS NEXT PAGE.
```

- An absolute value specifies the line number for the next group.
- `PLUS integer` specifies the number of blank lines after the group.
- `NEXT PAGE` forces a page break after the group.

---

## Procedure Division Statements

### INITIATE Statement

```cobol
INITIATE report-name-1 [report-name-2 ...].
```

The INITIATE statement initializes the Report Writer control logic for the named report. It resets all SUM counters to zero and prepares internal line and page counters. The associated file must be opened with [OPEN](../procedure-division/io/open.md) before INITIATE is executed.

### GENERATE Statement

```cobol
GENERATE detail-report-group-name.
GENERATE report-name.
```

The GENERATE statement is the primary driver of report production. Each execution causes the Report Writer to:

1. Evaluate control data items for control breaks.
2. Produce any required control footings (minor to major).
3. Produce any required control headings (major to minor).
4. Present the specified DETAIL group (or, for a summary report with `GENERATE report-name`, accumulate SUM counters without presenting detail lines).
5. Handle page overflow automatically, producing page footings and page headings as needed.

On the first execution of GENERATE, the Report Writer also produces the REPORT HEADING and the first PAGE HEADING.

### TERMINATE Statement

```cobol
TERMINATE report-name-1 [report-name-2 ...].
```

The TERMINATE statement finalizes the report. It produces all remaining control footings (from minor to FINAL), the REPORT FOOTING, and the final PAGE FOOTING. The associated file must be closed with [CLOSE](../procedure-division/io/close.md) after TERMINATE is executed.

---

## USE BEFORE REPORTING Declarative

```cobol
DECLARATIVES.
SUPPRESS-SECTION SECTION.
    USE BEFORE REPORTING report-group-name.
    IF SUPPRESS-FLAG = "Y"
        MOVE ZERO TO PRINT-LINE-COUNT
        SUPPRESS PRINTING
    END-IF.
END DECLARATIVES.
```

The USE BEFORE REPORTING declarative is invoked automatically by the Report Writer just before the named report group is presented. It permits the program to inspect or modify report data items, or to suppress the printing of a group with the `SUPPRESS PRINTING` statement.

---

## Rules Summary

1. The Report Section must appear after the FILE SECTION, WORKING-STORAGE SECTION, and LOCAL-STORAGE SECTION in the Data Division.
2. Each RD entry must correspond to a `REPORT IS` clause in an FD entry in the FILE SECTION.
3. Report group entries use level numbers 01 through 49. Level 01 entries define report groups; subordinate entries define lines and items within a group.
4. A report group entry at level 01 must contain a TYPE clause.
5. The [PICTURE clause](picture.md) is required on elementary items that have a SOURCE, SUM, or VALUE clause. Non-presentable items (those without COLUMN) do not require PICTURE.
6. The SUM clause is permitted only in CONTROL FOOTING groups.
7. GROUP INDICATE is permitted only in DETAIL groups.
8. INITIATE must be executed before GENERATE; TERMINATE must be executed after the last GENERATE.
9. The file must be opened before INITIATE and closed after TERMINATE.

---

## Examples

### Sales Report with Control Breaks and Subtotals

```cobol
       IDENTIFICATION DIVISION.
       PROGRAM-ID. SALES-RPT.

       ENVIRONMENT DIVISION.
       INPUT-OUTPUT SECTION.
       FILE-CONTROL.
           SELECT SALES-FILE  ASSIGN TO "sales.dat"
               ORGANIZATION IS LINE SEQUENTIAL.
           SELECT REPORT-FILE ASSIGN TO "report.out"
               ORGANIZATION IS LINE SEQUENTIAL.

       DATA DIVISION.
       FILE SECTION.
       FD  SALES-FILE.
       01  SALES-RECORD.
           05  SR-REGION       PIC X(10).
           05  SR-BRANCH       PIC X(15).
           05  SR-SALESPERSON  PIC X(20).
           05  SR-AMOUNT       PIC S9(7)V99.

       FD  REPORT-FILE
           REPORT IS SALES-REPORT.

       WORKING-STORAGE SECTION.
       01  WS-EOF              PIC X VALUE "N".
           88  END-OF-FILE     VALUE "Y".

       REPORT SECTION.
       RD  SALES-REPORT
           CONTROL IS FINAL SR-REGION SR-BRANCH
           PAGE LIMIT IS 60 LINES
               HEADING 1
               FIRST DETAIL 6
               LAST DETAIL 54
               FOOTING 58.

      *> --- Report Heading (printed once) ---
       01  TYPE IS REPORT HEADING.
           05  LINE 1.
               10  COLUMN 20  PIC X(30)
                   VALUE "ANNUAL SALES REPORT".
           05  LINE 2.
               10  COLUMN 20  PIC X(30)
                   VALUE "==============================".

      *> --- Page Heading (top of each page) ---
       01  TYPE IS PAGE HEADING.
           05  LINE 4.
               10  COLUMN 1   PIC X(10)  VALUE "REGION".
               10  COLUMN 12  PIC X(15)  VALUE "BRANCH".
               10  COLUMN 28  PIC X(20)  VALUE "SALESPERSON".
               10  COLUMN 52  PIC X(12)  VALUE "AMOUNT".
           05  LINE 5.
               10  COLUMN 1   PIC X(63)  VALUE ALL "-".

      *> --- Control Heading for Region ---
       01  TYPE IS CONTROL HEADING SR-REGION.
           05  LINE PLUS 1.
               10  COLUMN 1   PIC X(10)
                   SOURCE IS SR-REGION.

      *> --- Detail Line ---
       01  SALES-DETAIL TYPE IS DETAIL.
           05  LINE PLUS 1.
               10  COLUMN 1   PIC X(10)
                   SOURCE IS SR-REGION
                   GROUP INDICATE.
               10  COLUMN 12  PIC X(15)
                   SOURCE IS SR-BRANCH
                   GROUP INDICATE.
               10  COLUMN 28  PIC X(20)
                   SOURCE IS SR-SALESPERSON.
               10  COLUMN 49  PIC $$$,$$$,$$9.99
                   SOURCE IS SR-AMOUNT.

      *> --- Control Footing for Branch (subtotal) ---
       01  TYPE IS CONTROL FOOTING SR-BRANCH.
           05  LINE PLUS 1.
               10  COLUMN 28  PIC X(20)
                   VALUE "BRANCH TOTAL:".
               10  COLUMN 49  PIC $$$,$$$,$$9.99
                   SUM SR-AMOUNT.

      *> --- Control Footing for Region (subtotal) ---
       01  TYPE IS CONTROL FOOTING SR-REGION.
           05  LINE PLUS 1.
               10  COLUMN 28  PIC X(20)
                   VALUE "REGION TOTAL:".
               10  COLUMN 49  PIC $$$,$$$,$$9.99
                   SUM SR-AMOUNT.
           05  LINE PLUS 1.
               10  COLUMN 1   PIC X(63) VALUE ALL "=".

      *> --- Control Footing FINAL (grand total) ---
       01  TYPE IS CONTROL FOOTING FINAL.
           05  LINE PLUS 2.
               10  COLUMN 28  PIC X(20)
                   VALUE "GRAND TOTAL:".
               10  COLUMN 49  PIC $$$,$$$,$$9.99
                   SUM SR-AMOUNT.

      *> --- Page Footing ---
       01  TYPE IS PAGE FOOTING.
           05  LINE 59.
               10  COLUMN 50  PIC X(10)  VALUE "Page:".
               10  COLUMN 61  PIC Z9
                   SOURCE IS PAGE-COUNTER.

      *> --- Report Footing (printed once at end) ---
       01  TYPE IS REPORT FOOTING.
           05  LINE PLUS 2.
               10  COLUMN 20  PIC X(18)
                   VALUE "*** END OF REPORT".

       PROCEDURE DIVISION.
       MAIN-LOGIC.
           OPEN INPUT SALES-FILE
                OUTPUT REPORT-FILE
           INITIATE SALES-REPORT

           READ SALES-FILE
               AT END SET END-OF-FILE TO TRUE
           END-READ

           PERFORM UNTIL END-OF-FILE
               GENERATE SALES-DETAIL
               READ SALES-FILE
                   AT END SET END-OF-FILE TO TRUE
               END-READ
           END-PERFORM

           TERMINATE SALES-REPORT
           CLOSE SALES-FILE
                 REPORT-FILE
           STOP RUN.
```

This report produces:

- A report heading on the first page.
- Page headings with column titles on each page.
- Detail lines for each sales record, with region and branch names suppressed after the first occurrence (GROUP INDICATE).
- Branch subtotals at each branch control break.
- Region subtotals at each region control break.
- A grand total at the end of the report (CONTROL FOOTING FINAL).
- Page footings with page numbers.
- A report footing on the last page.

---

## See Also

- [Data Division](index.md)
- [Level Numbers](level-numbers.md)
- [PICTURE Clause](picture.md)
- [VALUE Clause](value.md)
- [OPEN Statement](../procedure-division/io/open.md)
- [CLOSE Statement](../procedure-division/io/close.md)
- [READ Statement](../procedure-division/io/read.md)
- [WRITE Statement](../procedure-division/io/write.md)
- [PERFORM Statement](../procedure-division/control-flow/perform.md)
- [SELECT Clause](../environment-division/select.md)
