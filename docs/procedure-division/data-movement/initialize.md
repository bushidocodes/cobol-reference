# INITIALIZE

The `INITIALIZE` statement sets the contents of a group item or elementary item to predetermined values based on category, replacing the need for multiple MOVE statements.

- **Standard:** COBOL-85, COBOL 2002, COBOL 2014, COBOL 2023
- **Division:** Procedure Division
- **Category:** Data Movement

---

## Syntax

```cobol
INITIALIZE identifier-1 [ identifier-2 ] ...
    [ WITH FILLER ]
    [ ALL TO VALUE ]
    [ REPLACING { ALPHABETIC | ALPHANUMERIC | NUMERIC
                | ALPHANUMERIC-EDITED | NUMERIC-EDITED
                | NATIONAL | NATIONAL-EDITED }
        DATA BY { identifier | literal } ] ...
```

---

## Rules

1. Without any phrases, INITIALIZE sets elementary items within the target to their default values based on category:

    | Category | Default Value |
    |----------|--------------|
    | Alphabetic | SPACES |
    | Alphanumeric | SPACES |
    | Alphanumeric-Edited | SPACES |
    | National | SPACES |
    | National-Edited | SPACES |
    | Numeric | ZEROS |
    | Numeric-Edited | ZEROS |

2. **FILLER items** are skipped by default. The `WITH FILLER` phrase (COBOL 2002) causes FILLER items to be initialized as well.

3. **REPLACING** allows overriding the defaults for specific categories. For example, `REPLACING NUMERIC DATA BY 1` sets all numeric items to 1 instead of zero.

4. **ALL TO VALUE** (COBOL 2002) reinitializes items to the values specified in their `VALUE` clauses. Items without VALUE clauses are initialized to category defaults.

5. **OCCURS** items: all occurrences are initialized.

6. **REDEFINES** items: only the subject of a REDEFINES is initialized, not the redefined area.

7. **Index data items** are not affected by INITIALIZE.

8. Multiple identifiers can be initialized in a single statement.

---

## Examples

### Basic Group Initialization

```cobol
       01  WS-OUTPUT-RECORD.
           05  WS-NAME        PIC X(30).
           05  WS-AMOUNT      PIC 9(7)V99.
           05  WS-CODE        PIC X(5).
           05  WS-COUNT       PIC 9(3).

       INITIALIZE WS-OUTPUT-RECORD
       *> WS-NAME   = SPACES
       *> WS-AMOUNT = 0000000.00
       *> WS-CODE   = SPACES
       *> WS-COUNT  = 000
```

### Using REPLACING

```cobol
       INITIALIZE WS-OUTPUT-RECORD
           REPLACING NUMERIC DATA BY -1
       *> WS-NAME   = SPACES (alphanumeric default)
       *> WS-AMOUNT = -0000001.00
       *> WS-CODE   = SPACES (alphanumeric default)
       *> WS-COUNT  = -001
```

### Initializing Before Output

```cobol
       PERFORM VARYING WS-IDX FROM 1 BY 1
           UNTIL WS-IDX > WS-RECORD-COUNT
           INITIALIZE WS-PRINT-LINE
           MOVE CUST-NAME(WS-IDX) TO PL-NAME
           MOVE CUST-BAL(WS-IDX)  TO PL-BALANCE
           WRITE PRINT-RECORD FROM WS-PRINT-LINE
       END-PERFORM
```

### INITIALIZE vs MOVE

```cobol
      *> Without INITIALIZE — multiple statements needed
       MOVE SPACES TO WS-NAME WS-CODE
       MOVE ZEROS  TO WS-AMOUNT WS-COUNT

      *> With INITIALIZE — single statement
       INITIALIZE WS-OUTPUT-RECORD
```

INITIALIZE is especially valuable for group items with many subordinate fields of mixed categories, where a simple `MOVE SPACES` or `MOVE ZEROS` would not correctly initialize all fields.

---

## See Also

- [MOVE](move.md) -- data movement
- [VALUE Clause](../../data-division/value.md) -- compile-time initialization
- [Level Numbers](../../data-division/level-numbers.md) -- data hierarchy
