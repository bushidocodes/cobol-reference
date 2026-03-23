# MULTIPLY

The `MULTIPLY` statement multiplies two numeric operands and stores the product. It supports multiplying into the operand in place or storing the product in a separate receiving item.

- **Standard:** COBOL-60, COBOL-61, COBOL-61 Extended, COBOL-65, COBOL-68, COBOL-74, COBOL-85, COBOL 2002, COBOL 2014, COBOL 2023
- **Division:** Procedure Division
- **Category:** Arithmetic

---

## Syntax

### Format 1: MULTIPLY ... BY

```cobol
MULTIPLY {identifier-1 | literal-1}
    BY {identifier-2 [ROUNDED [MODE IS rounding-mode]]} ...
    [ON SIZE ERROR imperative-statement-1]
    [NOT ON SIZE ERROR imperative-statement-2]
[END-MULTIPLY]
```

### Format 2: MULTIPLY ... BY ... GIVING

```cobol
MULTIPLY {identifier-1 | literal-1}
    BY {identifier-2 | literal-2}
    GIVING {identifier-3 [ROUNDED [MODE IS rounding-mode]]} ...
    [ON SIZE ERROR imperative-statement-1]
    [NOT ON SIZE ERROR imperative-statement-2]
[END-MULTIPLY]
```

Where *rounding-mode* is one of: `AWAY-FROM-ZERO`, `NEAREST-AWAY-FROM-ZERO`, `NEAREST-EVEN`, `NEAREST-TOWARD-ZERO`, `PROHIBITED`, `TOWARD-GREATER`, `TOWARD-LESSER`, `TRUNCATION`.

---

## Rules

### Format 1: MULTIPLY ... BY

The value of *identifier-1* or *literal-1* is multiplied by the current value of each *identifier-2*. The product replaces the value of each *identifier-2*. Each *identifier-2* is both an operand and a receiving item.

```cobol
MULTIPLY WS-RATE BY WS-AMOUNT-1 WS-AMOUNT-2
*> Equivalent to:
*>   WS-AMOUNT-1 = WS-RATE * WS-AMOUNT-1
*>   WS-AMOUNT-2 = WS-RATE * WS-AMOUNT-2
```

### Format 2: MULTIPLY ... BY ... GIVING

The value of *identifier-1* or *literal-1* is multiplied by the value of *identifier-2* or *literal-2*. The product is stored in each *identifier-3*. The identifiers after `GIVING` are receiving items only; their original values do not participate in the multiplication.

```cobol
MULTIPLY WS-PRICE BY WS-QTY GIVING WS-TOTAL
*> Equivalent to:
*>   WS-TOTAL = WS-PRICE * WS-QTY
```

In Format 2, the identifiers after `GIVING` may be numeric or numeric-edited items. All other operands must be numeric.

### ROUNDED Phrase

When the `ROUNDED` phrase is specified, the result is rounded rather than truncated before being stored in the receiving identifier. Without `ROUNDED`, excess decimal digits are truncated.

!!! note "COBOL 2002"
    The `ROUNDED MODE IS` phrase with named rounding modes was introduced in COBOL 2002. Prior standards support only the basic `ROUNDED` keyword, which uses an implementation-defined rounding mode (typically nearest-away-from-zero).

### ON SIZE ERROR

The `ON SIZE ERROR` phrase specifies statements to execute when the product exceeds the capacity of a receiving identifier. When a size error occurs, the content of the receiving identifier that caused the error is not modified.

The `NOT ON SIZE ERROR` phrase specifies statements to execute when the operation completes successfully without a size error.

### END-MULTIPLY

The `END-MULTIPLY` scope terminator delimits the scope of the `MULTIPLY` statement. It is required when the `MULTIPLY` statement is nested within another statement and an explicit scope terminator is needed to prevent ambiguity.

!!! note "COBOL-85"
    The `END-MULTIPLY` scope terminator was introduced in COBOL-85. In earlier standards, the scope of the `ON SIZE ERROR` phrase is implicitly terminated by the next sentence-ending period.

### Multiple Receiving Fields

In Format 1, multiple receiving identifiers may be specified after `BY`. Each is independently multiplied by the first operand. In Format 2, multiple receiving identifiers may be specified after `GIVING`. The product is computed once and stored in each. A size error in one receiving identifier does not affect the others.

### Operand Requirements

All identifiers used as operands (excluding `GIVING` targets in Format 2) must refer to numeric data items. The [PICTURE](../../data-division/picture.md) clause of each operand determines its decimal point alignment. The [USAGE](../../data-division/usage.md) clause of operands may differ; the compiler performs any necessary conversions.

---

## Behavior

- In Format 1, the multiplier (*identifier-1* or *literal-1*) is multiplied by each *identifier-2* independently. Each *identifier-2* receives its own product.
- In Format 2, the two operands are multiplied, and the product is moved to each `GIVING` identifier. The receiving identifiers' original values are not part of the computation.
- When no `ON SIZE ERROR` phrase is specified and a size error occurs, the result in the affected receiving identifier is undefined.
- The intermediate product has a precision sufficient to represent the full result without truncation. The number of decimal places in the intermediate result equals the sum of the decimal places of the two operands.

---

## Examples

### Simple Multiplication in Place

```cobol
01 WS-QUANTITY  PIC 9(5)     VALUE 00025.
01 WS-PRICE     PIC 9(5)V99  VALUE 00012.50.

MULTIPLY WS-QUANTITY BY WS-PRICE
*> WS-PRICE = 00312.50  (25 * 12.50)
```

### MULTIPLY ... BY ... GIVING

```cobol
01 WS-HOURS    PIC 9(3)V9   VALUE 040.0.
01 WS-RATE     PIC 9(3)V99  VALUE 025.75.
01 WS-PAY      PIC 9(7)V99.

MULTIPLY WS-HOURS BY WS-RATE GIVING WS-PAY
*> WS-PAY = 0001030.00  (40.0 * 25.75)
*> WS-HOURS and WS-RATE are unchanged
```

### Multiplication with a Literal

```cobol
01 WS-AMOUNT  PIC 9(7)V99  VALUE 0001000.00.

MULTIPLY 1.08 BY WS-AMOUNT
*> WS-AMOUNT = 0001080.00  (applying 8% increase)
```

### MULTIPLY with ROUNDED

```cobol
01 WS-BASE    PIC 9(5)V99  VALUE 00333.33.
01 WS-FACTOR  PIC 9V99     VALUE 3.00.
01 WS-RESULT  PIC 9(5)V99.

MULTIPLY WS-BASE BY WS-FACTOR
    GIVING WS-RESULT ROUNDED
*> WS-RESULT = 00999.99  (333.33 * 3.00)
```

### MULTIPLY with SIZE ERROR

```cobol
01 WS-BIG-A   PIC 9(5)  VALUE 99999.
01 WS-BIG-B   PIC 9(5)  VALUE 99999.
01 WS-RESULT  PIC 9(7).

MULTIPLY WS-BIG-A BY WS-BIG-B
    GIVING WS-RESULT
    ON SIZE ERROR
        DISPLAY "Product exceeds field capacity"
        MOVE 0 TO WS-RESULT
    NOT ON SIZE ERROR
        DISPLAY "Product: " WS-RESULT
END-MULTIPLY
*> Displays "Product exceeds field capacity"
*> (99999 * 99999 = 9999800001, exceeds PIC 9(7))
```

### Multiple Receiving Identifiers (Format 1)

```cobol
01 WS-RATE       PIC 9V99   VALUE 1.10.
01 WS-SALARY-1   PIC 9(7)V99  VALUE 0045000.00.
01 WS-SALARY-2   PIC 9(7)V99  VALUE 0052000.00.

MULTIPLY WS-RATE BY WS-SALARY-1 WS-SALARY-2
*> WS-SALARY-1 = 0049500.00  (45000 * 1.10)
*> WS-SALARY-2 = 0057200.00  (52000 * 1.10)
```

### Multiple Receiving Identifiers (Format 2)

```cobol
01 WS-LENGTH    PIC 9(3)    VALUE 012.
01 WS-WIDTH     PIC 9(3)    VALUE 008.
01 WS-AREA      PIC 9(5).
01 WS-AREA-RPT  PIC Z,ZZ9.

MULTIPLY WS-LENGTH BY WS-WIDTH
    GIVING WS-AREA WS-AREA-RPT
*> WS-AREA     = 00096
*> WS-AREA-RPT = "   96"
```

### Percentage Calculation

```cobol
01 WS-GROSS-PAY  PIC 9(7)V99  VALUE 0005000.00.
01 WS-TAX-RATE   PIC V9(4)    VALUE .2150.
01 WS-TAX-AMT    PIC 9(7)V99.

MULTIPLY WS-GROSS-PAY BY WS-TAX-RATE
    GIVING WS-TAX-AMT ROUNDED
*> WS-TAX-AMT = 0001075.00  (5000.00 * 0.2150)
```

---

## See Also

- [ADD](add.md) — adds numeric operands
- [SUBTRACT](subtract.md) — subtracts numeric operands
- [DIVIDE](divide.md) — divides numeric operands
- [COMPUTE](compute.md) — evaluates arithmetic expressions
- [PICTURE](../../data-division/picture.md) — defines data item format and size
- [USAGE](../../data-division/usage.md) — defines internal representation of data items
