# VALUE Clause

The VALUE clause specifies the initial value of a data item in WORKING-STORAGE or LOCAL-STORAGE, or specifies the values associated with a condition name (level 88).

**Standard:** COBOL-60, COBOL-61, COBOL-61 Extended, COBOL-65, COBOL-68, COBOL-85, COBOL 2002, COBOL 2014, COBOL 2023

---

## Syntax

### Elementary Items

```cobol
level-number  data-name  PIC picture-string
    VALUE IS literal.
```

### Group Items

```cobol
level-number  data-name
    VALUE IS literal.
    05  ...
```

### Condition Names (Level 88)

```cobol
88  condition-name  VALUE IS literal-1
    [ { THRU | THROUGH } literal-2 ].

88  condition-name  VALUES ARE literal-1 [ literal-2 ... ]
    [ literal-3 { THRU | THROUGH } literal-4 ] ...
```

---

## Description

The VALUE clause serves two distinct purposes depending on context:

1. **Data initialization.** For elementary and group items in WORKING-STORAGE SECTION and LOCAL-STORAGE SECTION, the VALUE clause specifies the initial contents of the data item when the program begins execution (or, for LOCAL-STORAGE, on each invocation).

2. **Condition-name definition.** For level-88 entries, the VALUE clause specifies the value or values that make the condition name true. See [Condition Names (Level 88)](condition-names.md) for full details.

---

## Rules

### Section Restrictions

1. In COBOL-85, the VALUE clause for data initialization is permitted in WORKING-STORAGE SECTION only. It is not permitted in the FILE SECTION or LINKAGE SECTION.

!!! note "COBOL 2002 and later"
    COBOL 2002 extends VALUE clause support to LOCAL-STORAGE SECTION. The restriction against VALUE in the FILE SECTION and LINKAGE SECTION remains in effect for data initialization, though level-88 entries (condition names) are permitted in all sections.

2. Level-88 VALUE clauses (condition-name definitions) are permitted in any section of the Data Division, including the FILE SECTION and LINKAGE SECTION.

### Type Compatibility

1. The literal specified in a VALUE clause must be compatible with the PICTURE and USAGE of the data item.
2. For a numeric data item, the literal must be a numeric literal. The literal value is aligned on the decimal point and truncated or zero-filled as necessary to fit the PICTURE.
3. For an alphanumeric data item, the literal must be an alphanumeric literal or a figurative constant. The value is left-justified and truncated or space-filled on the right.
4. For an alphabetic data item, the literal must contain only alphabetic characters and spaces.

### VALUE with REDEFINES

1. In COBOL-85, a VALUE clause must not appear in any entry that contains a REDEFINES clause, nor in any entry subordinate to such an item. Only the redefined (original) item may carry VALUE clauses.

!!! note "COBOL 2002 and later"
    COBOL 2002 removes this restriction. VALUE clauses are permitted on items with REDEFINES.

### VALUE with OCCURS

1. In COBOL-85, a VALUE clause must not appear on an item that has an OCCURS clause, nor on an item subordinate to one with an OCCURS clause.

!!! note "COBOL 2002 and later"
    COBOL 2002 permits VALUE clauses on items with OCCURS. When specified, the value applies to every occurrence of the item.

### Group-Level VALUE

1. When a VALUE clause appears on a group item, the literal is treated as an alphanumeric value applied to the entire group, regardless of the descriptions of subordinate items.
2. A group-level VALUE clause initializes the entire group storage with the specified literal, left-justified and truncated or space-filled.
3. VALUE clauses on subordinate items of a group item that itself has a VALUE clause are permitted but the group-level value takes precedence during initialization.

---

## Figurative Constants as Values

Figurative constants provide commonly used values without explicit literals.

| Figurative Constant | Meaning |
|---------------------|---------|
| `ZERO` / `ZEROS` / `ZEROES` | Numeric zero or the character `'0'`, depending on category |
| `SPACE` / `SPACES` | Space character(s) |
| `HIGH-VALUE` / `HIGH-VALUES` | Highest value in the collating sequence (typically `X'FF'`) |
| `LOW-VALUE` / `LOW-VALUES` | Lowest value in the collating sequence (typically `X'00'`) |
| `QUOTE` / `QUOTES` | Quotation mark character |
| `ALL literal` | The literal repeated to fill the item |

```cobol
       01  WS-BUFFER        PIC X(80)  VALUE SPACES.
       01  WS-COUNTER       PIC 9(5)   VALUE ZEROS.
       01  WS-HIGH-KEY      PIC X(10)  VALUE HIGH-VALUES.
       01  WS-SEPARATOR     PIC X(50)  VALUE ALL "-".
```

---

## INITIALIZE Statement vs. VALUE Clause

The VALUE clause provides a compile-time initial value. The `INITIALIZE` statement provides a runtime mechanism for resetting data items.

| Aspect | VALUE Clause | INITIALIZE Statement |
|--------|-------------|---------------------|
| When applied | Program load (WORKING-STORAGE) or invocation (LOCAL-STORAGE) | Runtime, when executed |
| Scope | Single item as declared | Item and all subordinate items by category |
| Default behavior | Exact literal specified | Spaces for alphanumeric, zeros for numeric |
| Reapplication | Not automatically reapplied | May be executed any number of times |

```cobol
       01  WS-RECORD.
           05  WS-NAME      PIC X(30) VALUE SPACES.
           05  WS-AMOUNT    PIC S9(7)V99 VALUE ZERO.
           05  WS-CODE      PIC X(3)  VALUE "NEW".

      *>   At program start, WS-RECORD is initialized per VALUE clauses.
      *>   To reset at runtime:
           INITIALIZE WS-RECORD.
      *>   WS-NAME is set to SPACES, WS-AMOUNT to ZERO,
      *>   WS-CODE to SPACES (not "NEW").
```

---

## Examples

### Elementary Item Values

```cobol
       WORKING-STORAGE SECTION.
       01  WS-COMPANY-NAME   PIC X(30) VALUE "ACME CORPORATION".
       01  WS-TAX-RATE       PIC V9(4) VALUE .0825.
       01  WS-MAX-RECORDS    PIC 9(5)  VALUE 10000.
       01  WS-SIGN-FIELD     PIC S9(3) VALUE -50.
       01  WS-BLANK-LINE     PIC X(80) VALUE SPACES.
```

### Group Item Value

```cobol
       01  WS-HEADER         VALUE "REPORT DATED 00/00/0000".
           05  WS-HDR-TEXT   PIC X(14).
           05  WS-HDR-MM     PIC XX.
           05  FILLER        PIC X VALUE "/".
           05  WS-HDR-DD     PIC XX.
           05  FILLER        PIC X VALUE "/".
           05  WS-HDR-YYYY   PIC X(4).
```

The group-level VALUE initializes the entire 23-byte area. The subordinate VALUE clauses on the FILLER items are superseded by the group-level value during initial allocation.

### Table Initialization (COBOL-85 Idiom)

```cobol
       01  WS-STATUS-DATA.
           05  FILLER  PIC X(10) VALUE "Active    ".
           05  FILLER  PIC X(10) VALUE "Inactive  ".
           05  FILLER  PIC X(10) VALUE "Suspended ".
           05  FILLER  PIC X(10) VALUE "Closed    ".
       01  WS-STATUS-TABLE REDEFINES WS-STATUS-DATA.
           05  WS-STATUS-NAME  PIC X(10) OCCURS 4 TIMES.
```

The VALUE clauses on the FILLER items fill the storage area. The [REDEFINES clause](redefines.md) imposes a table structure, permitting `WS-STATUS-NAME(N)` to retrieve the name for status `N`.

### Table Initialization (COBOL 2002)

```cobol
       01  WS-DAY-NAMES.
           05  WS-DAY-NAME  PIC X(9) OCCURS 7 TIMES
               VALUE "Monday" "Tuesday" "Wednesday"
                     "Thursday" "Friday" "Saturday"
                     "Sunday".
```

### Condition Names with VALUE

```cobol
       01  WS-ACCOUNT-STATUS    PIC X.
           88  ACCT-OPEN        VALUE "O".
           88  ACCT-CLOSED      VALUE "C".
           88  ACCT-SUSPENDED   VALUE "S".
           88  ACCT-VALID       VALUE "O" "C" "S".

       01  WS-TEMPERATURE       PIC S9(3).
           88  TEMP-FREEZING    VALUE -50 THRU 32.
           88  TEMP-NORMAL      VALUE 33 THRU 100.
           88  TEMP-EXTREME     VALUE 101 THRU 150.
```

See [Condition Names (Level 88)](condition-names.md) for comprehensive coverage of level-88 VALUE clauses.

### COMP and COMP-3 Items

```cobol
       01  WS-BINARY-CTR       PIC S9(8) COMP VALUE ZERO.
       01  WS-PACKED-AMT       PIC S9(7)V99 COMP-3 VALUE 1234.56.
       01  WS-INDEX-VAL        PIC S9(4) COMP VALUE 1.
```

The VALUE literal is specified in display form; the compiler converts it to the internal representation indicated by the USAGE clause.

---

## See Also

- [Data Division](index.md)
- [Level Numbers](level-numbers.md)
- [PICTURE Clause](picture.md)
- [USAGE Clause](usage.md)
- [OCCURS Clause](occurs.md)
- [REDEFINES Clause](redefines.md)
- [Condition Names (Level 88)](condition-names.md)
