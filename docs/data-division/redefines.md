# REDEFINES Clause

The REDEFINES clause allows a storage area previously defined for one data item to be described differently by another data item, enabling the same physical bytes of storage to be interpreted in multiple ways.

**Standard:** COBOL-60, COBOL-61, COBOL-61 Extended, COBOL-65, COBOL-68, COBOL-85, COBOL 2002, COBOL 2014, COBOL 2023

---

## Syntax

```cobol
level-number  data-name-1  REDEFINES  data-name-2
    [picture-and-other-clauses].
```

`data-name-1` is the **redefining item** and `data-name-2` is the **redefined item**. Both items share the same starting byte position in storage.

---

## Rules

### Level Number Rules

1. The level numbers of `data-name-1` and `data-name-2` must be identical.
2. The REDEFINES clause may appear on any level from 01 through 49. It may not appear on level 66, 77, or 88 items.
3. The level-01 entry of a record implicitly redefines the storage of any preceding level-01 entry in the same File Description (FD) entry, so an explicit REDEFINES on level-01 items is only required in WORKING-STORAGE and LOCAL-STORAGE.

### Placement Rules

1. In COBOL-85 and later, the data description entry for `data-name-1` must immediately follow the data description entry for `data-name-2` (and any of its subordinate items, if `data-name-2` is a group). No intervening items at the same or higher level are permitted between the redefined item and the redefining item.

!!! note "COBOL-68 and earlier"
    Earlier standards permitted the redefining entry to appear anywhere after the redefined entry at the same level, provided no intervening items occupied a higher level. COBOL-85 tightened this rule to require immediate adjacency.

2. Multiple data items may redefine the same item. Each redefining entry must immediately follow the preceding redefining entry (or its subordinate items).

### Size Rules

1. In COBOL-68, the redefining item and the redefined item must be the same size.

!!! note "COBOL-85 and later"
    COBOL-85 relaxed the equal-size requirement. The redefining item may be smaller than the redefined item. It must not be larger. When the redefining item is smaller, it occupies only the leftmost portion of the redefined storage area.

### OCCURS Restrictions

1. The redefined item (`data-name-2`) must not have an OCCURS clause or be subordinate to an item with an OCCURS clause.
2. The redefining item (`data-name-1`) may have an OCCURS clause.
3. An item with an OCCURS clause may be redefined, but the REDEFINES must apply to the item that has the OCCURS clause, not to a subordinate of that item.

### VALUE Restrictions

1. In COBOL-85, a VALUE clause must not appear in an entry that contains a REDEFINES clause or in any subordinate entry of such an item (in WORKING-STORAGE). The redefined item may have VALUE clauses, but the redefining item may not.

!!! note "COBOL 2002 and later"
    COBOL 2002 removes this restriction. VALUE clauses are permitted on redefining items.

### Other Restrictions

1. The REDEFINES clause must not be used when the data item acts as the subject of a RENAMES (level-66) clause.
2. `data-name-2` must not itself contain a REDEFINES clause for a different item, unless it redefines an item at level 01 in the File Section.

---

## Description

The REDEFINES clause causes `data-name-1` to share the same storage area as `data-name-2`. No additional storage is allocated for `data-name-1`. The clause establishes an alternate description of the same physical bytes, permitting the program to interpret the storage in more than one way.

### Implicit Redefinition

In the File Section, multiple level-01 records described under the same FD entry implicitly redefine each other. Each record description begins at the same storage address, and the record area is large enough to contain the largest record. No explicit REDEFINES clause is required.

```cobol
       FD  TRANSACTION-FILE.
       01  DEPOSIT-RECORD.
           05  DEPOSIT-TYPE    PIC X.
           05  DEPOSIT-AMOUNT  PIC S9(7)V99.
       01  WITHDRAWAL-RECORD.
           05  WITHDRAWAL-TYPE    PIC X.
           05  WITHDRAWAL-AMOUNT  PIC S9(7)V99.
           05  WITHDRAWAL-FEE     PIC S9(3)V99.
```

`DEPOSIT-RECORD` and `WITHDRAWAL-RECORD` implicitly share the same storage area.

---

## Common Uses

### Interpreting Storage as Different Types

A REDEFINES clause permits the same storage to be treated as different data types.

```cobol
       01  WS-DATE-NUMERIC        PIC 9(8).
       01  WS-DATE-PARTS REDEFINES WS-DATE-NUMERIC.
           05  WS-DATE-YYYY       PIC 9(4).
           05  WS-DATE-MM         PIC 9(2).
           05  WS-DATE-DD         PIC 9(2).
```

After moving `20250315` to `WS-DATE-NUMERIC`, the individual components `WS-DATE-YYYY`, `WS-DATE-MM`, and `WS-DATE-DD` contain `2025`, `03`, and `15` respectively.

### Variant Records

A group item may be redefined to describe multiple record formats that share the same storage.

```cobol
       01  WS-TRANSACTION.
           05  WS-TXN-TYPE        PIC X.
               88  TXN-SALE       VALUE 'S'.
               88  TXN-RETURN     VALUE 'R'.
           05  WS-TXN-DATA        PIC X(49).
           05  WS-SALE-DATA REDEFINES WS-TXN-DATA.
               10  WS-SALE-ITEM   PIC X(20).
               10  WS-SALE-QTY    PIC 9(5).
               10  WS-SALE-PRICE  PIC S9(5)V99.
               10  FILLER         PIC X(17).
           05  WS-RETURN-DATA REDEFINES WS-TXN-DATA.
               10  WS-RETURN-ITEM PIC X(20).
               10  WS-RETURN-QTY  PIC 9(5).
               10  WS-RETURN-REASON
                                   PIC X(24).
```

The program examines `WS-TXN-TYPE` to determine which redefinition to use when accessing the data.

### Table Initialization (COBOL-85 Idiom)

In COBOL-85, where VALUE is not permitted on items with OCCURS, REDEFINES provides a standard idiom for initializing tables with constant data.

```cobol
       01  WS-MONTH-NAME-DATA.
           05  FILLER  PIC X(9) VALUE "January  ".
           05  FILLER  PIC X(9) VALUE "February ".
           05  FILLER  PIC X(9) VALUE "March    ".
           05  FILLER  PIC X(9) VALUE "April    ".
           05  FILLER  PIC X(9) VALUE "May      ".
           05  FILLER  PIC X(9) VALUE "June     ".
           05  FILLER  PIC X(9) VALUE "July     ".
           05  FILLER  PIC X(9) VALUE "August   ".
           05  FILLER  PIC X(9) VALUE "September".
           05  FILLER  PIC X(9) VALUE "October  ".
           05  FILLER  PIC X(9) VALUE "November ".
           05  FILLER  PIC X(9) VALUE "December ".
       01  WS-MONTH-NAME-TABLE REDEFINES WS-MONTH-NAME-DATA.
           05  WS-MONTH-NAME  PIC X(9) OCCURS 12 TIMES.
```

The VALUE clauses on the FILLER items initialize the storage. The REDEFINES imposes the table structure over that storage, permitting `WS-MONTH-NAME(N)` to retrieve the name of month `N`.

---

## Examples

### Date Reformatting

```cobol
       WORKING-STORAGE SECTION.
       01  WS-INPUT-DATE          PIC X(10).
       01  WS-INPUT-PARTS REDEFINES WS-INPUT-DATE.
           05  WS-IN-YYYY         PIC X(4).
           05  WS-IN-DASH1        PIC X.
           05  WS-IN-MM           PIC X(2).
           05  WS-IN-DASH2        PIC X.
           05  WS-IN-DD           PIC X(2).

       01  WS-OUTPUT-DATE         PIC X(8).
       01  WS-OUTPUT-PARTS REDEFINES WS-OUTPUT-DATE.
           05  WS-OUT-MM          PIC X(2).
           05  WS-OUT-DD          PIC X(2).
           05  WS-OUT-YYYY        PIC X(4).

       PROCEDURE DIVISION.
           MOVE "2025-03-15" TO WS-INPUT-DATE.
           MOVE WS-IN-MM   TO WS-OUT-MM.
           MOVE WS-IN-DD   TO WS-OUT-DD.
           MOVE WS-IN-YYYY TO WS-OUT-YYYY.
      *>   WS-OUTPUT-DATE now contains "03152025"
           DISPLAY "Reformatted: " WS-OUTPUT-DATE.
           STOP RUN.
```

### Redefining a Numeric Item as Alphanumeric

```cobol
       01  WS-AMOUNT             PIC S9(7)V99 COMP-3.
       01  WS-AMOUNT-BYTES REDEFINES WS-AMOUNT
                                  PIC X(5).
```

This permits byte-level operations on the COMP-3 field, such as inspecting individual bytes for diagnostic purposes. The size of the redefining item (5 bytes) matches the COMP-3 storage size for `S9(7)V99`.

### Multiple Redefinitions

```cobol
       01  WS-CODE               PIC X(4).
       01  WS-CODE-NUM REDEFINES WS-CODE
                                  PIC 9(4).
       01  WS-CODE-PARTS REDEFINES WS-CODE.
           05  WS-CODE-PREFIX    PIC X(2).
           05  WS-CODE-SUFFIX    PIC X(2).
```

Both `WS-CODE-NUM` and `WS-CODE-PARTS` redefine `WS-CODE`. Each occupies the same four bytes. Note that `WS-CODE-PARTS` redefines `WS-CODE`, not `WS-CODE-NUM`.

---

## See Also

- [Data Division](index.md)
- [Level Numbers](level-numbers.md)
- [PICTURE Clause](picture.md)
- [USAGE Clause](usage.md)
- [OCCURS Clause](occurs.md)
- [VALUE Clause](value.md)
- [Condition Names (Level 88)](condition-names.md)
