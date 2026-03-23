# Level Numbers

Level numbers define the hierarchy of data items within a record and indicate the relationship between group items and their subordinate elementary items.

**Standard:** COBOL-60, COBOL-61, COBOL-61 Extended, COBOL-65, COBOL-68, COBOL-85, COBOL 2002, COBOL 2014, COBOL 2023

---

## Syntax

```cobol
level-number data-name-or-FILLER [clauses].
```

```
level-number := 01-49 | 66 | 77 | 88
```

The level number is a one- or two-digit integer written in Area A (columns 8-11) for level 01 and 77, or in Area B (columns 12-72) for all other levels.

## Level Number Categories

### Levels 01 through 49 -- Group and Elementary Items

Levels 01 through 49 define the hierarchical structure of a record. Level 01 denotes the record itself; levels 02 through 49 denote subordinate items within that record.

- A **group item** is a data item that has one or more subordinate items defined at a higher level number beneath it. A group item has no `PICTURE` clause of its own; its size is the sum of the sizes of its subordinates.
- An **elementary item** is a data item that has no subordinates. It requires a `PICTURE` clause (unless its `USAGE` implies a fixed format, such as `INDEX`, `POINTER`, `COMP-1`, or `COMP-2`).

#### Hierarchy Rules

1. A level-01 entry defines the beginning of a new record. It must begin in Area A.
2. The level number of a subordinate item must be greater than the level number of its parent group.
3. Items with the same level number under the same parent are siblings.
4. The actual level number values need not be contiguous. For example, an 01 item may contain 05-level items, which may contain 10-level items. Skipping numbers (01, 05, 10, 15, ...) is conventional and permits later insertion of intermediate levels.
5. A group extends from its level number to just before the next item with a level number less than or equal to its own, or to the end of the record.

#### Common Conventions

| Level | Conventional Use |
|-------|-----------------|
| 01 | Record or top-level group |
| 05 | First level of subordination |
| 10 | Second level of subordination |
| 15 | Third level of subordination |
| 49 | Deepest standard subordination |

Any value from 01 to 49 is valid; the convention of using multiples of 5 is a widely followed practice, not a language requirement.

### Level 66 -- RENAMES Clause

Level 66 provides an alternative grouping of elementary items by assigning a new name to a contiguous range of items within a record.

```cobol
66  data-name  RENAMES  data-name-1  [THRU  data-name-2].
```

**Rules:**

1. Level-66 entries must immediately follow the last data description entry in the record they rename.
2. `data-name-1` and `data-name-2` must refer to elementary items or group items within the same level-01 record.
3. `data-name-1` must not be a subordinate of `data-name-2`.
4. `data-name-2`, if specified, must follow `data-name-1` in the record's physical layout.
5. Neither `data-name-1` nor `data-name-2` may be a level-66, level-77, or level-88 item.
6. Neither `data-name-1` nor `data-name-2` may have an `OCCURS` clause or be subordinate to an item with an `OCCURS` clause.
7. The renamed area spans from the first byte of `data-name-1` through the last byte of `data-name-2` (or through the last byte of `data-name-1` if `THRU` is omitted).

### Level 77 -- Independent Elementary Items

Level 77 defines an elementary item that is not part of any record (group) structure.

```cobol
77  data-name  [PICTURE clause]  [USAGE clause]  [VALUE clause].
```

**Rules:**

1. Level-77 entries must appear in either the Working-Storage Section, Local-Storage Section, or Linkage Section.
2. A level-77 item may not be subdivided; it is always elementary.
3. A level-77 entry must begin in Area A.
4. Level-77 items have the same capabilities as level-01 elementary items. The COBOL 2002 standard notes that level-77 is functionally equivalent to a level-01 elementary item and is retained for backward compatibility.

### Level 88 -- Condition Names

Level 88 defines condition names (named conditions) that are associated with specific values of a conditional variable.

```cobol
88  condition-name  VALUE IS  literal-1  [THRU literal-2]
                    [literal-3  [THRU literal-4]] ...
```

**Rules:**

1. A level-88 entry must immediately follow the data description entry of its conditional variable.
2. The conditional variable may be any elementary item (levels 01-49, or 77) with a `PICTURE` clause.
3. Multiple values and ranges may be specified on a single level-88 entry.
4. When used in a conditional expression, the condition name evaluates to true if the current value of the conditional variable matches any of the specified values or falls within any of the specified ranges.
5. The `SET` statement can set the conditional variable to the first value specified in the level-88 entry.
6. A single conditional variable may have multiple level-88 entries.

See: [Condition Names (Level 88)](condition-names.md)

## Behavior

### Storage Allocation

- A level-01 item in the File Section defines a record whose size corresponds to the record size of the file.
- A level-01 item in Working-Storage or Local-Storage begins on a storage boundary as determined by the implementation; items within it are allocated contiguously.
- The size of a group item is the sum of the sizes of all its subordinate items, including any implicit padding required by `SYNCHRONIZED` clauses.
- Level-66 items do not allocate new storage; they provide an alternative name for an existing contiguous area.
- Level-88 items do not allocate any storage.

### Implicit Definitions

- If a level-01 or level-77 item in the File Section or Linkage Section has no `PICTURE` or `USAGE` clause and no subordinate items, the item is implicitly an alphanumeric group.
- `FILLER` may be used in place of a data name for any level from 01 to 49. In COBOL 2002 and later, the data name may be omitted entirely (implied `FILLER`).

## Examples

### Basic Record Structure

```cobol
       01  EMPLOYEE-RECORD.
           05  EMP-ID                PIC 9(6).
           05  EMP-NAME.
               10  EMP-FIRST-NAME   PIC X(15).
               10  EMP-LAST-NAME    PIC X(20).
           05  EMP-SALARY           PIC 9(7)V99.
           05  EMP-DEPARTMENT       PIC X(4).
```

In this structure:
- `EMPLOYEE-RECORD` (level 01) is a group item containing all subordinate items.
- `EMP-NAME` (level 05) is a group item containing `EMP-FIRST-NAME` and `EMP-LAST-NAME`.
- `EMP-ID`, `EMP-FIRST-NAME`, `EMP-LAST-NAME`, `EMP-SALARY`, and `EMP-DEPARTMENT` are elementary items.
- The total record size is 6 + 15 + 20 + 9 + 4 = 54 bytes (in `USAGE DISPLAY`).

### Level 66 RENAMES

```cobol
       01  DATE-RECORD.
           05  DATE-YEAR            PIC 9(4).
           05  DATE-MONTH           PIC 9(2).
           05  DATE-DAY             PIC 9(2).
       66  DATE-YEAR-MONTH  RENAMES DATE-YEAR THRU DATE-MONTH.
       66  DATE-FULL        RENAMES DATE-YEAR THRU DATE-DAY.
```

`DATE-YEAR-MONTH` refers to the first 6 bytes (year + month). `DATE-FULL` refers to all 8 bytes.

### Level 77 Independent Items

```cobol
       77  WS-PAGE-COUNT       PIC 9(5)   VALUE ZERO.
       77  WS-PROGRAM-NAME     PIC X(10)  VALUE 'ACCTUPD'.
       77  WS-TAX-RATE         PIC V9(4)  VALUE .0725.
```

### Level 88 Condition Names

```cobol
       01  WS-ACCOUNT-TYPE         PIC X(1).
           88  ACCT-CHECKING       VALUE 'C'.
           88  ACCT-SAVINGS        VALUE 'S'.
           88  ACCT-MONEY-MARKET   VALUE 'M'.
           88  ACCT-VALID          VALUE 'C' 'S' 'M'.
```

In the Procedure Division:

```cobol
       IF ACCT-CHECKING
           PERFORM PROCESS-CHECKING
       END-IF

       SET ACCT-SAVINGS TO TRUE
      *> WS-ACCOUNT-TYPE now contains 'S'
```

### Non-contiguous Level Numbers

```cobol
       01  TRANSACTION-REC.
           03  TXN-HEADER.
               06  TXN-TYPE        PIC X(2).
               06  TXN-DATE        PIC 9(8).
           03  TXN-BODY.
               06  TXN-AMOUNT      PIC S9(9)V99.
               06  TXN-ACCOUNT     PIC X(10).
```

This structure is equivalent to one using levels 01, 05, 10 -- the specific numbers chosen do not affect behavior, only the relative ordering matters.

## See Also

- [Data Division](index.md)
- [PICTURE Clause](picture.md)
- [USAGE Clause](usage.md)
- [OCCURS Clause](occurs.md)
- [Condition Names (Level 88)](condition-names.md)
- RENAMES Clause
- [VALUE Clause](value.md)
