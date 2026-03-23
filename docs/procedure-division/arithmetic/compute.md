# COMPUTE

The `COMPUTE` statement evaluates an arithmetic expression and stores the result in one or more receiving data items. It is the most flexible arithmetic statement in COBOL, supporting any combination of arithmetic operators in a single statement.

- **Standard:** COBOL-60, COBOL-61, COBOL-61 Extended, COBOL-65, COBOL-68, COBOL-74, COBOL-85, COBOL 2002, COBOL 2014, COBOL 2023
- **Division:** Procedure Division
- **Category:** Arithmetic

---

## Syntax

```cobol
COMPUTE {identifier-1 [ROUNDED [MODE IS {AWAY-FROM-ZERO
                                        | NEAREST-AWAY-FROM-ZERO
                                        | NEAREST-EVEN
                                        | NEAREST-TOWARD-ZERO
                                        | PROHIBITED
                                        | TOWARD-GREATER
                                        | TOWARD-LESSER
                                        | TRUNCATION}]
        } ...
    = arithmetic-expression-1
    [ON SIZE ERROR imperative-statement-1]
    [NOT ON SIZE ERROR imperative-statement-2]
[END-COMPUTE]
```

---

## Rules

### Arithmetic Expression

The arithmetic expression on the right side of the `=` sign may contain:

- Numeric identifiers
- Numeric literals
- Arithmetic operators
- Parentheses for grouping

### Arithmetic Operators

| Operator | Meaning | Example |
|----------|---------|---------|
| `+` | Addition | `A + B` |
| `-` | Subtraction | `A - B` |
| `*` | Multiplication | `A * B` |
| `/` | Division | `A / B` |
| `**` | Exponentiation | `A ** B` |

The unary `+` and `-` operators are also permitted (e.g., `-A`, `+B`).

### Operator Precedence

Operators are evaluated in the following order (highest precedence first):

1. Unary `+` and `-`
2. `**` (exponentiation)
3. `*` and `/` (multiplication and division)
4. `+` and `-` (addition and subtraction)

Operators of equal precedence are evaluated left to right. Parentheses override precedence.

```cobol
COMPUTE WS-RESULT = A + B * C       *> B * C computed first
COMPUTE WS-RESULT = (A + B) * C     *> A + B computed first
COMPUTE WS-RESULT = A ** B ** C     *> A ** B computed first (left to right)
```

!!! warning "Exponentiation Associativity"
    Unlike many other languages where exponentiation is right-associative, COBOL evaluates `**` left to right. `2 ** 3 ** 2` evaluates as `(2 ** 3) ** 2 = 64`, not `2 ** (3 ** 2) = 512`. Use parentheses to make intent explicit.

### ROUNDED Phrase

When the `ROUNDED` phrase is specified, the result is rounded rather than truncated before being stored in the receiving identifier. Without `ROUNDED`, excess decimal digits are truncated.

```cobol
01 WS-RESULT  PIC 9(3)V99.

COMPUTE WS-RESULT ROUNDED = 100 / 3
*> WS-RESULT = 033.33 (rounded)

COMPUTE WS-RESULT = 100 / 3
*> WS-RESULT = 033.33 (truncated; same in this case)
```

!!! note "COBOL 2002"
    The `ROUNDED MODE IS` phrase with named rounding modes was introduced in COBOL 2002. Prior standards support only the basic `ROUNDED` keyword, which uses an implementation-defined rounding mode (typically nearest-away-from-zero, also known as "round half up").

#### Rounding Modes (COBOL 2002)

| Mode | Description |
|------|-------------|
| `AWAY-FROM-ZERO` | Always rounds away from zero |
| `NEAREST-AWAY-FROM-ZERO` | Rounds to the nearest value; if equidistant, rounds away from zero |
| `NEAREST-EVEN` | Rounds to the nearest value; if equidistant, rounds to the nearest even digit (banker's rounding) |
| `NEAREST-TOWARD-ZERO` | Rounds to the nearest value; if equidistant, rounds toward zero |
| `PROHIBITED` | Raises a size error if rounding would be necessary |
| `TOWARD-GREATER` | Rounds toward positive infinity |
| `TOWARD-LESSER` | Rounds toward negative infinity |
| `TRUNCATION` | Truncates excess digits (equivalent to omitting `ROUNDED`) |

### ON SIZE ERROR

The `ON SIZE ERROR` phrase specifies statements to execute when the result of the arithmetic expression exceeds the capacity of the receiving identifier, or when division by zero occurs.

The `NOT ON SIZE ERROR` phrase specifies statements to execute when the operation completes successfully without a size error.

```cobol
COMPUTE WS-RESULT = WS-A / WS-B
    ON SIZE ERROR
        DISPLAY "Arithmetic overflow or division by zero"
        MOVE 0 TO WS-RESULT
    NOT ON SIZE ERROR
        DISPLAY "Result: " WS-RESULT
END-COMPUTE
```

!!! warning "Division by Zero"
    Without an `ON SIZE ERROR` phrase, dividing by zero produces undefined results. The program may abend, produce garbage, or silently continue. Always use `ON SIZE ERROR` when the divisor might be zero.

### Multiple Receiving Items

Multiple receiving identifiers may be specified. Each receives the result of the same arithmetic expression, subject to its own `ROUNDED` specification and size constraints.

```cobol
COMPUTE WS-ROUNDED-RESULT ROUNDED
        WS-TRUNCATED-RESULT
    = WS-TOTAL / WS-COUNT
END-COMPUTE
```

---

## Behavior

- The arithmetic expression is evaluated first, then the result is stored in each receiving identifier.
- The expression is evaluated using intermediate result precision, which is implementation-defined but must be sufficient to represent the result without loss of significant digits.
- If a size error occurs and the `ON SIZE ERROR` phrase is specified, the receiving identifiers are not modified.
- If no size error phrase is specified and a size error occurs, the result is undefined.

### Intermediate Results

The precision of intermediate results during expression evaluation varies by compiler. Some compilers use a fixed intermediate precision (e.g., 18 or 31 digits); others compute the required precision from the operands.

!!! warning "Intermediate Overflow"
    Even if the final result fits in the receiving item, an intermediate overflow may occur during expression evaluation. For example, `COMPUTE X = (A * B) / C` may overflow on `A * B` even if the final result is small. Some compilers handle this transparently; others raise a size error. Consult the compiler documentation for intermediate result rules.

---

## When to Use COMPUTE vs Individual Arithmetic Statements

| Use Case | Recommended Statement |
|----------|----------------------|
| Complex formula with multiple operations | `COMPUTE` |
| Simple addition to an accumulator | `ADD` |
| Simple subtraction | `SUBTRACT` |
| Single multiplication | `MULTIPLY` |
| Single division with remainder | `DIVIDE` (with `REMAINDER`) |
| Readability of business rule | Depends on context |

`COMPUTE` is generally preferred for any expression involving more than one operator, as it avoids the verbosity of chaining multiple `ADD`, `SUBTRACT`, `MULTIPLY`, and `DIVIDE` statements.

```cobol
*> Using COMPUTE (concise)
COMPUTE WS-TAX = WS-GROSS-PAY * WS-TAX-RATE / 100

*> Equivalent without COMPUTE (verbose)
MULTIPLY WS-GROSS-PAY BY WS-TAX-RATE
    GIVING WS-TEMP
END-MULTIPLY
DIVIDE WS-TEMP BY 100
    GIVING WS-TAX
END-DIVIDE
```

---

## Examples

### Simple Arithmetic

```cobol
01 WS-PRICE     PIC 9(5)V99  VALUE 00150.00.
01 WS-QTY       PIC 9(3)     VALUE 025.
01 WS-TOTAL     PIC 9(7)V99.

COMPUTE WS-TOTAL = WS-PRICE * WS-QTY
*> WS-TOTAL = 0003750.00
```

### Complex Formula with Parentheses

```cobol
01 WS-PRINCIPAL  PIC 9(9)V99.
01 WS-RATE       PIC 9V9(4).
01 WS-YEARS      PIC 99.
01 WS-COMPOUND   PIC 9(12)V99.

*> Compound interest: A = P * (1 + r) ** n
COMPUTE WS-COMPOUND ROUNDED =
    WS-PRINCIPAL * (1 + WS-RATE) ** WS-YEARS
    ON SIZE ERROR
        DISPLAY "Result exceeds field capacity"
    NOT ON SIZE ERROR
        DISPLAY "Future value: " WS-COMPOUND
END-COMPUTE
```

### Division with Size Error Check

```cobol
01 WS-TOTAL    PIC 9(7)V99  VALUE 1000.00.
01 WS-COUNT    PIC 9(5)     VALUE 0.
01 WS-AVERAGE  PIC 9(5)V99.

COMPUTE WS-AVERAGE ROUNDED = WS-TOTAL / WS-COUNT
    ON SIZE ERROR
        DISPLAY "Cannot compute average: count is zero"
        MOVE 0 TO WS-AVERAGE
    NOT ON SIZE ERROR
        DISPLAY "Average: " WS-AVERAGE
END-COMPUTE
```

### Exponentiation

```cobol
01 WS-BASE     PIC 9(3)  VALUE 2.
01 WS-EXP      PIC 9(2)  VALUE 10.
01 WS-POWER    PIC 9(10).

COMPUTE WS-POWER = WS-BASE ** WS-EXP
*> WS-POWER = 0000001024
```

### Multiple Receiving Items

```cobol
01 WS-BONUS-ROUNDED    PIC 9(7)V99.
01 WS-BONUS-TRUNCATED  PIC 9(7)V99.

COMPUTE WS-BONUS-ROUNDED ROUNDED
        WS-BONUS-TRUNCATED
    = WS-SALARY * WS-BONUS-RATE / 100
END-COMPUTE
```

### Using Intrinsic Functions

```cobol
01 WS-A     PIC S9(5)V99  VALUE -123.45.
01 WS-B     PIC S9(5)V99  VALUE 678.90.
01 WS-C     PIC S9(5)V99  VALUE 234.56.
01 WS-MAX   PIC S9(5)V99.
01 WS-ABS   PIC  9(5)V99.

COMPUTE WS-MAX = FUNCTION MAX(WS-A, WS-B, WS-C)
*> WS-MAX = 678.90

COMPUTE WS-ABS = FUNCTION ABS(WS-A)
*> WS-ABS = 123.45
```

### Percentage Calculation

```cobol
01 WS-PART    PIC 9(7)   VALUE 350.
01 WS-WHOLE   PIC 9(7)   VALUE 1000.
01 WS-PCT     PIC 9(3)V99.

COMPUTE WS-PCT ROUNDED = (WS-PART / WS-WHOLE) * 100
*> WS-PCT = 035.00
```

---

## See Also

- [ADD](add.md) — adds numeric operands
- [SUBTRACT](subtract.md) — subtracts numeric operands
- [MULTIPLY](multiply.md) — multiplies numeric operands
- [DIVIDE](divide.md) — divides numeric operands
- [Procedure Division Overview](../index.md)
