# SET

The `SET` statement establishes values for index data items, condition names, and pointer data items.

- **Standard:** COBOL-68, COBOL-85, COBOL 2002, COBOL 2014, COBOL 2023
- **Division:** Procedure Division
- **Category:** Data Movement

---

## Syntax

### Format 1 — Index Setting

```cobol
SET { index-name-1 | identifier-1 } ... TO
    { index-name-2 | identifier-2 | integer-1 }

SET index-name-1 ... UP BY { identifier-1 | integer-1 }

SET index-name-1 ... DOWN BY { identifier-1 | integer-1 }
```

### Format 2 — Condition Name

```cobol
SET condition-name-1 [ condition-name-2 ] ... TO TRUE

SET condition-name-1 [ condition-name-2 ] ... TO FALSE
```

### Format 3 — Pointer

```cobol
SET { pointer-1 | ADDRESS OF identifier-1 } TO
    { pointer-2 | ADDRESS OF identifier-2 | NULL | NULLS }
```

---

## Rules

### Index Setting (Format 1)

1. SET is the **only** way to assign a value to an index data item (defined by INDEXED BY in an OCCURS clause).
2. `SET index TO integer` sets the index to reference the occurrence at position `integer`.
3. `SET index UP BY n` increments the index by `n` occurrences.
4. `SET index DOWN BY n` decrements the index by `n` occurrences.
5. `SET identifier TO index` converts an index value to an integer and stores it in `identifier`.
6. Multiple receiving items can be set in a single statement.

### Condition Name (Format 2)

1. `SET condition-name TO TRUE` moves the first VALUE associated with the level-88 condition name to the conditional variable.
2. `SET condition-name TO FALSE` (COBOL 2002) moves the FALSE value to the conditional variable. The level-88 entry must include a `WHEN SET TO FALSE IS` phrase.
3. Multiple condition names can be set in a single statement.

### Pointer (Format 3)

1. Sets a USAGE POINTER item to the address of a data item or to NULL.
2. `ADDRESS OF identifier` obtains the memory address of a data item.
3. `SET ADDRESS OF based-item TO pointer` associates a LINKAGE SECTION item with dynamically allocated storage.

---

## Examples

### Index Operations

```cobol
       01  WS-TABLE.
           05  WS-ENTRY PIC X(20) OCCURS 50 TIMES
               INDEXED BY WS-IDX.

       SET WS-IDX TO 1
       DISPLAY WS-ENTRY(WS-IDX)

       SET WS-IDX UP BY 1
       *> WS-IDX now points to the 2nd occurrence

       SET WS-IDX DOWN BY 1
       *> WS-IDX now points to the 1st occurrence
```

### Condition Names

```cobol
       01  WS-STATUS     PIC X.
           88  ACTIVE     VALUE "A".
           88  INACTIVE   VALUE "I".
           88  DELETED    VALUE "D"
               WHEN SET TO FALSE IS "A".

       SET ACTIVE TO TRUE
       *> WS-STATUS = "A"

       SET INACTIVE TO TRUE
       *> WS-STATUS = "I"

       SET DELETED TO FALSE
       *> WS-STATUS = "A" (the FALSE value)
```

### Pointer Operations

```cobol
       WORKING-STORAGE SECTION.
       01  WS-PTR        USAGE POINTER.
       01  WS-RECORD.
           05  WS-NAME   PIC X(30).

       LINKAGE SECTION.
       01  LS-BUFFER     PIC X(100) BASED.

       PROCEDURE DIVISION.
           SET WS-PTR TO ADDRESS OF WS-RECORD
           SET WS-PTR TO NULL

           ALLOCATE LS-BUFFER RETURNING WS-PTR
           SET ADDRESS OF LS-BUFFER TO WS-PTR
           MOVE "Data" TO LS-BUFFER
           FREE LS-BUFFER
```

---

## See Also

- [OCCURS Clause](../../data-division/occurs.md) -- table definitions with INDEXED BY
- [Condition Names](../../data-division/condition-names.md) -- level-88 entries
- [SEARCH](search.md) -- table lookup using indexes
- [ALLOCATE and FREE](../memory/allocate-free.md) -- dynamic memory with pointers
