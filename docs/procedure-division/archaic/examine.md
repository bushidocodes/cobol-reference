# EXAMINE (Removed)

The `EXAMINE` statement scanned a data item to tally and/or replace occurrences of a given character. It was the predecessor to the modern `INSPECT` statement.

- **Introduced:** COBOL-60
- **Replaced by:** `INSPECT` in COBOL-74
- **Status:** Removed

!!! warning "Removed"
    `EXAMINE` was replaced by `INSPECT` in COBOL-74. Use
    [INSPECT](../string-handling/inspect.md) in all new programs.

---

## Syntax

### Format 1: TALLYING

```cobol
EXAMINE data-name TALLYING { ALL | LEADING | UNTIL FIRST }
    literal-1 [ REPLACING BY literal-2 ]
```

### Format 2: REPLACING

```cobol
EXAMINE data-name REPLACING { ALL | LEADING | FIRST | UNTIL FIRST }
    literal-1 BY literal-2
```

---

## Rules

1. `data-name` must be a USAGE DISPLAY item.
2. `literal-1` and `literal-2` must each be a single character.
3. When TALLYING is specified, the count of matched characters is placed in the special register **TALLY** (a system-defined numeric data item).
4. `ALL` counts or replaces every occurrence of the character.
5. `LEADING` counts or replaces occurrences at the beginning of the item, stopping at the first non-matching character.
6. `UNTIL FIRST` counts or replaces characters from the beginning of the item until the first occurrence of `literal-1` is found.
7. `FIRST` (Format 2 only) replaces only the first occurrence.

---

## The TALLY Special Register

`TALLY` was a compiler-defined numeric data item automatically available to every COBOL program. When `EXAMINE ... TALLYING` was executed, the count was placed in TALLY. The programmer could also use TALLY as a general-purpose counter.

```cobol
EXAMINE DEPARTMENT-NUMBER TALLYING LEADING ZERO
    REPLACING BY SPACE
*> TALLY now contains the count of leading zeros
*> The leading zeros have been replaced by spaces
```

TALLY was removed along with EXAMINE when INSPECT was introduced. INSPECT uses explicit counter fields specified by the programmer.

---

## Examples

### Counting Characters

```cobol
EXAMINE EMPLOYEE-NAME TALLYING ALL SPACE
*> TALLY = number of spaces in EMPLOYEE-NAME
```

### Replacing Leading Zeros with Spaces

```cobol
EXAMINE ACCOUNT-NUMBER TALLYING LEADING ZERO
    REPLACING BY SPACE
*> TALLY = count of leading zeros removed
*> Leading zeros in ACCOUNT-NUMBER are now spaces
```

### Replacing All Occurrences

```cobol
EXAMINE INPUT-FIELD REPLACING ALL "," BY "."
*> All commas replaced with periods
```

### Counting Until First Occurrence

```cobol
EXAMINE TEXT-LINE TALLYING UNTIL FIRST "*"
*> TALLY = number of characters before the first asterisk
```

---

## Migration to INSPECT

| EXAMINE | INSPECT Equivalent |
|---------|-------------------|
| `EXAMINE data TALLYING ALL char` | `INSPECT data TALLYING counter FOR ALL char` |
| `EXAMINE data TALLYING LEADING char` | `INSPECT data TALLYING counter FOR LEADING char` |
| `EXAMINE data REPLACING ALL char-1 BY char-2` | `INSPECT data REPLACING ALL char-1 BY char-2` |
| `EXAMINE data REPLACING LEADING char-1 BY char-2` | `INSPECT data REPLACING LEADING char-1 BY char-2` |
| `EXAMINE data REPLACING FIRST char-1 BY char-2` | `INSPECT data REPLACING FIRST char-1 BY char-2` |

Key differences:

- INSPECT supports multi-character patterns; EXAMINE was limited to single characters.
- INSPECT requires an explicit counter field; EXAMINE used the implicit TALLY register.
- INSPECT adds `CONVERTING` (COBOL-85) and `BEFORE/AFTER` phrases for bounded scanning.

---

## See Also

- [INSPECT](../string-handling/inspect.md) -- modern replacement
- [Special Registers](../../language/special-registers.md) -- TALLY register
