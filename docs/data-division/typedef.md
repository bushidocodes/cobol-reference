# User-Defined Types (TYPEDEF)

The `TYPEDEF` clause allows programmers to define named data types that can be reused throughout a program or across programs. A type definition establishes a template for data items without allocating storage; actual storage is allocated when the type is referenced with the `TYPE` clause.

- **Standard:** COBOL 2002, COBOL 2014

!!! note "Compiler Support"
    TYPEDEF support varies across compilers. GnuCOBOL 3.x, Micro Focus Visual
    COBOL, and IBM Enterprise COBOL 6+ provide varying levels of support.
    Check your compiler documentation for specific conformance details.

---

## Defining a Type

A type is defined using the `TYPEDEF` clause on a data description entry. The definition describes the structure, PICTURE, USAGE, and other attributes of the type but does not allocate storage.

### Syntax

```cobol
level-number type-name IS TYPEDEF.
    [subordinate entries]
```

- **level-number** -- the level number of the type definition. Typically `01` for group types or `01`/`77` for elementary types.
- **type-name** -- the name of the type being defined.
- **IS TYPEDEF** -- marks the entry as a type definition rather than a data item.

### Rules

- A TYPEDEF entry does not allocate storage. It serves only as a template.
- A TYPEDEF may be an elementary item or a group item with subordinate entries.
- TYPEDEF entries may appear in the `WORKING-STORAGE SECTION`, `LOCAL-STORAGE SECTION`, or `LINKAGE SECTION`.
- A TYPEDEF may reference another user-defined type via the `TYPE` clause.
- The `STRONG` keyword may be added to create a strong type that prevents implicit assignment from items of a different type.

---

## Using a Type

Once a type is defined, data items of that type are declared with the `TYPE` clause:

```cobol
level-number data-name TYPE type-name.
```

The data item inherits all attributes (PICTURE, USAGE, subordinate structure, etc.) from the type definition.

---

## Elementary Type Examples

### Simple Typed Fields

```cobol
       WORKING-STORAGE SECTION.
      *> Type definitions
       01  MONEY-TYPE     IS TYPEDEF  PIC S9(9)V99
                          USAGE PACKED-DECIMAL.
       01  RATE-TYPE      IS TYPEDEF  PIC 9V9(6).
       01  ACCOUNT-ID     IS TYPEDEF  PIC X(10).

      *> Data items using the types
       01  WS-BALANCE     TYPE MONEY-TYPE.
       01  WS-CREDIT      TYPE MONEY-TYPE.
       01  WS-DEBIT       TYPE MONEY-TYPE.
       01  WS-INT-RATE    TYPE RATE-TYPE.
       01  WS-ACCT-NO     TYPE ACCOUNT-ID.
```

Each item declared with `TYPE MONEY-TYPE` inherits `PIC S9(9)V99 USAGE PACKED-DECIMAL` without repeating the full definition.

### Using Types in Computations

```cobol
       WORKING-STORAGE SECTION.
       01  QUANTITY-TYPE   IS TYPEDEF  PIC 9(5).
       01  PRICE-TYPE      IS TYPEDEF  PIC 9(5)V99.
       01  TOTAL-TYPE      IS TYPEDEF  PIC 9(9)V99.

       01  WS-QTY          TYPE QUANTITY-TYPE.
       01  WS-UNIT-PRICE   TYPE PRICE-TYPE.
       01  WS-LINE-TOTAL   TYPE TOTAL-TYPE.

       PROCEDURE DIVISION.
           MOVE 150 TO WS-QTY
           MOVE 29.95 TO WS-UNIT-PRICE
           COMPUTE WS-LINE-TOTAL =
               WS-QTY * WS-UNIT-PRICE
           DISPLAY "Line total: " WS-LINE-TOTAL
           STOP RUN.
```

---

## Group Type Examples

### Record Structure Type

```cobol
       WORKING-STORAGE SECTION.
      *> Define a type for an address structure
       01  ADDRESS-TYPE IS TYPEDEF.
           05  ADDR-STREET    PIC X(30).
           05  ADDR-CITY      PIC X(20).
           05  ADDR-STATE     PIC X(2).
           05  ADDR-ZIP       PIC X(10).

      *> Define a type for a person record
       01  PERSON-TYPE IS TYPEDEF.
           05  PERSON-NAME.
               10  PERSON-FIRST  PIC X(20).
               10  PERSON-LAST   PIC X(25).
           05  PERSON-DOB        PIC 9(8).
           05  PERSON-ADDR       TYPE ADDRESS-TYPE.

      *> Allocate actual storage using the types
       01  WS-CUSTOMER        TYPE PERSON-TYPE.
       01  WS-EMPLOYEE        TYPE PERSON-TYPE.
       01  WS-SHIP-ADDR       TYPE ADDRESS-TYPE.
       01  WS-BILL-ADDR       TYPE ADDRESS-TYPE.
```

Both `WS-CUSTOMER` and `WS-EMPLOYEE` have identical structure including the nested `ADDRESS-TYPE`, without duplicating the field definitions.

### Accessing Fields in Typed Items

Fields within a typed group item are accessed normally:

```cobol
       PROCEDURE DIVISION.
           MOVE "Jane"       TO PERSON-FIRST OF WS-CUSTOMER
           MOVE "Doe"        TO PERSON-LAST OF WS-CUSTOMER
           MOVE 19900315     TO PERSON-DOB OF WS-CUSTOMER
           MOVE "123 Main"   TO ADDR-STREET OF
                                PERSON-ADDR OF WS-CUSTOMER
           MOVE "Springfield" TO ADDR-CITY OF
                                PERSON-ADDR OF WS-CUSTOMER
           MOVE "IL"         TO ADDR-STATE OF
                                PERSON-ADDR OF WS-CUSTOMER
           MOVE "62704"      TO ADDR-ZIP OF
                                PERSON-ADDR OF WS-CUSTOMER
           STOP RUN.
```

---

## Tables with Typed Elements

TYPEDEF can be combined with `OCCURS` to create tables of typed items:

```cobol
       WORKING-STORAGE SECTION.
       01  DATE-TYPE IS TYPEDEF  PIC 9(8).

       01  TRANSACTION-TYPE IS TYPEDEF.
           05  TXN-DATE       TYPE DATE-TYPE.
           05  TXN-AMOUNT     PIC S9(9)V99.
           05  TXN-DESC       PIC X(30).

       01  WS-TXN-TABLE.
           05  WS-TXN         TYPE TRANSACTION-TYPE
                               OCCURS 100 TIMES
                               INDEXED BY TXN-IDX.
       01  WS-TXN-COUNT       PIC 9(3) VALUE 0.

       PROCEDURE DIVISION.
           ADD 1 TO WS-TXN-COUNT
           MOVE 20250322 TO TXN-DATE OF
                            WS-TXN(WS-TXN-COUNT)
           MOVE 150.00   TO TXN-AMOUNT OF
                            WS-TXN(WS-TXN-COUNT)
           MOVE "Office supplies" TO TXN-DESC OF
                            WS-TXN(WS-TXN-COUNT)
           STOP RUN.
```

---

## Strong Typing

The `STRONG` keyword creates a type that enforces type safety. Items of a strong type cannot be implicitly moved to or compared with items of a different type, even if they have the same underlying PICTURE and USAGE.

```cobol
       WORKING-STORAGE SECTION.
       01  CELSIUS-TYPE    IS TYPEDEF STRONG PIC S9(3)V9.
       01  FAHRENHEIT-TYPE IS TYPEDEF STRONG PIC S9(3)V9.

       01  WS-TEMP-C       TYPE CELSIUS-TYPE.
       01  WS-TEMP-F       TYPE FAHRENHEIT-TYPE.

       PROCEDURE DIVISION.
           MOVE 100.0 TO WS-TEMP-C
      *>   The following would cause a compilation error
      *>   because CELSIUS-TYPE and FAHRENHEIT-TYPE are
      *>   different strong types:
      *>   MOVE WS-TEMP-C TO WS-TEMP-F
           STOP RUN.
```

Strong types prevent accidental mixing of semantically different values that happen to share the same physical representation.

!!! warning "Compiler Support"
    Strong typing support is less widely implemented than basic TYPEDEF.
    Verify your compiler supports the `STRONG` keyword before relying on it.

---

## Types Across Programs

Type definitions can be shared across programs by placing them in a copybook:

**`TYPES.cpy`:**

```cobol
       01  MONEY-TYPE   IS TYPEDEF PIC S9(9)V99
                        USAGE PACKED-DECIMAL.
       01  DATE-TYPE    IS TYPEDEF PIC 9(8).
       01  STATUS-TYPE  IS TYPEDEF PIC X(2).
```

**Calling program:**

```cobol
       DATA DIVISION.
       WORKING-STORAGE SECTION.
       COPY TYPES.

       01  WS-BALANCE   TYPE MONEY-TYPE.
       01  WS-TXN-DATE  TYPE DATE-TYPE.
       01  WS-RESULT    TYPE STATUS-TYPE.
```

This ensures consistent data definitions across all programs that use the shared types.

---

## TYPEDEF vs REDEFINES vs COPY

| Feature | TYPEDEF | REDEFINES | COPY |
|---------|---------|-----------|------|
| Purpose | Define reusable data types | Overlay storage with alternate layout | Include source text |
| Storage | No storage allocated | Shares existing storage | Allocates new storage per inclusion |
| Type safety | Optional (with STRONG) | None | None |
| Scope | Program or copybook | Single data item | Wherever included |
| Standard | COBOL 2002 | COBOL-68 | COBOL-68 |

---

## See Also

- [Level Numbers](level-numbers.md) -- data item level numbers
- [PICTURE Clause](picture.md) -- data format specification
- [USAGE Clause](usage.md) -- internal storage format
- [OCCURS Clause](occurs.md) -- table definitions
- [REDEFINES Clause](redefines.md) -- storage overlay
- [COPY](../copy-replace/copy.md) -- source text inclusion
