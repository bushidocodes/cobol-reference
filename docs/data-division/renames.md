# RENAMES (Level 66)

The `RENAMES` clause provides an alternative grouping of elementary items by allowing a new data name to refer to a contiguous range of items that may span across existing group boundaries.

- **Standard:** COBOL-61, COBOL-68, COBOL-85, COBOL 2002, COBOL 2014, COBOL 2023
- **Division:** Data Division
- **Level Number:** 66

---

## Syntax

```cobol
66  data-name-1 RENAMES data-name-2 [ THRU data-name-3 ].
```

---

## Rules

1. Level-66 entries must immediately follow the last data description entry (the last non-66 entry) of the record they reference.
2. `data-name-2` is the first (or only) elementary item being renamed.
3. `data-name-3`, if specified, is the last elementary item in the range. `data-name-1` then refers to the contiguous storage from `data-name-2` through `data-name-3`.
4. `data-name-2` and `data-name-3` must be elementary items or group items within the same level-01 record.
5. `data-name-2` must not be the same as `data-name-3`.
6. Neither `data-name-2` nor `data-name-3` may have an OCCURS clause or be subordinate to an item with an OCCURS clause.
7. `data-name-3` must not precede `data-name-2` in the record.
8. If `data-name-2` is a group item, `data-name-3` must not be subordinate to it.
9. The RENAMES item (`data-name-1`) cannot be referenced by another RENAMES.
10. No VALUE, OCCURS, PICTURE, USAGE, or other data description clauses are permitted on a level-66 entry (the characteristics are derived from the renamed items).
11. Multiple level-66 entries may follow a single record.

---

## Examples

### Basic RENAMES

```cobol
       01  EMPLOYEE-RECORD.
           05  EMP-NAME.
               10  EMP-FIRST    PIC X(15).
               10  EMP-MIDDLE   PIC X(1).
               10  EMP-LAST     PIC X(20).
           05  EMP-ADDRESS.
               10  EMP-STREET   PIC X(30).
               10  EMP-CITY     PIC X(20).
               10  EMP-STATE    PIC X(2).
               10  EMP-ZIP      PIC X(10).
           05  EMP-SALARY       PIC 9(7)V99.
           66  EMP-FULL-ADDRESS RENAMES EMP-STREET
                                THRU EMP-ZIP.
           66  EMP-LOCATION     RENAMES EMP-CITY
                                THRU EMP-STATE.
```

`EMP-FULL-ADDRESS` refers to the contiguous 62 bytes covering STREET + CITY + STATE + ZIP. `EMP-LOCATION` covers CITY + STATE (22 bytes).

### Single Item RENAMES

```cobol
       01  TRANSACTION-REC.
           05  TXN-DATE.
               10  TXN-YEAR    PIC 9(4).
               10  TXN-MONTH   PIC 9(2).
               10  TXN-DAY     PIC 9(2).
           05  TXN-AMOUNT      PIC S9(9)V99.
           05  TXN-TYPE        PIC X(1).
           66  TXN-YEAR-MONTH  RENAMES TXN-YEAR
                               THRU TXN-MONTH.
           66  TXN-HEADER      RENAMES TXN-DATE.
```

### Cross-Group RENAMES

The key feature of RENAMES is that it can span across group boundaries:

```cobol
       01  RECORD-A.
           05  GROUP-1.
               10  FIELD-A     PIC X(10).
               10  FIELD-B     PIC X(10).
           05  GROUP-2.
               10  FIELD-C     PIC X(10).
               10  FIELD-D     PIC X(10).
           66  MIDDLE-FIELDS   RENAMES FIELD-B
                               THRU FIELD-C.
```

`MIDDLE-FIELDS` spans the boundary between GROUP-1 and GROUP-2, covering FIELD-B and FIELD-C (20 bytes). This cannot be achieved with standard group items or REDEFINES.

---

## RENAMES vs REDEFINES

| Aspect | RENAMES (66) | REDEFINES |
|--------|-------------|-----------|
| Purpose | Alternative grouping of contiguous items | Alternate interpretation of same storage |
| Can span groups | Yes | No (applies to one item) |
| Creates new storage | No | No |
| Level restriction | Must be level 66 | Any level (01-49) |
| Position | Must follow all non-66 entries in record | Must immediately follow redefined item |
| With OCCURS | Not permitted | Permitted |

---

## When to Use RENAMES

- **Cross-group access** — when you need to reference a range of fields that spans existing group boundaries
- **Alternate record views** — when different parts of a program need different logical groupings of the same physical data
- **Legacy data formats** — when a record layout was defined one way but processing requires a different grouping

!!! tip "Rarely Used in Modern Code"
    RENAMES is infrequently used in new COBOL programs. REDEFINES and
    reference modification typically provide more flexible alternatives.
    However, RENAMES remains valuable for its unique ability to create
    views that span group boundaries.

---

## See Also

- [Level Numbers](level-numbers.md) -- data item level numbers
- [REDEFINES Clause](redefines.md) -- storage overlay
- [Reference Modification](../language/reference-modification.md) -- substring access
