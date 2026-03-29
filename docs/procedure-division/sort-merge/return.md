# RETURN

The `RETURN` statement retrieves the next sorted record during a SORT or MERGE OUTPUT PROCEDURE.

- **Standard:** COBOL-74, COBOL-85, COBOL 2002, COBOL 2014, COBOL 2023
- **Division:** Procedure Division
- **Category:** Sort/Merge

---

## Syntax

```cobol
RETURN file-name-1 [ INTO identifier-1 ]
    AT END imperative-statement-1
    [ NOT AT END imperative-statement-2 ]
END-RETURN
```

---

## Rules

1. RETURN can only appear within a **SORT or MERGE OUTPUT PROCEDURE**.
2. `file-name-1` must be a file described under an **SD** (Sort Description) entry.
3. Records are returned in sorted (or merged) order.
4. `INTO identifier-1` copies the returned record into identifier-1 (like READ INTO).
5. `AT END` is executed when no more sorted records remain.
6. The **SORT-RETURN** special register is set to 0 on success and 16 on failure.
7. Each RETURN retrieves exactly one record.

---

## Examples

### Basic Output Procedure

```cobol
       WRITE-SORTED SECTION.
           OPEN OUTPUT OUTPUT-FILE
           PERFORM UNTIL WS-SORT-DONE
               RETURN SORT-FILE INTO WS-OUTPUT-RECORD
                   AT END
                       SET WS-SORT-DONE TO TRUE
                   NOT AT END
                       WRITE OUTPUT-RECORD
                           FROM WS-OUTPUT-RECORD
               END-RETURN
           END-PERFORM
           CLOSE OUTPUT-FILE.
```

### Complete Sort with Input and Output Procedures

```cobol
       SORT SORT-FILE
           ON ASCENDING KEY SORT-CUST-NAME
           INPUT PROCEDURE IS FILTER-RECORDS
           OUTPUT PROCEDURE IS FORMAT-OUTPUT.

       FILTER-RECORDS SECTION.
           OPEN INPUT RAW-FILE
           PERFORM UNTIL WS-EOF
               READ RAW-FILE INTO WS-RAW
                   AT END SET WS-EOF TO TRUE
               END-READ
               IF NOT WS-EOF AND WS-RAW-STATUS = "A"
                   RELEASE SORT-RECORD FROM WS-RAW
               END-IF
           END-PERFORM
           CLOSE RAW-FILE.

       FORMAT-OUTPUT SECTION.
           OPEN OUTPUT REPORT-FILE
           MOVE 0 TO WS-COUNT
           PERFORM UNTIL WS-SORT-DONE
               RETURN SORT-FILE INTO WS-SORTED
                   AT END SET WS-SORT-DONE TO TRUE
               END-RETURN
               IF NOT WS-SORT-DONE
                   ADD 1 TO WS-COUNT
                   MOVE WS-SORTED TO REPORT-RECORD
                   WRITE REPORT-RECORD
               END-IF
           END-PERFORM
           DISPLAY "Records written: " WS-COUNT
           CLOSE REPORT-FILE.
```

---

## See Also

- [SORT](sort.md) -- sort statement with INPUT/OUTPUT PROCEDURE
- [MERGE](merge.md) -- merging sorted files
- [RELEASE](release.md) -- sending records to the sort process
