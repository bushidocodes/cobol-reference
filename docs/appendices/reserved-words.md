# Reserved Words

Reserved words in COBOL have special meaning to the compiler and cannot be used as
user-defined names (data names, paragraph names, section names, etc.). Using a reserved
word as a data name is one of the most common sources of compilation errors.

!!! warning "Compiler Variations"
    Reserved word lists vary by compiler and standard. A word reserved in one compiler
    may be unreserved in another. Always consult your specific compiler's documentation.

## COBOL-85 Standard Reserved Words

The following words are reserved in the ANSI COBOL-85 standard (ANSI X3.23-1985).

### A

| Word | Context |
|------|---------|
| `ACCEPT` | Statement |
| `ACCESS` | Clause |
| `ADD` | Statement |
| `ADVANCING` | Phrase |
| `AFTER` | Phrase |
| `ALL` | Figurative constant / phrase |
| `ALPHABET` | Clause |
| `ALPHABETIC` | Class condition |
| `ALPHABETIC-LOWER` | Class condition |
| `ALPHABETIC-UPPER` | Class condition |
| `ALSO` | Phrase |
| `ALTER` | Statement |
| `ALTERNATE` | Clause |
| `AND` | Logical operator |
| `ANY` | Phrase |
| `APPLY` | Clause |
| `ARE` | Noise word |
| `AREA` | Clause |
| `AREAS` | Clause |
| `ASCENDING` | Phrase |
| `ASSIGN` | Clause |
| `AT` | Phrase |
| `AUTHOR` | Paragraph |

### B

| Word | Context |
|------|---------|
| `BEFORE` | Phrase |
| `BINARY` | Usage |
| `BLANK` | Clause |
| `BLOCK` | Clause |
| `BOTTOM` | Phrase |
| `BY` | Phrase |

### C

| Word | Context |
|------|---------|
| `CALL` | Statement |
| `CANCEL` | Statement |
| `CD` | Entry |
| `CF` | Report group |
| `CH` | Report group |
| `CHARACTER` | Phrase |
| `CHARACTERS` | Phrase |
| `CLASS` | Condition / clause |
| `CLOCK-UNITS` | Clause |
| `CLOSE` | Statement |
| `COBOL` | Keyword |
| `CODE` | Clause |
| `CODE-SET` | Clause |
| `COLLATING` | Clause |
| `COLUMN` | Clause |
| `COMMA` | Separator |
| `COMMON` | Clause |
| `COMMUNICATION` | Section |
| `COMP` | Usage |
| `COMP-1` | Usage (extension) |
| `COMP-2` | Usage (extension) |
| `COMP-3` | Usage (extension) |
| `COMP-4` | Usage (extension) |
| `COMP-5` | Usage (extension) |
| `COMPUTATIONAL` | Usage |
| `COMPUTATIONAL-1` | Usage (extension) |
| `COMPUTATIONAL-2` | Usage (extension) |
| `COMPUTATIONAL-3` | Usage (extension) |
| `COMPUTATIONAL-4` | Usage (extension) |
| `COMPUTATIONAL-5` | Usage (extension) |
| `COMPUTE` | Statement |
| `CONFIGURATION` | Section |
| `CONTAINS` | Clause |
| `CONTENT` | Phrase |
| `CONTINUE` | Statement |
| `CONTROL` | Clause |
| `CONTROLS` | Clause |
| `CONVERTING` | Phrase |
| `COPY` | Statement |
| `CORR` | Phrase |
| `CORRESPONDING` | Phrase |
| `COUNT` | Phrase |
| `CURRENCY` | Clause |

### D

| Word | Context |
|------|---------|
| `DATA` | Division |
| `DATE` | Function / phrase |
| `DATE-COMPILED` | Paragraph |
| `DATE-WRITTEN` | Paragraph |
| `DAY` | Function / phrase |
| `DAY-OF-WEEK` | Function |
| `DE` | Report group |
| `DEBUGGING` | Clause |
| `DECIMAL-POINT` | Clause |
| `DECLARATIVES` | Section |
| `DELETE` | Statement |
| `DELIMITED` | Phrase |
| `DELIMITER` | Phrase |
| `DEPENDING` | Clause |
| `DESCENDING` | Phrase |
| `DESTINATION` | Clause |
| `DETAIL` | Report group |
| `DISABLE` | Statement |
| `DISPLAY` | Statement / usage |
| `DIVIDE` | Statement |
| `DIVISION` | Structure |
| `DOWN` | Phrase |
| `DUPLICATES` | Clause |
| `DYNAMIC` | Clause |

### E

| Word | Context |
|------|---------|
| `EGI` | Communication |
| `ELSE` | Phrase |
| `EMI` | Communication |
| `ENABLE` | Statement |
| `END` | Phrase |
| `END-ADD` | Scope terminator |
| `END-CALL` | Scope terminator |
| `END-COMPUTE` | Scope terminator |
| `END-DELETE` | Scope terminator |
| `END-DIVIDE` | Scope terminator |
| `END-EVALUATE` | Scope terminator |
| `END-IF` | Scope terminator |
| `END-MULTIPLY` | Scope terminator |
| `END-OF-PAGE` | Phrase |
| `END-PERFORM` | Scope terminator |
| `END-READ` | Scope terminator |
| `END-RECEIVE` | Scope terminator |
| `END-RETURN` | Scope terminator |
| `END-REWRITE` | Scope terminator |
| `END-SEARCH` | Scope terminator |
| `END-START` | Scope terminator |
| `END-STRING` | Scope terminator |
| `END-SUBTRACT` | Scope terminator |
| `END-UNSTRING` | Scope terminator |
| `END-WRITE` | Scope terminator |
| `ENTER` | Statement |
| `ENVIRONMENT` | Division |
| `EOP` | Phrase |
| `EQUAL` | Condition |
| `ERROR` | Phrase |
| `ESI` | Communication |
| `EVALUATE` | Statement |
| `EVERY` | Phrase |
| `EXCEPTION` | Phrase |
| `EXIT` | Statement |
| `EXTEND` | Phrase |
| `EXTERNAL` | Clause |

### F

| Word | Context |
|------|---------|
| `FALSE` | Value |
| `FD` | Entry |
| `FILE` | Section / phrase |
| `FILE-CONTROL` | Paragraph |
| `FILLER` | Data name |
| `FINAL` | Phrase |
| `FIRST` | Phrase |
| `FOOTING` | Clause |
| `FOR` | Phrase |
| `FROM` | Phrase |
| `FUNCTION` | Keyword |

### G

| Word | Context |
|------|---------|
| `GENERATE` | Statement |
| `GIVING` | Phrase |
| `GLOBAL` | Clause |
| `GO` | Statement |
| `GREATER` | Condition |
| `GROUP` | Phrase |

### H

| Word | Context |
|------|---------|
| `HEADING` | Clause |
| `HIGH-VALUE` | Figurative constant |
| `HIGH-VALUES` | Figurative constant |

### I

| Word | Context |
|------|---------|
| `I-O` | Phrase |
| `I-O-CONTROL` | Paragraph |
| `IDENTIFICATION` | Division |
| `IF` | Statement |
| `IN` | Qualifier |
| `INDEX` | Usage |
| `INDEXED` | Clause |
| `INDICATE` | Clause |
| `INITIAL` | Clause |
| `INITIALIZE` | Statement |
| `INITIATE` | Statement |
| `INPUT` | Phrase |
| `INPUT-OUTPUT` | Section |
| `INSPECT` | Statement |
| `INSTALLATION` | Paragraph |
| `INTO` | Phrase |
| `INVALID` | Phrase |
| `IS` | Noise word |

### J–K

| Word | Context |
|------|---------|
| `JUST` | Clause |
| `JUSTIFIED` | Clause |
| `KEY` | Clause |

### L

| Word | Context |
|------|---------|
| `LABEL` | Clause |
| `LAST` | Phrase |
| `LEADING` | Phrase |
| `LEFT` | Phrase |
| `LENGTH` | Function |
| `LESS` | Condition |
| `LIMIT` | Clause |
| `LIMITS` | Clause |
| `LINAGE` | Clause |
| `LINAGE-COUNTER` | Special register |
| `LINE` | Clause |
| `LINE-COUNTER` | Special register |
| `LINES` | Clause |
| `LINKAGE` | Section |
| `LOCK` | Phrase |
| `LOW-VALUE` | Figurative constant |
| `LOW-VALUES` | Figurative constant |

### M

| Word | Context |
|------|---------|
| `MEMORY` | Clause |
| `MERGE` | Statement |
| `MESSAGE` | Clause |
| `MODE` | Clause |
| `MODULES` | Clause |
| `MOVE` | Statement |
| `MULTIPLE` | Clause |
| `MULTIPLY` | Statement |

### N

| Word | Context |
|------|---------|
| `NATIVE` | Phrase |
| `NEGATIVE` | Condition |
| `NEXT` | Phrase |
| `NO` | Phrase |
| `NOT` | Logical operator |
| `NUMBER` | Clause |
| `NUMERIC` | Class condition |
| `NUMERIC-EDITED` | Class condition |

### O

| Word | Context |
|------|---------|
| `OBJECT-COMPUTER` | Paragraph |
| `OCCURS` | Clause |
| `OF` | Qualifier |
| `OFF` | Phrase |
| `OMITTED` | Phrase |
| `ON` | Phrase |
| `OPEN` | Statement |
| `OPTIONAL` | Clause |
| `OR` | Logical operator |
| `ORDER` | Clause |
| `ORGANIZATION` | Clause |
| `OTHER` | Phrase |
| `OUTPUT` | Phrase |
| `OVERFLOW` | Phrase |

### P

| Word | Context |
|------|---------|
| `PACKED-DECIMAL` | Usage |
| `PADDING` | Clause |
| `PAGE` | Clause |
| `PAGE-COUNTER` | Special register |
| `PERFORM` | Statement |
| `PF` | Report group |
| `PH` | Report group |
| `PIC` | Clause |
| `PICTURE` | Clause |
| `PLUS` | Phrase |
| `POINTER` | Usage |
| `POSITION` | Phrase |
| `POSITIVE` | Condition |
| `PRINTING` | Phrase |
| `PROCEDURE` | Division |
| `PROCEDURES` | Phrase |
| `PROCEED` | Phrase |
| `PROGRAM` | Phrase |
| `PROGRAM-ID` | Paragraph |
| `PURGE` | Statement |

### Q

| Word | Context |
|------|---------|
| `QUEUE` | Clause |
| `QUOTE` | Figurative constant |
| `QUOTES` | Figurative constant |

### R

| Word | Context |
|------|---------|
| `RANDOM` | Clause |
| `RD` | Entry |
| `READ` | Statement |
| `RECEIVE` | Statement |
| `RECORD` | Clause |
| `RECORDS` | Clause |
| `REDEFINES` | Clause |
| `REEL` | Phrase |
| `REFERENCE` | Phrase |
| `REFERENCES` | Phrase |
| `RELATIVE` | Clause |
| `RELEASE` | Statement |
| `REMAINDER` | Phrase |
| `REMOVAL` | Phrase |
| `RENAMES` | Clause |
| `REPLACE` | Statement |
| `REPLACING` | Phrase |
| `REPORT` | Section / clause |
| `REPORTING` | Clause |
| `REPORTS` | Clause |
| `RERUN` | Clause |
| `RESERVE` | Clause |
| `RESET` | Clause |
| `RETURN` | Statement |
| `REVERSED` | Phrase |
| `REWIND` | Phrase |
| `REWRITE` | Statement |
| `RF` | Report group |
| `RH` | Report group |
| `RIGHT` | Phrase |
| `ROUNDED` | Phrase |
| `RUN` | Phrase |

### S

| Word | Context |
|------|---------|
| `SAME` | Clause |
| `SD` | Entry |
| `SEARCH` | Statement |
| `SECTION` | Structure |
| `SECURITY` | Paragraph |
| `SEGMENT` | Clause |
| `SEGMENT-LIMIT` | Clause |
| `SELECT` | Clause |
| `SEND` | Statement |
| `SENTENCE` | Phrase |
| `SEPARATE` | Phrase |
| `SEQUENCE` | Clause |
| `SEQUENTIAL` | Clause |
| `SET` | Statement |
| `SIGN` | Clause |
| `SIZE` | Phrase |
| `SORT` | Statement |
| `SORT-MERGE` | Section |
| `SOURCE` | Clause |
| `SOURCE-COMPUTER` | Paragraph |
| `SPACE` | Figurative constant |
| `SPACES` | Figurative constant |
| `SPECIAL-NAMES` | Paragraph |
| `STANDARD` | Phrase |
| `STANDARD-1` | Alphabet name |
| `STANDARD-2` | Alphabet name |
| `START` | Statement |
| `STATUS` | Clause |
| `STOP` | Statement |
| `STRING` | Statement |
| `SUB-QUEUE-1` | Communication |
| `SUB-QUEUE-2` | Communication |
| `SUB-QUEUE-3` | Communication |
| `SUBTRACT` | Statement |
| `SUM` | Clause |
| `SUPPRESS` | Statement |
| `SYMBOLIC` | Clause |
| `SYNC` | Clause |
| `SYNCHRONIZED` | Clause |

### T

| Word | Context |
|------|---------|
| `TABLE` | Clause |
| `TALLYING` | Phrase |
| `TAPE` | Phrase |
| `TERMINAL` | Communication |
| `TERMINATE` | Statement |
| `TEST` | Phrase |
| `TEXT` | Communication |
| `THAN` | Phrase |
| `THEN` | Phrase |
| `THROUGH` | Phrase |
| `THRU` | Phrase |
| `TIME` | Phrase |
| `TIMES` | Phrase |
| `TO` | Phrase |
| `TOP` | Phrase |
| `TRAILING` | Phrase |
| `TRUE` | Value |
| `TYPE` | Clause |

### U

| Word | Context |
|------|---------|
| `UNIT` | Phrase |
| `UNSTRING` | Statement |
| `UNTIL` | Phrase |
| `UP` | Phrase |
| `UPON` | Phrase |
| `USAGE` | Clause |
| `USE` | Statement |
| `USING` | Phrase |

### V

| Word | Context |
|------|---------|
| `VALUE` | Clause |
| `VALUES` | Clause |
| `VARYING` | Phrase |

### W

| Word | Context |
|------|---------|
| `WHEN` | Phrase |
| `WITH` | Phrase |
| `WORDS` | Clause |
| `WORKING-STORAGE` | Section |
| `WRITE` | Statement |

### Z

| Word | Context |
|------|---------|
| `ZERO` | Figurative constant |
| `ZEROES` | Figurative constant |
| `ZEROS` | Figurative constant |

---

## COBOL 2002 / 2014 Additions

The following words were added in the COBOL 2002 and COBOL 2014 standards.

| Word | Standard | Context |
|------|----------|---------|
| `ALLOCATE` | 2002 | Statement |
| `ANYCASE` | 2014 | Phrase |
| `B-AND` | 2002 | Boolean operator |
| `B-NOT` | 2002 | Boolean operator |
| `B-OR` | 2002 | Boolean operator |
| `B-XOR` | 2002 | Boolean operator |
| `BOOLEAN` | 2002 | Usage |
| `CAPACITY` | 2014 | Clause |
| `CONSTANT` | 2002 | Clause |
| `END-ACCEPT` | 2002 | Scope terminator |
| `END-DISPLAY` | 2002 | Scope terminator |
| `EXCEPTION-OBJECT` | 2002 | Special register |
| `EXPANDS` | 2002 | Phrase |
| `FACTORY` | 2002 | OO keyword |
| `FLOAT-EXTENDED` | 2002 | Usage |
| `FLOAT-LONG` | 2002 | Usage |
| `FLOAT-SHORT` | 2002 | Usage |
| `FORMAT` | 2002 | Clause |
| `FREE` | 2002 | Statement |
| `GET` | 2002 | Statement |
| `IMPLEMENTS` | 2002 | Phrase |
| `INHERITS` | 2002 | Phrase |
| `INTERFACE` | 2002 | OO keyword |
| `INTERFACE-ID` | 2002 | Paragraph |
| `INVOKE` | 2002 | Statement |
| `LOCALE` | 2002 | Clause |
| `METHOD` | 2002 | OO keyword |
| `METHOD-ID` | 2002 | Paragraph |
| `NESTED` | 2002 | Phrase |
| `OBJECT` | 2002 | OO keyword |
| `OPTIONS` | 2002 | Paragraph |
| `OVERRIDE` | 2002 | Phrase |
| `PROPERTY` | 2002 | OO keyword |
| `PROTOTYPE` | 2002 | Phrase |
| `RAISE` | 2002 | Statement |
| `RAISING` | 2002 | Phrase |
| `REPOSITORY` | 2002 | Paragraph |
| `RESUME` | 2002 | Statement |
| `RETRY` | 2002 | Phrase |
| `SELF` | 2002 | Object reference |
| `SOURCES` | 2002 | Clause |
| `STEP` | 2014 | Phrase |
| `STRONG` | 2002 | Phrase |
| `SUPER` | 2002 | Object reference |
| `TYPEDEF` | 2002 | Clause |
| `UNIVERSAL` | 2002 | Phrase |
| `VALIDATE` | 2002 | Statement |

---

## Common Vendor Extensions

Many compilers add their own reserved words beyond the standard. Below are commonly
encountered vendor-specific reserved words.

| Word | Vendor(s) | Purpose |
|------|-----------|---------|
| `COMP-5` | IBM, Micro Focus | Native binary |
| `COMP-X` | Micro Focus | Variable-length binary |
| `DISPLAY-1` | IBM | DBCS display |
| `EGCS` | IBM | Extended graphic character set |
| `EXEC` | IBM, Micro Focus | Embedded SQL/CICS prefix |
| `END-EXEC` | IBM, Micro Focus | Embedded SQL/CICS suffix |
| `GOBACK` | IBM, Micro Focus | Return to caller |
| `JSON` | IBM (Enterprise COBOL 6+) | JSON support |
| `NULL` | IBM, Micro Focus | Null pointer |
| `NULLS` | IBM, Micro Focus | Null pointer |
| `OBJECT` | Micro Focus | OO support |
| `SERVICE` | IBM | Service statement |
| `XML` | IBM (Enterprise COBOL 4+) | XML support |

---

## Tips for Avoiding Reserved Word Conflicts

1. **Prefix your data names** with a meaningful abbreviation:
   ```cobol
   01  WS-STATUS        PIC X(02).
   01  WS-COUNT         PIC 9(05).
   ```

2. **Use hyphens** to create compound names that are unlikely to collide:
   ```cobol
   01  RECORD-COUNT     PIC 9(05).
   01  FILE-STATUS-CODE PIC XX.
   ```

3. **Check your compiler's documentation** for the exact list of reserved words,
   as it may differ from the standard.

4. **Use a consistent naming convention** across your project to reduce accidental
   collisions.

!!! tip "Quick Test"
    If you're unsure whether a word is reserved, try using it as a data name in a
    small test program. The compiler will immediately tell you if it's reserved.
