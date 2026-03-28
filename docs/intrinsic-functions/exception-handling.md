# Exception Handling Functions

Intrinsic functions that return information about the most recent exception condition. These functions provide a standard mechanism for programs to query exception details without relying on vendor-specific error handling.

- **Standard:** COBOL 2002

---

## General Notes

The exception handling functions return information about the **last exception condition** that was raised during program execution. An exception condition is set by statements such as file I/O operations, arithmetic overflow, pointer operations, and other runtime events.

Exception information is cleared when the next statement that can raise an exception begins execution. Programs should query exception details immediately after the statement that raised the condition.

---

## EXCEPTION-FILE

Returns the name of the file associated with the most recent file I/O exception.

```cobol
FUNCTION EXCEPTION-FILE
```

- **Arguments** -- none.
- **Return category** -- alphanumeric.
- **Behavior** -- returns the name of the file that was being processed when the most recent file-related exception condition occurred. If no file exception is active, the result is spaces.

```cobol
       READ CUSTOMER-FILE INTO WS-RECORD
           AT END
               CONTINUE
           NOT AT END
               PERFORM PROCESS-RECORD
       END-READ
       IF FUNCTION EXCEPTION-STATUS NOT = SPACES
           DISPLAY "File error on: "
                   FUNCTION EXCEPTION-FILE
           DISPLAY "Status: "
                   FUNCTION EXCEPTION-STATUS
       END-IF
```

---

## EXCEPTION-LOCATION

Returns the location in the source program where the most recent exception occurred.

```cobol
FUNCTION EXCEPTION-LOCATION
```

- **Arguments** -- none.
- **Return category** -- alphanumeric.
- **Behavior** -- returns a string identifying the location of the most recent exception. The format of the location string is implementation-defined but typically includes the program name and paragraph or section name. If no exception is active, the result is spaces.

```cobol
       IF FUNCTION EXCEPTION-STATUS NOT = SPACES
           DISPLAY "Exception at: "
                   FUNCTION EXCEPTION-LOCATION
       END-IF
```

---

## EXCEPTION-STATEMENT

Returns the name of the statement that caused the most recent exception.

```cobol
FUNCTION EXCEPTION-STATEMENT
```

- **Arguments** -- none.
- **Return category** -- alphanumeric.
- **Behavior** -- returns a string identifying the COBOL statement that raised the most recent exception condition (e.g., "READ", "WRITE", "COMPUTE"). If no exception is active, the result is spaces.

```cobol
       IF FUNCTION EXCEPTION-STATUS NOT = SPACES
           DISPLAY "Exception in statement: "
                   FUNCTION EXCEPTION-STATEMENT
       END-IF
```

---

## EXCEPTION-STATUS

Returns the exception condition name for the most recent exception.

```cobol
FUNCTION EXCEPTION-STATUS
```

- **Arguments** -- none.
- **Return category** -- alphanumeric.
- **Behavior** -- returns the name of the most recent exception condition. Exception condition names follow a hierarchical naming scheme defined by the standard (e.g., `EC-I-O`, `EC-I-O-PERMANENT-ERROR`, `EC-SIZE-OVERFLOW`). If no exception is active, the result is spaces.

### Standard Exception Condition Names

Exception names use a hierarchical dot-separated scheme. The top-level categories are:

| Prefix | Category | Description |
|--------|----------|-------------|
| `EC-ARGUMENT` | Argument | Invalid function argument |
| `EC-BOUND` | Bounds | Subscript or reference out of bounds |
| `EC-DATA` | Data | Data conversion or incompatibility |
| `EC-FLOW` | Flow | Control flow exception (e.g., missing GO TO target) |
| `EC-I-O` | Input/Output | File I/O exceptions |
| `EC-IMP` | Implementation | Implementation-defined exceptions |
| `EC-OO` | Object-Oriented | OO COBOL exceptions |
| `EC-ORDER` | Order | Operations out of order |
| `EC-OVERFLOW` | Overflow | String or arithmetic overflow |
| `EC-PROGRAM` | Program | Program call exceptions |
| `EC-RANGE` | Range | Value out of range |
| `EC-SIZE` | Size | Arithmetic size error |
| `EC-SORT-MERGE` | Sort/Merge | Sort or merge operation exceptions |
| `EC-STORAGE` | Storage | Dynamic memory exceptions |

```cobol
COMPUTE WS-RESULT = WS-A / WS-B
IF FUNCTION EXCEPTION-STATUS = "EC-SIZE-ZERO-DIVIDE"
    DISPLAY "Division by zero"
END-IF
```

---

## Complete Error Reporting Example

```cobol
       WORKING-STORAGE SECTION.
       01  WS-RECORD.
           05  WS-CUST-ID     PIC X(10).
           05  WS-CUST-NAME   PIC X(30).
       01  WS-FILE-STATUS     PIC XX.

       PROCEDURE DIVISION.
       MAIN-PARA.
           OPEN INPUT CUSTOMER-FILE
           IF WS-FILE-STATUS NOT = "00"
               PERFORM REPORT-EXCEPTION
               STOP RUN
           END-IF

           PERFORM UNTIL WS-FILE-STATUS NOT = "00"
               READ CUSTOMER-FILE INTO WS-RECORD
                   AT END
                       MOVE "10" TO WS-FILE-STATUS
                   NOT AT END
                       PERFORM PROCESS-RECORD
               END-READ
           END-PERFORM

           CLOSE CUSTOMER-FILE
           STOP RUN.

       REPORT-EXCEPTION.
           DISPLAY "=== Exception Report ==="
           DISPLAY "Status:    "
                   FUNCTION EXCEPTION-STATUS
           DISPLAY "File:      "
                   FUNCTION EXCEPTION-FILE
           DISPLAY "Statement: "
                   FUNCTION EXCEPTION-STATEMENT
           DISPLAY "Location:  "
                   FUNCTION EXCEPTION-LOCATION
           DISPLAY "========================".
```

### Output Example

```
=== Exception Report ===
Status:    EC-I-O-PERMANENT-ERROR
File:      CUSTOMER-FILE
Statement: OPEN
Location:  MAIN-PARA IN CUST-REPORT
========================
```

---

## Relationship to Declaratives

The exception handling functions complement the traditional `USE` declarative approach:

```cobol
       DECLARATIVES.
       FILE-ERROR SECTION.
           USE AFTER STANDARD ERROR PROCEDURE ON INPUT.
       FILE-ERROR-PARA.
           DISPLAY "I/O error on "
                   FUNCTION EXCEPTION-FILE
           DISPLAY "Condition: "
                   FUNCTION EXCEPTION-STATUS.
       END DECLARATIVES.
```

The intrinsic functions can be used both inside and outside declaratives, providing more flexibility than the `FILE STATUS` variable alone.

---

## Comparison with FILE STATUS

| Feature | FILE STATUS | Exception Functions |
|---------|------------|-------------------|
| Scope | Single file | Any exception |
| Detail | 2-character code | Full condition name |
| File name | Must be tracked manually | EXCEPTION-FILE provides it |
| Statement | Not available | EXCEPTION-STATEMENT provides it |
| Location | Not available | EXCEPTION-LOCATION provides it |
| Standard | COBOL-85 | COBOL 2002 |
| Availability | All compilers | COBOL 2002+ compilers |

!!! tip "Best Practice"
    Use FILE STATUS for routine file I/O checking (it is universally supported
    and well-understood). Use the EXCEPTION functions for detailed diagnostic
    reporting, especially in error-handling routines and centralized exception
    handlers.

---

## See Also

- [Intrinsic Functions](index.md) -- overview of all intrinsic functions
- [OPEN](../procedure-division/io/open.md) -- opening files
- [READ](../procedure-division/io/read.md) -- reading records
- [WRITE](../procedure-division/io/write.md) -- writing records
