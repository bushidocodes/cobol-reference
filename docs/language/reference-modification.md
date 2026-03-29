# Reference Modification

Reference modification provides substring access to any alphanumeric or national data item, allowing a program to reference a contiguous portion of a data item without defining subordinate items.

- **Standard:** COBOL-85, COBOL 2002, COBOL 2014, COBOL 2023

---

## Syntax

```cobol
identifier ( leftmost-character-position : [ length ] )
```

- **leftmost-character-position** -- an arithmetic expression specifying the starting position (1-based).
- **length** -- an optional arithmetic expression specifying the number of character positions. If omitted, the reference extends from the starting position to the end of the data item.

---

## Rules

1. The leftmost character position must be a **positive integer** (1 or greater).
2. The leftmost character position must not exceed the length of the data item.
3. If length is specified, the sum of leftmost-character-position and length minus 1 must not exceed the length of the data item.
4. If length is omitted, the reference extends from the starting position through the end of the data item.
5. Reference modification may be applied to **any** data item of category alphanumeric, alphanumeric-edited, DBCS, or national.
6. The result of a reference modification is always treated as an **elementary alphanumeric** (or national) item.
7. Reference modification can be combined with subscripting: `table-item(subscript)(start:length)`.
8. Both the starting position and length can be data names, literals, or arithmetic expressions.

---

## Examples

### Basic Substring Access

```cobol
       01  WS-DATE       PIC X(8) VALUE "20250322".
       01  WS-YEAR       PIC X(4).
       01  WS-MONTH      PIC X(2).
       01  WS-DAY        PIC X(2).

       MOVE WS-DATE(1:4) TO WS-YEAR
       *> WS-YEAR = "2025"

       MOVE WS-DATE(5:2) TO WS-MONTH
       *> WS-MONTH = "03"

       MOVE WS-DATE(7:2) TO WS-DAY
       *> WS-DAY = "22"
```

### Omitting Length

```cobol
       01  WS-FULLNAME   PIC X(30) VALUE "John Smith".

       DISPLAY WS-FULLNAME(6:)
       *> Displays "Smith" followed by trailing spaces
```

### With Arithmetic Expressions

```cobol
       01  WS-TEXT       PIC X(50).
       01  WS-START      PIC 99.
       01  WS-LEN        PIC 99.

       MOVE 10 TO WS-START
       MOVE 5  TO WS-LEN
       DISPLAY WS-TEXT(WS-START:WS-LEN)
       *> Displays 5 characters starting at position 10
```

### Combined with Subscripting

```cobol
       01  WS-TABLE.
           05  WS-ENTRY  PIC X(20) OCCURS 10 TIMES.

       DISPLAY WS-ENTRY(3)(1:5)
       *> Displays first 5 characters of the 3rd entry
```

### Practical: Extracting from CURRENT-DATE

```cobol
       01  WS-DATETIME   PIC X(21).

       MOVE FUNCTION CURRENT-DATE TO WS-DATETIME
       DISPLAY "Year:  " WS-DATETIME(1:4)
       DISPLAY "Month: " WS-DATETIME(5:2)
       DISPLAY "Day:   " WS-DATETIME(7:2)
       DISPLAY "Hour:  " WS-DATETIME(9:2)
       DISPLAY "Min:   " WS-DATETIME(11:2)
       DISPLAY "Sec:   " WS-DATETIME(13:2)
```

### Modifying a Substring

```cobol
       01  WS-SSN    PIC X(11) VALUE "123-45-6789".

       MOVE "XXX" TO WS-SSN(1:3)
       MOVE "XX"  TO WS-SSN(5:2)
       *> WS-SSN = "XXX-XX-6789" (masked)
```

---

## Reference Modification vs Subordinate Items

| Approach | Advantages | Disadvantages |
|----------|-----------|---------------|
| Reference modification | No additional data definitions needed; positions can be computed at runtime | Less readable; no meaningful names for parts |
| Subordinate items (REDEFINES or group) | Self-documenting; named fields | Requires additional data definitions; positions fixed at compile time |

Use reference modification for dynamic substring access and one-off extractions. Use subordinate items for structured data with well-known layouts.

---

## See Also

- [PICTURE Clause](../data-division/picture.md) -- data format specification
- [OCCURS Clause](../data-division/occurs.md) -- subscripting
- [MOVE](../procedure-division/data-movement/move.md) -- data movement
- [STRING](../procedure-division/string-handling/string.md) -- string concatenation
- [INSPECT](../procedure-division/string-handling/inspect.md) -- character scanning
