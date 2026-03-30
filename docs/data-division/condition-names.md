# Condition Names (Level 88)

A condition name associates a user-defined name with a specific value, set of values, or range of values of an elementary data item (the conditional variable). When used in a conditional expression, the condition name tests whether the conditional variable currently contains one of the specified values.

**Standard:** COBOL-60, COBOL-61, COBOL-61 Extended, COBOL-65, COBOL-68, COBOL-85, COBOL 2002, COBOL 2014, COBOL 2023

---

## Syntax

```cobol
88  condition-name  { VALUE IS   | VALUES ARE }
    literal-1  [ { THRU | THROUGH } literal-2 ]
    [ literal-3  [ { THRU | THROUGH } literal-4 ] ] ...
    .
```

A level-88 entry must immediately follow the data description entry of its conditional variable.

---

## Description

A condition name is a level-88 data description entry that does not define a data item; it defines a named condition. The data item immediately preceding the level-88 entry (or the set of level-88 entries) is the **conditional variable**. The condition name evaluates to true when the conditional variable contains any one of the values specified in the `VALUE` clause.

Condition names make programs more readable and maintainable by replacing literal comparisons with meaningful names.

---

## Rules

### Definition Rules

1. A level-88 entry must immediately follow its conditional variable. The conditional variable may be any elementary item defined at levels 01-49 or level 77.
2. A level-88 entry may not itself have subordinate entries or clauses other than `VALUE`.
3. A level-88 item does not occupy any storage.
4. Multiple level-88 entries may be associated with the same conditional variable.
5. The conditional variable must not have a USAGE of `INDEX` or `POINTER`.
6. The `VALUE` clause is required on every level-88 entry.

### Value Rules

1. Literal values must be consistent with the PICTURE and USAGE of the conditional variable.
2. For a numeric conditional variable, literal values must be numeric.
3. For an alphabetic or alphanumeric conditional variable, literal values must be alphanumeric (or figurative constants).
4. When `THRU` is specified, `literal-1` must be less than `literal-2` according to the collating sequence in effect.
5. Multiple individual values and ranges may be combined in a single level-88 entry.
6. Figurative constants (`ZERO`, `SPACES`, `HIGH-VALUES`, `LOW-VALUES`) may be used as literal values.

### Evaluation Rules

1. A condition name evaluates to **true** if the current value of the conditional variable is equal to any single value specified, or falls within any range specified (inclusive of both endpoints).
2. A condition name evaluates to **false** otherwise.
3. For ranges on alphanumeric items, the comparison uses the program's collating sequence.

---

## Using Condition Names in Conditional Expressions

Condition names may appear anywhere a condition is expected: `IF`, `EVALUATE`, `PERFORM UNTIL`, `SEARCH WHEN`, etc.

### In IF Statements

```cobol
       01  WS-STATUS-CODE     PIC X(2).
           88  STATUS-OK      VALUE '00'.
           88  STATUS-EOF     VALUE '10'.
           88  STATUS-ERROR   VALUE '30' THRU '99'.

       ...
       IF STATUS-OK
           PERFORM PROCESS-RECORD
       END-IF.

       IF STATUS-EOF
           PERFORM END-OF-FILE-ROUTINE
       END-IF.

       IF STATUS-ERROR
           DISPLAY "Error: " WS-STATUS-CODE
           PERFORM ERROR-ROUTINE
       END-IF.
```

The `IF STATUS-OK` statement is equivalent to `IF WS-STATUS-CODE = '00'`.

### In EVALUATE Statements

```cobol
       01  WS-TRANSACTION-TYPE  PIC X.
           88  TXN-DEPOSIT      VALUE 'D'.
           88  TXN-WITHDRAWAL   VALUE 'W'.
           88  TXN-TRANSFER     VALUE 'T'.
           88  TXN-INQUIRY      VALUE 'I'.
           88  TXN-VALID        VALUE 'D' 'W' 'T' 'I'.

       ...
       EVALUATE TRUE
           WHEN TXN-DEPOSIT
               PERFORM PROCESS-DEPOSIT
           WHEN TXN-WITHDRAWAL
               PERFORM PROCESS-WITHDRAWAL
           WHEN TXN-TRANSFER
               PERFORM PROCESS-TRANSFER
           WHEN TXN-INQUIRY
               PERFORM PROCESS-INQUIRY
           WHEN OTHER
               DISPLAY "Invalid transaction type"
       END-EVALUATE.
```

The `EVALUATE TRUE` / `WHEN condition-name` pattern is the standard COBOL idiom for a multi-branch decision based on condition names.

### In PERFORM UNTIL

```cobol
       01  WS-EOF-FLAG     PIC X  VALUE 'N'.
           88  END-OF-FILE  VALUE 'Y'.
           88  NOT-EOF      VALUE 'N'.

       ...
       PERFORM READ-NEXT-RECORD
           UNTIL END-OF-FILE.
```

### In SEARCH WHEN

```cobol
       01  STATE-TABLE.
           05  STATE-ENTRY  OCCURS 50 TIMES
                            INDEXED BY ST-IDX.
               10  STATE-CODE  PIC XX.
               10  STATE-NAME  PIC X(20).
               10  STATE-REGION PIC 9.
                   88  REGION-NORTHEAST  VALUE 1.
                   88  REGION-SOUTHEAST  VALUE 2.
                   88  REGION-MIDWEST    VALUE 3.
                   88  REGION-WEST       VALUE 4.

       ...
       SET ST-IDX TO 1.
       SEARCH STATE-ENTRY
           AT END DISPLAY "Not found"
           WHEN REGION-WEST(ST-IDX)
               DISPLAY STATE-NAME(ST-IDX)
       END-SEARCH.
```

When the conditional variable is subordinate to an OCCURS item, the subscript or index must be provided with the condition name.

---

## The SET Statement with Condition Names

The `SET` statement sets the conditional variable to the first value specified in the level-88 entry, making the condition true.

### Syntax

```cobol
SET condition-name-1 [condition-name-2 ...] TO TRUE.
```

### Behavior

1. `SET condition-name TO TRUE` moves the first literal value from the condition name's `VALUE` clause into the conditional variable.
2. If the condition name specifies a range (`THRU`), the first value of the range is moved.
3. Multiple condition names may be set in a single `SET` statement, each associated with a different conditional variable.
4. `SET condition-name TO FALSE` is defined in COBOL 2002 and later, provided the level-88 entry has a `WHEN SET TO FALSE` clause:

```cobol
       01  WS-ACTIVE-FLAG    PIC X.
           88  IS-ACTIVE     VALUE 'Y'
                             WHEN SET TO FALSE 'N'.

       SET IS-ACTIVE TO TRUE.     *> WS-ACTIVE-FLAG = 'Y'
       SET IS-ACTIVE TO FALSE.    *> WS-ACTIVE-FLAG = 'N'
```

### Examples

```cobol
       01  WS-ACCOUNT-TYPE      PIC X.
           88  ACCT-CHECKING    VALUE 'C'.
           88  ACCT-SAVINGS     VALUE 'S'.
           88  ACCT-LOAN        VALUE 'L'.

       ...
       SET ACCT-SAVINGS TO TRUE.
      *> WS-ACCOUNT-TYPE now contains 'S'

       01  WS-PROCESS-FLAG      PIC 9.
           88  PROCESS-COMPLETE VALUE 1.
           88  PROCESS-PENDING  VALUE 0.

       ...
       SET PROCESS-COMPLETE TO TRUE.
      *> WS-PROCESS-FLAG now contains 1
```

---

## Single Values, Multiple Values, and Ranges

### Single Value

```cobol
       01  WS-GENDER  PIC X.
           88  GENDER-MALE    VALUE 'M'.
           88  GENDER-FEMALE  VALUE 'F'.
```

### Multiple Discrete Values

```cobol
       01  WS-VOWEL-CHECK  PIC X.
           88  IS-VOWEL     VALUE 'A' 'E' 'I' 'O' 'U'
                                  'a' 'e' 'i' 'o' 'u'.
```

`IS-VOWEL` is true if `WS-VOWEL-CHECK` contains any one of the ten listed characters.

### Range of Values

```cobol
       01  WS-SCORE  PIC 9(3).
           88  SCORE-FAIL     VALUE 0 THRU 59.
           88  SCORE-PASS     VALUE 60 THRU 100.

       01  WS-CHAR  PIC X.
           88  IS-UPPERCASE   VALUE 'A' THRU 'Z'.
           88  IS-LOWERCASE   VALUE 'a' THRU 'z'.
           88  IS-DIGIT       VALUE '0' THRU '9'.
           88  IS-ALPHA       VALUE 'A' THRU 'Z'
                                    'a' THRU 'z'.
           88  IS-ALPHANUMERIC VALUE 'A' THRU 'Z'
                                     'a' THRU 'z'
                                     '0' THRU '9'.
```

### Combining Discrete Values and Ranges

```cobol
       01  WS-MONTH  PIC 99.
           88  MONTH-VALID       VALUE 1 THRU 12.
           88  MONTH-QUARTER-1   VALUE 1 THRU 3.
           88  MONTH-QUARTER-2   VALUE 4 THRU 6.
           88  MONTH-QUARTER-3   VALUE 7 THRU 9.
           88  MONTH-QUARTER-4   VALUE 10 THRU 12.
           88  MONTH-31-DAYS     VALUE 1 3 5 7 8 10 12.
           88  MONTH-30-DAYS     VALUE 4 6 9 11.
           88  MONTH-FEBRUARY    VALUE 2.
```

---

## Relationship to the Conditional Variable

The condition name does not exist independently. It is an attribute of the conditional variable and evaluates against that variable's current value at the time of the test.

```cobol
       01  WS-FILE-STATUS   PIC XX.
           88  FS-SUCCESS    VALUE '00'.
           88  FS-EOF        VALUE '10'.
           88  FS-NOT-FOUND  VALUE '23'.
           88  FS-DUPLICATE  VALUE '22'.
           88  FS-PERM-ERROR VALUE '30' THRU '39'.

       ...
       READ INPUT-FILE INTO WS-RECORD
           FILE STATUS IS WS-FILE-STATUS
       END-READ.

       IF FS-SUCCESS
           ADD 1 TO WS-READ-COUNT
       ELSE IF FS-EOF
           SET END-OF-FILE TO TRUE
       ELSE
           DISPLAY "File error: " WS-FILE-STATUS
       END-IF.
```

After the `READ`, the runtime sets `WS-FILE-STATUS`. Each condition name test checks the current content of `WS-FILE-STATUS`. If the file status is `'00'`, then `FS-SUCCESS` is true. If `'35'`, then `FS-PERM-ERROR` is true (because `'35'` falls within `'30' THRU '39'`).

---

## Condition Names with Tables

When the conditional variable is part of a table (has an OCCURS clause or is subordinate to one), the condition name must be subscripted or indexed.

```cobol
       01  EMPLOYEE-TABLE.
           05  EMP-ENTRY  OCCURS 100 TIMES
                          INDEXED BY EMP-IDX.
               10  EMP-ID       PIC 9(6).
               10  EMP-STATUS   PIC X.
                   88  EMP-ACTIVE    VALUE 'A'.
                   88  EMP-INACTIVE  VALUE 'I'.
                   88  EMP-RETIRED   VALUE 'R'.

       ...
       PERFORM VARYING EMP-IDX FROM 1 BY 1
               UNTIL EMP-IDX > 100
           IF EMP-ACTIVE(EMP-IDX)
               ADD 1 TO WS-ACTIVE-COUNT
           END-IF
       END-PERFORM.
```

---

## Common Patterns

### Boolean Flag

```cobol
       01  WS-MORE-RECORDS     PIC X  VALUE 'Y'.
           88  MORE-RECORDS    VALUE 'Y'.
           88  NO-MORE-RECORDS VALUE 'N'.

       PERFORM UNTIL NO-MORE-RECORDS
           READ INPUT-FILE INTO WS-RECORD
               AT END SET NO-MORE-RECORDS TO TRUE
               NOT AT END PERFORM PROCESS-RECORD
           END-READ
       END-PERFORM.
```

### Validation

```cobol
       01  WS-PAYMENT-METHOD   PIC X(2).
           88  PAY-CASH        VALUE 'CA'.
           88  PAY-CHECK       VALUE 'CK'.
           88  PAY-CREDIT      VALUE 'CC'.
           88  PAY-DEBIT       VALUE 'DB'.
           88  PAY-WIRE        VALUE 'WR'.
           88  PAY-VALID       VALUE 'CA' 'CK' 'CC' 'DB' 'WR'.

       ...
       IF NOT PAY-VALID
           DISPLAY "Invalid payment method: " WS-PAYMENT-METHOD
           PERFORM REJECT-TRANSACTION
       END-IF.
```

### Grouping Related Conditions

```cobol
       01  WS-HTTP-STATUS   PIC 9(3).
           88  HTTP-OK              VALUE 200.
           88  HTTP-CREATED         VALUE 201.
           88  HTTP-NO-CONTENT      VALUE 204.
           88  HTTP-SUCCESS         VALUE 200 THRU 299.
           88  HTTP-BAD-REQUEST     VALUE 400.
           88  HTTP-UNAUTHORIZED    VALUE 401.
           88  HTTP-FORBIDDEN       VALUE 403.
           88  HTTP-NOT-FOUND       VALUE 404.
           88  HTTP-CLIENT-ERROR    VALUE 400 THRU 499.
           88  HTTP-SERVER-ERROR    VALUE 500 THRU 599.
           88  HTTP-ERROR           VALUE 400 THRU 599.
```

---

## See Also

- [Data Division](index.md)
- [Level Numbers](level-numbers.md)
- [PICTURE Clause](picture.md)
- [VALUE Clause](value.md)
- [SET Statement](../procedure-division/data-movement/set.md)
- [IF Statement](../procedure-division/control-flow/if.md)
- [EVALUATE Statement](../procedure-division/control-flow/evaluate.md)
