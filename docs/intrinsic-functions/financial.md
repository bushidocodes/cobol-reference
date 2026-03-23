# Financial Functions

Intrinsic functions that perform financial calculations including annuity ratios and present value computations.

**Standard:** COBOL-85 Amendment 1 (1989), COBOL 2002, COBOL 2014, COBOL 2023

---

## General Syntax

```cobol
FUNCTION function-name ( argument-1  argument-2 [ argument-3 ] ... )
```

Financial intrinsic functions return values of category **numeric**. The results are suitable for use in `COMPUTE` statements and arithmetic expressions. The receiving data item should have a [PICTURE](../../data-division/picture.md) and [USAGE](../../data-division/usage.md) appropriate for the precision required by the financial calculation.

---

## ANNUITY

Returns the ratio used to calculate a series of equal periodic payments (an annuity) that would repay a loan of one unit at a given interest rate over a given number of periods.

```cobol
FUNCTION ANNUITY ( argument-1  argument-2 )
```

- **argument-1** -- numeric. The periodic interest rate. Must be zero or positive. A value of 0 indicates a zero-interest loan.
- **argument-2** -- integer, positive. The number of periods.
- **Return category** -- numeric.

### Behavior

When argument-1 (interest rate) is greater than zero, the function returns:

```
argument-1 / (1 - (1 + argument-1) ** (- argument-2))
```

When argument-1 is zero, the function returns:

```
1 / argument-2
```

The returned value is the payment per period necessary to repay a loan of one monetary unit. To calculate the actual periodic payment for a given loan amount, multiply the result by the loan principal.

The ANNUITY function computes the **capital recovery factor** -- the reciprocal of the present value of an ordinary annuity of one unit per period.

### Examples

#### Basic Annuity Calculation

```cobol
       WORKING-STORAGE SECTION.
       01  WS-PRINCIPAL    PIC 9(7)V99 VALUE 100000.00.
       01  WS-ANNUAL-RATE  PIC 9V9(4)  VALUE 0.0600.
       01  WS-MONTHLY-RATE PIC 9V9(8).
       01  WS-PERIODS      PIC 9(3)    VALUE 360.
       01  WS-PAYMENT      PIC 9(5)V99.
       01  WS-ANNUITY      PIC 9V9(8).

       PROCEDURE DIVISION.
      *>   Calculate monthly payment on a 30-year mortgage
      *>   at 6% annual interest
           COMPUTE WS-MONTHLY-RATE =
               WS-ANNUAL-RATE / 12
           COMPUTE WS-ANNUITY =
               FUNCTION ANNUITY(WS-MONTHLY-RATE WS-PERIODS)
           COMPUTE WS-PAYMENT =
               WS-PRINCIPAL * WS-ANNUITY
           DISPLAY "Monthly payment: " WS-PAYMENT
           *> Monthly payment: 00599.55
           STOP RUN.
```

#### Zero Interest Annuity

```cobol
       WORKING-STORAGE SECTION.
       01  WS-PRINCIPAL    PIC 9(7)V99 VALUE 12000.00.
       01  WS-PERIODS      PIC 9(3)    VALUE 12.
       01  WS-PAYMENT      PIC 9(5)V99.

       PROCEDURE DIVISION.
      *>   0% interest: simply divides evenly
           COMPUTE WS-PAYMENT =
               WS-PRINCIPAL * FUNCTION ANNUITY(0 WS-PERIODS)
           DISPLAY "Monthly payment: " WS-PAYMENT
           *> Monthly payment: 01000.00
           STOP RUN.
```

#### Comparing Loan Terms

```cobol
       WORKING-STORAGE SECTION.
       01  WS-PRINCIPAL    PIC 9(7)V99  VALUE 250000.00.
       01  WS-RATE         PIC 9V9(6).
       01  WS-PMT-15       PIC 9(5)V99.
       01  WS-PMT-30       PIC 9(5)V99.
       01  WS-TOTAL-15     PIC 9(7)V99.
       01  WS-TOTAL-30     PIC 9(7)V99.

       PROCEDURE DIVISION.
           COMPUTE WS-RATE = 0.055 / 12

      *>   15-year mortgage (180 months)
           COMPUTE WS-PMT-15 =
               WS-PRINCIPAL * FUNCTION ANNUITY(WS-RATE 180)
      *>   30-year mortgage (360 months)
           COMPUTE WS-PMT-30 =
               WS-PRINCIPAL * FUNCTION ANNUITY(WS-RATE 360)

           COMPUTE WS-TOTAL-15 = WS-PMT-15 * 180
           COMPUTE WS-TOTAL-30 = WS-PMT-30 * 360

           DISPLAY "15-year payment: " WS-PMT-15
           DISPLAY "15-year total:   " WS-TOTAL-15
           DISPLAY "30-year payment: " WS-PMT-30
           DISPLAY "30-year total:   " WS-TOTAL-30
           *> 15-year payment: 02042.71
           *> 15-year total:   0367687.80
           *> 30-year payment: 01419.47
           *> 30-year total:   0511009.20
           STOP RUN.
```

---

## PRESENT-VALUE

Returns the present value (net present value) of a series of future amounts at a given discount rate.

```cobol
FUNCTION PRESENT-VALUE ( argument-1  argument-2 ... )
```

- **argument-1** -- numeric. The discount rate per period. Must be greater than -1.
- **argument-2 ...** -- one or more numeric arguments. Each represents a future amount to be received at the end of the corresponding period.
- **Return category** -- numeric.

### Behavior

The function returns the sum of the discounted values of each future amount:

```
result = argument-2 / (1 + argument-1) ** 1
       + argument-3 / (1 + argument-1) ** 2
       + argument-4 / (1 + argument-1) ** 3
       + ...
```

In general terms:

```
result = SUM( amount-i / (1 + rate) ** i )   for i = 1 to n
```

where amount-i is the i-th amount argument and n is the number of amount arguments. This calculates the net present value (NPV) of a series of cash flows discounted at rate argument-1.

If all amount arguments are equal, the result is equivalent to `amount * (1 / FUNCTION ANNUITY(argument-1, n))` when argument-1 is greater than zero.

### Examples

#### Present Value of Equal Cash Flows

```cobol
       WORKING-STORAGE SECTION.
       01  WS-RATE         PIC 9V9(4)  VALUE 0.0800.
       01  WS-PV           PIC 9(7)V99.

       PROCEDURE DIVISION.
      *>   Present value of receiving $1000 per year
      *>   for 5 years at 8% discount rate
           COMPUTE WS-PV = FUNCTION PRESENT-VALUE(
               WS-RATE
               1000.00  1000.00  1000.00
               1000.00  1000.00)
           DISPLAY "Present value: " WS-PV
           *> Present value: 0003992.71
           *>   Year 1: 1000 / 1.08     =  925.93
           *>   Year 2: 1000 / 1.08^2   =  857.34
           *>   Year 3: 1000 / 1.08^3   =  793.83
           *>   Year 4: 1000 / 1.08^4   =  735.03
           *>   Year 5: 1000 / 1.08^5   =  680.58
           *>   Total                    = 3992.71
           STOP RUN.
```

#### Present Value of Varying Cash Flows

```cobol
       WORKING-STORAGE SECTION.
       01  WS-RATE         PIC 9V9(4) VALUE 0.1000.
       01  WS-PV           PIC 9(7)V99.

       PROCEDURE DIVISION.
      *>   NPV of an investment with varying annual returns
      *>   at 10% discount rate
           COMPUTE WS-PV = FUNCTION PRESENT-VALUE(
               WS-RATE
               5000.00  8000.00  12000.00
               10000.00  6000.00)
           DISPLAY "Net present value: " WS-PV
           *> Net present value: 0030722.84
           *>   Year 1: 5000  / 1.10     =  4545.45
           *>   Year 2: 8000  / 1.10^2   =  6611.57
           *>   Year 3: 12000 / 1.10^3   =  9015.78
           *>   Year 4: 10000 / 1.10^4   =  6830.13
           *>   Year 5: 6000  / 1.10^5   =  3725.53
           *>   Total                     = 30728.46
           STOP RUN.
```

#### Investment Decision

```cobol
       WORKING-STORAGE SECTION.
       01  WS-COST         PIC 9(7)V99 VALUE 50000.00.
       01  WS-RATE         PIC 9V9(4)  VALUE 0.1200.
       01  WS-PV           PIC 9(7)V99.
       01  WS-NPV          PIC S9(7)V99.

       PROCEDURE DIVISION.
      *>   Evaluate whether a $50,000 investment is worthwhile
      *>   given projected cash flows at 12% discount rate
           COMPUTE WS-PV = FUNCTION PRESENT-VALUE(
               WS-RATE
               15000.00  18000.00  20000.00
               12000.00  10000.00)
           COMPUTE WS-NPV = WS-PV - WS-COST
           DISPLAY "Present value of cash flows: " WS-PV
           DISPLAY "Net present value:           " WS-NPV
           IF WS-NPV > 0
               DISPLAY "Investment is profitable"
           ELSE
               DISPLAY "Investment is not profitable"
           END-IF
           *> Present value of cash flows: 0054578.19
           *> Net present value:           +0004578.19
           *> Investment is profitable
           STOP RUN.
```

#### Using PRESENT-VALUE with a Table

```cobol
       WORKING-STORAGE SECTION.
       01  WS-CASH-FLOWS.
           05  WS-CF       PIC 9(7)V99 OCCURS 4 TIMES.
       01  WS-RATE         PIC 9V9(4) VALUE 0.0500.
       01  WS-PV           PIC 9(7)V99.

       PROCEDURE DIVISION.
           MOVE 10000.00 TO WS-CF(1)
           MOVE 15000.00 TO WS-CF(2)
           MOVE 20000.00 TO WS-CF(3)
           MOVE 25000.00 TO WS-CF(4)

           COMPUTE WS-PV = FUNCTION PRESENT-VALUE(
               WS-RATE  WS-CF(ALL))
           DISPLAY "Present value: " WS-PV
           *> Present value: 0061698.86
           *>   Year 1: 10000 / 1.05     =  9523.81
           *>   Year 2: 15000 / 1.05^2   = 13605.44
           *>   Year 3: 20000 / 1.05^3   = 17276.75
           *>   Year 4: 25000 / 1.05^4   = 20567.05
           *>   Total                     = 60973.05
           STOP RUN.
```

---

## See Also

- [Intrinsic Functions](index.md) -- overview of all intrinsic functions
- [Numeric Functions](numeric.md) -- mathematical intrinsic functions
- [String Functions](string.md) -- string manipulation intrinsic functions
- [Date and Time Functions](date-time.md) -- date and time intrinsic functions
- [PICTURE](../../data-division/picture.md) -- data item format specification
- [USAGE](../../data-division/usage.md) -- internal representation and storage format
