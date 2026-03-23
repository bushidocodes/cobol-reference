# String Functions

Intrinsic functions that perform operations on alphanumeric and national character strings.

**Standard:** COBOL-85 Amendment 1 (1989), COBOL 2002, COBOL 2014, COBOL 2023

---

## General Syntax

```cobol
FUNCTION function-name ( argument-1 [ argument-2 ] ... )
```

String intrinsic functions return a value of category **alphanumeric**, **national**, or **integer** depending on the function. Alphanumeric and national results may be used wherever a data item of the corresponding category is permitted, including in `MOVE`, `STRING`, `DISPLAY`, and `IF` statements.

---

## CHAR

Returns the character at a specified position in the program collating sequence.

```cobol
FUNCTION CHAR ( argument-1 )
```

- **argument-1** -- integer, in the range 1 through the number of positions in the program collating sequence.
- **Return category** -- alphanumeric, length 1.
- **Behavior** -- returns the character that occupies ordinal position argument-1 in the program collating sequence. The program collating sequence is determined by the ALPHABET clause in the SPECIAL-NAMES paragraph, or the native collating sequence if no ALPHABET clause is specified.

```cobol
MOVE FUNCTION CHAR(66) TO WS-CH
*> In ASCII, ordinal 66 corresponds to 'A' (code point 65)
*> CHAR uses ordinal (1-based), not code point (0-based)

MOVE FUNCTION CHAR(1) TO WS-FIRST
*> Returns the first character in the collating sequence
```

---

## CONCATENATE

Returns the concatenation of the argument strings.

!!! note "COBOL 2002"
    The CONCATENATE function was introduced in COBOL 2002.

```cobol
FUNCTION CONCATENATE ( argument-1 ... )
```

- **argument-1 ...** -- one or more arguments of category alphanumeric or national. All arguments must be of the same category.
- **Return category** -- same as the argument category. The length of the result is the sum of the lengths of all arguments.
- **Behavior** -- returns a character string that is the concatenation of the arguments in the order specified. No padding or trimming is performed; trailing spaces in each argument are preserved in the result.

```cobol
MOVE FUNCTION CONCATENATE(WS-FIRST " " WS-LAST)
    TO WS-FULL-NAME

MOVE FUNCTION CONCATENATE("Hello" ", " "World" "!")
    TO WS-GREETING
*> WS-GREETING = "Hello, World!"

*> Note: trailing spaces in arguments are included
MOVE "John      " TO WS-FIRST
MOVE FUNCTION CONCATENATE(WS-FIRST WS-LAST)
    TO WS-FULL
*> WS-FULL includes trailing spaces from WS-FIRST
*> Use TRIM to remove them:
MOVE FUNCTION CONCATENATE(
    FUNCTION TRIM(WS-FIRST TRAILING)
    " "
    WS-LAST)
    TO WS-FULL
```

---

## LENGTH

Returns the length of the argument in character positions.

```cobol
FUNCTION LENGTH ( argument-1 )
```

- **argument-1** -- any category. May be a literal, data-name, or another function reference.
- **Return category** -- integer.
- **Behavior** -- returns the number of character positions in argument-1. For a data item, the length is the defined length (the number of character positions in its [PICTURE](../../data-division/picture.md)), not the length of the data content. For a national item, it returns the number of national character positions.

```cobol
01  WS-NAME  PIC X(20) VALUE "SMITH".

COMPUTE WS-LEN = FUNCTION LENGTH(WS-NAME)
*> WS-LEN = 20 (defined length, not content length)

COMPUTE WS-LEN = FUNCTION LENGTH("HELLO")
*> WS-LEN = 5

IF FUNCTION LENGTH(WS-FIELD) > 10
    DISPLAY "Field exceeds 10 characters"
END-IF
```

---

## LOWER-CASE

Returns the argument with all uppercase letters converted to lowercase.

```cobol
FUNCTION LOWER-CASE ( argument-1 )
```

- **argument-1** -- alphanumeric or national.
- **Return category** -- same as argument-1 category. Length equals the length of argument-1.
- **Behavior** -- returns a character string with the same length as argument-1, in which each uppercase letter is replaced by its lowercase equivalent. Characters that are not uppercase letters are returned unchanged.

```cobol
MOVE FUNCTION LOWER-CASE("HELLO WORLD")
    TO WS-RESULT
*> WS-RESULT = "hello world"

MOVE FUNCTION LOWER-CASE(WS-INPUT) TO WS-OUTPUT
```

---

## ORD

Returns the ordinal position of a character in the program collating sequence.

```cobol
FUNCTION ORD ( argument-1 )
```

- **argument-1** -- alphanumeric, length 1.
- **Return category** -- integer.
- **Behavior** -- returns the ordinal position of argument-1 in the program collating sequence. ORD and CHAR are inverse functions: `FUNCTION CHAR(FUNCTION ORD(x)) = x` for any single character x.

```cobol
COMPUTE WS-POS = FUNCTION ORD("A")
*> In ASCII native collating sequence, WS-POS = 66
*> (ordinal = code point + 1)

COMPUTE WS-POS = FUNCTION ORD(WS-CHAR)
```

---

## ORD-MAX

Returns the ordinal position of the argument that contains the maximum value.

```cobol
FUNCTION ORD-MAX ( argument-1 ... )
```

- **argument-1 ...** -- one or more arguments of the same category (alphanumeric, national, or numeric).
- **Return category** -- integer.
- **Behavior** -- returns the ordinal number (1-based position in the argument list) of the argument that has the maximum value. If two or more arguments are equal and are the maximum, the ordinal of the leftmost such argument is returned. Comparison uses the program collating sequence for alphanumeric and national arguments.

```cobol
COMPUTE WS-POS = FUNCTION ORD-MAX("C" "A" "B")
*> WS-POS = 1 (first argument "C" is the maximum)

COMPUTE WS-POS = FUNCTION ORD-MAX(10 30 20)
*> WS-POS = 2 (second argument 30 is the maximum)
```

---

## ORD-MIN

Returns the ordinal position of the argument that contains the minimum value.

```cobol
FUNCTION ORD-MIN ( argument-1 ... )
```

- **argument-1 ...** -- one or more arguments of the same category.
- **Return category** -- integer.
- **Behavior** -- returns the ordinal number (1-based position in the argument list) of the argument that has the minimum value. If two or more arguments are equal and are the minimum, the ordinal of the leftmost such argument is returned.

```cobol
COMPUTE WS-POS = FUNCTION ORD-MIN("C" "A" "B")
*> WS-POS = 2 (second argument "A" is the minimum)

COMPUTE WS-POS = FUNCTION ORD-MIN(10 30 20)
*> WS-POS = 1 (first argument 10 is the minimum)
```

---

## REVERSE

Returns the argument with characters in reversed order.

```cobol
FUNCTION REVERSE ( argument-1 )
```

- **argument-1** -- alphanumeric or national.
- **Return category** -- same as argument-1 category. Length equals the length of argument-1.
- **Behavior** -- returns a character string of the same length as argument-1 with the characters in reverse order. The last character of argument-1 becomes the first character of the result, and so on.

```cobol
MOVE FUNCTION REVERSE("ABCDE") TO WS-RESULT
*> WS-RESULT = "EDCBA"

MOVE FUNCTION REVERSE(WS-INPUT) TO WS-OUTPUT
*> Note: trailing spaces in WS-INPUT become leading spaces
```

---

## SUBSTITUTE

Returns the argument with specified substitutions applied.

!!! note "COBOL 2023"
    The SUBSTITUTE function was introduced in COBOL 2023.

```cobol
FUNCTION SUBSTITUTE ( argument-1
    argument-2  argument-3
  [ argument-4  argument-5 ] ... )
```

- **argument-1** -- alphanumeric or national. The source string.
- **argument-2, argument-4, ...** -- same category as argument-1. The search strings.
- **argument-3, argument-5, ...** -- same category as argument-1. The replacement strings.
- **Return category** -- same as argument-1 category.
- **Behavior** -- returns a character string derived from argument-1 by replacing each non-overlapping occurrence of each search string with the corresponding replacement string. Substitution pairs are processed from left to right within argument-1. The replacement strings need not be the same length as the search strings, so the result length may differ from the length of argument-1.

```cobol
MOVE FUNCTION SUBSTITUTE(
    "Hello World" "World" "COBOL")
    TO WS-RESULT
*> WS-RESULT = "Hello COBOL"

MOVE FUNCTION SUBSTITUTE(
    WS-TEXT
    "FOO" "BAR"
    "BAZ" "QUX")
    TO WS-OUTPUT
*> Replaces all "FOO" with "BAR" and all "BAZ" with "QUX"
```

---

## TRIM

Returns the argument with leading or trailing spaces removed.

!!! note "COBOL 2002"
    The TRIM function was introduced in COBOL 2002.

```cobol
FUNCTION TRIM ( argument-1 [ LEADING | TRAILING ] )
```

- **argument-1** -- alphanumeric or national.
- **LEADING** -- if specified, only leading spaces are removed.
- **TRAILING** -- if specified, only trailing spaces are removed.
- If neither LEADING nor TRAILING is specified, both leading and trailing spaces are removed.
- **Return category** -- same as argument-1 category. The length of the result depends on the number of spaces removed. If argument-1 contains only spaces, the result is a single space.

```cobol
MOVE FUNCTION TRIM("   Hello   ") TO WS-RESULT
*> WS-RESULT = "Hello"

MOVE FUNCTION TRIM("   Hello   " LEADING) TO WS-RESULT
*> WS-RESULT = "Hello   "

MOVE FUNCTION TRIM("   Hello   " TRAILING) TO WS-RESULT
*> WS-RESULT = "   Hello"

MOVE FUNCTION TRIM(WS-INPUT) TO WS-CLEAN
```

---

## UPPER-CASE

Returns the argument with all lowercase letters converted to uppercase.

```cobol
FUNCTION UPPER-CASE ( argument-1 )
```

- **argument-1** -- alphanumeric or national.
- **Return category** -- same as argument-1 category. Length equals the length of argument-1.
- **Behavior** -- returns a character string with the same length as argument-1, in which each lowercase letter is replaced by its uppercase equivalent. Characters that are not lowercase letters are returned unchanged.

```cobol
MOVE FUNCTION UPPER-CASE("hello world")
    TO WS-RESULT
*> WS-RESULT = "HELLO WORLD"

MOVE FUNCTION UPPER-CASE(WS-NAME) TO WS-NAME-UPPER
```

---

## Examples

### Building a Formatted Name

```cobol
       WORKING-STORAGE SECTION.
       01  WS-FIRST       PIC X(20) VALUE "  john  ".
       01  WS-LAST        PIC X(20) VALUE "  SMITH  ".
       01  WS-FORMATTED   PIC X(50).

       PROCEDURE DIVISION.
           MOVE FUNCTION CONCATENATE(
               FUNCTION UPPER-CASE(
                   FUNCTION TRIM(WS-FIRST))
               ", "
               FUNCTION UPPER-CASE(
                   FUNCTION TRIM(WS-LAST)))
               TO WS-FORMATTED
           DISPLAY WS-FORMATTED
           *> "JOHN, SMITH"
           STOP RUN.
```

### Validating Input Length

```cobol
       WORKING-STORAGE SECTION.
       01  WS-INPUT       PIC X(100).
       01  WS-TRIMMED     PIC X(100).
       01  WS-LENGTH      PIC 9(3).

       PROCEDURE DIVISION.
           ACCEPT WS-INPUT FROM CONSOLE
           MOVE FUNCTION TRIM(WS-INPUT) TO WS-TRIMMED
           COMPUTE WS-LENGTH =
               FUNCTION LENGTH(
                   FUNCTION TRIM(WS-INPUT))
           IF WS-LENGTH > 50
               DISPLAY "Input too long"
           ELSE IF WS-LENGTH = 0
               DISPLAY "Input required"
           ELSE
               PERFORM PROCESS-INPUT
           END-IF
           STOP RUN.
```

### Character Code Conversion

```cobol
       WORKING-STORAGE SECTION.
       01  WS-CHAR        PIC X(1).
       01  WS-ORD         PIC 9(3).
       01  WS-SHIFTED     PIC X(1).

       PROCEDURE DIVISION.
      *>   Shift a character by 1 position in the
      *>   collating sequence
           MOVE "A" TO WS-CHAR
           COMPUTE WS-ORD = FUNCTION ORD(WS-CHAR) + 1
           MOVE FUNCTION CHAR(WS-ORD) TO WS-SHIFTED
           DISPLAY WS-SHIFTED
           *> Displays "B"
           STOP RUN.
```

---

## See Also

- [Intrinsic Functions](index.md) -- overview of all intrinsic functions
- [Numeric Functions](numeric.md) -- mathematical intrinsic functions
- [Date and Time Functions](date-time.md) -- date and time intrinsic functions
- [Financial Functions](financial.md) -- financial calculation intrinsic functions
- [PICTURE](../../data-division/picture.md) -- data item format specification
- [USAGE](../../data-division/usage.md) -- internal representation and storage format
