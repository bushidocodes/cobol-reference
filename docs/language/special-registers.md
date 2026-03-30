# Special Registers

Special registers are compiler-generated data items that provide information about the program's execution environment or communicate status between the program and the runtime system. They are available without explicit declaration.

- **Standard:** Various (some since COBOL-60, others added in later standards)

---

## Commonly Used Special Registers

### RETURN-CODE

A binary numeric item that passes a return code between programs.

- **Type:** Numeric (typically PIC S9(4) COMP or BINARY-LONG, implementation-dependent)
- **Standard:** COBOL-85+

```cobol
      *> In a called subprogram:
       MOVE 0 TO RETURN-CODE
       GOBACK.

      *> In the calling program:
       CALL "SUB-PROG" USING WS-DATA
       IF RETURN-CODE NOT = 0
           DISPLAY "Subprogram failed: " RETURN-CODE
       END-IF
```

- Set by a called program before returning to indicate success (0) or failure (non-zero).
- The calling program can examine RETURN-CODE after the CALL.
- RETURN-CODE is implicitly set by the RETURNING phrase on CALL and GOBACK.
- Convention: 0 = success, 4 = warning, 8 = error, 12 = severe error, 16 = critical.

### SORT-RETURN

Indicates the status of the most recent SORT, MERGE, RELEASE, or RETURN operation.

- **Type:** Numeric
- **Standard:** COBOL-74+

| Value | Meaning |
|-------|---------|
| 0 | Successful |
| 16 | Unsuccessful |

```cobol
       SORT SORT-FILE ON ASCENDING KEY SORT-KEY
           USING INPUT-FILE
           GIVING OUTPUT-FILE
       IF SORT-RETURN NOT = 0
           DISPLAY "Sort failed"
       END-IF
```

### LINAGE-COUNTER

Tracks the current line position within the page body of a file with a LINAGE clause.

- **Type:** Numeric
- **Standard:** COBOL-74+

See [LINAGE Clause](../data-division/linage.md) for details.

### TALLY (Obsolete)

A system-defined numeric counter used with the EXAMINE verb.

- **Type:** Numeric (PIC 9(5) or similar)
- **Standard:** COBOL-60 through COBOL-68; retained for compatibility in some compilers

!!! warning "Obsolete"
    TALLY was replaced when EXAMINE was superseded by INSPECT in COBOL-74.
    INSPECT uses explicit programmer-defined counters instead of TALLY.
    Some compilers (including IBM Enterprise COBOL and GnuCOBOL) still
    support TALLY for backward compatibility.

### LENGTH OF

Returns the length in bytes of a data item.

- **Type:** Numeric (integer)
- **Standard:** COBOL-85+

```cobol
       DISPLAY "Record length: " LENGTH OF CUSTOMER-RECORD
       COMPUTE WS-SIZE = LENGTH OF WS-TABLE-ENTRY * WS-COUNT
```

LENGTH OF returns the **byte length** of the data item, including all subordinate items for group items. It differs from FUNCTION LENGTH which returns character positions.

### ADDRESS OF

Returns the address of a data item in memory.

- **Type:** USAGE POINTER
- **Standard:** COBOL-85+

```cobol
       SET WS-PTR TO ADDRESS OF WS-RECORD
       SET ADDRESS OF LS-BUFFER TO WS-PTR
```

Used with LINKAGE SECTION items, pointers, and dynamic memory management. See [ALLOCATE and FREE](../procedure-division/memory/allocate-free.md).

---

## XML/JSON Special Registers

These registers are set automatically during XML PARSE, XML GENERATE, JSON PARSE, and JSON GENERATE operations.

| Register | Type | Purpose |
|----------|------|---------|
| XML-CODE | Numeric | Status code from XML operations (0 = success) |
| XML-EVENT | Alphanumeric | Type of XML parse event (e.g., "START-OF-ELEMENT") |
| XML-TEXT | Alphanumeric | Text content from XML parse event |
| XML-NTEXT | National | National character version of XML-TEXT |
| XML-NAMESPACE | Alphanumeric | XML namespace URI |
| XML-NAMESPACE-PREFIX | Alphanumeric | XML namespace prefix |
| JSON-CODE | Numeric | Status code from JSON operations (0 = success) |
| JSON-STATUS | Alphanumeric | Additional JSON status information |

See [XML Processing](../extensions/xml.md) and [JSON Processing](../extensions/json.md).

---

## Sort-Related Special Registers

| Register | Purpose | Notes |
|----------|---------|-------|
| SORT-RETURN | Sort/merge operation status | 0 = success, 16 = failure |
| SORT-CONTROL | Name of sort control file | Implementation-dependent |
| SORT-CORE-SIZE | Memory for sort | Obsolete |
| SORT-FILE-SIZE | Estimated record count | Implementation-dependent |
| SORT-MESSAGE | Sort message destination | Implementation-dependent |
| SORT-MODE-SIZE | Record size for sort | Obsolete |

---

## Debug Special Register (Removed)

### DEBUG-ITEM

A group item set by the runtime when a USE FOR DEBUGGING declarative is triggered.

- **Standard:** COBOL-74, obsolete in COBOL-85, removed in COBOL 2002

| Subordinate Field | Content |
|-------------------|---------|
| DEBUG-LINE | Source line number |
| DEBUG-NAME | Name of the item or procedure |
| DEBUG-SUB-1 | First subscript value |
| DEBUG-SUB-2 | Second subscript value |
| DEBUG-SUB-3 | Third subscript value |
| DEBUG-CONTENTS | Current value (data items) or spaces (procedures) |

---

## Compilation Special Register

### WHEN-COMPILED

The date and time the program was compiled.

- **Type:** PIC X(16) in the format `MM/DD/YYhh.mm.ss`
- **Standard:** COBOL-68+

```cobol
       DISPLAY "Compiled: " WHEN-COMPILED
```

!!! note
    The WHEN-COMPILED special register (PIC X(16)) is distinct from the
    FUNCTION WHEN-COMPILED intrinsic function (PIC X(21) in
    YYYYMMDDHHMMSSssGHHMM format). The function provides a four-digit
    year and UTC offset.

---

## See Also

- [CALL](../procedure-division/program-linkage/call.md) -- RETURN-CODE usage
- [SORT](../procedure-division/sort-merge/sort.md) -- SORT-RETURN usage
- [LINAGE Clause](../data-division/linage.md) -- LINAGE-COUNTER
- [XML Processing](../extensions/xml.md) -- XML special registers
- [JSON Processing](../extensions/json.md) -- JSON special registers
- [ALLOCATE and FREE](../procedure-division/memory/allocate-free.md) -- ADDRESS OF
- [INSPECT](../procedure-division/string-handling/inspect.md) -- replaced EXAMINE/TALLY
- [EXAMINE (Removed)](../procedure-division/archaic/examine.md) -- historical TALLY usage
