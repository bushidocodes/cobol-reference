# Identification Division

The Identification Division is the first and only required division in a COBOL program. It identifies the program and may provide additional documentary information.

**Standard:** COBOL-61, COBOL-61 Extended, COBOL-65, COBOL-68, COBOL-85, COBOL 2002, COBOL 2014, COBOL 2023

!!! info "Historical Note"
    The Identification Division was not part of the original COBOL-60 specification,
    which defined only three divisions (Procedure, Data, and Environment). It was
    first defined in COBOL-61 with seven paragraphs: PROGRAM-ID (required, must be
    first), AUTHOR, INSTALLATION, DATE-WRITTEN, DATE-COMPILED, SECURITY, and
    REMARKS (all optional). COBOL-61 also included the REMARKS paragraph, which was
    later dropped from formal standards. The documentary paragraphs (AUTHOR,
    INSTALLATION, DATE-WRITTEN, DATE-COMPILED, SECURITY) were classified as obsolete
    in COBOL-85 and removed from the standard in COBOL 2002, though most compilers
    still accept them for backward compatibility.

---

## Syntax

```cobol
IDENTIFICATION DIVISION.
PROGRAM-ID. program-name [IS INITIAL PROGRAM]
                         [IS COMMON PROGRAM]
                         [IS RECURSIVE PROGRAM].

[AUTHOR. comment-entry.]
[INSTALLATION. comment-entry.]
[DATE-WRITTEN. comment-entry.]
[DATE-COMPILED. comment-entry.]
[SECURITY. comment-entry.]
```

The division header `IDENTIFICATION DIVISION.` is required. The abbreviated form `ID DIVISION.` is accepted by most compilers.

## PROGRAM-ID paragraph

The `PROGRAM-ID` paragraph is the only required paragraph in the Identification Division. It specifies the name by which the program is identified to the operating system and to other COBOL programs.

```cobol
PROGRAM-ID. program-name [IS { INITIAL | COMMON | RECURSIVE } PROGRAM].
```

**program-name** is a user-defined word of up to 30 characters. Some compilers restrict the name further (e.g., 8 characters for certain mainframe environments).

### INITIAL clause

When the `INITIAL` clause is specified, the program is placed in its initial state every time it is called. All `WORKING-STORAGE` data items are reinitialized, all internal files are closed, and all `PERFORM` ranges are reset. This is useful for programs that must not retain state between invocations.

```cobol
PROGRAM-ID. RESET-HANDLER IS INITIAL PROGRAM.
```

### COMMON clause

The `COMMON` clause is valid only for programs contained within another program. A common program is available for calling by all programs contained within the same outermost program, not just the program that directly contains it.

```cobol
PROGRAM-ID. SHARED-UTIL IS COMMON PROGRAM.
```

### RECURSIVE clause

*COBOL 2002 and later.*

The `RECURSIVE` clause permits a program to be invoked while a previous invocation is still active. Without this clause, recursive invocation results in undefined behavior.

```cobol
PROGRAM-ID. FACTORIAL IS RECURSIVE PROGRAM.
```

When `RECURSIVE` is specified, each invocation of the program receives a separate copy of its `WORKING-STORAGE SECTION` data. The `LOCAL-STORAGE SECTION` always receives a fresh copy per invocation regardless of this clause.

## Optional documentary paragraphs

The following paragraphs provide comment information. They have no effect on program execution. All were classified as obsolete in COBOL 2002 but remain widely used in existing programs.

| Paragraph | Purpose |
|---|---|
| `AUTHOR` | Names the author of the program. |
| `INSTALLATION` | Identifies the installation where the program was written. |
| `DATE-WRITTEN` | Records the date the program was written. |
| `DATE-COMPILED` | Records the compilation date. Some compilers replace the comment entry with the actual compilation date and time. |
| `SECURITY` | Documents security restrictions or classification of the program. |

```cobol
AUTHOR. J. DOE.
INSTALLATION. HEADQUARTERS DATA CENTER.
DATE-WRITTEN. 2024-01-15.
DATE-COMPILED. 2024-02-01.
SECURITY. CONFIDENTIAL.
```

The content of each paragraph is treated as a comment entry. It has no syntactic constraints beyond the standard COBOL reference format rules.

## Nested programs (contained programs)

A COBOL source unit may contain one or more nested programs between the last line of the outermost program's `PROCEDURE DIVISION` and the `END PROGRAM` header. Each nested program has its own complete Identification Division.

```cobol
IDENTIFICATION DIVISION.
PROGRAM-ID. OUTER-PROG.
...
PROCEDURE DIVISION.
    CALL "INNER-CALC"
    STOP RUN.

    IDENTIFICATION DIVISION.
    PROGRAM-ID. INNER-CALC.
    ...
    PROCEDURE DIVISION.
        ...
    END PROGRAM INNER-CALC.

END PROGRAM OUTER-PROG.
```

A contained program may access only the data and files of its directly containing program if those items are declared with the `GLOBAL` clause. A contained program is not visible to sibling programs unless it specifies the `COMMON` clause.

Contained programs may themselves contain further programs, forming a nesting hierarchy.

## CLASS-ID, FACTORY, and OBJECT paragraphs

*COBOL 2002 and later.*

For object-oriented COBOL programs, the Identification Division uses `CLASS-ID` in place of `PROGRAM-ID` to define a class.

```cobol
IDENTIFICATION DIVISION.
CLASS-ID. CustomerAccount
    INHERITS Base.
```

Within a class definition, `FACTORY` and `OBJECT` paragraphs delineate factory (class-level) and instance-level definitions, respectively.

```cobol
FACTORY.
    ...
END FACTORY.

OBJECT.
    ...
END OBJECT.
```

These features are part of the OO COBOL extensions defined in COBOL 2002 and COBOL 2014. Compiler support varies.

## Examples

### Minimal Identification Division

```cobol
       IDENTIFICATION DIVISION.
       PROGRAM-ID. HELLO.
       PROCEDURE DIVISION.
           DISPLAY "HELLO, WORLD".
           STOP RUN.
```

### Full Identification Division

```cobol
       IDENTIFICATION DIVISION.
       PROGRAM-ID. INV-REPORT IS INITIAL PROGRAM.
       AUTHOR. J. DOE.
       INSTALLATION. CENTRAL DATA CENTER.
       DATE-WRITTEN. 2024-01-15.
       DATE-COMPILED.
       SECURITY. UNRESTRICTED.

       ENVIRONMENT DIVISION.
       ...
```

### Identification Division with RECURSIVE

```cobol
       IDENTIFICATION DIVISION.
       PROGRAM-ID. FACTORIAL IS RECURSIVE PROGRAM.

       DATA DIVISION.
       WORKING-STORAGE SECTION.
       01  WS-NUM      PIC 9(10) VALUE 0.
       01  WS-RESULT   PIC 9(18) VALUE 0.

       LOCAL-STORAGE SECTION.
       01  LS-N        PIC 9(10).
       01  LS-SUB      PIC 9(18).

       PROCEDURE DIVISION.
           ACCEPT WS-NUM
           MOVE WS-NUM TO LS-N
           IF LS-N <= 1
               MOVE 1 TO WS-RESULT
           ELSE
               SUBTRACT 1 FROM LS-N GIVING WS-NUM
               CALL "FACTORIAL"
               MULTIPLY LS-N BY WS-RESULT
           END-IF
           DISPLAY WS-RESULT
           STOP RUN.
```

## See also

- [Program Attributes (INITIAL, COMMON, RECURSIVE)](program-attributes.md)
- [Environment Division](../environment-division/index.md)
- [Data Division](../data-division/index.md)
- [Procedure Division](../procedure-division/index.md)
- [User-Defined Functions (FUNCTION-ID)](../intrinsic-functions/user-defined.md)
