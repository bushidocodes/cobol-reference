# Arithmetic Precision

COBOL performs arithmetic using a fixed-point decimal model rather than binary floating-point. This ensures that decimal values are represented exactly, making COBOL well-suited for financial and business calculations where rounding errors from binary representation are unacceptable. The rules governing precision, intermediate results, and rounding behavior have evolved across successive standards.

**Standard:** COBOL-60, COBOL-61, COBOL-61 Extended, COBOL-65, COBOL-68, COBOL-74, COBOL-85, COBOL 2002, COBOL 2014, COBOL 2023

---

## Decimal Arithmetic Model

All standard arithmetic operations in COBOL use a decimal (base-10) arithmetic model. Numeric data items declared with a [PICTURE](../data-division/picture.md) clause containing `9`, `V`, `S`, and `P` symbols store values as decimal digits. Arithmetic on these items produces exact decimal results, subject only to the precision limits of the receiving item and any intermediate result fields.

This contrasts with languages that use binary floating-point (IEEE 754 binary) by default, where values like `0.1` cannot be represented exactly. In COBOL, `PIC 9V9 VALUE 0.1` stores exactly one-tenth.

!!! note "USAGE Clause Interaction"
    The [USAGE](../data-division/usage.md) clause determines the internal storage format (DISPLAY, COMP, COMP-3, etc.), but the decimal arithmetic model applies regardless of usage. The compiler converts between internal representations and the decimal model as needed during arithmetic operations.

---

## Maximum Significant Digits

The maximum number of decimal digits that a numeric data item can hold is governed by the standard and the compiler implementation.

| Standard | Maximum Digits | Notes |
|----------|---------------|-------|
| COBOL-68 through COBOL-85 | 18 | The [PICTURE](../data-division/picture.md) string may contain at most 18 digit positions (`9` symbols) |
| COBOL 2002 | Implementation-defined (at least 18) | Most compilers support 18 or 31 digits |
| COBOL 2014, COBOL 2023 | Implementation-defined (at least 18) | 31-digit support is common in modern compilers |

A data item declared as `PIC 9(18)V9(18)` is invalid under COBOL-85 because it specifies 36 digit positions. The total number of `9` symbols (including those implied by `P`) must not exceed the maximum.

---

## Intermediate Result Precision

When the [COMPUTE](../procedure-division/arithmetic/compute.md) statement evaluates a compound arithmetic expression, the compiler generates intermediate results for each sub-operation. The precision of these intermediate results determines whether a calculation succeeds or loses significant digits.

### Rules for Intermediate Results

The COBOL standard does not prescribe a single algorithm for intermediate result precision. Instead, it requires that intermediate results have sufficient precision to avoid loss of significant digits, subject to implementation-defined limits. In practice, compilers use one of two approaches:

1. **Fixed intermediate precision** --- The compiler uses a fixed number of digits (commonly 18 or 31) for all intermediate results, regardless of the operands.

2. **Computed intermediate precision** --- The compiler analyzes the PICTURE clauses of the operands and calculates the minimum number of integer and decimal digits required for each intermediate result.

!!! warning "Intermediate Overflow"
    Even if the final result fits within the receiving item, an intermediate result may overflow. For example, in `COMPUTE X = (A * B) / C`, the product `A * B` may exceed the intermediate result capacity even though the final quotient is small. This can cause a size error or silent truncation, depending on the compiler and whether ON SIZE ERROR is specified.

### Example: Intermediate Precision Loss

```cobol
01 WS-A      PIC 9(10)V99   VALUE 9999999999.99.
01 WS-B      PIC 9(10)V99   VALUE 9999999999.99.
01 WS-C      PIC 9(10)V99   VALUE 9999999999.99.
01 WS-RESULT PIC 9(10)V99.

*> The product A * B produces up to 24 digits.
*> On a compiler with 18-digit intermediates, this overflows.
COMPUTE WS-RESULT = (WS-A * WS-B) / WS-C
    ON SIZE ERROR
        DISPLAY "Intermediate overflow occurred"
END-COMPUTE
```

To avoid intermediate overflow, restructure the expression:

```cobol
*> Dividing first keeps intermediate values smaller.
COMPUTE WS-RESULT = (WS-A / WS-C) * WS-B
    ON SIZE ERROR
        DISPLAY "Overflow occurred"
END-COMPUTE
```

---

## Precision of Individual Arithmetic Statements vs COMPUTE

The individual arithmetic statements ([ADD](../procedure-division/arithmetic/add.md), [SUBTRACT](../procedure-division/arithmetic/subtract.md), [MULTIPLY](../procedure-division/arithmetic/multiply.md), [DIVIDE](../procedure-division/arithmetic/divide.md)) each perform a single operation with well-defined precision rules. The [COMPUTE](../procedure-division/arithmetic/compute.md) statement, by contrast, evaluates an arbitrarily complex expression and must manage intermediate results.

### Individual Statements

For `ADD` and `SUBTRACT`, the result has enough integer digits to hold the sum or difference and enough decimal digits to hold the operand with the most decimal places. The final result is then stored in the receiving item, with truncation or rounding applied.

For `MULTIPLY`, the number of integer digits in the intermediate result equals the sum of the integer digits of the two operands, and the number of decimal digits equals the sum of the decimal digits.

For `DIVIDE`, the intermediate quotient has a number of decimal digits that is implementation-defined but sufficient for the precision of the receiving item.

### COMPUTE Statements

The `COMPUTE` statement evaluates its expression left to right (respecting operator precedence), and each intermediate sub-result follows the rules of the corresponding operation. The key difference is that chained operations accumulate precision requirements, increasing the risk of intermediate overflow.

```cobol
*> Individual statements: each intermediate is well-bounded.
MULTIPLY WS-RATE BY WS-HOURS GIVING WS-GROSS
ADD WS-BONUS TO WS-GROSS
SUBTRACT WS-DEDUCTIONS FROM WS-GROSS GIVING WS-NET

*> COMPUTE: single expression, intermediate precision must cover all steps.
COMPUTE WS-NET = WS-RATE * WS-HOURS + WS-BONUS
                 - WS-DEDUCTIONS
```

!!! note
    For simple operations, individual arithmetic statements and `COMPUTE` produce identical results. The distinction matters primarily for complex expressions where intermediate precision is a concern.

---

## Decimal Point Alignment

When arithmetic is performed on operands with different numbers of decimal places, COBOL aligns values on their decimal points before the operation. This alignment is sometimes called "inner decimal point alignment."

### Alignment Rules

1. Each operand is conceptually extended with leading or trailing zeros so that all operands have the same number of integer and decimal positions.
2. The operation is performed on the aligned values.
3. The result is stored in the receiving item, which may truncate high-order integer digits (if the item is too small) or low-order decimal digits (if the item has fewer decimal places).

### Example

```cobol
01 WS-A      PIC 9(3)V9      VALUE 123.4.
01 WS-B      PIC 9(2)V9(3)   VALUE 45.678.
01 WS-RESULT PIC 9(3)V99.

ADD WS-A TO WS-B GIVING WS-RESULT
*> Alignment:
*>   WS-A  =  123.400
*>   WS-B  =  045.678
*>   Sum   =  169.078
*> Stored in WS-RESULT (PIC 9(3)V99):  169.07  (truncated)
```

---

## Truncation vs Rounding

When the result of an arithmetic operation has more decimal digits than the receiving item can hold, the excess digits are either truncated or rounded, depending on whether the `ROUNDED` phrase is specified.

### Truncation (Default)

Without the `ROUNDED` phrase, excess low-order decimal digits are simply discarded. No adjustment is made to the remaining digits.

```cobol
01 WS-RESULT PIC 9(3)V99.

COMPUTE WS-RESULT = 100 / 3
*> Exact result:  033.33333...
*> Stored:        033.33  (truncated)
```

### ROUNDED Phrase

When `ROUNDED` is specified, the absolute value of the excess portion is compared to half of the value of the last retained digit position. If the excess equals or exceeds half, the last retained digit is incremented by one.

```cobol
01 WS-RESULT PIC 9(3)V99.

COMPUTE WS-RESULT ROUNDED = 100 / 3
*> Exact result:  033.33333...
*> Stored:        033.33  (excess 0.00333... < 0.005, so no rounding up)

COMPUTE WS-RESULT ROUNDED = 200 / 3
*> Exact result:  066.66666...
*> Stored:        066.67  (excess 0.00666... >= 0.005, so rounded up)
```

### Rounding Modes (COBOL 2002+)

COBOL 2002 introduced the `ROUNDED MODE IS` phrase, providing explicit control over rounding behavior. The following modes are available:

| Mode | Behavior |
|------|----------|
| `AWAY-FROM-ZERO` | Always rounds away from zero. `1.225` with 2 decimal places becomes `1.23`; `-1.225` becomes `-1.23`. |
| `NEAREST-AWAY-FROM-ZERO` | Rounds to nearest; ties round away from zero. This is the traditional "round half up" behavior. |
| `NEAREST-EVEN` | Rounds to nearest; ties round to the nearest even digit. Also known as banker's rounding. `1.225` becomes `1.22`; `1.235` becomes `1.24`. |
| `NEAREST-TOWARD-ZERO` | Rounds to nearest; ties round toward zero. `1.225` becomes `1.22`; `-1.225` becomes `-1.22`. |
| `PROHIBITED` | A size error is raised if the result has any excess digits that would require rounding. |
| `TOWARD-GREATER` | Rounds toward positive infinity. `1.221` becomes `1.23`; `-1.229` becomes `-1.22`. |
| `TOWARD-LESSER` | Rounds toward negative infinity. `1.229` becomes `1.22`; `-1.221` becomes `-1.23`. |
| `TRUNCATION` | Discards excess digits. Equivalent to omitting `ROUNDED`. |

```cobol
COMPUTE WS-RESULT ROUNDED MODE IS NEAREST-EVEN
    = WS-TOTAL / WS-COUNT
END-COMPUTE
```

!!! note
    Prior to COBOL 2002, the basic `ROUNDED` keyword uses an implementation-defined rounding mode, typically equivalent to NEAREST-AWAY-FROM-ZERO.

---

## ON SIZE ERROR

A size error occurs when the result of an arithmetic operation cannot be stored in the receiving data item because the result has more integer digits than the receiving item provides. Division by zero also constitutes a size error.

### What Constitutes a Size Error

- **Integer overflow:** The magnitude of the result exceeds the capacity of the receiving item's integer portion. For example, storing the value `1000` in `PIC 9(3)V99` causes a size error because only three integer digits are available.
- **Division by zero:** Any division where the divisor is zero.
- **PROHIBITED rounding mode:** In COBOL 2002+, if the `ROUNDED MODE IS PROHIBITED` phrase is specified and the result would require rounding.

### Behavior

When the ON SIZE ERROR phrase is specified and a size error occurs:

1. The value of the receiving item is **not modified**.
2. The imperative statement following ON SIZE ERROR is executed.

When ON SIZE ERROR is **not** specified and a size error occurs, the result is **undefined**. The receiving item may contain a truncated value, garbage, or the program may abend.

```cobol
01 WS-SMALL  PIC 99V99.

COMPUTE WS-SMALL = 999.99
    ON SIZE ERROR
        DISPLAY "Value too large for WS-SMALL"
    NOT ON SIZE ERROR
        DISPLAY "Stored: " WS-SMALL
END-COMPUTE
*> Displays "Value too large for WS-SMALL"
*> WS-SMALL is unchanged
```

!!! warning
    A size error is **not** raised for loss of decimal precision. Storing `1.23456` in `PIC 9V99` stores `1.23` without a size error --- only the excess decimal digits are lost. Size errors apply only to the integer portion.

---

## IEEE 754 Floating-Point Types (COBOL 2014)

COBOL 2014 introduced three floating-point [USAGE](../data-division/usage.md) types that conform to the IEEE 754 standard. These types supplement the traditional fixed-point decimal model for use cases that require very large ranges or scientific computation.

| USAGE | IEEE 754 Format | Approximate Range | Significant Digits |
|-------|----------------|-------------------|-------------------|
| `FLOAT-BINARY-32` | Binary32 (single) | ~1.2 x 10^-38 to ~3.4 x 10^38 | ~7 decimal digits |
| `FLOAT-BINARY-64` | Binary64 (double) | ~2.2 x 10^-308 to ~1.8 x 10^308 | ~15 decimal digits |
| `FLOAT-BINARY-128` | Binary128 (quad) | ~3.4 x 10^-4932 to ~1.2 x 10^4932 | ~34 decimal digits |
| `FLOAT-DECIMAL-16` | Decimal64 | ~1 x 10^-383 to ~9.999... x 10^384 | 16 decimal digits |
| `FLOAT-DECIMAL-34` | Decimal128 | ~1 x 10^-6143 to ~9.999... x 10^6144 | 34 decimal digits |
| `FLOAT-EXTENDED` | Implementation-defined | Implementation-defined | Implementation-defined |

```cobol
01 WS-SCIENTIFIC   USAGE FLOAT-BINARY-64.
01 WS-FINANCIAL    USAGE FLOAT-DECIMAL-34.
```

!!! warning "Binary Floating-Point Precision"
    The FLOAT-BINARY types use binary floating-point and are subject to the same representation errors as `double` in C or Java. The value `0.1` cannot be represented exactly in FLOAT-BINARY-64. For financial calculations, use fixed-point decimal items or FLOAT-DECIMAL types.

!!! note "COBOL 2014"
    The IEEE 754 floating-point types were introduced in COBOL 2014. They are not available in COBOL 2002 or earlier standards.

---

## Maximum PICTURE Sizes and Digit Capacities

The following table shows common [PICTURE](../data-division/picture.md) patterns and their maximum representable values. The total number of digit positions (`9` symbols) determines the precision of the item.

| PICTURE | Integer Digits | Decimal Digits | Total Digits | Maximum Value |
|---------|---------------|----------------|-------------|---------------|
| `9(4)` | 4 | 0 | 4 | 9999 |
| `9(9)` | 9 | 0 | 9 | 999,999,999 |
| `9(18)` | 18 | 0 | 18 | 999,999,999,999,999,999 |
| `9(5)V99` | 5 | 2 | 7 | 99999.99 |
| `9(7)V9(4)` | 7 | 4 | 11 | 9999999.9999 |
| `S9(9)V99` | 9 | 2 | 11 | +999,999,999.99 |
| `S9(15)V9(3)` | 15 | 3 | 18 | +999,999,999,999,999.999 |
| `S9(13)V9(5)` | 13 | 5 | 18 | +9,999,999,999,999.99999 |
| `9(31)` | 31 | 0 | 31 | 9,999,999,999,999,999,999,999,999,999,999 |

!!! note
    Under COBOL-85, the total digit count must not exceed 18. Under COBOL 2002 and later, the maximum is implementation-defined but commonly 31 or 38.

### Scaling with P Symbol

The `P` symbol represents an assumed decimal scaling position that is not stored. It extends the range of a numeric item without consuming storage.

```cobol
01 WS-LARGE  PIC 9(5)PPP.
*> Represents values from 0 to 99999000 (last 3 digits always zero)
*> Stored as 5 digits; effective value is multiplied by 1000

01 WS-TINY   PIC PPP9(5).
*> Represents values from 0 to 0.00099999
*> Stored as 5 digits; effective value is divided by 100000000
```

Each `P` counts toward the 18-digit (or implementation-defined) limit.

---

## Practical Examples

### Avoiding Precision Loss in Division

```cobol
01 WS-AMOUNT   PIC 9(7)V99     VALUE 1000000.00.
01 WS-DIVISOR  PIC 9(3)        VALUE 3.
01 WS-RESULT   PIC 9(7)V9(4).

*> Result field has 4 decimal places for better precision.
COMPUTE WS-RESULT = WS-AMOUNT / WS-DIVISOR
*> WS-RESULT = 0333333.3333  (truncated after 4 decimal places)

COMPUTE WS-RESULT ROUNDED = WS-AMOUNT / WS-DIVISOR
*> WS-RESULT = 0333333.3333  (same here; 5th digit is 3, < 5)
```

### Demonstrating Rounding Mode Differences (COBOL 2002)

```cobol
01 WS-VAL      PIC 9V9(4)      VALUE 1.2250.
01 WS-ROUND-A  PIC 9V99.
01 WS-ROUND-B  PIC 9V99.

COMPUTE WS-ROUND-A ROUNDED MODE IS NEAREST-AWAY-FROM-ZERO
    = WS-VAL
*> WS-ROUND-A = 1.23  (tie broken away from zero)

COMPUTE WS-ROUND-B ROUNDED MODE IS NEAREST-EVEN
    = WS-VAL
*> WS-ROUND-B = 1.22  (tie broken to nearest even digit)
```

### Detecting Intermediate Overflow

```cobol
01 WS-BIG-A    PIC 9(10)       VALUE 9999999999.
01 WS-BIG-B    PIC 9(10)       VALUE 9999999999.
01 WS-SMALL-C  PIC 9(10)       VALUE 9999999999.
01 WS-OUT      PIC 9(10).

*> This may cause an intermediate overflow on A * B,
*> even though the final result is approximately equal
*> to WS-BIG-A.
COMPUTE WS-OUT = (WS-BIG-A * WS-BIG-B) / WS-SMALL-C
    ON SIZE ERROR
        DISPLAY "Intermediate overflow"
END-COMPUTE

*> Restructured to avoid large intermediates:
COMPUTE WS-OUT = WS-BIG-A * (WS-BIG-B / WS-SMALL-C)
    ON SIZE ERROR
        DISPLAY "Overflow"
END-COMPUTE
*> WS-OUT = 9999999999 (approximately)
```

### PICTURE Size Impact on Calculation Accuracy

```cobol
01 WS-PRICE    PIC 9(5)V99      VALUE 99999.99.
01 WS-QTY      PIC 9(5)         VALUE 99999.
01 WS-TOTAL    PIC 9(7)V99.
01 WS-TOTAL-LG PIC 9(10)V99.

*> WS-TOTAL is too small: 99999.99 * 99999 = 9999899900.01
*> This exceeds PIC 9(7)V99 (max 9999999.99).
COMPUTE WS-TOTAL = WS-PRICE * WS-QTY
    ON SIZE ERROR
        DISPLAY "Result exceeds field capacity"
END-COMPUTE

*> WS-TOTAL-LG is large enough to hold the result.
COMPUTE WS-TOTAL-LG = WS-PRICE * WS-QTY
    ON SIZE ERROR
        DISPLAY "Overflow"
    NOT ON SIZE ERROR
        DISPLAY "Total: " WS-TOTAL-LG
END-COMPUTE
*> Displays: Total: 9999899900.01
```

---

## See Also

- [PICTURE Clause](../data-division/picture.md) --- defines numeric precision and format
- [USAGE Clause](../data-division/usage.md) --- specifies internal storage representation
- [COMPUTE](../procedure-division/arithmetic/compute.md) --- evaluates arithmetic expressions
- [ADD](../procedure-division/arithmetic/add.md) --- adds numeric operands
- [SUBTRACT](../procedure-division/arithmetic/subtract.md) --- subtracts numeric operands
- [MULTIPLY](../procedure-division/arithmetic/multiply.md) --- multiplies numeric operands
- [DIVIDE](../procedure-division/arithmetic/divide.md) --- divides numeric operands
- [Intrinsic Functions](../intrinsic-functions/index.md) --- built-in numeric and mathematical functions
