# Sample Programs

Complete COBOL programs demonstrating the interplay between all four divisions.

---

## Program 1: Sequential File Processing

A basic batch program that reads a sequential input file, processes records, and writes a report.

```cobol
       IDENTIFICATION DIVISION.
       PROGRAM-ID. CUST-REPORT.
       AUTHOR. COBOL-REFERENCE.

       ENVIRONMENT DIVISION.
       INPUT-OUTPUT SECTION.
       FILE-CONTROL.
           SELECT CUSTOMER-FILE
               ASSIGN TO "CUSTOMERS.DAT"
               ORGANIZATION IS LINE SEQUENTIAL
               FILE STATUS IS WS-CUST-STATUS.
           SELECT REPORT-FILE
               ASSIGN TO "REPORT.TXT"
               ORGANIZATION IS LINE SEQUENTIAL
               FILE STATUS IS WS-RPT-STATUS.

       DATA DIVISION.
       FILE SECTION.
       FD  CUSTOMER-FILE.
       01  CUSTOMER-RECORD.
           05  CUST-ID          PIC X(8).
           05  CUST-NAME        PIC X(25).
           05  CUST-BALANCE     PIC S9(7)V99.

       FD  REPORT-FILE.
       01  REPORT-LINE          PIC X(80).

       WORKING-STORAGE SECTION.
       01  WS-CUST-STATUS       PIC XX.
       01  WS-RPT-STATUS        PIC XX.
       01  WS-EOF               PIC X VALUE "N".
           88  EOF-REACHED      VALUE "Y".
       01  WS-RECORD-COUNT      PIC 9(5) VALUE 0.
       01  WS-TOTAL-BALANCE     PIC S9(9)V99 VALUE 0.
       01  WS-DISPLAY-BAL       PIC -ZZ,ZZZ,ZZ9.99.
       01  WS-DISPLAY-TOT       PIC -Z,ZZZ,ZZZ,ZZ9.99.

       PROCEDURE DIVISION.
       MAIN-PARA.
           OPEN INPUT  CUSTOMER-FILE
                OUTPUT REPORT-FILE

           MOVE "Customer Report" TO REPORT-LINE
           WRITE REPORT-LINE
           MOVE SPACES TO REPORT-LINE
           WRITE REPORT-LINE

           PERFORM READ-AND-PROCESS
               UNTIL EOF-REACHED

           MOVE SPACES TO REPORT-LINE
           WRITE REPORT-LINE
           MOVE WS-TOTAL-BALANCE TO WS-DISPLAY-TOT
           STRING "Total: " WS-DISPLAY-TOT
               DELIMITED BY SIZE INTO REPORT-LINE
           WRITE REPORT-LINE
           STRING "Records: " WS-RECORD-COUNT
               DELIMITED BY SIZE INTO REPORT-LINE
           WRITE REPORT-LINE

           CLOSE CUSTOMER-FILE REPORT-FILE
           DISPLAY "Report complete. Records: " WS-RECORD-COUNT
           STOP RUN.

       READ-AND-PROCESS.
           READ CUSTOMER-FILE
               AT END SET EOF-REACHED TO TRUE
           END-READ
           IF NOT EOF-REACHED
               ADD 1 TO WS-RECORD-COUNT
               ADD CUST-BALANCE TO WS-TOTAL-BALANCE
               MOVE CUST-BALANCE TO WS-DISPLAY-BAL
               MOVE SPACES TO REPORT-LINE
               STRING CUST-ID "  " CUST-NAME "  "
                   WS-DISPLAY-BAL
                   DELIMITED BY SIZE INTO REPORT-LINE
               WRITE REPORT-LINE
           END-IF.
```

---

## Program 2: Indexed File Maintenance

A program demonstrating OPEN I-O, READ, REWRITE, DELETE, and WRITE on an indexed file.

```cobol
       IDENTIFICATION DIVISION.
       PROGRAM-ID. CUST-MAINT.

       ENVIRONMENT DIVISION.
       INPUT-OUTPUT SECTION.
       FILE-CONTROL.
           SELECT CUSTOMER-FILE
               ASSIGN TO "CUSTFILE"
               ORGANIZATION IS INDEXED
               ACCESS MODE IS DYNAMIC
               RECORD KEY IS CM-CUST-ID
               FILE STATUS IS WS-STATUS.

       DATA DIVISION.
       FILE SECTION.
       FD  CUSTOMER-FILE.
       01  CUSTOMER-RECORD.
           05  CM-CUST-ID      PIC X(8).
           05  CM-CUST-NAME    PIC X(25).
           05  CM-CUST-BAL     PIC S9(7)V99.
           05  CM-CUST-STATUS  PIC X.

       WORKING-STORAGE SECTION.
       01  WS-STATUS            PIC XX.
       01  WS-ACTION            PIC X.
           88  ACTION-ADD       VALUE "A".
           88  ACTION-UPDATE    VALUE "U".
           88  ACTION-DELETE    VALUE "D".
           88  ACTION-INQUIRE   VALUE "I".
           88  ACTION-QUIT      VALUE "Q".

       PROCEDURE DIVISION.
       MAIN-PARA.
           OPEN I-O CUSTOMER-FILE
           IF WS-STATUS NOT = "00" AND "05"
               DISPLAY "Open failed: " WS-STATUS
               STOP RUN
           END-IF

           PERFORM UNTIL ACTION-QUIT
               DISPLAY "Action (A/U/D/I/Q): "
                   WITH NO ADVANCING
               ACCEPT WS-ACTION
               EVALUATE TRUE
                   WHEN ACTION-ADD     PERFORM ADD-CUSTOMER
                   WHEN ACTION-UPDATE  PERFORM UPDATE-CUSTOMER
                   WHEN ACTION-DELETE  PERFORM DELETE-CUSTOMER
                   WHEN ACTION-INQUIRE PERFORM INQUIRE-CUSTOMER
                   WHEN ACTION-QUIT    CONTINUE
                   WHEN OTHER
                       DISPLAY "Invalid action"
               END-EVALUATE
           END-PERFORM

           CLOSE CUSTOMER-FILE
           STOP RUN.

       ADD-CUSTOMER.
           DISPLAY "ID: " WITH NO ADVANCING
           ACCEPT CM-CUST-ID
           DISPLAY "Name: " WITH NO ADVANCING
           ACCEPT CM-CUST-NAME
           MOVE 0 TO CM-CUST-BAL
           MOVE "A" TO CM-CUST-STATUS
           WRITE CUSTOMER-RECORD
               INVALID KEY
                   DISPLAY "Customer already exists"
               NOT INVALID KEY
                   DISPLAY "Customer added"
           END-WRITE.

       UPDATE-CUSTOMER.
           DISPLAY "ID: " WITH NO ADVANCING
           ACCEPT CM-CUST-ID
           READ CUSTOMER-FILE
               INVALID KEY
                   DISPLAY "Customer not found"
                   EXIT PARAGRAPH
           END-READ
           DISPLAY "Current name: " CM-CUST-NAME
           DISPLAY "New name (or ENTER to keep): "
               WITH NO ADVANCING
           ACCEPT CM-CUST-NAME
           REWRITE CUSTOMER-RECORD
               INVALID KEY DISPLAY "Update failed"
           END-REWRITE.

       DELETE-CUSTOMER.
           DISPLAY "ID: " WITH NO ADVANCING
           ACCEPT CM-CUST-ID
           DELETE CUSTOMER-FILE RECORD
               INVALID KEY
                   DISPLAY "Customer not found"
               NOT INVALID KEY
                   DISPLAY "Customer deleted"
           END-DELETE.

       INQUIRE-CUSTOMER.
           DISPLAY "ID: " WITH NO ADVANCING
           ACCEPT CM-CUST-ID
           READ CUSTOMER-FILE
               INVALID KEY
                   DISPLAY "Customer not found"
               NOT INVALID KEY
                   DISPLAY "Name:    " CM-CUST-NAME
                   DISPLAY "Balance: " CM-CUST-BAL
                   DISPLAY "Status:  " CM-CUST-STATUS
           END-READ.
```

---

## Program 3: Sort with Input/Output Procedures

Demonstrates SORT with filtering (INPUT PROCEDURE) and formatting (OUTPUT PROCEDURE).

```cobol
       IDENTIFICATION DIVISION.
       PROGRAM-ID. SORT-DEMO.

       ENVIRONMENT DIVISION.
       INPUT-OUTPUT SECTION.
       FILE-CONTROL.
           SELECT RAW-FILE  ASSIGN TO "RAW.DAT"
               ORGANIZATION IS LINE SEQUENTIAL.
           SELECT SORT-WORK ASSIGN TO "SORTWORK".
           SELECT OUT-FILE  ASSIGN TO "SORTED.DAT"
               ORGANIZATION IS LINE SEQUENTIAL.

       DATA DIVISION.
       FILE SECTION.
       FD  RAW-FILE.
       01  RAW-RECORD.
           05  RAW-DEPT     PIC X(5).
           05  RAW-NAME     PIC X(25).
           05  RAW-SALARY   PIC 9(7)V99.
           05  RAW-STATUS   PIC X.

       SD  SORT-WORK.
       01  SORT-RECORD.
           05  SORT-DEPT    PIC X(5).
           05  SORT-NAME    PIC X(25).
           05  SORT-SALARY  PIC 9(7)V99.

       FD  OUT-FILE.
       01  OUT-RECORD       PIC X(80).

       WORKING-STORAGE SECTION.
       01  WS-EOF           PIC X VALUE "N".
           88  EOF-REACHED  VALUE "Y".
       01  WS-COUNT         PIC 9(5) VALUE 0.
       01  WS-DISPLAY-SAL   PIC $ZZZ,ZZ9.99.

       PROCEDURE DIVISION.
       MAIN-PARA.
           SORT SORT-WORK
               ON ASCENDING KEY SORT-DEPT
               ON ASCENDING KEY SORT-NAME
               INPUT PROCEDURE IS FILTER-ACTIVE
               OUTPUT PROCEDURE IS WRITE-REPORT.
           DISPLAY "Sorted " WS-COUNT " active employees"
           STOP RUN.

       FILTER-ACTIVE SECTION.
           OPEN INPUT RAW-FILE
           PERFORM UNTIL EOF-REACHED
               READ RAW-FILE
                   AT END SET EOF-REACHED TO TRUE
               END-READ
               IF NOT EOF-REACHED AND RAW-STATUS = "A"
                   MOVE RAW-DEPT   TO SORT-DEPT
                   MOVE RAW-NAME   TO SORT-NAME
                   MOVE RAW-SALARY TO SORT-SALARY
                   RELEASE SORT-RECORD
               END-IF
           END-PERFORM
           CLOSE RAW-FILE.

       WRITE-REPORT SECTION.
           OPEN OUTPUT OUT-FILE
           MOVE "Dept  Name                      Salary"
               TO OUT-RECORD
           WRITE OUT-RECORD
           MOVE SPACES TO OUT-RECORD
           WRITE OUT-RECORD
           PERFORM UNTIL EOF-REACHED
               RETURN SORT-WORK
                   AT END SET EOF-REACHED TO TRUE
               END-RETURN
               IF NOT EOF-REACHED
                   ADD 1 TO WS-COUNT
                   MOVE SORT-SALARY TO WS-DISPLAY-SAL
                   MOVE SPACES TO OUT-RECORD
                   STRING SORT-DEPT "  " SORT-NAME "  "
                       WS-DISPLAY-SAL
                       DELIMITED BY SIZE INTO OUT-RECORD
                   WRITE OUT-RECORD
               END-IF
           END-PERFORM
           CLOSE OUT-FILE.
```

---

## Program 4: Subprogram with CALL

A main program calling a subprogram that validates and formats a date.

**Main program:**

```cobol
       IDENTIFICATION DIVISION.
       PROGRAM-ID. DATE-MAIN.

       DATA DIVISION.
       WORKING-STORAGE SECTION.
       01  WS-INPUT-DATE    PIC 9(8).
       01  WS-OUTPUT-DATE   PIC X(10).
       01  WS-VALID         PIC X.

       PROCEDURE DIVISION.
           MOVE 20250322 TO WS-INPUT-DATE
           CALL "DATE-FMT" USING
               WS-INPUT-DATE WS-OUTPUT-DATE WS-VALID

           IF WS-VALID = "Y"
               DISPLAY "Formatted: " WS-OUTPUT-DATE
           ELSE
               DISPLAY "Invalid date: " WS-INPUT-DATE
           END-IF
           STOP RUN.
```

**Subprogram:**

```cobol
       IDENTIFICATION DIVISION.
       PROGRAM-ID. DATE-FMT.

       DATA DIVISION.
       WORKING-STORAGE SECTION.
       01  WS-YEAR     PIC 9(4).
       01  WS-MONTH    PIC 9(2).
       01  WS-DAY      PIC 9(2).

       LINKAGE SECTION.
       01  LS-DATE-IN  PIC 9(8).
       01  LS-DATE-OUT PIC X(10).
       01  LS-VALID    PIC X.

       PROCEDURE DIVISION USING LS-DATE-IN LS-DATE-OUT
                                LS-VALID.
           MOVE "N" TO LS-VALID
           COMPUTE WS-YEAR  = LS-DATE-IN / 10000
           COMPUTE WS-MONTH =
               FUNCTION MOD(LS-DATE-IN / 100, 100)
           COMPUTE WS-DAY   =
               FUNCTION MOD(LS-DATE-IN, 100)

           IF WS-MONTH >= 1 AND WS-MONTH <= 12
               AND WS-DAY >= 1 AND WS-DAY <= 31
               MOVE "Y" TO LS-VALID
               STRING WS-YEAR "/" WS-MONTH "/" WS-DAY
                   DELIMITED BY SIZE INTO LS-DATE-OUT
           END-IF
           GOBACK.
```

---

## See Also

- [Procedure Division](../procedure-division/index.md) -- statement reference
- [Data Division](../data-division/index.md) -- data description
- [Environment Division](../environment-division/index.md) -- file definitions
