# COMMIT and ROLLBACK

The `COMMIT` statement makes permanent all changes to files since the last commit point. The `ROLLBACK` statement undoes all changes since the last commit point, restoring files to their previous state.

- **Standard:** COBOL 2023
- **Division:** Procedure Division
- **Category:** Input/Output / Transaction Processing

!!! note "COBOL 2023"
    COMMIT and ROLLBACK were introduced in COBOL 2023 as part of the
    optional commit and rollback processing facility. Prior to this,
    transaction processing was handled through vendor-specific mechanisms
    (EXEC CICS, EXEC SQL COMMIT/ROLLBACK, etc.).

---

## COMMIT

```cobol
COMMIT
END-COMMIT
```

### Rules

1. COMMIT makes permanent all changes (WRITE, REWRITE, DELETE) to files since the last commit point or since the start of processing.
2. After COMMIT, changes cannot be undone by ROLLBACK.
3. COMMIT establishes a new commit point.
4. All files opened with the COMMIT phrase on their SELECT clause participate in commit/rollback processing.

---

## ROLLBACK

```cobol
ROLLBACK
END-ROLLBACK
```

### Rules

1. ROLLBACK undoes all changes to files since the last commit point.
2. After ROLLBACK, the files are restored to the state they were in at the last COMMIT (or at OPEN time if no COMMIT has been executed).
3. Record locks acquired since the last commit point are released.

---

## SELECT Clause

Files that participate in commit/rollback must include the COMMIT phrase:

```cobol
SELECT CUSTOMER-FILE
    ASSIGN TO "CUSTFILE"
    ORGANIZATION IS INDEXED
    ACCESS MODE IS DYNAMIC
    RECORD KEY IS CUST-ID
    FILE STATUS IS WS-CUST-STATUS
    LOCK MODE IS AUTOMATIC WITH ROLLBACK.
```

---

## Example

```cobol
       OPEN I-O ACCOUNT-FILE
       OPEN I-O TRANSACTION-FILE

       PERFORM PROCESS-TRANSACTIONS
           UNTIL WS-DONE

       CLOSE ACCOUNT-FILE TRANSACTION-FILE
       STOP RUN.

       PROCESS-TRANSACTIONS.
           READ TRANSACTION-FILE
               AT END SET WS-DONE TO TRUE
           END-READ
           IF NOT WS-DONE
               PERFORM APPLY-TRANSACTION
               IF WS-TXN-OK
                   COMMIT
               ELSE
                   ROLLBACK
                   DISPLAY "Transaction rolled back"
               END-IF
           END-IF.
```

---

## Pre-2023 Transaction Processing

Before COBOL 2023, transaction processing was handled through:

- **EXEC CICS SYNCPOINT** / **EXEC CICS SYNCPOINT ROLLBACK** — in CICS environments
- **EXEC SQL COMMIT** / **EXEC SQL ROLLBACK** — in embedded SQL
- **Vendor-specific file system features** — some file systems provide journaling

---

## See Also

- [OPEN](open.md) -- file opening
- [Embedded SQL](../../extensions/embedded-sql.md) -- SQL COMMIT/ROLLBACK
- [Vendor Extensions](../../extensions/vendor-extensions.md) -- CICS transaction support
