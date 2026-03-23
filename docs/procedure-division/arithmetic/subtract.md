# SUBTRACT

The `SUBTRACT` statement subtracts one or more numeric operands from a given operand and stores the result. It supports subtracting from an item in place, storing the difference in a separate receiving item, and subtracting corresponding fields between group items.

- **Standard:** COBOL-60, COBOL-61, COBOL-61 Extended, COBOL-65, COBOL-68, COBOL-74, COBOL-85, COBOL 2002, COBOL 2014, COBOL 2023
- **Division:** Procedure Division
- **Category:** Arithmetic

---

## Syntax

### Format 1: SUBTRACT ... FROM

```cobol
SUBTRACT {identifier-1 | literal-1} ...
    FROM {identifier-2 [ROUNDED [MODE IS rounding-mode]]} ...
    [ON SIZE ERROR imperative-statement-1]
    [NOT ON SIZE ERROR imperative-statement-2]
[END-SUBTRACT]
```

### Format 2: SUBTRACT ... FROM ... GIVING

```cobol
SUBTRACT {identifier-1 | literal-1} ...
    FROM {identifier-2 | literal-2}
    GIVING {identifier-3 [ROUNDED [MODE IS rounding-mode]]} ...
    [ON SIZE ERROR imperative-statement-1]
    [NOT ON SIZE ERROR imperative-statement-2]
[END-SUBTRACT]
```

### Format 3: SUBTRACT CORRESPONDING

```cobol
SUBTRACT CORRESPONDING identifier-1
    FROM identifier-2 [ROUNDED [MODE IS rounding-mode]]
    [ON SIZE ERROR imperative-statement-1]
    [NOT ON SIZE ERROR imperative-statement-2]
[END-SUBTRACT]
```

Where *rounding-mode* is one of: `AWAY-FROM-ZERO`, `NEAREST-AWAY-FROM-ZERO`, `NEAREST-EVEN`, `NEAREST-TOWARD-ZERO`, `PROHIBITED`, `TOWARD-GREATER`, `TOWARD-LESSER`, `TRUNCATION`.

---

## Rules

### Format 1: SUBTRACT ... FROM

All identifiers and literals preceding the `FROM` keyword are summed, and that sum is subtracted from the current value of each identifier following `FROM`. Each identifier after `FROM` is both an operand and a receiving item.

```cobol
SUBTRACT WS-A WS-B FROM WS-C WS-D
*> Equivalent to:
*>   WS-C = WS-C - (WS-A + WS-B)
*>   WS-D = WS-D - (WS-A + WS-B)
```

### Format 2: SUBTRACT ... FROM ... GIVING

All identifiers and literals preceding `FROM` are summed, and that sum is subtracted from the single operand following `FROM`. The difference is stored in each identifier following the `GIVING` keyword. The identifiers after `GIVING` are receiving items only; their original values do not participate in the subtraction.

```cobol
SUBTRACT WS-A WS-B FROM WS-C GIVING WS-D WS-E
*> Equivalent to:
*>   WS-D = WS-C - (WS-A + WS-B)
*>   WS-E = WS-C - (WS-A + WS-B)
```

In Format 2, the identifiers after `GIVING` may be numeric or numeric-edited items. All other operands must be numeric.

### Format 3: SUBTRACT CORRESPONDING

The `CORRESPONDING` (or `CORR`) phrase causes the compiler to subtract each elementary numeric item in *identifier-1* from the corresponding elementary numeric item in *identifier-2*, based on matching data-names. Both identifiers must be group items.

Two items correspond when they have the same data-name (ignoring qualifiers unique to their respective groups), and both items (or the items subordinate to them) are elementary numeric items. Items with `REDEFINES`, `RENAMES`, `OCCURS`, or `INDEX` clauses, and items subordinate to such items, are excluded from correspondence.

```cobol
SUBTRACT CORRESPONDING WS-DEDUCTIONS FROM WS-PAY
```

### ROUNDED Phrase

When the `ROUNDED` phrase is specified, the result is rounded rather than truncated before being stored in the receiving identifier. Without `ROUNDED`, excess decimal digits are truncated.

!!! note "COBOL 2002"
    The `ROUNDED MODE IS` phrase with named rounding modes was introduced in COBOL 2002. Prior standards support only the basic `ROUNDED` keyword, which uses an implementation-defined rounding mode (typically nearest-away-from-zero).

### ON SIZE ERROR

The `ON SIZE ERROR` phrase specifies statements to execute when the result of the subtraction exceeds the capacity of a receiving identifier. When a size error occurs, the content of the receiving identifier that caused the error is not modified.

The `NOT ON SIZE ERROR` phrase specifies statements to execute when the operation completes successfully without a size error.

### END-SUBTRACT

The `END-SUBTRACT` scope terminator delimits the scope of the `SUBTRACT` statement. It is required when the `SUBTRACT` statement is nested within another statement and an explicit scope terminator is needed to prevent ambiguity.

!!! note "COBOL-85"
    The `END-SUBTRACT` scope terminator was introduced in COBOL-85. In earlier standards, the scope of the `ON SIZE ERROR` phrase is implicitly terminated by the next sentence-ending period.

### Multiple Receiving Fields

In Format 1 and Format 2, multiple receiving identifiers may be specified. The difference is computed once and then stored in each receiving identifier independently. A size error in one receiving identifier does not affect the others.

### Operand Requirements

All identifiers used as operands (excluding `GIVING` targets in Format 2) must refer to numeric data items. The [PICTURE](../../data-division/picture.md) clause of each operand determines its decimal point alignment. The [USAGE](../../data-division/usage.md) clause of operands may differ; the compiler performs any necessary conversions.

---

## Behavior

- In Format 1, the subtrahends (identifiers and literals before `FROM`) are summed first. That intermediate sum is then subtracted from the current value of each receiving identifier after `FROM`.
- In Format 2, the subtrahends are summed and subtracted from the operand after `FROM`. The result is moved to each `GIVING` identifier. The receiving identifiers' original values are not part of the computation.
- In Format 3, each pair of corresponding items is subtracted independently. A size error on one pair does not prevent subtraction of other pairs.
- When no `ON SIZE ERROR` phrase is specified and a size error occurs, the result in the affected receiving identifier is undefined.
- If the result of the subtraction is negative and the receiving identifier is unsigned (no `S` in its `PICTURE`), the absolute value of the result is stored and a size error condition exists.

---

## Examples

### Simple Subtraction

```cobol
01 WS-BALANCE  PIC 9(7)V99  VALUE 0005000.00.
01 WS-PAYMENT  PIC 9(5)V99  VALUE 00750.25.

SUBTRACT WS-PAYMENT FROM WS-BALANCE
*> WS-BALANCE = 0004249.75
```

### Subtracting Multiple Values

```cobol
01 WS-TAX         PIC 9(5)V99  VALUE 00500.00.
01 WS-INSURANCE   PIC 9(5)V99  VALUE 00200.00.
01 WS-GROSS-PAY   PIC 9(7)V99  VALUE 0005000.00.

SUBTRACT WS-TAX WS-INSURANCE FROM WS-GROSS-PAY
*> WS-GROSS-PAY = 0004300.00  (5000 - 500 - 200)
```

### SUBTRACT ... FROM ... GIVING

```cobol
01 WS-LIST-PRICE  PIC 9(5)V99  VALUE 00100.00.
01 WS-DISCOUNT    PIC 9(3)V99  VALUE 015.50.
01 WS-NET-PRICE   PIC $ZZZ,ZZ9.99.

SUBTRACT WS-DISCOUNT FROM WS-LIST-PRICE
    GIVING WS-NET-PRICE
*> WS-NET-PRICE = "     $84.50"
*> WS-LIST-PRICE is unchanged (still 00100.00)
```

### SUBTRACT CORRESPONDING

```cobol
01 WS-DEDUCTIONS.
   05 WS-FEDERAL-TAX   PIC 9(5)V99  VALUE 00800.00.
   05 WS-STATE-TAX     PIC 9(5)V99  VALUE 00300.00.
   05 WS-INSURANCE     PIC 9(5)V99  VALUE 00150.00.

01 WS-PAY.
   05 WS-FEDERAL-TAX   PIC 9(7)V99  VALUE 0005000.00.
   05 WS-STATE-TAX     PIC 9(7)V99  VALUE 0002000.00.
   05 WS-INSURANCE     PIC 9(7)V99  VALUE 0001200.00.

SUBTRACT CORRESPONDING WS-DEDUCTIONS FROM WS-PAY
*> WS-PAY:
*>   WS-FEDERAL-TAX = 0004200.00
*>   WS-STATE-TAX   = 0001700.00
*>   WS-INSURANCE   = 0001050.00
```

### SUBTRACT with SIZE ERROR

```cobol
01 WS-SMALL-FIELD  PIC 9(3)  VALUE 100.
01 WS-BIG-VALUE    PIC 9(5)  VALUE 05000.

SUBTRACT WS-BIG-VALUE FROM WS-SMALL-FIELD
    ON SIZE ERROR
        DISPLAY "Result too large or negative for field"
    NOT ON SIZE ERROR
        DISPLAY "Result: " WS-SMALL-FIELD
END-SUBTRACT
*> Displays "Result too large or negative for field"
*> WS-SMALL-FIELD remains 100
```

### Subtracting a Literal

```cobol
01 WS-COUNTER  PIC 9(5)  VALUE 00100.

SUBTRACT 1 FROM WS-COUNTER
*> WS-COUNTER = 00099
```

### Multiple Receiving Identifiers

```cobol
01 WS-DISCOUNT   PIC 9(3)V99  VALUE 010.00.
01 WS-PRICE-1    PIC 9(5)V99  VALUE 00250.00.
01 WS-PRICE-2    PIC 9(5)V99  VALUE 00175.00.

SUBTRACT WS-DISCOUNT FROM WS-PRICE-1 WS-PRICE-2
*> WS-PRICE-1 = 00240.00
*> WS-PRICE-2 = 00165.00
```

---

## See Also

- [ADD](add.md) — adds numeric operands
- [MULTIPLY](multiply.md) — multiplies numeric operands
- [DIVIDE](divide.md) — divides numeric operands
- [COMPUTE](compute.md) — evaluates arithmetic expressions
- [PICTURE](../../data-division/picture.md) — defines data item format and size
- [USAGE](../../data-division/usage.md) — defines internal representation of data items
