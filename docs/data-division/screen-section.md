---
search:
  boost: 2
tags:
  - screen section
  - screen handling
---

# Screen Section — Terminal-Based Screen I/O

The Screen Section defines screen layouts for interactive terminal-based input and output. It specifies the position, attributes, and data bindings of fields displayed on a character-mode terminal, enabling full-screen data entry and display without manual cursor control.

**Standard:** COBOL 2002, COBOL 2014 (optional), COBOL 2023

!!! note "Vendor Extension History"
    The Screen Section was widely available as a vendor extension before its standardization in COBOL 2002. Micro Focus COBOL and ACUCOBOL provided Screen Section support from the 1980s. GnuCOBOL provides comprehensive support. IBM Enterprise COBOL does not support the Screen Section; IBM environments typically use CICS BMS maps or IMS MFS for screen handling instead.

---

## Syntax

### Screen Section Declaration

```cobol
DATA DIVISION.
SCREEN SECTION.
01  screen-name.
    level-number  [screen-item-name]
        [BLANK SCREEN]
        [BLANK LINE]
        [BELL]
        [BLINK]
        [HIGHLIGHT]
        [LOWLIGHT]
        [REVERSE-VIDEO]
        [UNDERLINE]
        [FOREGROUND-COLOR IS integer]
        [BACKGROUND-COLOR IS integer]
        [LINE NUMBER IS { integer | identifier | PLUS integer }]
        [COLUMN NUMBER IS { integer | identifier | PLUS integer }]
        [VALUE IS literal]
        [PICTURE IS picture-string]
        [FROM identifier-1]
        [TO identifier-2]
        [USING identifier-3]
        [REQUIRED]
        [FULL]
        [AUTO]
        [SECURE]
        [screen-attribute-clauses ...].
```

### ACCEPT and DISPLAY for Screens

```cobol
DISPLAY screen-name
    [AT { LINE NUMBER integer | identifier }
        { COLUMN NUMBER integer | identifier }].

ACCEPT screen-name
    [AT { LINE NUMBER integer | identifier }
        { COLUMN NUMBER integer | identifier }].
```

---

## Description

The Screen Section defines screen items that map to positions on a character-mode terminal. Each screen item specifies its position (LINE, COLUMN), its visual attributes (color, highlighting), and its data binding (FROM, TO, or USING). The `DISPLAY screen-name` statement renders the screen, and the `ACCEPT screen-name` statement reads user input from the screen fields.

Screen items are organized hierarchically using [level numbers](level-numbers.md). A level-01 entry defines a screen, and subordinate entries define individual fields and literals within that screen.

---

## Screen Description Entry Clauses

### Positional Clauses

#### LINE Clause

```cobol
LINE NUMBER IS integer.
LINE NUMBER IS identifier.
LINE NUMBER IS PLUS integer.
```

The LINE clause specifies the row on the terminal screen (1-based). `PLUS integer` specifies a position relative to the current cursor line. If omitted, the line follows the previous item's position.

#### COLUMN Clause

```cobol
COLUMN NUMBER IS integer.
COLUMN NUMBER IS identifier.
COLUMN NUMBER IS PLUS integer.
```

The COLUMN clause specifies the column on the terminal screen (1-based). `PLUS integer` specifies a position relative to the current cursor column. If omitted, the column follows immediately after the previous item.

---

### Content Clauses

#### VALUE Clause

```cobol
VALUE IS literal.
```

The VALUE clause specifies a literal string to be displayed. An item with a VALUE clause is an output-only literal and has no associated data item. The VALUE clause is mutually exclusive with FROM, TO, and USING.

#### PICTURE Clause

```cobol
PICTURE IS picture-string.
```

The [PICTURE clause](picture.md) specifies the display format and size of a screen item. It is required on items that have a FROM, TO, or USING clause.

---

### Data Binding Clauses

#### FROM Clause (Output)

```cobol
FROM identifier-1.
```

The FROM clause specifies a data item whose value is displayed on the screen. The screen item is output-only. When the screen is displayed, the current value of identifier-1 is moved to the screen position according to the PICTURE.

#### TO Clause (Input)

```cobol
TO identifier-2.
```

The TO clause specifies a data item that receives user input. The screen item is input-only. When the screen is accepted, the user's input is moved from the screen position to identifier-2.

#### USING Clause (Input/Output)

```cobol
USING identifier-3.
```

The USING clause specifies a data item for both input and output. It is equivalent to specifying both `FROM identifier-3` and `TO identifier-3`. When the screen is displayed, the current value is shown; when accepted, the user's input is stored back.

---

### Attribute Clauses

#### Display Attributes

| Clause | Effect |
|--------|--------|
| `BLANK SCREEN` | Clears the entire screen before displaying this item |
| `BLANK LINE` | Clears the current line before displaying this item |
| `BELL` | Sounds the terminal bell when the item is displayed |
| `BLINK` | Displays the item with a blinking attribute |
| `HIGHLIGHT` | Displays the item with increased intensity (bold) |
| `LOWLIGHT` | Displays the item with decreased intensity (dim) |
| `REVERSE-VIDEO` | Displays the item with foreground and background colors swapped |
| `UNDERLINE` | Displays the item with an underline attribute |

#### Color Clauses

```cobol
FOREGROUND-COLOR IS integer.
BACKGROUND-COLOR IS integer.
```

The color value is an integer from 0 to 7, representing the standard terminal color palette.

| Value | Color |
|-------|-------|
| 0 | Black |
| 1 | Blue |
| 2 | Green |
| 3 | Cyan |
| 4 | Red |
| 5 | Magenta |
| 6 | Yellow (or Brown) |
| 7 | White |

!!! warning "Color Value Portability"
    The mapping of integer values to colors may vary between implementations. The values listed above follow the standard and GnuCOBOL conventions. Some Micro Focus implementations use a different mapping (e.g., 0=Black, 1=Red, 2=Green, 3=Yellow, 4=Blue, 5=Magenta, 6=Cyan, 7=White). Always consult the compiler documentation for the target platform.

---

### Input Validation Clauses

#### REQUIRED Clause

```cobol
REQUIRED.
```

The REQUIRED clause specifies that the user must enter at least one character in the field before the cursor can leave it. An empty field is not accepted.

#### FULL Clause

```cobol
FULL.
```

The FULL clause specifies that the user must fill the entire field. The cursor cannot leave the field until all character positions contain data.

#### AUTO Clause

```cobol
AUTO.
```

The AUTO clause specifies that the cursor automatically advances to the next input field when the current field is filled. Without AUTO, the user must press a key (typically Tab or Enter) to move to the next field.

#### SECURE Clause

```cobol
SECURE.
```

The SECURE clause specifies that the data entered by the user is not displayed on the screen. Asterisks or blanks are shown instead. This clause is typically used for password entry fields.

---

## DISPLAY Statement for Screens

```cobol
DISPLAY screen-name.
```

The DISPLAY statement renders the named screen on the terminal. All output items (VALUE, FROM, and USING clauses) are presented at their specified positions with their specified attributes. The screen is rendered in the order of the screen description entries.

---

## ACCEPT Statement for Screens

```cobol
ACCEPT screen-name.
```

The ACCEPT statement activates the named screen for user input. The cursor is positioned at the first input field (TO or USING clause). The user navigates between input fields and enters data. When the user presses the termination key (typically Enter on the last field, or a function key), control returns to the program and all input values are transferred to their associated data items.

---

## AT Clause

The AT clause permits positioning of a screen or individual item at a specified line and column.

```cobol
DISPLAY screen-name AT LINE 5 COLUMN 10.
ACCEPT screen-name AT LINE 5 COLUMN 10.
```

When AT is specified, it overrides the LINE and COLUMN values in the screen description.

---

## Rules

1. The Screen Section must appear after all other sections in the Data Division (after WORKING-STORAGE, LOCAL-STORAGE, LINKAGE, and REPORT sections).
2. Screen description entries use level numbers 01 through 49, following the same hierarchy rules as other Data Division entries.
3. A level-01 entry in the Screen Section defines a screen. Subordinate entries define fields within the screen.
4. Each elementary screen item must have exactly one of: VALUE, FROM, TO, or USING (or be a group item containing subordinate items with these clauses).
5. Items with FROM, TO, or USING must have a [PICTURE clause](picture.md). Items with VALUE must not have a PICTURE clause if the VALUE determines the size.
6. The data items referenced by FROM, TO, and USING must be defined in the WORKING-STORAGE SECTION, LOCAL-STORAGE SECTION, or LINKAGE SECTION -- not in the Screen Section itself.
7. Attribute clauses on a group item apply to all subordinate items unless overridden.
8. BLANK SCREEN clears the entire terminal and is typically specified on the level-01 entry or the first subordinate entry.

---

## Examples

### Data Entry Form with Validation

```cobol
       IDENTIFICATION DIVISION.
       PROGRAM-ID. DATA-ENTRY.

       DATA DIVISION.
       WORKING-STORAGE SECTION.
       01  WS-EMPLOYEE.
           05  WS-EMP-ID          PIC X(6).
           05  WS-EMP-NAME        PIC X(30).
           05  WS-EMP-DEPT        PIC X(4).
           05  WS-EMP-SALARY      PIC 9(7)V99.
           05  WS-EMP-PASSWORD    PIC X(12).

       01  WS-CONFIRM             PIC X.
           88  CONFIRMED          VALUE "Y" "y".

       01  WS-FORMATTED-SALARY    PIC $,$$$,$$9.99.

       SCREEN SECTION.
       01  ENTRY-SCREEN.
           05  BLANK SCREEN.
           05  LINE 1 COLUMN 20 VALUE "EMPLOYEE DATA ENTRY"
               HIGHLIGHT.
           05  LINE 2 COLUMN 20 VALUE "==================="
               HIGHLIGHT.

           05  LINE 4  COLUMN 5  VALUE "Employee ID:".
           05  LINE 4  COLUMN 20 PIC X(6) USING WS-EMP-ID
               REQUIRED HIGHLIGHT.

           05  LINE 6  COLUMN 5  VALUE "Name:".
           05  LINE 6  COLUMN 20 PIC X(30) USING WS-EMP-NAME
               REQUIRED.

           05  LINE 8  COLUMN 5  VALUE "Department:".
           05  LINE 8  COLUMN 20 PIC X(4) USING WS-EMP-DEPT
               REQUIRED FULL AUTO.

           05  LINE 10 COLUMN 5  VALUE "Salary:".
           05  LINE 10 COLUMN 20 PIC 9(7).99 USING WS-EMP-SALARY
               REQUIRED.

           05  LINE 12 COLUMN 5  VALUE "Password:".
           05  LINE 12 COLUMN 20 PIC X(12) USING WS-EMP-PASSWORD
               SECURE REQUIRED.

           05  LINE 16 COLUMN 5  VALUE "Confirm (Y/N): ".
           05  LINE 16 COLUMN 20 PIC X USING WS-CONFIRM
               AUTO.

       01  RESULT-SCREEN.
           05  BLANK SCREEN.
           05  LINE 1  COLUMN 20 VALUE "EMPLOYEE SAVED"
               HIGHLIGHT.
           05  LINE 3  COLUMN 5  VALUE "ID:".
           05  LINE 3  COLUMN 20 PIC X(6) FROM WS-EMP-ID.
           05  LINE 5  COLUMN 5  VALUE "Name:".
           05  LINE 5  COLUMN 20 PIC X(30) FROM WS-EMP-NAME.
           05  LINE 7  COLUMN 5  VALUE "Department:".
           05  LINE 7  COLUMN 20 PIC X(4) FROM WS-EMP-DEPT.
           05  LINE 9  COLUMN 5  VALUE "Salary:".
           05  LINE 9  COLUMN 20 PIC $,$$$,$$9.99
               FROM WS-EMP-SALARY.
           05  LINE 13 COLUMN 5
               VALUE "Press Enter to continue..."
               LOWLIGHT.

       PROCEDURE DIVISION.
       MAIN-LOGIC.
           INITIALIZE WS-EMPLOYEE
           MOVE SPACE TO WS-CONFIRM

           DISPLAY ENTRY-SCREEN
           ACCEPT ENTRY-SCREEN

           IF CONFIRMED
               DISPLAY RESULT-SCREEN
               ACCEPT RESULT-SCREEN
           END-IF

           STOP RUN.
```

### Color and Attribute Demonstration

```cobol
       SCREEN SECTION.
       01  COLOR-SCREEN.
           05  BLANK SCREEN
               BACKGROUND-COLOR 0.
           05  LINE 2  COLUMN 5
               VALUE "Normal text".
           05  LINE 3  COLUMN 5
               VALUE "Bold text"
               HIGHLIGHT.
           05  LINE 4  COLUMN 5
               VALUE "Dim text"
               LOWLIGHT.
           05  LINE 5  COLUMN 5
               VALUE "Underlined"
               UNDERLINE.
           05  LINE 6  COLUMN 5
               VALUE "Reversed"
               REVERSE-VIDEO.
           05  LINE 7  COLUMN 5
               VALUE "Blinking"
               BLINK.
           05  LINE 9  COLUMN 5
               VALUE "Red on Black"
               FOREGROUND-COLOR 4
               BACKGROUND-COLOR 0.
           05  LINE 10 COLUMN 5
               VALUE "Yellow on Blue"
               FOREGROUND-COLOR 6
               BACKGROUND-COLOR 1.
           05  LINE 11 COLUMN 5
               VALUE "White on Green"
               FOREGROUND-COLOR 7
               BACKGROUND-COLOR 2.
```

### Menu Selection Screen

```cobol
       WORKING-STORAGE SECTION.
       01  WS-CHOICE              PIC 9.
           88  VALID-CHOICE       VALUE 1 THRU 5.

       SCREEN SECTION.
       01  MENU-SCREEN.
           05  BLANK SCREEN.
           05  LINE 2  COLUMN 25
               VALUE "MAIN MENU" HIGHLIGHT.
           05  LINE 2  COLUMN 60 VALUE "v1.0"
               LOWLIGHT.
           05  LINE 4  COLUMN 20
               VALUE "1. Add Record".
           05  LINE 5  COLUMN 20
               VALUE "2. Update Record".
           05  LINE 6  COLUMN 20
               VALUE "3. Delete Record".
           05  LINE 7  COLUMN 20
               VALUE "4. Display Record".
           05  LINE 8  COLUMN 20
               VALUE "5. Exit".
           05  LINE 10 COLUMN 20
               VALUE "Enter choice: ".
           05  LINE 10 COLUMN 35
               PIC 9 USING WS-CHOICE
               AUTO REQUIRED HIGHLIGHT
               FOREGROUND-COLOR 6.
```

---

## See Also

- [Data Division](index.md)
- [Level Numbers](level-numbers.md)
- [PICTURE Clause](picture.md)
- [VALUE Clause](value.md)
- [Condition Names (Level 88)](condition-names.md)
- [USAGE Clause](usage.md)
- [IF Statement](../procedure-division/control-flow/if.md)
- [EVALUATE Statement](../procedure-division/control-flow/evaluate.md)
- [PERFORM Statement](../procedure-division/control-flow/perform.md)
- [MOVE Statement](../procedure-division/data-movement/move.md)
- [ACCEPT](../procedure-division/io/accept.md) -- screen input
- [DISPLAY](../procedure-division/io/display.md) -- screen output
- [SPECIAL-NAMES](../environment-division/special-names.md) -- CURSOR and CRT STATUS
