# DELETE

The `DELETE` statement removes a record from a relative or indexed file.

- **Standard:** COBOL-74, COBOL-85, COBOL 2002, COBOL 2014, COBOL 2023
- **Division:** Procedure Division
- **Category:** Input/Output

---

## Syntax

```cobol
DELETE file-name RECORD
    [ INVALID KEY imperative-statement-1 ]
    [ NOT INVALID KEY imperative-statement-2 ]
END-DELETE
```

---

## Rules

1. The file must be opened **I-O**.
2. **Sequential access mode:** The record most recently read by a successful READ is deleted. The INVALID KEY phrase must not be specified.
3. **Random or dynamic access mode:** The record identified by the current value of the file's RECORD KEY (indexed) or RELATIVE KEY (relative) is deleted. The INVALID KEY phrase should be specified.
4. `INVALID KEY` is triggered if the specified record does not exist.
5. After a successful DELETE, the record is no longer accessible in the file.
6. The file's FILE STATUS variable is updated after each DELETE.
7. For indexed files, all alternate key entries associated with the deleted record are also removed.

---

## FILE STATUS Values

| Status | Meaning |
|--------|---------|
| 00 | Successful completion |
| 23 | Record not found (INVALID KEY) |
| 41 | File not open for I-O |
| 43 | No valid previous READ (sequential mode) |

---

## Examples

### Delete by Key (Random Access)

```cobol
       MOVE "CUST-10042" TO CUST-KEY
       DELETE CUSTOMER-FILE RECORD
           INVALID KEY
               DISPLAY "Customer not found: " CUST-KEY
           NOT INVALID KEY
               DISPLAY "Customer deleted: " CUST-KEY
       END-DELETE
```

### Delete After Read (Sequential Access)

```cobol
       READ CUSTOMER-FILE
           AT END SET WS-EOF TO TRUE
       END-READ
       IF NOT WS-EOF
           IF CUST-STATUS = "INACTIVE"
               DELETE CUSTOMER-FILE RECORD
           END-IF
       END-IF
```

---

## See Also

- [READ](read.md) -- record input
- [REWRITE](rewrite.md) -- record replacement
- [WRITE](write.md) -- record output
- [START](start.md) -- file positioning
- [OPEN](open.md) -- file opening
- [File Status Codes](../../appendices/file-status-codes.md)
