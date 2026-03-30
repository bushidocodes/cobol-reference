# Migration Guide

Practical guidance for migrating COBOL programs between standard versions and from COBOL to other languages.

---

## COBOL-74 to COBOL-85

The most common migration in practice. COBOL-85 is backward-compatible with COBOL-74, but modernizing code to use COBOL-85 features improves maintainability.

### Key Transformations

| COBOL-74 | COBOL-85 | Benefit |
|----------|----------|---------|
| Period-terminated IF | `END-IF` | Eliminates scope ambiguity |
| Period-terminated PERFORM | `END-PERFORM` (inline) | Enables inline loops |
| `NEXT SENTENCE` | `CONTINUE` | Respects scope terminators |
| `EXAMINE` | `INSPECT` | Multi-character patterns |
| Nested IF with periods | Nested IF with `END-IF` | Clearer nesting |
| `GO TO` with `ALTER` | `EVALUATE` / condition variables | Readable branching |
| No `EVALUATE` | `EVALUATE TRUE/WHEN` | Multi-way branching |
| No `INITIALIZE` | `INITIALIZE` | Cleaner data setup |
| `NOT` conditions split | `NOT AT END`, `NOT INVALID KEY` | Inline success handling |

### Step-by-Step Process

1. **Add scope terminators** — Replace period-delimited IF/PERFORM with END-IF/END-PERFORM. This is the highest-value change.

    ```cobol
    *> COBOL-74
        IF A > B
            MOVE A TO C.
        PERFORM NEXT-STEP.

    *> COBOL-85
        IF A > B
            MOVE A TO C
        END-IF
        PERFORM NEXT-STEP
    ```

2. **Convert NEXT SENTENCE to CONTINUE**

    ```cobol
    *> COBOL-74
        IF A > B NEXT SENTENCE
        ELSE PERFORM PROCESS-A.

    *> COBOL-85
        IF A <= B
            PERFORM PROCESS-A
        END-IF
    ```

3. **Replace GO TO / ALTER with EVALUATE**

    ```cobol
    *> COBOL-74 (ALTER pattern)
        ALTER SWITCH-PARA TO PROCEED TO ROUTINE-2.
        ...
    SWITCH-PARA.
        GO TO ROUTINE-1.

    *> COBOL-85
        EVALUATE WS-STATE
            WHEN 1 PERFORM ROUTINE-1
            WHEN 2 PERFORM ROUTINE-2
        END-EVALUATE
    ```

4. **Replace EXAMINE with INSPECT**

    ```cobol
    *> COBOL-74
        EXAMINE WS-FIELD TALLYING LEADING ZERO
            REPLACING BY SPACE.

    *> COBOL-85
        INSPECT WS-FIELD TALLYING WS-COUNT
            FOR LEADING ZERO
        INSPECT WS-FIELD REPLACING LEADING ZERO BY SPACE
    ```

5. **Add inline PERFORM** where appropriate

    ```cobol
    *> COBOL-74
        PERFORM READ-LOOP THRU READ-LOOP-EXIT.
    READ-LOOP.
        READ INPUT-FILE AT END GO TO READ-LOOP-EXIT.
        PERFORM PROCESS-RECORD.
        GO TO READ-LOOP.
    READ-LOOP-EXIT.
        EXIT.

    *> COBOL-85
        PERFORM UNTIL WS-EOF
            READ INPUT-FILE
                AT END SET WS-EOF TO TRUE
                NOT AT END PERFORM PROCESS-RECORD
            END-READ
        END-PERFORM
    ```

---

## COBOL-85 to COBOL 2002

### Key New Features to Adopt

| Feature | Benefit |
|---------|---------|
| `LOCAL-STORAGE SECTION` | Clean state on each subprogram call |
| `FUNCTION ALL INTRINSIC` | Call functions without `FUNCTION` keyword |
| Free-form source format | No column restrictions |
| `FUNCTION-ID` | User-defined functions |
| `>>IF` / `>>DEFINE` | Conditional compilation |
| `TYPEDEF` | Reusable type definitions |
| `RAISE` / `RESUME` | Structured exception handling |
| `ALLOCATE` / `FREE` | Dynamic memory management |

### Migration Notes

- Free-form source requires `>>SET SOURCEFORMAT AS FREE` or a compiler option.
- `LOCAL-STORAGE` replaces the `INITIAL` clause pattern for stateless subprograms.
- User-defined functions require `FUNCTION-ID` and `REPOSITORY` declarations.

---

## COBOL to Java

Based on patterns from migration studies (Sneed 2010, De Marco 2018).

### Structural Mapping

| COBOL | Java |
|-------|------|
| Program (PROGRAM-ID) | Class |
| 01-level group item | Class with fields |
| Elementary item (PIC) | Typed field (`int`, `String`, `BigDecimal`) |
| Paragraph | Method |
| Section | Class or method group |
| Level-88 condition name | Boolean method or enum |
| Copybook | Shared class / interface |
| WORKING-STORAGE | Instance variables |
| LINKAGE SECTION | Method parameters |
| PERFORM paragraph | Method call |
| CALL subprogram | Method call or service call |

### Data Type Mapping

| COBOL PIC | Java Type | Notes |
|-----------|-----------|-------|
| `PIC X(n)` | `String` | Fixed-length in COBOL, variable in Java |
| `PIC 9(n)` | `int` / `long` | Depending on size |
| `PIC 9(n)V9(m)` | `BigDecimal` | Preserves exact decimal arithmetic |
| `PIC S9(n) COMP` | `int` / `long` | Binary representation |
| `COMP-1` | `float` | Single precision |
| `COMP-2` | `double` | Double precision |
| `COMP-3` | `BigDecimal` | Packed decimal |
| Level-88 | `boolean` method or `enum` | |
| `OCCURS n TIMES` | Array or `List<T>` | |
| `REDEFINES` | Separate classes or byte-array parsing | Complex mapping |

### Common Challenges

1. **REDEFINES** — No direct Java equivalent. Requires byte-array manipulation or separate class hierarchies.
2. **CORRESPONDING** — Must be implemented as explicit field-by-field copies.
3. **Decimal arithmetic** — Java `double` introduces rounding errors. Use `BigDecimal` for financial calculations.
4. **Fixed-length strings** — COBOL strings are space-padded; Java strings are variable-length. Add `.trim()` everywhere.
5. **Global state** — COBOL WORKING-STORAGE acts as global state. Java requires passing objects explicitly.
6. **GO TO** — No Java equivalent. Requires restructuring to loops and conditionals.
7. **PERFORM THRU** — Sequential paragraph execution has no Java equivalent. Convert to sequential method calls.
8. **File I/O** — COBOL's record-oriented I/O maps to Java's `BufferedReader`/`BufferedWriter` or database access.
9. **EBCDIC data** — Binary fields (COMP-3, COMP) in EBCDIC files require special ETL handling.

### Migration Approaches

| Approach | Effort | Result Quality | Risk |
|----------|--------|---------------|------|
| **Automated translation** | Low | "JOBOL" (COBOL idioms in Java) | Maintainability debt |
| **Rewrite** | Very high | Native Java | Schedule/budget risk |
| **Wrapping** (API layer) | Medium | COBOL stays, Java frontend | Ongoing COBOL dependency |
| **Hybrid** (automated + refactor) | Medium-high | Improved Java | Balanced risk |

The New York Times (2018) converted 2M+ lines of COBOL using automated translation at 1/10th the cost of a manual rewrite. Testing accounted for 70-80% of the project time.

---

## COBOL to C# / .NET

Similar to Java migration, with these differences:

| COBOL | C# |
|-------|----|
| `PIC 9(n)V9(m)` | `decimal` (native decimal type) |
| COMP-3 | `decimal` |
| Copybook | Shared class library |
| CICS | ASP.NET / WCF |
| JCL | Windows services / scheduled tasks |

C#'s native `decimal` type is a closer match to COBOL's fixed-point arithmetic than Java's `BigDecimal`.

---

## See Also

- [Standards](../standards.md) -- COBOL standard evolution
- [Best Practices](best-practices.md) -- modern COBOL coding patterns
- [Sample Programs](sample-programs.md) -- complete COBOL examples
- [Terminology Mapping](terminology.md) -- cross-language concept mapping
