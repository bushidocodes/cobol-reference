# DIVIDE

The `DIVIDE` statement divides one numeric operand by another and stores the quotient. It supports division in place, storing the quotient in a separate receiving item, and obtaining a remainder. COBOL provides both `INTO` and `BY` phrasing to express the division in the order most natural to the programmer.

- **Standard:** COBOL-60, COBOL-61, COBOL-61 Extended, COBOL-65, COBOL-68, COBOL-74, COBOL-85, COBOL 2002, COBOL 2014, COBOL 2023
- **Division:** Procedure Division
- **Category:** Arithmetic

---

## Syntax

### Format 1: DIVIDE ... INTO

```cobol
DIVIDE {identifier-1 | literal-1}
    INTO {identifier-2 [ROUNDED [MODE IS rounding-mode]]} ...
    [ON SIZE ERROR imperative-statement-1]
    [NOT ON SIZE ERROR imperative-statement-2]
[END-DIVIDE]
```

### Format 2: DIVIDE ... INTO ... GIVING

```cobol
DIVIDE {identifier-1 | literal-1}
    INTO {identifier-2 | literal-2}
    GIVING {identifier-3 [ROUNDED [MODE IS rounding-mode]]} ...
    [REMAINDER identifier-4]
    [ON SIZE ERROR imperative-statement-1]
    [NOT ON SIZE ERROR imperative-statement-2]
[END-DIVIDE]
```

### Format 3: DIVIDE ... BY ... GIVING

```cobol
DIVIDE {identifier-1 | literal-1}
    BY {identifier-2 | literal-2}
    GIVING {identifier-3 [ROUNDED [MODE IS rounding-mode]]} ...
    [REMAINDER identifier-4]
    [ON SIZE ERROR imperative-statement-1]
    [NOT ON SIZE ERROR imperative-statement-2]
[END-DIVIDE]
```

Where *rounding-mode* is one of: `AWAY-FROM-ZERO`, `NEAREST-AWAY-FROM-ZERO`, `NEAREST-EVEN`, `NEAREST-TOWARD-ZERO`, `PROHIBITED`, `TOWARD-GREATER`, `TOWARD-LESSER`, `TRUNCATION`.

---

## Rules

### Format 1: DIVIDE ... INTO

The value of *identifier-1* or *literal-1* is the divisor. Each *identifier-2* is divided by that divisor, and the quotient replaces the value of each *identifier-2*. Each *identifier-2* is both the dividend and a receiving item.

```cobol
DIVIDE WS-DIVISOR INTO WS-AMOUNT-1 WS-AMOUNT-2
*> Equivalent to:
*>   WS-AMOUNT-1 = WS-AMOUNT-1 / WS-DIVISOR
*>   WS-AMOUNT-2 = WS-AMOUNT-2 / WS-DIVISOR
```

!!! tip "Reading INTO vs BY"
    `DIVIDE A INTO B` means B / A (A divides into B). `DIVIDE A BY B` means A / B. The `INTO` phrasing reads as "divide A into B," which is the inverse of what many programmers expect. Use the `BY` format (Format 3) when the natural reading order is preferred.

### Format 2: DIVIDE ... INTO ... GIVING

The value of *identifier-2* or *literal-2* (the dividend) is divided by *identifier-1* or *literal-1* (the divisor). The quotient is stored in each *identifier-3*. The identifiers after `GIVING` are receiving items only; their original values do not participate in the division.

```cobol
DIVIDE WS-DIVISOR INTO WS-DIVIDEND GIVING WS-QUOTIENT
*> Equivalent to:
*>   WS-QUOTIENT = WS-DIVIDEND / WS-DIVISOR
```

### Format 3: DIVIDE ... BY ... GIVING

The value of *identifier-1* or *literal-1* (the dividend) is divided by *identifier-2* or *literal-2* (the divisor). The quotient is stored in each *identifier-3*.

```cobol
DIVIDE WS-DIVIDEND BY WS-DIVISOR GIVING WS-QUOTIENT
*> Equivalent to:
*>   WS-QUOTIENT = WS-DIVIDEND / WS-DIVISOR
```

In Formats 2 and 3, the identifiers after `GIVING` may be numeric or numeric-edited items. All other operands must be numeric.

### REMAINDER Clause

The `REMAINDER` clause is available in Format 2 and Format 3 only. It stores the remainder of the division in *identifier-4*, which must be a numeric data item.

The remainder is calculated as:

> remainder = dividend - (quotient * divisor)

The quotient used in this calculation is the truncated (not rounded) integer quotient, regardless of whether `ROUNDED` is specified on the `GIVING` identifier. If `ROUNDED` is specified, the quotient stored in the `GIVING` identifier is rounded, but the remainder is still computed from the truncated integer quotient.

```cobol
DIVIDE WS-DIVIDEND BY WS-DIVISOR
    GIVING WS-QUOTIENT REMAINDER WS-REMAINDER
```

!!! warning "REMAINDER with ROUNDED"
    When both `ROUNDED` and `REMAINDER` are specified, the value stored in the `GIVING` identifier is the rounded quotient, but the remainder is derived from the truncated integer quotient. The remainder value may therefore appear inconsistent with the stored quotient.

### ROUNDED Phrase

When the `ROUNDED` phrase is specified, the quotient is rounded rather than truncated before being stored in the receiving identifier. Without `ROUNDED`, excess decimal digits are truncated.

!!! note "COBOL 2002"
    The `ROUNDED MODE IS` phrase with named rounding modes was introduced in COBOL 2002. Prior standards support only the basic `ROUNDED` keyword, which uses an implementation-defined rounding mode (typically nearest-away-from-zero).

### ON SIZE ERROR

The `ON SIZE ERROR` phrase specifies statements to execute when the quotient exceeds the capacity of a receiving identifier, or when division by zero is attempted. When a size error occurs, the contents of the receiving identifiers (including the `REMAINDER` identifier, if specified) are not modified.

The `NOT ON SIZE ERROR` phrase specifies statements to execute when the operation completes successfully without a size error.

### END-DIVIDE

The `END-DIVIDE` scope terminator delimits the scope of the `DIVIDE` statement. It is required when the `DIVIDE` statement is nested within another statement and an explicit scope terminator is needed to prevent ambiguity.

!!! note "COBOL-85"
    The `END-DIVIDE` scope terminator was introduced in COBOL-85. In earlier standards, the scope of the `ON SIZE ERROR` phrase is implicitly terminated by the next sentence-ending period.

### Multiple Receiving Fields

In Format 1, multiple receiving identifiers may be specified after `INTO`. Each is independently divided by the divisor. In Formats 2 and 3, multiple receiving identifiers may be specified after `GIVING`. The quotient is computed once and stored in each. A size error in one receiving identifier does not affect the others.

When the `REMAINDER` clause is specified, only one identifier may appear after `GIVING`.

### Operand Requirements

All identifiers used as operands (excluding `GIVING` and `REMAINDER` targets) must refer to numeric data items. The [PICTURE](../../data-division/picture.md) clause of each operand determines its decimal point alignment. The [USAGE](../../data-division/usage.md) clause of operands may differ; the compiler performs any necessary conversions.

---

## Behavior

### General

- In Format 1, the divisor is divided into each *identifier-2* independently. Each *identifier-2* receives its own quotient.
- In Formats 2 and 3, the two operands are divided and the quotient is moved to each `GIVING` identifier. The receiving identifiers' original values are not part of the computation.
- When no `ON SIZE ERROR` phrase is specified and a size error occurs, the result in the affected receiving identifier is undefined.

### Division by Zero

Division by zero always causes a size error condition. If the `ON SIZE ERROR` phrase is specified, the imperative statement is executed and the receiving identifiers are not modified. If no `ON SIZE ERROR` phrase is specified, the behavior is undefined --- the program may abnormally terminate, produce unpredictable results, or raise an implementation-defined exception.

!!! warning "Division by Zero"
    Without an `ON SIZE ERROR` phrase, dividing by zero produces undefined results. The program may abend, produce garbage values, or silently continue with corrupted data. Always use `ON SIZE ERROR` when the divisor might be zero.

---

## Examples

### Simple Division in Place

```cobol
01 WS-TOTAL    PIC 9(7)V99  VALUE 0001000.00.
01 WS-DIVISOR  PIC 9(3)     VALUE 004.

DIVIDE WS-DIVISOR INTO WS-TOTAL
*> WS-TOTAL = 0000250.00  (1000.00 / 4)
```

### DIVIDE ... INTO ... GIVING

```cobol
01 WS-TOTAL     PIC 9(7)V99  VALUE 0001000.00.
01 WS-COUNT     PIC 9(3)     VALUE 003.
01 WS-AVERAGE   PIC 9(7)V99.

DIVIDE WS-COUNT INTO WS-TOTAL GIVING WS-AVERAGE
*> WS-AVERAGE = 0000333.33  (1000.00 / 3, truncated)
*> WS-TOTAL is unchanged (still 0001000.00)
```

### DIVIDE ... BY ... GIVING

```cobol
01 WS-DISTANCE  PIC 9(5)V9  VALUE 00350.0.
01 WS-TIME      PIC 9(3)V9  VALUE 005.5.
01 WS-SPEED     PIC 9(5)V99.

DIVIDE WS-DISTANCE BY WS-TIME GIVING WS-SPEED ROUNDED
*> WS-SPEED = 00063.64  (350.0 / 5.5, rounded)
```

### Division with REMAINDER

```cobol
01 WS-DIVIDEND   PIC 9(5)  VALUE 00100.
01 WS-DIVISOR    PIC 9(3)  VALUE 007.
01 WS-QUOTIENT   PIC 9(5).
01 WS-REMAINDER  PIC 9(3).

DIVIDE WS-DIVIDEND BY WS-DIVISOR
    GIVING WS-QUOTIENT REMAINDER WS-REMAINDER
*> WS-QUOTIENT  = 00014  (100 / 7 = 14 remainder 2)
*> WS-REMAINDER = 002    (100 - 14 * 7 = 2)
```

### Division by Zero with SIZE ERROR

```cobol
01 WS-AMOUNT   PIC 9(7)V99  VALUE 0001500.00.
01 WS-DIVISOR  PIC 9(3)     VALUE 000.
01 WS-RESULT   PIC 9(7)V99  VALUE 0000000.00.

DIVIDE WS-AMOUNT BY WS-DIVISOR
    GIVING WS-RESULT
    ON SIZE ERROR
        DISPLAY "Division by zero attempted"
    NOT ON SIZE ERROR
        DISPLAY "Result: " WS-RESULT
END-DIVIDE
*> Displays "Division by zero attempted"
*> WS-RESULT remains 0000000.00
```

### Converting Minutes to Hours and Minutes

```cobol
01 WS-TOTAL-MINUTES  PIC 9(5)  VALUE 00150.
01 WS-HOURS          PIC 9(3).
01 WS-MINUTES        PIC 9(2).

DIVIDE WS-TOTAL-MINUTES BY 60
    GIVING WS-HOURS REMAINDER WS-MINUTES
*> WS-HOURS   = 002  (150 / 60 = 2)
*> WS-MINUTES = 30   (150 - 2 * 60 = 30)
```

### Multiple Receiving Identifiers (Format 1)

```cobol
01 WS-FACTOR     PIC 9(3)     VALUE 002.
01 WS-VALUE-1    PIC 9(7)V99  VALUE 0001000.00.
01 WS-VALUE-2    PIC 9(7)V99  VALUE 0000500.00.

DIVIDE WS-FACTOR INTO WS-VALUE-1 WS-VALUE-2
*> WS-VALUE-1 = 0000500.00  (1000.00 / 2)
*> WS-VALUE-2 = 0000250.00  (500.00 / 2)
```

### Dividing with a Literal

```cobol
01 WS-ANNUAL-SALARY  PIC 9(7)V99  VALUE 0060000.00.
01 WS-MONTHLY-PAY    PIC 9(7)V99.

DIVIDE WS-ANNUAL-SALARY BY 12
    GIVING WS-MONTHLY-PAY ROUNDED
*> WS-MONTHLY-PAY = 0005000.00
```

---

## See Also

- [ADD](add.md) — adds numeric operands
- [SUBTRACT](subtract.md) — subtracts numeric operands
- [MULTIPLY](multiply.md) — multiplies numeric operands
- [COMPUTE](compute.md) — evaluates arithmetic expressions
- [PICTURE](../../data-division/picture.md) — defines data item format and size
- [USAGE](../../data-division/usage.md) — defines internal representation of data items
