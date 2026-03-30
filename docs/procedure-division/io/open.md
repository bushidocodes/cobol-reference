# OPEN

The `OPEN` statement makes a file available for processing. It establishes the connection between the program and the file, verifies that the file exists (for input), and positions the file pointer at the beginning of the file or at the end (for extend mode).

- **Standard:** COBOL-60, COBOL-61, COBOL-61 Extended, COBOL-65, COBOL-68, COBOL-74, COBOL-85, COBOL 2002, COBOL 2014, COBOL 2023
- **Division:** Procedure Division
- **Category:** Input/Output

---

## Syntax

```cobol
OPEN {INPUT  file-name-1 [WITH NO REWIND] ...} ...
     {OUTPUT file-name-2 [WITH NO REWIND] ...} ...
     {I-O    file-name-3 ...} ...
     {EXTEND file-name-4 ...} ...
    [SHARING WITH {ALL OTHER}
                  {NO OTHER}
                  {READ ONLY}]
```

Multiple files may be specified in a single `OPEN` statement, and multiple open modes may appear in the same statement:

```cobol
OPEN INPUT  CUSTOMER-FILE
             PRODUCT-FILE
     OUTPUT REPORT-FILE
     I-O    TRANS-FILE
```

---

## Rules

### Open Modes

The open mode determines which I/O operations are permitted on the file:

| Open Mode | Description |
|---|---|
| `INPUT` | The file is opened for reading only. The file must exist. |
| `OUTPUT` | The file is opened for writing only. If the file exists, its contents are erased. If it does not exist, it is created. |
| `I-O` | The file is opened for both reading and writing. The file must exist. |
| `EXTEND` | The file is opened for appending. New records are added after the last existing record. The file must exist. |

### Permitted I/O Operations by Open Mode

| Statement | INPUT | OUTPUT | I-O | EXTEND |
|---|---|---|---|---|
| [READ](read.md) | Yes | No | Yes | No |
| [WRITE](write.md) | No | Yes | No | Yes |
| REWRITE | No | No | Yes | No |
| DELETE | No | No | Yes | No |
| START | Yes | No | Yes | No |

!!! warning "Invalid I/O Operations"
    Executing an I/O statement that is not permitted for the current open mode results in a file status of `47` (READ) or `48` (WRITE/REWRITE) and causes the statement to fail. The `USE AFTER EXCEPTION` declarative, if defined, is invoked.

### I-O Mode Restrictions by File Organization

Not all file organizations support `I-O` mode:

| Organization | I-O Supported |
|---|---|
| Sequential | Yes (COBOL-85 and later) |
| Indexed | Yes |
| Relative | Yes |

!!! note "COBOL-85"
    COBOL-85 introduced support for `OPEN I-O` on sequential files, permitting the `REWRITE` statement for in-place record updates. Prior standards did not allow `I-O` mode for sequential files.

### EXTEND Mode Restrictions

The `EXTEND` mode is permitted for sequential files and, in some implementations, for indexed and relative files:

- **Sequential files:** Records are appended after the last existing record.
- **Indexed files (COBOL 2002+):** Records are written using normal indexed write rules; the file pointer is not positioned.
- **Relative files (COBOL 2002+):** Records are written to the next available relative record number.

!!! note "COBOL-85"
    The `EXTEND` mode was introduced in COBOL-85 for sequential files. COBOL 2002 extended it to indexed and relative files.

### File Must Not Already Be Open

A file must not be open when an `OPEN` statement is executed for that file. Attempting to open a file that is already open results in a file status of `41`.

### WITH NO REWIND Phrase

The `WITH NO REWIND` phrase applies only to sequential files on reel or tape media. When specified, the file is not repositioned to its beginning upon opening. This phrase is ignored for disk files.

- Permitted with `INPUT` and `OUTPUT` modes only.
- Not permitted with `I-O` or `EXTEND` modes.

!!! note "Obsolete Feature"
    The `WITH NO REWIND` phrase is classified as obsolete in COBOL 2002 and later editions. It is retained for backward compatibility with tape-based systems.

### SHARING Phrase

The `SHARING` phrase specifies the file-sharing mode for multi-user or multi-process environments.

| Phrase | Description |
|---|---|
| `SHARING WITH ALL OTHER` | Any other program may open the file in any mode. No restrictions. |
| `SHARING WITH NO OTHER` | No other program may open the file while it is open by this program. Exclusive access. |
| `SHARING WITH READ ONLY` | Other programs may open the file for `INPUT` only. They may not write to it. |

If the `SHARING` phrase is omitted, the sharing mode is implementation-defined.

!!! note "COBOL 2002"
    The `SHARING` phrase was standardized in COBOL 2002. Many compilers supported vendor-specific sharing mechanisms before this standard.

### WITH LOCK Phrase

When `WITH LOCK` is specified on `OPEN INPUT`, the file is locked so that no other program may open it. This is effectively the same as `SHARING WITH NO OTHER`.

```cobol
OPEN INPUT CUSTOMER-FILE WITH LOCK
```

!!! note "COBOL-85"
    The `WITH LOCK` phrase on `OPEN` was introduced in COBOL-85.

### Multiple Files in One Statement

A single `OPEN` statement may open multiple files, even in different modes. The files are opened in the order specified, from left to right. If an error occurs on one file, the remaining files may or may not be opened, depending on the implementation.

```cobol
OPEN INPUT  FILE-A FILE-B
     OUTPUT FILE-C
     I-O    FILE-D FILE-E
     EXTEND FILE-F
```

---

## FILE STATUS Values

After each `OPEN` operation, the file status data item (if specified in the [`SELECT`](../../environment-division/select.md) clause) is updated. Common status values:

| Status | Meaning |
|---|---|
| `00` | Successful completion |
| `05` | Successful open; file not present (optional file opened for INPUT) |
| `30` | Permanent I/O error |
| `35` | File not found (non-optional file opened for INPUT, I-O, or EXTEND) |
| `37` | File attribute conflict (file does not support the requested open mode) |
| `38` | File previously closed with LOCK; cannot be reopened |
| `39` | File attribute conflict detected (record length, key, or organization mismatch) |
| `41` | File already open |
| `9x` | Implementation-defined errors |

!!! warning "Always Check File Status After OPEN"
    Programs should verify the file status after every `OPEN` statement. A missing file, permission error, or attribute mismatch will not raise a runtime abort in all implementations; instead, subsequent I/O operations may behave unpredictably.

---

## Examples

### Basic File Open and Close

```cobol
ENVIRONMENT DIVISION.
INPUT-OUTPUT SECTION.
FILE-CONTROL.
    SELECT CUSTOMER-FILE ASSIGN TO "CUST.DAT"
        ORGANIZATION IS INDEXED
        ACCESS MODE IS DYNAMIC
        RECORD KEY IS CUST-ID
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
    IF WS-CUST-STATUS NOT = "00"
        DISPLAY "Error opening customer file: " WS-CUST-STATUS
        STOP RUN
    END-IF

    *> ... process records ...

    CLOSE CUSTOMER-FILE
    STOP RUN.
```

### Opening Multiple Files in One Statement

```cobol
OPEN INPUT  IN-FILE-1
             IN-FILE-2
     OUTPUT OUT-FILE
     EXTEND LOG-FILE
```

### OPEN with SHARING Phrase

```cobol
OPEN INPUT CUSTOMER-FILE
    SHARING WITH READ ONLY

IF WS-CUST-STATUS NOT = "00"
    DISPLAY "Unable to open file: " WS-CUST-STATUS
    STOP RUN
END-IF
```

### OPEN OUTPUT for a New Sequential File

```cobol
OPEN OUTPUT REPORT-FILE
IF WS-RPT-STATUS NOT = "00"
    DISPLAY "Cannot create report file: " WS-RPT-STATUS
    STOP RUN
END-IF

WRITE REPORT-RECORD FROM WS-HEADING
    AFTER ADVANCING PAGE
END-WRITE

CLOSE REPORT-FILE
```

### OPEN EXTEND to Append Records

```cobol
OPEN EXTEND TRANSACTION-LOG

MOVE FUNCTION CURRENT-DATE TO WS-TIMESTAMP
MOVE "Batch processing started" TO LOG-MESSAGE
WRITE LOG-RECORD FROM WS-LOG-ENTRY
END-WRITE

CLOSE TRANSACTION-LOG
```

### File Status Handling After OPEN

```cobol
OPEN INPUT MASTER-FILE

EVALUATE WS-MASTER-STATUS
    WHEN "00"
        CONTINUE
    WHEN "05"
        DISPLAY "Optional file not present; processing skipped"
        STOP RUN
    WHEN "35"
        DISPLAY "Master file not found"
        STOP RUN
    WHEN "37"
        DISPLAY "File does not support requested mode"
        STOP RUN
    WHEN "39"
        DISPLAY "File attribute mismatch"
        STOP RUN
    WHEN "41"
        DISPLAY "File is already open"
        STOP RUN
    WHEN OTHER
        DISPLAY "Unexpected open status: " WS-MASTER-STATUS
        STOP RUN
END-EVALUATE
```

---

## See Also

- [CLOSE](close.md) — closes a file
- [READ](read.md) — reads a record from a file
- [WRITE](write.md) — writes a record to a file
- [REWRITE](rewrite.md) — replaces a record
- [DELETE](delete.md) — removes a record
- [START](start.md) — positions within a file
- [SELECT](../../environment-division/select.md) — associates a file with an external resource
- [File Status Codes](../../appendices/file-status-codes.md)
