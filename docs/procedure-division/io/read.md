# READ

The `READ` statement retrieves a record from a file. The behavior of the `READ` statement varies depending on the file organization (sequential, indexed, or relative) and the access mode (sequential, random, or dynamic).

- **Standard:** COBOL-60, COBOL-61, COBOL-61 Extended, COBOL-65, COBOL-68, COBOL-74, COBOL-85, COBOL 2002, COBOL 2014, COBOL 2023
- **Division:** Procedure Division
- **Category:** Input/Output

---

## Syntax

### Format 1 — Sequential READ

```cobol
READ file-name-1 [NEXT | PREVIOUS] RECORD [INTO identifier-1]
    [WITH [NO] LOCK]
    [AT END imperative-statement-1]
    [NOT AT END imperative-statement-2]
[END-READ]
```

### Format 2 — Random READ

```cobol
READ file-name-1 RECORD [INTO identifier-1]
    [WITH [NO] LOCK]
    [KEY IS data-name-1]
    [INVALID KEY imperative-statement-3]
    [NOT INVALID KEY imperative-statement-4]
[END-READ]
```

---

## Rules

### File Must Be Open

The file must be open in `INPUT` or `I-O` mode before a `READ` is executed. A `READ` on a file that is not open or is open in `OUTPUT` or `EXTEND` mode produces undefined results.

### INTO Phrase

When `INTO identifier-1` is specified, the record is read into the file's record area and then moved to `identifier-1` according to the rules of the `MOVE` statement. The data is available in both the record area and `identifier-1`.

```cobol
READ CUSTOMER-FILE INTO WS-CUSTOMER-RECORD
    AT END MOVE "Y" TO WS-EOF
END-READ
```

### NEXT and PREVIOUS

- **NEXT**: Retrieves the next record in the logical sequence. Required when using sequential access on a file opened for dynamic access after a random `READ` or `START`, to resume sequential processing.
- **PREVIOUS**: Retrieves the previous record in the logical sequence. Available for indexed files with dynamic access.

!!! note "COBOL 2002"
    The `READ PREVIOUS` phrase was standardized in COBOL 2002. Some compilers supported it as a vendor extension in earlier versions.

### AT END Condition

The `AT END` condition arises when:

- A sequential `READ` detects the end of the file (no more records).
- For `READ PREVIOUS`, when there is no prior record.

When the `AT END` condition occurs:

- If the `AT END` phrase is specified, `imperative-statement-1` is executed.
- If no `AT END` phrase is specified, the associated `USE AFTER EXCEPTION` declarative (if any) is executed.
- The content of the record area is undefined after an `AT END` condition.

### INVALID KEY Condition

The `INVALID KEY` condition arises during a random `READ` when:

- The specified key value does not match any record in the file (indexed file).
- The relative record number does not correspond to an existing record (relative file).

When the `INVALID KEY` condition occurs:

- If the `INVALID KEY` phrase is specified, `imperative-statement-3` is executed.
- If no `INVALID KEY` phrase is specified, the associated `USE AFTER EXCEPTION` declarative (if any) is executed.

### KEY IS Phrase

For indexed files with alternate record keys, the `KEY IS` phrase specifies which key to use for the random read. If omitted, the primary record key is used.

```cobol
READ CUSTOMER-FILE
    KEY IS CUST-NAME
    INVALID KEY
        DISPLAY "Customer not found: " CUST-NAME
END-READ
```

### Lock Phrases

| Phrase | Description |
|--------|-------------|
| `WITH LOCK` | The record is locked for exclusive use by this program. Other programs cannot read or update the record until it is released. |
| `WITH NO LOCK` | The record is not locked. Other programs may access the record concurrently. |

If neither `WITH LOCK` nor `WITH NO LOCK` is specified, the locking behavior is implementation-defined.

!!! note "COBOL-85"
    The `WITH LOCK` and `WITH NO LOCK` phrases were introduced in COBOL-85 for use in multi-user environments.

---

## FILE STATUS Values

After each `READ` operation, the file status data item (if specified in the `FILE-CONTROL` paragraph) is updated. Common status values:

| Status | Meaning |
|--------|---------|
| `00` | Successful completion |
| `02` | Successful read; duplicate key exists (indexed file) |
| `04` | Record length does not match fixed file attributes |
| `10` | End of file (AT END condition) |
| `21` | Sequence error (key out of order for sequential read on indexed file) |
| `22` | Duplicate key (attempt to read with a key that has duplicates, where duplicates are not allowed) |
| `23` | Record not found (INVALID KEY condition) |
| `30` | Permanent I/O error |
| `47` | READ attempted on file not opened in INPUT or I-O mode |
| `96` | No current record defined for READ NEXT (no prior START or READ) |

!!! warning "Always Check File Status"
    Relying solely on `AT END` or `INVALID KEY` does not capture all possible error conditions. Programs should define a file status data item and check it after every I/O operation for robust error handling.

---

## Differences by File Organization

### Sequential Files

- Only Format 1 (sequential `READ`) is permitted.
- Records are read in the order they were written.
- The `NEXT` and `PREVIOUS` phrases are not used (they are implicit).
- The `KEY IS` phrase is not permitted.
- The `INVALID KEY` phrase is not permitted.

```cobol
SELECT TRANS-FILE ASSIGN TO "TRANS.DAT"
    ORGANIZATION IS SEQUENTIAL
    FILE STATUS IS WS-TRANS-STATUS.

READ TRANS-FILE INTO WS-TRANS-RECORD
    AT END
        MOVE "Y" TO WS-EOF
    NOT AT END
        ADD 1 TO WS-RECORD-COUNT
END-READ
```

### Indexed Files

- Both Format 1 and Format 2 are permitted, depending on the access mode.
- **Sequential access**: Only Format 1 is permitted. Records are read in key order.
- **Random access**: Only Format 2 is permitted. Records are read by key value.
- **Dynamic access**: Both formats are permitted. The program may intermix sequential and random reads.

```cobol
SELECT CUSTOMER-FILE ASSIGN TO "CUST.DAT"
    ORGANIZATION IS INDEXED
    ACCESS MODE IS DYNAMIC
    RECORD KEY IS CUST-ID
    ALTERNATE RECORD KEY IS CUST-NAME WITH DUPLICATES
    FILE STATUS IS WS-CUST-STATUS.

*> Random read by primary key
MOVE "C12345" TO CUST-ID
READ CUSTOMER-FILE
    INVALID KEY
        DISPLAY "Customer not found"
END-READ

*> Random read by alternate key
MOVE "SMITH" TO CUST-NAME
READ CUSTOMER-FILE
    KEY IS CUST-NAME
    INVALID KEY
        DISPLAY "Name not found"
END-READ

*> Sequential read from current position
READ CUSTOMER-FILE NEXT RECORD
    AT END
        MOVE "Y" TO WS-EOF
END-READ
```

### Relative Files

- Both Format 1 and Format 2 are permitted, depending on the access mode.
- **Sequential access**: Records are read in relative record number order.
- **Random access**: The relative key data item must be set to the desired record number before the `READ`.
- **Dynamic access**: Both sequential and random reads are permitted.

```cobol
SELECT REL-FILE ASSIGN TO "REL.DAT"
    ORGANIZATION IS RELATIVE
    ACCESS MODE IS DYNAMIC
    RELATIVE KEY IS WS-REL-KEY
    FILE STATUS IS WS-REL-STATUS.

*> Random read by relative record number
MOVE 42 TO WS-REL-KEY
READ REL-FILE
    INVALID KEY
        DISPLAY "Record 42 does not exist"
END-READ

*> Sequential read
READ REL-FILE NEXT RECORD
    AT END
        MOVE "Y" TO WS-EOF
END-READ
```

---

## Examples

### Read-Until-End-of-File Loop

```cobol
OPEN INPUT IN-FILE
MOVE "N" TO WS-EOF
PERFORM UNTIL WS-EOF = "Y"
    READ IN-FILE INTO WS-IN-RECORD
        AT END
            MOVE "Y" TO WS-EOF
        NOT AT END
            PERFORM PROCESS-RECORD
    END-READ
END-PERFORM
CLOSE IN-FILE
```

### Priming Read Pattern

A common pattern reads the first record before entering the loop:

```cobol
OPEN INPUT IN-FILE
READ IN-FILE INTO WS-RECORD
    AT END MOVE "Y" TO WS-EOF
END-READ
PERFORM UNTIL WS-EOF = "Y"
    PERFORM PROCESS-RECORD
    READ IN-FILE INTO WS-RECORD
        AT END MOVE "Y" TO WS-EOF
    END-READ
END-PERFORM
CLOSE IN-FILE
```

### Random Read on Indexed File with Error Handling

```cobol
MOVE WS-SEARCH-ID TO CUST-ID
READ CUSTOMER-FILE INTO WS-CUST-RECORD
    INVALID KEY
        EVALUATE WS-CUST-STATUS
            WHEN "23"
                DISPLAY "Record not found for ID: " WS-SEARCH-ID
            WHEN OTHER
                DISPLAY "Unexpected error: " WS-CUST-STATUS
        END-EVALUATE
    NOT INVALID KEY
        PERFORM DISPLAY-CUSTOMER
END-READ
```

### Sequential Browse on Indexed File (Dynamic Access)

```cobol
*> Position to the first record with key >= "M"
MOVE "M" TO CUST-ID
START CUSTOMER-FILE KEY IS >= CUST-ID
    INVALID KEY
        DISPLAY "No records from M onward"
        MOVE "Y" TO WS-EOF
END-START

PERFORM UNTIL WS-EOF = "Y"
    READ CUSTOMER-FILE NEXT RECORD
        AT END
            MOVE "Y" TO WS-EOF
        NOT AT END
            DISPLAY CUST-ID " " CUST-NAME
    END-READ
END-PERFORM
```

### READ with LOCK

```cobol
READ CUSTOMER-FILE INTO WS-CUST-RECORD
    WITH LOCK
    INVALID KEY
        DISPLAY "Record not found"
    NOT INVALID KEY
        PERFORM UPDATE-CUSTOMER
        REWRITE CUST-RECORD FROM WS-CUST-RECORD
END-READ
```

### File Status Check After READ

```cobol
READ IN-FILE INTO WS-RECORD
END-READ

EVALUATE WS-FILE-STATUS
    WHEN "00"
        PERFORM PROCESS-RECORD
    WHEN "10"
        MOVE "Y" TO WS-EOF
    WHEN "30"
        DISPLAY "Permanent I/O error on IN-FILE"
        PERFORM ABORT-PROGRAM
    WHEN OTHER
        DISPLAY "Unexpected status: " WS-FILE-STATUS
        PERFORM ABORT-PROGRAM
END-EVALUATE
```

---

## See Also

- WRITE — writes a record to a file
- REWRITE — replaces an existing record
- START — positions within an indexed or relative file
- OPEN — opens a file for processing
- CLOSE — closes a file
- [Procedure Division Overview](../index.md)
