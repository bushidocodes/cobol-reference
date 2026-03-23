# STRING

The `STRING` statement concatenates the partial or complete contents of one or more data items or literals into a single data item.

- **Standard:** COBOL-74, COBOL-85, COBOL 2002, COBOL 2014, COBOL 2023

## Syntax

```cobol
STRING {identifier-1 | literal-1} ...
       DELIMITED BY {identifier-2 | literal-2 | SIZE}
     [ {identifier-3 | literal-3} ...
       DELIMITED BY {identifier-4 | literal-4 | SIZE} ] ...
       INTO identifier-5
     [ WITH POINTER identifier-6 ]
     [ ON OVERFLOW imperative-statement-1 ]
     [ NOT ON OVERFLOW imperative-statement-2 ]
     [ END-STRING ]
```

## Rules

1. **Identifier-1, identifier-3** (source items) and **literal-1, literal-3** are the sending fields. Each sending field or group of sending fields is associated with a `DELIMITED BY` phrase.

2. **Identifier-5** (the receiving field) must reference an alphanumeric or national data item and must not be reference-modified. It is not padded with spaces when the transferred data does not fill it completely.

3. **Identifier-2, identifier-4** (delimiter items) must reference alphanumeric, alphabetic, or national data items. **Literal-2, literal-4** must be alphanumeric or national literals. The delimiter itself is not transferred to the receiving field.

4. **Identifier-6** (the pointer field) must reference an elementary integer data item large enough to contain a value equal to the length of identifier-5 plus 1. If the `POINTER` phrase is not specified, the `STRING` statement behaves as if the pointer were set to 1 before execution.

5. All literals may be any figurative constant without the `ALL` qualifier, except `NULL` / `NULLS`.

6. None of the identifiers in the `STRING` statement may reference a level-88 entry.

## Behavior

### Transfer of data

The `STRING` statement transfers characters from each source field to the receiving field (identifier-5) according to the following rules:

- Characters from each source field are transferred to the receiving field in the order in which the source fields appear in the `STRING` statement.
- Characters are placed into the receiving field starting at the position indicated by the pointer value (1-based).
- For each source field, characters are transferred beginning with the leftmost character and proceeding from left to right.

### DELIMITED BY SIZE

When `DELIMITED BY SIZE` is specified, the entire contents of the source field are transferred. No scanning for a delimiter takes place.

### DELIMITED BY literal or identifier

When `DELIMITED BY identifier-2` or `DELIMITED BY literal-2` is specified, characters from the source field are transferred one at a time until:

- The delimiter is encountered in the source field, or
- The end of the source field is reached.

The delimiter characters themselves are **not** transferred to the receiving field.

### POINTER clause

When the `WITH POINTER` phrase is specified:

1. The value of identifier-6 (the pointer) indicates the leftmost position within the receiving field where the first transferred character is stored.
2. After each character is transferred, the pointer value is incremented by 1.
3. When the `STRING` statement is complete, the pointer contains a value equal to its initial value plus the number of characters transferred.
4. The pointer is **1-based**: a value of 1 refers to the first character position of the receiving field.

!!! note "Programmer responsibility"
    The initial value of the pointer must be set by the programmer before the `STRING` statement is executed. Failure to initialize the pointer causes unpredictable results.

### ON OVERFLOW

The overflow condition occurs when:

- The value of the pointer (explicit or implicit) is less than 1.
- The value of the pointer exceeds the length of the receiving field.

When an overflow condition occurs during execution:

- The `STRING` operation is terminated.
- The `ON OVERFLOW` imperative statement, if specified, is executed.

When the `STRING` statement completes without an overflow condition, the `NOT ON OVERFLOW` imperative statement, if specified, is executed.

!!! info "COBOL-85 and later"
    The `END-STRING` explicit scope terminator delimits the scope of the `STRING` statement. It permits the use of `STRING` within `IF` and other conditional statements without the need for periods. `END-STRING` was introduced in COBOL-85.

## Examples

### Basic concatenation of two fields

```cobol
       WORKING-STORAGE SECTION.
       01  WS-FIRST-NAME     PIC X(10) VALUE "John".
       01  WS-LAST-NAME      PIC X(10) VALUE "Smith".
       01  WS-FULL-NAME      PIC X(25) VALUE SPACES.

       PROCEDURE DIVISION.
           STRING WS-FIRST-NAME DELIMITED BY "  "
                  " "           DELIMITED BY SIZE
                  WS-LAST-NAME  DELIMITED BY "  "
                  INTO WS-FULL-NAME
           END-STRING.
      *>   WS-FULL-NAME = "John Smith              "
      *>   Note: trailing spaces in source fields act as
      *>   effective delimiters via "  " (two spaces).
```

### Using the POINTER clause

```cobol
       WORKING-STORAGE SECTION.
       01  WS-OUTPUT          PIC X(30) VALUE SPACES.
       01  WS-POINTER         PIC 99    VALUE 1.
       01  WS-CITY            PIC X(10) VALUE "Seattle".
       01  WS-STATE           PIC X(2)  VALUE "WA".
       01  WS-ZIP             PIC X(5)  VALUE "98101".

       PROCEDURE DIVISION.
           STRING WS-CITY  DELIMITED BY "  "
                  ", "      DELIMITED BY SIZE
                  WS-STATE DELIMITED BY SIZE
                  " "       DELIMITED BY SIZE
                  WS-ZIP   DELIMITED BY SIZE
                  INTO WS-OUTPUT
                  WITH POINTER WS-POINTER
           END-STRING.
      *>   WS-OUTPUT = "Seattle, WA 98101         "
      *>   WS-POINTER = 21
```

### Handling overflow

```cobol
       WORKING-STORAGE SECTION.
       01  WS-RESULT          PIC X(10) VALUE SPACES.
       01  WS-LONG-VALUE      PIC X(20)
                               VALUE "This is a long value".

       PROCEDURE DIVISION.
           STRING WS-LONG-VALUE DELIMITED BY SIZE
                  INTO WS-RESULT
                  ON OVERFLOW
                     DISPLAY "Result truncated"
                  NOT ON OVERFLOW
                     DISPLAY "Transfer complete"
           END-STRING.
      *>   WS-RESULT = "This is a "
      *>   "Result truncated" is displayed.
```

### Building a delimited record

```cobol
       WORKING-STORAGE SECTION.
       01  WS-NAME            PIC X(10) VALUE "Alice".
       01  WS-DEPT            PIC X(5)  VALUE "Sales".
       01  WS-ID              PIC X(4)  VALUE "1234".
       01  WS-RECORD          PIC X(30) VALUE SPACES.

       PROCEDURE DIVISION.
           STRING WS-NAME DELIMITED BY "  "
                  "|"      DELIMITED BY SIZE
                  WS-DEPT DELIMITED BY "  "
                  "|"      DELIMITED BY SIZE
                  WS-ID   DELIMITED BY SIZE
                  INTO WS-RECORD
           END-STRING.
      *>   WS-RECORD = "Alice|Sales|1234          "
```

## See also

- [UNSTRING](unstring.md) -- splits a data item into multiple fields
- [INSPECT](inspect.md) -- tallies, replaces, or converts characters within a data item
- [PICTURE clause](../../data-division/picture.md) -- defines the category and editing of a data item
- MOVE statement -- transfers data between data items
