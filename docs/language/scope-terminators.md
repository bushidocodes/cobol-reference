# Scope Terminators

A scope terminator is a reserved word beginning with `END-` that explicitly marks the end of a statement. Scope terminators eliminate ambiguity in nested and conditional statement structures and are the preferred method of scope delimitation in modern COBOL programs.

**Standard:** COBOL-85, COBOL 2002, COBOL 2014, COBOL 2023

---

## Complete List of Scope Terminators

The following table lists all scope terminators defined in the COBOL standard:

| Scope Terminator | Statement | Standard |
|---|---|---|
| `END-ACCEPT` | ACCEPT | COBOL 2002 |
| `END-ADD` | [ADD](../procedure-division/arithmetic/add.md) | COBOL-85 |
| `END-CALL` | CALL | COBOL-85 |
| `END-COMPUTE` | [COMPUTE](../procedure-division/arithmetic/compute.md) | COBOL-85 |
| `END-DELETE` | DELETE | COBOL-85 |
| `END-DISPLAY` | DISPLAY | COBOL 2002 |
| `END-DIVIDE` | DIVIDE | COBOL-85 |
| `END-EVALUATE` | [EVALUATE](../procedure-division/control-flow/evaluate.md) | COBOL-85 |
| `END-IF` | [IF](../procedure-division/control-flow/if.md) | COBOL-85 |
| `END-MULTIPLY` | MULTIPLY | COBOL-85 |
| `END-PERFORM` | [PERFORM](../procedure-division/control-flow/perform.md) (inline) | COBOL-85 |
| `END-READ` | [READ](../procedure-division/io/read.md) | COBOL-85 |
| `END-RETURN` | RETURN | COBOL-85 |
| `END-REWRITE` | REWRITE | COBOL-85 |
| `END-SEARCH` | SEARCH | COBOL-85 |
| `END-START` | START | COBOL-85 |
| `END-STRING` | [STRING](../procedure-division/string-handling/string.md) | COBOL-85 |
| `END-SUBTRACT` | SUBTRACT | COBOL-85 |
| `END-UNSTRING` | UNSTRING | COBOL-85 |
| `END-WRITE` | [WRITE](../procedure-division/io/write.md) | COBOL-85 |

!!! note
    `END-ACCEPT` and `END-DISPLAY` were introduced in COBOL 2002. All other scope terminators were introduced in COBOL-85 as Nucleus Level 2 features. Their addition was one of the most significant changes in the COBOL-85 standard, enabling structured programming without reliance on the period as a scope delimiter. Prior to COBOL-85, scope termination was performed exclusively by the period.

## Explicit vs. Implicit Scope Termination

COBOL provides two mechanisms for terminating the scope of a conditional or inline statement.

### Explicit Scope Termination

Explicit scope termination uses an `END-verb` keyword to mark the precise end of a statement. The scope terminator corresponds to the verb that began the statement:

```cobol
       IF WS-BALANCE < 0
           DISPLAY "Overdrawn"
           MOVE "Y" TO WS-OVERDRAWN-FLAG
       END-IF
```

Explicit scope terminators turn a conditional statement into a delimited statement, which is classified as imperative. A delimited statement may therefore appear anywhere an imperative statement is required, including within the branches of other conditional statements.

### Implicit Scope Termination

Implicit scope termination occurs when a period (`.`) ends a sentence. The period terminates all open statements in the sentence simultaneously:

```cobol
       IF WS-BALANCE < 0
           DISPLAY "Overdrawn"
           MOVE "Y" TO WS-OVERDRAWN-FLAG.
```

The period at the end closes the [IF](../procedure-division/control-flow/if.md) statement. While this is sufficient in simple cases, it becomes problematic with nested statements because a single period closes every open scope at once.

## Why Scope Terminators Matter

### Nested Statements and the Dangling ELSE Problem

Scope terminators are essential for correct nesting of conditional statements. Without them, the compiler cannot reliably determine which `ELSE` belongs to which `IF`:

**Correct -- with scope terminators:**

```cobol
       IF WS-TYPE = "A"
           IF WS-AMOUNT > 1000
               DISPLAY "Large type A"
           ELSE
               DISPLAY "Small type A"
           END-IF
       ELSE
           DISPLAY "Not type A"
       END-IF
```

**Ambiguous -- without scope terminators (dangling ELSE):**

```cobol
       IF WS-TYPE = "A"
           IF WS-AMOUNT > 1000
               DISPLAY "Large type A"
           ELSE
               DISPLAY "Not type A".
```

In the second example, the `ELSE` associates with the inner [IF](../procedure-division/control-flow/if.md), not the outer one. The period terminates both `IF` statements, so there is no outer `ELSE` branch at all. This is a common source of logic errors in programs that do not use scope terminators.

### Inline PERFORM

The `END-PERFORM` scope terminator enables inline [PERFORM](../procedure-division/control-flow/perform.md), where the loop body is written directly within the PERFORM statement rather than in a separate paragraph:

```cobol
       PERFORM VARYING WS-I FROM 1 BY 1
           UNTIL WS-I > 10
           DISPLAY WS-I
           ADD WS-I TO WS-TOTAL
       END-PERFORM
```

Without `END-PERFORM`, the PERFORM statement must reference a paragraph or section name and the loop body is written separately.

### Conditional I/O Statements

Scope terminators clarify the extent of exception-handling phrases in I/O statements:

```cobol
       READ INPUT-FILE INTO WS-RECORD
           AT END
               SET WS-EOF TO TRUE
           NOT AT END
               ADD 1 TO WS-RECORD-COUNT
       END-READ
```

Without `END-READ`, the statements following `NOT AT END` would need to be terminated by a period, preventing them from being embedded within other control structures.

## Common Pitfalls

### Premature Period Termination

A period inside a nested structure terminates all open statements, which can produce unexpected behavior:

```cobol
      * BUG: The period terminates both the inner and outer IF.
       IF WS-STATUS = "ACTIVE"
           IF WS-BALANCE > 0
               ADD WS-BALANCE TO WS-TOTAL.
           DISPLAY "Status checked"
       END-IF
```

The period after `WS-TOTAL` closes both `IF` statements. The `DISPLAY` and `END-IF` that follow are not part of any `IF` statement -- the `DISPLAY` executes unconditionally and the `END-IF` produces a compilation error. The corrected version uses scope terminators throughout:

```cobol
       IF WS-STATUS = "ACTIVE"
           IF WS-BALANCE > 0
               ADD WS-BALANCE TO WS-TOTAL
           END-IF
           DISPLAY "Status checked"
       END-IF
```

### Mixing Periods and Scope Terminators

Programs that inconsistently mix periods and scope terminators are error-prone. Best practice is to use scope terminators for all nested and conditional statements, and to use the period only to end the final statement in a paragraph or sentence:

```cobol
       PROCESS-RECORD.
           EVALUATE TRUE
               WHEN WS-TYPE = "A"
                   PERFORM PROCESS-TYPE-A
               WHEN WS-TYPE = "B"
                   PERFORM PROCESS-TYPE-B
               WHEN OTHER
                   DISPLAY "Unknown type"
           END-EVALUATE

           IF WS-COUNT > WS-LIMIT
               DISPLAY "Limit exceeded"
           END-IF
           .
```

!!! note
    Some coding standards place the terminating period on a line by itself (as shown above) to make it visually distinct and to reduce the risk of accidentally terminating a statement prematurely.

### Arithmetic Statements with ON SIZE ERROR

Scope terminators are particularly important for arithmetic statements that include `ON SIZE ERROR` or `NOT ON SIZE ERROR` phrases:

```cobol
       ADD WS-A TO WS-B
           ON SIZE ERROR
               DISPLAY "Overflow occurred"
           NOT ON SIZE ERROR
               DISPLAY "Addition successful"
       END-ADD
```

Without `END-ADD`, the compiler cannot determine where the `NOT ON SIZE ERROR` phrase ends, especially if the statement is nested within an [IF](../procedure-division/control-flow/if.md) or [EVALUATE](../procedure-division/control-flow/evaluate.md).

## See Also

- [Character Set and Words](character-set.md) -- COBOL sentences, statements, and clauses
- [Figurative Constants](figurative-constants.md) -- predefined constant values
- [IF](../procedure-division/control-flow/if.md) -- conditional branching
- [EVALUATE](../procedure-division/control-flow/evaluate.md) -- multi-branch conditional
- [PERFORM](../procedure-division/control-flow/perform.md) -- loop and paragraph invocation
- [READ](../procedure-division/io/read.md) -- file read operations
- [WRITE](../procedure-division/io/write.md) -- file write operations
