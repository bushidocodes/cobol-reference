# CLOSE

The `CLOSE` statement terminates the processing of a file. It releases the connection between the program and the file, ensures that all buffered records are written, and frees system resources associated with the file.

- **Standard:** COBOL-60, COBOL-61, COBOL-61 Extended, COBOL-65, COBOL-68, COBOL-74, COBOL-85, COBOL 2002, COBOL 2014, COBOL 2023
- **Division:** Procedure Division
- **Category:** Input/Output

---

## Syntax

```cobol
CLOSE file-name-1 [WITH LOCK]
                   [WITH NO REWIND]
                   [FOR REMOVAL]
                   [REEL | UNIT [FOR REMOVAL]]
                   [REEL | UNIT [WITH NO REWIND]]
      [file-name-2 ...]
```

Multiple files may be closed in a single `CLOSE` statement:

```cobol
CLOSE CUSTOMER-FILE
      REPORT-FILE
      TRANSACTION-LOG
```

---

## Rules

### File Must Be Open

The file must be in an open state when a `CLOSE` statement is executed. Attempting to close a file that is not open results in a file status of `42`.

### Effect of CLOSE

When a `CLOSE` statement is executed:

1. Any records remaining in the output buffer are written to the file.
2. The file is disassociated from the program.
3. The file status data item is updated.
4. The file may not be referenced by any I/O statement until it is opened again.

After a successful `CLOSE`, the file may be reopened with a subsequent `OPEN` statement, potentially in a different mode.

### WITH LOCK Phrase

When `WITH LOCK` is specified, the file is closed and locked so that it cannot be reopened within the same execution of the program. A subsequent `OPEN` statement for this file results in a file status of `38`.

```cobol
CLOSE MASTER-FILE WITH LOCK
```

This is used to prevent accidental reopening of a file that should only be processed once during a program run.

!!! note "COBOL-74"
    The `WITH LOCK` phrase on `CLOSE` was introduced in COBOL-74.

### WITH NO REWIND Phrase

The `WITH NO REWIND` phrase applies only to sequential files on reel or tape media. When specified, the file is not repositioned to its beginning upon closing. The tape remains at its current position.

- Permitted only for sequential files.
- Has no effect on disk files and is typically ignored.

!!! note "Obsolete Feature"
    The `WITH NO REWIND` phrase is classified as obsolete in COBOL 2002 and later editions. It is retained for backward compatibility with tape-based systems.

### FOR REMOVAL Phrase

The `FOR REMOVAL` phrase is used with reel/unit operations on tape files. It indicates that the current reel or unit should be physically removed (dismounted) after closing.

- When specified without `REEL` or `UNIT`, the entire file is closed and the media is dismounted.
- When specified with `REEL` or `UNIT`, only the current reel or unit is dismounted.

!!! note "Obsolete Feature"
    The `FOR REMOVAL` phrase is classified as obsolete in COBOL 2002 and later editions. It was designed for tape-based environments and is rarely relevant in modern systems.

### REEL and UNIT Phrases

The `REEL` and `UNIT` phrases are used with multi-reel or multi-unit tape files:

| Phrase | Effect |
|---|---|
| `CLOSE file-name REEL` | Closes the current reel and positions to the next reel. The file remains open. |
| `CLOSE file-name UNIT` | Closes the current unit and positions to the next unit. The file remains open. |
| `CLOSE file-name REEL FOR REMOVAL` | Closes and dismounts the current reel, then positions to the next reel. |
| `CLOSE file-name UNIT WITH NO REWIND` | Closes the current unit without rewinding it. |

`REEL` and `UNIT` are interchangeable in most implementations. These phrases are valid only for sequential files.

!!! warning "Obsolete Tape Operations"
    The `REEL` and `UNIT` phrases, along with `FOR REMOVAL` and `WITH NO REWIND`, are classified as obsolete in COBOL 2002 and later editions. They remain in the standard for backward compatibility but are not expected to be used in new programs. Modern programs should omit these phrases entirely.

### Multiple Files in One Statement

A single `CLOSE` statement may close multiple files. Each file may have its own closing option. The files are closed in the order specified, from left to right.

```cobol
CLOSE INPUT-FILE WITH LOCK
      OUTPUT-FILE
      WORK-FILE
```

If an error occurs closing one file, the implementation determines whether the remaining files are still closed.

### Implicit CLOSE

If a program terminates (via `STOP RUN` or abnormal termination) without explicitly closing its files, the runtime system performs an implicit close. However, relying on implicit close is not recommended because:

- Buffered output records may not be flushed on all implementations.
- File locks may not be properly released.
- File status is not updated for the program to check.

!!! warning "Always Close Files Explicitly"
    Programs should explicitly close all open files before terminating. Relying on implicit close at `STOP RUN` may result in lost data or locked files on some implementations.

---

## FILE STATUS Values

After each `CLOSE` operation, the file status data item (if specified in the [`SELECT`](../../environment-division/select.md) clause) is updated. Common status values:

| Status | Meaning |
|---|---|
| `00` | Successful completion |
| `30` | Permanent I/O error |
| `38` | File previously closed with LOCK; cannot be reopened (returned on a subsequent OPEN, not on the CLOSE itself) |
| `42` | File not open; CLOSE attempted on a file that is not in an open state |
| `9x` | Implementation-defined errors |

---

## Examples

### Basic Open, Process, Close Pattern

```cobol
OPEN INPUT CUSTOMER-FILE
IF WS-CUST-STATUS NOT = "00"
    DISPLAY "Error opening file: " WS-CUST-STATUS
    STOP RUN
END-IF

PERFORM UNTIL WS-EOF = "Y"
    READ CUSTOMER-FILE INTO WS-CUSTOMER-RECORD
        AT END
            MOVE "Y" TO WS-EOF
        NOT AT END
            PERFORM PROCESS-CUSTOMER
    END-READ
END-PERFORM

CLOSE CUSTOMER-FILE
IF WS-CUST-STATUS NOT = "00"
    DISPLAY "Error closing file: " WS-CUST-STATUS
END-IF
```

### Closing Multiple Files

```cobol
CLOSE MASTER-FILE
      TRANSACTION-FILE
      REPORT-FILE
      ERROR-LOG

IF WS-MASTER-STATUS NOT = "00"
    DISPLAY "Error closing master file: " WS-MASTER-STATUS
END-IF
```

### CLOSE WITH LOCK to Prevent Reopening

```cobol
OPEN INPUT OLD-MASTER-FILE
OPEN OUTPUT NEW-MASTER-FILE

*> ... merge/update processing ...

CLOSE OLD-MASTER-FILE WITH LOCK
CLOSE NEW-MASTER-FILE WITH LOCK

*> OLD-MASTER-FILE and NEW-MASTER-FILE cannot be
*> reopened during this execution.
```

### File Status Check After CLOSE

```cobol
CLOSE REPORT-FILE

EVALUATE WS-RPT-STATUS
    WHEN "00"
        DISPLAY "Report file closed successfully"
    WHEN "42"
        DISPLAY "Report file was not open"
    WHEN "30"
        DISPLAY "I/O error closing report file"
        PERFORM ABORT-PROGRAM
    WHEN OTHER
        DISPLAY "Unexpected close status: " WS-RPT-STATUS
        PERFORM ABORT-PROGRAM
END-EVALUATE
```

### Closing Files in an Error-Handling Paragraph

```cobol
CLOSE-ALL-FILES.
    IF MASTER-FILE-OPEN
        CLOSE MASTER-FILE
        MOVE "N" TO MASTER-FILE-OPEN
    END-IF
    IF TRANS-FILE-OPEN
        CLOSE TRANS-FILE
        MOVE "N" TO TRANS-FILE-OPEN
    END-IF
    IF REPORT-FILE-OPEN
        CLOSE REPORT-FILE
        MOVE "N" TO REPORT-FILE-OPEN
    END-IF.
```

---

## See Also

- [OPEN](open.md) — opens a file for processing
- [READ](read.md) — reads a record from a file
- [WRITE](write.md) — writes a record to a file
- [SELECT](../../environment-division/select.md) — associates a file with an external resource
- [Procedure Division Overview](../index.md)
