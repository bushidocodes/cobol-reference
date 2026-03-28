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

## BYTE-LENGTH

Returns the length of a data item in bytes.

!!! note "COBOL 2002"
    The BYTE-LENGTH function was introduced in COBOL 2002.

```cobol
FUNCTION BYTE-LENGTH ( argument-1 )
```

- **argument-1** -- any category. May be a literal, data-name, or another function reference.
- **Return category** -- integer.
- **Behavior** -- returns the number of bytes occupied by argument-1. For alphanumeric items, BYTE-LENGTH and LENGTH return the same value. For national (UTF-16) items, BYTE-LENGTH returns twice the value of LENGTH because each national character occupies 2 bytes. For items with `USAGE COMP` or other non-display usages, BYTE-LENGTH returns the actual storage size in bytes.

```cobol
01  WS-ALPHA   PIC X(10).
01  WS-NATIONAL PIC N(10).
01  WS-COMP    PIC 9(8) USAGE COMP.

COMPUTE WS-LEN = FUNCTION BYTE-LENGTH(WS-ALPHA)
*> WS-LEN = 10

COMPUTE WS-LEN = FUNCTION BYTE-LENGTH(WS-NATIONAL)
*> WS-LEN = 20 (10 national chars * 2 bytes each)

COMPUTE WS-LEN = FUNCTION BYTE-LENGTH(WS-COMP)
*> WS-LEN = 4 (binary storage for 9(8))

COMPUTE WS-LEN = FUNCTION LENGTH(WS-NATIONAL)
*> WS-LEN = 10 (character positions, not bytes)
```

### BYTE-LENGTH vs LENGTH

| Data Item | LENGTH | BYTE-LENGTH |
|-----------|--------|-------------|
| `PIC X(20)` | 20 | 20 |
| `PIC N(10)` (national/UTF-16) | 10 | 20 |
| `PIC 9(8) COMP` | 8 | 4 |
| `PIC 9(8) COMP-3` | 8 | 5 |
| `"HELLO"` (literal) | 5 | 5 |

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

## CHAR-NATIONAL

Returns the national character at a specified ordinal position.

!!! note "COBOL 2002"
    The CHAR-NATIONAL function was introduced in COBOL 2002.

```cobol
FUNCTION CHAR-NATIONAL ( argument-1 )
```

- **argument-1** -- integer, in the range 1 through the number of positions in the national character set.
- **Return category** -- national, length 1.
- **Behavior** -- returns the national character that occupies ordinal position argument-1 in the national character set. CHAR-NATIONAL is the national character equivalent of CHAR; where CHAR returns an alphanumeric character from the program collating sequence, CHAR-NATIONAL returns a national character from the national character set.

```cobol
MOVE FUNCTION CHAR-NATIONAL(66) TO WS-NCHAR
*> Returns the national character at ordinal position 66

MOVE FUNCTION CHAR-NATIONAL(1) TO WS-FIRST-NCHAR
*> Returns the first character in the national character set
```

---

## CONCAT

Returns the concatenation of the argument strings.

!!! note "COBOL 2014"
    The CONCAT function was introduced in COBOL 2014 as the revised shorter name for CONCATENATE. Both names may be used interchangeably; CONCATENATE remains available as a synonym for backward compatibility.

```cobol
FUNCTION CONCAT ( argument-1 ... )
```

- **argument-1 ...** -- one or more arguments of category alphanumeric or national. All arguments must be of the same category.
- **Return category** -- same as the argument category. The length of the result is the sum of the lengths of all arguments.
- **Behavior** -- returns a character string that is the concatenation of the arguments in the order specified. No padding or trimming is performed; trailing spaces in each argument are preserved in the result. The behavior is identical to CONCATENATE.

```cobol
MOVE FUNCTION CONCAT(WS-FIRST " " WS-LAST)
    TO WS-FULL-NAME

MOVE FUNCTION CONCAT("Hello" ", " "World" "!")
    TO WS-GREETING
*> WS-GREETING = "Hello, World!"
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

## DISPLAY-OF

Converts a national character string to alphanumeric representation.

!!! note "COBOL 2002"
    The DISPLAY-OF function was introduced in COBOL 2002.

```cobol
FUNCTION DISPLAY-OF ( argument-1 [ argument-2 ] )
```

- **argument-1** -- national. The national character string to convert.
- **argument-2** -- optional, alphanumeric. The code page to use for the conversion. If omitted, the system default code page is used.
- **Return category** -- alphanumeric.
- **Behavior** -- converts a national (Unicode/UTF-16) character string to its alphanumeric representation. DISPLAY-OF is the inverse of NATIONAL-OF. The result length depends on the national characters and the target code page, as some national characters may require multiple alphanumeric positions.

```cobol
01  WS-NATIONAL   PIC N(10).
01  WS-ALPHA      PIC X(20).

MOVE FUNCTION DISPLAY-OF(WS-NATIONAL) TO WS-ALPHA
*> Converts national string to alphanumeric using default
*> code page

MOVE FUNCTION DISPLAY-OF(WS-NATIONAL "UTF-8")
    TO WS-ALPHA
*> Converts national string to alphanumeric using UTF-8
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

## LENGTH-AN

Returns the length of an argument in alphanumeric character positions.

!!! note "COBOL 2002"
    The LENGTH-AN function was introduced in COBOL 2002.

```cobol
FUNCTION LENGTH-AN ( argument-1 )
```

- **argument-1** -- any category. May be a literal, data-name, or another function reference.
- **Return category** -- integer.
- **Behavior** -- returns the length of argument-1 measured in alphanumeric character positions, regardless of the argument's actual category. For alphanumeric items, LENGTH-AN returns the same value as LENGTH. For national items, LENGTH-AN returns the number of alphanumeric character positions that would be needed to represent the item, which may differ from the number of national character positions returned by LENGTH.

```cobol
01  WS-ALPHA     PIC X(10).
01  WS-NATIONAL  PIC N(10).

COMPUTE WS-LEN = FUNCTION LENGTH-AN(WS-ALPHA)
*> WS-LEN = 10 (same as LENGTH for alphanumeric)

COMPUTE WS-LEN = FUNCTION LENGTH-AN(WS-NATIONAL)
*> Returns the number of alphanumeric positions needed
*> (may differ from FUNCTION LENGTH(WS-NATIONAL) which
*>  returns 10 national character positions)
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

## NATIONAL-OF

Converts an alphanumeric string to national (Unicode/UTF-16) representation.

!!! note "COBOL 2002"
    The NATIONAL-OF function was introduced in COBOL 2002.

```cobol
FUNCTION NATIONAL-OF ( argument-1 [ argument-2 ] )
```

- **argument-1** -- alphanumeric. The alphanumeric string to convert.
- **argument-2** -- optional, alphanumeric. The code page of argument-1. If omitted, the system default code page is assumed.
- **Return category** -- national.
- **Behavior** -- converts an alphanumeric character string to its national (Unicode/UTF-16) representation. NATIONAL-OF is the inverse of DISPLAY-OF. The result length in national character positions depends on the source characters and the source code page.

```cobol
01  WS-ALPHA     PIC X(20).
01  WS-NATIONAL  PIC N(20).

MOVE FUNCTION NATIONAL-OF(WS-ALPHA) TO WS-NATIONAL
*> Converts alphanumeric string to national using default
*> code page

MOVE FUNCTION NATIONAL-OF(WS-ALPHA "IBM-1047")
    TO WS-NATIONAL
*> Converts from EBCDIC code page IBM-1047 to national
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

## STANDARD-COMPARE

Compares two strings using standardized comparison rules.

!!! note "COBOL 2002"
    The STANDARD-COMPARE function was introduced in COBOL 2002.

```cobol
FUNCTION STANDARD-COMPARE ( argument-1  argument-2
    [ argument-3 [ argument-4 ] ] )
```

- **argument-1** -- alphanumeric or national. The first string to compare.
- **argument-2** -- alphanumeric or national. The second string to compare.
- **argument-3** -- optional, alphanumeric. The comparison method to use for alphanumeric arguments.
- **argument-4** -- optional, alphanumeric. The comparison method to use for national arguments.
- **Return category** -- alphanumeric, length 1.
- **Behavior** -- compares argument-1 and argument-2 using standardized comparison rules (as opposed to the program collating sequence used by normal comparison or the locale-specific rules used by LOCALE-COMPARE) and returns:
    - `"<"` if argument-1 is less than argument-2
    - `"="` if argument-1 equals argument-2
    - `">"` if argument-1 is greater than argument-2

The optional argument-3 and argument-4 parameters allow specification of the comparison method for alphanumeric and national arguments respectively.

```cobol
MOVE FUNCTION STANDARD-COMPARE("apple" "banana")
    TO WS-RESULT
*> WS-RESULT = "<"

MOVE FUNCTION STANDARD-COMPARE("ABC" "ABC")
    TO WS-RESULT
*> WS-RESULT = "="

MOVE FUNCTION STANDARD-COMPARE(WS-STR-1 WS-STR-2)
    TO WS-RESULT
IF WS-RESULT = ">"
    DISPLAY "First string is greater"
END-IF
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

## LOCALE-COMPARE

Returns a value indicating the result of comparing two strings according to the rules of a specified locale.

!!! note "COBOL 2002"
    The LOCALE-COMPARE function was introduced in COBOL 2002.

```cobol
FUNCTION LOCALE-COMPARE ( argument-1  argument-2 [ argument-3 ] )
```

- **argument-1** -- alphanumeric or national. The first string.
- **argument-2** -- alphanumeric or national. The second string. Must be the same category as argument-1.
- **argument-3** -- optional, alphanumeric. The locale identifier. If omitted, the system default locale is used.
- **Return category** -- alphanumeric, length 1.
- **Behavior** -- compares argument-1 and argument-2 according to the cultural sorting rules of the specified locale and returns:
    - `"<"` if argument-1 is less than argument-2
    - `"="` if argument-1 equals argument-2
    - `">"` if argument-1 is greater than argument-2

Locale-aware comparison differs from standard collating sequence comparison in that it accounts for language-specific rules such as accent ordering, case folding, and multi-character collation elements (e.g., in German, "ä" sorts near "a"; in Swedish, "ä" sorts after "z").

```cobol
MOVE FUNCTION LOCALE-COMPARE("apple" "banana")
    TO WS-RESULT
*> WS-RESULT = "<"

MOVE FUNCTION LOCALE-COMPARE("straße" "strasse" "de_DE")
    TO WS-RESULT
*> Result depends on German locale collation rules

MOVE FUNCTION LOCALE-COMPARE("résumé" "resume" "fr_FR")
    TO WS-RESULT
*> French locale determines how accented characters compare
```

### Practical Example

```cobol
       WORKING-STORAGE SECTION.
       01  WS-NAME-1       PIC X(30) VALUE "Müller".
       01  WS-NAME-2       PIC X(30) VALUE "Mueller".
       01  WS-CMP          PIC X(1).

       PROCEDURE DIVISION.
           MOVE FUNCTION LOCALE-COMPARE(
               WS-NAME-1 WS-NAME-2 "de_DE")
               TO WS-CMP
           EVALUATE WS-CMP
               WHEN "<"
                   DISPLAY WS-NAME-1 " sorts before "
                           WS-NAME-2
               WHEN "="
                   DISPLAY WS-NAME-1 " equals "
                           WS-NAME-2
               WHEN ">"
                   DISPLAY WS-NAME-1 " sorts after "
                           WS-NAME-2
           END-EVALUATE
           STOP RUN.
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
