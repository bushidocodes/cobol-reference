# Debugging Techniques

Approaches for finding and fixing bugs in COBOL programs, from simple DISPLAY-based tracing to modern IDE debuggers.

---

## DISPLAY Debugging

The simplest and most universal technique — insert DISPLAY statements to trace execution flow and data values.

```cobol
       DISPLAY ">>> Entering PROCESS-RECORD"
       DISPLAY "    CUST-ID = " CUST-ID
       DISPLAY "    BALANCE = " CUST-BALANCE
       PERFORM CALCULATE-INTEREST
       DISPLAY "    INTEREST = " WS-INTEREST
       DISPLAY "<<< Exiting PROCESS-RECORD"
```

### Tips

- Use a consistent prefix (`>>>`, `DBG:`, etc.) so debug output is easy to find and remove.
- Display FILE STATUS after every I/O operation during debugging.
- Use conditional compilation to enable/disable debug output:

```cobol
>>DEFINE DEBUG-MODE AS 1

>>IF DEBUG-MODE IS DEFINED
       DISPLAY "DBG: WS-STATUS = " WS-STATUS
>>END-IF
```

Or use a runtime flag:

```cobol
       01  WS-DEBUG    PIC X VALUE "N".
           88  DEBUG-ON VALUE "Y".

       IF DEBUG-ON
           DISPLAY "DBG: WS-STATUS = " WS-STATUS
       END-IF
```

---

## FILE STATUS Checking

The most common source of COBOL bugs is unchecked I/O errors. Always define and check FILE STATUS.

```cobol
       READ CUSTOMER-FILE
       END-READ
       IF WS-CUST-STATUS NOT = "00" AND "10"
           DISPLAY "READ error: status = " WS-CUST-STATUS
                   " on CUSTOMER-FILE"
           DISPLAY "  CUST-ID was: " CUST-ID
           PERFORM ABORT-PROGRAM
       END-IF
```

See [File Status Codes](file-status-codes.md) for the complete status code reference.

---

## Common Bug Patterns

### 1. Uninitialized Data

WORKING-STORAGE items without VALUE clauses have unpredictable initial content.

```cobol
*> Bug: WS-TOTAL may contain garbage
01  WS-TOTAL    PIC 9(7)V99.
    ADD WS-AMOUNT TO WS-TOTAL

*> Fix: initialize with VALUE or INITIALIZE
01  WS-TOTAL    PIC 9(7)V99 VALUE 0.
```

### 2. Numeric Truncation

Moving a larger numeric value to a smaller field silently truncates.

```cobol
01  WS-BIG     PIC 9(7) VALUE 1234567.
01  WS-SMALL   PIC 9(3).

MOVE WS-BIG TO WS-SMALL
*> WS-SMALL = 567 (left digits silently lost!)
```

Use ON SIZE ERROR on arithmetic statements to catch overflows.

### 3. NEXT SENTENCE vs CONTINUE

```cobol
*> Bug: NEXT SENTENCE jumps past END-IF to the period
IF A > B
    NEXT SENTENCE
ELSE
    PERFORM PROCESS-B
END-IF
PERFORM ALWAYS-DO-THIS.    *> NEXT SENTENCE jumps here!

*> Fix: use CONTINUE
IF A > B
    CONTINUE
ELSE
    PERFORM PROCESS-B
END-IF
PERFORM ALWAYS-DO-THIS     *> Always executed
```

See [NEXT SENTENCE](../procedure-division/control-flow/next-sentence.md) for details.

### 4. Alphanumeric to Numeric MOVE

Moving a space-padded alphanumeric field to a numeric field causes a data exception (S0C7 on IBM mainframes).

```cobol
01  WS-INPUT   PIC X(5) VALUE "  42 ".
01  WS-NUMBER  PIC 9(5).

MOVE WS-INPUT TO WS-NUMBER    *> S0C7 abend!

*> Fix: validate first
IF WS-INPUT IS NUMERIC
    MOVE WS-INPUT TO WS-NUMBER
ELSE
    DISPLAY "Invalid numeric: [" WS-INPUT "]"
END-IF
```

### 5. Subscript Out of Range

```cobol
01  WS-TABLE.
    05  WS-ENTRY PIC X(10) OCCURS 50 TIMES.
01  WS-IDX     PIC 9(3) VALUE 0.

MOVE WS-ENTRY(WS-IDX) TO WS-OUTPUT  *> IDX=0 is invalid!
*> Subscripts are 1-based, not 0-based
```

### 6. PERFORM Fall-Through

A paragraph reached by fall-through (not PERFORM) continues to the next paragraph instead of returning.

```cobol
PARAGRAPH-A.
    PERFORM PARAGRAPH-B.
    DISPLAY "After B".
    *> Falls through to PARAGRAPH-B again!

PARAGRAPH-B.
    DISPLAY "In B".
```

Fix: ensure paragraphs end with explicit control flow or use sections.

### 7. Unchecked CALL

```cobol
CALL "MISSING-PROG" USING WS-DATA
*> If MISSING-PROG doesn't exist, program abends

*> Fix: use ON EXCEPTION
CALL "MISSING-PROG" USING WS-DATA
    ON EXCEPTION
        DISPLAY "Program not found"
END-CALL
```

---

## Compiler Debugging Options

### IBM Enterprise COBOL

| Option | Effect |
|--------|--------|
| `TEST` | Enable debugging (required for IBM Debug Tool) |
| `SSRANGE` | Runtime subscript range checking |
| `CHECK(ON)` | Numeric data validation on MOVE |
| `LIST` | Generate assembler listing |
| `OFFSET` | Condensed procedure listing with offsets |

### GnuCOBOL

| Option | Effect |
|--------|--------|
| `-fdebugging-line` | Compile D-lines as executable |
| `-debug` | Enable runtime checks |
| `-ftrace` | Trace paragraph/section entry |
| `-ftraceall` | Trace every statement |
| `-fsource-location` | Include source info in runtime errors |
| `-g` | Generate debug info for GDB |

### Micro Focus

| Option | Effect |
|--------|--------|
| `ANIM` | Enable Animator (interactive debugger) |
| `SSRANGE` | Subscript range checking |
| `CHECK` | Runtime checking |

---

## USE FOR DEBUGGING (Historical)

COBOL-74 and COBOL-85 included a Debug module with USE FOR DEBUGGING declaratives. This was marked obsolete in COBOL-85 and removed in COBOL 2002.

```cobol
*> Historical — do not use in new programs
DECLARATIVES.
DEBUG-SECTION SECTION.
    USE FOR DEBUGGING ON ALL PROCEDURES.
DEBUG-PARA.
    DISPLAY DEBUG-LINE " " DEBUG-NAME.
END DECLARATIVES.
```

Modern IDE debuggers and compiler options have completely replaced this facility.

---

## See Also

- [File Status Codes](file-status-codes.md) -- I/O status reference
- [Conditional Phrases](../language/conditional-phrases.md) -- ON SIZE ERROR, AT END, etc.
- [DECLARATIVES](../procedure-division/control-flow/declaratives.md) -- file error handling
- [Conditional Compilation](../extensions/conditional-compilation.md) -- >>IF DEBUG-MODE
- [Best Practices](best-practices.md) -- coding patterns
