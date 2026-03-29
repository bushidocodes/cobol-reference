# DISPLAY

The `DISPLAY` statement outputs data to the console, printer, or another low-volume output device.

- **Standard:** COBOL-60, COBOL-68, COBOL-85, COBOL 2002, COBOL 2014, COBOL 2023
- **Division:** Procedure Division
- **Category:** Input/Output

---

## Syntax

### Format 1 — Console or Device Output

```cobol
DISPLAY { identifier-1 | literal-1 }
        [ { identifier-2 | literal-2 } ] ...
    [ UPON mnemonic-name ]
    [ WITH NO ADVANCING ]
```

### Format 2 — Screen (COBOL 2002)

```cobol
DISPLAY screen-name
    [ AT { LINE NUMBER | COLUMN NUMBER } { identifier | integer } ]
END-DISPLAY
```

---

## Rules

1. One or more identifiers or literals can be displayed in a single statement. They are concatenated in order without separators.
2. Without `UPON`, output goes to the default display device (typically the console/standard output).
3. `UPON mnemonic-name` directs output to a device defined in the SPECIAL-NAMES paragraph.
4. By default, a newline is appended after the last item.
5. `WITH NO ADVANCING` (COBOL-85) suppresses the trailing newline, keeping the cursor on the same line. This is useful for prompts.
6. DISPLAY is intended for **low-volume** output such as error messages, operator instructions, and debugging. For high-volume output, use WRITE with files.
7. Numeric items are converted to their display representation automatically.

---

## Examples

### Basic Output

```cobol
       DISPLAY "Hello, World!"
       DISPLAY "Processing complete. Records: " WS-COUNT
```

### Multiple Items

```cobol
       DISPLAY WS-CUST-NAME " owes " WS-BALANCE
       *> Items are concatenated: "John Smith owes 01250.75"
```

### Prompting with NO ADVANCING

```cobol
       DISPLAY "Enter your name: " WITH NO ADVANCING
       ACCEPT WS-NAME
```

### Output to a Specific Device

```cobol
       SPECIAL-NAMES.
           SYSERR IS ERR-OUTPUT.

       DISPLAY "Fatal error: file not found" UPON ERR-OUTPUT
```

### Debugging Output

```cobol
       DISPLAY "DEBUG: WS-STATUS = " WS-STATUS
               " WS-COUNT = " WS-COUNT
               " AT PARAGRAPH " "PROCESS-RECORDS"
```

---

## DISPLAY vs WRITE

| Aspect | DISPLAY | WRITE |
|--------|---------|-------|
| Purpose | Low-volume console/operator output | High-volume file output |
| Requires file definition | No | Yes (FD, SELECT) |
| Formatting control | Minimal (concatenation only) | Full (PICTURE, ADVANCING) |
| Performance | Not optimized for volume | Buffered I/O |
| Typical use | Messages, debugging, prompts | Reports, data files |

---

## See Also

- [ACCEPT](accept.md) -- console input
- [WRITE](write.md) -- file output
- [SPECIAL-NAMES](../../environment-division/special-names.md) -- device mappings
