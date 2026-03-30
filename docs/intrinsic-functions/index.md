# Intrinsic Functions

Intrinsic functions return a value derived from zero or more arguments at the point of reference. They are invoked inline within expressions and do not require explicit declaration.

- **Standard:** COBOL-85 Amendment 1 (1989), COBOL 2002, COBOL 2014, COBOL 2023

## Syntax

```cobol
FUNCTION function-name [ ( { argument-1 } ... ) ]
```

## Rules

### General Invocation Rules

An intrinsic function is referenced by specifying the keyword `FUNCTION` followed by the function name and, where required, a parenthesized list of arguments. The function reference returns a single value and may appear anywhere a literal or identifier of the corresponding category is permitted.

The word `FUNCTION` is mandatory. Without it, the compiler interprets the function name as a data-name reference.

!!! note "COBOL 2014"
    COBOL 2014 introduced the `FUNCTION ALL INTRINSIC` directive in the `REPOSITORY` paragraph, which allows intrinsic functions to be referenced without the `FUNCTION` keyword.

### Argument Rules

Arguments are specified as a parenthesized, space-separated list. The number of arguments, their categories, and their permitted ranges are defined individually for each function. Some functions accept a variable number of arguments.

An argument may be a literal, a data-name reference, an arithmetic expression, or another function reference. When a function accepts a table argument, the `ALL` subscript may be used to pass all occurrences of a table element.

```cobol
COMPUTE WS-MEAN = FUNCTION MEAN(SCORE(ALL))
```

### Function Value Categories

Each intrinsic function returns a value of one of the following categories:

- **Alphanumeric** -- the function returns a character string value.
- **National** -- the function returns a national character string value.
- **Numeric** -- the function returns a numeric value. Numeric functions may appear in arithmetic expressions.
- **Integer** -- the function returns an integer numeric value. Integer functions are a subset of numeric functions.
- **Boolean** -- the function returns a boolean value.

The category of a function determines where it may be referenced.

### Inline Usage

Because intrinsic functions are evaluated inline, they may be used directly in `MOVE`, `COMPUTE`, `IF`, `EVALUATE`, `STRING`, `DISPLAY`, and other statements wherever an appropriately categorized operand is accepted.

```cobol
MOVE FUNCTION UPPER-CASE(WS-INPUT) TO WS-OUTPUT
IF FUNCTION LENGTH(WS-FIELD) > 10
    DISPLAY "Field exceeds maximum length"
END-IF
COMPUTE WS-ROOT = FUNCTION SQRT(WS-VALUE)
```

## Function Categories

The following tables list all standard intrinsic functions grouped by category. Functions may appear in more than one category where applicable.

### Numeric and Mathematical Functions

| Function | Description | Since |
|---|---|---|
| ABS | Absolute value of an argument | COBOL-85 Amdt. 1 |
| ACOS | Arccosine of an argument | COBOL-85 Amdt. 1 |
| ANNUITY | Ratio of annuity paid for a given period and interest rate | COBOL-85 Amdt. 1 |
| ASIN | Arcsine of an argument | COBOL-85 Amdt. 1 |
| ATAN | Arctangent of an argument | COBOL-85 Amdt. 1 |
| CONVERT | Base conversion of a numeric string | COBOL 2023 |
| COS | Cosine of an argument | COBOL-85 Amdt. 1 |
| E | Value of the mathematical constant *e* | COBOL 2023 |
| EXP | *e* raised to the power of the argument | COBOL-85 Amdt. 1 |
| EXP10 | 10 raised to the power of the argument | COBOL-85 Amdt. 1 |
| FACTORIAL | Factorial of an argument | COBOL-85 Amdt. 1 |
| FRACTION-PART | Fractional portion of the argument | COBOL 2014 |
| HIGHEST-ALGEBRAIC | Highest value storable in a data item | COBOL 2014 |
| INTEGER | Greatest integer not exceeding the argument value | COBOL-85 Amdt. 1 |
| INTEGER-PART | Integer portion of the argument | COBOL-85 Amdt. 1 |
| LOG | Natural logarithm of an argument | COBOL-85 Amdt. 1 |
| LOG10 | Common (base-10) logarithm of an argument | COBOL-85 Amdt. 1 |
| LOWEST-ALGEBRAIC | Lowest value storable in a data item | COBOL 2014 |
| MAX | Maximum value among arguments | COBOL-85 Amdt. 1 |
| MEAN | Arithmetic mean of the arguments | COBOL-85 Amdt. 1 |
| MEDIAN | Median value of the arguments | COBOL-85 Amdt. 1 |
| MIDRANGE | Mean of the minimum and maximum arguments | COBOL-85 Amdt. 1 |
| MIN | Minimum value among arguments | COBOL-85 Amdt. 1 |
| MOD | Modulus of the first argument with respect to the second | COBOL-85 Amdt. 1 |
| PI | Value of the mathematical constant *pi* | COBOL 2023 |
| PRESENT-VALUE | Present value of a series of future amounts at a given rate | COBOL-85 Amdt. 1 |
| RANDOM | Pseudorandom number in the range 0 through 1 | COBOL-85 Amdt. 1 |
| RANGE | Difference between the maximum and minimum arguments | COBOL-85 Amdt. 1 |
| REM | Remainder of the division of the first argument by the second | COBOL-85 Amdt. 1 |
| SIGN | Indication of the sign of the argument (-1, 0, or 1) | COBOL 2023 |
| SIN | Sine of an argument | COBOL-85 Amdt. 1 |
| SQRT | Square root of an argument | COBOL-85 Amdt. 1 |
| STANDARD-DEVIATION | Standard deviation of the arguments | COBOL-85 Amdt. 1 |
| SUM | Sum of the arguments | COBOL-85 Amdt. 1 |
| TAN | Tangent of an argument | COBOL-85 Amdt. 1 |
| VARIANCE | Variance of the arguments | COBOL-85 Amdt. 1 |

!!! note "COBOL 2014"
    The functions FRACTION-PART, HIGHEST-ALGEBRAIC, and LOWEST-ALGEBRAIC were introduced in COBOL 2014.

!!! note "COBOL 2023"
    The functions CONVERT, E, PI, SIGN, and SUBSTITUTE were introduced in COBOL 2023.

### String Functions

| Function | Description | Since |
|---|---|---|
| BYTE-LENGTH | Length of an argument in bytes | COBOL 2002 |
| CHAR | Character at a specified position in the program collating sequence | COBOL-85 Amdt. 1 |
| CHAR-NATIONAL | National character at a specified ordinal position | COBOL 2002 |
| CONCAT | Concatenation of the argument strings (revised name) | COBOL 2014 |
| CONCATENATE | Concatenation of the argument strings | COBOL 2002 |
| DISPLAY-OF | National string converted to alphanumeric | COBOL 2002 |
| LENGTH | Length of an argument in character positions | COBOL-85 Amdt. 1 |
| LENGTH-AN | Length in alphanumeric character positions | COBOL 2002 |
| LOCALE-COMPARE | Locale-aware comparison of two strings | COBOL 2002 |
| LOWER-CASE | Argument converted to lowercase | COBOL-85 Amdt. 1 |
| NATIONAL-OF | Alphanumeric string converted to national | COBOL 2002 |
| ORD | Ordinal position of a character in the program collating sequence | COBOL-85 Amdt. 1 |
| ORD-MAX | Ordinal position of the argument with the maximum value | COBOL-85 Amdt. 1 |
| ORD-MIN | Ordinal position of the argument with the minimum value | COBOL-85 Amdt. 1 |
| REVERSE | Argument with characters in reversed order | COBOL-85 Amdt. 1 |
| STANDARD-COMPARE | Comparison using standardized rules | COBOL 2002 |
| SUBSTITUTE | Argument with specified substitutions applied | COBOL 2023 |
| TRIM | Argument with leading or trailing spaces removed | COBOL 2002 |
| UPPER-CASE | Argument converted to uppercase | COBOL-85 Amdt. 1 |

!!! note "COBOL 2002"
    The functions BYTE-LENGTH, CHAR-NATIONAL, CONCATENATE, DISPLAY-OF, LENGTH-AN, LOCALE-COMPARE, NATIONAL-OF, STANDARD-COMPARE, and TRIM were introduced in COBOL 2002.

!!! note "COBOL 2014"
    CONCAT was introduced in COBOL 2014 as the preferred shorter name for CONCATENATE.

!!! note "COBOL 2023"
    The function SUBSTITUTE was introduced in COBOL 2023.

### Date and Time Functions

| Function | Description | Since |
|---|---|---|
| COMBINED-DATETIME | Combined date and time as a single numeric value | COBOL 2002 |
| CURRENT-DATE | Current date and time as a 21-character alphanumeric value | COBOL-85 Amdt. 1 |
| DATE-OF-INTEGER | Standard date (YYYYMMDD) corresponding to an integer date | COBOL-85 Amdt. 1 |
| DATE-TO-YYYYMMDD | Conversion of a date from YYMMDD to YYYYMMDD using a windowing rule | COBOL 2002 |
| DAY-OF-INTEGER | Julian date (YYYYDDD) corresponding to an integer date | COBOL-85 Amdt. 1 |
| DAY-TO-YYYYDDD | Conversion of a date from YYDDD to YYYYDDD using a windowing rule | COBOL 2002 |
| FORMATTED-CURRENT-DATE | Current date/time formatted per a format string | COBOL 2014 |
| FORMATTED-DATE | Integer date formatted per a format string | COBOL 2014 |
| FORMATTED-DATETIME | Combined date-time formatted per a format string | COBOL 2014 |
| FORMATTED-TIME | Time formatted per a format string | COBOL 2014 |
| INTEGER-OF-DATE | Integer date corresponding to a standard date (YYYYMMDD) | COBOL-85 Amdt. 1 |
| INTEGER-OF-DAY | Integer date corresponding to a Julian date (YYYYDDD) | COBOL-85 Amdt. 1 |
| INTEGER-OF-FORMATTED-DATE | Integer date from a formatted date string | COBOL 2014 |
| LOCALE-DATE | Date formatted according to locale conventions | COBOL 2002 |
| LOCALE-TIME | Time formatted according to locale conventions | COBOL 2002 |
| LOCALE-TIME-FROM-SECONDS | Locale-formatted time from seconds past midnight | COBOL 2002 |
| SECONDS-FROM-FORMATTED-TIME | Number of seconds from a formatted time string | COBOL 2002 |
| SECONDS-PAST-MIDNIGHT | Number of seconds since midnight | COBOL-85 Amdt. 1 |
| TEST-DATE-YYYYMMDD | Tests validity of a YYYYMMDD date | COBOL 2002 |
| TEST-DAY-YYYYDDD | Tests validity of a YYYYDDD Julian date | COBOL 2002 |
| TEST-FORMATTED-DATETIME | Tests validity of a formatted date/time string | COBOL 2014 |
| WHEN-COMPILED | Date and time the program was compiled | COBOL-85 Amdt. 1 |
| YEAR-TO-YYYY | Conversion of a two-digit year to a four-digit year using a windowing rule | COBOL 2002 |

!!! note "COBOL 2002"
    The functions COMBINED-DATETIME, DATE-TO-YYYYMMDD, DAY-TO-YYYYDDD, LOCALE-DATE, LOCALE-TIME, LOCALE-TIME-FROM-SECONDS, SECONDS-FROM-FORMATTED-TIME, TEST-DATE-YYYYMMDD, TEST-DAY-YYYYDDD, and YEAR-TO-YYYY were introduced in COBOL 2002.

!!! note "COBOL 2014"
    The formatted date/time family (FORMATTED-CURRENT-DATE, FORMATTED-DATE, FORMATTED-DATETIME, FORMATTED-TIME, INTEGER-OF-FORMATTED-DATE, TEST-FORMATTED-DATETIME) was introduced in COBOL 2014.

### Financial Functions

| Function | Description | Since |
|---|---|---|
| ANNUITY | Ratio of annuity paid for a given period and interest rate | COBOL-85 Amdt. 1 |
| PRESENT-VALUE | Present value of a series of future amounts at a given rate | COBOL-85 Amdt. 1 |

### Numeric Conversion Functions

| Function | Description | Since |
|---|---|---|
| NUMVAL | Numeric value of an alphanumeric string in simple format | COBOL-85 Amdt. 1 |
| NUMVAL-C | Numeric value of an alphanumeric string with optional currency sign | COBOL-85 Amdt. 1 |
| NUMVAL-F | Numeric value of an alphanumeric string in floating-point format | COBOL 2002 |
| TEST-NUMVAL | Tests validity of a string for NUMVAL | COBOL 2014 |
| TEST-NUMVAL-C | Tests validity of a string for NUMVAL-C | COBOL 2014 |
| TEST-NUMVAL-F | Tests validity of a string for NUMVAL-F | COBOL 2014 |

!!! note "COBOL 2002"
    The function NUMVAL-F was introduced in COBOL 2002.

!!! note "COBOL 2014"
    The validation functions TEST-NUMVAL, TEST-NUMVAL-C, and TEST-NUMVAL-F were introduced in COBOL 2014.

### Boolean and Ordinal Functions

| Function | Description | Since |
|---|---|---|
| BOOLEAN-OF-INTEGER | Integer converted to boolean representation | COBOL 2002 |
| INTEGER-OF-BOOLEAN | Boolean value converted to integer | COBOL 2002 |
| ORD-MAX | Ordinal position of the argument with the maximum value | COBOL-85 Amdt. 1 |
| ORD-MIN | Ordinal position of the argument with the minimum value | COBOL-85 Amdt. 1 |

### Exception Handling Functions

| Function | Description | Since |
|---|---|---|
| EXCEPTION-FILE | File name associated with the last I/O exception | COBOL 2002 |
| EXCEPTION-LOCATION | Source location of the last exception | COBOL 2002 |
| EXCEPTION-STATEMENT | Statement that caused the last exception | COBOL 2002 |
| EXCEPTION-STATUS | Exception condition name for the last exception | COBOL 2002 |

## Examples

### Numeric Functions

```cobol
COMPUTE WS-HYPOTENUSE =
    FUNCTION SQRT(WS-SIDE-A ** 2 + WS-SIDE-B ** 2)

COMPUTE WS-HIGHEST = FUNCTION MAX(SCORE(ALL))
COMPUTE WS-LOWEST  = FUNCTION MIN(SCORE(ALL))
COMPUTE WS-AVERAGE = FUNCTION MEAN(SCORE(ALL))
```

### String Functions

```cobol
MOVE FUNCTION UPPER-CASE(WS-NAME) TO WS-NAME-UPPER
MOVE FUNCTION TRIM(WS-INPUT) TO WS-TRIMMED
MOVE FUNCTION CONCATENATE(WS-FIRST " " WS-LAST) TO WS-FULL

IF FUNCTION LENGTH(WS-DATA) > 0
    PERFORM PROCESS-DATA
END-IF
```

### Date and Time Functions

```cobol
MOVE FUNCTION CURRENT-DATE TO WS-DATETIME
*> WS-DATETIME contains YYYYMMDDHHMMSSssGHHMM

COMPUTE WS-INTEGER-DATE =
    FUNCTION INTEGER-OF-DATE(WS-YYYYMMDD)
COMPUTE WS-NEXT-DATE =
    FUNCTION DATE-OF-INTEGER(WS-INTEGER-DATE + 30)
```

### Numeric Conversion

```cobol
COMPUTE WS-VALUE = FUNCTION NUMVAL(WS-AMOUNT-STRING)
COMPUTE WS-CURRENCY = FUNCTION NUMVAL-C(WS-DOLLAR-STRING "$")
```

## See Also

- [PICTURE](../data-division/picture.md) -- data item format specification
- [OCCURS](../data-division/occurs.md) -- table definitions for use with ALL subscript
