# Embedded SQL

Embedded SQL provides a mechanism for COBOL programs to issue SQL statements against a relational database, using host variables to exchange data between COBOL data items and SQL columns.

- **Standard:** Not part of the COBOL standard. Defined by the SQL standard (ISO/IEC 9075, Part 2 and Part 10) and implemented by vendor-specific precompilers.
- **Category:** Extension -- Database Access

---

## Syntax

### General Form

All embedded SQL statements are delimited by `EXEC SQL` and `END-EXEC`:

```cobol
EXEC SQL
    sql-statement
END-EXEC
```

The precompiler replaces each `EXEC SQL ... END-EXEC` block with generated COBOL code (typically `CALL` statements to a database runtime library) before the COBOL compiler processes the source.

### Host Variables

COBOL data items referenced within SQL statements are prefixed with a colon (`:`) to distinguish them from SQL column names:

```cobol
EXEC SQL
    SELECT CUST-NAME, CUST-BALANCE
    INTO :WS-CUST-NAME, :WS-CUST-BALANCE
    FROM CUSTOMER
    WHERE CUST-ID = :WS-CUST-ID
END-EXEC
```

Outside of `EXEC SQL` blocks, host variables are referenced without the colon, following normal COBOL syntax.

### INCLUDE SQLCA

The SQL Communication Area is included in the WORKING-STORAGE SECTION:

```cobol
WORKING-STORAGE SECTION.
    EXEC SQL INCLUDE SQLCA END-EXEC
```

This expands to a group item named `SQLCA` containing diagnostic fields. Programs may also code the SQLCA manually rather than using `INCLUDE`.

---

## Rules

### Host Variable Declaration

Host variables must be declared in the WORKING-STORAGE SECTION or LINKAGE SECTION. Some precompilers require that host variables appear within an explicit `BEGIN DECLARE SECTION` / `END DECLARE SECTION` block:

```cobol
EXEC SQL BEGIN DECLARE SECTION END-EXEC
01  WS-CUST-ID       PIC X(10).
01  WS-CUST-NAME     PIC X(40).
01  WS-CUST-BALANCE  PIC S9(9)V99 COMP-3.
EXEC SQL END DECLARE SECTION END-EXEC
```

Other precompilers (notably IBM DB2 for z/OS) allow any COBOL data item in WORKING-STORAGE to serve as a host variable without a declare section.

### Host Variable Type Mapping

The precompiler maps COBOL data types to SQL data types:

| COBOL PICTURE / USAGE | SQL Type |
|------------------------|----------|
| `PIC X(n)` | `CHAR(n)` or `VARCHAR(n)` |
| `PIC S9(n) COMP` or `BINARY` | `SMALLINT` (1-4 digits), `INTEGER` (5-9), `BIGINT` (10-18) |
| `PIC S9(n)V9(m) COMP-3` | `DECIMAL(n+m, m)` |
| `COMP-1` | `REAL` |
| `COMP-2` | `DOUBLE` |

!!! note "Vendor Differences"
    The exact type mappings vary by precompiler. Consult the precompiler documentation for the specific database product in use.

### SQLCODE and SQLSTATE

After each SQL statement executes, the database sets diagnostic values:

| Field | Type | Meaning |
|-------|------|---------|
| `SQLCODE` | Numeric | `0` = success, positive = warning or informational, negative = error. `+100` indicates no rows found. |
| `SQLSTATE` | `PIC X(5)` | Five-character status code. `'00000'` = success. `'02000'` = no data. Class `'01'` = warning. Any other value beginning with a digit other than `0` indicates an error. |

`SQLCODE` is available through the SQLCA. `SQLSTATE` is available in the SQLCA (as `SQLCAID` subfield) or through a standalone host variable, depending on the precompiler.

### Indicator Variables

Indicator variables detect and set SQL NULL values. An indicator variable is a `PIC S9(4) COMP` item associated with a host variable:

```cobol
01  WS-CUST-BALANCE    PIC S9(9)V99 COMP-3.
01  WS-BALANCE-IND     PIC S9(4) COMP.
```

```cobol
EXEC SQL
    SELECT CUST_BALANCE
    INTO :WS-CUST-BALANCE :WS-BALANCE-IND
    FROM CUSTOMER
    WHERE CUST_ID = :WS-CUST-ID
END-EXEC
```

| Indicator Value | Meaning |
|-----------------|---------|
| `0` | Column value is not NULL; host variable contains the value |
| `-1` | Column value is NULL; host variable content is undefined |
| Positive | Column value was truncated; indicator contains original length |

To insert or update a NULL value, the program sets the indicator variable to `-1` before executing the SQL statement.

---

## WHENEVER Directive

The `WHENEVER` directive establishes automatic error-handling behavior for subsequent SQL statements:

```cobol
EXEC SQL WHENEVER condition action END-EXEC
```

### Conditions

| Condition | Triggers When |
|-----------|---------------|
| `SQLERROR` | `SQLCODE` is negative |
| `SQLWARNING` | `SQLCODE` is positive (but not `+100`) |
| `NOT FOUND` | `SQLCODE` is `+100` (no rows returned or affected) |

### Actions

| Action | Behavior |
|--------|----------|
| `CONTINUE` | No automatic action; the program continues to the next statement (default) |
| `GO TO paragraph-name` | Control transfers to the specified paragraph |
| `STOP` | The program terminates (supported by some precompilers) |

!!! warning "WHENEVER Scope"
    The `WHENEVER` directive is a precompiler directive, not a runtime statement. It applies to all subsequent `EXEC SQL` statements in the source file until overridden by another `WHENEVER` for the same condition. This positional scope can cause unintended behavior if the directive is placed carelessly.

---

## Cursor Processing

Cursors allow a program to process a multi-row result set one row at a time.

### DECLARE CURSOR

```cobol
EXEC SQL
    DECLARE CUST-CURSOR CURSOR FOR
    SELECT CUST_ID, CUST_NAME, CUST_BALANCE
    FROM CUSTOMER
    WHERE CUST_REGION = :WS-REGION
    ORDER BY CUST_NAME
END-EXEC
```

The `DECLARE CURSOR` statement defines the cursor and its associated query. It is a declarative statement and does not execute the query.

### OPEN

```cobol
EXEC SQL OPEN CUST-CURSOR END-EXEC
```

The `OPEN` statement executes the query and positions the cursor before the first row.

### FETCH

```cobol
EXEC SQL
    FETCH CUST-CURSOR
    INTO :WS-CUST-ID, :WS-CUST-NAME, :WS-CUST-BALANCE
END-EXEC
```

Each `FETCH` retrieves the next row and advances the cursor. When no more rows remain, `SQLCODE` is set to `+100`.

### CLOSE

```cobol
EXEC SQL CLOSE CUST-CURSOR END-EXEC
```

The `CLOSE` statement releases the cursor and its associated resources.

---

## Static and Dynamic SQL

### Static SQL

In static SQL, the complete SQL statement text is known at precompile time. The precompiler validates the statement against the database catalog, determines access paths, and generates a fixed execution plan (a "package" or "access module").

```cobol
EXEC SQL
    UPDATE CUSTOMER
    SET CUST_BALANCE = CUST_BALANCE + :WS-PAYMENT
    WHERE CUST_ID = :WS-CUST-ID
END-EXEC
```

### Dynamic SQL

Dynamic SQL constructs and executes SQL statements at runtime. It is used when the SQL text is not known until execution.

#### PREPARE and EXECUTE

```cobol
01  WS-SQL-TEXT  PIC X(500).

MOVE "UPDATE CUSTOMER SET CUST_BALANCE = CUST_BALANCE + ?"
     " WHERE CUST_ID = ?"
  TO WS-SQL-TEXT

EXEC SQL
    PREPARE STMT1 FROM :WS-SQL-TEXT
END-EXEC

EXEC SQL
    EXECUTE STMT1 USING :WS-PAYMENT, :WS-CUST-ID
END-EXEC
```

The `PREPARE` statement parses and optimizes the SQL text at runtime. Parameter markers (`?`) serve as placeholders for host variables supplied on `EXECUTE`.

#### EXECUTE IMMEDIATE

For statements executed only once, `EXECUTE IMMEDIATE` combines preparation and execution:

```cobol
MOVE "DELETE FROM TEMP_TABLE" TO WS-SQL-TEXT
EXEC SQL EXECUTE IMMEDIATE :WS-SQL-TEXT END-EXEC
```

---

## Common Precompilers

| Precompiler | Database | Notes |
|-------------|----------|-------|
| IBM DB2 Precompiler | DB2 for z/OS, DB2 for LUW | Uses DBRM (database request module) and packages. Supports `EXEC SQL INCLUDE`. |
| IBM DB2 Coprocessor | DB2 for z/OS | Integrated into the Enterprise COBOL compiler; processes SQL inline without a separate precompile step. |
| Oracle Pro*COBOL | Oracle Database | Precompiles `.pco` files. Uses Oracle-specific options such as `MODE=ORACLE` vs `MODE=ANSI`. |
| Micro Focus COBSQL | Various (via ODBC/JDBC) | Directive-driven SQL preprocessing integrated with the Micro Focus COBOL compiler. |
| ECPG (PostgreSQL) | PostgreSQL | Open-source embedded SQL precompiler; supports COBOL as a host language. |

---

## Examples

### SELECT INTO (Single-Row Query)

```cobol
       WORKING-STORAGE SECTION.
           EXEC SQL INCLUDE SQLCA END-EXEC.
       01  WS-CUST-ID        PIC X(10).
       01  WS-CUST-NAME      PIC X(40).
       01  WS-CUST-BALANCE   PIC S9(9)V99 COMP-3.
       01  WS-BALANCE-IND    PIC S9(4) COMP.

       PROCEDURE DIVISION.
           MOVE "C00012345" TO WS-CUST-ID

           EXEC SQL
               SELECT CUST_NAME, CUST_BALANCE
               INTO :WS-CUST-NAME,
                    :WS-CUST-BALANCE :WS-BALANCE-IND
               FROM CUSTOMER
               WHERE CUST_ID = :WS-CUST-ID
           END-EXEC

           EVALUATE SQLCODE
               WHEN 0
                   IF WS-BALANCE-IND = -1
                       DISPLAY "Balance is NULL"
                   ELSE
                       DISPLAY "Balance: " WS-CUST-BALANCE
                   END-IF
               WHEN +100
                   DISPLAY "Customer not found"
               WHEN OTHER
                   DISPLAY "SQL error: " SQLCODE
           END-EVALUATE

           STOP RUN.
```

### Cursor Loop

```cobol
       WORKING-STORAGE SECTION.
           EXEC SQL INCLUDE SQLCA END-EXEC.
       01  WS-REGION          PIC X(5).
       01  WS-CUST-ID         PIC X(10).
       01  WS-CUST-NAME       PIC X(40).
       01  WS-CUST-BALANCE    PIC S9(9)V99 COMP-3.
       01  WS-ROW-COUNT       PIC 9(7) VALUE 0.

           EXEC SQL
               DECLARE REGION-CURSOR CURSOR FOR
               SELECT CUST_ID, CUST_NAME, CUST_BALANCE
               FROM CUSTOMER
               WHERE CUST_REGION = :WS-REGION
               ORDER BY CUST_NAME
           END-EXEC.

       PROCEDURE DIVISION.
       MAIN-PARA.
           MOVE "EAST" TO WS-REGION
           EXEC SQL OPEN REGION-CURSOR END-EXEC

           IF SQLCODE NOT = 0
               DISPLAY "Error opening cursor: " SQLCODE
               STOP RUN
           END-IF

           PERFORM FETCH-ROW UNTIL SQLCODE NOT = 0

           EXEC SQL CLOSE REGION-CURSOR END-EXEC
           DISPLAY "Rows processed: " WS-ROW-COUNT
           STOP RUN.

       FETCH-ROW.
           EXEC SQL
               FETCH REGION-CURSOR
               INTO :WS-CUST-ID, :WS-CUST-NAME,
                    :WS-CUST-BALANCE
           END-EXEC

           IF SQLCODE = 0
               ADD 1 TO WS-ROW-COUNT
               DISPLAY WS-CUST-ID " " WS-CUST-NAME
                       " " WS-CUST-BALANCE
           END-IF.
```

### INSERT

```cobol
       EXEC SQL
           INSERT INTO CUSTOMER
               (CUST_ID, CUST_NAME, CUST_BALANCE, CUST_REGION)
           VALUES
               (:WS-CUST-ID, :WS-CUST-NAME,
                :WS-CUST-BALANCE, :WS-REGION)
       END-EXEC

       IF SQLCODE NOT = 0
           DISPLAY "Insert failed: " SQLCODE
       END-IF
```

### UPDATE with Error Handling via WHENEVER

```cobol
           EXEC SQL WHENEVER SQLERROR GO TO SQL-ERROR END-EXEC
           EXEC SQL WHENEVER NOT FOUND CONTINUE END-EXEC

           EXEC SQL
               UPDATE CUSTOMER
               SET CUST_BALANCE = CUST_BALANCE + :WS-PAYMENT
               WHERE CUST_ID = :WS-CUST-ID
           END-EXEC

           IF SQLCODE = +100
               DISPLAY "No customer found for update"
           ELSE
               DISPLAY "Customer updated successfully"
           END-IF

           EXEC SQL WHENEVER SQLERROR CONTINUE END-EXEC
           GO TO MAIN-CONTINUE.

       SQL-ERROR.
           DISPLAY "SQL error occurred: " SQLCODE
           EXEC SQL WHENEVER SQLERROR CONTINUE END-EXEC
           EXEC SQL ROLLBACK END-EXEC
           STOP RUN.
```

### Transaction Control

```cobol
           EXEC SQL
               UPDATE ACCOUNT
               SET BALANCE = BALANCE - :WS-AMOUNT
               WHERE ACCT_ID = :WS-FROM-ACCT
           END-EXEC

           IF SQLCODE NOT = 0
               EXEC SQL ROLLBACK END-EXEC
               DISPLAY "Debit failed"
               STOP RUN
           END-IF

           EXEC SQL
               UPDATE ACCOUNT
               SET BALANCE = BALANCE + :WS-AMOUNT
               WHERE ACCT_ID = :WS-TO-ACCT
           END-EXEC

           IF SQLCODE NOT = 0
               EXEC SQL ROLLBACK END-EXEC
               DISPLAY "Credit failed"
               STOP RUN
           END-IF

           EXEC SQL COMMIT END-EXEC
```

---

## See Also

- [USAGE Clause](../data-division/usage.md) -- host variable storage formats
- [PICTURE Clause](../data-division/picture.md) -- host variable type definitions
- [CALL Statement](../procedure-division/program-linkage/call.md) -- alternative interface for database access via API calls
- [PERFORM Statement](../procedure-division/control-flow/perform.md) -- loop constructs used with cursor processing
- [Vendor Extensions](vendor-extensions.md) -- vendor-specific SQL extensions such as EXEC CICS and EXEC DLI
- [Compiler Directives](compiler-directives.md) -- precompiler and compiler directive interaction
