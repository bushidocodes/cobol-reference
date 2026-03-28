# Numeric Conversion Functions

Intrinsic functions that convert alphanumeric strings to numeric values.

**Standard:** COBOL-85 Amendment 1 (1989), COBOL 2002

---

## General Syntax

```cobol
FUNCTION function-name ( argument-1 [ argument-2 ] )
```

Numeric conversion functions accept alphanumeric string arguments and return **numeric** values. They are used to convert human-readable numeric strings (such as those received from screens, files, or external systems) into numeric values that can participate in arithmetic operations.

---

## NUMVAL

Returns the numeric value of an alphanumeric string in simple format.

```cobol
FUNCTION NUMVAL ( argument-1 )
```

- **argument-1** -- alphanumeric. A string containing a numeric value in simple format.
- **Return category** -- numeric.
- **Behavior** -- parses argument-1 as a numeric value and returns its numeric equivalent. The string may contain leading and trailing spaces, a leading or trailing sign (+ or -), and a decimal point. The string must represent a valid numeric value.

### Accepted Formats

The argument may be in any of the following formats (where `s` is an optional sign, `9` represents digits, and `b` represents optional spaces):

| Format | Example |
|--------|---------|
| `b s 9 b` | `"  -42  "` |
| `b s 9.9 b` | `"  +123.45  "` |
| `b 9 s b` | `"  42-  "` |
| `b 9.9 s b` | `"  123.45+  "` |

### Examples

```cobol
COMPUTE WS-VALUE = FUNCTION NUMVAL("  123.45  ")
*> WS-VALUE = 123.45

COMPUTE WS-VALUE = FUNCTION NUMVAL("-42")
*> WS-VALUE = -42

COMPUTE WS-VALUE = FUNCTION NUMVAL("  100.00+  ")
*> WS-VALUE = 100.00

COMPUTE WS-VALUE = FUNCTION NUMVAL("50.25-")
*> WS-VALUE = -50.25
```

### Practical Example

```cobol
       WORKING-STORAGE SECTION.
       01  WS-INPUT        PIC X(20).
       01  WS-AMOUNT       PIC S9(7)V99.

       PROCEDURE DIVISION.
           ACCEPT WS-INPUT FROM CONSOLE
           COMPUTE WS-AMOUNT = FUNCTION NUMVAL(WS-INPUT)
           IF WS-AMOUNT > 0
               DISPLAY "Positive amount: " WS-AMOUNT
           ELSE
               DISPLAY "Negative or zero: " WS-AMOUNT
           END-IF
           STOP RUN.
```

---

## NUMVAL-C

Returns the numeric value of an alphanumeric string that may contain a currency sign and optional comma separators.

```cobol
FUNCTION NUMVAL-C ( argument-1 [ argument-2 ] )
```

- **argument-1** -- alphanumeric. A string containing a numeric value that may include a currency sign and comma separators.
- **argument-2** -- optional, alphanumeric. The currency sign string to recognize. Defaults to the currency sign specified in the SPECIAL-NAMES paragraph, or `"$"` if none is specified.
- **Return category** -- numeric.
- **Behavior** -- parses argument-1 as a numeric value, ignoring any occurrences of the currency sign (argument-2) and commas. The currency sign may appear as a leading or trailing character. Commas may appear between digits as thousands separators. The string may also contain leading/trailing spaces and a sign.

### Accepted Formats

The argument may contain any of the elements accepted by NUMVAL, plus:

- A currency sign (before or after the digits)
- Commas between digit groups

| Format | Example |
|--------|---------|
| `$9,999.99` | `"$1,234.56"` |
| `9,999.99$` | `"1,234.56$"` |
| `$9,999.99-` | `"$1,234.56-"` |
| `-$9,999.99` | `"-$1,234.56"` |

### Examples

```cobol
COMPUTE WS-VALUE = FUNCTION NUMVAL-C("$1,234.56")
*> WS-VALUE = 1234.56

COMPUTE WS-VALUE = FUNCTION NUMVAL-C("$1,234.56-")
*> WS-VALUE = -1234.56

COMPUTE WS-VALUE = FUNCTION NUMVAL-C("  $100,000.00  ")
*> WS-VALUE = 100000.00

*> Using a custom currency sign
COMPUTE WS-VALUE = FUNCTION NUMVAL-C("EUR 1.234,50" "EUR")
*> Note: behavior with non-$ currencies depends on implementation
```

### Practical Example

```cobol
       WORKING-STORAGE SECTION.
       01  WS-FILE-REC.
           05  WS-ACCT-NO    PIC X(10).
           05  WS-AMOUNT-STR PIC X(15).
       01  WS-AMOUNT         PIC S9(7)V99.
       01  WS-TOTAL          PIC S9(9)V99 VALUE 0.

       PROCEDURE DIVISION.
           OPEN INPUT TRANS-FILE
           PERFORM UNTIL WS-EOF
               READ TRANS-FILE INTO WS-FILE-REC
                   AT END SET WS-EOF TO TRUE
                   NOT AT END
                       COMPUTE WS-AMOUNT =
                           FUNCTION NUMVAL-C(WS-AMOUNT-STR)
                       ADD WS-AMOUNT TO WS-TOTAL
               END-READ
           END-PERFORM
           CLOSE TRANS-FILE
           DISPLAY "Total: $" WS-TOTAL
           STOP RUN.
```

---

## NUMVAL-F

Returns the numeric value of an alphanumeric string in floating-point format.

!!! note "COBOL 2002"
    The NUMVAL-F function was introduced in COBOL 2002.

```cobol
FUNCTION NUMVAL-F ( argument-1 )
```

- **argument-1** -- alphanumeric. A string containing a numeric value in floating-point (scientific notation) format.
- **Return category** -- numeric.
- **Behavior** -- parses argument-1 as a floating-point numeric value and returns its numeric equivalent. The string must contain a mantissa and an exponent separated by "E" or "e". The mantissa and exponent may each have an optional sign.

### Accepted Formats

| Format | Example |
|--------|---------|
| `s 9.9 E s 9` | `"1.5E+02"` |
| `s 9.9 e s 9` | `"-3.14e+00"` |
| `s 9 E s 9` | `"5E-3"` |

The mantissa may be an integer or have a decimal point. The exponent indicator may be uppercase `E` or lowercase `e`. Leading and trailing spaces are allowed.

### Examples

```cobol
COMPUTE WS-VALUE = FUNCTION NUMVAL-F("1.5E+02")
*> WS-VALUE = 150.0

COMPUTE WS-VALUE = FUNCTION NUMVAL-F("-3.14E+00")
*> WS-VALUE = -3.14

COMPUTE WS-VALUE = FUNCTION NUMVAL-F("5E-3")
*> WS-VALUE = 0.005

COMPUTE WS-VALUE = FUNCTION NUMVAL-F("  6.022E+23  ")
*> WS-VALUE = 6.022 * 10^23
```

### Practical Example

```cobol
       WORKING-STORAGE SECTION.
       01  WS-SCI-INPUT     PIC X(30).
       01  WS-RESULT        PIC 9(15)V9(6).

       PROCEDURE DIVISION.
      *>   Parse scientific notation from an external source
           MOVE "2.998E+08" TO WS-SCI-INPUT
           COMPUTE WS-RESULT =
               FUNCTION NUMVAL-F(WS-SCI-INPUT)
           DISPLAY "Value: " WS-RESULT
           *> Value: 299800000.000000
           STOP RUN.
```

---

## Comparison of NUMVAL Functions

| Feature | NUMVAL | NUMVAL-C | NUMVAL-F |
|---------|--------|----------|----------|
| Simple numeric strings | Yes | Yes | No |
| Currency signs | No | Yes | No |
| Comma separators | No | Yes | No |
| Scientific notation | No | No | Yes |
| Leading/trailing sign | Yes | Yes | Yes (mantissa and exponent) |
| Leading/trailing spaces | Yes | Yes | Yes |
| Standard | COBOL-85 Amdt. 1 | COBOL-85 Amdt. 1 | COBOL 2002 |

---

## Validation Functions (TEST-NUMVAL)

COBOL 2014 introduced validation functions that test whether a string is valid input for the corresponding NUMVAL function, eliminating the need for manual inspection.

### TEST-NUMVAL

Tests whether a string is valid input for the NUMVAL function.

!!! note "COBOL 2014"
    The TEST-NUMVAL function was introduced in COBOL 2014.

```cobol
FUNCTION TEST-NUMVAL ( argument-1 )
```

- **argument-1** -- alphanumeric. The string to validate.
- **Return category** -- integer.
- **Behavior** -- returns 0 if argument-1 is a valid input for FUNCTION NUMVAL. If argument-1 is invalid, returns a nonzero value indicating the character position of the first error.

```cobol
IF FUNCTION TEST-NUMVAL(WS-INPUT) = 0
    COMPUTE WS-VALUE = FUNCTION NUMVAL(WS-INPUT)
ELSE
    DISPLAY "Invalid numeric string"
END-IF
```

### TEST-NUMVAL-C

Tests whether a string is valid input for the NUMVAL-C function.

!!! note "COBOL 2014"
    The TEST-NUMVAL-C function was introduced in COBOL 2014.

```cobol
FUNCTION TEST-NUMVAL-C ( argument-1  argument-2 )
```

- **argument-1** -- alphanumeric. The string to validate.
- **argument-2** -- alphanumeric. The currency sign string.
- **Return category** -- integer.
- **Behavior** -- returns 0 if argument-1 is a valid input for FUNCTION NUMVAL-C with the specified currency sign. Returns nonzero on failure.

```cobol
IF FUNCTION TEST-NUMVAL-C(WS-INPUT "$") = 0
    COMPUTE WS-VALUE = FUNCTION NUMVAL-C(WS-INPUT "$")
ELSE
    DISPLAY "Invalid currency string"
END-IF
```

### TEST-NUMVAL-F

Tests whether a string is valid input for the NUMVAL-F function.

!!! note "COBOL 2014"
    The TEST-NUMVAL-F function was introduced in COBOL 2014.

```cobol
FUNCTION TEST-NUMVAL-F ( argument-1 )
```

- **argument-1** -- alphanumeric. The string to validate.
- **Return category** -- integer.
- **Behavior** -- returns 0 if argument-1 is a valid floating-point string for FUNCTION NUMVAL-F. Returns nonzero on failure.

```cobol
IF FUNCTION TEST-NUMVAL-F(WS-INPUT) = 0
    COMPUTE WS-VALUE = FUNCTION NUMVAL-F(WS-INPUT)
ELSE
    DISPLAY "Invalid scientific notation"
END-IF
```

### Validation Pattern

The TEST-NUMVAL functions replace the error-prone manual validation that was previously required:

```cobol
*> COBOL 2014+ recommended approach: validate then convert
IF FUNCTION TEST-NUMVAL-C(WS-AMOUNT-STR "$") = 0
    COMPUTE WS-AMOUNT =
        FUNCTION NUMVAL-C(WS-AMOUNT-STR "$")
    PERFORM PROCESS-AMOUNT
ELSE
    DISPLAY "Invalid amount: " WS-AMOUNT-STR
    ADD 1 TO WS-ERROR-COUNT
END-IF
```

---

## Error Conditions

All NUMVAL functions have undefined behavior when argument-1 does not contain a valid numeric string for the expected format. Common causes of errors include:

- Empty or all-spaces string
- Non-numeric characters (other than allowed signs, decimal point, currency, commas)
- Multiple decimal points
- Multiple signs
- For NUMVAL-F: missing exponent indicator

!!! warning "No Runtime Validation"
    Prior to COBOL 2014, most compilers did not validate the input string at
    runtime. Passing an invalid string may produce incorrect results or cause
    an abend. Use the TEST-NUMVAL functions (COBOL 2014+) to validate input
    strings before passing them to NUMVAL functions.

```cobol
*> Pre-2014 approach: manual validation (error-prone)
INSPECT WS-INPUT TALLYING WS-NON-NUM
    FOR ALL "A" THRU "Z"
IF WS-NON-NUM > 0
    DISPLAY "Invalid numeric input"
ELSE
    COMPUTE WS-VALUE = FUNCTION NUMVAL(WS-INPUT)
END-IF
```

---

## See Also

- [Intrinsic Functions](index.md) -- overview of all intrinsic functions
- [Numeric Functions](numeric.md) -- mathematical intrinsic functions
- [String Functions](string.md) -- string manipulation intrinsic functions
- [PICTURE](../../data-division/picture.md) -- data item format specification
- [USAGE](../../data-division/usage.md) -- internal representation and storage format
