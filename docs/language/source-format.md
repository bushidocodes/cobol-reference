# Source Format

COBOL source programs are written in one of two source formats: **fixed-form** (traditional) or **free-form** (COBOL 2002+). The source format determines how the compiler interprets column positions within each source line.

**Standard:** COBOL-60, COBOL-61, COBOL-61 Extended, COBOL-65, COBOL-68, COBOL-85, COBOL 2002, COBOL 2014, COBOL 2023

---

## Fixed-Form Reference Format

The fixed-form reference format divides each 80-character source line into five areas:

| Columns | Area | Purpose |
|---|---|---|
| 1--6 | Sequence number area | Optional line numbering; ignored by the compiler. |
| 7 | Indicator area | Controls line interpretation (comment, continuation, debugging). |
| 8--11 | Area A | Division headers, section headers, paragraph names, FD entries, and level-01/level-77 data items must begin here. |
| 12--72 | Area B | Statements, clauses, sentences, continuation text, and subordinate data items (levels 02--49, 66, 88) are written here. |
| 73--80 | Identification area | Optional program identification; ignored by the compiler. |

### Column Layout

```
Columns:  1234567 8901 234567890...                                 2...78901
          ||||  | |||  |||                                            |||||
          Seq#  | A    B (statements, clauses)                        Ident
                |
                Indicator
```

### Area A (Columns 8--11)

The following constructs must begin in Area A:

- Division headers (`IDENTIFICATION DIVISION.`)
- Section headers (`WORKING-STORAGE SECTION.`, user-defined sections)
- Paragraph names (in the PROCEDURE DIVISION and IDENTIFICATION DIVISION)
- `FD` and `SD` entries
- Level-01 and level-77 data description entries

### Area B (Columns 12--72)

The following constructs are written in Area B:

- Executable statements and clauses
- Subordinate data description entries (levels 02--49, 66, 88)
- Continuation text from a previous line
- Entries within paragraphs of the IDENTIFICATION DIVISION

Text must not extend beyond column 72. Any characters in columns 73--80 are treated as commentary and are ignored by the compiler.

### Indicator Area (Column 7)

The character in column 7 determines how the line is interpreted:

| Character | Meaning |
|---|---|
| (space) | Normal source line. |
| `*` | **Comment line.** The entire line is treated as a comment and produces no compiled code. |
| `/` | **Comment line with page eject.** Same as `*`, but also causes a page break in the compiler listing. |
| `-` | **Continuation line.** The text in Area B is a continuation of the previous line (see [Continuation Lines](#continuation-lines)). |
| `D` | **Debugging line.** The line is compiled only when the `WITH DEBUGGING MODE` clause is specified. |

### Continuation Lines

When a literal, word, or clause does not fit within columns 12--72, the source may be continued on the next line by placing a hyphen (`-`) in column 7 of the continuation line.

Rules for continuation:

1. The continued line ends at or before column 72.
2. The continuation line has a hyphen in column 7.
3. Continuation text begins in Area B (column 12 or later).
4. For non-numeric literals, the opening quotation mark on the continuation line immediately precedes the remaining text. The closing quotation mark of the original line is omitted.

```
       01  WS-LONG-STRING PIC X(60) VALUE "This is a very long str
      -    "ing value that spans two lines".
```

### Comment Lines

A line with an asterisk (`*`) or slash (`/`) in column 7 is a comment line. Comment lines may appear anywhere in the source after the IDENTIFICATION DIVISION header. The entire content of the line is ignored by the compiler.

```
      * This is a comment line.
      / This is a comment line that starts a new listing page.
```

### Fixed-Form Example

```cobol
000100 IDENTIFICATION DIVISION.
000200 PROGRAM-ID. FIXED-EXAMPLE.
000300*
000400* This program demonstrates fixed-form source format.
000500*
000600 ENVIRONMENT DIVISION.
000700 DATA DIVISION.
000800 WORKING-STORAGE SECTION.
000900 01  WS-NAME    PIC X(30).
001000 01  WS-LONG    PIC X(50) VALUE "This is a long value that nee
001100-    "ds continuation".
001200
001300 PROCEDURE DIVISION.
001400 MAIN-LOGIC.
001500     DISPLAY "Enter your name: "
001600     ACCEPT WS-NAME
001700     DISPLAY "Hello, " WS-NAME
001800     STOP RUN.
```

## Free-Form Reference Format

The free-form reference format, introduced in COBOL 2002, removes all column-based restrictions. Source lines may be up to an implementation-defined length (commonly 256 or more characters), and all text is significant.

### Characteristics

- There is no sequence number area, indicator area, Area A, Area B, or identification area.
- Division headers, section headers, paragraph names, and level numbers may begin in any column.
- Source lines may extend to the implementation-defined maximum line length.
- There is no column 7 indicator; comments and continuation use different mechanisms.

### Comments

In free-form source, comments are introduced by the `*>` character sequence. An inline comment may appear on any line after the source text:

```cobol
MOVE 0 TO WS-COUNTER  *> Reset the counter
```

A full-line comment begins with `*>` as the first non-whitespace text on the line:

```cobol
*> This is a full-line comment.
```

The fixed-form comment indicators (`*` and `/` in column 7) are not recognized in free-form source.

### Continuation

In free-form source, there is no explicit continuation indicator. Continuation is handled as follows:

- Statements, clauses, and entries may span multiple lines without any continuation character.
- A non-numeric literal that is not closed (no terminating quotation mark) on a line is continued on the next line. The continuation text begins with a hyphen followed by a quotation mark (`-"`):

```cobol
MOVE "This is a very long string that needs -
    "to be continued" TO WS-RESULT
```

In practice, most free-form COBOL code avoids literal continuation by using concatenation or shorter literals.

### Compiler Directives

Free-form source uses `>>` directives for compiler control. These directives begin with `>>` and may appear anywhere a statement may appear:

| Directive | Purpose |
|---|---|
| `>>SOURCE FORMAT IS FIXED` | Switches the compiler to fixed-form format for subsequent lines. |
| `>>SOURCE FORMAT IS FREE` | Switches the compiler to free-form format for subsequent lines. |
| `>>DEFINE` | Defines a compilation variable. |
| `>>IF` / `>>ELSE` / `>>END-IF` | Conditional compilation. |
| `>>EVALUATE` / `>>WHEN` / `>>END-EVALUATE` | Conditional compilation (COBOL 2014+). |
| `>>TURN` | Controls exception checking. |

### Free-Form Example

```cobol
>>SOURCE FORMAT IS FREE
IDENTIFICATION DIVISION.
PROGRAM-ID. FREE-EXAMPLE.

*> This program demonstrates free-form source format.

ENVIRONMENT DIVISION.
DATA DIVISION.
WORKING-STORAGE SECTION.
01 WS-NAME PIC X(30).

PROCEDURE DIVISION.
MAIN-LOGIC.
    DISPLAY "Enter your name: "
    ACCEPT WS-NAME
    DISPLAY "Hello, " WS-NAME  *> Greet the user
    STOP RUN.
```

## Selecting the Source Format

The source format is determined by one of the following mechanisms, listed in order of typical precedence:

### Compiler Options

Most compilers provide a command-line option or configuration setting to specify the source format. Examples:

| Compiler | Fixed-Form | Free-Form |
|---|---|---|
| GnuCOBOL | `-fixed` (default) | `-free` |
| IBM Enterprise COBOL | Default | `SQLCCSID(FREE)` or source directives |
| Micro Focus | `SOURCEFORMAT(FIXED)` | `SOURCEFORMAT(FREE)` |

### The >>SOURCE Directive

The `>>SOURCE FORMAT` directive switches the source format within a compilation unit. It takes effect on the line immediately following the directive:

```cobol
>>SOURCE FORMAT IS FREE
*> Free-form code follows
IDENTIFICATION DIVISION.
PROGRAM-ID. MIXED-FORMAT.
```

The directive may appear multiple times in a single source file, allowing sections of fixed-form and free-form code to coexist. This is primarily useful when incorporating copybooks written in a different format.

### SOURCEFORMAT Compiler Directive (Micro Focus)

Some compilers recognize a `$SET` directive at the beginning of the source file:

```cobol
$SET SOURCEFORMAT(FREE)
```

## Comparison of Formats

| Feature | Fixed-Form | Free-Form |
|---|---|---|
| Line structure | Column-based (80 columns) | Free-flowing (implementation-defined length) |
| Sequence numbers | Columns 1--6 | Not supported |
| Indicator area | Column 7 | Not applicable |
| Area A / Area B | Enforced (columns 8--11 / 12--72) | Not enforced |
| Comment lines | `*` or `/` in column 7 | `*>` anywhere |
| Inline comments | Not available | `*>` after source text |
| Continuation | `-` in column 7 of next line | Implicit (unclosed literals) |
| Compiler directives | Limited | `>>` directives |

## See Also

- [Program Structure](index.md) -- divisions, sections, paragraphs, and sentences
- [IDENTIFICATION DIVISION](../identification-division/index.md)
