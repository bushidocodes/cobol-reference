# SORT

The `SORT` statement sorts records in a sort-merge file according to specified keys.

- **Standard:** COBOL-61 Extended, COBOL-65, COBOL-68, COBOL-74, COBOL-85, COBOL 2002, COBOL 2014, COBOL 2023

## Syntax

```cobol
SORT sort-file-name
    { ON { ASCENDING | DESCENDING } KEY { data-name-1 } ... } ...
    [ WITH DUPLICATES IN ORDER ]
    [ COLLATING SEQUENCE IS alphabet-name ]
    { INPUT PROCEDURE IS procedure-name-1 [ { THROUGH | THRU } procedure-name-2 ]
    | USING { file-name-1 } ... }
    { OUTPUT PROCEDURE IS procedure-name-3 [ { THROUGH | THRU } procedure-name-4 ]
    | GIVING { file-name-2 } ... }
```

## Rules

### Sort-Merge File Description (SD)

The sort-merge file referenced by `sort-file-name` must be described with an `SD` entry in the Data Division. The `SD` entry defines the record layout for the sort work file, including the key fields referenced in the `SORT` statement.

```cobol
DATA DIVISION.
FILE SECTION.
SD  SORT-WORK-FILE.
01  SORT-RECORD.
    05  SORT-KEY-LAST-NAME   PIC X(30).
    05  SORT-KEY-FIRST-NAME  PIC X(20).
    05  SORT-DATA             PIC X(50).
```

The sort-merge file must not be opened with an `OPEN` statement. The `SORT` statement itself manages opening and closing of the sort work file.

### Sort Keys

At least one `ASCENDING KEY` or `DESCENDING KEY` phrase must be specified. Each `data-name` must reference a field defined within the record description of the `SD` entry.

When multiple keys are specified, the first key named is the major key, and subsequent keys are progressively minor keys. Ascending and descending order may be mixed across different key phrases.

```cobol
SORT SORT-FILE
    ON ASCENDING KEY DEPARTMENT-CODE
    ON DESCENDING KEY SALARY
    ON ASCENDING KEY EMPLOYEE-ID
```

### DUPLICATES Phrase

When `WITH DUPLICATES IN ORDER` is specified, records with equal key values retain their original input order. When omitted, the order of records with equal keys is undefined.

!!! note "COBOL-85"
    The `WITH DUPLICATES IN ORDER` phrase was introduced in COBOL-85.

### COLLATING SEQUENCE Phrase

The `COLLATING SEQUENCE` phrase specifies the collating sequence to use for comparing nonnumeric key fields. The `alphabet-name` must reference an alphabet defined in the `SPECIAL-NAMES` paragraph. When omitted, the program collating sequence applies.

### INPUT PROCEDURE vs. USING

The input source for the sort is specified by either the `INPUT PROCEDURE` phrase or the `USING` phrase.

**USING:** The runtime opens each `file-name`, reads all records, transfers them to the sort work file, and closes each file. The files must not already be open when the `SORT` statement executes.

**INPUT PROCEDURE:** The specified procedure (section or range of paragraphs) executes to supply records to the sort. Within the input procedure, the `RELEASE` statement transfers records to the sort work file one at a time. The input procedure may perform filtering, transformation, or record generation before releasing records.

A `RELEASE` statement may only appear within the range of an input procedure. The input procedure must not execute a `SORT` or `MERGE` statement.

### OUTPUT PROCEDURE vs. GIVING

The output destination for sorted records is specified by either the `OUTPUT PROCEDURE` phrase or the `GIVING` phrase.

**GIVING:** The runtime opens each `file-name`, writes all sorted records to it, and closes it. The files must not already be open when the `SORT` statement executes.

**OUTPUT PROCEDURE:** The specified procedure executes to retrieve sorted records. Within the output procedure, the `RETURN` statement retrieves the next sorted record from the sort work file. The output procedure may perform filtering, aggregation, or reporting on the sorted records.

A `RETURN` statement may only appear within the range of an output procedure. The output procedure must not execute a `SORT` or `MERGE` statement.

### SORT STATUS

If a file status data item is associated with the sort-merge file via the `FILE STATUS` clause in the [SELECT](../../environment-division/select.md) entry, the runtime sets the status value upon completion of the `SORT` statement. A value of `"00"` indicates successful completion.

!!! note "COBOL-85"
    File status reporting for sort-merge files was standardized in COBOL-85.

## Behavior

1. If `USING` is specified, the runtime opens each input file, reads and transfers all records to the sort work file, and closes each input file.
2. If `INPUT PROCEDURE` is specified, control passes to the input procedure. The procedure executes, releasing records to the sort work file via `RELEASE` statements.
3. The sort work file records are sorted according to the specified key hierarchy and order (ascending or descending).
4. If `GIVING` is specified, the runtime opens each output file, writes all sorted records to it, and closes each output file.
5. If `OUTPUT PROCEDURE` is specified, control passes to the output procedure. The procedure retrieves records from the sort work file via `RETURN` statements.
6. The sort status is set, and control passes to the next executable statement following the `SORT`.

If the sort operation fails, the sort status value indicates the nature of the failure.

## Examples

### USING and GIVING

The following example sorts an input file by last name in ascending order and writes the sorted result to an output file.

```cobol
       IDENTIFICATION DIVISION.
       PROGRAM-ID. SORT-EXAMPLE-1.

       ENVIRONMENT DIVISION.
       INPUT-OUTPUT SECTION.
       FILE-CONTROL.
           SELECT UNSORTED-FILE ASSIGN TO "UNSORTED.DAT"
               ORGANIZATION IS SEQUENTIAL.
           SELECT SORTED-FILE ASSIGN TO "SORTED.DAT"
               ORGANIZATION IS SEQUENTIAL.
           SELECT SORT-WORK ASSIGN TO "SORTWORK".

       DATA DIVISION.
       FILE SECTION.
       FD  UNSORTED-FILE.
       01  UNSORTED-RECORD          PIC X(80).

       FD  SORTED-FILE.
       01  SORTED-RECORD            PIC X(80).

       SD  SORT-WORK.
       01  SORT-RECORD.
           05  SR-LAST-NAME         PIC X(30).
           05  SR-FIRST-NAME        PIC X(20).
           05  SR-DATA              PIC X(30).

       PROCEDURE DIVISION.
           SORT SORT-WORK
               ON ASCENDING KEY SR-LAST-NAME
               USING UNSORTED-FILE
               GIVING SORTED-FILE
           STOP RUN.
```

### INPUT PROCEDURE and OUTPUT PROCEDURE

The following example uses an input procedure to filter records and an output procedure to count sorted records.

```cobol
       IDENTIFICATION DIVISION.
       PROGRAM-ID. SORT-EXAMPLE-2.

       ENVIRONMENT DIVISION.
       INPUT-OUTPUT SECTION.
       FILE-CONTROL.
           SELECT INPUT-FILE ASSIGN TO "INPUT.DAT"
               ORGANIZATION IS SEQUENTIAL.
           SELECT OUTPUT-FILE ASSIGN TO "OUTPUT.DAT"
               ORGANIZATION IS SEQUENTIAL.
           SELECT SORT-WORK ASSIGN TO "SORTWORK".

       DATA DIVISION.
       FILE SECTION.
       FD  INPUT-FILE.
       01  INPUT-RECORD.
           05  IN-DEPARTMENT        PIC X(4).
           05  IN-NAME              PIC X(30).
           05  IN-SALARY            PIC 9(7)V99.
           05  FILLER               PIC X(37).

       FD  OUTPUT-FILE.
       01  OUTPUT-RECORD            PIC X(80).

       SD  SORT-WORK.
       01  SORT-RECORD.
           05  SORT-DEPARTMENT      PIC X(4).
           05  SORT-NAME            PIC X(30).
           05  SORT-SALARY          PIC 9(7)V99.
           05  FILLER               PIC X(37).

       WORKING-STORAGE SECTION.
       01  WS-RECORD-COUNT          PIC 9(6) VALUE 0.
       01  WS-EOF                   PIC 9    VALUE 0.

       PROCEDURE DIVISION.
       MAIN-PROGRAM.
           SORT SORT-WORK
               ON ASCENDING KEY SORT-DEPARTMENT
               ON DESCENDING KEY SORT-SALARY
               INPUT PROCEDURE IS SELECT-RECORDS
               OUTPUT PROCEDURE IS WRITE-RECORDS
           DISPLAY "RECORDS WRITTEN: " WS-RECORD-COUNT
           STOP RUN.

       SELECT-RECORDS.
           OPEN INPUT INPUT-FILE
           PERFORM UNTIL WS-EOF = 1
               READ INPUT-FILE
                   AT END
                       MOVE 1 TO WS-EOF
                   NOT AT END
                       IF IN-DEPARTMENT NOT = "TERM"
                           RELEASE SORT-RECORD FROM INPUT-RECORD
                       END-IF
               END-READ
           END-PERFORM
           CLOSE INPUT-FILE.

       WRITE-RECORDS.
           OPEN OUTPUT OUTPUT-FILE
           MOVE 0 TO WS-EOF
           PERFORM UNTIL WS-EOF = 1
               RETURN SORT-WORK
                   AT END
                       MOVE 1 TO WS-EOF
                   NOT AT END
                       WRITE OUTPUT-RECORD FROM SORT-RECORD
                       ADD 1 TO WS-RECORD-COUNT
               END-RETURN
           END-PERFORM
           CLOSE OUTPUT-FILE.
```

## See Also

- [MERGE](merge.md) -- merges pre-sorted files
- [SELECT](../../environment-division/select.md) -- file control entry for file status
- [PICTURE](../../data-division/picture.md) -- data item format specification
