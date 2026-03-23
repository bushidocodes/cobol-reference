# WRITE

The `WRITE` statement releases a logical record to an output or input-output file. The behavior of the `WRITE` statement varies depending on the file organization (sequential, indexed, or relative) and whether advancing or positioning options are specified.

- **Standard:** COBOL-60, COBOL-61, COBOL-61 Extended, COBOL-65, COBOL-68, COBOL-74, COBOL-85, COBOL 2002, COBOL 2014, COBOL 2023
- **Division:** Procedure Division
- **Category:** Input/Output

---

## Syntax

### Format 1 — Sequential Files

```cobol
WRITE record-name-1 [FROM identifier-1]
    [BEFORE | AFTER ADVANCING
        {identifier-2 LINE | LINES}
      | {integer-1 LINE | LINES}
      | {mnemonic-name-1}
      | {PAGE}
    ]
    [AT END-OF-PAGE | EOP imperative-statement-1]
    [NOT AT END-OF-PAGE | EOP imperative-statement-2]
[END-WRITE]
```

### Format 2 — Indexed and Relative Files

```cobol
WRITE record-name-1 [FROM identifier-1]
    [INVALID KEY imperative-statement-3]
    [NOT INVALID KEY imperative-statement-4]
[END-WRITE]
```

---

## Rules

### Record Name, Not File Name

The `WRITE` statement specifies a **record name**, not a file name. The record name is defined in the `FD` entry for the file in the Data Division. This differs from `READ`, which specifies a file name.

```cobol
FD  PRINT-FILE.
01  PRINT-RECORD                   PIC X(132).

WRITE PRINT-RECORD FROM WS-DETAIL-LINE
    AFTER ADVANCING 1 LINE
END-WRITE
```

### File Must Be Open

The file associated with `record-name-1` must be open in `OUTPUT`, `I-O`, or `EXTEND` mode before a `WRITE` is executed. The permitted open modes depend on the file organization and access mode:

| Organization | Access Mode | Permitted Open Modes |
|---|---|---|
| Sequential | Sequential | `OUTPUT`, `EXTEND` |
| Indexed | Sequential | `OUTPUT` |
| Indexed | Random | `OUTPUT`, `I-O` |
| Indexed | Dynamic | `OUTPUT`, `I-O` |
| Relative | Sequential | `OUTPUT` |
| Relative | Random | `OUTPUT`, `I-O` |
| Relative | Dynamic | `OUTPUT`, `I-O` |

### FROM Phrase

When `FROM identifier-1` is specified, the contents of `identifier-1` are moved to `record-name-1` according to the rules of the [`MOVE`](../data-movement/move.md) statement, and then the record is written. This is equivalent to a `MOVE` followed by a `WRITE`.

### ADVANCING Phrase

The `ADVANCING` phrase controls vertical positioning of lines on a print file. It is valid only for sequential files.

| Phrase | Effect |
|---|---|
| `BEFORE ADVANCING` | The record is written, then the page is advanced. |
| `AFTER ADVANCING` | The page is advanced, then the record is written. |
| `ADVANCING integer-1 LINES` | Advances the specified number of lines (0 through an implementation-defined maximum). |
| `ADVANCING identifier-2 LINES` | Advances the number of lines specified by the value of `identifier-2`. |
| `ADVANCING PAGE` | Advances to the top of the next page. |
| `ADVANCING mnemonic-name-1` | Advances according to the hardware-specific function associated with `mnemonic-name-1` in the SPECIAL-NAMES paragraph. |

If neither `BEFORE` nor `AFTER` is specified, the default is `BEFORE ADVANCING 1 LINE` (implementation-defined in some compilers).

!!! note "COBOL-68"
    The `ADVANCING` phrase was formalized in COBOL-68. Earlier standards relied on implementation-defined print control characters.

### END-OF-PAGE (EOP) Phrase

The `END-OF-PAGE` phrase (or its abbreviation `EOP`) applies only to files with a `LINAGE` clause. The `END-OF-PAGE` condition arises when a `WRITE` causes printing or spacing to occur within the footing area of a page body, or when the page body is exceeded.

- If the `END-OF-PAGE` phrase is specified, `imperative-statement-1` is executed when the condition arises.
- If the `NOT END-OF-PAGE` phrase is specified, `imperative-statement-2` is executed when the condition does not arise.

### INVALID KEY Condition

The `INVALID KEY` condition applies to indexed and relative files. It arises when:

- **Indexed files (sequential access):** The value of the prime record key is not greater than that of the previously written record (records must be in ascending key order).
- **Indexed files (random or dynamic access):** A record with the same prime record key already exists and the key does not allow duplicates.
- **Relative files (random or dynamic access):** A record with the same relative record number already exists.

When the `INVALID KEY` condition occurs:

- If the `INVALID KEY` phrase is specified, `imperative-statement-3` is executed.
- If no `INVALID KEY` phrase is specified, the associated `USE AFTER EXCEPTION` declarative (if any) is executed.

### END-WRITE Scope Terminator

The `END-WRITE` explicit scope terminator delimits the scope of the `WRITE` statement. It is especially important when the `WRITE` statement is nested within another statement, such as an [`IF`](../control-flow/if.md) or [`PERFORM`](../control-flow/perform.md).

!!! note "COBOL-85"
    The `END-WRITE` scope terminator was introduced in COBOL-85.

---

## LINAGE Interaction

When a file has a `LINAGE` clause in its `FD` entry, the `WRITE` statement interacts with the logical page structure defined by that clause:

- **LINAGE IS integer-3 LINES**: Defines the number of lines in the page body.
- **WITH FOOTING AT integer-4**: Defines the line number at which the footing area begins.
- **LINES AT TOP integer-5**: Defines the number of lines in the top margin.
- **LINES AT BOTTOM integer-6**: Defines the number of lines in the bottom margin.

The special register `LINAGE-COUNTER` contains the current line number within the page body. It is automatically updated by each `WRITE` statement and is reset to 1 when a new page begins.

When `WRITE` with `ADVANCING PAGE` is executed on a file with `LINAGE`:

1. The remaining lines in the current page body are skipped.
2. The bottom margin lines are skipped.
3. The top margin lines of the next page are skipped.
4. `LINAGE-COUNTER` is set to 1.

```cobol
FD  REPORT-FILE
    LINAGE IS 60 LINES
    WITH FOOTING AT 55
    LINES AT TOP 3
    LINES AT BOTTOM 3.
01  REPORT-RECORD                  PIC X(132).

WRITE REPORT-RECORD FROM WS-DETAIL-LINE
    AFTER ADVANCING 1 LINE
    AT END-OF-PAGE
        PERFORM WRITE-PAGE-FOOTER
        PERFORM WRITE-PAGE-HEADER
    NOT AT END-OF-PAGE
        CONTINUE
END-WRITE
```

---

## FILE STATUS Values

After each `WRITE` operation, the file status data item (if specified in the [`SELECT`](../../environment-division/select.md) clause) is updated. Common status values:

| Status | Meaning |
|---|---|
| `00` | Successful completion |
| `02` | Successful write; record written with a duplicate alternate key |
| `10` | End-of-page condition (sequential file with LINAGE) |
| `21` | Sequence error — prime record key not in ascending order (indexed, sequential access) |
| `22` | Duplicate key — record with same key already exists (indexed or relative) |
| `24` | Boundary violation — attempt to write beyond file boundaries (relative file) or disk full |
| `30` | Permanent I/O error |
| `34` | Boundary violation — disk full or file size limit exceeded |
| `48` | WRITE attempted on file not opened in OUTPUT, I-O, or EXTEND mode |

!!! warning "Always Check File Status"
    Relying solely on `INVALID KEY` or `END-OF-PAGE` does not capture all possible error conditions. Programs should define a file status data item and check it after every I/O operation for robust error handling.

---

## Differences by File Organization

### Sequential Files

- Only Format 1 is permitted.
- Records are written in the order in which the `WRITE` statements are executed.
- The `ADVANCING` phrase and `END-OF-PAGE` phrase are permitted.
- The `INVALID KEY` phrase is not permitted.
- The file must be open in `OUTPUT` or `EXTEND` mode.

### Indexed Files

- Only Format 2 is permitted.
- **Sequential access:** Records must be written in ascending order of the prime record key. The `INVALID KEY` condition arises if the key value is not greater than the previously written key value.
- **Random access:** Records may be written in any order. The `INVALID KEY` condition arises if a record with a duplicate prime record key already exists.
- **Dynamic access:** Behaves the same as random access for `WRITE`.
- The `ADVANCING` phrase and `END-OF-PAGE` phrase are not permitted.

### Relative Files

- Only Format 2 is permitted.
- **Sequential access:** The runtime system assigns relative record numbers starting from 1 and incrementing by 1 for each record written. If the `RELATIVE KEY` data item is defined, it is updated with the assigned number.
- **Random access:** The program sets the `RELATIVE KEY` data item before each `WRITE`. The `INVALID KEY` condition arises if a record with that relative record number already exists.
- **Dynamic access:** Behaves the same as random access for `WRITE`.
- The `ADVANCING` phrase and `END-OF-PAGE` phrase are not permitted.

---

## Examples

### Writing to a Print File with ADVANCING

```cobol
IDENTIFICATION DIVISION.
PROGRAM-ID. PRINT-REPORT.

ENVIRONMENT DIVISION.
INPUT-OUTPUT SECTION.
FILE-CONTROL.
    SELECT PRINT-FILE ASSIGN TO "REPORT.LST"
        ORGANIZATION IS SEQUENTIAL
        FILE STATUS IS WS-PRINT-STATUS.

DATA DIVISION.
FILE SECTION.
FD  PRINT-FILE.
01  PRINT-RECORD                   PIC X(80).

WORKING-STORAGE SECTION.
01  WS-PRINT-STATUS                PIC XX.
01  WS-HEADING-LINE.
    05  FILLER         PIC X(30)   VALUE "EMPLOYEE REPORT".
    05  FILLER         PIC X(50)   VALUE SPACES.
01  WS-DETAIL-LINE.
    05  WS-EMPL-ID     PIC X(10).
    05  FILLER         PIC X(5)    VALUE SPACES.
    05  WS-EMPL-NAME   PIC X(30).
    05  FILLER         PIC X(5)    VALUE SPACES.
    05  WS-EMPL-SALARY PIC ZZZ,ZZ9.99.

PROCEDURE DIVISION.
    OPEN OUTPUT PRINT-FILE

    WRITE PRINT-RECORD FROM WS-HEADING-LINE
        AFTER ADVANCING PAGE
    END-WRITE

    WRITE PRINT-RECORD FROM WS-DETAIL-LINE
        AFTER ADVANCING 2 LINES
    END-WRITE

    CLOSE PRINT-FILE
    STOP RUN.
```

### Writing to an Indexed File

```cobol
ENVIRONMENT DIVISION.
INPUT-OUTPUT SECTION.
FILE-CONTROL.
    SELECT CUSTOMER-FILE ASSIGN TO "CUST.DAT"
        ORGANIZATION IS INDEXED
        ACCESS MODE IS RANDOM
        RECORD KEY IS CUST-ID
        ALTERNATE RECORD KEY IS CUST-NAME WITH DUPLICATES
        FILE STATUS IS WS-CUST-STATUS.

DATA DIVISION.
FILE SECTION.
FD  CUSTOMER-FILE.
01  CUSTOMER-RECORD.
    05  CUST-ID                    PIC X(10).
    05  CUST-NAME                  PIC X(30).
    05  CUST-BALANCE               PIC S9(7)V99.

WORKING-STORAGE SECTION.
01  WS-CUST-STATUS                 PIC XX.

PROCEDURE DIVISION.
    OPEN I-O CUSTOMER-FILE

    MOVE "C00012345" TO CUST-ID
    MOVE "SMITH, JOHN" TO CUST-NAME
    MOVE 1500.00 TO CUST-BALANCE

    WRITE CUSTOMER-RECORD
        INVALID KEY
            EVALUATE WS-CUST-STATUS
                WHEN "22"
                    DISPLAY "Duplicate key: " CUST-ID
                WHEN OTHER
                    DISPLAY "Write error: " WS-CUST-STATUS
            END-EVALUATE
        NOT INVALID KEY
            DISPLAY "Customer added: " CUST-ID
    END-WRITE

    CLOSE CUSTOMER-FILE
    STOP RUN.
```

### Writing to a Relative File

```cobol
ENVIRONMENT DIVISION.
INPUT-OUTPUT SECTION.
FILE-CONTROL.
    SELECT REL-FILE ASSIGN TO "REL.DAT"
        ORGANIZATION IS RELATIVE
        ACCESS MODE IS RANDOM
        RELATIVE KEY IS WS-REL-KEY
        FILE STATUS IS WS-REL-STATUS.

DATA DIVISION.
FILE SECTION.
FD  REL-FILE.
01  REL-RECORD                     PIC X(80).

WORKING-STORAGE SECTION.
01  WS-REL-KEY                     PIC 9(5).
01  WS-REL-STATUS                  PIC XX.

PROCEDURE DIVISION.
    OPEN OUTPUT REL-FILE

    MOVE 100 TO WS-REL-KEY
    MOVE "Record at slot 100" TO REL-RECORD
    WRITE REL-RECORD
        INVALID KEY
            DISPLAY "Slot occupied: " WS-REL-KEY
    END-WRITE

    CLOSE REL-FILE
    STOP RUN.
```

### WRITE with LINAGE and END-OF-PAGE

```cobol
FD  REPORT-FILE
    LINAGE IS 60 LINES
    WITH FOOTING AT 55
    LINES AT TOP 3
    LINES AT BOTTOM 3.
01  REPORT-LINE                    PIC X(132).

WORKING-STORAGE SECTION.
01  WS-PAGE-NUMBER                 PIC 9(3) VALUE 1.

PROCEDURE DIVISION.
    OPEN OUTPUT REPORT-FILE
    PERFORM WRITE-PAGE-HEADER

    PERFORM VARYING WS-INDEX FROM 1 BY 1
        UNTIL WS-INDEX > WS-TOTAL-RECORDS
        PERFORM FORMAT-DETAIL-LINE
        WRITE REPORT-LINE FROM WS-DETAIL-LINE
            AFTER ADVANCING 1 LINE
            AT END-OF-PAGE
                PERFORM WRITE-PAGE-FOOTER
                ADD 1 TO WS-PAGE-NUMBER
                PERFORM WRITE-PAGE-HEADER
        END-WRITE
    END-PERFORM

    PERFORM WRITE-PAGE-FOOTER
    CLOSE REPORT-FILE
    STOP RUN.
```

### File Status Check After WRITE

```cobol
WRITE CUSTOMER-RECORD
END-WRITE

EVALUATE WS-CUST-STATUS
    WHEN "00"
        ADD 1 TO WS-RECORDS-WRITTEN
    WHEN "22"
        DISPLAY "Duplicate key: " CUST-ID
        ADD 1 TO WS-DUPLICATES
    WHEN "24"
        DISPLAY "Boundary violation"
        PERFORM ABORT-PROGRAM
    WHEN "34"
        DISPLAY "Disk full"
        PERFORM ABORT-PROGRAM
    WHEN OTHER
        DISPLAY "Unexpected status: " WS-CUST-STATUS
        PERFORM ABORT-PROGRAM
END-EVALUATE
```

---

## See Also

- [READ](read.md) — reads a record from a file
- REWRITE — replaces an existing record in a file
- [OPEN](open.md) — opens a file for processing
- [CLOSE](close.md) — closes a file
- [MOVE](../data-movement/move.md) — moves data between data items
- [SELECT](../../environment-division/select.md) — associates a file with an external resource
- [Procedure Division Overview](../index.md)
