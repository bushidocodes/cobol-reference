# ADD

The `ADD` statement adds two or more numeric operands and stores the result. It supports adding to an accumulator in place, storing the sum in a separate receiving item, and adding corresponding fields between group items.

- **Standard:** COBOL-60, COBOL-61, COBOL-61 Extended, COBOL-65, COBOL-68, COBOL-74, COBOL-85, COBOL 2002, COBOL 2014, COBOL 2023
- **Division:** Procedure Division
- **Category:** Arithmetic

---

## Syntax

### Format 1: ADD ... TO

```cobol
ADD {identifier-1 | literal-1} ...
    TO {identifier-2 [ROUNDED [MODE IS rounding-mode]]} ...
    [ON SIZE ERROR imperative-statement-1]
    [NOT ON SIZE ERROR imperative-statement-2]
[END-ADD]
```

### Format 2: ADD ... GIVING

```cobol
ADD {identifier-1 | literal-1} ...
    TO {identifier-2 | literal-2}
    GIVING {identifier-3 [ROUNDED [MODE IS rounding-mode]]} ...
    [ON SIZE ERROR imperative-statement-1]
    [NOT ON SIZE ERROR imperative-statement-2]
[END-ADD]
```

### Format 3: ADD CORRESPONDING

```cobol
ADD CORRESPONDING identifier-1
    TO identifier-2 [ROUNDED [MODE IS rounding-mode]]
    [ON SIZE ERROR imperative-statement-1]
    [NOT ON SIZE ERROR imperative-statement-2]
[END-ADD]
```

Where *rounding-mode* is one of: `AWAY-FROM-ZERO`, `NEAREST-AWAY-FROM-ZERO`, `NEAREST-EVEN`, `NEAREST-TOWARD-ZERO`, `PROHIBITED`, `TOWARD-GREATER`, `TOWARD-LESSER`, `TRUNCATION`.

---

## Rules

### Format 1: ADD ... TO

All identifiers and literals preceding the `TO` keyword are added together, and each sum is then added to the current value of each identifier following `TO`. Each identifier after `TO` is both an operand and a receiving item.

```cobol
ADD WS-A WS-B TO WS-C WS-D
*> Equivalent to:
*>   WS-C = WS-A + WS-B + WS-C
*>   WS-D = WS-A + WS-B + WS-D
```

### Format 2: ADD ... GIVING

All identifiers and literals preceding and following the `TO` keyword are added together, and the sum is stored in each identifier following the `GIVING` keyword. The identifiers after `GIVING` are receiving items only; their original values do not participate in the addition.

```cobol
ADD WS-A WS-B TO WS-C GIVING WS-D WS-E
*> Equivalent to:
*>   WS-D = WS-A + WS-B + WS-C
*>   WS-E = WS-A + WS-B + WS-C
```

In Format 2, the identifiers after `GIVING` may be numeric or numeric-edited items. All other operands must be numeric.

### Format 3: ADD CORRESPONDING

The `CORRESPONDING` (or `CORR`) phrase causes the compiler to add each elementary numeric item in *identifier-1* to the corresponding elementary numeric item in *identifier-2*, based on matching data-names. Both identifiers must be group items.

Two items correspond when they have the same data-name (ignoring qualifiers unique to their respective groups), and both items (or the items subordinate to them) are elementary numeric items. Items with `REDEFINES`, `RENAMES`, `OCCURS`, or `INDEX` clauses, and items subordinate to such items, are excluded from correspondence.

```cobol
ADD CORRESPONDING WS-OLD-TOTALS TO WS-NEW-TOTALS
```

### ROUNDED Phrase

When the `ROUNDED` phrase is specified, the result is rounded rather than truncated before being stored in the receiving identifier. Without `ROUNDED`, excess decimal digits are truncated.

!!! note "COBOL 2002"
    The `ROUNDED MODE IS` phrase with named rounding modes was introduced in COBOL 2002. Prior standards support only the basic `ROUNDED` keyword, which uses an implementation-defined rounding mode (typically nearest-away-from-zero).

### ON SIZE ERROR

The `ON SIZE ERROR` phrase specifies statements to execute when the result of the addition exceeds the capacity of a receiving identifier. When a size error occurs, the content of the receiving identifier that caused the error is not modified.

The `NOT ON SIZE ERROR` phrase specifies statements to execute when the operation completes successfully without a size error.

### END-ADD

The `END-ADD` scope terminator delimits the scope of the `ADD` statement. It is required when the `ADD` statement is nested within another statement and an explicit scope terminator is needed to prevent ambiguity.

!!! note "COBOL-85"
    The `END-ADD` scope terminator was introduced in COBOL-85. In earlier standards, the scope of the `ON SIZE ERROR` phrase is implicitly terminated by the next sentence-ending period.

### Multiple Receiving Fields

In Format 1 and Format 2, multiple receiving identifiers may be specified. The sum is computed once and then stored in each receiving identifier independently. A size error in one receiving identifier does not affect the others.

### Operand Requirements

All identifiers used as operands (excluding `GIVING` targets in Format 2) must refer to numeric data items. The [PICTURE](../../data-division/picture.md) clause of each operand determines its decimal point alignment. The [USAGE](../../data-division/usage.md) clause of operands may differ; the compiler performs any necessary conversions.

---

## Behavior

- In Format 1, the addends (identifiers and literals before `TO`) are summed first. That intermediate sum is then added to the current value of each receiving identifier after `TO`.
- In Format 2, all operands (before and after `TO`) are summed, and the result is moved to each `GIVING` identifier. The receiving identifiers' original values are not part of the computation.
- In Format 3, each pair of corresponding items is added independently. A size error on one pair does not prevent addition of other pairs.
- When no `ON SIZE ERROR` phrase is specified and a size error occurs, the result in the affected receiving identifier is undefined.

---

## Examples

### Simple Addition to an Accumulator

```cobol
01 WS-TOTAL     PIC 9(7)V99  VALUE 1000.00.
01 WS-AMOUNT    PIC 9(5)V99  VALUE 00250.75.

ADD WS-AMOUNT TO WS-TOTAL
*> WS-TOTAL = 0001250.75
```

### Adding Multiple Values

```cobol
01 WS-A  PIC 9(3)  VALUE 010.
01 WS-B  PIC 9(3)  VALUE 020.
01 WS-C  PIC 9(3)  VALUE 030.
01 WS-D  PIC 9(5)  VALUE 00100.

ADD WS-A WS-B WS-C TO WS-D
*> WS-D = 00160  (10 + 20 + 30 + 100)
```

### ADD ... GIVING with Numeric-Edited Receiving Item

```cobol
01 WS-PRICE     PIC 9(5)V99  VALUE 00100.00.
01 WS-TAX       PIC 9(5)V99  VALUE 00008.50.
01 WS-TOTAL     PIC $ZZZ,ZZ9.99.

ADD WS-PRICE TO WS-TAX GIVING WS-TOTAL
*> WS-TOTAL = "    $108.50"
```

### ADD with Literals

```cobol
01 WS-COUNTER  PIC 9(5)  VALUE 00042.

ADD 1 TO WS-COUNTER
*> WS-COUNTER = 00043
```

### ADD CORRESPONDING

```cobol
01 WS-MONTHLY-SALES.
   05 WS-PRODUCT-A  PIC 9(7)V99  VALUE 0005000.00.
   05 WS-PRODUCT-B  PIC 9(7)V99  VALUE 0003200.00.
   05 WS-PRODUCT-C  PIC 9(7)V99  VALUE 0001800.00.

01 WS-YEARLY-SALES.
   05 WS-PRODUCT-A  PIC 9(9)V99  VALUE 000050000.00.
   05 WS-PRODUCT-B  PIC 9(9)V99  VALUE 000038000.00.
   05 WS-PRODUCT-C  PIC 9(9)V99  VALUE 000021000.00.

ADD CORRESPONDING WS-MONTHLY-SALES TO WS-YEARLY-SALES
*> WS-YEARLY-SALES:
*>   WS-PRODUCT-A = 000055000.00
*>   WS-PRODUCT-B = 000041200.00
*>   WS-PRODUCT-C = 000022800.00
```

### ADD with ROUNDED and SIZE ERROR

```cobol
01 WS-A       PIC 9(3)V99  VALUE 999.99.
01 WS-B       PIC 9(3)V99  VALUE 000.02.
01 WS-RESULT  PIC 9(3)V99.

ADD WS-A WS-B GIVING WS-RESULT ROUNDED
    ON SIZE ERROR
        DISPLAY "Overflow in addition"
        MOVE 0 TO WS-RESULT
    NOT ON SIZE ERROR
        DISPLAY "Result: " WS-RESULT
END-ADD
*> Displays "Overflow in addition" because 1000.01 exceeds PIC 9(3)V99
```

### Multiple Receiving Identifiers

```cobol
01 WS-BONUS     PIC 9(5)V99  VALUE 01500.00.
01 WS-PAY-1     PIC 9(7)V99  VALUE 0045000.00.
01 WS-PAY-2     PIC 9(7)V99  VALUE 0052000.00.

ADD WS-BONUS TO WS-PAY-1 WS-PAY-2
*> WS-PAY-1 = 0046500.00
*> WS-PAY-2 = 0053500.00
```

---

## See Also

- [SUBTRACT](subtract.md) — subtracts numeric operands
- [MULTIPLY](multiply.md) — multiplies numeric operands
- [DIVIDE](divide.md) — divides numeric operands
- [COMPUTE](compute.md) — evaluates arithmetic expressions
- [PICTURE](../../data-division/picture.md) — defines data item format and size
- [USAGE](../../data-division/usage.md) — defines internal representation of data items
