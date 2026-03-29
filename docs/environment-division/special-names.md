# SPECIAL-NAMES Paragraph

The `SPECIAL-NAMES` paragraph in the Configuration Section associates implementor-defined names with user-defined names, defines the currency symbol, overrides the decimal point character, establishes custom alphabet orderings, defines user class conditions, and assigns symbolic names to characters.

- **Standard:** COBOL-60, COBOL-68, COBOL-85, COBOL 2002, COBOL 2014

---

## Syntax

```cobol
CONFIGURATION SECTION.
SPECIAL-NAMES.
    [ implementor-name IS mnemonic-name ]
    [ ALPHABET alphabet-name IS
        { STANDARD-1 | STANDARD-2 | NATIVE | EBCDIC
        | literal THRU literal [ literal THRU literal ] ...
        | literal ALSO literal [ ALSO literal ] ... } ]
    [ SYMBOLIC CHARACTERS symbol-1 IS integer-1
                        [ symbol-2 IS integer-2 ] ...
        [ IN alphabet-name ] ]
    [ CLASS class-name IS
        { literal THRU literal | literal }
        [ { literal THRU literal | literal } ] ... ]
    [ CURRENCY SIGN IS literal
        [ WITH PICTURE SYMBOL literal ] ]
    [ DECIMAL-POINT IS COMMA ]
    [ CURSOR IS data-name ]
    [ CRT STATUS IS data-name ]
    [ LOCALE locale-name IS literal ]
    .
```

All clauses within SPECIAL-NAMES are optional. The paragraph is terminated by a period.

---

## Implementor-Names (Mnemonic Names)

Implementor-names associate system devices, switches, or features with user-defined mnemonic names. The available implementor-names are compiler-specific.

### Syntax

```cobol
SPECIAL-NAMES.
    implementor-name IS mnemonic-name.
```

### Common Implementor-Names

| Implementor-Name | Typical Meaning | Usage |
|------------------|----------------|-------|
| `CONSOLE` | Operator console | ACCEPT/DISPLAY target |
| `SYSIN` | Standard input | ACCEPT source |
| `SYSOUT` | Standard output | DISPLAY target |
| `SYSERR` | Standard error | DISPLAY target |
| `PRINTER` | System printer | DISPLAY target |
| `CRT` | Terminal screen | ACCEPT/DISPLAY target |
| `COMMAND-LINE` | Command line args | ACCEPT source |
| `ENVIRONMENT-NAME` | Environment variable name | SET source |
| `ENVIRONMENT-VALUE` | Environment variable value | SET/ACCEPT source |

### Example

```cobol
       SPECIAL-NAMES.
           CONSOLE IS TTY
           SYSERR IS ERROUT.

       PROCEDURE DIVISION.
           DISPLAY "Normal output" UPON TTY
           DISPLAY "Error message" UPON ERROUT
           STOP RUN.
```

### Switches

External switches can be tested in the program using ON/OFF status:

```cobol
       SPECIAL-NAMES.
           SWITCH-1 IS DEBUG-SWITCH
               ON STATUS IS DEBUG-ON
               OFF STATUS IS DEBUG-OFF.

       DATA DIVISION.
       WORKING-STORAGE SECTION.
       88  DEBUG-ON       VALUE 1.
       88  DEBUG-OFF      VALUE 0.

       PROCEDURE DIVISION.
           IF DEBUG-ON
               DISPLAY "Debug mode is active"
           END-IF
           STOP RUN.
```

### Environment Variables

Accessing environment variables (IBM, Micro Focus, GnuCOBOL):

```cobol
       SPECIAL-NAMES.
           ENVIRONMENT-NAME IS ENV-NAME
           ENVIRONMENT-VALUE IS ENV-VALUE.

       DATA DIVISION.
       WORKING-STORAGE SECTION.
       01  WS-HOME         PIC X(200).

       PROCEDURE DIVISION.
           MOVE "HOME" TO ENV-NAME
           ACCEPT WS-HOME FROM ENV-VALUE
           DISPLAY "Home directory: " WS-HOME
           STOP RUN.
```

---

## CURRENCY SIGN

Defines the currency symbol used in `PICTURE` character strings.

### Syntax

```cobol
SPECIAL-NAMES.
    CURRENCY SIGN IS literal
        [ WITH PICTURE SYMBOL literal ].
```

- **literal** -- a single character or (in COBOL 2002+) a multi-character string.
- **WITH PICTURE SYMBOL** -- specifies a separate character to use in PICTURE strings when the currency sign is multi-character.

### Rules

- The default currency sign is `$`.
- The currency sign character cannot be a digit, the letters A, B, C, D, P, R, S, V, X, Z, the space, or any special PICTURE character (+, -, *, /, comma, period, semicolon).
- COBOL 2002 permits multi-character currency signs (e.g., "EUR", "GBP").

### Examples

```cobol
      *> Single-character currency
       SPECIAL-NAMES.
           CURRENCY SIGN IS "£".

       01  WS-AMOUNT  PIC £££,£££.99.
```

```cobol
      *> Multi-character currency (COBOL 2002+)
       SPECIAL-NAMES.
           CURRENCY SIGN IS "EUR"
               WITH PICTURE SYMBOL "E".

       01  WS-AMOUNT  PIC EEE,EEE.99.
      *>   Displays as: EUR  1,250.75
```

```cobol
      *> Multiple currencies in the same program
       SPECIAL-NAMES.
           CURRENCY SIGN IS "USD"
               WITH PICTURE SYMBOL "$"
           CURRENCY SIGN IS "EUR"
               WITH PICTURE SYMBOL "E"
           CURRENCY SIGN IS "GBP"
               WITH PICTURE SYMBOL "£".

       01  WS-USD-AMT  PIC $$$,$$$.99.
       01  WS-EUR-AMT  PIC EEE,EEE.99.
       01  WS-GBP-AMT  PIC £££,£££.99.
```

---

## DECIMAL-POINT IS COMMA

Exchanges the roles of the period and comma in PICTURE character strings and numeric literals throughout the program.

### Syntax

```cobol
SPECIAL-NAMES.
    DECIMAL-POINT IS COMMA.
```

### Effect

| Without clause | With clause |
|---------------|-------------|
| `PIC 9(5).99` -- period is decimal point | `PIC 9(5),99` -- comma is decimal point |
| `PIC 9(3),999` -- comma is grouping separator | `PIC 9(3).999` -- period is grouping separator |
| `MOVE 1234.56 TO ...` | `MOVE 1234,56 TO ...` |

### Example

```cobol
       SPECIAL-NAMES.
           DECIMAL-POINT IS COMMA.

       DATA DIVISION.
       WORKING-STORAGE SECTION.
       01  WS-PRICE    PIC 9(5),99.
       01  WS-QUANTITY  PIC ZZ.ZZZ.

       PROCEDURE DIVISION.
           MOVE 1250,75 TO WS-PRICE
           MOVE 15000 TO WS-QUANTITY
           DISPLAY WS-PRICE
           *> Displays: 01250,75
           DISPLAY WS-QUANTITY
           *> Displays: 15.000
           STOP RUN.
```

This is commonly used in European locales where the comma is the decimal separator.

---

## ALPHABET

Defines a named collating sequence for use in comparisons, SORT, and MERGE operations.

### Syntax

```cobol
SPECIAL-NAMES.
    ALPHABET alphabet-name IS
        { STANDARD-1 | STANDARD-2 | NATIVE | EBCDIC
        | literal THRU literal
          [ literal THRU literal ] ...
        | literal ALSO literal [ ALSO literal ] ... }.
```

### Predefined Alphabets

| Name | Meaning |
|------|---------|
| `STANDARD-1` | ASCII (ISO 646 IRV) |
| `STANDARD-2` | ISO 646 (7-bit) |
| `NATIVE` | System default (EBCDIC on mainframes, ASCII on others) |
| `EBCDIC` | EBCDIC (IBM extension, not in the standard) |

### Custom Alphabet Example

```cobol
       SPECIAL-NAMES.
           ALPHABET CUSTOM-ORDER IS
               "0" THRU "9"
               "A" THRU "Z"
               "a" THRU "z".

       OBJECT-COMPUTER. X86-64
           PROGRAM COLLATING SEQUENCE IS CUSTOM-ORDER.
```

With this custom alphabet, digits sort before uppercase letters, which sort before lowercase letters.

### Character Equivalence with ALSO

The `ALSO` keyword declares characters as equivalent in the collating sequence:

```cobol
       SPECIAL-NAMES.
           ALPHABET CASE-INSENSITIVE IS
               "A" ALSO "a"
               "B" ALSO "b"
               "C" ALSO "c"
               "D" ALSO "d"
               ...
               "Z" ALSO "z".
```

This creates a collating sequence where uppercase and lowercase letters are treated as equal in comparisons.

---

## CLASS

Defines a user-specified class condition that tests whether a data item contains only characters from a specified set.

### Syntax

```cobol
SPECIAL-NAMES.
    CLASS class-name IS
        { literal-1 THRU literal-2 | literal-3 }
        [ { literal-4 THRU literal-5 | literal-6 } ] ...
```

### Rules

- The class-name may be used in conditional expressions with `IS class-name`.
- The test returns true if every character of the data item is within one of the specified ranges or literal values.
- Character ranges follow the program collating sequence.

### Examples

```cobol
       SPECIAL-NAMES.
           CLASS VALID-NAME IS
               "A" THRU "Z"
               "a" THRU "z"
               SPACE "-" "'"
           CLASS VALID-DIGIT IS
               "0" THRU "9"
           CLASS HEX-CHAR IS
               "0" THRU "9"
               "A" THRU "F"
               "a" THRU "f".

       PROCEDURE DIVISION.
           IF WS-INPUT IS VALID-NAME
               DISPLAY "Valid name"
           END-IF

           IF WS-CODE IS HEX-CHAR
               DISPLAY "Valid hex string"
           END-IF
```

### Practical Validation Example

```cobol
       SPECIAL-NAMES.
           CLASS VALID-PHONE IS
               "0" THRU "9" "-" "(" ")" SPACE "+".

       DATA DIVISION.
       WORKING-STORAGE SECTION.
       01  WS-PHONE       PIC X(15).

       PROCEDURE DIVISION.
           ACCEPT WS-PHONE FROM CONSOLE
           IF WS-PHONE IS VALID-PHONE
               PERFORM PROCESS-PHONE
           ELSE
               DISPLAY "Invalid phone number format"
           END-IF
           STOP RUN.
```

---

## SYMBOLIC CHARACTERS

Assigns user-defined names to characters identified by their ordinal position in the program collating sequence.

### Syntax

```cobol
SPECIAL-NAMES.
    SYMBOLIC CHARACTERS symbol-1 IS integer-1
                      [ symbol-2 IS integer-2 ] ...
        [ IN alphabet-name ].
```

The integer specifies the ordinal position (1-based) in the collating sequence, not the code point value.

### Example

```cobol
       SPECIAL-NAMES.
           SYMBOLIC CHARACTERS
               SYM-TAB IS 10
               SYM-LF IS 11
               SYM-CR IS 14.

       PROCEDURE DIVISION.
           INSPECT WS-INPUT
               REPLACING ALL SYM-TAB BY SPACE
           INSPECT WS-INPUT
               REPLACING ALL SYM-CR BY SPACE
               REPLACING ALL SYM-LF BY SPACE
           STOP RUN.
```

!!! warning
    The ordinal position is collating-sequence-dependent, not code-point-dependent.
    In ASCII, ordinal 10 is the character with code point 9 (horizontal tab).
    In EBCDIC, ordinal 10 is a different character. Always verify against your
    platform's collating sequence.

---

## CURSOR and CRT STATUS

Used with screen-based I/O (ACCEPT/DISPLAY with Screen Section).

### Syntax

```cobol
SPECIAL-NAMES.
    CURSOR IS data-name-1
    CRT STATUS IS data-name-2.
```

- **CURSOR** -- a 4- or 6-digit numeric field that receives or sets the cursor position (line and column) after a screen ACCEPT.
- **CRT STATUS** -- a 3-character field that receives a status code indicating how the user terminated the ACCEPT (Enter, function key, etc.).

### Example

```cobol
       SPECIAL-NAMES.
           CURSOR IS WS-CURSOR-POS
           CRT STATUS IS WS-CRT-STATUS.

       DATA DIVISION.
       WORKING-STORAGE SECTION.
       01  WS-CURSOR-POS.
           05  WS-CURSOR-LINE  PIC 99.
           05  WS-CURSOR-COL   PIC 99.
       01  WS-CRT-STATUS.
           05  WS-CRT-KEY1     PIC 9.
           05  WS-CRT-KEY2     PIC 99.

       SCREEN SECTION.
       01  INPUT-SCREEN.
           05  LINE 5 COL 10 VALUE "Name: ".
           05  LINE 5 COL 16 PIC X(20)
               USING WS-NAME.

       PROCEDURE DIVISION.
           DISPLAY INPUT-SCREEN
           ACCEPT INPUT-SCREEN
           EVALUATE WS-CRT-KEY1
               WHEN 0
                   DISPLAY "Enter pressed"
               WHEN 1
                   DISPLAY "Function key: " WS-CRT-KEY2
           END-EVALUATE
           STOP RUN.
```

---

## Complete Example

```cobol
       ENVIRONMENT DIVISION.
       CONFIGURATION SECTION.
       SOURCE-COMPUTER. X86-64.
       OBJECT-COMPUTER. X86-64
           PROGRAM COLLATING SEQUENCE IS ASCII-SEQ.

       SPECIAL-NAMES.
           CONSOLE IS TTY
           ENVIRONMENT-NAME IS ENV-NAME
           ENVIRONMENT-VALUE IS ENV-VALUE
           ALPHABET ASCII-SEQ IS STANDARD-1
           SYMBOLIC CHARACTERS SYM-TAB IS 10
           CLASS VALID-AMOUNT IS
               "0" THRU "9" "." "-" "+" "$" ","
           CLASS ALPHA-ONLY IS
               "A" THRU "Z" "a" THRU "z" SPACE
           CURRENCY SIGN IS "$"
           DECIMAL-POINT IS PERIOD.
```

---

## See Also

- [Environment Division](index.md) -- division overview
- [SELECT](select.md) -- file-control entries
- [PICTURE Clause](../data-division/picture.md) -- data format specification
- [Collating Sequence](../appendices/collating-sequence.md) -- character ordering
- [Character Set](../language/character-set.md) -- COBOL character set
