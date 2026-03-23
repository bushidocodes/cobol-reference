# Figurative Constants

Figurative constants are reserved words that represent specific predefined values. They may be used in most contexts where a literal is permitted. The compiler generates the appropriate value based on the context and the size of the receiving data item.

**Standard:** COBOL-60, COBOL-61, COBOL-61 Extended, COBOL-65, COBOL-68, COBOL-74, COBOL-85, COBOL 2002, COBOL 2014, COBOL 2023

---

## Summary

| Figurative Constant | Value |
|---|---|
| `ZERO` / `ZEROS` / `ZEROES` | Numeric zero or one or more zero characters |
| `SPACE` / `SPACES` | One or more space characters |
| `HIGH-VALUE` / `HIGH-VALUES` | One or more characters with the highest ordinal value in the current [collating sequence](../appendices/collating-sequence.md) |
| `LOW-VALUE` / `LOW-VALUES` | One or more characters with the lowest ordinal value in the current [collating sequence](../appendices/collating-sequence.md) |
| `QUOTE` / `QUOTES` | One or more quotation mark characters |
| `ALL` literal | The literal value repeated to fill the receiving field |
| `NULL` / `NULLS` | A null pointer value (COBOL 2002+) |

The singular and plural forms of each figurative constant (e.g., `ZERO` and `ZEROS`) are interchangeable. The choice is a matter of readability and has no effect on program behavior.

## ZERO / ZEROS / ZEROES

`ZERO` represents the value zero. Its interpretation depends on context:

- When used as the sending item in a [MOVE](../procedure-division/data-movement/move.md) to a numeric data item, `ZERO` represents the numeric value 0.
- When moved to an alphanumeric data item, `ZERO` fills the item with the character `"0"`.
- When used in a comparison, `ZERO` takes the type and size of the item it is compared against.

```cobol
       01  WS-AMOUNT     PIC 9(5)V99.
       01  WS-NAME       PIC X(10).

       MOVE ZEROS TO WS-AMOUNT
      *> WS-AMOUNT contains numeric 0.00

       MOVE ZEROS TO WS-NAME
      *> WS-NAME contains "0000000000"

       IF WS-AMOUNT = ZERO
           DISPLAY "Amount is zero"
       END-IF
```

## SPACE / SPACES

`SPACE` represents one or more space characters (X'20' in ASCII, X'40' in EBCDIC). When moved to an alphanumeric data item, `SPACE` fills the entire item with spaces.

```cobol
       01  WS-BUFFER     PIC X(80).

       MOVE SPACES TO WS-BUFFER
      *> WS-BUFFER contains 80 space characters

       IF WS-BUFFER = SPACES
           DISPLAY "Buffer is empty"
       END-IF
```

!!! note
    `SPACE` must not be moved to a numeric data item. The result of such a [MOVE](../procedure-division/data-movement/move.md) is undefined and most compilers issue a diagnostic.

## HIGH-VALUE / HIGH-VALUES

`HIGH-VALUE` represents one or more characters with the highest ordinal position in the current program [collating sequence](../appendices/collating-sequence.md). The actual character value depends on the character encoding:

| Encoding | Character | Hexadecimal Value |
|---|---|---|
| ASCII | DEL (or X'FF' in most implementations) | `X'FF'` |
| EBCDIC | X'FF' | `X'FF'` |

In most implementations, `HIGH-VALUE` corresponds to `X'FF'` regardless of encoding. However, if the program collating sequence is redefined via the PROGRAM COLLATING SEQUENCE clause in the OBJECT-COMPUTER paragraph, the character corresponding to `HIGH-VALUE` changes to the character assigned to the highest ordinal position in that sequence.

When moved to an alphanumeric data item, `HIGH-VALUE` fills the entire item with the highest-value character.

```cobol
       01  WS-SENTINEL   PIC X(1).
       01  WS-KEY        PIC X(10).

       MOVE HIGH-VALUES TO WS-SENTINEL
      *> WS-SENTINEL contains X'FF'

       IF WS-KEY < HIGH-VALUE
           DISPLAY "Key is not at maximum"
       END-IF
```

`HIGH-VALUE` is commonly used as a sentinel value in sorting and table processing to mark the logical end of data.

!!! note
    `HIGH-VALUE` must not be moved to a numeric data item.

## LOW-VALUE / LOW-VALUES

`LOW-VALUE` represents one or more characters with the lowest ordinal position in the current program [collating sequence](../appendices/collating-sequence.md):

| Encoding | Character | Hexadecimal Value |
|---|---|---|
| ASCII | NUL | `X'00'` |
| EBCDIC | NUL | `X'00'` |

In most implementations, `LOW-VALUE` corresponds to `X'00'` regardless of encoding. As with `HIGH-VALUE`, if the program collating sequence is redefined, the character corresponding to `LOW-VALUE` changes accordingly.

When moved to an alphanumeric data item, `LOW-VALUE` fills the entire item with the lowest-value character.

```cobol
       01  WS-INIT-KEY   PIC X(10).

       MOVE LOW-VALUES TO WS-INIT-KEY
      *> WS-INIT-KEY contains ten X'00' characters

       IF WS-INIT-KEY = LOW-VALUE
           DISPLAY "Key is at minimum"
       END-IF
```

`LOW-VALUE` is frequently used to initialize record keys and control fields before processing begins.

!!! note
    `LOW-VALUE` must not be moved to a numeric data item.

## QUOTE / QUOTES

`QUOTE` represents one or more quotation mark characters. Whether `QUOTE` produces a double quotation mark (`"`) or an apostrophe (`'`) depends on the compiler's APOST/QUOTE option.

```cobol
       01  WS-QUOTED     PIC X(12).

       STRING QUOTE "Hello" QUOTE DELIMITED BY SIZE
           INTO WS-QUOTED
       END-STRING
      *> WS-QUOTED contains: "Hello"
```

!!! note
    `QUOTE` must not be used to delimit a literal. The expression `MOVE QUOTE TO WS-X` moves a quotation mark character; the expression `MOVE "QUOTE" TO WS-X` moves the five-letter word QUOTE.

## ALL Literal

`ALL` followed by a literal produces a figurative constant consisting of the literal value repeated as many times as necessary to fill the receiving data item.

```cobol
       01  WS-DASHES     PIC X(40).
       01  WS-PATTERN    PIC X(12).

       MOVE ALL "-" TO WS-DASHES
      *> WS-DASHES contains "----------------------------------------"

       MOVE ALL "AB" TO WS-PATTERN
      *> WS-PATTERN contains "ABABABABABAB"
```

When `ALL` is used with a single-character literal, it behaves identically to the other figurative constants: the character fills the receiving field. When used with a multi-character literal, the literal is cyclically repeated to fill the receiving field, truncating the last repetition if necessary.

## NULL / NULLS

`NULL` represents a null pointer value. It is used with data items that have [USAGE](../data-division/usage.md) POINTER, USAGE FUNCTION-POINTER, or USAGE PROGRAM-POINTER.

```cobol
       01  WS-PTR        USAGE POINTER.

       SET WS-PTR TO NULL

       IF WS-PTR = NULL
           DISPLAY "Pointer is null"
       END-IF
```

!!! note
    `NULL` was introduced in COBOL 2002. In COBOL-85, pointer support (where available) is implementation-defined and may use `NULLS` or other mechanisms.

## Behavior in Different Contexts

When a figurative constant is used as a sending item, it conceptually generates a value whose size matches that of the receiving data item. This behavior applies uniformly across [MOVE](../procedure-division/data-movement/move.md) statements, comparisons, and VALUE clauses.

### VALUE Clauses

Figurative constants may appear in [VALUE](../data-division/value.md) clauses of data description entries to specify initial values:

```cobol
       01  WS-STATUS     PIC X(5) VALUE SPACES.
       01  WS-TOTAL      PIC 9(7) VALUE ZEROS.
       01  WS-MARKER     PIC X    VALUE HIGH-VALUE.
```

Figurative constants may also appear in [condition-name](../data-division/condition-names.md) (level-88) VALUE clauses:

```cobol
       01  WS-FLAG       PIC X.
           88  WS-FLAG-EMPTY     VALUE SPACE.
           88  WS-FLAG-CLEAR     VALUE LOW-VALUE.
```

### Conditional Expressions

When a figurative constant appears in a comparison within an [IF](../procedure-division/control-flow/if.md) statement or other conditional expression, it assumes the size and category of the data item it is compared against:

```cobol
       01  WS-CODE       PIC X(3).
       01  WS-COUNT      PIC 9(4).

       IF WS-CODE = SPACES
      *>   Equivalent to: IF WS-CODE = "   "
           DISPLAY "Code is blank"
       END-IF

       IF WS-COUNT = ZERO
      *>   Equivalent to: IF WS-COUNT = 0000
           DISPLAY "Count is zero"
       END-IF
```

### MOVE Statements

When a figurative constant is the sending item in a [MOVE](../procedure-division/data-movement/move.md) statement, it generates a value sized to fill the receiving data item completely:

```cobol
       01  WS-SMALL      PIC X(5).
       01  WS-LARGE      PIC X(100).

       MOVE SPACES TO WS-SMALL WS-LARGE
      *> WS-SMALL receives 5 spaces; WS-LARGE receives 100 spaces
```

### I/O Statements

Figurative constants may appear in WRITE and READ contexts where literals are expected, such as in WRITE FROM or as comparison values in AT END clauses.

## See Also

- [Character Set and Words](character-set.md) -- COBOL character set and literal types
- [Scope Terminators](scope-terminators.md) -- explicit statement termination
- Reserved Words -- complete list of reserved words
- [VALUE Clause](../data-division/value.md) -- initial value specification
- [PICTURE Clause](../data-division/picture.md) -- data item format specification
- [Condition-Names](../data-division/condition-names.md) -- level-88 entries
- [USAGE Clause](../data-division/usage.md) -- data item storage format
- [MOVE](../procedure-division/data-movement/move.md) -- data movement statement
- [Collating Sequence](../appendices/collating-sequence.md) -- character ordering
