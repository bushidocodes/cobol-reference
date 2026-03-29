# Conditions and Conditional Expressions

Conditions are expressions that evaluate to true or false at runtime. They are used in IF, EVALUATE, PERFORM UNTIL, SEARCH WHEN, and other conditional contexts.

- **Standard:** COBOL-60, COBOL-68, COBOL-85, COBOL 2002, COBOL 2014, COBOL 2023

---

## Types of Simple Conditions

### Relation Conditions

Compare two operands using relational operators.

```cobol
identifier-1 IS [NOT] { GREATER THAN | > }  identifier-2
                       { LESS THAN    | < }
                       { EQUAL TO     | = }
                       { GREATER THAN OR EQUAL TO | >= }
                       { LESS THAN OR EQUAL TO    | <= }
```

```cobol
IF WS-AMOUNT > 1000
IF WS-NAME = "SMITH"
IF WS-DATE NOT < WS-CUTOFF-DATE
```

### Class Conditions

Test whether a data item contains a specific class of characters.

```cobol
identifier IS [NOT] { NUMERIC
                    | ALPHABETIC
                    | ALPHABETIC-LOWER
                    | ALPHABETIC-UPPER
                    | DBCS
                    | KANJI
                    | class-name }
```

```cobol
IF WS-INPUT IS NUMERIC
    COMPUTE WS-VALUE = FUNCTION NUMVAL(WS-INPUT)
END-IF

IF WS-CODE IS NOT ALPHABETIC
    DISPLAY "Invalid code"
END-IF
```

- NUMERIC: contains only digits 0-9 (and sign for signed items)
- ALPHABETIC: contains only letters A-Z, a-z, and spaces
- ALPHABETIC-LOWER/UPPER: added in COBOL-85
- User-defined class names are defined in the SPECIAL-NAMES paragraph

!!! note
    Only USAGE DISPLAY items can be tested with NUMERIC or ALPHABETIC.
    COMP/BINARY items are always numeric by definition.

### Sign Conditions

Test the algebraic sign of a numeric item.

```cobol
identifier IS [NOT] { POSITIVE | NEGATIVE | ZERO }
```

```cobol
IF WS-BALANCE IS NEGATIVE
    DISPLAY "Account overdrawn"
END-IF
IF WS-COUNT IS ZERO
    DISPLAY "No records found"
END-IF
```

- POSITIVE: value > 0
- NEGATIVE: value < 0
- ZERO: value = 0

### Condition-Name Conditions

Test whether a data item has one of the values associated with a level-88 condition name.

```cobol
condition-name
```

```cobol
01  WS-STATUS     PIC X.
    88  ACTIVE     VALUE "A".
    88  INACTIVE   VALUE "I".
    88  DELETED    VALUE "D".

IF ACTIVE
    PERFORM PROCESS-ACTIVE
END-IF
```

See [Condition Names](../data-division/condition-names.md).

### Switch-Status Conditions

Test the status of an external switch defined in SPECIAL-NAMES.

```cobol
switch-status-name
```

```cobol
SPECIAL-NAMES.
    SWITCH-1 ON STATUS IS DEBUG-MODE.

IF DEBUG-MODE
    DISPLAY "Debug output enabled"
END-IF
```

---

## Compound Conditions

Simple conditions can be combined with logical operators.

### AND

True only if both conditions are true.

```cobol
IF WS-AGE >= 18 AND WS-AGE <= 65
    PERFORM STANDARD-PROCESSING
END-IF
```

### OR

True if either or both conditions are true.

```cobol
IF WS-STATUS = "A" OR WS-STATUS = "P"
    PERFORM PROCESS-ACTIVE
END-IF
```

### NOT

Reverses the truth value of a condition.

```cobol
IF NOT WS-EOF
    PERFORM READ-NEXT
END-IF
```

### Precedence

Evaluation order (highest to lowest):
1. Parenthesized expressions
2. NOT
3. AND
4. OR

```cobol
*> This:
IF A = B OR C = D AND E = F

*> Is equivalent to:
IF A = B OR (C = D AND E = F)

*> Use parentheses for clarity:
IF (A = B OR C = D) AND E = F
```

---

## Abbreviated Combined Conditions

COBOL allows shorthand notation when the same subject or relational operator is used in consecutive conditions.

### Abbreviated Subject

```cobol
*> Full form:
IF WS-CODE = "A" OR WS-CODE = "B" OR WS-CODE = "C"

*> Abbreviated (subject implied):
IF WS-CODE = "A" OR "B" OR "C"
```

### Abbreviated Subject and Operator

```cobol
*> Full form:
IF WS-VALUE > 10 AND WS-VALUE > 20

*> Abbreviated:
IF WS-VALUE > 10 AND 20
```

### Complex Abbreviation

```cobol
*> Full form:
IF WS-AGE >= 18 AND WS-AGE <= 65

*> Abbreviated (subject and first operator implied):
IF WS-AGE >= 18 AND <= 65
```

!!! warning "Abbreviation Pitfalls"
    Abbreviated conditions can be confusing. `IF A = B OR C` means
    `IF A = B OR A = C`, **not** `IF (A = B) OR C`. When C is a condition
    name, this distinction matters. Use parentheses for clarity.

---

## Negated Conditions

The NOT keyword can negate any condition:

```cobol
IF NOT (WS-STATUS = "A" OR "B")
*> True when WS-STATUS is neither "A" nor "B"

IF WS-AMOUNT IS NOT NUMERIC
*> True when WS-AMOUNT contains non-numeric characters
```

---

## See Also

- [IF](../procedure-division/control-flow/if.md) -- conditional execution
- [EVALUATE](../procedure-division/control-flow/evaluate.md) -- multi-way branching
- [PERFORM](../procedure-division/control-flow/perform.md) -- conditional looping (UNTIL)
- [Condition Names](../data-division/condition-names.md) -- level-88 entries
- [SPECIAL-NAMES](../environment-division/special-names.md) -- class definitions and switches
