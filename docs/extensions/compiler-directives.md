# Compiler Directives

Compiler directives control conditional compilation, source format selection, and compiler behavior. The COBOL standard defines a set of directives beginning with `>>`, while vendors provide additional directive mechanisms.

- **Standard:** COBOL 2002, COBOL 2014, COBOL 2023 (standard directives). Vendor directives are implementation-defined.
- **Category:** Extension -- Compiler Control

---

## Syntax

### Standard Directives

Standard compiler directives begin with `>>` in Area A or Area B (the `>>` must be the first non-space characters on the line, optionally preceded by spaces). They are processed before compilation and do not generate executable code.

```cobol
>>DEFINE directive-name AS literal
>>IF condition
>>ELSE
>>END-IF
>>EVALUATE directive-expression
>>WHEN value
>>END-EVALUATE
>>SOURCE FORMAT IS format-type
>>TURN exception-name CHECKING ON|OFF
>>DISPLAY text
```

### Vendor Directive Syntax

```cobol
$SET directive           *> Micro Focus
CBL option-list          *> IBM (before IDENTIFICATION DIVISION)
PROCESS option-list      *> IBM (before IDENTIFICATION DIVISION)
>>D debugging-line       *> Standard debugging indicator
```

---

## Standard Directives

### >>DEFINE

The `>>DEFINE` directive defines a compile-time constant for use in conditional compilation:

```cobol
>>DEFINE USE-DB2 AS 1
>>DEFINE PLATFORM AS "ZOS"
>>DEFINE MAX-RECORDS AS 10000
```

A defined name may be tested with `>>IF` and related directives. Redefinition of an already-defined name replaces the previous value.

To undefine a name:

```cobol
>>DEFINE USE-DB2 OFF
```

### >>IF / >>ELSE / >>END-IF

The `>>IF` directive conditionally includes or excludes source lines based on compile-time conditions:

```cobol
>>IF USE-DB2 IS DEFINED
    EXEC SQL INCLUDE SQLCA END-EXEC
>>ELSE
    01  WS-STATUS  PIC X(2).
>>END-IF
```

Supported conditions:

| Condition | Meaning |
|-----------|---------|
| `name IS DEFINED` | True if `name` has been defined with `>>DEFINE` |
| `name IS NOT DEFINED` | True if `name` has not been defined |
| `name = literal` | True if the value of `name` equals the literal |
| `name > literal` | True if the value of `name` is greater than the literal |
| `name < literal` | True if the value of `name` is less than the literal |
| `name >= literal` | True if the value of `name` is greater than or equal to the literal |
| `name <= literal` | True if the value of `name` is less than or equal to the literal |

Conditions may be combined with `AND` and `OR`:

```cobol
>>IF PLATFORM = "ZOS" AND USE-DB2 IS DEFINED
    EXEC SQL INCLUDE SQLCA END-EXEC
>>END-IF
```

### >>EVALUATE / >>WHEN / >>END-EVALUATE

The `>>EVALUATE` directive provides multi-way conditional compilation:

```cobol
>>EVALUATE PLATFORM
>>WHEN "ZOS"
    EXEC SQL INCLUDE SQLCA END-EXEC
>>WHEN "UNIX"
    COPY "sqlca-unix.cpy".
>>WHEN "WINDOWS"
    COPY "sqlca-win.cpy".
>>WHEN OTHER
    COPY "sqlca-generic.cpy".
>>END-EVALUATE
```

!!! note "COBOL 2002"
    The `>>EVALUATE` directive was introduced in COBOL 2002. Not all compilers support it. Use `>>IF` / `>>ELSE` / `>>END-IF` chains for broader compatibility.

### >>SOURCE FORMAT

The `>>SOURCE FORMAT` directive selects the source format for subsequent lines:

```cobol
>>SOURCE FORMAT IS FIXED
>>SOURCE FORMAT IS FREE
```

| Format | Description |
|--------|-------------|
| `FIXED` | Traditional fixed-format: columns 1-6 are the sequence area, column 7 is the indicator area, columns 8-11 are Area A, columns 12-72 are Area B. |
| `FREE` | Free-format: no column restrictions. Statements may begin in any column. Line length is implementation-defined (typically up to 255 characters). |

!!! note "COBOL 2002"
    The `>>SOURCE FORMAT` directive was standardized in COBOL 2002. It allows a program to switch between fixed and free format within a single source file.

### >>TURN

The `>>TURN` directive enables or disables compile-time checking for specific exception conditions:

```cobol
>>TURN EC-BOUND-SUBSCRIPT CHECKING ON
>>TURN EC-SIZE-OVERFLOW CHECKING OFF
>>TURN ALL CHECKING ON
```

Exception condition names follow the `EC-` prefix convention defined by the COBOL standard (e.g., `EC-BOUND-SUBSCRIPT`, `EC-SIZE-OVERFLOW`, `EC-DATA-INCOMPATIBLE`).

When checking is enabled, the runtime raises the specified exception condition when the error occurs. When checking is disabled, the behavior for that condition is undefined.

!!! note "COBOL 2002"
    The `>>TURN` directive and the exception condition model were introduced in COBOL 2002. Support varies by compiler.

### >>DISPLAY

The `>>DISPLAY` directive outputs a message during compilation:

```cobol
>>DISPLAY "Compiling with DB2 support enabled"
```

This is useful for confirming which conditional compilation paths are active.

---

## Vendor-Specific Directives

### IBM Enterprise COBOL -- CBL and PROCESS

The `CBL` (or `PROCESS`) statement specifies compiler options and must appear before the `IDENTIFICATION DIVISION` header:

```cobol
       CBL RENT,APOST,MAP,XREF,OFFSET
       IDENTIFICATION DIVISION.
       PROGRAM-ID. MYPROGRAM.
```

Common IBM compiler options:

| Option | Description |
|--------|-------------|
| `RENT` | Generate reentrant code |
| `APOST` | Use apostrophe as literal delimiter |
| `QUOTE` | Use quotation mark as literal delimiter |
| `MAP` | Produce a Data Division map |
| `XREF` | Produce a cross-reference listing |
| `OFFSET` | Produce a condensed procedure listing |
| `OPTIMIZE` | Enable optimization |
| `SQL("DB2")` | Specify the SQL precompiler to use |
| `CICS` | Enable CICS command translation |
| `DBCS` | Enable double-byte character set support |

!!! warning "CBL Placement"
    The `CBL` or `PROCESS` statement must appear on a line before the `IDENTIFICATION DIVISION`. If placed elsewhere, the compiler ignores it or produces an error.

### Micro Focus -- $SET

The `$SET` directive controls Micro Focus compiler behavior and may appear anywhere in the source:

```cobol
$SET SOURCEFORMAT"FREE"
$SET DIALECT"MF"
$SET ILUSING"System"
$SET ASSIGN"EXTERNAL"
$SET COMP5"2"
```

Common `$SET` directives:

| Directive | Description |
|-----------|-------------|
| `SOURCEFORMAT"FREE"` | Switch to free-format source |
| `SOURCEFORMAT"FIXED"` | Switch to fixed-format source |
| `DIALECT"MF"` | Use Micro Focus dialect rules |
| `DIALECT"ENTCOBOL"` | Emulate IBM Enterprise COBOL behavior |
| `ILUSING"namespace"` | Import a .NET namespace (managed COBOL) |
| `ASSIGN"EXTERNAL"` | External file assignment semantics |
| `COMP5"2"` | COMP-5 storage behavior option |

### GnuCOBOL Directives

GnuCOBOL supports standard `>>` directives and additional configuration through the command line and configuration files:

```cobol
>>SOURCE FORMAT IS FREE
>>DEFINE GC-VERSION AS 3
>>IF GC-VERSION >= 3
    *> GnuCOBOL 3.x specific code
>>END-IF
```

GnuCOBOL compiler flags that affect source processing:

| Flag | Description |
|------|-------------|
| `-free` | Compile in free format |
| `-fixed` | Compile in fixed format |
| `-std=cobol2014` | Enforce COBOL 2014 standard |
| `-std=ibm` | Emulate IBM COBOL dialect |
| `-std=mf` | Emulate Micro Focus dialect |
| `-D name=value` | Define a compile-time constant |

---

## Conditional Compilation Patterns

### Platform-Specific Code

```cobol
>>DEFINE PLATFORM AS "ZOS"

>>IF PLATFORM = "ZOS"
       DISPLAY WS-MESSAGE UPON SYSOUT
>>ELSE
       DISPLAY WS-MESSAGE
>>END-IF
```

### Feature Toggles

```cobol
>>DEFINE USE-DB2 AS 1
>>DEFINE USE-LOGGING AS 1

       WORKING-STORAGE SECTION.
>>IF USE-DB2 IS DEFINED
           EXEC SQL INCLUDE SQLCA END-EXEC
>>END-IF

>>IF USE-LOGGING IS DEFINED
       01  WS-LOG-FILE-STATUS  PIC X(2).
>>END-IF
```

### Debug / Release Builds

```cobol
>>DEFINE DEBUG-MODE AS 1

       PROCEDURE DIVISION.
       PROCESS-RECORD.
>>IF DEBUG-MODE IS DEFINED
           DISPLAY "DEBUG: Processing record " WS-RECORD-ID
>>END-IF
           PERFORM VALIDATE-RECORD
           PERFORM UPDATE-DATABASE.
```

### Compiler Detection

```cobol
>>IF GCVERSION IS DEFINED
    *> GnuCOBOL-specific code
>>END-IF

>>IF P-IBM IS DEFINED
    *> IBM Enterprise COBOL-specific code
>>END-IF
```

!!! note "Predefined Names"
    Some compilers predefine names that can be tested with `>>IF`. GnuCOBOL defines `GCVERSION`. IBM predefines names based on compiler level. Consult the compiler documentation for available predefined names.

---

## Standard Debugging Lines

Before the `>>DEFINE` / `>>IF` mechanism, the COBOL standard provided a simpler conditional compilation mechanism using the indicator column:

```cobol
       D    DISPLAY "Debug: WS-COUNTER = " WS-COUNTER
```

A `D` in column 7 (fixed format) marks the line as a debugging line. These lines are compiled only when the `WITH DEBUGGING MODE` clause is specified in the SOURCE-COMPUTER paragraph:

```cobol
       SOURCE-COMPUTER. IBM-370 WITH DEBUGGING MODE.
```

When `WITH DEBUGGING MODE` is absent, debugging lines are treated as comments.

!!! warning "Obsolete Feature"
    The `WITH DEBUGGING MODE` clause and the `D` indicator were made obsolete in COBOL-85 and deleted in COBOL 2002. Use `>>IF` directives for conditional compilation in new programs.

---

## Examples

### Multi-Platform Source File

```cobol
>>SOURCE FORMAT IS FREE
>>DEFINE PLATFORM AS "ZOS"

IDENTIFICATION DIVISION.
PROGRAM-ID. MULTIPLATFORM.

DATA DIVISION.
WORKING-STORAGE SECTION.
>>IF PLATFORM = "ZOS"
    EXEC SQL INCLUDE SQLCA END-EXEC
>>END-IF
01  WS-MESSAGE  PIC X(80).

PROCEDURE DIVISION.
MAIN-PARA.
    MOVE "Program started" TO WS-MESSAGE
>>IF PLATFORM = "ZOS"
    DISPLAY WS-MESSAGE UPON SYSOUT
>>ELSE
    DISPLAY WS-MESSAGE
>>END-IF

>>IF PLATFORM = "ZOS"
    EXEC SQL
        SELECT CURRENT TIMESTAMP
        INTO :WS-MESSAGE
        FROM SYSIBM.SYSDUMMY1
    END-EXEC
>>END-IF

    STOP RUN.
```

### IBM CBL with Compiler Options

```cobol
       CBL RENT,APOST,MAP,XREF,SQL("DB2"),CICS
       IDENTIFICATION DIVISION.
       PROGRAM-ID. IBMPROG.

       ENVIRONMENT DIVISION.
       CONFIGURATION SECTION.
       SOURCE-COMPUTER. IBM-370.

       DATA DIVISION.
       WORKING-STORAGE SECTION.
           EXEC SQL INCLUDE SQLCA END-EXEC.
       01  WS-DATA  PIC X(100).

       PROCEDURE DIVISION.
       MAIN-PARA.
           DISPLAY "IBM program started"
           STOP RUN.
```

### Micro Focus $SET Directives

```cobol
$SET SOURCEFORMAT"FREE"
$SET DIALECT"MF"

IDENTIFICATION DIVISION.
PROGRAM-ID. MFPROG.

DATA DIVISION.
WORKING-STORAGE SECTION.
01  WS-COUNTER  PIC 9(4) VALUE 0.

PROCEDURE DIVISION.
MAIN-PARA.
    ADD 1 TO WS-COUNTER
    DISPLAY "Counter: " WS-COUNTER
    STOP RUN.
```

---

## See Also

- [Vendor Extensions](vendor-extensions.md) -- vendor-specific compiler features and extensions
- [Embedded SQL](embedded-sql.md) -- SQL preprocessing interacts with compiler directives
- [CALL Statement](../procedure-division/program-linkage/call.md) -- generated code from directives often produces CALL statements
- [PERFORM Statement](../procedure-division/control-flow/perform.md) -- conditional compilation within procedure code
