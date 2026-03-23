# XML Processing

COBOL provides statements for parsing XML documents into COBOL data structures and generating XML documents from COBOL data items.

- **Standard:** IBM Enterprise COBOL v4+ (`XML PARSE`, `XML GENERATE`). Micro Focus provides similar functionality. Not part of the ISO COBOL standard.
- **Category:** Extension -- Data Interchange

---

## XML PARSE

The `XML PARSE` statement parses an XML document and drives a processing procedure that receives events as the parser encounters each element, attribute, and content fragment.

### Syntax

```cobol
XML PARSE xml-document
    PROCESSING PROCEDURE IS procedure-name
    [ ON EXCEPTION imperative-statement ]
    [ NOT ON EXCEPTION imperative-statement ]
END-XML
```

- **xml-document** -- an alphanumeric data item or national data item containing the XML document text.
- **procedure-name** -- a paragraph or section that is invoked for each XML event.
- **ON EXCEPTION** -- executed if a parsing error occurs.
- **NOT ON EXCEPTION** -- executed if parsing completes without error.

### Special Registers

During XML PARSE, the runtime sets the following special registers before each invocation of the processing procedure:

| Register | Type | Description |
|----------|------|-------------|
| `XML-EVENT` | Alphanumeric | The type of XML event (see table below) |
| `XML-TEXT` | Alphanumeric | The text associated with the event |
| `XML-NTEXT` | National | The national character text (for national documents) |
| `XML-CODE` | Numeric | Status code; 0 indicates no error |

### XML Events

| Event Name | Description |
|------------|-------------|
| `START-OF-DOCUMENT` | Parsing has begun |
| `END-OF-DOCUMENT` | Parsing has completed |
| `START-OF-ELEMENT` | Opening tag encountered; XML-TEXT contains the element name |
| `END-OF-ELEMENT` | Closing tag encountered; XML-TEXT contains the element name |
| `ATTRIBUTE-NAME` | Attribute name encountered; XML-TEXT contains the name |
| `ATTRIBUTE-VALUE` | Attribute value encountered; XML-TEXT contains the value |
| `CONTENT-CHARACTERS` | Element text content; XML-TEXT contains the character data |
| `CONTENT-CHARACTER` | Single character of content (less common) |
| `PROCESSING-INSTRUCTION` | `<?...?>` encountered |
| `COMMENT` | `<!-- ... -->` encountered |
| `EXCEPTION` | A parsing error occurred; XML-CODE contains the error code |

### Basic Parsing Example

```cobol
       WORKING-STORAGE SECTION.
       01  WS-XML-DOC       PIC X(500).
       01  WS-CUST-NAME     PIC X(30).
       01  WS-CUST-ID       PIC X(10).
       01  WS-CURRENT-ELEM  PIC X(30).

       PROCEDURE DIVISION.
       MAIN-PARA.
           MOVE '<customer id="C001">'
             & '<name>John Smith</name>'
             & '</customer>'
               TO WS-XML-DOC

           XML PARSE WS-XML-DOC
               PROCESSING PROCEDURE IS XML-HANDLER
               ON EXCEPTION
                   DISPLAY "XML parse error: " XML-CODE
               NOT ON EXCEPTION
                   DISPLAY "Customer: " WS-CUST-NAME
                   DISPLAY "ID:       " WS-CUST-ID
           END-XML
           STOP RUN.

       XML-HANDLER.
           EVALUATE XML-EVENT
               WHEN "START-OF-ELEMENT"
                   MOVE XML-TEXT TO WS-CURRENT-ELEM
               WHEN "CONTENT-CHARACTERS"
                   EVALUATE WS-CURRENT-ELEM
                       WHEN "name"
                           MOVE XML-TEXT TO WS-CUST-NAME
                   END-EVALUATE
               WHEN "ATTRIBUTE-NAME"
                   CONTINUE
               WHEN "ATTRIBUTE-VALUE"
                   IF WS-CURRENT-ELEM = "customer"
                       MOVE XML-TEXT TO WS-CUST-ID
                   END-IF
               WHEN OTHER
                   CONTINUE
           END-EVALUATE.
```

### Parsing a Multi-Record Document

```cobol
       WORKING-STORAGE SECTION.
       01  WS-XML-DOC       PIC X(2000).
       01  WS-CURRENT-ELEM  PIC X(30).
       01  WS-IN-ITEM       PIC 9 VALUE 0.
       01  WS-ITEM-COUNT    PIC 9(3) VALUE 0.
       01  WS-ITEMS.
           05  WS-ITEM OCCURS 50 TIMES.
               10  WS-ITEM-CODE  PIC X(10).
               10  WS-ITEM-QTY   PIC 9(5).
               10  WS-ITEM-PRICE PIC 9(7)V99.

       PROCEDURE DIVISION.
       MAIN-PARA.
           MOVE '<order>'
             & '<item><code>A100</code>'
             & '<qty>10</qty><price>25.50</price></item>'
             & '<item><code>B200</code>'
             & '<qty>5</qty><price>100.00</price></item>'
             & '</order>'
               TO WS-XML-DOC

           XML PARSE WS-XML-DOC
               PROCESSING PROCEDURE IS ORDER-HANDLER
               ON EXCEPTION
                   DISPLAY "Parse error: " XML-CODE
           END-XML

           PERFORM VARYING WS-IDX FROM 1 BY 1
               UNTIL WS-IDX > WS-ITEM-COUNT
               DISPLAY WS-ITEM-CODE(WS-IDX) " "
                       WS-ITEM-QTY(WS-IDX) " "
                       WS-ITEM-PRICE(WS-IDX)
           END-PERFORM
           STOP RUN.

       ORDER-HANDLER.
           EVALUATE XML-EVENT
               WHEN "START-OF-ELEMENT"
                   MOVE XML-TEXT TO WS-CURRENT-ELEM
                   IF XML-TEXT = "item"
                       ADD 1 TO WS-ITEM-COUNT
                       MOVE 1 TO WS-IN-ITEM
                   END-IF
               WHEN "END-OF-ELEMENT"
                   IF XML-TEXT = "item"
                       MOVE 0 TO WS-IN-ITEM
                   END-IF
               WHEN "CONTENT-CHARACTERS"
                   IF WS-IN-ITEM = 1
                       EVALUATE WS-CURRENT-ELEM
                           WHEN "code"
                               MOVE XML-TEXT TO
                                 WS-ITEM-CODE(WS-ITEM-COUNT)
                           WHEN "qty"
                               MOVE XML-TEXT TO
                                 WS-ITEM-QTY(WS-ITEM-COUNT)
                           WHEN "price"
                               MOVE XML-TEXT TO
                                 WS-ITEM-PRICE(WS-ITEM-COUNT)
                       END-EVALUATE
                   END-IF
               WHEN OTHER
                   CONTINUE
           END-EVALUATE.
```

### Stopping the Parser

To stop parsing before the end of the document, set `XML-CODE` to -1 in the processing procedure:

```cobol
       XML-HANDLER.
           EVALUATE XML-EVENT
               WHEN "START-OF-ELEMENT"
                   IF XML-TEXT = "error"
                       MOVE -1 TO XML-CODE
                   END-IF
               WHEN OTHER
                   CONTINUE
           END-EVALUATE.
```

---

## XML GENERATE

The `XML GENERATE` statement converts a COBOL group data item into an XML document.

### Syntax

```cobol
XML GENERATE xml-document FROM data-name
    [ COUNT IN count-field ]
    [ WITH ENCODING code-page ]
    [ WITH XML-DECLARATION ]
    [ WITH ATTRIBUTES ]
    [ NAMESPACE IS namespace-uri ]
    [ NAMESPACE-PREFIX IS prefix ]
    [ NAME OF
        data-name-1 IS xml-name-1
      [ data-name-2 IS xml-name-2 ] ... ]
    [ TYPE OF
        data-name-1 IS ATTRIBUTE
      [ data-name-2 IS ATTRIBUTE ] ... ]
    [ SUPPRESS
        data-name-1 WHEN ZEROES
      [ data-name-2 WHEN SPACES ] ... ]
    [ ON EXCEPTION imperative-statement ]
    [ NOT ON EXCEPTION imperative-statement ]
END-XML
```

- **xml-document** -- an alphanumeric data item to receive the generated XML.
- **data-name** -- the group item whose subordinate entries define the XML content.
- **COUNT IN** -- receives the number of characters written to xml-document.
- **WITH XML-DECLARATION** -- prepends `<?xml version="1.0" ...?>`.
- **WITH ATTRIBUTES** -- generates elementary items as XML attributes instead of child elements.
- **NAME OF** -- renames COBOL data names to different XML element/attribute names.
- **TYPE OF ... IS ATTRIBUTE** -- generates specific items as attributes.
- **SUPPRESS** -- omits items that contain zero or space values.

### Basic Generation Example

```cobol
       WORKING-STORAGE SECTION.
       01  WS-XML-OUTPUT    PIC X(1000).
       01  WS-XML-LENGTH    PIC 9(5).
       01  WS-CUSTOMER.
           05  CUST-ID      PIC X(5)  VALUE "C001".
           05  CUST-NAME    PIC X(20) VALUE "John Smith".
           05  CUST-BALANCE PIC 9(7)V99 VALUE 1250.75.

       PROCEDURE DIVISION.
           XML GENERATE WS-XML-OUTPUT
               FROM WS-CUSTOMER
               COUNT IN WS-XML-LENGTH
               NOT ON EXCEPTION
                   DISPLAY WS-XML-OUTPUT(1:WS-XML-LENGTH)
           END-XML
           STOP RUN.
```

**Output:**

```xml
<WS-CUSTOMER>
  <CUST-ID>C001</CUST-ID>
  <CUST-NAME>John Smith</CUST-NAME>
  <CUST-BALANCE>125075</CUST-BALANCE>
</WS-CUSTOMER>
```

### Generating with Attributes

```cobol
       WORKING-STORAGE SECTION.
       01  WS-XML-OUTPUT    PIC X(1000).
       01  WS-XML-LENGTH    PIC 9(5).
       01  WS-PRODUCT.
           05  PROD-ID      PIC X(8)  VALUE "PRD-1001".
           05  PROD-NAME    PIC X(25) VALUE "Widget".
           05  PROD-PRICE   PIC 9(5)V99 VALUE 29.95.
           05  PROD-QTY     PIC 9(5) VALUE 150.

       PROCEDURE DIVISION.
           XML GENERATE WS-XML-OUTPUT
               FROM WS-PRODUCT
               COUNT IN WS-XML-LENGTH
               WITH XML-DECLARATION
               TYPE OF PROD-ID IS ATTRIBUTE
               NAME OF WS-PRODUCT IS "product"
                       PROD-ID IS "id"
                       PROD-NAME IS "name"
                       PROD-PRICE IS "price"
                       PROD-QTY IS "quantity"
               NOT ON EXCEPTION
                   DISPLAY WS-XML-OUTPUT(1:WS-XML-LENGTH)
           END-XML
           STOP RUN.
```

**Output:**

```xml
<?xml version="1.0" encoding="IBM-1140"?>
<product id="PRD-1001">
  <name>Widget</name>
  <price>2995</price>
  <quantity>150</quantity>
</product>
```

### Generating with Namespaces

```cobol
       XML GENERATE WS-XML-OUTPUT
           FROM WS-INVOICE
           COUNT IN WS-XML-LENGTH
           WITH XML-DECLARATION
           NAMESPACE IS
               "http://example.com/invoice"
           NAMESPACE-PREFIX IS "inv"
       END-XML
```

### Suppressing Empty Fields

```cobol
       WORKING-STORAGE SECTION.
       01  WS-CONTACT.
           05  CONTACT-NAME   PIC X(30) VALUE "Jane Doe".
           05  CONTACT-EMAIL  PIC X(50) VALUE SPACES.
           05  CONTACT-PHONE  PIC X(15) VALUE "555-0100".

       PROCEDURE DIVISION.
           XML GENERATE WS-XML-OUTPUT
               FROM WS-CONTACT
               COUNT IN WS-XML-LENGTH
               SUPPRESS CONTACT-EMAIL WHEN SPACES
               NOT ON EXCEPTION
                   DISPLAY WS-XML-OUTPUT(1:WS-XML-LENGTH)
           END-XML
           STOP RUN.
```

**Output (CONTACT-EMAIL omitted):**

```xml
<WS-CONTACT>
  <CONTACT-NAME>Jane Doe</CONTACT-NAME>
  <CONTACT-PHONE>555-0100</CONTACT-PHONE>
</WS-CONTACT>
```

---

## Error Handling

Both `XML PARSE` and `XML GENERATE` set `XML-CODE` to indicate success or failure:

| XML-CODE Value | Meaning |
|----------------|---------|
| 0 | Successful completion |
| 1-99 | Warning (parsing continued) |
| 100+ | Error (parsing stopped) |
| Negative | User-requested termination |

Use the `ON EXCEPTION` / `NOT ON EXCEPTION` phrases to handle errors:

```cobol
       XML PARSE WS-XML-DOC
           PROCESSING PROCEDURE IS XML-HANDLER
           ON EXCEPTION
               DISPLAY "XML error code: " XML-CODE
               PERFORM XML-ERROR-HANDLER
           NOT ON EXCEPTION
               DISPLAY "XML parsed successfully"
       END-XML
```

---

## Micro Focus Extensions

Micro Focus Visual COBOL and COBOL-IT provide XML support through different mechanisms:

### XML Extensions Library

```cobol
       CALL "XML" USING XML-PARSE
           WS-XML-DOCUMENT
           WS-PARSE-FLAGS
           WS-STATUS-CODE
```

### XMLSS (XML Syntax Support)

Micro Focus also supports `XML PARSE` and `XML GENERATE` with syntax compatible with IBM Enterprise COBOL when the `XMLPARSE` compiler directive is set.

---

## See Also

- [JSON Processing](json.md) -- JSON parsing and generation
- [Embedded SQL](embedded-sql.md) -- database access
- [STRING](../procedure-division/string-handling/string.md) -- string concatenation
- [UNSTRING](../procedure-division/string-handling/unstring.md) -- string splitting
