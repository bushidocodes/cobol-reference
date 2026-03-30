# ACCEPT

The `ACCEPT` statement retrieves data from the console, system date/time, environment variables, or the command line.

- **Standard:** COBOL-60, COBOL-68, COBOL-85, COBOL 2002, COBOL 2014, COBOL 2023
- **Division:** Procedure Division
- **Category:** Input/Output

---

## Syntax

### Format 1 — From Console or Device

```cobol
ACCEPT identifier-1 [ FROM mnemonic-name ]
```

### Format 2 — From System Information

```cobol
ACCEPT identifier-1 FROM { DATE | DAY | DAY-OF-WEEK | TIME }
```

### Format 3 — From Environment (varies by compiler)

```cobol
ACCEPT identifier-1 FROM { ENVIRONMENT-NAME
                         | ENVIRONMENT-VALUE
                         | COMMAND-LINE }
```

### Format 4 — Screen (COBOL 2002)

```cobol
ACCEPT screen-name
    [ AT { LINE NUMBER | COLUMN NUMBER } { identifier | integer } ]
    [ ON EXCEPTION imperative-statement-1 ]
    [ NOT ON EXCEPTION imperative-statement-2 ]
END-ACCEPT
```

---

## Rules

### Console Input (Format 1)

1. Without `FROM`, data is read from the default input device (typically the console/terminal).
2. `FROM mnemonic-name` reads from a device defined in the SPECIAL-NAMES paragraph.
3. The data is moved to `identifier-1` according to standard MOVE rules.
4. ACCEPT is intended for **low-volume** input such as operator responses and runtime parameters. For high-volume data, use READ with files.

### System Information (Format 2)

| Source | Format | Length | Example |
|--------|--------|--------|---------|
| `DATE` | YYMMDD | 6 digits | 250322 |
| `DAY` | YYDDD | 5 digits | 25081 |
| `DAY-OF-WEEK` | D | 1 digit | 6 (Saturday) |
| `TIME` | HHMMSSCC | 8 digits | 14302500 |

- `DAY-OF-WEEK` returns 1 for Monday through 7 for Sunday.
- `TIME` includes hundredths of seconds in the last two digits.
- `DATE` uses a two-digit year. For four-digit years, use the `CURRENT-DATE` intrinsic function instead.

### Environment Variables (Format 3)

Reading environment variables typically requires SPECIAL-NAMES entries:

```cobol
SPECIAL-NAMES.
    ENVIRONMENT-NAME IS ENV-NAME
    ENVIRONMENT-VALUE IS ENV-VALUE.
```

Then in the Procedure Division:

```cobol
MOVE "HOME" TO ENV-NAME
ACCEPT WS-HOME FROM ENV-VALUE
```

`COMMAND-LINE` retrieves command-line arguments (compiler-dependent).

---

## Examples

### Console Input

```cobol
       DISPLAY "Enter customer ID: " WITH NO ADVANCING
       ACCEPT WS-CUST-ID
```

### System Date and Time

```cobol
       01  WS-DATE       PIC 9(6).
       01  WS-TIME       PIC 9(8).

       ACCEPT WS-DATE FROM DATE
       ACCEPT WS-TIME FROM TIME
       DISPLAY "Date: " WS-DATE " Time: " WS-TIME
```

### Environment Variable

```cobol
       MOVE "DATABASE_URL" TO ENV-NAME
       ACCEPT WS-DB-URL FROM ENV-VALUE
       IF WS-DB-URL = SPACES
           DISPLAY "DATABASE_URL not set"
           STOP RUN
       END-IF
```

---

## See Also

- [DISPLAY](display.md) -- console output
- [SPECIAL-NAMES](../../environment-division/special-names.md) -- device and environment mappings
- [CURRENT-DATE](../../intrinsic-functions/date-time.md) -- four-digit year date/time retrieval
- [Screen Section](../../data-division/screen-section.md) -- screen definitions for ACCEPT
