# Numeric Functions

Intrinsic functions that perform mathematical, statistical, and numeric operations on numeric arguments.

**Standard:** COBOL-85 Amendment 1 (1989), COBOL 2002, COBOL 2014, COBOL 2023

---

## General Syntax

```cobol
FUNCTION function-name ( argument-1 [ argument-2 ] ... )
```

All numeric intrinsic functions return a value of category **numeric** or **integer**. Numeric functions may appear anywhere an arithmetic expression is permitted, including in `COMPUTE`, `IF`, `EVALUATE`, and `DISPLAY` statements.

Arguments to numeric functions must be of category numeric unless otherwise specified. The result is a temporary numeric value with implementation-defined precision; it does not correspond to a declared [PICTURE](../../data-division/picture.md) or [USAGE](../../data-division/usage.md).

---

## Basic Numeric Functions

### ABS

Returns the absolute value of the argument.

```cobol
FUNCTION ABS ( argument-1 )
```

- **argument-1** -- numeric.
- **Return category** -- numeric. If argument-1 is an integer, the result is an integer.
- **Behavior** -- returns the absolute value of argument-1. If argument-1 is positive or zero, the result equals argument-1. If argument-1 is negative, the result equals the negation of argument-1.

```cobol
COMPUTE WS-RESULT = FUNCTION ABS(-42.5)
*> WS-RESULT = 42.5

COMPUTE WS-DIFF = FUNCTION ABS(WS-A - WS-B)
```

### SIGN

Returns an indication of the sign of the argument.

!!! note "COBOL 2023"
    The SIGN intrinsic function was introduced in COBOL 2023. It is distinct from the SIGN condition used in conditional expressions.

```cobol
FUNCTION SIGN ( argument-1 )
```

- **argument-1** -- numeric.
- **Return category** -- integer.
- **Behavior** -- returns +1 if argument-1 is positive, 0 if argument-1 is zero, and -1 if argument-1 is negative.

```cobol
COMPUTE WS-SIGN = FUNCTION SIGN(-100)
*> WS-SIGN = -1

COMPUTE WS-SIGN = FUNCTION SIGN(0)
*> WS-SIGN = 0

COMPUTE WS-SIGN = FUNCTION SIGN(55.3)
*> WS-SIGN = 1
```

### MOD

Returns the value of argument-1 modulo argument-2.

```cobol
FUNCTION MOD ( argument-1  argument-2 )
```

- **argument-1** -- integer.
- **argument-2** -- integer. Must not be zero.
- **Return category** -- integer.
- **Behavior** -- returns argument-1 modulo argument-2, defined as: `argument-1 - (argument-2 * FUNCTION INTEGER(argument-1 / argument-2))`. The result has the same sign as argument-2.

```cobol
COMPUTE WS-MOD = FUNCTION MOD(10 3)
*> WS-MOD = 1

COMPUTE WS-MOD = FUNCTION MOD(10 -3)
*> WS-MOD = -2

COMPUTE WS-MOD = FUNCTION MOD(-10 3)
*> WS-MOD = 2
```

### REM

Returns the remainder of the division of argument-1 by argument-2.

```cobol
FUNCTION REM ( argument-1  argument-2 )
```

- **argument-1** -- numeric.
- **argument-2** -- numeric. Must not be zero.
- **Return category** -- numeric.
- **Behavior** -- returns the remainder defined as: `argument-1 - (argument-2 * FUNCTION INTEGER-PART(argument-1 / argument-2))`. The result has the same sign as argument-1. REM differs from MOD in its treatment of negative arguments.

```cobol
COMPUTE WS-REM = FUNCTION REM(10 3)
*> WS-REM = 1

COMPUTE WS-REM = FUNCTION REM(-10 3)
*> WS-REM = -1

COMPUTE WS-REM = FUNCTION REM(10.5 3.2)
*> WS-REM = 0.9
```

### INTEGER

Returns the greatest integer value that is not greater than the argument.

```cobol
FUNCTION INTEGER ( argument-1 )
```

- **argument-1** -- numeric.
- **Return category** -- integer.
- **Behavior** -- returns the greatest integer not exceeding argument-1 (floor function). For positive values, this truncates toward zero. For negative non-integer values, this returns the next more-negative integer.

```cobol
COMPUTE WS-INT = FUNCTION INTEGER(3.7)
*> WS-INT = 3

COMPUTE WS-INT = FUNCTION INTEGER(-3.7)
*> WS-INT = -4

COMPUTE WS-INT = FUNCTION INTEGER(5.0)
*> WS-INT = 5
```

### INTEGER-PART

Returns the integer portion of the argument, truncating toward zero.

```cobol
FUNCTION INTEGER-PART ( argument-1 )
```

- **argument-1** -- numeric.
- **Return category** -- integer.
- **Behavior** -- returns the integer portion of argument-1 by truncation toward zero. For positive values, INTEGER-PART and INTEGER return the same result. For negative non-integer values, INTEGER-PART returns a value one greater than INTEGER.

```cobol
COMPUTE WS-INT = FUNCTION INTEGER-PART(3.7)
*> WS-INT = 3

COMPUTE WS-INT = FUNCTION INTEGER-PART(-3.7)
*> WS-INT = -3

COMPUTE WS-INT = FUNCTION INTEGER-PART(5.0)
*> WS-INT = 5
```

### FRACTION-PART

Returns the fractional portion of the argument.

!!! note "COBOL 2014"
    The FRACTION-PART function was introduced in COBOL 2014.

```cobol
FUNCTION FRACTION-PART ( argument-1 )
```

- **argument-1** -- numeric.
- **Return category** -- numeric.
- **Behavior** -- returns the fractional portion of argument-1, defined as: `argument-1 - FUNCTION INTEGER-PART(argument-1)`. For positive values, the result is non-negative. For negative values, the result is non-positive. FRACTION-PART is the complement of INTEGER-PART: `FUNCTION INTEGER-PART(x) + FUNCTION FRACTION-PART(x) = x` for all x.

```cobol
COMPUTE WS-FRAC = FUNCTION FRACTION-PART(3.75)
*> WS-FRAC = 0.75

COMPUTE WS-FRAC = FUNCTION FRACTION-PART(-3.75)
*> WS-FRAC = -0.75

COMPUTE WS-FRAC = FUNCTION FRACTION-PART(5.0)
*> WS-FRAC = 0
```

---

## Trigonometric Functions

All trigonometric functions operate in **radians**.

### SIN

Returns the sine of the argument.

```cobol
FUNCTION SIN ( argument-1 )
```

- **argument-1** -- numeric, representing an angle in radians.
- **Return category** -- numeric.
- **Behavior** -- returns the sine of argument-1. The result is in the range -1 through +1.

```cobol
COMPUTE WS-SINE = FUNCTION SIN(1.5708)
*> WS-SINE is approximately 1.0

COMPUTE WS-SINE = FUNCTION SIN(FUNCTION PI / 6)
*> WS-SINE is approximately 0.5
```

### COS

Returns the cosine of the argument.

```cobol
FUNCTION COS ( argument-1 )
```

- **argument-1** -- numeric, representing an angle in radians.
- **Return category** -- numeric.
- **Behavior** -- returns the cosine of argument-1. The result is in the range -1 through +1.

```cobol
COMPUTE WS-COS = FUNCTION COS(0)
*> WS-COS = 1.0

COMPUTE WS-COS = FUNCTION COS(FUNCTION PI)
*> WS-COS = -1.0
```

### TAN

Returns the tangent of the argument.

```cobol
FUNCTION TAN ( argument-1 )
```

- **argument-1** -- numeric, representing an angle in radians. Must not be an odd multiple of pi/2.
- **Return category** -- numeric.
- **Behavior** -- returns the tangent of argument-1.

```cobol
COMPUTE WS-TAN = FUNCTION TAN(0.7854)
*> WS-TAN is approximately 1.0

COMPUTE WS-TAN = FUNCTION TAN(FUNCTION PI / 4)
*> WS-TAN is approximately 1.0
```

### ACOS

Returns the arccosine of the argument.

```cobol
FUNCTION ACOS ( argument-1 )
```

- **argument-1** -- numeric, in the range -1 through +1.
- **Return category** -- numeric.
- **Behavior** -- returns the arccosine (inverse cosine) of argument-1 in radians. The result is in the range 0 through pi.

```cobol
COMPUTE WS-ANGLE = FUNCTION ACOS(0.5)
*> WS-ANGLE is approximately 1.0472 (pi/3)

COMPUTE WS-ANGLE = FUNCTION ACOS(0)
*> WS-ANGLE is approximately 1.5708 (pi/2)
```

### ASIN

Returns the arcsine of the argument.

```cobol
FUNCTION ASIN ( argument-1 )
```

- **argument-1** -- numeric, in the range -1 through +1.
- **Return category** -- numeric.
- **Behavior** -- returns the arcsine (inverse sine) of argument-1 in radians. The result is in the range -pi/2 through +pi/2.

```cobol
COMPUTE WS-ANGLE = FUNCTION ASIN(1.0)
*> WS-ANGLE is approximately 1.5708 (pi/2)

COMPUTE WS-ANGLE = FUNCTION ASIN(0.5)
*> WS-ANGLE is approximately 0.5236 (pi/6)
```

### ATAN

Returns the arctangent of the argument.

```cobol
FUNCTION ATAN ( argument-1 )
```

- **argument-1** -- numeric.
- **Return category** -- numeric.
- **Behavior** -- returns the arctangent (inverse tangent) of argument-1 in radians. The result is in the range -pi/2 through +pi/2 (exclusive).

```cobol
COMPUTE WS-ANGLE = FUNCTION ATAN(1.0)
*> WS-ANGLE is approximately 0.7854 (pi/4)

COMPUTE WS-ANGLE = FUNCTION ATAN(0)
*> WS-ANGLE = 0
```

---

## Exponential and Logarithmic Functions

### EXP

Returns *e* raised to the power of the argument.

```cobol
FUNCTION EXP ( argument-1 )
```

- **argument-1** -- numeric.
- **Return category** -- numeric.
- **Behavior** -- returns the value of the mathematical constant *e* (approximately 2.71828) raised to the power argument-1.

```cobol
COMPUTE WS-RESULT = FUNCTION EXP(1)
*> WS-RESULT is approximately 2.71828

COMPUTE WS-RESULT = FUNCTION EXP(0)
*> WS-RESULT = 1.0
```

### EXP10

Returns 10 raised to the power of the argument.

```cobol
FUNCTION EXP10 ( argument-1 )
```

- **argument-1** -- numeric.
- **Return category** -- numeric.
- **Behavior** -- returns 10 raised to the power argument-1.

```cobol
COMPUTE WS-RESULT = FUNCTION EXP10(3)
*> WS-RESULT = 1000.0

COMPUTE WS-RESULT = FUNCTION EXP10(0)
*> WS-RESULT = 1.0

COMPUTE WS-RESULT = FUNCTION EXP10(-2)
*> WS-RESULT = 0.01
```

### LOG

Returns the natural logarithm (base *e*) of the argument.

```cobol
FUNCTION LOG ( argument-1 )
```

- **argument-1** -- numeric, greater than zero.
- **Return category** -- numeric.
- **Behavior** -- returns the natural logarithm of argument-1. The result satisfies the identity `FUNCTION EXP(FUNCTION LOG(x)) = x` for positive x.

```cobol
COMPUTE WS-RESULT = FUNCTION LOG(FUNCTION E)
*> WS-RESULT is approximately 1.0

COMPUTE WS-RESULT = FUNCTION LOG(1)
*> WS-RESULT = 0
```

### LOG10

Returns the common (base-10) logarithm of the argument.

```cobol
FUNCTION LOG10 ( argument-1 )
```

- **argument-1** -- numeric, greater than zero.
- **Return category** -- numeric.
- **Behavior** -- returns the base-10 logarithm of argument-1. The result satisfies the identity `FUNCTION EXP10(FUNCTION LOG10(x)) = x` for positive x.

```cobol
COMPUTE WS-RESULT = FUNCTION LOG10(100)
*> WS-RESULT = 2.0

COMPUTE WS-RESULT = FUNCTION LOG10(1)
*> WS-RESULT = 0
```

---

## Statistical Functions

### MAX

Returns the maximum value among the arguments.

```cobol
FUNCTION MAX ( argument-1 ... )
```

- **argument-1 ...** -- one or more arguments. All must be of the same category (numeric, alphanumeric, or national).
- **Return category** -- same as the argument category. If all arguments are integer, the result is integer.
- **Behavior** -- returns the content of the argument that has the maximum value. For alphanumeric and national arguments, comparison uses the program collating sequence.

```cobol
COMPUTE WS-HIGH = FUNCTION MAX(10 25 7 42 18)
*> WS-HIGH = 42

COMPUTE WS-HIGH = FUNCTION MAX(SCORE(ALL))
*> Returns the maximum value from the SCORE table
```

### MIN

Returns the minimum value among the arguments.

```cobol
FUNCTION MIN ( argument-1 ... )
```

- **argument-1 ...** -- one or more arguments. All must be of the same category.
- **Return category** -- same as the argument category. If all arguments are integer, the result is integer.
- **Behavior** -- returns the content of the argument that has the minimum value.

```cobol
COMPUTE WS-LOW = FUNCTION MIN(10 25 7 42 18)
*> WS-LOW = 7

COMPUTE WS-LOW = FUNCTION MIN(SCORE(ALL))
*> Returns the minimum value from the SCORE table
```

### MEAN

Returns the arithmetic mean of the arguments.

```cobol
FUNCTION MEAN ( argument-1 ... )
```

- **argument-1 ...** -- one or more numeric arguments.
- **Return category** -- numeric.
- **Behavior** -- returns the arithmetic mean (average) of the arguments, calculated as the sum of all arguments divided by the number of arguments.

```cobol
COMPUTE WS-AVG = FUNCTION MEAN(10 20 30)
*> WS-AVG = 20.0

COMPUTE WS-AVG = FUNCTION MEAN(SCORE(ALL))
```

### MEDIAN

Returns the median value of the arguments.

```cobol
FUNCTION MEDIAN ( argument-1 ... )
```

- **argument-1 ...** -- one or more numeric arguments.
- **Return category** -- numeric.
- **Behavior** -- returns the median of the arguments. If the number of arguments is odd, the median is the middle value when the arguments are arranged in sorted order. If the number of arguments is even, the median is the arithmetic mean of the two middle values.

```cobol
COMPUTE WS-MED = FUNCTION MEDIAN(10 40 20 30 50)
*> WS-MED = 30

COMPUTE WS-MED = FUNCTION MEDIAN(10 20 30 40)
*> WS-MED = 25 (mean of 20 and 30)
```

### MIDRANGE

Returns the arithmetic mean of the minimum and maximum argument values.

```cobol
FUNCTION MIDRANGE ( argument-1 ... )
```

- **argument-1 ...** -- one or more numeric arguments.
- **Return category** -- numeric.
- **Behavior** -- returns the value `(FUNCTION MIN(arguments) + FUNCTION MAX(arguments)) / 2`.

```cobol
COMPUTE WS-MID = FUNCTION MIDRANGE(10 40 20 30 50)
*> WS-MID = 30 ( (10 + 50) / 2 )

COMPUTE WS-MID = FUNCTION MIDRANGE(3 7)
*> WS-MID = 5
```

### RANGE

Returns the difference between the maximum and minimum argument values.

```cobol
FUNCTION RANGE ( argument-1 ... )
```

- **argument-1 ...** -- one or more numeric arguments.
- **Return category** -- numeric. If all arguments are integer, the result is integer.
- **Behavior** -- returns `FUNCTION MAX(arguments) - FUNCTION MIN(arguments)`.

```cobol
COMPUTE WS-RANGE = FUNCTION RANGE(10 40 20 30 50)
*> WS-RANGE = 40 (50 - 10)
```

### SUM

Returns the sum of the arguments.

```cobol
FUNCTION SUM ( argument-1 ... )
```

- **argument-1 ...** -- one or more numeric arguments.
- **Return category** -- numeric. If all arguments are integer, the result is integer.
- **Behavior** -- returns the arithmetic sum of all arguments.

```cobol
COMPUTE WS-TOTAL = FUNCTION SUM(10 20 30)
*> WS-TOTAL = 60

COMPUTE WS-TOTAL = FUNCTION SUM(AMOUNT(ALL))
```

### STANDARD-DEVIATION

Returns the standard deviation of the arguments.

```cobol
FUNCTION STANDARD-DEVIATION ( argument-1 ... )
```

- **argument-1 ...** -- one or more numeric arguments.
- **Return category** -- numeric.
- **Behavior** -- returns the population standard deviation of the arguments, calculated as the square root of VARIANCE. The formula is: `SQRT( SUM((xi - mean)^2) / n )` where mean is the arithmetic mean and n is the number of arguments.

```cobol
COMPUTE WS-SD = FUNCTION STANDARD-DEVIATION(2 4 4 4 5 5 7 9)
*> WS-SD = 2.0
```

### VARIANCE

Returns the variance of the arguments.

```cobol
FUNCTION VARIANCE ( argument-1 ... )
```

- **argument-1 ...** -- one or more numeric arguments.
- **Return category** -- numeric.
- **Behavior** -- returns the population variance of the arguments, calculated as: `SUM((xi - mean)^2) / n` where mean is the arithmetic mean and n is the number of arguments.

```cobol
COMPUTE WS-VAR = FUNCTION VARIANCE(2 4 4 4 5 5 7 9)
*> WS-VAR = 4.0
```

---

## Mathematical Constants

### E

Returns the value of the mathematical constant *e*.

!!! note "COBOL 2023"
    The E function was introduced in COBOL 2023.

```cobol
FUNCTION E
```

- **Arguments** -- none.
- **Return category** -- numeric.
- **Behavior** -- returns the value of Euler's number *e* (approximately 2.718281828459045) to the maximum precision supported by the implementation.

```cobol
COMPUTE WS-E = FUNCTION E
*> WS-E is approximately 2.71828182845904

COMPUTE WS-RESULT = FUNCTION E ** 2
*> Equivalent to FUNCTION EXP(2)
```

### PI

Returns the value of the mathematical constant *pi*.

!!! note "COBOL 2023"
    The PI function was introduced in COBOL 2023.

```cobol
FUNCTION PI
```

- **Arguments** -- none.
- **Return category** -- numeric.
- **Behavior** -- returns the value of *pi* (approximately 3.141592653589793) to the maximum precision supported by the implementation.

```cobol
COMPUTE WS-PI = FUNCTION PI
*> WS-PI is approximately 3.14159265358979

COMPUTE WS-AREA = FUNCTION PI * WS-RADIUS ** 2

COMPUTE WS-DEGREES = WS-RADIANS * 180 / FUNCTION PI
```

---

## Other Functions

### RANDOM

Returns a pseudorandom number.

```cobol
FUNCTION RANDOM [ ( argument-1 ) ]
```

- **argument-1** -- optional, integer, non-negative. When specified, it serves as the seed value for the pseudorandom number sequence.
- **Return category** -- numeric.
- **Behavior** -- returns a numeric value greater than or equal to zero and less than one. When argument-1 is specified, it initializes the pseudorandom sequence with that seed, producing a repeatable sequence. When argument-1 is omitted, the function returns the next value in the current sequence. If no seed has been set, the initial seed is implementation-defined.

```cobol
COMPUTE WS-RAND = FUNCTION RANDOM(12345)
*> Initializes the sequence with seed 12345

COMPUTE WS-RAND = FUNCTION RANDOM
*> Returns the next pseudorandom number in the sequence

*> Generate a random integer between 1 and 100
COMPUTE WS-RAND-INT =
    FUNCTION INTEGER(FUNCTION RANDOM * 100) + 1
```

### FACTORIAL

Returns the factorial of the argument.

```cobol
FUNCTION FACTORIAL ( argument-1 )
```

- **argument-1** -- integer, non-negative.
- **Return category** -- integer.
- **Behavior** -- returns the factorial of argument-1 (argument-1!). FACTORIAL(0) returns 1. For large values of argument-1, the result may exceed the maximum integer supported by the implementation.

```cobol
COMPUTE WS-FACT = FUNCTION FACTORIAL(5)
*> WS-FACT = 120

COMPUTE WS-FACT = FUNCTION FACTORIAL(0)
*> WS-FACT = 1

COMPUTE WS-FACT = FUNCTION FACTORIAL(10)
*> WS-FACT = 3628800
```

### SQRT

Returns the square root of the argument.

```cobol
FUNCTION SQRT ( argument-1 )
```

- **argument-1** -- numeric, non-negative.
- **Return category** -- numeric.
- **Behavior** -- returns the non-negative square root of argument-1.

```cobol
COMPUTE WS-ROOT = FUNCTION SQRT(144)
*> WS-ROOT = 12.0

COMPUTE WS-HYPOTENUSE =
    FUNCTION SQRT(WS-SIDE-A ** 2 + WS-SIDE-B ** 2)
```

---

## Data Item Bounds

### HIGHEST-ALGEBRAIC

Returns the highest value that can be stored in a specified data item.

!!! note "COBOL 2014"
    The HIGHEST-ALGEBRAIC function was introduced in COBOL 2014.

```cobol
FUNCTION HIGHEST-ALGEBRAIC ( argument-1 )
```

- **argument-1** -- a numeric data-name. Must not be a literal.
- **Return category** -- numeric.
- **Behavior** -- returns the highest value that can be stored in argument-1, based on its PICTURE and USAGE. Useful for bounds checking and initialization.

```cobol
01  WS-UNSIGNED   PIC 9(3).
01  WS-SIGNED     PIC S9(3).
01  WS-DECIMAL    PIC S9(3)V99.

COMPUTE WS-RESULT = FUNCTION HIGHEST-ALGEBRAIC(WS-UNSIGNED)
*> WS-RESULT = 999

COMPUTE WS-RESULT = FUNCTION HIGHEST-ALGEBRAIC(WS-SIGNED)
*> WS-RESULT = 999

COMPUTE WS-RESULT = FUNCTION HIGHEST-ALGEBRAIC(WS-DECIMAL)
*> WS-RESULT = 999.99
```

### LOWEST-ALGEBRAIC

Returns the lowest (most negative) value that can be stored in a specified data item.

!!! note "COBOL 2014"
    The LOWEST-ALGEBRAIC function was introduced in COBOL 2014.

```cobol
FUNCTION LOWEST-ALGEBRAIC ( argument-1 )
```

- **argument-1** -- a numeric data-name. Must not be a literal.
- **Return category** -- numeric.
- **Behavior** -- returns the lowest value that can be stored in argument-1, based on its PICTURE and USAGE. For unsigned data items, the result is zero. For signed data items, the result is the most negative value representable.

```cobol
01  WS-UNSIGNED   PIC 9(3).
01  WS-SIGNED     PIC S9(3).
01  WS-DECIMAL    PIC S9(3)V99.

COMPUTE WS-RESULT = FUNCTION LOWEST-ALGEBRAIC(WS-UNSIGNED)
*> WS-RESULT = 0

COMPUTE WS-RESULT = FUNCTION LOWEST-ALGEBRAIC(WS-SIGNED)
*> WS-RESULT = -999

COMPUTE WS-RESULT = FUNCTION LOWEST-ALGEBRAIC(WS-DECIMAL)
*> WS-RESULT = -999.99
```

---

## Boolean Conversion Functions

### BOOLEAN-OF-INTEGER

Converts an integer to a boolean value.

!!! note "COBOL 2002"
    The BOOLEAN-OF-INTEGER function was introduced in COBOL 2002.

```cobol
FUNCTION BOOLEAN-OF-INTEGER ( argument-1  argument-2 )
```

- **argument-1** -- integer. The value to convert to boolean.
- **argument-2** -- integer. The length of the result in boolean positions.
- **Return category** -- boolean.
- **Behavior** -- converts argument-1 to its boolean (bit-string) representation with a length of argument-2 boolean positions. The result is the binary representation of argument-1, padded on the left with zero bits if necessary to fill argument-2 positions.

```cobol
MOVE FUNCTION BOOLEAN-OF-INTEGER(5 8) TO WS-BOOL
*> WS-BOOL = "00000101"

MOVE FUNCTION BOOLEAN-OF-INTEGER(255 8) TO WS-BOOL
*> WS-BOOL = "11111111"
```

### INTEGER-OF-BOOLEAN

Converts a boolean value to its integer representation.

!!! note "COBOL 2002"
    The INTEGER-OF-BOOLEAN function was introduced in COBOL 2002.

```cobol
FUNCTION INTEGER-OF-BOOLEAN ( argument-1 )
```

- **argument-1** -- boolean.
- **Return category** -- integer.
- **Behavior** -- converts argument-1 from its boolean (bit-string) representation to the corresponding integer value. The result is the integer equivalent of the binary value represented by argument-1.

```cobol
COMPUTE WS-INT = FUNCTION INTEGER-OF-BOOLEAN(WS-BOOL)
*> If WS-BOOL = "00000101", WS-INT = 5

COMPUTE WS-INT = FUNCTION INTEGER-OF-BOOLEAN(WS-FLAGS)
*> Converts boolean flags to their integer equivalent
```

---

## Examples

### Calculating Distance Between Two Points

```cobol
       WORKING-STORAGE SECTION.
       01  WS-X1          PIC S9(5)V99 VALUE 3.00.
       01  WS-Y1          PIC S9(5)V99 VALUE 4.00.
       01  WS-X2          PIC S9(5)V99 VALUE 7.00.
       01  WS-Y2          PIC S9(5)V99 VALUE 1.00.
       01  WS-DISTANCE    PIC 9(5)V9(4).

       PROCEDURE DIVISION.
           COMPUTE WS-DISTANCE =
               FUNCTION SQRT(
                   (WS-X2 - WS-X1) ** 2 +
                   (WS-Y2 - WS-Y1) ** 2)
           DISPLAY "Distance: " WS-DISTANCE
           *> Distance: 00005.0000
           STOP RUN.
```

### Statistical Analysis of a Table

```cobol
       WORKING-STORAGE SECTION.
       01  WS-SCORES.
           05  WS-SCORE    PIC 999 OCCURS 5 TIMES.
       01  WS-STATS.
           05  WS-AVG     PIC ZZ9.99.
           05  WS-HIGH    PIC ZZ9.
           05  WS-LOW     PIC ZZ9.
           05  WS-SD      PIC ZZ9.99.

       PROCEDURE DIVISION.
           MOVE 85  TO WS-SCORE(1)
           MOVE 92  TO WS-SCORE(2)
           MOVE 78  TO WS-SCORE(3)
           MOVE 95  TO WS-SCORE(4)
           MOVE 88  TO WS-SCORE(5)

           COMPUTE WS-AVG  = FUNCTION MEAN(WS-SCORE(ALL))
           COMPUTE WS-HIGH = FUNCTION MAX(WS-SCORE(ALL))
           COMPUTE WS-LOW  = FUNCTION MIN(WS-SCORE(ALL))
           COMPUTE WS-SD   =
               FUNCTION STANDARD-DEVIATION(WS-SCORE(ALL))

           DISPLAY "Average: " WS-AVG
           DISPLAY "Highest: " WS-HIGH
           DISPLAY "Lowest:  " WS-LOW
           DISPLAY "Std Dev: " WS-SD
           STOP RUN.
```

### Converting Degrees to Radians

```cobol
       WORKING-STORAGE SECTION.
       01  WS-DEGREES     PIC S9(3)V99.
       01  WS-RADIANS     PIC S9V9(8).
       01  WS-RESULT      PIC S9V9(8).

       PROCEDURE DIVISION.
           MOVE 45 TO WS-DEGREES
           COMPUTE WS-RADIANS =
               WS-DEGREES * FUNCTION PI / 180
           COMPUTE WS-RESULT = FUNCTION SIN(WS-RADIANS)
           DISPLAY "sin(45) = " WS-RESULT
           *> sin(45) = approximately 0.70710678
           STOP RUN.
```

---

## See Also

- [Intrinsic Functions](index.md) -- overview of all intrinsic functions
- [String Functions](string.md) -- string manipulation intrinsic functions
- [Date and Time Functions](date-time.md) -- date and time intrinsic functions
- [Financial Functions](financial.md) -- financial calculation intrinsic functions
- [PICTURE](../../data-division/picture.md) -- data item format specification
- [USAGE](../../data-division/usage.md) -- internal representation and storage format
