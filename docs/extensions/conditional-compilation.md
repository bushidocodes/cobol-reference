# Conditional Compilation

Conditional compilation directives allow sections of source code to be included or excluded at compile time based on defined conditions. These are processed by the Compiler Directing Facility (CDF) before the main compilation phase.

- **Standard:** COBOL 2002, COBOL 2014, COBOL 2023

---

## >>DEFINE

Defines a compile-time variable.

```cobol
>>DEFINE variable-name AS literal
>>DEFINE variable-name AS OFF
>>DEFINE variable-name AS PARAMETER
```

- `AS literal` — assigns a value (numeric or alphanumeric).
- `AS OFF` — undefines the variable.
- `AS PARAMETER` — retrieves the value from an environment variable at compile time.

```cobol
>>DEFINE DEBUG-MODE AS 1
>>DEFINE VERSION AS "2.5"
>>DEFINE DB-HOST AS PARAMETER
```

---

## >>IF / >>ELIF / >>ELSE / >>END-IF

Conditionally includes or excludes source text.

```cobol
>>IF condition
    [COBOL source text included when true]
>>ELIF condition
    [COBOL source text included when first condition is false]
>>ELSE
    [COBOL source text included when all conditions are false]
>>END-IF
```

### Conditions

| Condition | True when |
|-----------|-----------|
| `variable-name IS DEFINED` | The variable has been defined |
| `variable-name IS NOT DEFINED` | The variable has not been defined |
| `variable-name = literal` | The variable's value equals the literal |
| `variable-name > literal` | Greater than comparison |
| `variable-name < literal` | Less than comparison |
| `variable-name >= literal` | Greater than or equal |
| `variable-name <= literal` | Less than or equal |

### Examples

```cobol
>>DEFINE TARGET-SYSTEM AS "IBM"

>>IF TARGET-SYSTEM = "IBM"
       EXEC SQL
           SELECT * FROM CUSTOMERS
       END-EXEC
>>ELIF TARGET-SYSTEM = "MICROFOCUS"
       CALL "DB-ACCESS" USING WS-QUERY
>>ELSE
       DISPLAY "Unsupported platform"
       STOP RUN
>>END-IF

>>IF DEBUG-MODE IS DEFINED
       DISPLAY "Debug: WS-STATUS = " WS-STATUS
>>END-IF
```

---

## >>SET

Sets compiler options.

```cobol
>>SET SOURCEFORMAT AS { FIXED | FREE }
>>SET CONSTANT variable-name AS literal
```

- `SOURCEFORMAT` switches between fixed and free-form source within a file.
- `CONSTANT` defines a variable usable as a literal in COBOL statements (not just CDF directives).

---

## >>TURN

Controls exception checking.

```cobol
>>TURN exception-name CHECKING { ON | OFF }
```

```cobol
>>TURN EC-BOUND-SUBSCRIPT CHECKING ON
>>TURN EC-SIZE-OVERFLOW CHECKING ON
```

---

## Vendor Extensions

### $ Directives (Micro Focus / GnuCOBOL)

Some compilers support `$` as an alias for `>>`:

```cobol
$SET SOURCEFORMAT "FREE"
$IF DEBUG-MODE DEFINED
    DISPLAY "Debug mode"
$END
```

### D Lines (Debugging Lines)

Lines with `D` in column 7 (fixed format) or `>>D` (free format) are compiled only when debugging mode is active:

```cobol
D      DISPLAY "Debug: entering PROCESS-RECORD"
       PERFORM PROCESS-RECORD
D      DISPLAY "Debug: exiting PROCESS-RECORD"
```

---

## See Also

- [Source Format](../language/source-format.md) -- fixed vs free-form source
- [COPY](../copy-replace/copy.md) -- source text inclusion
- [REPLACE](../copy-replace/replace.md) -- source text replacement
- [Compiler Directives](compiler-directives.md) -- vendor-specific directives
