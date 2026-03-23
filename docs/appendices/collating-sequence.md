# Collating Sequence

A collating sequence defines the ordering of characters used by a COBOL program for comparisons, sorting, and merging. The sequence determines which characters are considered "less than," "equal to," or "greater than" other characters. COBOL supports both the native collating sequence of the underlying character set and program-specified alternative sequences.

**Standard:** COBOL-60, COBOL-61, COBOL-61 Extended, COBOL-65, COBOL-68, COBOL-74, COBOL-85, COBOL 2002, COBOL 2014, COBOL 2023

---

## Overview

The collating sequence affects:

- **Relational conditions** --- The result of `IF A > B` depends on the ordinal position of each character in the active collating sequence.
- **SORT and MERGE operations** --- Records are ordered according to the collating sequence in effect for the sort or merge key.
- **Figurative constants** --- The values of `LOW-VALUE` and `HIGH-VALUE` correspond to the first and last characters in the collating sequence, respectively.
- **Range conditions** --- Evaluating whether a character falls within a range (e.g., `"A" THRU "Z"`) depends on the sequence.

Unless overridden, a COBOL program uses the native collating sequence of the platform's character set.

---

## Native Collating Sequence

The native collating sequence is determined by the underlying character encoding of the system on which the program executes. The two dominant encodings in COBOL environments are ASCII and EBCDIC.

### ASCII (American Standard Code for Information Interchange)

ASCII assigns a numeric code point (0--127) to each character. The collating sequence follows the numeric order of these code points.

| Range | Characters | Code Points |
|-------|-----------|-------------|
| Control characters | NUL, SOH, ... DEL | 0--31, 127 |
| Space | ` ` | 32 |
| Special characters | `!` `"` `#` `$` `%` `&` `'` `(` `)` `*` `+` `,` `-` `.` `/` | 33--47 |
| Digits | `0`--`9` | 48--57 |
| Special characters | `:` `;` `<` `=` `>` `?` `@` | 58--64 |
| Uppercase letters | `A`--`Z` | 65--90 |
| Special characters | `[` `\` `]` `^` `_` `` ` `` | 91--96 |
| Lowercase letters | `a`--`z` | 97--122 |
| Special characters | `{` `\|` `}` `~` | 123--126 |

In ASCII:

- Digits sort **before** uppercase letters.
- Uppercase letters sort **before** lowercase letters.
- `"1" < "A" < "a"` is true.

### EBCDIC (Extended Binary Coded Decimal Interchange Code)

EBCDIC is used primarily on IBM mainframe systems. Its character ordering differs substantially from ASCII.

| Range | Characters | Approximate Code Points |
|-------|-----------|------------------------|
| Control characters and space | NUL, ... SP | 0x00--0x40 |
| Special characters | `.` `<` `(` `+` `\|` `&` `!` `$` `*` `)` `;` etc. | 0x4B--0x6F |
| Lowercase letters | `a`--`z` (with gaps) | 0x81--0xA9 |
| Uppercase letters | `A`--`Z` (with gaps) | 0xC1--0xE9 |
| Digits | `0`--`9` | 0xF0--0xF9 |

In EBCDIC:

- Lowercase letters sort **before** uppercase letters.
- Letters sort **before** digits.
- `"a" < "A" < "1"` is true.

!!! warning "Key Difference"
    The relative ordering of letters and digits is reversed between ASCII and EBCDIC. Code that compares alphanumeric data containing mixed letters and digits may produce different results on ASCII and EBCDIC platforms. A comparison like `IF WS-CODE < "A"` is true for digit values in ASCII but false in EBCDIC.

---

## ASCII vs EBCDIC: Comparison Summary

| Comparison | ASCII Result | EBCDIC Result |
|-----------|-------------|---------------|
| `"A" < "a"` | True | False |
| `"a" < "A"` | False | True |
| `"1" < "A"` | True | False |
| `"A" < "1"` | False | True |
| `"Z" < "a"` | True | False |
| `SPACE < "0"` | True | True |
| `SPACE < "A"` | True | True |

---

## PROGRAM COLLATING SEQUENCE

The `PROGRAM COLLATING SEQUENCE` clause in the OBJECT-COMPUTER paragraph of the Environment Division specifies an alternative collating sequence for the entire program. This sequence applies to all nonnumeric comparisons and to any SORT or MERGE statement that does not specify its own collating sequence.

```cobol
ENVIRONMENT DIVISION.
CONFIGURATION SECTION.
OBJECT-COMPUTER. computer-name
    PROGRAM COLLATING SEQUENCE IS alphabet-name-1.
SPECIAL-NAMES.
    ALPHABET alphabet-name-1 IS STANDARD-1.
```

When this clause is specified, the named alphabet (defined in SPECIAL-NAMES) replaces the native sequence for program-wide comparisons.

!!! note
    The PROGRAM COLLATING SEQUENCE does not affect numeric comparisons. Numeric items are always compared by their algebraic values.

---

## ALPHABET Clause

The ALPHABET clause in the SPECIAL-NAMES paragraph defines a named collating sequence. Several predefined sequences are available, and user-defined sequences can be constructed.

### Predefined Alphabets

| Name | Meaning |
|------|---------|
| `STANDARD-1` | The ASCII character set (ISO 646 IRV / ASCII). On an EBCDIC platform, specifying STANDARD-1 causes comparisons to use ASCII ordering. |
| `STANDARD-2` | The ISO 646 International Reference Version character set. In most implementations, this is equivalent to STANDARD-1. |
| `NATIVE` | The native character set of the platform. This is the default if no alphabet is specified. |
| `EBCDIC` | The EBCDIC character set. On an ASCII platform, specifying EBCDIC causes comparisons to use EBCDIC ordering. (Implementation-defined; available in many compilers as an extension.) |

```cobol
SPECIAL-NAMES.
    ALPHABET ASCII-ORDER  IS STANDARD-1.
    ALPHABET EBCDIC-ORDER IS EBCDIC.
    ALPHABET NATIVE-ORDER IS NATIVE.
```

### User-Defined Alphabets

A user-defined alphabet specifies a custom ordering of characters. Characters are listed in the desired collating order, from lowest to highest.

```cobol
SPECIAL-NAMES.
    ALPHABET CUSTOM-ORDER IS
        SPACE
        "0" THRU "9"
        "A" THRU "Z"
        "a" THRU "z".
```

In this example, space is the lowest character, followed by digits, then uppercase letters, then lowercase letters. All characters not listed sort after the explicitly listed characters, in their native order.

Characters can also be specified by their ordinal position (numeric literal representing the code point):

```cobol
SPECIAL-NAMES.
    ALPHABET MY-ORDER IS
        32          *> space
        48 THRU 57  *> digits 0-9
        65 THRU 90  *> uppercase A-Z
        97 THRU 122. *> lowercase a-z
```

The `ALSO` keyword places multiple characters at the same ordinal position in the sequence, making them equivalent for comparison purposes:

```cobol
SPECIAL-NAMES.
    ALPHABET CASE-INSENSITIVE IS
        "A" ALSO "a"
        "B" ALSO "b"
        "C" ALSO "c"
        *> ... and so on for all letters
```

---

## COLLATING SEQUENCE in SORT and MERGE

The SORT and MERGE statements accept an optional `COLLATING SEQUENCE` phrase that overrides both the native sequence and any program-level collating sequence for that specific operation.

```cobol
SORT SORT-FILE
    ON ASCENDING KEY SORT-NAME
    COLLATING SEQUENCE IS ASCII-ORDER
    USING INPUT-FILE
    GIVING OUTPUT-FILE.
```

When specified, the named alphabet determines the ordering of the sort keys. When omitted, the SORT uses the program collating sequence (if specified) or the native sequence.

!!! note
    The COLLATING SEQUENCE phrase in SORT and MERGE affects only alphanumeric keys. Numeric keys are always sorted by algebraic value.

---

## Figurative Constants

The figurative constants `LOW-VALUE` and `HIGH-VALUE` are defined in terms of the program collating sequence.

| Figurative Constant | Meaning |
|---------------------|---------|
| `LOW-VALUE` / `LOW-VALUES` | The character with the lowest ordinal position in the program collating sequence. In ASCII, this is NUL (0x00). In EBCDIC, this is also NUL (0x00). |
| `HIGH-VALUE` / `HIGH-VALUES` | The character with the highest ordinal position in the program collating sequence. In ASCII, this is DEL (0x7F) or 0xFF for extended ASCII. In EBCDIC, this is 0xFF. |
| `SPACE` / `SPACES` | The space character (0x20 in ASCII, 0x40 in EBCDIC). |

When a user-defined alphabet is in effect, `LOW-VALUE` corresponds to the first character listed in the alphabet definition, and `HIGH-VALUE` corresponds to the last.

```cobol
SPECIAL-NAMES.
    ALPHABET MY-ALPHA IS "A" THRU "Z".

*> LOW-VALUE is "A", HIGH-VALUE is "Z" under MY-ALPHA.
```

!!! warning
    If a user-defined alphabet is specified as the program collating sequence, the values of `LOW-VALUE` and `HIGH-VALUE` change accordingly. Code that assumes `LOW-VALUE` is NUL or `HIGH-VALUE` is 0xFF may behave unexpectedly with a custom collating sequence.

---

## National (Unicode) Collating Sequence

COBOL 2002 introduced national data items (`PIC N` or `USAGE NATIONAL`) for Unicode character data. The collating sequence for national data items is based on the binary order of Unicode code points (UTF-16 or UTF-32, depending on implementation).

The national collating sequence is always based on the ordinal values of the national characters and is not affected by the PROGRAM COLLATING SEQUENCE clause or the ALPHABET clause, which apply only to alphanumeric data.

!!! note "COBOL 2002"
    National data items and the national collating sequence were introduced in COBOL 2002. Earlier standards do not support Unicode natively.

---

## Effect on Relational Conditions

When two alphanumeric or alphabetic items are compared using relational operators (`=`, `<`, `>`, `<=`, `>=`), the comparison proceeds character by character from left to right using the active collating sequence. The shorter operand is conceptually padded with spaces on the right.

```cobol
01 WS-NAME-A  PIC X(10)  VALUE "SMITH".
01 WS-NAME-B  PIC X(10)  VALUE "JONES".

*> In both ASCII and EBCDIC, "S" > "J", so:
IF WS-NAME-A > WS-NAME-B
    DISPLAY "SMITH sorts after JONES"
END-IF
*> This is true in both ASCII and EBCDIC.
```

The ordering becomes significant with mixed-case or mixed-type data:

```cobol
01 WS-CODE-A  PIC X(5)  VALUE "abc".
01 WS-CODE-B  PIC X(5)  VALUE "ABC".

*> In ASCII:  "a" (97) > "A" (65)  --> WS-CODE-A > WS-CODE-B
*> In EBCDIC: "a" (0x81) < "A" (0xC1)  --> WS-CODE-A < WS-CODE-B

IF WS-CODE-A > WS-CODE-B
    DISPLAY "Lowercase > uppercase (ASCII behavior)"
ELSE
    DISPLAY "Lowercase < uppercase (EBCDIC behavior)"
END-IF
```

---

## Practical Examples

### Sorting the Same Data on ASCII vs EBCDIC

Consider the following unsorted values:

```
"apple", "BANANA", "123", "Cherry", "456"
```

**ASCII sort order** (digits < uppercase < lowercase):

```
"123", "456", "BANANA", "Cherry", "apple"
```

**EBCDIC sort order** (lowercase < uppercase < digits):

```
"apple", "BANANA", "Cherry", "123", "456"
```

### Forcing ASCII Order on an EBCDIC System

```cobol
ENVIRONMENT DIVISION.
CONFIGURATION SECTION.
SOURCE-COMPUTER. IBM-390.
OBJECT-COMPUTER. IBM-390
    PROGRAM COLLATING SEQUENCE IS ASCII-SEQ.
SPECIAL-NAMES.
    ALPHABET ASCII-SEQ IS STANDARD-1.

*> All comparisons and SORT/MERGE operations now use
*> ASCII ordering, even on this EBCDIC system.
```

### Using a Custom Sequence for Case-Insensitive Sorting

```cobol
ENVIRONMENT DIVISION.
CONFIGURATION SECTION.
OBJECT-COMPUTER. X86-LINUX.
SPECIAL-NAMES.
    ALPHABET CASE-FOLD IS
        SPACE
        "0" THRU "9"
        "A" ALSO "a"
        "B" ALSO "b"
        "C" ALSO "c"
        "D" ALSO "d"
        "E" ALSO "e"
        "F" ALSO "f"
        "G" ALSO "g"
        "H" ALSO "h"
        "I" ALSO "i"
        "J" ALSO "j"
        "K" ALSO "k"
        "L" ALSO "l"
        "M" ALSO "m"
        "N" ALSO "n"
        "O" ALSO "o"
        "P" ALSO "p"
        "Q" ALSO "q"
        "R" ALSO "r"
        "S" ALSO "s"
        "T" ALSO "t"
        "U" ALSO "u"
        "V" ALSO "v"
        "W" ALSO "w"
        "X" ALSO "x"
        "Y" ALSO "y"
        "Z" ALSO "z".

*> With CASE-FOLD, "apple" and "APPLE" are considered equal.
SORT SORT-FILE
    ON ASCENDING KEY SORT-NAME
    COLLATING SEQUENCE IS CASE-FOLD
    USING INPUT-FILE
    GIVING OUTPUT-FILE.
```

### Demonstrating Figurative Constant Behavior

```cobol
01 WS-TEST  PIC X(5).

MOVE LOW-VALUE TO WS-TEST

IF WS-TEST < SPACE
    DISPLAY "LOW-VALUE is less than SPACE"
END-IF
*> True in both ASCII and EBCDIC (NUL < SPACE).

MOVE HIGH-VALUE TO WS-TEST

IF WS-TEST > "Z"
    DISPLAY "HIGH-VALUE is greater than Z"
END-IF
*> True in both ASCII and EBCDIC.
```

### Collating Sequence Impact on Conditional Logic

```cobol
01 WS-INPUT  PIC X(1).

ACCEPT WS-INPUT

*> This test behaves differently on ASCII vs EBCDIC:
IF WS-INPUT >= "A" AND WS-INPUT <= "Z"
    DISPLAY "Uppercase letter"
END-IF
*> In ASCII, digits and special characters below "A" (65)
*> are excluded. Characters "[" through "`" (91-96) are
*> incorrectly included.
*> In EBCDIC, letters are not contiguous. Characters in
*> the gaps between letter groups are incorrectly included.

*> The portable approach uses class conditions:
IF WS-INPUT IS ALPHABETIC-UPPER
    DISPLAY "Uppercase letter (portable)"
END-IF
```

!!! note "Portability"
    Range comparisons on alphanumeric data (e.g., `>= "A" AND <= "Z"`) are not portable between ASCII and EBCDIC because the character sets have different layouts and gaps. Use class conditions (`IS ALPHABETIC`, `IS ALPHABETIC-UPPER`, `IS ALPHABETIC-LOWER`, `IS NUMERIC`) for portable character classification.

---

## See Also

- [SELECT](../environment-division/select.md) --- file control entry in the Environment Division
- [PICTURE Clause](../data-division/picture.md) --- defines data item categories including alphanumeric and national
- [USAGE Clause](../data-division/usage.md) --- specifies internal storage representation including NATIONAL
