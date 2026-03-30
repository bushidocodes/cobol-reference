# CORRESPONDING Phrase

The `CORRESPONDING` (abbreviated `CORR`) phrase enables operations on matching elementary items between two group items, based on name. It can be used with MOVE, ADD, and SUBTRACT.

- **Standard:** COBOL-60 (Elective), COBOL-68, COBOL-85, COBOL 2002, COBOL 2014, COBOL 2023
- **Division:** Procedure Division
- **Category:** Data Movement / Arithmetic

---

## Syntax

```cobol
MOVE CORRESPONDING group-1 TO group-2

ADD CORRESPONDING group-1 TO group-2
    [ ROUNDED ] [ ON SIZE ERROR imperative-statement ]
    [ NOT ON SIZE ERROR imperative-statement ]
END-ADD

SUBTRACT CORRESPONDING group-1 FROM group-2
    [ ROUNDED ] [ ON SIZE ERROR imperative-statement ]
    [ NOT ON SIZE ERROR imperative-statement ]
END-SUBTRACT
```

---

## Matching Rules

Two elementary items **correspond** if all of the following are true:

1. They have the **same name** (data-name, not FILLER).
2. At least one of them is not a FILLER item.
3. The items at each level of qualification up to (but not including) the group items named in the CORRESPONDING phrase have the **same name**.
4. Neither item is defined with an OCCURS clause or is subordinate to an item with an OCCURS clause (for MOVE CORRESPONDING — ADD/SUBTRACT CORRESPONDING additionally require both items to be numeric).
5. For MOVE CORRESPONDING: at least one of the items is an elementary item.
6. For ADD/SUBTRACT CORRESPONDING: both items must be numeric elementary items.

Items that don't match are **ignored** — no error is raised.

---

## Examples

### MOVE CORRESPONDING

```cobol
       01  INPUT-RECORD.
           05  EMPLOYEE-NAME    PIC X(30).
           05  DEPARTMENT       PIC X(10).
           05  SALARY           PIC 9(7)V99.
           05  HIRE-DATE        PIC 9(8).

       01  OUTPUT-RECORD.
           05  EMPLOYEE-NAME    PIC X(30).
           05  SALARY           PIC 9(7)V99.
           05  FILLER           PIC X(40).

       MOVE CORRESPONDING INPUT-RECORD TO OUTPUT-RECORD
       *> Moves EMPLOYEE-NAME and SALARY only
       *> DEPARTMENT and HIRE-DATE have no match, so are skipped
       *> FILLER items are never matched
```

This is equivalent to:

```cobol
       MOVE EMPLOYEE-NAME OF INPUT-RECORD
           TO EMPLOYEE-NAME OF OUTPUT-RECORD
       MOVE SALARY OF INPUT-RECORD
           TO SALARY OF OUTPUT-RECORD
```

### ADD CORRESPONDING

```cobol
       01  MONTHLY-TOTALS.
           05  REVENUE         PIC 9(9)V99.
           05  EXPENSES        PIC 9(9)V99.
           05  TAX             PIC 9(7)V99.

       01  ANNUAL-TOTALS.
           05  REVENUE         PIC 9(11)V99.
           05  EXPENSES        PIC 9(11)V99.
           05  TAX             PIC 9(9)V99.
           05  YEAR            PIC 9(4).

       ADD CORRESPONDING MONTHLY-TOTALS TO ANNUAL-TOTALS
       *> Adds REVENUE, EXPENSES, and TAX
       *> YEAR has no match in MONTHLY-TOTALS, so is unchanged
```

### SUBTRACT CORRESPONDING

```cobol
       01  DEDUCTIONS.
           05  FEDERAL-TAX     PIC 9(5)V99.
           05  STATE-TAX       PIC 9(5)V99.
           05  INSURANCE       PIC 9(5)V99.

       01  GROSS-PAY-REC.
           05  GROSS-AMOUNT    PIC 9(7)V99.
           05  FEDERAL-TAX     PIC 9(5)V99.
           05  STATE-TAX       PIC 9(5)V99.
           05  INSURANCE       PIC 9(5)V99.
           05  NET-PAY         PIC 9(7)V99.

       SUBTRACT CORRESPONDING DEDUCTIONS FROM GROSS-PAY-REC
       *> Subtracts FEDERAL-TAX, STATE-TAX, and INSURANCE
```

### Nested Groups with Matching

```cobol
       01  SOURCE-REC.
           05  ADDRESS-INFO.
               10  STREET      PIC X(30).
               10  CITY        PIC X(20).
               10  STATE       PIC X(2).
           05  PHONE           PIC X(15).

       01  DEST-REC.
           05  ADDRESS-INFO.
               10  STREET      PIC X(30).
               10  CITY        PIC X(20).
               10  ZIP         PIC X(10).
           05  PHONE           PIC X(15).

       MOVE CORRESPONDING SOURCE-REC TO DEST-REC
       *> Matches: STREET (via ADDRESS-INFO), CITY (via ADDRESS-INFO), PHONE
       *> STATE and ZIP don't match (different names)
```

---

## When to Use CORRESPONDING

- **Record-to-record copies** where source and destination share many fields with the same names
- **Accumulation** — adding monthly figures to annual totals
- **Avoiding long sequences** of individual MOVE statements

## When to Avoid CORRESPONDING

- **Fragile matching** — adding or renaming a field in one record can silently change which items correspond
- **Performance** — some compilers generate less efficient code for CORRESPONDING than for individual operations
- **Readability** — it's not immediately obvious which fields are being affected without examining both group definitions

!!! tip "Best Practice"
    Many coding standards discourage CORRESPONDING because the implicit
    matching makes programs harder to maintain. Explicit individual MOVE
    statements are more self-documenting.

---

## See Also

- [MOVE](move.md) -- data movement
- [ADD](../arithmetic/add.md) -- addition
- [SUBTRACT](../arithmetic/subtract.md) -- subtraction
- [Name Qualification](../../language/qualification.md) -- IN/OF disambiguation
