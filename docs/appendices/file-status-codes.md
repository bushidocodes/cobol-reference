# File Status Codes

Every file I/O statement updates the FILE STATUS variable (if one is specified in the SELECT clause). The two-character status code indicates the result of the operation.

---

## Status Code Categories

| First Digit | Category |
|-------------|----------|
| 0 | Successful completion |
| 1 | At end condition |
| 2 | Invalid key condition |
| 3 | Permanent error |
| 4 | Logic error |
| 9 | Implementor-defined |

---

## Complete Status Code Reference

### Successful (0x)

| Status | Meaning |
|--------|---------|
| `00` | Successful completion |
| `02` | Successful; duplicate key detected (READ) or alternate key duplicate exists (WRITE/REWRITE) |
| `04` | Successful; record length does not match fixed file attributes |
| `05` | Successful; referenced optional file not present at OPEN time (file created) |
| `07` | Successful; CLOSE with NO REWIND or FOR REMOVAL on non-reel/unit medium |

### At End (1x)

| Status | Meaning |
|--------|---------|
| `10` | At end; no next logical record exists (sequential READ past last record) |
| `14` | At end; relative record number exceeds RELATIVE KEY size |

### Invalid Key (2x)

| Status | Meaning |
|--------|---------|
| `21` | Sequence error; key value not in ascending/descending order on sequential WRITE to indexed file |
| `22` | Duplicate key; attempt to WRITE or REWRITE a record with a duplicate primary key, or duplicate alternate key where duplicates are not allowed |
| `23` | Record not found; no record matches the specified key (READ, START, DELETE in random/dynamic mode) |
| `24` | Boundary violation; WRITE beyond externally defined file boundary (disk full, relative record number too large) |

### Permanent Error (3x)

| Status | Meaning |
|--------|---------|
| `30` | Permanent I/O error; no further detail available |
| `34` | Boundary violation; sequential WRITE beyond externally defined file boundaries |
| `35` | File not found; OPEN on a non-optional file that does not exist |
| `37` | OPEN mode not permitted; file does not support the requested open mode for its organization |
| `38` | OPEN attempted on a file previously closed WITH LOCK |
| `39` | File attribute conflict; the fixed file attributes (record length, key, organization) in the program do not match the actual file |

### Logic Error (4x)

| Status | Meaning |
|--------|---------|
| `41` | OPEN on a file that is already open |
| `42` | CLOSE on a file that is not open |
| `43` | DELETE or REWRITE in sequential mode without a prior successful READ |
| `44` | Record length error on REWRITE (record too large/small) or attempt to WRITE a record larger than the maximum or smaller than the minimum |
| `46` | Sequential READ attempted but no valid next record established (no prior READ or START succeeded, or AT END condition occurred on previous READ) |
| `47` | READ or START on a file not opened INPUT or I-O |
| `48` | WRITE on a file not opened OUTPUT, I-O, or EXTEND |
| `49` | DELETE or REWRITE on a file not opened I-O |

### Implementor-Defined (9x)

| Status | Meaning |
|--------|---------|
| `90`-`99` | Implementor-defined; consult your compiler documentation |
| `91` | Common extension: file not available (varies by vendor) |

---

## Using FILE STATUS

### Declaring

```cobol
       ENVIRONMENT DIVISION.
       INPUT-OUTPUT SECTION.
       FILE-CONTROL.
           SELECT CUSTOMER-FILE
               ASSIGN TO "CUSTFILE"
               ORGANIZATION IS INDEXED
               ACCESS MODE IS DYNAMIC
               RECORD KEY IS CUST-ID
               FILE STATUS IS WS-CUST-STATUS.

       DATA DIVISION.
       WORKING-STORAGE SECTION.
       01  WS-CUST-STATUS  PIC XX.
```

### Checking After I/O

```cobol
       OPEN I-O CUSTOMER-FILE
       IF WS-CUST-STATUS NOT = "00"
           DISPLAY "Open failed: " WS-CUST-STATUS
           STOP RUN
       END-IF

       READ CUSTOMER-FILE
           KEY IS CUST-ID
       END-READ
       EVALUATE WS-CUST-STATUS
           WHEN "00"  CONTINUE
           WHEN "23"  DISPLAY "Record not found"
           WHEN OTHER DISPLAY "Read error: " WS-CUST-STATUS
       END-EVALUATE
```

### Common Pattern

```cobol
       PERFORM UNTIL WS-EOF
           READ INPUT-FILE
               AT END SET WS-EOF TO TRUE
           END-READ
           EVALUATE WS-CUST-STATUS
               WHEN "00"
                   PERFORM PROCESS-RECORD
               WHEN "10"
                   CONTINUE
               WHEN OTHER
                   DISPLAY "Unexpected status: "
                       WS-CUST-STATUS
                   STOP RUN
           END-EVALUATE
       END-PERFORM
```

!!! tip
    Always define and check FILE STATUS variables. Without FILE STATUS,
    I/O errors cause the program to abend with a system message rather
    than giving you a chance to handle the error gracefully.

---

## See Also

- [SELECT](../environment-division/select.md) -- file-control entries (FILE STATUS clause)
- [OPEN](../procedure-division/io/open.md) -- opening files
- [READ](../procedure-division/io/read.md) -- reading records
- [WRITE](../procedure-division/io/write.md) -- writing records
- [DECLARATIVES and USE](../procedure-division/control-flow/declaratives.md) -- automatic error handling
