# Additional Data Description Clauses

This page documents data description clauses that do not have dedicated pages: BLANK WHEN ZERO, JUSTIFIED, SYNCHRONIZED, and SIGN.

- **Standard:** COBOL-68, COBOL-85, COBOL 2002, COBOL 2014, COBOL 2023
- **Division:** Data Division

---

## BLANK WHEN ZERO

Causes the data item to display as spaces when its value is zero.

```cobol
level-number data-name PIC clause BLANK WHEN ZERO.
```

### Rules

1. Can only be specified for elementary numeric or numeric-edited items.
2. When the item's value is zero, the entire field is filled with spaces instead of displaying zeros.
3. Useful for report formatting to avoid cluttered columns of zeros.
4. Cannot be used with `PIC *` (check protection) items.

### Example

```cobol
       01  WS-AMOUNT    PIC ZZ,ZZ9.99 BLANK WHEN ZERO.

       MOVE 1234.56 TO WS-AMOUNT
       DISPLAY WS-AMOUNT        *> " 1,234.56"

       MOVE 0 TO WS-AMOUNT
       DISPLAY WS-AMOUNT        *> "         " (all spaces)
```

---

## JUSTIFIED (JUST) RIGHT

Overrides the default left-justification of alphanumeric data on receiving.

```cobol
level-number data-name PIC X(n) JUSTIFIED RIGHT.
```

### Rules

1. Can only be specified for elementary alphabetic or alphanumeric items.
2. When data is moved to a JUSTIFIED RIGHT item, it is right-justified with left padding of spaces (instead of the default left-justification with right padding).
3. If the source data is longer than the receiving item, leftmost characters are truncated (instead of rightmost).
4. Cannot be used with numeric or edited items.
5. `JUST` is an abbreviation for `JUSTIFIED`.

### Example

```cobol
       01  WS-LEFT      PIC X(10).
       01  WS-RIGHT     PIC X(10) JUSTIFIED RIGHT.

       MOVE "ABC" TO WS-LEFT
       *> WS-LEFT = "ABC       " (left-justified, right-padded)

       MOVE "ABC" TO WS-RIGHT
       *> WS-RIGHT = "       ABC" (right-justified, left-padded)

       MOVE "ABCDEFGHIJKLM" TO WS-RIGHT
       *> WS-RIGHT = "DEFGHIJKLM" (left truncation)
```

---

## SYNCHRONIZED (SYNC)

Aligns a binary data item on its natural storage boundary for performance.

```cobol
level-number data-name PIC clause USAGE BINARY
    SYNCHRONIZED [ LEFT | RIGHT ].
```

### Rules

1. Primarily applies to BINARY / COMP / COMP-4 items.
2. Without SYNCHRONIZED, binary items may start at any byte offset within a record, requiring the CPU to perform unaligned access (slower on most architectures).
3. With SYNCHRONIZED, the compiler inserts **slack bytes** (padding) before the item to align it on a halfword (2-byte), fullword (4-byte), or doubleword (8-byte) boundary.
4. Slack bytes are not accessible by the programmer and do not appear in the record description.
5. SYNCHRONIZED can increase record size due to padding.
6. `SYNC` is an abbreviation for `SYNCHRONIZED`.
7. `LEFT` and `RIGHT` specify which end of the item is aligned; if omitted, the compiler chooses.

### Example

```cobol
       01  WS-RECORD.
           05  WS-FLAG       PIC X(1).
           05  WS-COUNT      PIC S9(8) COMP SYNC.
           *> Compiler inserts 3 slack bytes after WS-FLAG
           *> to align WS-COUNT on a fullword boundary
           05  WS-NAME       PIC X(20).
```

!!! warning "Record Size Impact"
    SYNCHRONIZED can change the actual storage size of a record, causing
    mismatches between the COBOL record description and the physical file
    layout. Avoid SYNCHRONIZED on items in file records unless the file
    format accounts for the padding.

---

## SIGN Clause

Specifies the position and representation of the sign in a signed numeric DISPLAY item.

```cobol
level-number data-name PIC S9(n)
    [ SIGN IS ] { LEADING | TRAILING } [ SEPARATE CHARACTER ].
```

### Rules

1. Applies only to signed numeric DISPLAY items (PIC with `S`).
2. Without the SIGN clause, the sign is embedded in the last digit (zone portion) — the default for most systems.
3. **LEADING** — sign is stored with the first digit.
4. **TRAILING** — sign is stored with the last digit (default on IBM mainframes).
5. **SEPARATE CHARACTER** — the sign occupies its own byte position (`+` or `-`), increasing the item size by 1 byte. Without SEPARATE, the sign is combined with a digit.
6. The SIGN clause can be specified at the group level, affecting all subordinate signed numeric items.
7. SEPARATE CHARACTER is often needed for ASCII systems or when exchanging data between EBCDIC and ASCII platforms.

### Example

```cobol
      *> Default: sign embedded in trailing digit
       01  WS-AMT-DEFAULT  PIC S9(5) VALUE -12345.
      *> Internal: "1234N" (EBCDIC) or similar

      *> Trailing separate: sign is a separate "-" character
       01  WS-AMT-SEP      PIC S9(5) SIGN IS TRAILING
                            SEPARATE CHARACTER.
      *> Internal: "12345-" (6 bytes, not 5)

      *> Leading separate
       01  WS-AMT-LEAD     PIC S9(5) SIGN IS LEADING
                            SEPARATE CHARACTER.
      *> Internal: "-12345" (6 bytes)
```

### When to Use SEPARATE CHARACTER

- Exchanging data between EBCDIC and ASCII systems
- Writing files read by non-COBOL programs
- Interfacing with databases that expect explicit sign characters
- When using DISPLAY items in ACCEPT/DISPLAY operations

---

## See Also

- [PICTURE Clause](picture.md) -- data format specification
- [USAGE Clause](usage.md) -- internal storage format
- [Level Numbers](level-numbers.md) -- data hierarchy
- [VALUE Clause](value.md) -- data initialization
