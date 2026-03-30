# Inter-Language Interoperability

COBOL programs can call and be called by programs written in other languages, particularly C. This is essential for integrating COBOL business logic with modern systems.

- **Standard:** CALL BY VALUE (COBOL 2002), vendor extensions for C interop

---

## COBOL Calling C

### Data Type Mapping

| COBOL | C | Size |
|-------|---|------|
| `BINARY-CHAR` | `char` / `int8_t` | 1 byte |
| `BINARY-SHORT` | `short` / `int16_t` | 2 bytes |
| `BINARY-LONG` | `int` / `int32_t` | 4 bytes |
| `BINARY-DOUBLE` | `long long` / `int64_t` | 8 bytes |
| `COMP-1` / `FLOAT-SHORT` | `float` | 4 bytes |
| `COMP-2` / `FLOAT-LONG` | `double` | 8 bytes |
| `PIC X(n)` | `char[n]` (not null-terminated) | n bytes |
| `USAGE POINTER` | `void *` | 4/8 bytes |

### Null-Terminated Strings

COBOL strings are fixed-length and space-padded. C strings are null-terminated. Use Z-literals or explicit null bytes:

```cobol
*> Z-literal (null-terminated string)
CALL "c_function" USING Z"Hello"

*> Manual null termination
STRING WS-TEXT DELIMITED BY SPACES
       X"00" DELIMITED BY SIZE
    INTO WS-BUFFER
CALL "c_function" USING WS-BUFFER
```

### Calling Convention

```cobol
CALL "strlen" USING BY REFERENCE WS-STRING
    RETURNING WS-LENGTH
```

- `BY VALUE` passes the actual value (required for C scalar arguments)
- `BY REFERENCE` passes a pointer (default, matches C pointer parameters)
- `BY CONTENT` passes a copy's address (callee gets a pointer but can't modify original)

### Example: Calling C Math Library

```cobol
       01  WS-ANGLE    COMP-2 VALUE 1.5708.
       01  WS-RESULT   COMP-2.

       CALL "sin" USING BY VALUE WS-ANGLE
           RETURNING WS-RESULT
       DISPLAY "sin(pi/2) = " WS-RESULT
```

---

## C Calling COBOL

The C program must initialize the COBOL runtime before calling COBOL programs.

### GnuCOBOL Example

**C caller:**
```c
#include <libcob.h>

int main(int argc, char *argv[]) {
    cob_init(argc, argv);

    /* Call a COBOL program */
    int ret = CALC_TAX("10000.00", "0.0825");
    printf("Return code: %d\n", ret);

    return 0;
}
```

**COBOL callee:**
```cobol
       IDENTIFICATION DIVISION.
       PROGRAM-ID. CALC-TAX.
       DATA DIVISION.
       LINKAGE SECTION.
       01  LS-AMOUNT   PIC X(10).
       01  LS-RATE     PIC X(6).
       PROCEDURE DIVISION USING LS-AMOUNT LS-RATE.
           DISPLAY "Amount: " LS-AMOUNT
           DISPLAY "Rate: " LS-RATE
           MOVE 0 TO RETURN-CODE
           GOBACK.
```

### Compile and Link

```bash
# GnuCOBOL
cobc -c calc-tax.cob          # Compile COBOL
gcc -c main.c $(cob-config --cflags)  # Compile C
gcc -o program main.o calc-tax.o $(cob-config --libs)  # Link
```

---

## COBOL and Java (IBM)

IBM Enterprise COBOL supports Java interoperability through JNI:

```cobol
       01  JNIENVPTR   USAGE POINTER.   *> JNI environment
       01  WS-JAVA-OBJ USAGE OBJECT REFERENCE.

       CALL "JNIGETMETHOD" USING JNIENVPTR ...
```

---

## COBOL and SQL

See [Embedded SQL](embedded-sql.md) for the primary database interop mechanism.

---

## See Also

- [CALL](../procedure-division/program-linkage/call.md) -- inter-program communication
- [USAGE Clause](../data-division/usage.md) -- BINARY-CHAR/SHORT/LONG/DOUBLE types
- [Embedded SQL](embedded-sql.md) -- database access
