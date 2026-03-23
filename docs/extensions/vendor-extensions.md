# Vendor Extensions

Vendor extensions are non-standard features provided by COBOL compiler implementations that extend the language beyond the ISO COBOL standard. These extensions address platform-specific requirements, performance optimization, and integration with external systems.

- **Standard:** Not part of the COBOL standard (ISO/IEC 1989). Individual features may be adopted into later standards.
- **Category:** Extension -- Vendor-Specific

---

## IBM Enterprise COBOL

IBM Enterprise COBOL for z/OS is the primary COBOL compiler for IBM mainframe environments. It provides extensions for CICS, IMS, DB2, and z/OS system services.

### EXEC CICS

The `EXEC CICS` interface provides access to CICS (Customer Information Control System) transaction processing services:

```cobol
EXEC CICS SEND TEXT
    FROM(WS-MESSAGE)
    LENGTH(WS-MSG-LEN)
    ERASE
END-EXEC
```

```cobol
EXEC CICS READ
    FILE('CUSTFILE')
    INTO(WS-CUST-RECORD)
    RIDFLD(WS-CUST-ID)
    LENGTH(WS-REC-LEN)
END-EXEC
```

CICS commands follow the `EXEC CICS ... END-EXEC` delimiters and are processed by the CICS translator before COBOL compilation.

### EXEC DLI

The `EXEC DLI` interface accesses IMS (Information Management System) hierarchical databases:

```cobol
EXEC DLI GU
    USING PCB(1)
    SEGMENT(CUSTOMER)
    INTO(WS-CUST-RECORD)
    WHERE(CUSTKEY = WS-CUST-KEY)
END-EXEC
```

### COMP-5 (Native Binary)

IBM's `COMP-5` stores data in native binary format, allowing values to use the full range of the binary storage rather than being limited to the PICTURE-defined digit count. This is essential for interfacing with system APIs and non-COBOL programs.

```cobol
01  WS-RETURN-CODE  PIC S9(9) COMP-5.
```

### XML and JSON Extensions

IBM Enterprise COBOL provides statements for parsing and generating XML and JSON:

```cobol
XML PARSE WS-XML-DOCUMENT
    PROCESSING PROCEDURE IS XML-HANDLER
END-XML
```

```cobol
JSON GENERATE WS-JSON-OUTPUT FROM WS-DATA-RECORD
    COUNT IN WS-JSON-LENGTH
END-JSON
```

```cobol
JSON PARSE WS-JSON-INPUT INTO WS-DATA-RECORD
END-JSON
```

!!! note "IBM Enterprise COBOL V6+"
    The `JSON GENERATE` and `JSON PARSE` statements were introduced in IBM Enterprise COBOL V6.1.

### DISPLAY UPON SYSOUT

IBM extends the `DISPLAY` statement to direct output to specific system destinations:

```cobol
DISPLAY "Processing complete" UPON SYSOUT
DISPLAY "Error detected" UPON SYSERR
DISPLAY WS-DEBUG-INFO UPON CONSOLE
```

### GOBACK

The `GOBACK` statement returns control to the calling program or the operating system, functioning as a logical return:

```cobol
GOBACK.
```

`GOBACK` was an IBM extension that was widely adopted by other vendors and was standardized in COBOL 2002.

### BY VALUE in CALL

IBM supports passing parameters by value in the `CALL` statement, enabling calls to C functions and system APIs:

```cobol
CALL "CEELOCT" USING
    BY REFERENCE WS-LILIAN
    BY REFERENCE WS-SECONDS
    BY REFERENCE WS-GREGORIAN
    BY REFERENCE WS-FEEDBACK
END-CALL
```

---

## Micro Focus COBOL

Micro Focus COBOL (Visual COBOL, Enterprise Developer) targets distributed platforms and provides extensions for modern development environments.

### $SET Directives

Micro Focus uses `$SET` directives for compiler control:

```cobol
$SET SOURCEFORMAT"FREE"
$SET DIALECT"MF"
$SET ILUSING"System"
```

### Managed COBOL (.NET and JVM)

Micro Focus extends COBOL for the .NET Common Language Runtime and the Java Virtual Machine, enabling interoperability with C#, Java, and other managed languages:

```cobol
$SET ILUSING"System"
$SET ILUSING"System.Collections.Generic"

CLASS-ID. CustomerService PUBLIC.

METHOD-ID. GetCustomer.
01 customer-name STRING.
PROCEDURE DIVISION USING BY VALUE customer-id AS BINARY-LONG
                   RETURNING customer-name.
    *> Method implementation
END METHOD.

END CLASS.
```

### SCREEN SECTION Enhancements

Micro Focus provides extensive SCREEN SECTION capabilities for terminal-based applications:

```cobol
SCREEN SECTION.
01  MAIN-SCREEN.
    05  BLANK SCREEN.
    05  LINE 1 COL 20 VALUE "CUSTOMER ENTRY"
        HIGHLIGHT FOREGROUND-COLOR 7.
    05  LINE 3 COL 5 VALUE "ID:".
    05  LINE 3 COL 10 PIC X(10) USING WS-CUST-ID
        HIGHLIGHT AUTO REQUIRED.
    05  LINE 5 COL 5 VALUE "NAME:".
    05  LINE 5 COL 12 PIC X(30) USING WS-CUST-NAME
        HIGHLIGHT.
```

### COMP-X

`COMP-X` stores unsigned binary data in a number of bytes determined by the PICTURE clause character count rather than the digit range:

```cobol
01  WS-BYTE-VALUE   PIC X    COMP-X.   *> 1 byte, 0-255
01  WS-WORD-VALUE   PIC XX   COMP-X.   *> 2 bytes, 0-65535
01  WS-DWORD-VALUE  PIC X(4) COMP-X.   *> 4 bytes, 0-4294967295
```

### Pointer and Method Invocation

Micro Focus supports object-oriented COBOL with method invocation syntax:

```cobol
INVOKE customer-object "GetName"
    RETURNING ws-name
END-INVOKE
```

---

## GnuCOBOL

GnuCOBOL is an open-source COBOL compiler that translates COBOL to C and then compiles with the system C compiler. It supports a wide range of extensions for compatibility with other COBOL dialects.

### SCREEN SECTION

GnuCOBOL provides SCREEN SECTION support for console-based user interfaces:

```cobol
SCREEN SECTION.
01  DATA-ENTRY-SCREEN.
    05  BLANK SCREEN.
    05  LINE 2 COL 10 VALUE "Employee ID: ".
    05  LINE 2 COL 25 PIC X(8) USING WS-EMP-ID
        HIGHLIGHT AUTO.
    05  LINE 4 COL 10 VALUE "Name: ".
    05  LINE 4 COL 25 PIC X(30) USING WS-EMP-NAME
        HIGHLIGHT.
```

### Extended ACCEPT and DISPLAY

GnuCOBOL extends `ACCEPT` and `DISPLAY` with screen positioning:

```cobol
DISPLAY "Enter customer ID: " AT LINE 5 COL 1
ACCEPT WS-CUST-ID AT LINE 5 COL 22
    WITH AUTO FOREGROUND-COLOR 3
```

### CBL_OC_ Library Routines

GnuCOBOL provides C-library-accessible routines:

```cobol
CALL "CBL_OC_GETOPT" USING
    BY REFERENCE WS-OPTION-CHAR
    BY REFERENCE WS-LONG-OPTIONS
    BY REFERENCE WS-LONG-INDEX
    BY REFERENCE WS-OPTION-ARG
    BY VALUE WS-ARGUMENT-COUNT
    BY REFERENCE WS-ARGUMENT-VALUES
END-CALL
```

Other library routines include `CBL_OC_HOSTED`, `CBL_EXIT_PROC`, `CBL_ERROR_PROC`, and standard Micro Focus-compatible library routines such as `CBL_READ_DIR`, `CBL_OPEN_FILE`, and `CBL_CLOSE_FILE`.

### FLOAT-LONG and FLOAT-SHORT

GnuCOBOL provides explicit floating-point types:

```cobol
01  WS-SHORT-FLOAT  USAGE FLOAT-SHORT.   *> 32-bit IEEE 754
01  WS-LONG-FLOAT   USAGE FLOAT-LONG.    *> 64-bit IEEE 754
```

These correspond to C `float` and `double` types, respectively.

### COMP-5 (Native Binary)

GnuCOBOL supports `COMP-5` with the same semantics as IBM and Micro Focus -- values may use the full binary range of the storage allocated:

```cobol
01  WS-NATIVE-INT  PIC S9(9) COMP-5.
```

---

## Fujitsu NetCOBOL

Fujitsu NetCOBOL targets Windows, Linux, and Solaris platforms with extensions for GUI and report development.

### FORMAT Clause

The `FORMAT` clause associates a record with an externally-defined screen or print format:

```cobol
FD  PRINT-FILE
    FORMAT TYPE IS PRINT-FORMAT.
```

### DISPLAY WINDOW

Fujitsu provides a `DISPLAY WINDOW` statement for creating GUI windows in Windows environments:

```cobol
DISPLAY WINDOW UPON CRT
    TITLE "Customer Maintenance"
    SIZE 80 LINES 24
```

---

## Common Extensions Across Vendors

Several extensions originated with specific vendors but are now supported by multiple compilers:

| Extension | IBM | Micro Focus | GnuCOBOL | Fujitsu | Standard |
|-----------|-----|-------------|-----------|---------|----------|
| `COMP-5` (native binary) | Yes | Yes | Yes | No | No |
| `GOBACK` | Yes | Yes | Yes | Yes | COBOL 2002 |
| Extended `ACCEPT`/`DISPLAY` | Partial | Yes | Yes | Yes | No |
| `SCREEN SECTION` | No | Yes | Yes | Yes | COBOL 2002 (limited) |
| `COMP-X` (unsigned binary) | No | Yes | Yes | No | No |
| `COMP-1` (single float) | Yes | Yes | Yes | Yes | No |
| `COMP-2` (double float) | Yes | Yes | Yes | Yes | No |
| `EXEC SQL` | Yes | Yes | Yes (external) | Yes | No (SQL standard) |
| `EXEC CICS` | Yes | Yes (emulated) | No | No | No |
| `BY VALUE` in `CALL` | Yes | Yes | Yes | Yes | COBOL 2002 |
| XML statements | Yes | Yes | Yes (limited) | No | COBOL 2014 (limited) |
| JSON statements | Yes | No | No | No | No |
| Object-oriented COBOL | Yes | Yes | No | No | COBOL 2002 |
| `FLOAT-LONG`/`FLOAT-SHORT` | No | Yes | Yes | No | COBOL 2002 |

---

## Portability Considerations

!!! warning "Portability"
    Programs that rely on vendor extensions cannot be compiled without modification on a different vendor's compiler. The following practices improve portability:

    - Isolate vendor-specific code in separate paragraphs, sections, or copybooks.
    - Use conditional compilation (`>>IF`) to select vendor-specific code paths.
    - Prefer standard COBOL constructs where they provide equivalent functionality.
    - Document all vendor dependencies clearly in program comments.
    - Use `COMP-3` (PACKED-DECIMAL) for business arithmetic, as it is universally supported.
    - Avoid `COMP-5` unless interfacing with non-COBOL programs; use `BINARY` for standard code.

### Isolating Vendor-Specific Code

```cobol
>>DEFINE VENDOR AS "IBM"

>>EVALUATE VENDOR
>>WHEN "IBM"
    COPY "io-ibm.cpy".
>>WHEN "MF"
    COPY "io-mf.cpy".
>>WHEN "GC"
    COPY "io-gc.cpy".
>>END-EVALUATE
```

### Dialect Emulation

Several compilers support emulating other vendors' dialects:

| Compiler | Emulation Option | Emulated Dialect |
|----------|-----------------|------------------|
| Micro Focus | `$SET DIALECT"ENTCOBOL"` | IBM Enterprise COBOL |
| GnuCOBOL | `-std=ibm` | IBM Enterprise COBOL |
| GnuCOBOL | `-std=mf` | Micro Focus COBOL |
| GnuCOBOL | `-std=cobol2014` | ISO COBOL 2014 |

Dialect emulation adjusts reserved words, data item behavior, and other compiler semantics to match the target dialect. It does not provide full compatibility -- programs may still require source changes.

---

## Examples

### IBM CICS Program Structure

```cobol
       CBL RENT,APOST,CICS
       IDENTIFICATION DIVISION.
       PROGRAM-ID. CUSTINQ.

       DATA DIVISION.
       WORKING-STORAGE SECTION.
       01  WS-CUST-ID       PIC X(10).
       01  WS-CUST-NAME     PIC X(40).
       01  WS-RESPONSE       PIC S9(8) COMP.

       PROCEDURE DIVISION.
       MAIN-PARA.
           EXEC CICS RECEIVE
               INTO(WS-CUST-ID)
               LENGTH(LENGTH OF WS-CUST-ID)
           END-EXEC

           EXEC CICS READ
               FILE('CUSTFILE')
               INTO(WS-CUST-NAME)
               RIDFLD(WS-CUST-ID)
               RESP(WS-RESPONSE)
           END-EXEC

           IF WS-RESPONSE = DFHRESP(NORMAL)
               EXEC CICS SEND TEXT
                   FROM(WS-CUST-NAME)
                   LENGTH(LENGTH OF WS-CUST-NAME)
               END-EXEC
           ELSE
               EXEC CICS SEND TEXT
                   FROM('CUSTOMER NOT FOUND')
                   LENGTH(18)
               END-EXEC
           END-IF

           EXEC CICS RETURN END-EXEC.
```

### GnuCOBOL Screen Application

```cobol
>>SOURCE FORMAT IS FREE
IDENTIFICATION DIVISION.
PROGRAM-ID. SCREEN-DEMO.

DATA DIVISION.
WORKING-STORAGE SECTION.
01  WS-EMP-ID     PIC X(8)  VALUE SPACES.
01  WS-EMP-NAME   PIC X(30) VALUE SPACES.
01  WS-ACTION     PIC X     VALUE SPACE.

SCREEN SECTION.
01  MAIN-SCREEN.
    05  BLANK SCREEN BACKGROUND-COLOR 1.
    05  LINE 1 COL 25 VALUE "EMPLOYEE INQUIRY"
        FOREGROUND-COLOR 7 HIGHLIGHT.
    05  LINE 3 COL 5 VALUE "Employee ID: ".
    05  LINE 3 COL 20 PIC X(8) USING WS-EMP-ID
        FOREGROUND-COLOR 3 HIGHLIGHT AUTO.
    05  LINE 5 COL 5 VALUE "Name: ".
    05  LINE 5 COL 20 PIC X(30) FROM WS-EMP-NAME
        FOREGROUND-COLOR 7.
    05  LINE 20 COL 5 VALUE "Action (Q=Quit): ".
    05  LINE 20 COL 25 PIC X USING WS-ACTION
        FOREGROUND-COLOR 3 HIGHLIGHT AUTO.

PROCEDURE DIVISION.
MAIN-PARA.
    PERFORM UNTIL WS-ACTION = "Q" OR "q"
        DISPLAY MAIN-SCREEN
        ACCEPT MAIN-SCREEN
        IF WS-ACTION NOT = "Q" AND "q"
            PERFORM LOOKUP-EMPLOYEE
        END-IF
    END-PERFORM
    STOP RUN.

LOOKUP-EMPLOYEE.
    MOVE "JOHN SMITH" TO WS-EMP-NAME.
```

### Conditional Compilation for Multi-Vendor Support

```cobol
>>SOURCE FORMAT IS FREE
>>DEFINE VENDOR AS "IBM"

IDENTIFICATION DIVISION.
PROGRAM-ID. PORTABLE.

DATA DIVISION.
WORKING-STORAGE SECTION.
01  WS-MESSAGE  PIC X(80).
01  WS-STATUS   PIC X(2).

PROCEDURE DIVISION.
MAIN-PARA.
    MOVE "Program started" TO WS-MESSAGE

>>EVALUATE VENDOR
>>WHEN "IBM"
    DISPLAY WS-MESSAGE UPON SYSOUT
>>WHEN "MF"
    DISPLAY WS-MESSAGE
>>WHEN "GC"
    DISPLAY WS-MESSAGE
>>END-EVALUATE

    PERFORM BUSINESS-LOGIC
    GOBACK.

BUSINESS-LOGIC.
    MOVE "Processing complete" TO WS-MESSAGE
>>IF VENDOR = "IBM"
    DISPLAY WS-MESSAGE UPON SYSOUT
>>ELSE
    DISPLAY WS-MESSAGE
>>END-IF
    .
```

---

## See Also

- [USAGE Clause](../data-division/usage.md) -- standard and vendor-specific USAGE types including COMP-5
- [PICTURE Clause](../data-division/picture.md) -- PICTURE interaction with vendor-specific USAGE types
- [CALL Statement](../procedure-division/program-linkage/call.md) -- BY VALUE and other vendor extensions in program linkage
- [SELECT Statement](../environment-division/select.md) -- vendor-specific file assignment extensions
- [Embedded SQL](embedded-sql.md) -- EXEC SQL vendor precompiler details
- [Compiler Directives](compiler-directives.md) -- vendor-specific directive mechanisms
