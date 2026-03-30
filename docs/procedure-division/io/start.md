# START

The `START` statement positions within an indexed or relative file for subsequent sequential READ operations without actually reading a record.

- **Standard:** COBOL-74, COBOL-85, COBOL 2002, COBOL 2014, COBOL 2023
- **Division:** Procedure Division
- **Category:** Input/Output

---

## Syntax

```cobol
START file-name
    [ KEY IS { EQUAL TO | = |
              GREATER THAN | > |
              NOT LESS THAN | >= |
              GREATER THAN OR EQUAL TO |
              LESS THAN | < |
              NOT GREATER THAN | <= |
              LESS THAN OR EQUAL TO }
        data-name-1 ]
    [ INVALID KEY imperative-statement-1 ]
    [ NOT INVALID KEY imperative-statement-2 ]
END-START
```

---

## Rules

1. The file must be opened **INPUT** or **I-O**.
2. The access mode must be **SEQUENTIAL** or **DYNAMIC**.
3. START establishes a logical position in the file; it does **not** read a record.
4. If the KEY clause is omitted, `KEY IS EQUAL TO` the prime record key is assumed.
5. `data-name-1` must be the record key, an alternate key, or a **leftmost portion** of a key (enabling generic/partial key searches).
6. `INVALID KEY` is triggered if no record satisfies the specified condition.
7. After a successful START, the next sequential READ retrieves the record at the established position.
8. For indexed files, the comparison uses the collating sequence. For relative files, the RELATIVE KEY is compared as an integer.

---

## FILE STATUS Values

| Status | Meaning |
|--------|---------|
| 00 | Successful completion |
| 23 | No record satisfies the key condition |
| 41 | File not open |
| 46 | No valid next record (position undefined) |

---

## Examples

### Position for Sequential Reading

```cobol
       MOVE "NY" TO WS-STATE-KEY
       START CUSTOMER-FILE KEY IS EQUAL TO CUST-STATE
           INVALID KEY
               DISPLAY "No customers in state: " WS-STATE-KEY
           NOT INVALID KEY
               PERFORM READ-CUSTOMERS-IN-STATE
       END-START
```

### Generic Key (Partial Key Match)

```cobol
      *> Read all customers whose ID starts with "ACME"
       01  CUST-RECORD.
           05  CUST-ID     PIC X(10).
           05  CUST-NAME   PIC X(30).
       01  WS-PARTIAL-KEY  PIC X(4).

       MOVE "ACME" TO WS-PARTIAL-KEY
       START CUSTOMER-FILE
           KEY IS NOT LESS THAN CUST-ID
           INVALID KEY
               DISPLAY "No matching records"
       END-START

       PERFORM UNTIL WS-EOF OR CUST-ID(1:4) NOT = "ACME"
           READ CUSTOMER-FILE NEXT
               AT END SET WS-EOF TO TRUE
           END-READ
           IF NOT WS-EOF
               DISPLAY CUST-ID " " CUST-NAME
           END-IF
       END-PERFORM
```

### Position to Read in Reverse

```cobol
      *> Position to the last record <= a given key
       MOVE "ZZZZZZZZZZ" TO CUST-ID
       START CUSTOMER-FILE
           KEY IS LESS THAN OR EQUAL TO CUST-ID
       END-START
```

---

## See Also

- [READ](read.md) -- record input (sequential READ NEXT after START)
- [DELETE](delete.md) -- record removal
- [REWRITE](rewrite.md) -- record replacement
- [OPEN](open.md) -- file opening
- [File Status Codes](../../appendices/file-status-codes.md)
