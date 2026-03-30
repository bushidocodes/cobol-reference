# Best Practices and Common Patterns

Practical guidance for writing maintainable, portable, and correct COBOL programs.

---

## Program Structure

- **Always use scope terminators** (END-IF, END-PERFORM, END-READ, etc.) instead of relying on periods for scope termination.
- **One statement per sentence** — avoid multiple statements in a single period-terminated sentence.
- **Use GOBACK** instead of STOP RUN in subprograms. GOBACK works correctly in both main programs and subprograms.
- **Name paragraphs descriptively** — use prefixes to indicate hierarchy (e.g., `1000-MAIN`, `2000-PROCESS`, `2100-VALIDATE`).
- **Avoid GO TO** where possible. Use PERFORM and EVALUATE for structured control flow.
- **Never use ALTER** — it was removed in COBOL 2002 for good reason.

## Data Division

- **Prefix data names** with a section indicator: `WS-` for Working-Storage, `LS-` for Local-Storage or Linkage, `FD-` or file abbreviation for File Section.
- **Use level-88 condition names** instead of comparing data items to literals:
  ```cobol
  *> Avoid:
  IF WS-STATUS = "A"
  *> Prefer:
  IF ACTIVE
  ```
- **Avoid level-77** — use level-01 group items to organize related fields.
- **Use INITIALIZE** instead of multiple MOVE SPACES / MOVE ZEROS.
- **Define FILE STATUS** on every file and check it after every I/O operation.
- **Use COPY for shared record layouts** — define record structures in copybooks to ensure consistency.

## Arithmetic

- **Use COMPUTE** for complex formulas instead of chaining ADD/SUBTRACT/MULTIPLY/DIVIDE.
- **Always specify ON SIZE ERROR** on arithmetic statements that could overflow.
- **Use ROUNDED** when the result has fewer decimal places than the calculation.
- **Prefer PACKED-DECIMAL (COMP-3)** for financial amounts — it avoids binary-to-decimal conversion overhead.

## String Handling

- **Use FUNCTION TRIM** instead of manual space handling.
- **Use reference modification** `identifier(start:length)` for simple substring access.
- **Use STRING/UNSTRING** for complex concatenation and parsing.

## File I/O

- **Always check FILE STATUS** after OPEN, READ, WRITE, REWRITE, DELETE, START, and CLOSE.
- **Use DECLARATIVES** (USE AFTER ERROR) for centralized file error handling in production programs.
- **Close files in error handlers** — use a paragraph that closes all open files before abnormal termination.
- **Open files as late as possible**, close them as soon as possible.

## Error Handling Pattern

```cobol
       DECLARATIVES.
       FILE-ERROR SECTION.
           USE AFTER STANDARD ERROR PROCEDURE ON INPUT.
       FILE-ERROR-PARA.
           EVALUATE WS-FILE-STATUS
               WHEN "10" CONTINUE    *> AT END is normal
               WHEN "35"
                   DISPLAY "File not found"
                   PERFORM ABORT-PROGRAM
               WHEN OTHER
                   DISPLAY "I/O error: " WS-FILE-STATUS
                   PERFORM ABORT-PROGRAM
           END-EVALUATE.
       END DECLARATIVES.
```

## Portability

- **Avoid vendor extensions** unless necessary. Mark them clearly with comments.
- **Use STANDARD-1 (ASCII)** collating sequence explicitly if sort order matters.
- **Avoid SYNCHRONIZED** in file records — it inserts platform-dependent padding.
- **Use SIGN IS TRAILING SEPARATE** when exchanging data between EBCDIC and ASCII systems.
- **Test on multiple compilers** if portability is a requirement.

## Naming Conventions

| Convention | Example | Purpose |
|-----------|---------|---------|
| WS- prefix | `WS-CUSTOMER-NAME` | Working-Storage items |
| LS- prefix | `LS-INPUT-AMOUNT` | Linkage Section items |
| 88-level names | `ACTIVE`, `EOF-REACHED` | Condition names (no prefix) |
| Verb-based paragraphs | `PROCESS-TRANSACTION` | Action paragraphs |
| Numbered hierarchy | `1000-MAIN-PROCESS` | Main flow |
| FILLER for padding | `FILLER PIC X(10)` | Unused record areas |

---

## See Also

- [Terminology Mapping](terminology.md) -- cross-language concept mapping
- [Sample Programs](sample-programs.md) -- complete example programs
- [File Status Codes](file-status-codes.md) -- I/O status reference
