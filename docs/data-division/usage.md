# USAGE Clause

The USAGE clause specifies the internal representation and storage format of a data item.

**Standard:** COBOL-60, COBOL-61, COBOL-61 Extended, COBOL-65, COBOL-68, COBOL-85, COBOL 2002, COBOL 2014, COBOL 2023

---

## Syntax

```cobol
level-number  data-name  [PICTURE clause]  [USAGE [IS] usage-type].
```

```
usage-type := DISPLAY
            | BINARY
            | COMP | COMPUTATIONAL
            | COMP-1 | COMPUTATIONAL-1
            | COMP-2 | COMPUTATIONAL-2
            | COMP-3 | COMPUTATIONAL-3
            | PACKED-DECIMAL
            | COMP-4 | COMPUTATIONAL-4
            | COMP-5 | COMPUTATIONAL-5
            | INDEX
            | POINTER
            | NATIONAL
```

If the USAGE clause is omitted, the default is `DISPLAY`.

When specified on a group item, the USAGE clause applies to every elementary item within that group.

---

## USAGE Types

### DISPLAY

```cobol
05  WS-NAME    PIC X(20)  USAGE DISPLAY.
05  WS-AMOUNT  PIC 9(7)V99.
```

Each character or digit occupies one byte (one character position) in the native character set (EBCDIC or ASCII). This is the default USAGE for all data items.

- Numeric items in DISPLAY store each digit as one byte in its character representation (`'0'` through `'9'`).
- The operational sign, if present, is stored as an overpunch on the trailing (or leading) digit by default, or in a separate byte if `SIGN IS SEPARATE` is specified.
- Suitable for data that is displayed, printed, or written to text files.

**Storage size:** One byte per character position defined by the PICTURE clause, plus one byte if `SIGN IS SEPARATE`.

### BINARY (COMP / COMP-4)

```cobol
05  WS-COUNT   PIC S9(4)  USAGE BINARY.
05  WS-TOTAL   PIC S9(9)  COMP.
```

The item is stored as a binary (two's complement) integer. The number of digit positions in the PICTURE clause determines the storage allocated:

| PICTURE Digits | Storage | Value Range |
|---------------|---------|-------------|
| 1-4 | 2 bytes (halfword) | -9,999 to +9,999 |
| 5-9 | 4 bytes (fullword) | -999,999,999 to +999,999,999 |
| 10-18 | 8 bytes (doubleword) | -999,999,999,999,999,999 to +999,999,999,999,999,999 |

**Rules:**

1. A PICTURE clause is required and may contain only `9`, `S`, and `V`.
2. The PICTURE defines the precision, not the binary capacity. A `PIC S9(4) BINARY` item occupies 2 bytes (capable of holding -32768 to +32767) but the standard requires that values be limited to -9999 to +9999 after any arithmetic operation.
3. On mainframes, `COMP` and `COMP-4` are typically synonymous with `BINARY`. Behavior may differ across implementations.

**Alignment:** Binary items are typically aligned on halfword (2-byte) or fullword (4-byte) boundaries. The `SYNCHRONIZED` clause requests optimal boundary alignment.

### COMP-1 (COMPUTATIONAL-1) -- Single-Precision Floating Point

```cobol
05  WS-FLOAT  USAGE COMP-1.
```

The item is stored as a single-precision (32-bit) IEEE 754 floating-point number (or the platform's native single-precision format, such as IBM hexadecimal floating point on z/OS).

- No PICTURE clause is permitted.
- Storage size: 4 bytes.
- Approximate precision: 7 significant decimal digits.
- Used for scientific or engineering calculations where range is more important than exact decimal precision.

**Standard:** COMP-1 is not defined in the COBOL standard. It is a widespread vendor extension present in IBM COBOL, Micro Focus COBOL, GnuCOBOL, and most other implementations.

### COMP-2 (COMPUTATIONAL-2) -- Double-Precision Floating Point

```cobol
05  WS-DOUBLE  USAGE COMP-2.
```

The item is stored as a double-precision (64-bit) floating-point number.

- No PICTURE clause is permitted.
- Storage size: 8 bytes.
- Approximate precision: 15-16 significant decimal digits.
- Preferred over COMP-1 when greater precision is required.

**Standard:** COMP-2 is not defined in the COBOL standard. It is a widespread vendor extension.

### COMP-3 / PACKED-DECIMAL

```cobol
05  WS-AMOUNT  PIC S9(7)V99  USAGE COMP-3.
05  WS-PRICE   PIC 9(5)V99   PACKED-DECIMAL.
```

The item is stored in packed decimal format: each byte holds two decimal digits (one in each nibble), except for the trailing byte, whose low nibble holds the sign.

**Storage size formula:**

```
bytes = TRUNC((number-of-digits / 2) + 1)
```

| PICTURE Digits | Storage |
|---------------|---------|
| 1 | 1 byte |
| 2-3 | 2 bytes |
| 4-5 | 3 bytes |
| 6-7 | 4 bytes |
| 8-9 | 5 bytes |
| 10-11 | 6 bytes |
| 12-13 | 7 bytes |
| 14-15 | 8 bytes |
| 16-17 | 9 bytes |
| 18 | 10 bytes |

**Sign nibble values:**

| Nibble | Meaning |
|--------|---------|
| `C` (hex) | Positive |
| `D` (hex) | Negative |
| `F` (hex) | Unsigned |

**Example:** `PIC S9(5)V99 COMP-3` storing the value -12345.67:

```
Storage: 01 23 45 67 0D  (5 bytes)
         ^          ^^
         |          |+-- sign nibble (D = negative)
         |          +--- last digit (0, filler)
         +-------------- digits 1-2-3-4-5-6-7
```

Wait -- more precisely, for `S9(5)V99` with 7 digits:

```
Storage: 01 23 45 67 D  -> 4 bytes: 0x01 0x23 0x45 0x67 0x0D?
```

Corrected: 7 digits -> 4 bytes. Storage = `12 34 56 7D` (4 bytes).

- `PACKED-DECIMAL` is the standard name; `COMP-3` is the traditional name used by IBM and most vendors.
- Packed decimal is efficient for business arithmetic because it avoids binary-to-decimal conversion.

### COMP-5 (COMPUTATIONAL-5) -- Native Binary

```cobol
05  WS-NATIVE-INT  PIC S9(4)  USAGE COMP-5.
```

The item is stored in the platform's native binary format, and values may use the full range of the binary storage rather than being limited to the PICTURE-defined number of digits.

| PICTURE Digits | Storage | Full Range |
|---------------|---------|-----------|
| 1-4 | 2 bytes | -32,768 to +32,767 |
| 5-9 | 4 bytes | -2,147,483,648 to +2,147,483,647 |
| 10-18 | 8 bytes | -9,223,372,036,854,775,808 to +9,223,372,036,854,775,807 |

This differs from standard `BINARY` in that `COMP-5` allows the full binary range rather than truncating to the PICTURE's decimal precision.

**Standard:** COMP-5 is a vendor extension (IBM, Micro Focus, GnuCOBOL). It is essential for interfacing with C, Java, and operating system APIs where full native integer ranges are expected.

### INDEX

```cobol
05  WS-IDX  USAGE INDEX.
```

The item is stored as an index data item, typically occupying 4 or 8 bytes depending on the platform's native word size.

- No PICTURE clause is permitted.
- Used to hold index values set by `SET` statements or `INDEXED BY` phrases.
- An index item can only be used in `SET`, `SEARCH`, or relational condition statements involving other index items or index names.
- Index values represent displacement (byte offset), not occurrence numbers.

### POINTER

```cobol
05  WS-PTR  USAGE POINTER.
```

The item stores a memory address (pointer).

- No PICTURE clause is permitted.
- Storage size: typically 4 bytes (32-bit systems) or 8 bytes (64-bit systems).
- Used primarily for interfacing with non-COBOL routines and for `PROCEDURE-POINTER` or `FUNCTION-POINTER` types.
- Pointer items may be compared with relational operators and may be set with the `SET` statement.
- The special value `NULL` (or `NULLS`) represents a pointer that does not reference any data.

**Standard:** COBOL 2002 and later.

### NATIONAL

```cobol
05  WS-UNAME  PIC N(20)  USAGE NATIONAL.
```

The item is stored in national character encoding (typically UTF-16).

- Each character position occupies 2 bytes.
- The PICTURE clause uses the symbol `N` to represent national character positions.
- Used for Unicode / double-byte character data.

**Standard:** COBOL 2002 and later.

---

## Vendor Variations

| USAGE | IBM z/OS COBOL | Micro Focus | GnuCOBOL | Fujitsu |
|-------|---------------|-------------|-----------|---------|
| COMP / COMP-4 | Big-endian binary | Native-endian binary | Native-endian binary | Big-endian binary |
| COMP-1 | Hex float (default) or IEEE | IEEE 754 single | IEEE 754 single | IEEE 754 single |
| COMP-2 | Hex float (default) or IEEE | IEEE 754 double | IEEE 754 double | IEEE 754 double |
| COMP-3 | Packed decimal | Packed decimal | Packed decimal | Packed decimal |
| COMP-5 | Native binary (full range) | Native binary (full range) | Native binary (full range) | Not supported |
| COMP-6 | Not supported | Unsigned packed | Not supported | Not supported |
| COMP-X | Not supported | Unsigned binary (byte-sized) | Unsigned binary | Not supported |

---

## Rules

1. If USAGE is specified on a group item, it applies to every elementary item within that group. The USAGE clause may not be specified on an elementary item if it contradicts the group-level USAGE.
2. Items with USAGE other than DISPLAY (or NATIONAL) may not be referenced in `ACCEPT`, `DISPLAY`, `STRING`, `UNSTRING`, or `INSPECT` statements (except by vendor extension).
3. Numeric items used in arithmetic statements (`ADD`, `SUBTRACT`, `MULTIPLY`, `DIVIDE`, `COMPUTE`) may have any numeric USAGE; the compiler generates necessary conversions.
4. For performance, use `BINARY` or `COMP-5` for subscripts, counters, and loop variables. Use `COMP-3` for business arithmetic. Use `DISPLAY` for I/O records.
5. Items with USAGE `INDEX` or `POINTER` may not have a PICTURE, VALUE, or BLANK WHEN ZERO clause.

## Alignment and the SYNCHRONIZED Clause

```cobol
05  WS-BIN  PIC S9(9) BINARY SYNCHRONIZED.
```

The `SYNCHRONIZED` (`SYNC`) clause requests that a binary item be aligned on its natural boundary (halfword, fullword, or doubleword). Without `SYNC`, binary items may be placed at any byte offset, potentially causing performance penalties due to unaligned access. The compiler may insert slack bytes (implicit `FILLER`) to achieve alignment.

---

## Examples

### Mixed Usage in a Record

```cobol
       01  ORDER-RECORD.
           05  ORD-ID          PIC 9(8)       USAGE DISPLAY.
           05  ORD-QTY         PIC S9(4)      USAGE BINARY.
           05  ORD-PRICE       PIC S9(5)V99   USAGE COMP-3.
           05  ORD-TOTAL       PIC S9(7)V99   USAGE COMP-3.
           05  ORD-DESC        PIC X(30)      USAGE DISPLAY.
           05  ORD-FLAGS       PIC S9(9)      USAGE COMP-5.
```

| Field | USAGE | Storage |
|-------|-------|---------|
| `ORD-ID` | DISPLAY | 8 bytes |
| `ORD-QTY` | BINARY | 2 bytes |
| `ORD-PRICE` | COMP-3 | 4 bytes |
| `ORD-TOTAL` | COMP-3 | 5 bytes |
| `ORD-DESC` | DISPLAY | 30 bytes |
| `ORD-FLAGS` | COMP-5 | 4 bytes |

### Group-Level USAGE

```cobol
       01  PACKED-COUNTERS  USAGE COMP-3.
           05  CTR-READS       PIC 9(7).
           05  CTR-WRITES      PIC 9(7).
           05  CTR-ERRORS      PIC 9(5).
```

All three elementary items inherit `USAGE COMP-3` from the group.

### Pointer Usage

```cobol
       WORKING-STORAGE SECTION.
       01  WS-DATA-PTR    USAGE POINTER  VALUE NULL.
       01  WS-BUFFER      PIC X(1024).

       LINKAGE SECTION.
       01  LK-EXT-DATA    PIC X(256).

       PROCEDURE DIVISION.
           SET ADDRESS OF LK-EXT-DATA TO WS-DATA-PTR
           IF WS-DATA-PTR = NULL
               DISPLAY "Pointer is null"
           END-IF.
```

---

## See Also

- [Data Division](index.md)
- [PICTURE Clause](picture.md)
- [Level Numbers](level-numbers.md)
- [OCCURS Clause](occurs.md)
- SYNCHRONIZED Clause
- SIGN Clause
