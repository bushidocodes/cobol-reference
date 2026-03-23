# JSON Processing

COBOL provides statements for generating JSON documents from COBOL data structures and parsing JSON documents into COBOL data items.

- **Standard:** IBM Enterprise COBOL v6.1+ (`JSON GENERATE`, `JSON PARSE`). Micro Focus provides similar functionality. Not part of the ISO COBOL standard.
- **Category:** Extension -- Data Interchange

---

## JSON GENERATE

The `JSON GENERATE` statement converts a COBOL group data item into a JSON document.

### Syntax

```cobol
JSON GENERATE json-document FROM data-name
    [ COUNT IN count-field ]
    [ NAME OF
        data-name-1 IS json-name-1 | OMITTED
      [ data-name-2 IS json-name-2 | OMITTED ] ... ]
    [ SUPPRESS
        data-name-1 WHEN ZEROES
      [ data-name-2 WHEN SPACES ] ... ]
    [ ON EXCEPTION imperative-statement ]
    [ NOT ON EXCEPTION imperative-statement ]
END-JSON
```

- **json-document** -- an alphanumeric data item to receive the generated JSON.
- **data-name** -- the group item whose subordinate entries become JSON name-value pairs.
- **COUNT IN** -- receives the number of characters written to json-document.
- **NAME OF ... IS** -- renames COBOL data names to different JSON key names.
- **NAME OF ... IS OMITTED** -- removes the wrapper object for a group item, promoting its children.
- **SUPPRESS** -- omits items that contain zero or space values.

### Rules

- Elementary items within the source group are generated as JSON name-value pairs.
- Subordinate group items are generated as nested JSON objects.
- `OCCURS` items are generated as JSON arrays.
- Numeric items are generated as JSON numbers (unquoted).
- Alphanumeric items are generated as JSON strings (quoted).
- COBOL data names are used as JSON keys by default (in uppercase). Use `NAME OF` to override.
- FILLER items are suppressed by default.

### Basic Generation Example

```cobol
       WORKING-STORAGE SECTION.
       01  WS-JSON-OUTPUT   PIC X(1000).
       01  WS-JSON-LENGTH   PIC 9(5).
       01  WS-EMPLOYEE.
           05  EMP-ID        PIC 9(5)  VALUE 10042.
           05  EMP-NAME      PIC X(20) VALUE "Jane Smith".
           05  EMP-DEPT      PIC X(10) VALUE "Finance".
           05  EMP-SALARY    PIC 9(7)V99 VALUE 85000.00.

       PROCEDURE DIVISION.
           JSON GENERATE WS-JSON-OUTPUT
               FROM WS-EMPLOYEE
               COUNT IN WS-JSON-LENGTH
               NOT ON EXCEPTION
                   DISPLAY WS-JSON-OUTPUT(1:WS-JSON-LENGTH)
           END-JSON
           STOP RUN.
```

**Output:**

```json
{"WS-EMPLOYEE":{"EMP-ID":10042,"EMP-NAME":"Jane Smith","EMP-DEPT":"Finance","EMP-SALARY":8500000}}
```

### Renaming JSON Keys

COBOL data names typically use uppercase with hyphens, which is not idiomatic JSON. Use `NAME OF` to produce cleaner output:

```cobol
       JSON GENERATE WS-JSON-OUTPUT
           FROM WS-EMPLOYEE
           COUNT IN WS-JSON-LENGTH
           NAME OF WS-EMPLOYEE IS OMITTED
                   EMP-ID IS "id"
                   EMP-NAME IS "name"
                   EMP-DEPT IS "department"
                   EMP-SALARY IS "salary"
       END-JSON
```

**Output:**

```json
{"id":10042,"name":"Jane Smith","department":"Finance","salary":8500000}
```

Using `OMITTED` on the top-level group removes the wrapper object, producing a flat JSON object.

### Generating Nested Objects

```cobol
       WORKING-STORAGE SECTION.
       01  WS-JSON-OUTPUT   PIC X(2000).
       01  WS-JSON-LENGTH   PIC 9(5).
       01  WS-ORDER.
           05  ORDER-ID      PIC 9(8) VALUE 10001234.
           05  ORDER-DATE    PIC X(10) VALUE "2025-03-22".
           05  ORDER-CUSTOMER.
               10  CUST-NAME PIC X(25) VALUE "Acme Corp".
               10  CUST-ID   PIC X(10) VALUE "ACME-001".
           05  ORDER-TOTAL   PIC 9(9)V99 VALUE 4599.98.

       PROCEDURE DIVISION.
           JSON GENERATE WS-JSON-OUTPUT
               FROM WS-ORDER
               COUNT IN WS-JSON-LENGTH
               NAME OF WS-ORDER IS OMITTED
                       ORDER-ID IS "orderId"
                       ORDER-DATE IS "orderDate"
                       ORDER-CUSTOMER IS "customer"
                       CUST-NAME IS "name"
                       CUST-ID IS "id"
                       ORDER-TOTAL IS "total"
           END-JSON
           STOP RUN.
```

**Output:**

```json
{"orderId":10001234,"orderDate":"2025-03-22","customer":{"name":"Acme Corp","id":"ACME-001"},"total":459998}
```

### Generating JSON Arrays

Items defined with `OCCURS` are automatically generated as JSON arrays:

```cobol
       WORKING-STORAGE SECTION.
       01  WS-JSON-OUTPUT   PIC X(2000).
       01  WS-JSON-LENGTH   PIC 9(5).
       01  WS-SCORES.
           05  WS-LABEL     PIC X(10) VALUE "Math".
           05  WS-SCORE     PIC 9(3) OCCURS 4 TIMES.

       PROCEDURE DIVISION.
           MOVE 92 TO WS-SCORE(1)
           MOVE 87 TO WS-SCORE(2)
           MOVE 95 TO WS-SCORE(3)
           MOVE 78 TO WS-SCORE(4)

           JSON GENERATE WS-JSON-OUTPUT
               FROM WS-SCORES
               COUNT IN WS-JSON-LENGTH
               NAME OF WS-SCORES IS OMITTED
                       WS-LABEL IS "subject"
                       WS-SCORE IS "scores"
           END-JSON
           STOP RUN.
```

**Output:**

```json
{"subject":"Math","scores":[92,87,95,78]}
```

### Suppressing Empty Fields

```cobol
       WORKING-STORAGE SECTION.
       01  WS-JSON-OUTPUT   PIC X(1000).
       01  WS-JSON-LENGTH   PIC 9(5).
       01  WS-CONTACT.
           05  CONTACT-NAME    PIC X(30) VALUE "Jane Doe".
           05  CONTACT-EMAIL   PIC X(50) VALUE SPACES.
           05  CONTACT-PHONE   PIC X(15) VALUE "555-0100".
           05  CONTACT-FAX     PIC X(15) VALUE SPACES.

       PROCEDURE DIVISION.
           JSON GENERATE WS-JSON-OUTPUT
               FROM WS-CONTACT
               COUNT IN WS-JSON-LENGTH
               NAME OF WS-CONTACT IS OMITTED
                       CONTACT-NAME IS "name"
                       CONTACT-EMAIL IS "email"
                       CONTACT-PHONE IS "phone"
                       CONTACT-FAX IS "fax"
               SUPPRESS CONTACT-EMAIL WHEN SPACES
                        CONTACT-FAX WHEN SPACES
           END-JSON
           STOP RUN.
```

**Output:**

```json
{"name":"Jane Doe","phone":"555-0100"}
```

---

## JSON PARSE

The `JSON PARSE` statement parses a JSON document and populates a COBOL group data item with the values from the JSON name-value pairs.

### Syntax

```cobol
JSON PARSE json-document INTO data-name
    [ NAME OF
        data-name-1 IS json-name-1 | OMITTED
      [ data-name-2 IS json-name-2 | OMITTED ] ... ]
    [ SUPPRESS
        data-name-1
      [ data-name-2 ] ... ]
    [ CONVERTING
        json-name-1 FROM BOOLEAN
            USING TRUE IS true-value FALSE IS false-value ]
    [ ON EXCEPTION imperative-statement ]
    [ NOT ON EXCEPTION imperative-statement ]
END-JSON
```

- **json-document** -- an alphanumeric data item containing the JSON text.
- **data-name** -- the group item to populate with parsed values.
- **NAME OF** -- maps JSON key names to COBOL data names.
- **NAME OF ... IS OMITTED** -- skips a group wrapper level in the JSON.
- **SUPPRESS** -- skips specified data items during parsing.
- **CONVERTING ... FROM BOOLEAN** -- converts JSON boolean values to COBOL values.

### Rules

- JSON string values are moved to the corresponding alphanumeric COBOL items.
- JSON number values are moved to the corresponding numeric COBOL items.
- JSON arrays are parsed into `OCCURS` items.
- JSON keys are matched to COBOL data names (case-insensitive by default). Use `NAME OF` when the JSON keys differ from the COBOL names.
- JSON null values result in the COBOL item being set to spaces (alphanumeric) or zeros (numeric).
- Unmatched JSON keys (keys with no corresponding COBOL item) are ignored.
- Unmatched COBOL items (items with no corresponding JSON key) retain their prior values.

### Basic Parsing Example

```cobol
       WORKING-STORAGE SECTION.
       01  WS-JSON-INPUT    PIC X(500).
       01  WS-EMPLOYEE.
           05  EMP-ID        PIC 9(5).
           05  EMP-NAME      PIC X(20).
           05  EMP-DEPT      PIC X(10).
           05  EMP-SALARY    PIC 9(7)V99.

       PROCEDURE DIVISION.
           MOVE '{"EMP-ID":10042,"EMP-NAME":"Jane Smith",'
             & '"EMP-DEPT":"Finance","EMP-SALARY":8500000}'
               TO WS-JSON-INPUT

           JSON PARSE WS-JSON-INPUT
               INTO WS-EMPLOYEE
               NOT ON EXCEPTION
                   DISPLAY "ID:     " EMP-ID
                   DISPLAY "Name:   " EMP-NAME
                   DISPLAY "Dept:   " EMP-DEPT
                   DISPLAY "Salary: " EMP-SALARY
           END-JSON
           STOP RUN.
```

### Parsing with Key Name Mapping

When JSON keys don't match COBOL data names, use `NAME OF` to establish the mapping:

```cobol
       WORKING-STORAGE SECTION.
       01  WS-JSON-INPUT    PIC X(500).
       01  WS-CUSTOMER.
           05  CUST-ID       PIC X(10).
           05  CUST-NAME     PIC X(30).
           05  CUST-EMAIL    PIC X(50).
           05  CUST-ACTIVE   PIC X(1).

       PROCEDURE DIVISION.
           MOVE '{"id":"C-1001","name":"Acme Corp",'
             & '"email":"info@acme.com","active":true}'
               TO WS-JSON-INPUT

           JSON PARSE WS-JSON-INPUT
               INTO WS-CUSTOMER
               NAME OF WS-CUSTOMER IS OMITTED
                       CUST-ID IS "id"
                       CUST-NAME IS "name"
                       CUST-EMAIL IS "email"
                       CUST-ACTIVE IS "active"
               CONVERTING "active" FROM BOOLEAN
                   USING TRUE IS "Y" FALSE IS "N"
               NOT ON EXCEPTION
                   DISPLAY "ID:     " CUST-ID
                   DISPLAY "Name:   " CUST-NAME
                   DISPLAY "Email:  " CUST-EMAIL
                   DISPLAY "Active: " CUST-ACTIVE
           END-JSON
           STOP RUN.
```

### Parsing JSON Arrays

JSON arrays are parsed into COBOL tables defined with `OCCURS`:

```cobol
       WORKING-STORAGE SECTION.
       01  WS-JSON-INPUT    PIC X(1000).
       01  WS-CLASS-DATA.
           05  CLASS-NAME    PIC X(20).
           05  CLASS-SCORES.
               10  SCORE     PIC 9(3) OCCURS 5 TIMES.

       PROCEDURE DIVISION.
           MOVE '{"CLASS-NAME":"Biology",'
             & '"CLASS-SCORES":[92,87,95,78,88]}'
               TO WS-JSON-INPUT

           JSON PARSE WS-JSON-INPUT
               INTO WS-CLASS-DATA
               NOT ON EXCEPTION
                   DISPLAY "Class: " CLASS-NAME
                   DISPLAY "Score 1: " SCORE(1)
                   DISPLAY "Score 2: " SCORE(2)
           END-JSON
           STOP RUN.
```

### Parsing Nested Objects

```cobol
       WORKING-STORAGE SECTION.
       01  WS-JSON-INPUT    PIC X(1000).
       01  WS-ORDER.
           05  ORDER-ID      PIC 9(8).
           05  ORDER-CUSTOMER.
               10  CUST-NAME PIC X(25).
               10  CUST-ID   PIC X(10).
           05  ORDER-TOTAL   PIC 9(9)V99.

       PROCEDURE DIVISION.
           MOVE '{"orderId":10001234,'
             & '"customer":{"name":"Acme Corp",'
             & '"id":"ACME-001"},"total":459998}'
               TO WS-JSON-INPUT

           JSON PARSE WS-JSON-INPUT
               INTO WS-ORDER
               NAME OF WS-ORDER IS OMITTED
                       ORDER-ID IS "orderId"
                       ORDER-CUSTOMER IS "customer"
                       CUST-NAME IS "name"
                       CUST-ID IS "id"
                       ORDER-TOTAL IS "total"
               NOT ON EXCEPTION
                   DISPLAY "Order: " ORDER-ID
                   DISPLAY "Customer: " CUST-NAME
                   DISPLAY "Total: " ORDER-TOTAL
           END-JSON
           STOP RUN.
```

---

## Error Handling

Both `JSON GENERATE` and `JSON PARSE` set a return code accessible via the `ON EXCEPTION` phrase. The special register `JSON-CODE` contains the status:

| JSON-CODE Value | Meaning |
|-----------------|---------|
| 0 | Successful completion |
| Non-zero | An error occurred |

```cobol
       JSON PARSE WS-JSON-INPUT
           INTO WS-DATA
           ON EXCEPTION
               DISPLAY "JSON error code: " JSON-CODE
               PERFORM JSON-ERROR-HANDLER
           NOT ON EXCEPTION
               DISPLAY "JSON parsed successfully"
       END-JSON
```

### Common Error Conditions

| Condition | Cause |
|-----------|-------|
| Invalid JSON syntax | Malformed JSON (missing quotes, brackets, etc.) |
| Data overflow | JSON value too large for the receiving COBOL item |
| Type mismatch | JSON string in a numeric field or vice versa |
| Array overflow | JSON array has more elements than the OCCURS count |
| Output overflow | Generated JSON exceeds the receiving field length |

---

## Roundtrip Example

Generating JSON from a COBOL structure and parsing it back:

```cobol
       WORKING-STORAGE SECTION.
       01  WS-JSON-BUFFER   PIC X(2000).
       01  WS-JSON-LENGTH   PIC 9(5).

       01  WS-INVOICE.
           05  INV-NUMBER    PIC 9(8)    VALUE 20250001.
           05  INV-DATE      PIC X(10)   VALUE "2025-03-22".
           05  INV-VENDOR    PIC X(25)   VALUE "Parts Co".
           05  INV-LINES.
               10  INV-LINE OCCURS 3 TIMES.
                   15  LINE-DESC   PIC X(20).
                   15  LINE-AMT   PIC 9(7)V99.
           05  INV-TOTAL     PIC 9(9)V99 VALUE 0.

       01  WS-PARSED-INVOICE.
           05  P-INV-NUMBER  PIC 9(8).
           05  P-INV-DATE    PIC X(10).
           05  P-INV-VENDOR  PIC X(25).
           05  P-INV-TOTAL   PIC 9(9)V99.

       PROCEDURE DIVISION.
      *>   Set up line items
           MOVE "Bolts"     TO LINE-DESC(1)
           MOVE 150.00      TO LINE-AMT(1)
           MOVE "Nuts"      TO LINE-DESC(2)
           MOVE 75.50       TO LINE-AMT(2)
           MOVE "Washers"   TO LINE-DESC(3)
           MOVE 42.25       TO LINE-AMT(3)
           COMPUTE INV-TOTAL =
               LINE-AMT(1) + LINE-AMT(2) + LINE-AMT(3)

      *>   Generate JSON
           JSON GENERATE WS-JSON-BUFFER
               FROM WS-INVOICE
               COUNT IN WS-JSON-LENGTH
               NAME OF WS-INVOICE IS OMITTED
                       INV-NUMBER IS "invoiceNumber"
                       INV-DATE IS "date"
                       INV-VENDOR IS "vendor"
                       INV-LINES IS "lineItems"
                       INV-LINE IS OMITTED
                       LINE-DESC IS "description"
                       LINE-AMT IS "amount"
                       INV-TOTAL IS "total"
               ON EXCEPTION
                   DISPLAY "Generate error: " JSON-CODE
                   STOP RUN
           END-JSON

           DISPLAY "Generated JSON:"
           DISPLAY WS-JSON-BUFFER(1:WS-JSON-LENGTH)

      *>   Parse the JSON back into a different structure
           JSON PARSE WS-JSON-BUFFER
               INTO WS-PARSED-INVOICE
               NAME OF WS-PARSED-INVOICE IS OMITTED
                       P-INV-NUMBER IS "invoiceNumber"
                       P-INV-DATE IS "date"
                       P-INV-VENDOR IS "vendor"
                       P-INV-TOTAL IS "total"
               ON EXCEPTION
                   DISPLAY "Parse error: " JSON-CODE
                   STOP RUN
           END-JSON

           DISPLAY "Parsed back:"
           DISPLAY "  Invoice: " P-INV-NUMBER
           DISPLAY "  Date:    " P-INV-DATE
           DISPLAY "  Vendor:  " P-INV-VENDOR
           DISPLAY "  Total:   " P-INV-TOTAL
           STOP RUN.
```

---

## Micro Focus Extensions

Micro Focus Visual COBOL supports JSON through:

- **JSON GENERATE / JSON PARSE** -- compatible syntax with IBM Enterprise COBOL
- **CBL_JSON_* library routines** -- procedural API for fine-grained JSON construction and parsing

```cobol
      *> Micro Focus procedural approach
       CALL "CBL_JSON_PARSE_START" USING
           WS-JSON-INPUT WS-JSON-LENGTH WS-HANDLE
       CALL "CBL_JSON_PARSE_GET_NEXT" USING
           WS-HANDLE WS-KEY WS-VALUE WS-TYPE
```

---

## See Also

- [XML Processing](xml.md) -- XML parsing and generation
- [Embedded SQL](embedded-sql.md) -- database access
- [OCCURS Clause](../data-division/occurs.md) -- table definitions for JSON arrays
- [STRING](../procedure-division/string-handling/string.md) -- string concatenation
