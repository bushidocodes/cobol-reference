# REWRITE

The `REWRITE` statement replaces a record in a file with new data.

- **Standard:** COBOL-68, COBOL-85, COBOL 2002, COBOL 2014, COBOL 2023
- **Division:** Procedure Division
- **Category:** Input/Output

---

## Syntax

```cobol
REWRITE record-name-1 [ FROM identifier-1 ]
    [ INVALID KEY imperative-statement-1 ]
    [ NOT INVALID KEY imperative-statement-2 ]
END-REWRITE
```

---

## Rules

1. The file must be opened **I-O**.
2. `record-name-1` must be a record defined in the file's FD entry.
3. **Sequential access mode:** The record most recently read by a successful READ is replaced. The INVALID KEY phrase must not be specified. The replacement record must be the **same length** as the original for fixed-length record files.
4. **Random or dynamic access mode (indexed/relative files):** The record identified by the current value of the RECORD KEY or RELATIVE KEY is replaced. INVALID KEY should be specified.
5. `FROM identifier-1` copies data from identifier-1 to the record area before rewriting, equivalent to `MOVE identifier-1 TO record-name-1` followed by `REWRITE record-name-1`.
6. A successful READ must precede REWRITE in sequential access mode.
7. The file's FILE STATUS variable is updated after each REWRITE.
8. For indexed files, the prime record key **must not be changed**. Alternate keys may be changed.

---

## FILE STATUS Values

| Status | Meaning |
|--------|---------|
| 00 | Successful completion |
| 21 | Key sequence error (prime key changed) |
| 22 | Duplicate alternate key |
| 41 | File not open for I-O |
| 43 | No valid previous READ (sequential mode) |
| 44 | Record length error |

---

## Examples

### Update a Record After Reading

```cobol
       READ CUSTOMER-FILE
           AT END SET WS-EOF TO TRUE
       END-READ
       IF NOT WS-EOF
           ADD WS-PAYMENT TO CUST-BALANCE
           REWRITE CUSTOMER-RECORD
       END-IF
```

### Update by Key (Random Access)

```cobol
       MOVE "CUST-10042" TO CUST-KEY
       READ CUSTOMER-FILE
           INVALID KEY
               DISPLAY "Customer not found"
       END-READ
       MOVE "INACTIVE" TO CUST-STATUS
       REWRITE CUSTOMER-RECORD
           INVALID KEY
               DISPLAY "Rewrite failed"
       END-REWRITE
```

### Using FROM

```cobol
       MOVE CORRESPONDING WS-UPDATE TO CUSTOMER-RECORD
       REWRITE CUSTOMER-RECORD FROM WS-UPDATE
           INVALID KEY
               DISPLAY "Update failed: " CUST-KEY
       END-REWRITE
```

---

## See Also

- [READ](read.md) -- record input
- [WRITE](write.md) -- record creation
- [DELETE](delete.md) -- record removal
- [START](start.md) -- file positioning
