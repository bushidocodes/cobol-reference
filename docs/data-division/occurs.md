---
search:
  boost: 2
tags:
  - array
  - table
---

# OCCURS Clause — Arrays and Tables

The OCCURS clause defines tables (arrays) by specifying that a data item is repeated a fixed or variable number of times. The OCCURS clause is the COBOL equivalent of an array in other programming languages.

**Standard:** COBOL-60, COBOL-61, COBOL-61 Extended, COBOL-65, COBOL-68, COBOL-85, COBOL 2002, COBOL 2014, COBOL 2023

---

## Syntax

### Fixed-Length Table

```cobol
level-number  data-name  OCCURS integer-1 TIMES
    [ASCENDING|DESCENDING KEY IS data-name-1 [data-name-2 ...]] ...
    [INDEXED BY index-name-1 [index-name-2 ...]]
    [picture-and-other-clauses].
```

### Variable-Length Table

```cobol
level-number  data-name  OCCURS integer-1 TO integer-2 TIMES
    DEPENDING ON data-name-d
    [ASCENDING|DESCENDING KEY IS data-name-1 [data-name-2 ...]] ...
    [INDEXED BY index-name-1 [index-name-2 ...]]
    [picture-and-other-clauses].
```

---

## Description

The OCCURS clause specifies that the data item (and all of its subordinate items, if it is a group) is repeated to form a table. Each repetition is called an **element** (or **occurrence**) of the table. Elements are accessed by subscript or index.

---

## Fixed-Length Tables

```cobol
       01  MONTHLY-TOTALS.
           05  MONTH-AMOUNT  PIC S9(7)V99  OCCURS 12 TIMES.
```

The table `MONTH-AMOUNT` contains exactly 12 elements. The total storage allocated is 12 * 9 = 108 bytes (USAGE DISPLAY).

### Rules

1. The integer specifies the exact number of occurrences. It must be a positive integer.
2. The OCCURS clause may not appear on a level-01 or level-77 item.
3. A data item with an OCCURS clause, or any item subordinate to it, may not have a `VALUE` clause in COBOL-85. COBOL 2002 permits `VALUE` on items with OCCURS.

---

## Variable-Length Tables

```cobol
       01  TRANSACTION-TABLE.
           05  TXN-COUNT       PIC 99.
           05  TXN-ENTRY       OCCURS 1 TO 50 TIMES
                               DEPENDING ON TXN-COUNT.
               10  TXN-DATE    PIC 9(8).
               10  TXN-AMOUNT  PIC S9(7)V99.
```

The number of active elements in `TXN-ENTRY` is determined at runtime by the current value of `TXN-COUNT`.

### Rules

1. `integer-1` is the minimum number of occurrences (may be zero in COBOL 2002 and later; must be at least 1 in COBOL-85).
2. `integer-2` is the maximum number of occurrences.
3. `data-name-d` (the DEPENDING ON object) must be a positive integer data item. It must not be subordinate to the item containing the OCCURS clause.
4. Storage is allocated for the maximum number of occurrences (`integer-2`).
5. At any point in execution, only the first `n` occurrences (where `n` = the current value of `data-name-d`) are considered active. References to occurrences beyond `n` produce undefined results.
6. The DEPENDING ON item must be set before the variable-length table is referenced.
7. In the File Section, only the last (or only) variable-length table in a record determines the variable portion of the record length.

---

## KEY IS Phrase

```cobol
       01  EMPLOYEE-TABLE.
           05  EMP-COUNT       PIC 9(3).
           05  EMP-ENTRY       OCCURS 1 TO 500 TIMES
                               DEPENDING ON EMP-COUNT
                               ASCENDING KEY IS EMP-ID
                               INDEXED BY EMP-IDX.
               10  EMP-ID      PIC 9(6).
               10  EMP-NAME    PIC X(30).
               10  EMP-SALARY  PIC S9(7)V99.
```

The `KEY IS` phrase identifies one or more data items within the table entry that define the ordering of the table. It is required when using the `SEARCH ALL` (binary search) statement.

### Rules

1. `ASCENDING` indicates the key values are in ascending order (low to high).
2. `DESCENDING` indicates the key values are in descending order.
3. Multiple keys may be specified; the first named key is the primary key.
4. The COBOL runtime does not enforce the ordering -- the programmer is responsible for ensuring the table is sorted before using `SEARCH ALL`.
5. Key data items must be subordinate to the item with the OCCURS clause.

---

## INDEXED BY Phrase

```cobol
       05  RATE-ENTRY  OCCURS 10 TIMES
                       INDEXED BY RATE-IDX, RATE-IDX-2.
```

The `INDEXED BY` phrase creates one or more **index names** associated with the table. An index name holds a byte displacement (offset), not an occurrence number.

### Rules

1. Index names are not data items -- they are not defined elsewhere in the Data Division.
2. Index names are implicitly allocated by the compiler.
3. An index name may be used only in a `SET`, `SEARCH`, `PERFORM VARYING`, or relational condition referencing its associated table.
4. Index names are set with `SET index-name TO integer`, `SET index-name UP BY integer`, or `SET index-name DOWN BY integer`.
5. Multiple index names may be declared for a single table (useful for nested searches or saving an index position).

---

## Subscripting vs. Indexing

COBOL provides two mechanisms for referencing table elements.

### Subscripting

A **subscript** is an integer data item or literal enclosed in parentheses that specifies an occurrence number (1-based).

```cobol
       MOVE MONTH-AMOUNT(3) TO WS-TOTAL.
       MOVE MONTH-AMOUNT(WS-MONTH) TO WS-TOTAL.
```

- Subscripts are 1-based: the first element is element 1.
- A subscript may be an integer literal, a data item with an integer value, or an arithmetic expression (COBOL 2002).
- Multiple subscripts are separated by commas or spaces for multi-dimensional tables.
- Relative subscripting (e.g., `ITEM(SUB + 1)`) is permitted in COBOL-85 and later.

### Indexing

An **index** is a value set by `SET`, `SEARCH`, or `PERFORM VARYING` that represents a byte offset internally but is treated as an occurrence number in source code.

```cobol
       SET RATE-IDX TO 1.
       MOVE RATE-ENTRY(RATE-IDX) TO WS-RATE.
```

- Indexing is generally more efficient than subscripting because the compiler can use the offset directly.
- An index name may only reference the table it is associated with.
- An index may be saved to an index data item (`USAGE INDEX`) for later use.

---

## Nested Tables (Multi-Dimensional)

Tables may be nested to create multi-dimensional arrays. Each level of nesting requires its own OCCURS clause and its own subscript or index for reference.

```cobol
       01  SALES-TABLE.
           05  REGION-ENTRY    OCCURS 4 TIMES
                               INDEXED BY REG-IDX.
               10  MONTH-ENTRY OCCURS 12 TIMES
                               INDEXED BY MON-IDX.
                   15  SALES-AMOUNT  PIC S9(9)V99.
```

Referencing an element requires two subscripts:

```cobol
       MOVE SALES-AMOUNT(2, 6)  TO WS-AMOUNT.
       MOVE SALES-AMOUNT(REG-IDX, MON-IDX) TO WS-AMOUNT.
```

### Maximum Dimensions

The COBOL-85 standard permits up to **three** levels of nesting (three-dimensional tables). COBOL 2002 raises the limit to **seven** dimensions. Most implementations support at least three.

### Three-Dimensional Example

```cobol
       01  CUBE-TABLE.
           05  DIMENSION-1     OCCURS 10 TIMES.
               10  DIMENSION-2 OCCURS 20 TIMES.
                   15  DIMENSION-3
                                OCCURS 30 TIMES.
                       20  CELL-VALUE  PIC S9(5)V99.
```

Total elements: 10 * 20 * 30 = 6,000. Each reference requires three subscripts:

```cobol
       MOVE CELL-VALUE(3, 12, 7) TO WS-RESULT.
```

---

## Reference Modification

Reference modification extracts a substring from a data item by specifying a starting position and optional length:

```cobol
       data-name(start:length)
```

Reference modification may be combined with subscripting:

```cobol
       MOVE EMP-NAME(EMP-IDX)(1:10) TO WS-SHORT-NAME.
```

This moves the first 10 characters of the selected `EMP-NAME` element.

### Rules

1. `start` is a 1-based byte position. It must be a positive integer.
2. `length`, if omitted, extends to the end of the data item.
3. `start` and `length` may be arithmetic expressions.
4. Reference modification is valid on any alphanumeric or national data item.

---

## SEARCH Statement Interaction

### Sequential Search (SEARCH)

```cobol
       SET EMP-IDX TO 1.
       SEARCH EMP-ENTRY
           AT END
               DISPLAY "Not found"
           WHEN EMP-ID(EMP-IDX) = WS-TARGET-ID
               DISPLAY "Found: " EMP-NAME(EMP-IDX)
       END-SEARCH.
```

`SEARCH` performs a sequential (linear) search from the current index position.

### Binary Search (SEARCH ALL)

```cobol
       SEARCH ALL EMP-ENTRY
           AT END
               DISPLAY "Not found"
           WHEN EMP-ID(EMP-IDX) = WS-TARGET-ID
               DISPLAY "Found: " EMP-NAME(EMP-IDX)
       END-SEARCH.
```

`SEARCH ALL` performs a binary search. The table must have an `ASCENDING` or `DESCENDING KEY IS` clause and must be sorted on that key.

---

## Rules Summary

1. The OCCURS clause may not appear on level-01, level-66, level-77, or level-88 items.
2. An item with OCCURS may have a REDEFINES clause, but an item that is the object of a REDEFINES may not have an OCCURS clause.
3. In COBOL-85, an item with OCCURS may not have a VALUE clause. In COBOL 2002, VALUE is permitted.
4. Variable-length tables (`OCCURS ... DEPENDING ON`) may not be nested in COBOL-85. COBOL 2002 permits nested variable-length tables.
5. Out-of-range subscript or index values produce undefined behavior. Many implementations offer runtime bounds checking as a compile option.
6. The `INITIALIZE` statement initializes all occurrences of a table according to their categories.

---

## Examples

### Fixed Table with Initialization (COBOL 2002)

```cobol
       01  DAY-NAMES.
           05  DAY-NAME  PIC X(9) OCCURS 7 TIMES
               VALUE "Monday" "Tuesday" "Wednesday"
                     "Thursday" "Friday" "Saturday"
                     "Sunday".
```

### Fixed Table with Initialization (COBOL-85 Idiom)

```cobol
       01  DAY-NAME-DATA.
           05  FILLER  PIC X(9) VALUE "Monday   ".
           05  FILLER  PIC X(9) VALUE "Tuesday  ".
           05  FILLER  PIC X(9) VALUE "Wednesday".
           05  FILLER  PIC X(9) VALUE "Thursday ".
           05  FILLER  PIC X(9) VALUE "Friday   ".
           05  FILLER  PIC X(9) VALUE "Saturday ".
           05  FILLER  PIC X(9) VALUE "Sunday   ".
       01  DAY-NAME-TABLE REDEFINES DAY-NAME-DATA.
           05  DAY-NAME  PIC X(9) OCCURS 7 TIMES.
```

### Variable-Length Table with Search

```cobol
       01  PRODUCT-TABLE.
           05  PROD-COUNT        PIC 9(3) VALUE ZERO.
           05  PROD-ENTRY        OCCURS 0 TO 999 TIMES
                                 DEPENDING ON PROD-COUNT
                                 ASCENDING KEY IS PROD-CODE
                                 INDEXED BY PROD-IDX.
               10  PROD-CODE     PIC X(8).
               10  PROD-DESC     PIC X(30).
               10  PROD-PRICE    PIC S9(5)V99 COMP-3.

       PROCEDURE DIVISION.
           SEARCH ALL PROD-ENTRY
               AT END
                   DISPLAY "Product not found"
               WHEN PROD-CODE(PROD-IDX) = "WIDGET01"
                   DISPLAY PROD-DESC(PROD-IDX)
                   DISPLAY PROD-PRICE(PROD-IDX)
           END-SEARCH.
```

### Nested Table with PERFORM

```cobol
       01  MATRIX.
           05  MAT-ROW          OCCURS 10 TIMES
                                INDEXED BY ROW-IDX.
               10  MAT-COL      OCCURS 10 TIMES
                                INDEXED BY COL-IDX.
                   15  MAT-CELL PIC S9(5)V99 COMP-3.

       PROCEDURE DIVISION.
           PERFORM VARYING ROW-IDX FROM 1 BY 1
                   UNTIL ROW-IDX > 10
               PERFORM VARYING COL-IDX FROM 1 BY 1
                       UNTIL COL-IDX > 10
                   MOVE ZERO TO MAT-CELL(ROW-IDX, COL-IDX)
               END-PERFORM
           END-PERFORM.
```

---

## See Also

- [Data Division](index.md)
- [Level Numbers](level-numbers.md)
- [PICTURE Clause](picture.md)
- [USAGE Clause](usage.md)
- [SEARCH Statement](../procedure-division/data-movement/search.md)
- [PERFORM Statement](../procedure-division/control-flow/perform.md)
- [SET Statement](../procedure-division/data-movement/set.md)
- [REDEFINES Clause](redefines.md)
