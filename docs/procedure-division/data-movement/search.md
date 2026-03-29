# SEARCH and SEARCH ALL

The `SEARCH` statement performs a sequential or binary search on a table (an OCCURS item with an index) to locate an entry that satisfies a specified condition.

- **Standard:** COBOL-68 (SEARCH), COBOL-74 (SEARCH ALL), COBOL-85, COBOL 2002, COBOL 2014, COBOL 2023
- **Division:** Procedure Division
- **Category:** Table Handling

---

## Syntax

### Format 1 — Sequential Search (SEARCH)

```cobol
SEARCH identifier-1
    [ VARYING { identifier-2 | index-name-1 } ]
    [ AT END imperative-statement-1 ]
    WHEN condition-1 { imperative-statement-2 | NEXT SENTENCE }
    [ WHEN condition-2 { imperative-statement-3 | NEXT SENTENCE } ] ...
END-SEARCH
```

### Format 2 — Binary Search (SEARCH ALL)

```cobol
SEARCH ALL identifier-1
    [ AT END imperative-statement-1 ]
    WHEN { data-name-1 { = | EQUAL TO }
           { identifier-2 | literal-1 | arithmetic-expression-1 } }
         [ AND { data-name-2 { = | EQUAL TO } ... } ] ...
    { imperative-statement-2 | NEXT SENTENCE }
END-SEARCH
```

---

## Rules

### Sequential Search (SEARCH)

1. `identifier-1` must be a data item described with an OCCURS clause and an INDEXED BY phrase.
2. The index is automatically incremented after each unsuccessful comparison.
3. The search begins at the current index value — you must SET the index before searching if you want to start from the beginning.
4. `AT END` is executed if no WHEN condition is satisfied before the index exceeds the table size.
5. Multiple WHEN conditions can be specified; they are evaluated in order for each occurrence.
6. When a WHEN condition is true, the associated statement executes and the search ends.
7. `VARYING` allows a secondary index or identifier to be incremented in step with the primary index.

### Binary Search (SEARCH ALL)

1. The table must have an `ASCENDING KEY` or `DESCENDING KEY` phrase in its OCCURS clause.
2. The table **must be sorted** on the specified key(s) before SEARCH ALL is executed.
3. Only **equality** conditions are allowed (no GREATER THAN or LESS THAN).
4. Only one WHEN phrase is allowed (but it may contain AND conditions for compound keys).
5. The compiler generates a binary search algorithm — O(log n) performance.
6. The index is set by the runtime; do not SET it before a SEARCH ALL.

---

## Examples

### Sequential Search

```cobol
       WORKING-STORAGE SECTION.
       01  WS-STATE-TABLE.
           05  WS-STATE-ENTRY OCCURS 50 TIMES
               INDEXED BY STATE-IDX.
               10  WS-STATE-CODE   PIC X(2).
               10  WS-STATE-NAME   PIC X(20).
               10  WS-STATE-TAX    PIC 9V9(4).

       01  WS-LOOKUP-CODE   PIC X(2).
       01  WS-FOUND-TAX     PIC 9V9(4).

       PROCEDURE DIVISION.
           MOVE "IL" TO WS-LOOKUP-CODE
           SET STATE-IDX TO 1
           SEARCH WS-STATE-ENTRY
               AT END
                   DISPLAY "State not found"
               WHEN WS-STATE-CODE(STATE-IDX) =
                   WS-LOOKUP-CODE
                   MOVE WS-STATE-TAX(STATE-IDX)
                       TO WS-FOUND-TAX
                   DISPLAY "Tax rate: " WS-FOUND-TAX
           END-SEARCH
```

### Binary Search

```cobol
       WORKING-STORAGE SECTION.
       01  WS-PRODUCT-TABLE.
           05  WS-PRODUCT OCCURS 1000 TIMES
               ASCENDING KEY IS WS-PROD-ID
               INDEXED BY PROD-IDX.
               10  WS-PROD-ID     PIC X(8).
               10  WS-PROD-NAME   PIC X(30).
               10  WS-PROD-PRICE  PIC 9(5)V99.

       01  WS-SEARCH-ID    PIC X(8).

       PROCEDURE DIVISION.
           MOVE "PRD-0542" TO WS-SEARCH-ID
           SEARCH ALL WS-PRODUCT
               AT END
                   DISPLAY "Product not found"
               WHEN WS-PROD-ID(PROD-IDX) = WS-SEARCH-ID
                   DISPLAY "Found: "
                       WS-PROD-NAME(PROD-IDX)
                   DISPLAY "Price: "
                       WS-PROD-PRICE(PROD-IDX)
           END-SEARCH
```

### Compound Key Binary Search

```cobol
       01  WS-RATE-TABLE.
           05  WS-RATE OCCURS 500 TIMES
               ASCENDING KEY IS WS-REGION WS-CATEGORY
               INDEXED BY RATE-IDX.
               10  WS-REGION     PIC X(3).
               10  WS-CATEGORY   PIC X(5).
               10  WS-AMOUNT     PIC 9(7)V99.

       SEARCH ALL WS-RATE
           AT END
               DISPLAY "Rate not found"
           WHEN WS-REGION(RATE-IDX) = "NE"
               AND WS-CATEGORY(RATE-IDX) = "PRIME"
               MOVE WS-AMOUNT(RATE-IDX) TO WS-RESULT
       END-SEARCH
```

---

## SEARCH vs SEARCH ALL

| Aspect | SEARCH | SEARCH ALL |
|--------|--------|------------|
| Algorithm | Sequential (linear) | Binary (logarithmic) |
| Performance | O(n) | O(log n) |
| Table must be sorted | No | Yes |
| Conditions | Any condition | Equality only |
| Multiple WHEN | Yes | No (AND for compound keys) |
| Set index before | Yes (required) | No (set by runtime) |
| Standard | COBOL-68 | COBOL-74 |

---

## See Also

- [OCCURS Clause](../../data-division/occurs.md) -- table definitions
- [SET](set.md) -- index manipulation
- [PERFORM](../control-flow/perform.md) -- alternative iteration for searches
