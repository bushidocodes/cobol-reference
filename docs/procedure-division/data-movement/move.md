# MOVE

The `MOVE` statement transfers data from a sending operand to one or more receiving operands, with any necessary conversion, padding, or truncation.

- **Standard:** COBOL-60, COBOL-61, COBOL-61 Extended, COBOL-65, COBOL-68, COBOL-74, COBOL-85, COBOL 2002, COBOL 2014, COBOL 2023
- **Division:** Procedure Division
- **Category:** Data Movement

---

## Syntax

### Format 1 — Simple MOVE

```cobol
MOVE {identifier-1 | literal-1} TO identifier-2 [identifier-3] ...
```

### Format 2 — MOVE CORRESPONDING

```cobol
MOVE CORRESPONDING identifier-1 TO identifier-2
```

In Format 2, `CORRESPONDING` may be abbreviated as `CORR`.

---

## Rules

### Elementary MOVE

When the sending and receiving items are both elementary, data is moved according to the categories of the items involved. The following table summarizes the permissible moves:

| Sending Category | Receiving Category | Action |
|---|---|---|
| Numeric | Numeric | Aligned on decimal point; truncated or zero-filled as needed |
| Numeric | Numeric-edited | Edited according to receiving item's PICTURE |
| Alphanumeric | Alphanumeric | Left-justified; truncated or space-filled on the right |
| Alphanumeric | Numeric | Treated as an unsigned integer; aligned on decimal point |
| Alphanumeric | Numeric-edited | Treated as an unsigned integer; then edited |
| Numeric-integer | Alphanumeric | Treated as alphanumeric (no editing); left-justified |
| Alphabetic | Alphabetic or Alphanumeric | Left-justified; truncated or space-filled on the right |

!!! warning "Illegal Moves"
    Not all category combinations are permitted. Moving an alphabetic item to a numeric item, or a numeric-edited item to a numeric item (without de-editing support), produces undefined results or a compile-time diagnostic. Consult the compiler's documentation for the exact list of permissible moves.

### Group MOVE

When either the sending item or the receiving item (or both) is a group item, the move is treated as an **alphanumeric-to-alphanumeric** move regardless of the subordinate items' categories. No conversion or editing takes place. The sending item is treated as if it were a single alphanumeric field and is left-justified in the receiving item.

### MOVE CORRESPONDING

`MOVE CORRESPONDING` moves data between pairs of elementary items that share the same name within two group items. For each elementary item in identifier-2 that has a corresponding item in identifier-1 (matching by name, after excluding FILLER items and items that do not have corresponding items in the other group), an individual MOVE is executed.

```cobol
01 INPUT-REC.
   05 CUST-ID      PIC X(10).
   05 CUST-NAME    PIC X(30).
   05 CUST-BALANCE PIC 9(7)V99.

01 OUTPUT-REC.
   05 CUST-ID      PIC X(10).
   05 CUST-NAME    PIC X(30).
   05 CUST-STATUS  PIC X(1).

MOVE CORRESPONDING INPUT-REC TO OUTPUT-REC
*> Moves CUST-ID and CUST-NAME; CUST-BALANCE and CUST-STATUS
*> are not moved because they have no corresponding match.
```

`CORRESPONDING` can be abbreviated as `CORR`. The same matching logic applies to `ADD CORRESPONDING` and `SUBTRACT CORRESPONDING`, which perform arithmetic on matching numeric fields between two group items.

---

## Data Conversion Rules

### Numeric to Numeric

The sending value is aligned on the decimal point of the receiving item. Excess digits on either side of the decimal point are truncated. Missing digits are filled with zeros.

```cobol
01 WS-SEND   PIC 9(5)V99  VALUE 12345.67.
01 WS-RECV   PIC 9(3)V9.

MOVE WS-SEND TO WS-RECV
*> WS-RECV contains 345.6 (high-order and low-order truncation)
```

### Alphanumeric to Alphanumeric

The sending data is placed in the receiving item starting at the leftmost position. If the sending item is shorter, the receiving item is padded on the right with spaces. If the sending item is longer, the data is truncated on the right.

```cobol
01 WS-SHORT  PIC X(5)  VALUE "HELLO".
01 WS-LONG   PIC X(10).

MOVE WS-SHORT TO WS-LONG
*> WS-LONG contains "HELLO     " (padded with 5 spaces)
```

### Numeric to Alphanumeric

The numeric item is treated as a character string (without editing). The sending item is moved as if it were alphanumeric.

### Alphanumeric to Numeric

The sending item is treated as an unsigned numeric integer, right-justified and aligned on an assumed decimal point at the rightmost position.

```cobol
01 WS-ALPHA  PIC X(5)  VALUE "00123".
01 WS-NUM    PIC 9(5).

MOVE WS-ALPHA TO WS-NUM
*> WS-NUM contains 00123
```

!!! warning "Non-Numeric Data in Alphanumeric-to-Numeric Moves"
    If the sending alphanumeric item contains non-numeric characters (letters, special characters), the result of moving it to a numeric item is undefined. No runtime error is guaranteed; the receiving field may contain garbage data.

### De-editing

When a numeric-edited item is the sending field and a numeric item is the receiving field, some compilers perform **de-editing**: the numeric value is extracted from the edited field and moved to the numeric receiver.

!!! note "COBOL 2002"
    De-editing (moving a numeric-edited item to a numeric item) was standardized in COBOL 2002. Prior to that standard, de-editing was a vendor extension. Programs relying on de-editing should verify compiler support.

---

## Behavior

- The sending item is not modified by the MOVE statement.
- When multiple receiving items are specified (`MOVE X TO A B C`), the statement behaves as if a separate MOVE were executed for each receiver, all using the original value of the sending item.
- The sending and receiving items may overlap in storage only if they reference the same data item exactly. Overlapping but non-identical storage references produce undefined results.

---

## Examples

### Basic Numeric Move with Truncation and Zero-Fill

```cobol
01 WS-AMT-IN   PIC 9(7)V99    VALUE 1234567.89.
01 WS-AMT-OUT  PIC 9(5)V9(4).

MOVE WS-AMT-IN TO WS-AMT-OUT
*> WS-AMT-OUT = 34567.8900
*> High-order truncation of "12"; low-order zero-fill of "00"
```

### Moving a Literal to Multiple Receivers

```cobol
01 WS-FLAG-A   PIC X     VALUE "Y".
01 WS-FLAG-B   PIC X     VALUE "Y".
01 WS-FLAG-C   PIC X     VALUE "Y".

MOVE "N" TO WS-FLAG-A WS-FLAG-B WS-FLAG-C
*> All three flags are now "N"
```

### Numeric to Numeric-Edited Move

```cobol
01 WS-SALARY   PIC 9(6)V99   VALUE 045000.50.
01 WS-DISPLAY  PIC $ZZZ,ZZ9.99.

MOVE WS-SALARY TO WS-DISPLAY
*> WS-DISPLAY = " $45,000.50"
```

### Group Move (No Conversion)

```cobol
01 REC-A.
   05 FIELD-1   PIC 9(3)  VALUE 123.
   05 FIELD-2   PIC X(5)  VALUE "ABCDE".

01 REC-B.
   05 FIELD-X   PIC X(4).
   05 FIELD-Y   PIC X(4).

MOVE REC-A TO REC-B
*> REC-B treated as PIC X(8), receives "123ABCDE" left-justified
*> FIELD-X = "123A", FIELD-Y = "BCDE"
*> No numeric conversion occurs; this is an alphanumeric group move.
```

### MOVE CORRESPONDING

```cobol
01 WS-INPUT.
   05 EMP-ID      PIC X(6)    VALUE "E12345".
   05 EMP-NAME    PIC X(20)   VALUE "SMITH, JOHN".
   05 EMP-DEPT    PIC X(4)    VALUE "IT01".
   05 EMP-SALARY  PIC 9(7)V99 VALUE 0065000.00.

01 WS-OUTPUT.
   05 EMP-ID      PIC X(6).
   05 EMP-NAME    PIC X(25).
   05 EMP-TITLE   PIC X(15).

MOVE CORRESPONDING WS-INPUT TO WS-OUTPUT
*> EMP-ID   in WS-OUTPUT = "E12345"
*> EMP-NAME in WS-OUTPUT = "SMITH, JOHN              "
*> EMP-TITLE is unchanged (no corresponding field)
*> EMP-DEPT and EMP-SALARY are not moved (no corresponding field)
```

### Signed Numeric Move

```cobol
01 WS-SIGNED   PIC S9(5)V99  VALUE -12345.67.
01 WS-UNSIGNED PIC  9(5)V99.

MOVE WS-SIGNED TO WS-UNSIGNED
*> WS-UNSIGNED = 12345.67 (sign is lost; absolute value is moved)
```

---

## See Also

- INITIALIZE — sets data items to default values
- STRING — concatenates data items
- UNSTRING — splits a data item
- [Procedure Division Overview](../index.md)
