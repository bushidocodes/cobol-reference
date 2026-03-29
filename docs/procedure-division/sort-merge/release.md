# RELEASE

The `RELEASE` statement sends a record to the sort process during a SORT INPUT PROCEDURE.

- **Standard:** COBOL-74, COBOL-85, COBOL 2002, COBOL 2014, COBOL 2023
- **Division:** Procedure Division
- **Category:** Sort/Merge

---

## Syntax

```cobol
RELEASE record-name-1 [ FROM identifier-1 ]
```

---

## Rules

1. RELEASE can only appear within a **SORT INPUT PROCEDURE**.
2. `record-name-1` must be a record described under an **SD** (Sort Description) entry.
3. `FROM identifier-1` copies data from identifier-1 to the sort record area before releasing (equivalent to MOVE identifier-1 TO record-name-1 followed by RELEASE).
4. Each RELEASE sends one record to the sort process.
5. The INPUT PROCEDURE typically reads records from an input file and RELEASEs each one, optionally filtering or transforming records before release.
6. The **SORT-RETURN** special register is set to 0 on success and 16 on failure.

---

## Examples

### Basic Input Procedure

```cobol
       SD  SORT-FILE.
       01  SORT-RECORD.
           05  SORT-KEY      PIC X(10).
           05  SORT-DATA     PIC X(70).

       PROCEDURE DIVISION.
           SORT SORT-FILE
               ON ASCENDING KEY SORT-KEY
               INPUT PROCEDURE IS READ-AND-FILTER
               OUTPUT PROCEDURE IS WRITE-SORTED.
           STOP RUN.

       READ-AND-FILTER SECTION.
           OPEN INPUT INPUT-FILE
           PERFORM UNTIL WS-EOF
               READ INPUT-FILE INTO WS-INPUT-RECORD
                   AT END SET WS-EOF TO TRUE
               END-READ
               IF NOT WS-EOF
                   IF WS-STATUS = "ACTIVE"
                       MOVE WS-INPUT-RECORD TO SORT-RECORD
                       RELEASE SORT-RECORD
                   END-IF
               END-IF
           END-PERFORM
           CLOSE INPUT-FILE.
```

### Using FROM

```cobol
       RELEASE SORT-RECORD FROM WS-INPUT-RECORD
```

---

## See Also

- [SORT](sort.md) -- sort statement with INPUT/OUTPUT PROCEDURE
- [RETURN](return.md) -- retrieving sorted records
- [MERGE](merge.md) -- merging sorted files
