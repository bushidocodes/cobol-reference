# Character Set and Words

The COBOL character set defines the permissible characters in a source program. All COBOL source text is composed of **words**, **literals**, **separators**, and **comments**, which combine to form sentences, statements, and clauses.

**Standard:** COBOL-60, COBOL-61, COBOL-61 Extended, COBOL-65, COBOL-68, COBOL-74, COBOL-85, COBOL 2002, COBOL 2014, COBOL 2023

---

## COBOL Character Set

The standard COBOL character set consists of the following characters:

| Category | Characters |
|---|---|
| Uppercase letters | `A` through `Z` |
| Lowercase letters | `a` through `z` |
| Digits | `0` through `9` |
| Space | (blank character) |
| Plus sign | `+` |
| Minus sign (hyphen) | `-` |
| Asterisk | `*` |
| Slash | `/` |
| Equal sign | `=` |
| Dollar sign | `$` |
| Comma | `,` |
| Semicolon | `;` |
| Period (decimal point) | `.` |
| Quotation mark | `"` |
| Apostrophe | `'` |
| Left parenthesis | `(` |
| Right parenthesis | `)` |
| Greater-than | `>` |
| Less-than | `<` |
| Colon | `:` |

Lowercase letters are treated as equivalent to their uppercase counterparts in all contexts except within alphanumeric and national literals.

## Words

A **word** is a contiguous sequence of characters from the COBOL character set, bounded by separators. COBOL defines several categories of words.

### User-Defined Words

A user-defined word is a word that the programmer creates to name data items, paragraphs, sections, files, and other program entities. The following rules apply:

- A user-defined word contains 1 to 30 characters.
- Only letters (`A`--`Z`, `a`--`z`), digits (`0`--`9`), and hyphens (`-`) are permitted.
- At least one character must be a letter.
- The word must not begin or end with a hyphen.
- The word must not be a reserved word.

Examples of valid user-defined words:

```
WS-COUNTER
Employee-Name
TOTAL-AMT-01
X
A1B2C3
```

### System-Names

System-names are words that identify features of the operating environment. They include computer-names, language-names (such as `ALPHANUMERIC`), implementor-names, and feature-names. The set of valid system-names is implementation-defined.

### Reserved Words

Reserved words are words with predefined meanings in COBOL. They must not be used as user-defined names. Reserved words include keywords (such as `MOVE`, `IF`, `PERFORM`), optional words, and [figurative constants](figurative-constants.md) (such as `ZERO`, `SPACE`, `HIGH-VALUE`).

### Function-Names

Function-names identify intrinsic functions (e.g., `FUNCTION CURRENT-DATE`, `FUNCTION LENGTH`). A function-name is a reserved word when it appears immediately after the keyword `FUNCTION`.

## Separators

Separators delimit words and literals in COBOL source text. The following separators are defined:

| Separator | Description |
|---|---|
| Space | Required between consecutive words. One or more spaces are treated identically. |
| Comma (`,`) | May be used in place of a space where a space is permitted. The comma must be followed by at least one space. |
| Semicolon (`;`) | May be used in place of a space where a space is permitted. The semicolon must be followed by at least one space. |
| Period (`.`) | Terminates entries, sentences, and certain clauses. The period must be followed by at least one space (except at the end of a line). |
| Colon (`:`) | Used in [reference modification](#reference-modification) syntax. |
| Parentheses (`(` `)`) | Enclose subscripts, reference modification, arithmetic expressions, and condition groupings. |
| Quotation mark (`"`) or apostrophe (`'`) | Delimits alphanumeric literals. |
| `==` | Delimits pseudo-text in COPY REPLACING and REPLACE statements. |

!!! note
    Commas and semicolons are syntactically interchangeable with spaces. They serve only as visual aids for readability and have no effect on program meaning.

## Literals

A literal is a value specified directly in the source text. COBOL defines several types of literals.

### Numeric Literals

A numeric literal is a string of 1 to 18 digits, optionally preceded by a sign (`+` or `-`) and optionally containing a single decimal point. The decimal point must not be the rightmost character.

```
42
-3.14
+100
0.001
```

### Alphanumeric Literals

An alphanumeric literal is a sequence of characters enclosed in quotation marks (`"..."`) or apostrophes (`'...'`). The delimiter used to open the literal must be the same as the one used to close it. The literal may contain 1 to 160 characters (implementation limits may vary).

```
"HELLO WORLD"
'Account Number'
"It""s here"
```

To include the delimiter character within the literal, it is written twice (e.g., `""` within a quoted literal produces a single `"`).

### National Literals

A national literal, introduced in COBOL 2002, represents a string of national (Unicode) characters. It is written with the prefix `N` followed by a quoted string:

```
N"Hello"
N"Unicode text"
```

### Boolean Literals

A boolean literal represents a value composed of boolean digits (`0` and `1`). It is written with the prefix `B` followed by a quoted string:

```
B"0"
B"1010"
```

### Figurative Constants

[Figurative constants](figurative-constants.md) are reserved words that represent specific constant values. They may be used wherever a literal is permitted (with certain restrictions). The figurative constants are `ZERO`, `SPACE`, `HIGH-VALUE`, `LOW-VALUE`, `QUOTE`, `ALL literal`, and `NULL`.

## Sentences, Statements, and Clauses

### Statements

A **statement** is an executable instruction beginning with a COBOL verb. Statements are classified as:

- **Imperative statements** -- unconditional actions (e.g., `MOVE X TO Y`, `DISPLAY "Hello"`)
- **Conditional statements** -- actions that include conditions or exception branches (e.g., [IF](../procedure-division/control-flow/if.md), EVALUATE, `READ ... AT END ...`)
- **Compiler-directing statements** -- instructions to the compiler (e.g., `COPY`, `REPLACE`)

### Sentences

A **sentence** is one or more statements terminated by a period. In modern practice, each sentence typically contains a single statement, with [scope terminators](scope-terminators.md) used to delimit nested structures.

```cobol
MOVE 0 TO WS-TOTAL.
ADD WS-AMOUNT TO WS-TOTAL.
```

### Clauses

A **clause** is a component of an entry or statement that specifies an attribute or option. Clauses appear in data description entries (e.g., [PICTURE](../data-division/picture.md), [USAGE](../data-division/usage.md), [VALUE](../data-division/value.md)) and in certain statements.

## Reference Modification

Reference modification provides a means of referring to a substring of a data item. The syntax is:

```
identifier(leftmost-character-position : length)
```

- **leftmost-character-position** is an arithmetic expression specifying the ordinal position of the first character of the substring (starting at 1).
- **length** is an arithmetic expression specifying the number of characters in the substring. If omitted, the substring extends from the leftmost character position to the end of the data item.

```cobol
       01  WS-NAME    PIC X(20) VALUE "JOHN DOE            ".

       DISPLAY WS-NAME(1:4)
      *> Displays "JOHN"

       DISPLAY WS-NAME(6:3)
      *> Displays "DOE"

       MOVE "SMITH" TO WS-NAME(6:5)
      *> WS-NAME is now "JOHN SMITH          "
```

The leftmost character position must be a positive integer. The sum of the leftmost character position and the length, minus one, must not exceed the size of the data item.

!!! note
    Reference modification applies only to data items with [USAGE](../data-division/usage.md) DISPLAY or DISPLAY-1. It does not apply to items with USAGE COMP or USAGE PACKED-DECIMAL.

## See Also

- [Source Format](source-format.md) -- fixed-form and free-form source layouts
- [Figurative Constants](figurative-constants.md) -- ZERO, SPACE, HIGH-VALUE, and others
- [Scope Terminators](scope-terminators.md) -- explicit statement termination
- [PICTURE Clause](../data-division/picture.md) -- data item format specification
- [USAGE Clause](../data-division/usage.md) -- internal data representation
- [VALUE Clause](../data-division/value.md) -- initial value specification
- [Level Numbers](../data-division/level-numbers.md) -- data hierarchy and special levels
- [Condition-Names](../data-division/condition-names.md) -- level-88 condition-names
