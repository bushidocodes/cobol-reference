# File Description (FD) Entry

The File Description entry defines the physical characteristics of a file — its blocking, record size, label handling, and associated records. Every file named in a SELECT clause must have a corresponding FD entry in the FILE SECTION.

- **Standard:** COBOL-60, COBOL-68, COBOL-85, COBOL 2002, COBOL 2014, COBOL 2023
- **Division:** Data Division (File Section)

---

## Syntax

```cobol
FD  file-name
    [ IS EXTERNAL ]
    [ IS GLOBAL ]
    [ BLOCK CONTAINS [ integer-1 TO ] integer-2
        { RECORDS | CHARACTERS } ]
    [ RECORD { CONTAINS integer-3 CHARACTERS
             | CONTAINS integer-4 TO integer-5 CHARACTERS
             | IS VARYING IN SIZE
                 [ FROM integer-6 ] [ TO integer-7 ] CHARACTERS
                 [ DEPENDING ON data-name-1 ] } ]
    [ LABEL { RECORD IS  | RECORDS ARE }
        { STANDARD | OMITTED } ]
    [ VALUE OF data-name-2 IS { data-name-3 | literal-1 }
        [ data-name-4 IS { data-name-5 | literal-2 } ] ... ]
    [ DATA { RECORD IS  | RECORDS ARE }
        data-name-6 [ data-name-7 ] ... ]
    [ LINAGE IS { data-name-8 | integer-8 } LINES
        [ WITH FOOTING AT { data-name-9 | integer-9 } ]
        [ LINES AT TOP { data-name-10 | integer-10 } ]
        [ LINES AT BOTTOM { data-name-11 | integer-11 } ] ]
    [ RECORDING MODE IS mode ]
    [ CODE-SET IS alphabet-name ]
    .
```

The FD entry is followed by one or more record description entries (01-level items).

---

## Clauses

### BLOCK CONTAINS

Specifies the physical grouping of logical records into blocks.

```cobol
BLOCK CONTAINS 10 RECORDS
BLOCK CONTAINS 800 CHARACTERS
BLOCK CONTAINS 5 TO 10 RECORDS
```

- If RECORDS is specified, `integer-2` is the number of records per block.
- If CHARACTERS is specified, `integer-2` is the block size in characters.
- If omitted, each block contains exactly one record.
- Variable-length blocks use the `integer-1 TO integer-2` form.

### RECORD CONTAINS

Specifies the size of logical records in the file.

```cobol
RECORD CONTAINS 80 CHARACTERS
RECORD CONTAINS 50 TO 200 CHARACTERS
RECORD IS VARYING IN SIZE FROM 50 TO 200 CHARACTERS
    DEPENDING ON WS-REC-LEN
```

- Fixed-length: `RECORD CONTAINS integer CHARACTERS`
- Variable-length: `RECORD CONTAINS integer TO integer CHARACTERS`
- `VARYING ... DEPENDING ON` sets the data item to the actual record length on READ.

### LABEL RECORDS

Specifies whether the file has standard labels, no labels, or user-defined labels.

```cobol
LABEL RECORDS ARE STANDARD
LABEL RECORDS ARE OMITTED
```

!!! warning "Obsolete"
    The LABEL RECORDS clause was marked obsolete in COBOL-85. Most compilers
    still accept it for backward compatibility. Omitting it is equivalent to
    STANDARD on most systems.

### VALUE OF

Specifies values for implementor-defined items in file labels.

!!! warning "Obsolete"
    The VALUE OF clause was marked obsolete in COBOL-85.

### DATA RECORDS

Lists the record names associated with the file.

```cobol
DATA RECORDS ARE CUSTOMER-RECORD, HEADER-RECORD
```

!!! warning "Obsolete"
    The DATA RECORDS clause was marked obsolete in COBOL-85.
    The record descriptions following the FD entry serve this purpose.

### RECORDING MODE (IBM Extension)

Specifies the format of records in the file.

```cobol
RECORDING MODE IS F    *> Fixed-length
RECORDING MODE IS V    *> Variable-length
RECORDING MODE IS U    *> Undefined
RECORDING MODE IS S    *> Spanned
```

### CODE-SET

Specifies the character code set used for the file.

```cobol
CODE-SET IS ASCII-CODE
```

The alphabet-name must be defined in the SPECIAL-NAMES paragraph.

### LINAGE

See [LINAGE Clause](linage.md) for full details on page-structured print files.

### EXTERNAL and GLOBAL

- `IS EXTERNAL` — the file is shared across separately compiled programs in the same run unit.
- `IS GLOBAL` — the file is visible to nested programs.

---

## Examples

### Sequential File

```cobol
FD  TRANSACTION-FILE
    BLOCK CONTAINS 20 RECORDS
    RECORD CONTAINS 120 CHARACTERS.
01  TRANSACTION-RECORD.
    05  TXN-DATE       PIC 9(8).
    05  TXN-AMOUNT     PIC S9(9)V99.
    05  TXN-DESC       PIC X(99).
```

### Indexed File with Variable Records

```cobol
FD  CUSTOMER-FILE
    RECORD IS VARYING IN SIZE FROM 50 TO 500 CHARACTERS
        DEPENDING ON WS-CUST-REC-LEN.
01  CUSTOMER-RECORD.
    05  CUST-ID        PIC X(10).
    05  CUST-NAME      PIC X(30).
    05  CUST-DATA      PIC X(460).
```

### Print File with LINAGE

```cobol
FD  REPORT-FILE
    LINAGE IS 60 LINES
    WITH FOOTING AT 55
    LINES AT TOP 3
    LINES AT BOTTOM 3.
01  REPORT-LINE        PIC X(132).
```

### Minimal FD (Modern Style)

```cobol
FD  INPUT-FILE.
01  INPUT-RECORD       PIC X(80).
```

In modern COBOL, only the FD header and record descriptions are required. All clauses are optional.

---

## See Also

- [SELECT](../environment-division/select.md) -- file-control entries
- [SD Entry](sd-entry.md) -- sort/merge file descriptions
- [LINAGE Clause](linage.md) -- page structure for print files
- [OPEN](../procedure-division/io/open.md) -- opening files
- [File Status Codes](../appendices/file-status-codes.md)
