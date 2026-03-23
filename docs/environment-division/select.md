# SELECT Statement

The `SELECT` statement appears in the `FILE-CONTROL` paragraph of the Environment Division. It associates a file-name used in the program with an external file, and specifies the file's organization, access mode, keys, and status handling.

**Standard:** COBOL-60, COBOL-61, COBOL-61 Extended, COBOL-65, COBOL-68, COBOL-74, COBOL-85, COBOL 2002, COBOL 2014, COBOL 2023

---

## Syntax

```cobol
SELECT [OPTIONAL] file-name-1

    ASSIGN TO { literal-1 | implementor-name-1 | data-name-1 }

    [ORGANIZATION IS { SEQUENTIAL | LINE SEQUENTIAL | INDEXED | RELATIVE }]

    [ACCESS MODE IS { SEQUENTIAL | RANDOM | DYNAMIC }]

    [RECORD KEY IS data-name-2]

    [ALTERNATE RECORD KEY IS data-name-3 [WITH DUPLICATES]] ...

    [RELATIVE KEY IS data-name-4]

    [FILE STATUS IS data-name-5]

    [RESERVE integer-1 {AREA | AREAS}]

    [PADDING CHARACTER IS { data-name-6 | literal-2 }]

    [RECORD DELIMITER IS { STANDARD-1 | implementor-name-2 }]

    .
```

Each `SELECT` statement is terminated by a period. Multiple `SELECT` statements may appear within the same `FILE-CONTROL` paragraph.

---

## Clauses

### OPTIONAL

When `OPTIONAL` is specified, the file need not be present when the program executes an `OPEN` statement with `INPUT` or `I-O` mode. If the file is absent, the first `READ` returns an at-end condition. If the file is opened for `I-O`, it is created when the first `WRITE` is executed.

`OPTIONAL` is valid only for files opened in `INPUT`, `I-O`, or `EXTEND` mode.

### ASSIGN

The `ASSIGN` clause specifies the external file to which the internal file-name is mapped.

```cobol
ASSIGN TO "CUSTFILE"
ASSIGN TO WS-FILENAME
ASSIGN TO DISK
```

The operand may be:
- A nonnumeric literal specifying a file path or system name.
- A data-name containing the file path at run time (dynamic file assignment).
- An implementor-name specifying a device or system-defined file type (e.g., `DISK`, `PRINTER`, `KEYBOARD`, `DISPLAY`).

The interpretation of the `ASSIGN` operand is implementor-defined and varies significantly between compilers and platforms.

### ORGANIZATION

The `ORGANIZATION` clause specifies the logical structure of the file.

| Organization | Description |
|---|---|
| `SEQUENTIAL` | Records are stored and accessed in the order they are written. This is the default if `ORGANIZATION` is omitted. |
| `LINE SEQUENTIAL` | Records are stored as text lines delimited by line terminators. This is a common extension; not part of the COBOL standard prior to COBOL 2002. |
| `INDEXED` | Records are stored with one or more key fields. Records may be accessed sequentially (in key order), randomly (by key value), or dynamically (both). |
| `RELATIVE` | Records are stored by relative record number. Records may be accessed sequentially, randomly (by record number), or dynamically. |

### ACCESS MODE

The `ACCESS MODE` clause specifies how records in the file are retrieved or stored.

| Access Mode | Description | Valid Organizations |
|---|---|---|
| `SEQUENTIAL` | Records are accessed in the order determined by the file organization (physical order for sequential files, key order for indexed files, ascending relative record number for relative files). This is the default. | All |
| `RANDOM` | Records are accessed by the value of a key (indexed) or relative record number (relative). Each I/O statement accesses one individually identified record. | `INDEXED`, `RELATIVE` |
| `DYNAMIC` | Both sequential and random access are permitted within the same program. The program may switch between sequential reads (`READ ... NEXT`) and random reads (`READ ... KEY IS`). | `INDEXED`, `RELATIVE` |

### RECORD KEY

The `RECORD KEY` clause specifies the prime record key for an indexed file. The data-name must reference a field defined within the file's record description in the Data Division.

```cobol
RECORD KEY IS CUST-ID
```

The prime record key uniquely identifies each record in the file. Duplicate values in the prime key are not permitted.

### ALTERNATE RECORD KEY

The `ALTERNATE RECORD KEY` clause specifies one or more alternate keys for an indexed file. Multiple alternate keys may be defined by repeating the clause.

```cobol
ALTERNATE RECORD KEY IS CUST-NAME WITH DUPLICATES
ALTERNATE RECORD KEY IS CUST-REGION
```

When `WITH DUPLICATES` is specified, multiple records may have the same value for that alternate key. Without `WITH DUPLICATES`, each alternate key value must be unique.

### RELATIVE KEY

The `RELATIVE KEY` clause specifies a data item that contains the relative record number for a relative file. The data-name must reference an unsigned integer data item that is not part of the file's record description.

```cobol
RELATIVE KEY IS WS-REC-NUM
```

The `RELATIVE KEY` clause is required when `ACCESS MODE IS RANDOM` or `ACCESS MODE IS DYNAMIC` is specified for a relative file. For sequential access, it is optional; if specified, the relative record number of each record read is placed in the data item.

### FILE STATUS

The `FILE STATUS` clause specifies a data item that receives a two-character status code after each I/O operation on the file.

```cobol
FILE STATUS IS WS-FILE-STATUS
```

The data item is typically defined as `PIC XX` or `PIC 99` in `WORKING-STORAGE`. Common status values include:

| Status | Meaning |
|---|---|
| `00` | Successful completion |
| `02` | Successful completion, duplicate key detected |
| `10` | End of file (at end) |
| `22` | Duplicate key (write) |
| `23` | Record not found |
| `30` | Permanent I/O error |
| `35` | File not found on OPEN |
| `39` | File attribute conflict |
| `41` | File already open |
| `42` | File not open |
| `46` | Read failed, no valid next record |
| `47` | READ on file not opened INPUT or I-O |
| `48` | WRITE on file not opened OUTPUT, I-O, or EXTEND |

### RESERVE

The `RESERVE` clause specifies the number of input/output buffer areas allocated for the file. If omitted, the compiler uses a default value.

```cobol
RESERVE 2 AREAS
```

### PADDING CHARACTER

The `PADDING CHARACTER` clause specifies the character used to pad blocks of sequential files on external media. It is relevant only for `ORGANIZATION IS SEQUENTIAL` files.

```cobol
PADDING CHARACTER IS SPACES
```

This clause is obsolete in COBOL 2002.

### RECORD DELIMITER

The `RECORD DELIMITER` clause specifies the method of determining the length of a variable-length record on an external medium. It is relevant only for `ORGANIZATION IS SEQUENTIAL` files.

```cobol
RECORD DELIMITER IS STANDARD-1
```

This clause is obsolete in COBOL 2002.

---

## Examples

### Sequential file

```cobol
       FILE-CONTROL.
           SELECT TRANSACTION-FILE
               ASSIGN TO "TRANS.DAT"
               ORGANIZATION IS SEQUENTIAL
               ACCESS MODE IS SEQUENTIAL
               FILE STATUS IS WS-TRANS-STATUS.
```

### Line sequential file

```cobol
       FILE-CONTROL.
           SELECT LOG-FILE
               ASSIGN TO "APPLICATION.LOG"
               ORGANIZATION IS LINE SEQUENTIAL
               FILE STATUS IS WS-LOG-STATUS.
```

### Indexed file with alternate keys

```cobol
       FILE-CONTROL.
           SELECT CUSTOMER-FILE
               ASSIGN TO "CUSTOMER.DAT"
               ORGANIZATION IS INDEXED
               ACCESS MODE IS DYNAMIC
               RECORD KEY IS CUST-ID
               ALTERNATE RECORD KEY IS CUST-NAME
                   WITH DUPLICATES
               ALTERNATE RECORD KEY IS CUST-REGION
                   WITH DUPLICATES
               FILE STATUS IS WS-CUST-STATUS.
```

### Relative file

```cobol
       FILE-CONTROL.
           SELECT SLOT-FILE
               ASSIGN TO "SLOTS.DAT"
               ORGANIZATION IS RELATIVE
               ACCESS MODE IS RANDOM
               RELATIVE KEY IS WS-SLOT-NUM
               FILE STATUS IS WS-SLOT-STATUS.
```

### Dynamic file assignment

```cobol
       FILE-CONTROL.
           SELECT REPORT-FILE
               ASSIGN TO WS-REPORT-PATH
               ORGANIZATION IS SEQUENTIAL
               FILE STATUS IS WS-RPT-STATUS.
```

In this case, `WS-REPORT-PATH` is a `WORKING-STORAGE` data item whose value is set at run time before the `OPEN` statement.

### Optional input file

```cobol
       FILE-CONTROL.
           SELECT OPTIONAL CONFIG-FILE
               ASSIGN TO "CONFIG.DAT"
               ORGANIZATION IS LINE SEQUENTIAL
               FILE STATUS IS WS-CFG-STATUS.
```

When `CONFIG.DAT` does not exist, an `OPEN INPUT` succeeds and the first `READ` returns an at-end condition.

---

## See also

- [Environment Division](index.md)
- [READ](../procedure-division/io/read.md)
- WRITE
- OPEN
- CLOSE
- [Data Division](../data-division/index.md)
