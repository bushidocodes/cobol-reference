# COBOL 2023 New Statements

This page documents statements and features introduced in the COBOL 2023 standard (ISO/IEC 1989:2023) that do not have dedicated pages elsewhere.

- **Standard:** COBOL 2023

!!! note "Compiler Support"
    COBOL 2023 features have very limited compiler support as of 2025.
    GCC 15.1's `gcobol` front-end targets this standard. Other compilers
    may add support incrementally.

---

## DELETE FILE

Deletes a file from external storage.

```cobol
DELETE FILE file-name-1 [ file-name-2 ] ...
```

### Rules

1. The file must be closed (not open) when DELETE FILE is executed.
2. The file is removed from external storage (disk, etc.).
3. This is distinct from the `DELETE` statement (which removes a record from an open file).
4. FILE STATUS is updated after the operation.

```cobol
CLOSE OLD-MASTER-FILE
DELETE FILE OLD-MASTER-FILE
IF WS-STATUS NOT = "00"
    DISPLAY "Could not delete file"
END-IF
```

---

## SEND and RECEIVE

Asynchronous messaging statements for inter-program or inter-system communication.

```cobol
SEND message-data TO destination
    [ ON EXCEPTION imperative-statement ]
END-SEND

RECEIVE message-data FROM source
    [ ON EXCEPTION imperative-statement ]
END-RECEIVE
```

### Rules

1. SEND transmits a message to a specified destination.
2. RECEIVE retrieves a message from a specified source.
3. These statements support asynchronous, message-based communication patterns.
4. The exact semantics of destinations and sources are implementation-defined.

!!! note
    SEND and RECEIVE in COBOL 2023 are distinct from the obsolete
    Communication module's SEND/RECEIVE (COBOL-85), which dealt with
    message control systems (MCS). The 2023 versions provide modern
    asynchronous messaging.

---

## PERFORM UNTIL EXIT

A defined infinite looping construct, terminated only by EXIT PERFORM within the loop body.

```cobol
PERFORM UNTIL EXIT
    ACCEPT WS-INPUT
    IF WS-INPUT = "QUIT"
        EXIT PERFORM
    END-IF
    PERFORM PROCESS-INPUT
END-PERFORM
```

### Rules

1. The loop condition is always false (never causes termination).
2. The loop must be terminated by an explicit `EXIT PERFORM` statement within the body.
3. This replaces the common idiom `PERFORM UNTIL 1 = 0` or `PERFORM UNTIL WS-DONE`.

---

## Boolean XOR Operator

Exclusive-or logical operator for conditional expressions.

```cobol
IF condition-1 XOR condition-2
    imperative-statement
END-IF
```

True when exactly one of the conditions is true, false when both are true or both are false.

---

## Boolean Shifting Operators

Logical shift operations on boolean (bit-string) data items.

| Operator | Description |
|----------|-------------|
| `B-SHIFT-L` | Shift bits left |
| `B-SHIFT-R` | Shift bits right |
| `B-SHIFT-LC` | Circular shift left |
| `B-SHIFT-RC` | Circular shift right |

These operate on USAGE BIT data items and return boolean results.

---

## CONTINUE with Duration

Pauses program execution for a specified number of seconds.

```cobol
CONTINUE AFTER { identifier | arithmetic-expression } SECONDS
```

```cobol
CONTINUE AFTER 5 SECONDS
CONTINUE AFTER WS-WAIT-TIME SECONDS
```

This was added to provide a standard sleep/delay mechanism. Previously, programmers used vendor-specific routines (e.g., `CALL "C$SLEEP"`).

See [CONTINUE](control-flow/continue.md) for the full statement documentation.

---

## INSPECT IN REVERSE

The INSPECT statement can now scan in reverse direction.

```cobol
INSPECT WS-DATA IN REVERSE TALLYING WS-COUNT
    FOR TRAILING SPACES
```

---

## Other COBOL 2023 Changes

- **Unsigned packed-decimal** via the NO SIGN phrase
- **63-character identifiers** (increased from 30)
- **USER-DEFINED PICTURE editing** via the EDITING phrase
- **Dynamic-length elementary items** via SET LENGTH
- **Alternate key suppression** on indexed files (SUPPRESS WHEN phrase)

---

## See Also

- [Standards](../standards.md) -- complete standards history
- [CONTINUE](control-flow/continue.md) -- CONTINUE statement (including timed wait)
- [DELETE](io/delete.md) -- record deletion (distinct from DELETE FILE)
- [Conditions](../language/conditions.md) -- conditional expressions
