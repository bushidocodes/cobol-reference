# MERGE

The `MERGE` statement combines two or more identically sequenced (pre-sorted) files on a set of specified keys, producing a single output in sorted order.

- **Standard:** COBOL-74, COBOL-85, COBOL 2002, COBOL 2014, COBOL 2023

## Syntax

```cobol
MERGE sort-file-name
    { ON { ASCENDING | DESCENDING } KEY { data-name-1 } ... } ...
    [ COLLATING SEQUENCE IS alphabet-name ]
    USING file-name-1 file-name-2 { file-name-3 } ...
    { OUTPUT PROCEDURE IS procedure-name-1 [ { THROUGH | THRU } procedure-name-2 ]
    | GIVING { file-name-4 } ... }
```

## Rules

### Sort-Merge File Description (SD)

The sort-merge file referenced by `sort-file-name` must be described with an `SD` entry in the Data Division. The `SD` entry defines the record layout for the merge work file, including the key fields referenced in the `MERGE` statement.

The sort-merge file must not be opened with an `OPEN` statement. The `MERGE` statement manages the work file internally.

### Pre-Sorted Input Requirement

Each input file specified in the `USING` phrase must already be sorted in the order described by the `ASCENDING KEY` and `DESCENDING KEY` phrases. If an input file is not in the correct order, the results of the merge operation are undefined.

This is the principal distinction from the [SORT](sort.md) statement, which accepts unsorted input. `MERGE` does not rearrange records within an input file; it interleaves records from multiple files that are each already in the correct sequence.

### Merge Keys

At least one `ASCENDING KEY` or `DESCENDING KEY` phrase must be specified. Each `data-name` must reference a field defined within the record description of the `SD` entry.

When multiple keys are specified, the first key named is the major key, and subsequent keys are progressively minor keys.

### USING Phrase

At least two input files must be specified. Each file must be described with an `FD` entry in the Data Division and must not already be open when the `MERGE` statement executes. The runtime opens each input file, reads all records, merges them into the work file, and closes each input file.

!!! note "COBOL 2002"
    COBOL 2002 clarified that MERGE supports more than two input files. Most implementations supported multiple input files prior to this clarification.

### COLLATING SEQUENCE Phrase

The `COLLATING SEQUENCE` phrase specifies the collating sequence for comparing nonnumeric key fields. The `alphabet-name` must reference an alphabet defined in the `SPECIAL-NAMES` paragraph. When omitted, the program collating sequence applies.

### OUTPUT PROCEDURE vs. GIVING

The output destination is specified by either the `OUTPUT PROCEDURE` phrase or the `GIVING` phrase.

**GIVING:** The runtime opens each output file, writes all merged records to it, and closes it. The files must not already be open when the `MERGE` statement executes.

**OUTPUT PROCEDURE:** The specified procedure executes to retrieve merged records. Within the output procedure, the `RETURN` statement retrieves the next merged record from the sort-merge file. The output procedure may perform filtering, aggregation, or reporting on the merged records.

A `RETURN` statement may only appear within the range of an output procedure. The output procedure must not execute a `SORT` or `MERGE` statement.

### Differences from SORT

| Feature | SORT | MERGE |
|---|---|---|
| Input | Unsorted records | Pre-sorted files |
| INPUT PROCEDURE | Supported | Not supported |
| USING minimum files | One | Two |
| Record reordering | Full sort performed | Interleave only |
| Standard introduced | COBOL-61 Extended | COBOL-74 |

### MERGE STATUS

If a file status data item is associated with the sort-merge file via the `FILE STATUS` clause in the [SELECT](../../environment-division/select.md) entry, the runtime sets the status value upon completion of the `MERGE` statement. A value of `"00"` indicates successful completion.

## Behavior

1. The runtime opens each input file specified in the `USING` phrase.
2. Records from all input files are merged based on the specified key hierarchy and order. For records with equal keys across different input files, the record from the file named first in the `USING` phrase appears first in the merged output.
3. The runtime closes each input file.
4. If `GIVING` is specified, the runtime opens each output file, writes all merged records to it, and closes each output file.
5. If `OUTPUT PROCEDURE` is specified, control passes to the output procedure. The procedure retrieves records via `RETURN` statements.
6. The merge status is set, and control passes to the next executable statement following the `MERGE`.

## Examples

### USING and GIVING

The following example merges three pre-sorted regional sales files into a single consolidated file.

```cobol
       IDENTIFICATION DIVISION.
       PROGRAM-ID. MERGE-EXAMPLE-1.

       ENVIRONMENT DIVISION.
       INPUT-OUTPUT SECTION.
       FILE-CONTROL.
           SELECT NORTH-FILE ASSIGN TO "NORTH.DAT"
               ORGANIZATION IS SEQUENTIAL.
           SELECT SOUTH-FILE ASSIGN TO "SOUTH.DAT"
               ORGANIZATION IS SEQUENTIAL.
           SELECT WEST-FILE ASSIGN TO "WEST.DAT"
               ORGANIZATION IS SEQUENTIAL.
           SELECT MERGED-FILE ASSIGN TO "ALLSALES.DAT"
               ORGANIZATION IS SEQUENTIAL.
           SELECT MERGE-WORK ASSIGN TO "MERGEWORK".

       DATA DIVISION.
       FILE SECTION.
       FD  NORTH-FILE.
       01  NORTH-RECORD             PIC X(80).

       FD  SOUTH-FILE.
       01  SOUTH-RECORD             PIC X(80).

       FD  WEST-FILE.
       01  WEST-RECORD              PIC X(80).

       FD  MERGED-FILE.
       01  MERGED-RECORD            PIC X(80).

       SD  MERGE-WORK.
       01  MERGE-RECORD.
           05  MR-SALES-DATE        PIC 9(8).
           05  MR-REGION            PIC X(5).
           05  MR-AMOUNT            PIC 9(9)V99.
           05  FILLER               PIC X(56).

       PROCEDURE DIVISION.
           MERGE MERGE-WORK
               ON ASCENDING KEY MR-SALES-DATE
               USING NORTH-FILE SOUTH-FILE WEST-FILE
               GIVING MERGED-FILE
           STOP RUN.
```

### OUTPUT PROCEDURE

The following example merges two pre-sorted files and uses an output procedure to produce a summary report.

```cobol
       IDENTIFICATION DIVISION.
       PROGRAM-ID. MERGE-EXAMPLE-2.

       ENVIRONMENT DIVISION.
       INPUT-OUTPUT SECTION.
       FILE-CONTROL.
           SELECT FILE-A ASSIGN TO "FILEA.DAT"
               ORGANIZATION IS SEQUENTIAL.
           SELECT FILE-B ASSIGN TO "FILEB.DAT"
               ORGANIZATION IS SEQUENTIAL.
           SELECT REPORT-FILE ASSIGN TO "REPORT.DAT"
               ORGANIZATION IS SEQUENTIAL.
           SELECT MERGE-WORK ASSIGN TO "MERGEWORK".

       DATA DIVISION.
       FILE SECTION.
       FD  FILE-A.
       01  REC-A                    PIC X(60).

       FD  FILE-B.
       01  REC-B                    PIC X(60).

       FD  REPORT-FILE.
       01  REPORT-RECORD            PIC X(80).

       SD  MERGE-WORK.
       01  MERGE-RECORD.
           05  MR-ACCOUNT-NO        PIC 9(8).
           05  MR-TRANS-TYPE        PIC X(2).
           05  MR-AMOUNT            PIC 9(7)V99.
           05  FILLER               PIC X(41).

       WORKING-STORAGE SECTION.
       01  WS-EOF                   PIC 9     VALUE 0.
       01  WS-TOTAL                 PIC 9(11)V99 VALUE 0.
       01  WS-COUNT                 PIC 9(6)  VALUE 0.
       01  WS-REPORT-LINE           PIC X(80).

       PROCEDURE DIVISION.
       MAIN-PROGRAM.
           MERGE MERGE-WORK
               ON ASCENDING KEY MR-ACCOUNT-NO
               USING FILE-A FILE-B
               OUTPUT PROCEDURE IS PRODUCE-REPORT
           STOP RUN.

       PRODUCE-REPORT.
           OPEN OUTPUT REPORT-FILE
           PERFORM UNTIL WS-EOF = 1
               RETURN MERGE-WORK
                   AT END
                       MOVE 1 TO WS-EOF
                   NOT AT END
                       ADD MR-AMOUNT TO WS-TOTAL
                       ADD 1 TO WS-COUNT
               END-RETURN
           END-PERFORM
           STRING "TOTAL RECORDS: " WS-COUNT
               "  TOTAL AMOUNT: " WS-TOTAL
               DELIMITED BY SIZE INTO WS-REPORT-LINE
           WRITE REPORT-RECORD FROM WS-REPORT-LINE
           CLOSE REPORT-FILE.
```

## See Also

- [SORT](sort.md) -- sorts records from unsorted input
- [SELECT](../../environment-division/select.md) -- file control entry for file status
- [PICTURE](../../data-division/picture.md) -- data item format specification
