# UNSTRING

The `UNSTRING` statement splits the contents of a single sending field into multiple receiving fields, based on one or more delimiters.

- **Standard:** COBOL-74, COBOL-85, COBOL 2002, COBOL 2014, COBOL 2023

## Syntax

```cobol
UNSTRING identifier-1
       DELIMITED BY [ALL] {identifier-2 | literal-1}
     [ OR [ALL] {identifier-3 | literal-2} ] ...
       INTO identifier-4 [ DELIMITER IN identifier-5 ]
                          [ COUNT IN identifier-6 ]
          [ identifier-7 [ DELIMITER IN identifier-8 ]
                          [ COUNT IN identifier-9 ] ] ...
     [ WITH POINTER identifier-10 ]
     [ TALLYING IN identifier-11 ]
     [ ON OVERFLOW imperative-statement-1 ]
     [ NOT ON OVERFLOW imperative-statement-2 ]
     [ END-UNSTRING ]
```

## Rules

1. **Identifier-1** (the sending field) must reference an alphanumeric or national data item.

2. **Identifier-2, identifier-3** (delimiter identifiers) must reference alphanumeric, alphabetic, or national data items. **Literal-1, literal-2** must be alphanumeric or national literals. Figurative constants (other than `ALL` literal and `NULL` / `NULLS`) may be used.

3. **Identifier-4, identifier-7** (receiving fields) must reference alphabetic, alphanumeric, alphanumeric-edited, national, national-edited, or numeric data items. If a receiving field is a numeric data item, it must be described as an integer without the symbol `P` in the [PICTURE](../../data-division/picture.md) string.

4. **Identifier-5, identifier-8** (delimiter receiving fields) must reference alphanumeric or national data items. When specified, the delimiter that caused the separation is stored in this field.

5. **Identifier-6, identifier-9** (count fields) must reference integer data items. When specified, the count of characters transferred to the corresponding receiving field is stored in this field.

6. **Identifier-10** (pointer field) must reference an elementary integer data item large enough to contain a value equal to the length of identifier-1 plus 1.

7. **Identifier-11** (tallying field) must reference an integer data item. It is incremented by 1 for each receiving field acted upon.

## Behavior

### Examination of the sending field

The `UNSTRING` statement examines the sending field (identifier-1) beginning at the position indicated by the pointer value (or position 1, if the `POINTER` phrase is not specified). The characters are examined from left to right.

### Delimiter scanning

When the `DELIMITED BY` phrase is specified:

- Characters in the sending field are compared to the delimiter(s) in the order the delimiters appear in the statement.
- When a match is found, the characters preceding the delimiter are transferred to the current receiving field.
- The delimiter itself is **not** transferred to the receiving field. If the `DELIMITER IN` phrase is specified, the delimiter is stored in the corresponding delimiter receiving field.
- Examination then continues with the character immediately following the delimiter.

### ALL phrase

When `ALL` is specified before a delimiter:

- Multiple contiguous occurrences of that delimiter are treated as a single delimiter.
- Without `ALL`, each occurrence of the delimiter produces a separate (potentially empty) receiving field.

### Multiple delimiters with OR

When `OR` is used, multiple delimiters can be specified. The sending field is examined for each delimiter in the order listed. The first matching delimiter at any position is used.

### POINTER clause

When the `WITH POINTER` phrase is specified:

1. The pointer value indicates the starting position (1-based) within the sending field where examination begins.
2. As characters are examined, the pointer is incremented by 1 for each character.
3. When the `UNSTRING` statement is complete, the pointer contains the position of the character following the last character examined.

!!! note "Programmer responsibility"
    The initial value of the pointer must be set by the programmer before the `UNSTRING` statement is executed.

### TALLYING IN clause

When the `TALLYING IN` phrase is specified:

- Identifier-11 is incremented by 1 for each receiving field acted upon during execution.
- The tallying field is **not** initialized to zero by the `UNSTRING` statement. The programmer must initialize it before execution if a fresh count is required.

### Receiving field processing

- If the number of characters transferred to a receiving field is less than the size of that field, the field is padded with spaces (for alphanumeric fields) or zeros (for numeric fields).
- If the number of characters exceeds the size of the receiving field, truncation occurs (on the left for numeric fields, on the right for alphanumeric fields).

### ON OVERFLOW

The overflow condition occurs when:

- The value of the pointer (explicit or implicit) is less than 1.
- The value of the pointer exceeds the length of the sending field.
- All receiving fields have been acted upon, but the sending field still contains unexamined characters.

When an overflow condition occurs:

- The `UNSTRING` operation is terminated.
- The `ON OVERFLOW` imperative statement, if specified, is executed.

When the `UNSTRING` statement completes without an overflow condition, the `NOT ON OVERFLOW` imperative statement, if specified, is executed.

!!! info "COBOL-85 and later"
    The `END-UNSTRING` explicit scope terminator delimits the scope of the `UNSTRING` statement. It permits the use of `UNSTRING` within conditional statements without the need for periods. `END-UNSTRING` was introduced in COBOL-85.

## Examples

### Splitting a name into parts

```cobol
       WORKING-STORAGE SECTION.
       01  WS-FULL-NAME       PIC X(30)
                               VALUE "John Adam Smith".
       01  WS-FIRST-NAME      PIC X(15).
       01  WS-MIDDLE-NAME     PIC X(15).
       01  WS-LAST-NAME       PIC X(15).

       PROCEDURE DIVISION.
           UNSTRING WS-FULL-NAME
                    DELIMITED BY SPACE
                    INTO WS-FIRST-NAME
                         WS-MIDDLE-NAME
                         WS-LAST-NAME
           END-UNSTRING.
      *>   WS-FIRST-NAME  = "John           "
      *>   WS-MIDDLE-NAME = "Adam           "
      *>   WS-LAST-NAME   = "Smith          "
```

### Parsing with DELIMITER IN and COUNT IN

```cobol
       WORKING-STORAGE SECTION.
       01  WS-INPUT            PIC X(30)
                               VALUE "100,USD,2025-01-15".
       01  WS-AMOUNT           PIC X(10).
       01  WS-CURRENCY         PIC X(5).
       01  WS-DATE-VAL         PIC X(12).
       01  WS-DELIM-1          PIC X.
       01  WS-DELIM-2          PIC X.
       01  WS-COUNT-1          PIC 99.
       01  WS-COUNT-2          PIC 99.
       01  WS-COUNT-3          PIC 99.

       PROCEDURE DIVISION.
           UNSTRING WS-INPUT
                    DELIMITED BY ","
                    INTO WS-AMOUNT   DELIMITER IN WS-DELIM-1
                                     COUNT IN WS-COUNT-1
                         WS-CURRENCY DELIMITER IN WS-DELIM-2
                                     COUNT IN WS-COUNT-2
                         WS-DATE-VAL COUNT IN WS-COUNT-3
           END-UNSTRING.
      *>   WS-AMOUNT   = "100       "   WS-COUNT-1 = 03
      *>   WS-CURRENCY = "USD  "        WS-COUNT-2 = 03
      *>   WS-DATE-VAL = "2025-01-15  " WS-COUNT-3 = 10
      *>   WS-DELIM-1  = ","            WS-DELIM-2 = ","
```

### Using ALL to handle repeated delimiters

```cobol
       WORKING-STORAGE SECTION.
       01  WS-TEXT             PIC X(30)
                               VALUE "one   two   three".
       01  WS-WORD-1           PIC X(10).
       01  WS-WORD-2           PIC X(10).
       01  WS-WORD-3           PIC X(10).

       PROCEDURE DIVISION.
           UNSTRING WS-TEXT
                    DELIMITED BY ALL SPACES
                    INTO WS-WORD-1
                         WS-WORD-2
                         WS-WORD-3
           END-UNSTRING.
      *>   WS-WORD-1 = "one       "
      *>   WS-WORD-2 = "two       "
      *>   WS-WORD-3 = "three     "
      *>   Without ALL, WS-WORD-2 and WS-WORD-3 would contain
      *>   empty/partial results due to each space being treated
      *>   as a separate delimiter.
```

### Parsing with multiple delimiters and TALLYING

```cobol
       WORKING-STORAGE SECTION.
       01  WS-CSV-LINE         PIC X(40)
                               VALUE "Smith,Jane;Accounting".
       01  WS-LAST             PIC X(15).
       01  WS-FIRST            PIC X(15).
       01  WS-DEPT             PIC X(15).
       01  WS-FIELD-COUNT      PIC 99 VALUE 0.
       01  WS-PTR              PIC 99 VALUE 1.

       PROCEDURE DIVISION.
           UNSTRING WS-CSV-LINE
                    DELIMITED BY "," OR ";"
                    INTO WS-LAST
                         WS-FIRST
                         WS-DEPT
                    WITH POINTER WS-PTR
                    TALLYING IN WS-FIELD-COUNT
                    ON OVERFLOW
                       DISPLAY "Not all data processed"
                    NOT ON OVERFLOW
                       DISPLAY "Parse complete"
           END-UNSTRING.
      *>   WS-LAST        = "Smith          "
      *>   WS-FIRST       = "Jane           "
      *>   WS-DEPT        = "Accounting     "
      *>   WS-FIELD-COUNT = 03
      *>   WS-PTR         = 22
```

### Processing CSV records in a loop

```cobol
       WORKING-STORAGE SECTION.
       01  WS-CSV-RECORD       PIC X(50)
                               VALUE "10,20,30,40,50".
       01  WS-PTR              PIC 99 VALUE 1.
       01  WS-FIELD            PIC X(10).
       01  WS-TOTAL            PIC 9(5) VALUE 0.
       01  WS-NUM              PIC 9(5).
       01  WS-DONE-FLAG        PIC 9 VALUE 0.
         88  WS-DONE           VALUE 1.

       PROCEDURE DIVISION.
           PERFORM UNTIL WS-DONE
               UNSTRING WS-CSV-RECORD
                        DELIMITED BY ","
                        INTO WS-FIELD
                        WITH POINTER WS-PTR
                        ON OVERFLOW
                           CONTINUE
                        NOT ON OVERFLOW
                           SET WS-DONE TO TRUE
               END-UNSTRING
               MOVE WS-FIELD TO WS-NUM
               ADD WS-NUM TO WS-TOTAL
           END-PERFORM.
      *>   WS-TOTAL = 00150 (10 + 20 + 30 + 40 + 50)
```

## See also

- [STRING](string.md) -- concatenates multiple fields into a single data item
- [INSPECT](inspect.md) -- tallies, replaces, or converts characters within a data item
- [PICTURE clause](../../data-division/picture.md) -- defines the category and editing of a data item
- MOVE statement -- transfers data between data items
