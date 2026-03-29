# Name Qualification

Qualification is the mechanism for disambiguating data names and procedure names that appear more than once in a program. A non-unique name is qualified by specifying the hierarchy of group items or sections that contain it.

- **Standard:** COBOL-60, COBOL-68, COBOL-85, COBOL 2002, COBOL 2014, COBOL 2023

---

## Syntax

### Data Name Qualification

```cobol
data-name [ { IN | OF } group-name ] ...
           [ { IN | OF } file-name ]
```

### Procedure Name Qualification

```cobol
paragraph-name [ { IN | OF } section-name ]
```

The keywords `IN` and `OF` are interchangeable.

---

## Rules

1. A data name must be **unique** within a program, or must be made unique through qualification.
2. Qualification proceeds from the most specific (innermost) to the most general (outermost) level.
3. Not every level of the hierarchy needs to be specified — intermediate levels can be **omitted** as long as the result is unambiguous.
4. The final qualifier for a data item can be a file name (FD entry name).
5. Qualification is resolved at compile time, not runtime.
6. Condition names (level-88) are qualified by the data item they are defined on.
7. FILLER items cannot be qualified (they have no name by definition).

---

## Examples

### Basic Data Name Qualification

```cobol
       FD  CUSTOMER-FILE.
       01  CUSTOMER-RECORD.
           05  CUST-NAME    PIC X(30).
           05  CUST-ADDRESS.
               10  STREET   PIC X(30).
               10  CITY     PIC X(20).
               10  STATE    PIC X(2).

       FD  VENDOR-FILE.
       01  VENDOR-RECORD.
           05  VENDOR-NAME  PIC X(30).
           05  VENDOR-ADDRESS.
               10  STREET   PIC X(30).
               10  CITY     PIC X(20).
               10  STATE    PIC X(2).

      *> STREET is ambiguous -- it exists in both records
      *> Qualify to disambiguate:

       MOVE "123 Main St" TO STREET OF CUST-ADDRESS
       MOVE "456 Oak Ave" TO STREET IN VENDOR-ADDRESS

      *> Or qualify with the file name:
       MOVE "NY" TO STATE OF CUSTOMER-FILE
```

### Skipping Intermediate Levels

```cobol
      *> Full qualification:
       MOVE "IL" TO STATE OF CUST-ADDRESS OF CUSTOMER-RECORD
                    OF CUSTOMER-FILE

      *> Sufficient qualification (skip intermediates):
       MOVE "IL" TO STATE OF CUSTOMER-FILE

      *> As long as the result is unambiguous, any level works:
       MOVE "IL" TO STATE OF CUST-ADDRESS
```

### Procedure Name Qualification

```cobol
       PROCESS-SECTION SECTION.
       READ-PARA.
           READ INPUT-FILE ...

       OUTPUT-SECTION SECTION.
       READ-PARA.
           READ OUTPUT-FILE ...

      *> Qualify paragraph names by section:
       PERFORM READ-PARA OF PROCESS-SECTION
       PERFORM READ-PARA IN OUTPUT-SECTION
```

### Condition Name Qualification

```cobol
       01  CUST-STATUS    PIC X.
           88  ACTIVE      VALUE "A".
       01  ORDER-STATUS   PIC X.
           88  ACTIVE      VALUE "Y".

      *> ACTIVE is ambiguous:
       IF ACTIVE OF CUST-STATUS
           DISPLAY "Customer is active"
       END-IF
```

---

## Qualification vs Subscripting

Qualification and subscripting serve different purposes and may be combined:

| Mechanism | Purpose | Syntax |
|-----------|---------|--------|
| Qualification | Disambiguate non-unique names | `name OF group` |
| Subscripting | Select an occurrence in a table | `name(subscript)` |
| Both | Disambiguate and select occurrence | `name OF group (subscript)` |

```cobol
      *> Combined: qualify a subscripted item
       MOVE 100 TO AMOUNT OF TRANS-RECORD(WS-IDX)
```

---

## See Also

- [Level Numbers](../data-division/level-numbers.md) -- data hierarchy
- [OCCURS Clause](../data-division/occurs.md) -- table definitions and subscripting
- [Reference Modification](reference-modification.md) -- substring access
- [Condition Names](../data-division/condition-names.md) -- level-88 entries
