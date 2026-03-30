# Sort Description (SD) Entry

The Sort Description entry defines the records used by the SORT and MERGE statements. It is analogous to an FD entry but describes a sort work file rather than an I/O file.

- **Standard:** COBOL-68, COBOL-85, COBOL 2002, COBOL 2014, COBOL 2023
- **Division:** Data Division (File Section)

---

## Syntax

```cobol
SD  sort-file-name
    [ RECORD { CONTAINS integer-1 CHARACTERS
             | CONTAINS integer-2 TO integer-3 CHARACTERS
             | IS VARYING IN SIZE
                 [ FROM integer-4 ] [ TO integer-5 ] CHARACTERS
                 [ DEPENDING ON data-name-1 ] } ]
    [ DATA { RECORD IS | RECORDS ARE }
        data-name-2 [ data-name-3 ] ... ]
    .
```

The SD entry is followed by one or more record description entries (01-level items) describing the sort record layout.

---

## Rules

1. Every sort or merge file named in a SORT or MERGE statement must have a corresponding SD entry.
2. The sort file must also have a SELECT clause in the FILE-CONTROL paragraph, but it is not opened or closed by the programmer — the SORT/MERGE statement handles this automatically.
3. SD entries do **not** have BLOCK CONTAINS, LABEL RECORDS, VALUE OF, LINAGE, or RECORDING MODE clauses (sort work files are managed entirely by the sort subsystem).
4. The record description following the SD must include the sort key fields referenced in the SORT/MERGE statement.
5. RECORD CONTAINS is optional; if omitted, record size is determined from the record descriptions.

---

## Examples

### Basic Sort File

```cobol
       ENVIRONMENT DIVISION.
       INPUT-OUTPUT SECTION.
       FILE-CONTROL.
           SELECT SORT-WORK ASSIGN TO "SORTWORK".

       DATA DIVISION.
       FILE SECTION.
       SD  SORT-WORK.
       01  SORT-RECORD.
           05  SORT-KEY-NAME    PIC X(30).
           05  SORT-KEY-DATE    PIC 9(8).
           05  SORT-DATA        PIC X(42).

       PROCEDURE DIVISION.
           SORT SORT-WORK
               ON ASCENDING KEY SORT-KEY-NAME
               ON DESCENDING KEY SORT-KEY-DATE
               USING INPUT-FILE
               GIVING OUTPUT-FILE.
```

### Sort with Input/Output Procedures

```cobol
       SD  SORT-FILE.
       01  SORT-REC.
           05  SR-DEPT         PIC X(5).
           05  SR-NAME         PIC X(25).
           05  SR-SALARY       PIC 9(7)V99.

       PROCEDURE DIVISION.
           SORT SORT-FILE
               ON ASCENDING KEY SR-DEPT SR-NAME
               INPUT PROCEDURE IS FILTER-SECTION
               OUTPUT PROCEDURE IS REPORT-SECTION.
```

---

## See Also

- [SORT](../procedure-division/sort-merge/sort.md) -- sort statement
- [MERGE](../procedure-division/sort-merge/merge.md) -- merge statement
- [RELEASE](../procedure-division/sort-merge/release.md) -- send records to sort
- [RETURN](../procedure-division/sort-merge/return.md) -- retrieve sorted records
- [FD Entry](fd-entry.md) -- file description entry
