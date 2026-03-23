# PICTURE Clause

The PICTURE clause specifies the category, size, and editing characteristics of an elementary data item by means of a character string of picture symbols.

**Standard:** COBOL-60, COBOL-61, COBOL-61 Extended, COBOL-65, COBOL-68, COBOL-85, COBOL 2002, COBOL 2014, COBOL 2023

---

## Syntax

```cobol
level-number  data-name  PIC[TURE] [IS]  picture-string.
```

The picture string is a sequence of picture symbols that together define the item's category, size, and editing format.

### Repetition Factor

Any symbol may be followed by a parenthesized integer to indicate repetition:

```
9(5)      is equivalent to   99999
X(10)     is equivalent to   XXXXXXXXXX
Z(4)9     is equivalent to   ZZZZ9
```

The maximum length of a picture string (after expansion of repetition factors) is 30 characters for COBOL-85, and implementation-defined but typically at least 50 for COBOL 2002 and later.

---

## Picture Symbols

| Symbol | Meaning |
|--------|---------|
| `9` | Numeric digit position |
| `X` | Any character position (alphanumeric) |
| `A` | Alphabetic character position (A-Z, a-z, space) |
| `V` | Assumed decimal point (not stored) |
| `S` | Operational sign (not stored as a character unless `SIGN IS SEPARATE`) |
| `P` | Assumed decimal scaling position |
| `Z` | Zero-suppression replacement with space |
| `*` | Zero-suppression replacement with asterisk |
| `+` | Replacement sign, positive = space or `+`, negative = `-` |
| `-` | Replacement sign, positive = space, negative = `-` |
| `CR` | Credit symbol, appears only if value is negative |
| `DB` | Debit symbol, appears only if value is negative |
| `$` | Currency sign (fixed or floating) |
| `.` | Decimal point (insertion character) |
| `,` | Comma (insertion character) |
| `B` | Space insertion character |
| `0` | Zero insertion character |
| `/` | Slash insertion character |

---

## Data Categories

The combination of picture symbols determines the data item's category. Each elementary item with a PICTURE clause belongs to exactly one category.

### Alphabetic

Contains only `A` symbols.

```cobol
PIC A(10)
```

Permits only the letters A-Z, a-z, and the space character.

### Alphanumeric

Contains only `X` symbols, or a combination of `A`, `X`, and `9` symbols used together (when the string contains more than one type among `A`, `X`, and `9` without editing symbols).

```cobol
PIC X(20)
PIC X(5)9(3)      *> treated as alphanumeric, size 8
```

Permits any character in the character set.

### Numeric

Contains only `9`, `V`, `S`, and `P` symbols.

```cobol
PIC 9(5)
PIC S9(7)V99
PIC S999V9(4)
PIC PPP999
```

Numeric items are used in arithmetic. The maximum number of digit positions (`9` and `P` combined) is 18 in standard COBOL (31 or more in some implementations).

### Numeric Edited

Contains `9` symbols mixed with one or more editing symbols: `Z`, `*`, `+`, `-`, `CR`, `DB`, `$`, `.`, `,`, `B`, `0`, `/`.

```cobol
PIC ZZZ,ZZ9.99
PIC $$$,$$9.99CR
PIC *(6).**
```

Numeric edited items are the result of editing a numeric value for display. They cannot be used as operands in arithmetic statements.

### Alphanumeric Edited

Contains `X`, `A`, or `9` symbols combined with `B`, `0`, or `/` insertion symbols, where the result is not a numeric edited category.

```cobol
PIC X(5)BX(5)
PIC 99/99/9999
PIC XX-XX       *> not standard; implementation-dependent
```

---

## Core Symbols in Detail

### 9 -- Digit Position

Each `9` represents one decimal digit (0-9) in storage.

```cobol
       05  QUANTITY   PIC 9(4).
```

| Stored Value | Displayed As |
|-------------|-------------|
| 0000 | `0000` |
| 0042 | `0042` |
| 9999 | `9999` |

### X -- Character Position

Each `X` represents one character position that may contain any character.

```cobol
       05  CUSTOMER-NAME   PIC X(25).
```

### A -- Alphabetic Position

Each `A` represents one character position restricted to letters and spaces.

```cobol
       05  STATE-ABBREV   PIC AA.
```

### V -- Assumed Decimal Point

`V` marks the position of the decimal point within a numeric item. It does not occupy storage; it defines where the compiler assumes the decimal point to be for alignment in arithmetic.

```cobol
       05  UNIT-PRICE   PIC 9(5)V99.
```

| Internal Value | Represents |
|---------------|-----------|
| `0012350` | 123.50 |
| `9999999` | 99999.99 |

A picture string may contain at most one `V`. If no `V` is specified, the decimal point is assumed to be to the right of the rightmost digit.

### S -- Operational Sign

`S` indicates that the item has an operational sign (positive or negative). It must be the leftmost symbol in the picture string.

```cobol
       05  NET-AMOUNT   PIC S9(7)V99.
```

Without `S`, a numeric item is treated as unsigned (absolute value).

**Sign storage:** By default, the sign is stored as an overpunch on the last (or first) digit, sharing storage with that digit (no extra byte). With `SIGN IS SEPARATE`, an additional byte is used:

```cobol
       05  SIGNED-AMT   PIC S9(5)  SIGN IS LEADING SEPARATE.
```

| Clause | Storage Size for `S9(5)` | Example (value -123) |
|--------|-------------------------|---------------------|
| (default, trailing) | 5 bytes | `0012M` (EBCDIC overpunch) |
| SIGN IS LEADING | 5 bytes | `J0123` (EBCDIC overpunch) |
| SIGN IS TRAILING SEPARATE | 6 bytes | `00123-` |
| SIGN IS LEADING SEPARATE | 6 bytes | `-00123` |

### P -- Decimal Scaling Position

`P` represents an assumed decimal scaling position. Each `P` implies a digit position that is not stored but is used for decimal alignment. `P` symbols must appear at the left end or right end of the digit string (but not both).

```cobol
       05  BIG-NUMBER   PIC 9(3)PPP.
       05  TINY-NUMBER  PIC PPP999.
```

| Picture | Size | Value 123 Represents |
|---------|------|---------------------|
| `9(3)PPP` | 3 bytes | 123,000 |
| `PPP999` | 3 bytes | 0.000123 |
| `PPP9` | 1 byte | 0.0001 through 0.0009 |

`P` counts toward the maximum 18-digit limit. `V` is implied at the end farthest from the `9` symbols.

---

## Editing Symbols in Detail

Editing symbols transform a numeric value into a human-readable format. The edited result is stored in the data item.

### Z -- Zero Suppression with Space

Each `Z` represents a digit position where a leading zero is replaced by a space. `Z` symbols must be contiguous and must appear to the left of any `9` symbols (except in an all-`Z` picture).

```cobol
       05  EDITED-QTY   PIC Z(4)9.
```

| Value | Displayed As | Notes |
|-------|-------------|-------|
| 00000 | `    0` | Rightmost `9` always shows |
| 00042 | `   42` | Leading zeros suppressed |
| 12345 | `12345` | No suppression needed |

```cobol
       05  ALL-SUPPRESS   PIC Z(5).
```

| Value | Displayed As |
|-------|-------------|
| 00000 | `     ` (all spaces) |
| 00042 | `   42` |

### * -- Zero Suppression with Asterisk (Check Protection)

Behaves like `Z`, but replaces leading zeros with `*` instead of spaces. Commonly used for check printing.

```cobol
       05  CHECK-AMT   PIC **(6).**
```

| Value | Displayed As |
|-------|-------------|
| 00000000 | `******.**` |
| 00012350 | `***123.50` |
| 12345678 | `123456.78` |

### + -- Plus Sign (Floating or Fixed)

As a **fixed** sign (single `+` at the beginning or end): displays `+` for positive values and `-` for negative values.

As a **floating** sign (multiple contiguous `+` symbols): the sign floats to the position just left of the first significant digit; leading zeros are replaced by spaces.

```cobol
       05  FIXED-SIGN    PIC +9(5).
       05  FLOAT-SIGN    PIC +(6).99.
```

Fixed `+9(5)`:

| Value | Displayed As |
|-------|-------------|
| +00042 | `+00042` |
| -00042 | `-00042` |

Floating `+(6).99`:

| Value | Displayed As |
|-------|-------------|
| +0012350 | `  +123.50` |
| -0012350 | `  -123.50` |
| +0000000 | `      +.00` |

### - -- Minus Sign (Floating or Fixed)

As a **fixed** sign: displays `-` for negative values and a space for positive values.

As a **floating** sign: behaves like floating `+` but positive values show a space instead of `+`.

```cobol
       05  FIXED-NEG    PIC 9(5)-.
       05  FLOAT-NEG    PIC -(6).99.
```

Fixed `9(5)-`:

| Value | Displayed As |
|-------|-------------|
| +00042 | `00042 ` |
| -00042 | `00042-` |

Floating `-(6).99`:

| Value | Displayed As |
|-------|-------------|
| +0012350 | `   123.50` |
| -0012350 | `  -123.50` |

### CR and DB -- Credit and Debit Symbols

`CR` and `DB` each occupy two character positions at the rightmost end of the picture string. They appear only when the value is negative; otherwise, two spaces are displayed.

```cobol
       05  BALANCE-CR   PIC 9(5).99CR.
       05  BALANCE-DB   PIC 9(5).99DB.
```

| Value | `9(5).99CR` | `9(5).99DB` |
|-------|------------|------------|
| +12350 | `00123.50  ` | `00123.50  ` |
| -12350 | `00123.50CR` | `00123.50DB` |

### $ -- Currency Sign

As a **fixed** currency sign (single `$` at the left): the `$` appears in a fixed position.

As a **floating** currency sign (multiple contiguous `$` symbols): the `$` floats to the position just left of the first significant digit.

```cobol
       05  FIXED-AMT    PIC $9(5).99.
       05  FLOAT-AMT    PIC $$(5).99.
```

Fixed `$9(5).99`:

| Value | Displayed As |
|-------|-------------|
| 0012350 | `$00123.50` |
| 9999999 | `$99999.99` |

Floating `$$(5).99`:

| Value | Displayed As |
|-------|-------------|
| 0012350 | `  $123.50` |
| 0000050 | `    $0.50` |
| 0000000 | `     $.00` |

The actual currency character is determined by the `CURRENCY SIGN` clause in the `SPECIAL-NAMES` paragraph. The default is `$`.

### . -- Decimal Point (Insertion)

The period (`.`) inserts an actual decimal point character in the edited result and defines the decimal alignment position for the source value.

```cobol
       05  EDITED-PRICE   PIC 9(3).99.
```

| Value | Displayed As |
|-------|-------------|
| 12350 | `123.50` |
| 00099 | `000.99` |

Only one `.` may appear in a picture string. The `DECIMAL-POINT IS COMMA` clause in `SPECIAL-NAMES` swaps the roles of `.` and `,`.

### , -- Comma (Insertion)

The comma (`,`) inserts a literal comma in the output at its position. It is commonly used as a thousands separator.

```cobol
       05  BIG-AMT   PIC 9(3),9(3),9(3).99.
```

| Value | Displayed As |
|-------|-------------|
| 00100000000 | `001,000,000.00` |

When used with zero suppression, commas in suppressed positions are replaced by the suppression character (space or `*`).

```cobol
       05  SUPPRESS-AMT   PIC ZZZ,ZZ9.99.
```

| Value | Displayed As |
|-------|-------------|
| 00000000 | `      0.00` |
| 00012350 | `    123.50` |
| 12345678 | `123,456.78` |

### B -- Space Insertion

Each `B` inserts a space at its position in the output.

```cobol
       05  PHONE-FMT   PIC 9(3)B9(3)B9(4).
```

| Value | Displayed As |
|-------|-------------|
| 5551234567 | `555 123 4567` |

### 0 -- Zero Insertion

Each `0` inserts a literal zero at its position.

```cobol
       05  EXTENDED-CODE   PIC 9(5)00.
```

| Value | Displayed As |
|-------|-------------|
| 12345 | `1234500` |

### / -- Slash Insertion

Each `/` inserts a literal slash at its position. Commonly used for date formatting.

```cobol
       05  DATE-FMT   PIC 99/99/9999.
```

| Value | Displayed As |
|-------|-------------|
| 03222026 | `03/22/2026` |
| 12311999 | `12/31/1999` |

---

## Picture String Rules

### Allowed Combinations

The following table summarizes which symbols may appear together in a single picture string:

| Category | Allowed Symbols |
|----------|----------------|
| Alphabetic | `A` only |
| Alphanumeric | `X` only (or a mix of `A`, `X`, `9` without editing symbols) |
| Numeric | `9`, `V`, `S`, `P` |
| Numeric Edited | `9` with one or more of: `Z`, `*`, `+`, `-`, `CR`, `DB`, `$`, `.`, `,`, `B`, `0`, `/` |
| Alphanumeric Edited | `A`, `X`, or `9` with `B`, `0`, or `/` (and the result is not numeric edited) |

### Prohibited Combinations

1. `Z` and `*` may not appear in the same picture string.
2. `+` and `-` may not appear in the same picture string.
3. `CR` and `DB` may not appear in the same picture string.
4. Only one of `CR`, `DB`, `+`, or `-` may serve as a sign indicator (floating or fixed) in a given picture.
5. `S` may not appear in an edited picture.
6. Only one `V` or `.` may appear.
7. `$` may not be used with floating `+`, floating `-`, or floating `Z`/`*` as the leftmost symbol in the same picture.

### Floating Insertion Symbols

A floating insertion symbol (`+`, `-`, `$`, `Z`, `*`) is one that appears two or more times consecutively before the first `9`. The floating string defines the positions in which leading zeros are suppressed, and the insertion character (`+`, `-`, or `$`) floats rightward to appear just before the first significant digit.

The number of digit positions represented by the floating string is one fewer than the number of symbols in the string (one position is reserved for the inserted character itself).

```
$$$$$.99   -> 4 digit positions + the $ = 5 symbols, represents up to 9999.99
+++,++9.99 -> 4 digit positions (the + symbols after first), plus fixed positions
```

### Size Calculation

The **size** of an edited item (in characters/bytes) is the number of character positions in the expanded picture string. Each symbol contributes to the size as follows:

| Symbol(s) | Bytes |
|-----------|-------|
| `9`, `A`, `X` | 1 each |
| `Z`, `*` | 1 each |
| `+`, `-` (each occurrence) | 1 each |
| `$` (each occurrence) | 1 each |
| `.`, `,` | 1 each |
| `B`, `0`, `/` | 1 each |
| `CR`, `DB` | 2 each |
| `V` | 0 (not stored) |
| `S` | 0 (unless SIGN IS SEPARATE, then 1) |
| `P` (each occurrence) | 0 (not stored) |

### Numeric Precision Limits

The maximum number of digit positions in a numeric (non-edited) item is **18** in COBOL-85 and COBOL 2002. This includes positions represented by `9` and `P` symbols. Some implementations extend this to 31 or 38 digits.

---

## De-editing

When a numeric edited item is used as a sending item in a `MOVE` to a numeric or numeric edited receiving item, the compiler performs **de-editing**: it extracts the numeric value from the edited representation, respecting sign, decimal point, and suppression characters.

```cobol
       05  EDITED-AMT   PIC $ZZ,ZZ9.99-.
       05  NUMERIC-AMT  PIC S9(7)V99.
       ...
       MOVE EDITED-AMT TO NUMERIC-AMT.
```

If `EDITED-AMT` contains `" $1,234.56-"`, the de-edited value moved to `NUMERIC-AMT` is -1234.56.

De-editing is defined in COBOL-85 and later. It applies only when a numeric edited item is the sending item and the receiving item is numeric or numeric edited.

---

## Comprehensive Examples

### Numeric Items

```cobol
       05  WS-QTY          PIC 9(5).
       05  WS-AMOUNT       PIC S9(7)V99.
       05  WS-RATE         PIC V9(4).
       05  WS-BIG          PIC 9(5)PP.
       05  WS-SMALL        PIC PP9(3).
```

| Item | Size (bytes) | Max Value | Example Internal | Represents |
|------|-------------|-----------|-----------------|-----------|
| `WS-QTY` | 5 | 99999 | `00042` | 42 |
| `WS-AMOUNT` | 9 | +9999999.99 | `012345678` | +123456.78 |
| `WS-RATE` | 4 | 0.9999 | `0725` | 0.0725 |
| `WS-BIG` | 5 | 9999900 | `99999` | 9999900 |
| `WS-SMALL` | 3 | 0.00999 | `999` | 0.00999 |

### Numeric Edited Items -- Zero Suppression

```cobol
       05  ED-A   PIC Z(5).
       05  ED-B   PIC Z(4)9.
       05  ED-C   PIC ZZ,ZZ9.
       05  ED-D   PIC *(5).
       05  ED-E   PIC **(4)9.
```

| Item | Value | Displayed As |
|------|-------|-------------|
| `ED-A` | 0 | `     ` |
| `ED-A` | 42 | `   42` |
| `ED-B` | 0 | `    0` |
| `ED-B` | 42 | `   42` |
| `ED-C` | 0 | `     0` |
| `ED-C` | 1234 | `  1,234` |
| `ED-C` | 123456 | `123,456` |
| `ED-D` | 0 | `*****` |
| `ED-D` | 42 | `***42` |
| `ED-E` | 0 | `****0` |
| `ED-E` | 12345 | `12345` |

### Numeric Edited Items -- Currency and Signs

```cobol
       05  ED-F   PIC $9(5).99.
       05  ED-G   PIC $$(5).99.
       05  ED-H   PIC $$(5).99CR.
       05  ED-I   PIC +(5)9.99.
       05  ED-J   PIC -(5)9.99.
       05  ED-K   PIC $$$,$$9.99-.
```

| Item | Value | Displayed As |
|------|-------|-------------|
| `ED-F` | 12350 | `$00123.50` |
| `ED-G` | 12350 | `  $123.50` |
| `ED-G` | 50 | `    $0.50` |
| `ED-G` | 0 | `     $.00` |
| `ED-H` | +12350 | `  $123.50  ` |
| `ED-H` | -12350 | `  $123.50CR` |
| `ED-I` | +12350 | ` +1234.50` |
| `ED-I` | -12350 | ` -1234.50` |
| `ED-I` | 0 | `     +0.00` |
| `ED-J` | +12350 | `  1234.50` |
| `ED-J` | -12350 | ` -1234.50` |
| `ED-K` | +12350 | `    $123.50 ` |
| `ED-K` | -12350 | `    $123.50-` |
| `ED-K` | +1234567 | `$12,345.67 ` |
| `ED-K` | -1234567 | `$12,345.67-` |

### Numeric Edited Items -- Insertion Characters

```cobol
       05  ED-DATE    PIC 99/99/9999.
       05  ED-SSN     PIC 999B99B9999.
       05  ED-PHONE   PIC 9(3)B9(3)B9(4).
       05  ED-ZERO    PIC 9(5)00.
```

| Item | Value | Displayed As |
|------|-------|-------------|
| `ED-DATE` | 03222026 | `03/22/2026` |
| `ED-SSN` | 123456789 | `123 45 6789` |
| `ED-PHONE` | 5551234567 | `555 123 4567` |
| `ED-ZERO` | 12345 | `1234500` |

### Alphanumeric and Alphabetic Items

```cobol
       05  CUST-NAME     PIC X(20).
       05  STATE-CODE    PIC AA.
       05  MIXED-FIELD   PIC X(5)BX(5).
```

| Item | Value | Displayed As |
|------|-------|-------------|
| `CUST-NAME` | `SMITH` | `SMITH               ` |
| `STATE-CODE` | `CA` | `CA` |
| `MIXED-FIELD` | `ABCDE12345` | `ABCDE 12345` |

### Complete Working Example

```cobol
       WORKING-STORAGE SECTION.
       01  WS-SALARY        PIC S9(7)V99  VALUE 45000.00.
       01  WS-EDITED-SAL    PIC $$$,$$$,$$9.99-.
       01  WS-REPORT-LINE.
           05  RPT-NAME     PIC X(20).
           05  FILLER       PIC X(3)   VALUE SPACES.
           05  RPT-SALARY   PIC $$$,$$$,$$9.99-.
           05  FILLER       PIC X(2)   VALUE SPACES.
           05  RPT-DATE     PIC 99/99/9999.

       PROCEDURE DIVISION.
           MOVE "DOE, JOHN"    TO RPT-NAME
           MOVE WS-SALARY      TO RPT-SALARY
           MOVE 03222026       TO RPT-DATE
           DISPLAY WS-REPORT-LINE.
      *>   Output: "DOE, JOHN              $45,000.00    03/22/2026"
```

---

## See Also

- [Data Division](index.md)
- [Level Numbers](level-numbers.md)
- [USAGE Clause](usage.md)
- [Condition Names (Level 88)](condition-names.md)
- [MOVE Statement](../procedure-division/data-movement/move.md)
- Numeric Intrinsic Functions
