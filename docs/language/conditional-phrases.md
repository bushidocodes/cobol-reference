# Conditional Phrases

Conditional phrases are optional clauses on imperative statements that specify actions to take when specific conditions arise during execution. They provide inline error handling without requiring DECLARATIVES.

- **Standard:** COBOL-68, COBOL-85, COBOL 2002, COBOL 2014, COBOL 2023

---

## ON SIZE ERROR / NOT ON SIZE ERROR

Available on all arithmetic statements (ADD, SUBTRACT, MULTIPLY, DIVIDE, COMPUTE).

```cobol
COMPUTE WS-RESULT = WS-A / WS-B
    ON SIZE ERROR
        DISPLAY "Arithmetic overflow or division by zero"
        MOVE 0 TO WS-RESULT
    NOT ON SIZE ERROR
        DISPLAY "Result: " WS-RESULT
END-COMPUTE
```

Triggered when:
- The result exceeds the capacity of the receiving field
- Division by zero occurs
- Exponentiation of zero to a negative power

When ON SIZE ERROR is triggered, the receiving field is **unchanged** (the invalid result is not stored).

---

## AT END / NOT AT END

Available on READ and RETURN statements.

```cobol
READ INPUT-FILE INTO WS-RECORD
    AT END
        SET WS-EOF TO TRUE
    NOT AT END
        ADD 1 TO WS-RECORD-COUNT
        PERFORM PROCESS-RECORD
END-READ
```

Triggered when:
- No next logical record exists (sequential read past the last record)
- File status is "10"

---

## INVALID KEY / NOT INVALID KEY

Available on READ (random), WRITE (indexed/relative), REWRITE, DELETE, and START.

```cobol
READ CUSTOMER-FILE
    KEY IS CUST-ID
    INVALID KEY
        DISPLAY "Customer not found: " CUST-ID
    NOT INVALID KEY
        PERFORM DISPLAY-CUSTOMER
END-READ

WRITE CUSTOMER-RECORD
    INVALID KEY
        DISPLAY "Duplicate key or file full"
END-WRITE
```

Triggered when:
- No record matches the specified key (READ, START, DELETE)
- A duplicate primary key exists (WRITE, REWRITE)
- File boundary exceeded (WRITE)
- Key sequence error (sequential WRITE to indexed file)

---

## AT END-OF-PAGE / NOT AT END-OF-PAGE

Available on WRITE for files with a LINAGE clause.

```cobol
WRITE REPORT-LINE AFTER ADVANCING 1 LINE
    AT END-OF-PAGE
        PERFORM WRITE-PAGE-FOOTER
        PERFORM WRITE-PAGE-HEADER
    NOT AT END-OF-PAGE
        CONTINUE
END-WRITE
```

Triggered when LINAGE-COUNTER equals or exceeds the FOOTING value after the WRITE.

`AT EOP` is an abbreviation for `AT END-OF-PAGE`.

---

## ON OVERFLOW / NOT ON OVERFLOW

Available on STRING, UNSTRING, and CALL.

### STRING/UNSTRING

```cobol
STRING WS-FIRST DELIMITED BY SPACES
       " "     DELIMITED BY SIZE
       WS-LAST DELIMITED BY SPACES
    INTO WS-FULL-NAME
    WITH POINTER WS-PTR
    ON OVERFLOW
        DISPLAY "Name too long for field"
END-STRING
```

Triggered when the receiving field is full before all sending items have been processed.

### CALL

```cobol
CALL WS-PROGRAM-NAME USING WS-DATA
    ON OVERFLOW
        DISPLAY "Program not found: " WS-PROGRAM-NAME
    NOT ON OVERFLOW
        DISPLAY "Call successful"
END-CALL
```

Triggered when the called program cannot be found. `ON EXCEPTION` / `NOT ON EXCEPTION` are synonyms for `ON OVERFLOW` / `NOT ON OVERFLOW` on the CALL statement.

---

## General Rules

1. When a conditional phrase is triggered, the associated imperative statement executes and control passes to the statement after the scope terminator.
2. When the "NOT" phrase is triggered, execution of the "NOT" imperative occurs instead.
3. If neither phrase is specified and the condition arises, behavior depends on the statement:
   - Arithmetic: result is unpredictable
   - I/O: FILE STATUS is set but no special action is taken
4. Conditional phrases convert an imperative statement into a **conditional statement** — this affects NEXT SENTENCE behavior and sentence scoping.
5. The scope terminator (END-READ, END-WRITE, etc.) is required when using conditional phrases in structured code.

---

## Pattern: Defensive I/O

```cobol
       OPEN I-O CUSTOMER-FILE
       IF WS-CUST-STATUS NOT = "00"
           DISPLAY "Open failed: " WS-CUST-STATUS
           STOP RUN
       END-IF

       MOVE WS-KEY TO CUST-ID
       READ CUSTOMER-FILE
           INVALID KEY
               DISPLAY "Not found: " CUST-ID
               PERFORM NOT-FOUND-HANDLER
           NOT INVALID KEY
               PERFORM PROCESS-CUSTOMER
       END-READ
```

---

## See Also

- [Conditions](conditions.md) -- condition types (relation, class, sign)
- [File Status Codes](../appendices/file-status-codes.md) -- I/O status reference
- [DECLARATIVES](../procedure-division/control-flow/declaratives.md) -- automatic error handling
- [Scope Terminators](scope-terminators.md) -- END-IF, END-READ, etc.
