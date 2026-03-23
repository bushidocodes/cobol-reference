# User-Defined Functions

User-defined functions allow programmers to create their own functions that are invoked with the `FUNCTION` keyword, just like intrinsic functions. They encapsulate reusable logic and return a single value at the point of reference.

- **Standard:** COBOL 2002, COBOL 2014

!!! note "Compiler Support"
    User-defined functions require a COBOL 2002-compliant compiler. Support varies
    across vendors. GnuCOBOL 3.x, Micro Focus Visual COBOL, and IBM Enterprise
    COBOL 6+ support user-defined functions with varying degrees of conformance.

---

## Defining a Function

A user-defined function is a separately compiled function (or nested function) identified with the `FUNCTION-ID` paragraph. It receives arguments via the `USING` phrase and returns a value via the `RETURNING` phrase.

### Syntax

```cobol
IDENTIFICATION DIVISION.
FUNCTION-ID. function-name.

DATA DIVISION.
WORKING-STORAGE SECTION.
    [local data items]

LINKAGE SECTION.
    [parameter descriptions]
    [return value description]

PROCEDURE DIVISION USING argument-1 [ argument-2 ] ...
                   RETURNING return-item.
    [function logic]
    GOBACK.
END FUNCTION function-name.
```

### Rules

- The `FUNCTION-ID` paragraph identifies the compilation unit as a function rather than a program.
- The function is terminated with `END FUNCTION`, not `END PROGRAM`.
- Parameters in the `USING` phrase are passed **by reference** by default. Use `BY VALUE` or `BY CONTENT` as needed.
- The `RETURNING` phrase specifies a single data item that holds the function's return value.
- The function must terminate with `GOBACK` or `EXIT FUNCTION`, not `STOP RUN` (which would terminate the entire run unit).
- The return item must be defined in the `LINKAGE SECTION`.

---

## Registering a Function

Before a user-defined function can be invoked with `FUNCTION function-name(...)`, it must be declared in the `REPOSITORY` paragraph of the calling program's `CONFIGURATION SECTION`.

```cobol
ENVIRONMENT DIVISION.
CONFIGURATION SECTION.
REPOSITORY.
    FUNCTION function-name
    FUNCTION ALL INTRINSIC.
```

The `REPOSITORY` paragraph tells the compiler that `function-name` refers to a user-defined function rather than a data name. The optional `FUNCTION ALL INTRINSIC` directive allows all intrinsic functions to be called without the `FUNCTION` keyword.

---

## Invoking a Function

Once registered, a user-defined function is invoked the same way as an intrinsic function:

```cobol
FUNCTION function-name ( argument-1 [ argument-2 ] ... )
```

The function reference returns a single value and may appear anywhere a literal or identifier of the corresponding category is permitted.

```cobol
MOVE FUNCTION my-format(WS-DATE) TO WS-OUTPUT
COMPUTE WS-TAX = FUNCTION calc-tax(WS-AMOUNT WS-RATE)
IF FUNCTION is-valid(WS-INPUT) = "Y"
    PERFORM PROCESS-INPUT
END-IF
```

---

## Examples

### Calculating Sales Tax

```cobol
      *> ============================================
      *> Function: calc-sales-tax
      *> Returns the tax amount for a given subtotal
      *> and tax rate.
      *> ============================================
       IDENTIFICATION DIVISION.
       FUNCTION-ID. calc-sales-tax.

       DATA DIVISION.
       LINKAGE SECTION.
       01  LS-SUBTOTAL     PIC 9(7)V99.
       01  LS-TAX-RATE     PIC 9V9(4).
       01  LS-RESULT       PIC 9(7)V99.

       PROCEDURE DIVISION USING LS-SUBTOTAL LS-TAX-RATE
                          RETURNING LS-RESULT.
           COMPUTE LS-RESULT ROUNDED =
               LS-SUBTOTAL * LS-TAX-RATE
           GOBACK.
       END FUNCTION calc-sales-tax.
```

**Calling program:**

```cobol
       IDENTIFICATION DIVISION.
       PROGRAM-ID. invoice-program.

       ENVIRONMENT DIVISION.
       CONFIGURATION SECTION.
       REPOSITORY.
           FUNCTION calc-sales-tax
           FUNCTION ALL INTRINSIC.

       DATA DIVISION.
       WORKING-STORAGE SECTION.
       01  WS-SUBTOTAL     PIC 9(7)V99 VALUE 250.00.
       01  WS-TAX-RATE     PIC 9V9(4)  VALUE 0.0825.
       01  WS-TAX          PIC 9(7)V99.
       01  WS-TOTAL        PIC 9(7)V99.

       PROCEDURE DIVISION.
           COMPUTE WS-TAX =
               FUNCTION calc-sales-tax(
                   WS-SUBTOTAL WS-TAX-RATE)
           COMPUTE WS-TOTAL = WS-SUBTOTAL + WS-TAX
           DISPLAY "Subtotal: " WS-SUBTOTAL
           DISPLAY "Tax:      " WS-TAX
           DISPLAY "Total:    " WS-TOTAL
           STOP RUN.
       END PROGRAM invoice-program.
```

### String Padding Function

```cobol
      *> ============================================
      *> Function: pad-right
      *> Returns the input string padded on the right
      *> to a specified length with a fill character.
      *> ============================================
       IDENTIFICATION DIVISION.
       FUNCTION-ID. pad-right.

       DATA DIVISION.
       WORKING-STORAGE SECTION.
       01  WS-IDX          PIC 9(4).
       01  WS-INPUT-LEN    PIC 9(4).

       LINKAGE SECTION.
       01  LS-INPUT        PIC X(100).
       01  LS-PAD-LEN      PIC 9(4).
       01  LS-FILL-CHAR    PIC X(1).
       01  LS-RESULT       PIC X(100).

       PROCEDURE DIVISION USING LS-INPUT LS-PAD-LEN
                                LS-FILL-CHAR
                          RETURNING LS-RESULT.
           MOVE SPACES TO LS-RESULT
           MOVE LS-INPUT TO LS-RESULT

           COMPUTE WS-INPUT-LEN =
               FUNCTION LENGTH(FUNCTION TRIM(LS-INPUT))

           IF WS-INPUT-LEN < LS-PAD-LEN
               PERFORM VARYING WS-IDX
                   FROM WS-INPUT-LEN + 1 BY 1
                   UNTIL WS-IDX > LS-PAD-LEN
                   MOVE LS-FILL-CHAR TO
                       LS-RESULT(WS-IDX:1)
               END-PERFORM
           END-IF

           GOBACK.
       END FUNCTION pad-right.
```

### Boolean Validation Function

```cobol
      *> ============================================
      *> Function: is-valid-date
      *> Returns "Y" if the input is a valid date
      *> in YYYYMMDD format, "N" otherwise.
      *> ============================================
       IDENTIFICATION DIVISION.
       FUNCTION-ID. is-valid-date.

       DATA DIVISION.
       WORKING-STORAGE SECTION.
       01  WS-YEAR         PIC 9(4).
       01  WS-MONTH        PIC 9(2).
       01  WS-DAY          PIC 9(2).
       01  WS-MAX-DAY      PIC 9(2).
       01  WS-LEAP         PIC 9 VALUE 0.

       LINKAGE SECTION.
       01  LS-DATE-IN      PIC 9(8).
       01  LS-RESULT       PIC X(1).

       PROCEDURE DIVISION USING LS-DATE-IN
                          RETURNING LS-RESULT.
           MOVE "N" TO LS-RESULT

           IF LS-DATE-IN NOT NUMERIC
               GOBACK
           END-IF

           COMPUTE WS-YEAR  = LS-DATE-IN / 10000
           COMPUTE WS-MONTH =
               FUNCTION MOD(LS-DATE-IN / 100  100)
           COMPUTE WS-DAY   =
               FUNCTION MOD(LS-DATE-IN  100)

           IF WS-MONTH < 1 OR WS-MONTH > 12
               GOBACK
           END-IF

           IF WS-DAY < 1
               GOBACK
           END-IF

      *>   Determine leap year
           MOVE 0 TO WS-LEAP
           IF FUNCTION MOD(WS-YEAR 4) = 0
               MOVE 1 TO WS-LEAP
               IF FUNCTION MOD(WS-YEAR 100) = 0
                   MOVE 0 TO WS-LEAP
                   IF FUNCTION MOD(WS-YEAR 400) = 0
                       MOVE 1 TO WS-LEAP
                   END-IF
               END-IF
           END-IF

      *>   Determine max days in month
           EVALUATE WS-MONTH
               WHEN 1  MOVE 31 TO WS-MAX-DAY
               WHEN 2
                   IF WS-LEAP = 1
                       MOVE 29 TO WS-MAX-DAY
                   ELSE
                       MOVE 28 TO WS-MAX-DAY
                   END-IF
               WHEN 3  MOVE 31 TO WS-MAX-DAY
               WHEN 4  MOVE 30 TO WS-MAX-DAY
               WHEN 5  MOVE 31 TO WS-MAX-DAY
               WHEN 6  MOVE 30 TO WS-MAX-DAY
               WHEN 7  MOVE 31 TO WS-MAX-DAY
               WHEN 8  MOVE 31 TO WS-MAX-DAY
               WHEN 9  MOVE 30 TO WS-MAX-DAY
               WHEN 10 MOVE 31 TO WS-MAX-DAY
               WHEN 11 MOVE 30 TO WS-MAX-DAY
               WHEN 12 MOVE 31 TO WS-MAX-DAY
           END-EVALUATE

           IF WS-DAY <= WS-MAX-DAY
               MOVE "Y" TO LS-RESULT
           END-IF

           GOBACK.
       END FUNCTION is-valid-date.
```

**Calling program:**

```cobol
       ENVIRONMENT DIVISION.
       CONFIGURATION SECTION.
       REPOSITORY.
           FUNCTION is-valid-date.

       DATA DIVISION.
       WORKING-STORAGE SECTION.
       01  WS-INPUT-DATE   PIC 9(8).

       PROCEDURE DIVISION.
           MOVE 20250229 TO WS-INPUT-DATE
           IF FUNCTION is-valid-date(WS-INPUT-DATE) = "Y"
               DISPLAY WS-INPUT-DATE " is valid"
           ELSE
               DISPLAY WS-INPUT-DATE " is invalid"
           END-IF
           *> "20250229 is invalid" (2025 is not a leap year)

           MOVE 20240229 TO WS-INPUT-DATE
           IF FUNCTION is-valid-date(WS-INPUT-DATE) = "Y"
               DISPLAY WS-INPUT-DATE " is valid"
           ELSE
               DISPLAY WS-INPUT-DATE " is invalid"
           END-IF
           *> "20240229 is valid" (2024 is a leap year)

           STOP RUN.
```

---

## Nested Function Definitions

A user-defined function may be defined as a nested program within the calling program. This avoids the need for separate compilation.

```cobol
       IDENTIFICATION DIVISION.
       PROGRAM-ID. main-program.

       ENVIRONMENT DIVISION.
       CONFIGURATION SECTION.
       REPOSITORY.
           FUNCTION double-it.

       DATA DIVISION.
       WORKING-STORAGE SECTION.
       01  WS-VALUE        PIC 9(5) VALUE 42.
       01  WS-RESULT       PIC 9(5).

       PROCEDURE DIVISION.
           COMPUTE WS-RESULT =
               FUNCTION double-it(WS-VALUE)
           DISPLAY "Result: " WS-RESULT
           *> Result: 00084
           STOP RUN.

      *> --- Nested function definition ---
       IDENTIFICATION DIVISION.
       FUNCTION-ID. double-it.

       DATA DIVISION.
       LINKAGE SECTION.
       01  LS-INPUT        PIC 9(5).
       01  LS-RESULT       PIC 9(5).

       PROCEDURE DIVISION USING LS-INPUT
                          RETURNING LS-RESULT.
           COMPUTE LS-RESULT = LS-INPUT * 2
           GOBACK.
       END FUNCTION double-it.

       END PROGRAM main-program.
```

---

## User-Defined vs Intrinsic Functions

| Aspect | Intrinsic Functions | User-Defined Functions |
|--------|-------------------|----------------------|
| Definition | Built into the compiler | Written by the programmer |
| Registration | None required (or `FUNCTION ALL INTRINSIC`) | Must be declared in `REPOSITORY` |
| Invocation | `FUNCTION name(args)` | `FUNCTION name(args)` |
| Return value | Defined by the standard | Defined by `RETURNING` phrase |
| Arguments | Defined by the standard | Defined by `USING` phrase |
| Standard | COBOL-85 Amendment 1 | COBOL 2002 |

---

## Best Practices

1. **Use meaningful names** that describe what the function returns, not what it does internally:
   ```cobol
   *> Good: describes the return value
   FUNCTION-ID. sales-tax.
   FUNCTION-ID. is-valid-date.

   *> Avoid: describes an action, not a value
   FUNCTION-ID. calculate-tax.
   ```

2. **Keep functions pure** when possible -- avoid modifying external state or performing I/O. Functions that only compute a return value from their arguments are easier to test and reuse.

3. **Always use `GOBACK` or `EXIT FUNCTION`** instead of `STOP RUN` to return from a function. `STOP RUN` terminates the entire run unit, not just the function.

4. **Document parameter expectations** in comments, especially valid ranges and formats:
   ```cobol
   *> LS-RATE: annual interest rate as a decimal
   *>          (e.g., 0.05 for 5%)
   *> LS-PERIODS: number of monthly periods (1-360)
   ```

5. **Validate arguments** within the function when invalid input could cause abends or incorrect results.

---

## See Also

- [Intrinsic Functions](index.md) -- overview of built-in intrinsic functions
- [CALL](../procedure-division/program-linkage/call.md) -- calling subprograms
- [CANCEL](../procedure-division/program-linkage/cancel.md) -- releasing subprogram resources
