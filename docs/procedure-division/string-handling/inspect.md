# INSPECT

The `INSPECT` statement tallies, replaces, or converts occurrences of single characters or groups of characters within a data item.

- **Standard:** COBOL-74, COBOL-85, COBOL 2002, COBOL 2014, COBOL 2023

## Syntax

### Format 1: TALLYING

```cobol
INSPECT identifier-1 TALLYING
        { identifier-2 FOR
          { CHARACTERS
          | { ALL | LEADING } {identifier-3 | literal-1} ...
          }
          [ { BEFORE | AFTER } INITIAL {identifier-4 | literal-2} ] ...
        } ...
```

### Format 2: REPLACING

```cobol
INSPECT identifier-1 REPLACING
        { CHARACTERS BY {identifier-5 | literal-3}
          [ { BEFORE | AFTER } INITIAL {identifier-6 | literal-4} ]
        | { ALL | LEADING | FIRST } {identifier-3 | literal-1}
          BY {identifier-5 | literal-3}
          [ { BEFORE | AFTER } INITIAL {identifier-6 | literal-4} ] ...
        } ...
```

### Format 3: TALLYING and REPLACING

```cobol
INSPECT identifier-1
        TALLYING
          { identifier-2 FOR
            { CHARACTERS
            | { ALL | LEADING } {identifier-3 | literal-1} ...
            }
            [ { BEFORE | AFTER } INITIAL {identifier-4 | literal-2} ] ...
          } ...
        REPLACING
          { CHARACTERS BY {identifier-5 | literal-3}
            [ { BEFORE | AFTER } INITIAL {identifier-6 | literal-4} ]
          | { ALL | LEADING | FIRST } {identifier-3 | literal-1}
            BY {identifier-5 | literal-3}
            [ { BEFORE | AFTER } INITIAL {identifier-6 | literal-4} ] ...
          } ...
```

### Format 4: CONVERTING

```cobol
INSPECT identifier-1
        CONVERTING {identifier-7 | literal-5}
        TO         {identifier-8 | literal-6}
        [ { BEFORE | AFTER } INITIAL {identifier-4 | literal-2} ] ...
```

## Rules

1. **Identifier-1** (the inspected field) must reference an alphanumeric, alphabetic, national, or alphanumeric-edited data item.

2. **Identifier-2** (the tallying counter) must reference an elementary integer data item. The `INSPECT` statement does **not** initialize this field; the programmer must set it to the desired initial value (typically zero) before execution.

3. In **Format 2** (`REPLACING`), when the `CHARACTERS` phrase is specified, the replacement item (identifier-5 or literal-3) must be a single character.

4. In **Format 2** (`REPLACING`), when `ALL`, `LEADING`, or `FIRST` is specified, the replacement item must have the same length as the item being replaced.

5. In **Format 4** (`CONVERTING`), the `FROM` string (identifier-7 or literal-5) and the `TO` string (identifier-8 or literal-6) must have the same length. Each character in the `FROM` string maps positionally to the corresponding character in the `TO` string.

6. All literals may be alphanumeric or national literals. Figurative constants (without the `ALL` qualifier, except `NULL` / `NULLS`) may be used.

!!! note "COBOL-85 and later"
    Format 4 (`CONVERTING`) was introduced in COBOL-85. Programs targeting COBOL-74 must use Format 2 (`REPLACING ALL`) to achieve equivalent functionality.

## Behavior

### TALLYING (Format 1)

The `INSPECT` statement examines identifier-1 from left to right. For each tally clause:

- **CHARACTERS**: Identifier-2 is incremented by 1 for each character in the inspected field (subject to `BEFORE` / `AFTER` restrictions).
- **ALL identifier-3 / literal-1**: Identifier-2 is incremented by 1 for each non-overlapping occurrence of the specified string found in the inspected field.
- **LEADING identifier-3 / literal-1**: Identifier-2 is incremented by 1 for each contiguous occurrence of the specified string found at the beginning of the area being inspected (or the area defined by `AFTER INITIAL`). Counting stops when a non-matching character is encountered.

### REPLACING (Format 2)

The `INSPECT` statement examines identifier-1 from left to right. For each replace clause:

- **CHARACTERS BY**: Each character in the inspected field is replaced by the specified single character.
- **ALL**: Every non-overlapping occurrence of the match string is replaced by the replacement string.
- **LEADING**: Contiguous occurrences of the match string at the beginning of the inspected area are replaced. Replacement stops when a non-matching character is encountered.
- **FIRST**: Only the first (leftmost) occurrence of the match string is replaced.

### TALLYING and REPLACING (Format 3)

The `TALLYING` operation is performed first, and then the `REPLACING` operation is performed on the (original) inspected field. The two operations are independent; the tallying result does not influence the replacement.

### CONVERTING (Format 4)

Each character in identifier-1 is compared to the characters in the `FROM` string. When a match is found, the character is replaced by the character at the corresponding position in the `TO` string. Each character in identifier-1 is evaluated only once (left to right).

Format 4 is functionally equivalent to a series of `REPLACING ALL` clauses, one for each character pair in the `FROM` / `TO` strings.

### BEFORE / AFTER INITIAL

The `BEFORE INITIAL` and `AFTER INITIAL` phrases restrict the portion of identifier-1 that is inspected:

- **BEFORE INITIAL identifier-4 / literal-2**: Only characters to the left of the first occurrence of the specified value are inspected. If the value is not found, the entire field is inspected.
- **AFTER INITIAL identifier-4 / literal-2**: Only characters to the right of the first occurrence of the specified value are inspected. If the value is not found, no characters are inspected.

## Examples

### Format 1: Counting specific characters

```cobol
       WORKING-STORAGE SECTION.
       01  WS-TEXT         PIC X(20) VALUE "HELLO WORLD".
       01  WS-L-COUNT      PIC 99    VALUE 0.
       01  WS-CHAR-COUNT   PIC 99    VALUE 0.

       PROCEDURE DIVISION.
           INSPECT WS-TEXT
                   TALLYING WS-L-COUNT FOR ALL "L".
      *>   WS-L-COUNT = 03

           INSPECT WS-TEXT
                   TALLYING WS-CHAR-COUNT FOR CHARACTERS
                   BEFORE INITIAL SPACE.
      *>   WS-CHAR-COUNT = 05 (counts "HELLO")
```

### Format 1: Counting leading characters

```cobol
       WORKING-STORAGE SECTION.
       01  WS-AMOUNT       PIC X(10) VALUE "000012345".
       01  WS-ZEROES       PIC 99    VALUE 0.

       PROCEDURE DIVISION.
           INSPECT WS-AMOUNT
                   TALLYING WS-ZEROES FOR LEADING "0".
      *>   WS-ZEROES = 04
```

### Format 2: Replacing characters

```cobol
       WORKING-STORAGE SECTION.
       01  WS-PHONE        PIC X(12) VALUE "555.867.5309".

       PROCEDURE DIVISION.
           INSPECT WS-PHONE
                   REPLACING ALL "." BY "-".
      *>   WS-PHONE = "555-867-5309"
```

### Format 2: Replacing leading zeros with spaces

```cobol
       WORKING-STORAGE SECTION.
       01  WS-DISPLAY-AMT  PIC X(10) VALUE "0000012345".

       PROCEDURE DIVISION.
           INSPECT WS-DISPLAY-AMT
                   REPLACING LEADING "0" BY SPACE.
      *>   WS-DISPLAY-AMT = "     12345"
```

### Format 2: Replacing only the first occurrence

```cobol
       WORKING-STORAGE SECTION.
       01  WS-DATA         PIC X(15) VALUE "ABCABCABC".

       PROCEDURE DIVISION.
           INSPECT WS-DATA
                   REPLACING FIRST "ABC" BY "XYZ".
      *>   WS-DATA = "XYZABCABC      "
```

### Format 3: Combined tallying and replacing

```cobol
       WORKING-STORAGE SECTION.
       01  WS-INPUT        PIC X(20) VALUE "aAbBcCaBc".
       01  WS-A-COUNT      PIC 99    VALUE 0.

       PROCEDURE DIVISION.
           INSPECT WS-INPUT
                   TALLYING WS-A-COUNT FOR ALL "a"
                   REPLACING ALL "a" BY "X".
      *>   WS-A-COUNT = 02
      *>   WS-INPUT   = "XAbBcCXBc           "
```

### Format 4: Converting lowercase to uppercase

```cobol
       WORKING-STORAGE SECTION.
       01  WS-MESSAGE      PIC X(20) VALUE "hello, world!".

       PROCEDURE DIVISION.
           INSPECT WS-MESSAGE
                   CONVERTING "abcdefghijklmnopqrstuvwxyz"
                   TO         "ABCDEFGHIJKLMNOPQRSTUVWXYZ".
      *>   WS-MESSAGE = "HELLO, WORLD!       "
```

### Format 4: Converting with AFTER INITIAL

```cobol
       WORKING-STORAGE SECTION.
       01  WS-RECORD       PIC X(20) VALUE "HDR:abc123def456".

       PROCEDURE DIVISION.
           INSPECT WS-RECORD
                   CONVERTING "abcdef"
                   TO         "ABCDEF"
                   AFTER INITIAL ":".
      *>   WS-RECORD = "HDR:ABC123DEF456    "
      *>   Characters before ":" are not affected.
```

## See also

- [STRING](string.md) -- concatenates multiple fields into a single data item
- [UNSTRING](unstring.md) -- splits a data item into multiple fields
- [PICTURE clause](../../data-division/picture.md) -- defines the category and editing of a data item
- MOVE statement -- transfers data between data items
