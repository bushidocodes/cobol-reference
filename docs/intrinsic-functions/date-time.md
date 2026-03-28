# Date and Time Functions

Intrinsic functions that retrieve, convert, and manipulate date and time values.

**Standard:** COBOL-85 Amendment 1 (1989), COBOL 2002, COBOL 2014, COBOL 2023

---

## General Syntax

```cobol
FUNCTION function-name [ ( argument-1 [ argument-2 ] ... ) ]
```

Date and time intrinsic functions return values of category **alphanumeric** or **integer** depending on the function. Integer date functions use a Lilian date system in which day 1 corresponds to January 1, 1601. This base date enables conversion and arithmetic across the full range of dates supported by the standard.

---

## The Lilian Date System

The functions INTEGER-OF-DATE, INTEGER-OF-DAY, DATE-OF-INTEGER, and DAY-OF-INTEGER use an integer date representation where:

- **Day 1** corresponds to **January 1, 1601** (the beginning of a 400-year Gregorian cycle).
- **Day 146097** corresponds to December 31, 2000.
- The system accounts for leap years according to the Gregorian calendar.

This representation enables date arithmetic by converting dates to integers, performing arithmetic, and converting back.

---

## COMBINED-DATETIME

Combines an integer date and seconds past midnight into a single numeric value.

!!! note "COBOL 2002"
    The COMBINED-DATETIME function was introduced in COBOL 2002.

```cobol
FUNCTION COMBINED-DATETIME ( argument-1  argument-2 )
```

- **argument-1** -- integer. A Lilian day number representing the date.
- **argument-2** -- numeric. The number of seconds past midnight (0 through 86400).
- **Return category** -- numeric.
- **Behavior** -- returns a single numeric value that encodes both the date and time components. The result is suitable for comparison and arithmetic, allowing two date-time values to be compared directly or subtracted to find the difference.

```cobol
COMPUTE WS-DATE-INT =
    FUNCTION INTEGER-OF-DATE(20250322)
COMPUTE WS-COMBINED =
    FUNCTION COMBINED-DATETIME(WS-DATE-INT 52245)
*> Encodes March 22, 2025 at 14:30:45

*> Compare two date-time values
IF WS-COMBINED-1 > WS-COMBINED-2
    DISPLAY "Event 1 is later"
END-IF
```

---

## CURRENT-DATE

Returns the current date, time, and UTC offset as a 21-character alphanumeric value.

```cobol
FUNCTION CURRENT-DATE
```

- **Arguments** -- none.
- **Return category** -- alphanumeric, length 21.
- **Behavior** -- returns a 21-character string with the following layout:

| Positions | Content | Format |
|---|---|---|
| 1-4 | Year | YYYY |
| 5-6 | Month | MM (01-12) |
| 7-8 | Day | DD (01-31) |
| 9-10 | Hours | HH (00-23) |
| 11-12 | Minutes | MM (00-59) |
| 13-14 | Seconds | SS (00-59) |
| 15-16 | Hundredths of seconds | HH (00-99) |
| 17 | UTC offset sign | "+" or "-" or "0" |
| 18-19 | UTC offset hours | HH (00-13) |
| 20-21 | UTC offset minutes | MM (00-59) |

If the system cannot determine the UTC offset, position 17 contains "0" and positions 18-21 contain "0000".

```cobol
       01  WS-CURRENT-DATE.
           05  WS-DATE.
               10  WS-YEAR    PIC 9(4).
               10  WS-MONTH   PIC 9(2).
               10  WS-DAY     PIC 9(2).
           05  WS-TIME.
               10  WS-HOURS   PIC 9(2).
               10  WS-MINS    PIC 9(2).
               10  WS-SECS    PIC 9(2).
               10  WS-HSECS   PIC 9(2).
           05  WS-UTC-OFFSET.
               10  WS-UTC-SIGN  PIC X(1).
               10  WS-UTC-HRS   PIC 9(2).
               10  WS-UTC-MINS  PIC 9(2).

       PROCEDURE DIVISION.
           MOVE FUNCTION CURRENT-DATE TO WS-CURRENT-DATE
           DISPLAY "Today is " WS-YEAR "/" WS-MONTH "/" WS-DAY
           DISPLAY "Time is " WS-HOURS ":" WS-MINS ":" WS-SECS
```

---

## DATE-OF-INTEGER

Returns the standard date (YYYYMMDD) corresponding to an integer date.

```cobol
FUNCTION DATE-OF-INTEGER ( argument-1 )
```

- **argument-1** -- integer, positive. Represents a Lilian day number.
- **Return category** -- integer (8 digits in YYYYMMDD format).
- **Behavior** -- converts a Lilian day number to a standard date in YYYYMMDD format. The argument must be a valid day number within the supported date range.

```cobol
COMPUTE WS-DATE = FUNCTION DATE-OF-INTEGER(145732)
*> WS-DATE = 19991231 (December 31, 1999)

COMPUTE WS-TOMORROW =
    FUNCTION DATE-OF-INTEGER(
        FUNCTION INTEGER-OF-DATE(WS-TODAY) + 1)
```

---

## DATE-TO-YYYYMMDD

Converts a six-digit date (YYMMDD) to an eight-digit date (YYYYMMDD) using a windowing rule.

!!! note "COBOL 2002"
    The DATE-TO-YYYYMMDD function was introduced in COBOL 2002.

```cobol
FUNCTION DATE-TO-YYYYMMDD ( argument-1 [ argument-2 [ argument-3 ] ] )
```

- **argument-1** -- integer, a six-digit date in YYMMDD format.
- **argument-2** -- optional, integer. The ending year of the 100-year window, expressed as an offset from the current year. Default is 50.
- **argument-3** -- optional, integer. The current year override (four digits, YYYY). Default is the actual current year.
- **Return category** -- integer (8 digits in YYYYMMDD format).
- **Behavior** -- converts a two-digit year date to a four-digit year date. The function determines the century by placing the two-digit year within a 100-year window. The window ends at the current year plus argument-2. For example, with the default window offset of 50, in the year 2025 the window covers 1976 through 2075.

```cobol
*> Assuming current year is 2025, default window 1976-2075
COMPUTE WS-DATE =
    FUNCTION DATE-TO-YYYYMMDD(251225)
*> WS-DATE = 20251225

COMPUTE WS-DATE =
    FUNCTION DATE-TO-YYYYMMDD(990101)
*> WS-DATE = 19990101

*> Explicit window: current year + 20
COMPUTE WS-DATE =
    FUNCTION DATE-TO-YYYYMMDD(500601 20)
*> Window 1946-2045: WS-DATE = 19500601
```

---

## DAY-OF-INTEGER

Returns the Julian date (YYYYDDD) corresponding to an integer date.

```cobol
FUNCTION DAY-OF-INTEGER ( argument-1 )
```

- **argument-1** -- integer, positive. Represents a Lilian day number.
- **Return category** -- integer (7 digits in YYYYDDD format).
- **Behavior** -- converts a Lilian day number to a Julian date in YYYYDDD format, where DDD is the ordinal day of the year (001-366).

```cobol
COMPUTE WS-JULIAN = FUNCTION DAY-OF-INTEGER(145732)
*> WS-JULIAN = 1999365 (day 365 of 1999)
```

---

## DAY-TO-YYYYDDD

Converts a five-digit Julian date (YYDDD) to a seven-digit Julian date (YYYYDDD) using a windowing rule.

!!! note "COBOL 2002"
    The DAY-TO-YYYYDDD function was introduced in COBOL 2002.

```cobol
FUNCTION DAY-TO-YYYYDDD ( argument-1 [ argument-2 [ argument-3 ] ] )
```

- **argument-1** -- integer, a five-digit Julian date in YYDDD format.
- **argument-2** -- optional, integer. The ending year offset for the 100-year window. Default is 50.
- **argument-3** -- optional, integer. The current year override (YYYY). Default is the actual current year.
- **Return category** -- integer (7 digits in YYYYDDD format).
- **Behavior** -- converts a two-digit year Julian date to a four-digit year Julian date using the same windowing logic as DATE-TO-YYYYMMDD.

```cobol
COMPUTE WS-JULIAN =
    FUNCTION DAY-TO-YYYYDDD(25180)
*> WS-JULIAN = 2025180 (day 180 of 2025)
```

---

## FORMATTED-CURRENT-DATE

Returns the current date and time formatted according to a specified format string.

!!! note "COBOL 2014"
    The FORMATTED-CURRENT-DATE function was introduced in COBOL 2014 as part of the formatted date/time family.

```cobol
FUNCTION FORMATTED-CURRENT-DATE ( argument-1 )
```

- **argument-1** -- alphanumeric. A format string using patterns such as "YYYY-MM-DDThh:mm:ss" to describe the desired output layout.
- **Return category** -- alphanumeric.
- **Behavior** -- returns the current date and time formatted according to the format string specified in argument-1. Supports ISO 8601 formats. The format string determines which date and time components are included and how they are separated.

```cobol
MOVE FUNCTION FORMATTED-CURRENT-DATE(
    "YYYY-MM-DDThh:mm:ss") TO WS-ISO-DATETIME
*> e.g. "2025-03-22T14:30:45"

MOVE FUNCTION FORMATTED-CURRENT-DATE(
    "YYYYMMDD") TO WS-DATE-ONLY
*> e.g. "20250322"

MOVE FUNCTION FORMATTED-CURRENT-DATE(
    "YYYY-MM-DDThh:mm:ss+hh:mm") TO WS-WITH-OFFSET
*> e.g. "2025-03-22T14:30:45-05:00"
```

---

## FORMATTED-DATE

Formats an integer date according to a format string.

!!! note "COBOL 2014"
    The FORMATTED-DATE function was introduced in COBOL 2014 as part of the formatted date/time family.

```cobol
FUNCTION FORMATTED-DATE ( argument-1  argument-2 )
```

- **argument-1** -- alphanumeric. A format string describing the desired output layout (e.g., "YYYY-MM-DD").
- **argument-2** -- integer. A Lilian day number.
- **Return category** -- alphanumeric.
- **Behavior** -- returns a string containing the date corresponding to the Lilian day number in argument-2, formatted according to the format string in argument-1.

```cobol
COMPUTE WS-LILIAN =
    FUNCTION INTEGER-OF-DATE(20250322)
MOVE FUNCTION FORMATTED-DATE(
    "YYYY-MM-DD" WS-LILIAN) TO WS-DISPLAY
*> WS-DISPLAY = "2025-03-22"

MOVE FUNCTION FORMATTED-DATE(
    "DD/MM/YYYY" WS-LILIAN) TO WS-DISPLAY
*> WS-DISPLAY = "22/03/2025"
```

---

## FORMATTED-DATETIME

Formats a combined date-time value according to a format string.

!!! note "COBOL 2014"
    The FORMATTED-DATETIME function was introduced in COBOL 2014 as part of the formatted date/time family.

```cobol
FUNCTION FORMATTED-DATETIME ( argument-1  argument-2
    argument-3  [ argument-4 ] )
```

- **argument-1** -- alphanumeric. A format string describing the desired output layout.
- **argument-2** -- integer. A Lilian day number.
- **argument-3** -- numeric. The number of seconds past midnight (0 through 86400).
- **argument-4** -- optional, integer. The UTC offset in minutes. When provided, the offset is included in the formatted output.
- **Return category** -- alphanumeric.
- **Behavior** -- returns a string containing the date and time formatted according to the format string in argument-1. The date is specified by the Lilian day number in argument-2 and the time by seconds past midnight in argument-3. If argument-4 is supplied, the UTC offset is appended.

```cobol
COMPUTE WS-LILIAN =
    FUNCTION INTEGER-OF-DATE(20250322)
MOVE FUNCTION FORMATTED-DATETIME(
    "YYYY-MM-DDThh:mm:ss" WS-LILIAN 52245)
    TO WS-DISPLAY
*> WS-DISPLAY = "2025-03-22T14:30:45"

*> With UTC offset (-5 hours = -300 minutes)
MOVE FUNCTION FORMATTED-DATETIME(
    "YYYY-MM-DDThh:mm:ss+hh:mm"
    WS-LILIAN 52245 -300)
    TO WS-DISPLAY
*> WS-DISPLAY = "2025-03-22T14:30:45-05:00"
```

---

## FORMATTED-TIME

Formats a time value according to a format string.

!!! note "COBOL 2014"
    The FORMATTED-TIME function was introduced in COBOL 2014 as part of the formatted date/time family.

```cobol
FUNCTION FORMATTED-TIME ( argument-1  argument-2
    [ argument-3 ] )
```

- **argument-1** -- alphanumeric. A format string describing the desired output layout (e.g., "hh:mm:ss").
- **argument-2** -- numeric. The number of seconds past midnight (0 through 86400).
- **argument-3** -- optional, integer. The UTC offset in minutes.
- **Return category** -- alphanumeric.
- **Behavior** -- returns a string containing the time formatted according to the format string in argument-1. The time is specified by seconds past midnight in argument-2. If argument-3 is supplied, the UTC offset is included in the output.

```cobol
MOVE FUNCTION FORMATTED-TIME(
    "hh:mm:ss" 52245) TO WS-DISPLAY
*> WS-DISPLAY = "14:30:45"

MOVE FUNCTION FORMATTED-TIME(
    "hh:mm:ss+hh:mm" 52245 -300) TO WS-DISPLAY
*> WS-DISPLAY = "14:30:45-05:00"
```

---

## INTEGER-OF-DATE

Returns the Lilian day number corresponding to a standard date (YYYYMMDD).

```cobol
FUNCTION INTEGER-OF-DATE ( argument-1 )
```

- **argument-1** -- integer, a valid standard date in YYYYMMDD format. The year must be in the range 1601 through 9999.
- **Return category** -- integer.
- **Behavior** -- returns the Lilian day number corresponding to the specified date. January 1, 1601 returns 1. The argument must represent a valid date; behavior is undefined for invalid dates (such as month 13 or day 32).

```cobol
COMPUTE WS-LILIAN =
    FUNCTION INTEGER-OF-DATE(20250101)
*> WS-LILIAN = the Lilian day number for January 1, 2025

*> Calculate days between two dates
COMPUTE WS-DAYS =
    FUNCTION INTEGER-OF-DATE(WS-END-DATE)
  - FUNCTION INTEGER-OF-DATE(WS-START-DATE)
```

---

## INTEGER-OF-DAY

Returns the Lilian day number corresponding to a Julian date (YYYYDDD).

```cobol
FUNCTION INTEGER-OF-DAY ( argument-1 )
```

- **argument-1** -- integer, a valid Julian date in YYYYDDD format. The year must be in the range 1601 through 9999.
- **Return category** -- integer.
- **Behavior** -- returns the Lilian day number corresponding to the specified Julian date. This function is the inverse of DAY-OF-INTEGER.

```cobol
COMPUTE WS-LILIAN =
    FUNCTION INTEGER-OF-DAY(2025001)
*> Lilian day number for January 1, 2025

COMPUTE WS-LILIAN =
    FUNCTION INTEGER-OF-DAY(2025365)
*> Lilian day number for December 31, 2025
```

---

## INTEGER-OF-FORMATTED-DATE

Returns the Lilian day number from a formatted date string.

!!! note "COBOL 2014"
    The INTEGER-OF-FORMATTED-DATE function was introduced in COBOL 2014 as part of the formatted date/time family.

```cobol
FUNCTION INTEGER-OF-FORMATTED-DATE ( argument-1  argument-2 )
```

- **argument-1** -- alphanumeric. A format string describing the layout of argument-2 (e.g., "YYYY-MM-DD").
- **argument-2** -- alphanumeric. The formatted date string to parse.
- **Return category** -- integer.
- **Behavior** -- parses the formatted date string in argument-2 according to the format specified in argument-1 and returns the corresponding Lilian day number. This function is the inverse of FORMATTED-DATE.

```cobol
COMPUTE WS-LILIAN =
    FUNCTION INTEGER-OF-FORMATTED-DATE(
        "YYYY-MM-DD" "2025-03-22")
*> Returns the Lilian day number for March 22, 2025

*> Round-trip: format then parse
COMPUTE WS-LILIAN-2 =
    FUNCTION INTEGER-OF-FORMATTED-DATE(
        "DD/MM/YYYY"
        FUNCTION FORMATTED-DATE(
            "DD/MM/YYYY" WS-LILIAN))
*> WS-LILIAN-2 = WS-LILIAN
```

---

## SECONDS-FROM-FORMATTED-TIME

Returns the number of seconds from a formatted time string.

!!! note "COBOL 2002"
    The SECONDS-FROM-FORMATTED-TIME function was introduced in COBOL 2002.

```cobol
FUNCTION SECONDS-FROM-FORMATTED-TIME ( argument-1  argument-2 )
```

- **argument-1** -- alphanumeric. A format string describing the layout of argument-2. The format string uses "hh" for hours, "mm" for minutes, and "ss" for seconds, with any separators.
- **argument-2** -- alphanumeric. The formatted time string to parse.
- **Return category** -- numeric.
- **Behavior** -- parses argument-2 according to the format specified in argument-1 and returns the total number of seconds. The format string uses "hh" for the hours component, "mm" for the minutes component, and "ss" for the seconds component. Fractional seconds beyond whole seconds are included if present.

```cobol
COMPUTE WS-SECS =
    FUNCTION SECONDS-FROM-FORMATTED-TIME(
        "hh:mm:ss" "14:30:45")
*> WS-SECS = 52245 (14*3600 + 30*60 + 45)

COMPUTE WS-SECS =
    FUNCTION SECONDS-FROM-FORMATTED-TIME(
        "hhmm" "1430")
*> WS-SECS = 52200 (14*3600 + 30*60)
```

---

## SECONDS-PAST-MIDNIGHT

Returns the number of seconds past midnight at the time of invocation.

```cobol
FUNCTION SECONDS-PAST-MIDNIGHT
```

- **Arguments** -- none.
- **Return category** -- numeric.
- **Behavior** -- returns the number of seconds that have elapsed since midnight (00:00:00) of the current day. The precision is implementation-defined but is at least to whole seconds. The value is in the range 0 through 86399.

```cobol
COMPUTE WS-SECS = FUNCTION SECONDS-PAST-MIDNIGHT
*> If the time is 14:30:45, WS-SECS = 52245

*> Timing a section of code
COMPUTE WS-START = FUNCTION SECONDS-PAST-MIDNIGHT
PERFORM LONG-RUNNING-PROCESS
COMPUTE WS-END = FUNCTION SECONDS-PAST-MIDNIGHT
COMPUTE WS-ELAPSED = WS-END - WS-START
DISPLAY "Elapsed: " WS-ELAPSED " seconds"
```

---

## TEST-DATE-YYYYMMDD

Tests whether a value is a valid YYYYMMDD date.

!!! note "COBOL 2002"
    The TEST-DATE-YYYYMMDD function was introduced in COBOL 2002.

```cobol
FUNCTION TEST-DATE-YYYYMMDD ( argument-1 )
```

- **argument-1** -- integer. A value to be tested as a YYYYMMDD date.
- **Return category** -- integer.
- **Behavior** -- tests whether argument-1 represents a valid standard date in YYYYMMDD format. Returns 0 if the date is valid. If the date is invalid, returns a nonzero value indicating the position of the first invalid component (e.g., invalid month, invalid day for the given month, invalid leap year day).

```cobol
COMPUTE WS-RESULT =
    FUNCTION TEST-DATE-YYYYMMDD(20250322)
*> WS-RESULT = 0 (valid date)

COMPUTE WS-RESULT =
    FUNCTION TEST-DATE-YYYYMMDD(20251301)
*> WS-RESULT is nonzero (month 13 is invalid)

COMPUTE WS-RESULT =
    FUNCTION TEST-DATE-YYYYMMDD(20250229)
*> WS-RESULT is nonzero (2025 is not a leap year)
```

---

## TEST-DAY-YYYYDDD

Tests whether a value is a valid YYYYDDD Julian date.

!!! note "COBOL 2002"
    The TEST-DAY-YYYYDDD function was introduced in COBOL 2002.

```cobol
FUNCTION TEST-DAY-YYYYDDD ( argument-1 )
```

- **argument-1** -- integer. A value to be tested as a YYYYDDD Julian date.
- **Return category** -- integer.
- **Behavior** -- tests whether argument-1 represents a valid Julian date in YYYYDDD format. Returns 0 if the date is valid. If the date is invalid, returns a nonzero value.

```cobol
COMPUTE WS-RESULT =
    FUNCTION TEST-DAY-YYYYDDD(2025180)
*> WS-RESULT = 0 (valid Julian date: day 180 of 2025)

COMPUTE WS-RESULT =
    FUNCTION TEST-DAY-YYYYDDD(2025366)
*> WS-RESULT is nonzero (2025 is not a leap year,
*> so day 366 is invalid)
```

---

## TEST-FORMATTED-DATETIME

Tests whether a string is a valid formatted date/time according to a format string.

!!! note "COBOL 2014"
    The TEST-FORMATTED-DATETIME function was introduced in COBOL 2014 as part of the formatted date/time family.

```cobol
FUNCTION TEST-FORMATTED-DATETIME ( argument-1  argument-2 )
```

- **argument-1** -- alphanumeric. A format string describing the expected layout (e.g., "YYYY-MM-DDThh:mm:ss").
- **argument-2** -- alphanumeric. The string to be tested.
- **Return category** -- integer.
- **Behavior** -- tests whether argument-2 is a valid date, time, or date-time string according to the format specified in argument-1. Returns 0 if the string is valid. If the string is invalid, returns a nonzero value indicating the position of the first error.

```cobol
COMPUTE WS-RESULT =
    FUNCTION TEST-FORMATTED-DATETIME(
        "YYYY-MM-DD" "2025-03-22")
*> WS-RESULT = 0 (valid)

COMPUTE WS-RESULT =
    FUNCTION TEST-FORMATTED-DATETIME(
        "YYYY-MM-DD" "2025-13-01")
*> WS-RESULT is nonzero (month 13 is invalid)

*> Validate before parsing
IF FUNCTION TEST-FORMATTED-DATETIME(
    "YYYY-MM-DD" WS-INPUT-DATE) = 0
    COMPUTE WS-LILIAN =
        FUNCTION INTEGER-OF-FORMATTED-DATE(
            "YYYY-MM-DD" WS-INPUT-DATE)
ELSE
    DISPLAY "Invalid date format"
END-IF
```

---

## WHEN-COMPILED

Returns the date and time the program was compiled.

```cobol
FUNCTION WHEN-COMPILED
```

- **Arguments** -- none.
- **Return category** -- alphanumeric, length 21.
- **Behavior** -- returns the date and time when the source program was compiled. The format is identical to CURRENT-DATE (YYYYMMDDHHMMSSHHGMMMM -- a 21-character string including the UTC offset). The value is fixed at compile time and does not change during execution.

```cobol
       01  WS-COMPILED    PIC X(21).
       01  WS-COMP-DATE   REDEFINES WS-COMPILED.
           05  WS-COMP-YEAR   PIC 9(4).
           05  WS-COMP-MONTH  PIC 9(2).
           05  WS-COMP-DAY    PIC 9(2).
           05  FILLER         PIC X(13).

       PROCEDURE DIVISION.
           MOVE FUNCTION WHEN-COMPILED TO WS-COMPILED
           DISPLAY "Compiled on "
               WS-COMP-YEAR "/" WS-COMP-MONTH "/" WS-COMP-DAY
```

---

## YEAR-TO-YYYY

Converts a two-digit year to a four-digit year using a windowing rule.

!!! note "COBOL 2002"
    The YEAR-TO-YYYY function was introduced in COBOL 2002.

```cobol
FUNCTION YEAR-TO-YYYY ( argument-1 [ argument-2 [ argument-3 ] ] )
```

- **argument-1** -- integer, a two-digit year (0-99).
- **argument-2** -- optional, integer. The ending year offset for the 100-year window. Default is 50.
- **argument-3** -- optional, integer. The current year override (YYYY). Default is the actual current year.
- **Return category** -- integer (4 digits).
- **Behavior** -- converts a two-digit year to a four-digit year using the same windowing rule as DATE-TO-YYYYMMDD. The century is determined by placing argument-1 within a 100-year window ending at the current year (or argument-3) plus argument-2.

```cobol
*> Assuming current year is 2025, default window 1976-2075
COMPUTE WS-YEAR = FUNCTION YEAR-TO-YYYY(25)
*> WS-YEAR = 2025

COMPUTE WS-YEAR = FUNCTION YEAR-TO-YYYY(99)
*> WS-YEAR = 1999

COMPUTE WS-YEAR = FUNCTION YEAR-TO-YYYY(50 20)
*> Window 1946-2045: WS-YEAR = 1950
```

---

## LOCALE-DATE

Returns a date formatted according to the conventions of a specified locale.

!!! note "COBOL 2002"
    The LOCALE-DATE function was introduced in COBOL 2002.

```cobol
FUNCTION LOCALE-DATE ( argument-1 [ argument-2 ] )
```

- **argument-1** -- integer, a standard date in YYYYMMDD format.
- **argument-2** -- optional, alphanumeric. The locale identifier. If omitted, the system default locale is used.
- **Return category** -- alphanumeric.
- **Behavior** -- returns a string containing the date formatted according to the date conventions of the specified locale. The format and length of the returned string are locale-dependent (e.g., "03/22/2025" for en_US, "22.03.2025" for de_DE, "22/03/2025" for en_GB).

```cobol
MOVE FUNCTION LOCALE-DATE(20250322) TO WS-DISPLAY-DATE
*> Result depends on system locale, e.g. "03/22/2025"

MOVE FUNCTION LOCALE-DATE(20250322 "de_DE")
    TO WS-DISPLAY-DATE
*> Result: "22.03.2025" (German locale)

MOVE FUNCTION LOCALE-DATE(20250322 "ja_JP")
    TO WS-DISPLAY-DATE
*> Result: "2025/03/22" (Japanese locale)
```

### Practical Example

```cobol
       WORKING-STORAGE SECTION.
       01  WS-TODAY         PIC 9(8).
       01  WS-FORMATTED     PIC X(30).

       PROCEDURE DIVISION.
           MOVE FUNCTION CURRENT-DATE(1:8) TO WS-TODAY
           MOVE FUNCTION LOCALE-DATE(WS-TODAY)
               TO WS-FORMATTED
           DISPLAY "Today: " WS-FORMATTED
           STOP RUN.
```

---

## LOCALE-TIME

Returns a time formatted according to the conventions of a specified locale.

!!! note "COBOL 2002"
    The LOCALE-TIME function was introduced in COBOL 2002.

```cobol
FUNCTION LOCALE-TIME ( argument-1 [ argument-2 ] )
```

- **argument-1** -- numeric. The number of seconds past midnight (0 through 86399).
- **argument-2** -- optional, alphanumeric. The locale identifier. If omitted, the system default locale is used.
- **Return category** -- alphanumeric.
- **Behavior** -- returns a string containing the time formatted according to the time conventions of the specified locale. The format and length of the returned string are locale-dependent (e.g., "2:30:45 PM" for en_US, "14:30:45" for de_DE).

```cobol
MOVE FUNCTION LOCALE-TIME(52245) TO WS-DISPLAY-TIME
*> 52245 = 14:30:45
*> Result depends on system locale, e.g. "2:30:45 PM"

MOVE FUNCTION LOCALE-TIME(52245 "en_GB")
    TO WS-DISPLAY-TIME
*> Result: "14:30:45" (British locale, 24-hour)

MOVE FUNCTION LOCALE-TIME(0) TO WS-DISPLAY-TIME
*> Result: "12:00:00 AM" or "00:00:00" depending on locale
```

### Practical Example

```cobol
       WORKING-STORAGE SECTION.
       01  WS-SECS          PIC 9(5).
       01  WS-DATE-INT      PIC 9(8).
       01  WS-FMT-DATE      PIC X(30).
       01  WS-FMT-TIME      PIC X(30).

       PROCEDURE DIVISION.
           MOVE FUNCTION CURRENT-DATE(1:8) TO WS-DATE-INT
           COMPUTE WS-SECS = FUNCTION SECONDS-PAST-MIDNIGHT
           MOVE FUNCTION LOCALE-DATE(WS-DATE-INT)
               TO WS-FMT-DATE
           MOVE FUNCTION LOCALE-TIME(WS-SECS)
               TO WS-FMT-TIME
           DISPLAY "Date: " WS-FMT-DATE
           DISPLAY "Time: " WS-FMT-TIME
           STOP RUN.
```

---

## LOCALE-TIME-FROM-SECONDS

Returns a locale-formatted time from a number of seconds past midnight.

!!! note "COBOL 2002"
    The LOCALE-TIME-FROM-SECONDS function was introduced in COBOL 2002.

```cobol
FUNCTION LOCALE-TIME-FROM-SECONDS ( argument-1
    [ argument-2 ] )
```

- **argument-1** -- numeric. The number of seconds past midnight (0 through 86399).
- **argument-2** -- optional, alphanumeric. The locale identifier. If omitted, the system default locale is used.
- **Return category** -- alphanumeric.
- **Behavior** -- similar to LOCALE-TIME but takes the number of seconds past midnight directly as its argument rather than a formatted time value. Returns a string containing the time formatted according to the conventions of the specified locale.

```cobol
MOVE FUNCTION LOCALE-TIME-FROM-SECONDS(52245)
    TO WS-DISPLAY-TIME
*> 52245 = 14:30:45
*> Result depends on locale, e.g. "2:30:45 PM"

MOVE FUNCTION LOCALE-TIME-FROM-SECONDS(
    52245 "de_DE") TO WS-DISPLAY-TIME
*> Result: "14:30:45" (German locale)

COMPUTE WS-SECS = FUNCTION SECONDS-PAST-MIDNIGHT
MOVE FUNCTION LOCALE-TIME-FROM-SECONDS(WS-SECS)
    TO WS-DISPLAY-TIME
```

---

!!! info "COBOL 2014 Formatted Date/Time Family"
    COBOL 2014 introduced a major addition to the date/time intrinsic functions: the **formatted date/time family**. This family -- comprising FORMATTED-CURRENT-DATE, FORMATTED-DATE, FORMATTED-DATETIME, FORMATTED-TIME, INTEGER-OF-FORMATTED-DATE, and TEST-FORMATTED-DATETIME -- provides a unified, format-string-driven approach to date and time formatting, parsing, and validation. These functions support ISO 8601 formats and replace many ad hoc formatting techniques that were previously necessary in COBOL programs.

---

## Examples

### Calculating Days Between Dates

```cobol
       WORKING-STORAGE SECTION.
       01  WS-START-DATE  PIC 9(8) VALUE 20250101.
       01  WS-END-DATE    PIC 9(8) VALUE 20251231.
       01  WS-DAYS        PIC 9(5).

       PROCEDURE DIVISION.
           COMPUTE WS-DAYS =
               FUNCTION INTEGER-OF-DATE(WS-END-DATE)
             - FUNCTION INTEGER-OF-DATE(WS-START-DATE)
           DISPLAY "Days between: " WS-DAYS
           *> Days between: 00364
           STOP RUN.
```

### Adding Days to a Date

```cobol
       WORKING-STORAGE SECTION.
       01  WS-BASE-DATE   PIC 9(8) VALUE 20250315.
       01  WS-DAYS-TO-ADD PIC 9(3) VALUE 90.
       01  WS-NEW-DATE    PIC 9(8).

       PROCEDURE DIVISION.
           COMPUTE WS-NEW-DATE =
               FUNCTION DATE-OF-INTEGER(
                   FUNCTION INTEGER-OF-DATE(WS-BASE-DATE)
                   + WS-DAYS-TO-ADD)
           DISPLAY "New date: " WS-NEW-DATE
           *> New date: 20250613
           STOP RUN.
```

### Extracting Components from CURRENT-DATE

```cobol
       WORKING-STORAGE SECTION.
       01  WS-DATETIME    PIC X(21).
       01  WS-DISPLAY-DATE PIC X(10).
       01  WS-DISPLAY-TIME PIC X(8).

       PROCEDURE DIVISION.
           MOVE FUNCTION CURRENT-DATE TO WS-DATETIME
           STRING WS-DATETIME(1:4) "/"
                  WS-DATETIME(5:2) "/"
                  WS-DATETIME(7:2)
               DELIMITED BY SIZE INTO WS-DISPLAY-DATE
           STRING WS-DATETIME(9:2) ":"
                  WS-DATETIME(11:2) ":"
                  WS-DATETIME(13:2)
               DELIMITED BY SIZE INTO WS-DISPLAY-TIME
           DISPLAY "Date: " WS-DISPLAY-DATE
           DISPLAY "Time: " WS-DISPLAY-TIME
           STOP RUN.
```

### Converting Two-Digit Year Dates

```cobol
       WORKING-STORAGE SECTION.
       01  WS-OLD-DATE    PIC 9(6) VALUE 991231.
       01  WS-NEW-DATE    PIC 9(8).

       PROCEDURE DIVISION.
           COMPUTE WS-NEW-DATE =
               FUNCTION DATE-TO-YYYYMMDD(WS-OLD-DATE)
           DISPLAY "Converted: " WS-NEW-DATE
           *> Converted: 19991231
           STOP RUN.
```

---

## See Also

- [Intrinsic Functions](index.md) -- overview of all intrinsic functions
- [Numeric Functions](numeric.md) -- mathematical intrinsic functions
- [String Functions](string.md) -- string manipulation intrinsic functions
- [Financial Functions](financial.md) -- financial calculation intrinsic functions
- [PICTURE](../../data-division/picture.md) -- data item format specification
- [USAGE](../../data-division/usage.md) -- internal representation and storage format
