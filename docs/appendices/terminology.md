# Terminology Mapping

This page maps common programming concepts and terminology from other languages to their COBOL equivalents. Use your browser's search (Ctrl+F) or the site search to find COBOL features by the name you know from other languages.

---

## Data Types and Declarations

| Concept | COBOL | C / C++ | Java | Python | Rust |
|---------|-------|---------|------|--------|------|
| Integer | `PIC 9(n)` | `int` | `int` | `int` | `i32` |
| Decimal / fixed-point | `PIC 9(n)V9(m)` | — | `BigDecimal` | `Decimal` | — |
| Float | `COMP-1` / `FLOAT-SHORT` | `float` | `float` | `float` | `f32` |
| Double | `COMP-2` / `FLOAT-LONG` | `double` | `double` | `float` | `f64` |
| String (fixed-length) | `PIC X(n)` | `char[n]` | `String` | `str` | `String` |
| Boolean | Level-88 condition name | `bool` | `boolean` | `bool` | `bool` |
| Enum | Level-88 condition names | `enum` | `enum` | `Enum` | `enum` |
| Struct / record | Group item (01-49) | `struct` | `class` | `dataclass` | `struct` |
| Union | `REDEFINES` | `union` | — | — | `enum` (tagged) |
| Type alias | `TYPEDEF` | `typedef` | — | `TypeAlias` | `type` |
| Constant | `VALUE` clause / level-78 | `const` | `final` | `CONST` | `const` |
| Pointer | `USAGE POINTER` | `*` pointer | reference | — | `*const` / `*mut` |
| Null | `NULL` | `NULL` | `null` | `None` | `None` |
| Void / nothing | `FILLER` | `void` | `void` | `None` | `()` |

**See:** [PICTURE](../data-division/picture.md), [USAGE](../data-division/usage.md), [Level Numbers](../data-division/level-numbers.md), [Condition Names](../data-division/condition-names.md), [REDEFINES](../data-division/redefines.md), [TYPEDEF](../data-division/typedef.md)

---

## Variables and Scope

| Concept | COBOL | C / C++ | Java | Python |
|---------|-------|---------|------|--------|
| Global variable | WORKING-STORAGE | Global / `static` | `static` field | Module-level |
| Local variable | LOCAL-STORAGE | Local variable | Local variable | Local variable |
| Static variable (persists) | WORKING-STORAGE (in subprogram) | `static` local | — | — |
| Parameter | LINKAGE SECTION | Function parameter | Method parameter | Function parameter |
| Shared across modules | `EXTERNAL` clause | `extern` | `public static` | Module import |
| Visible to nested scope | `GLOBAL` clause | — | Inner class access | Closure |

**See:** [Storage Sections](../data-division/storage-sections.md)

---

## Data Structures

| Concept | COBOL | C / C++ | Java | Python |
|---------|-------|---------|------|--------|
| Array / vector | `OCCURS` clause | `array[]` | `ArrayList` | `list` |
| Fixed-size array | `OCCURS n TIMES` | `int[n]` | `int[]` | `tuple` |
| Variable-length array | `OCCURS DEPENDING ON` | `malloc` | `ArrayList` | `list` |
| Multidimensional array | Nested `OCCURS` | `int[][]` | `int[][]` | Nested lists |
| Table / dictionary | `OCCURS` with `SEARCH` | Hash table | `HashMap` | `dict` |
| Linked list | `POINTER` + `ALLOCATE` | Linked list | `LinkedList` | — |
| String | `PIC X(n)` | `char[]` | `String` | `str` |
| Substring | Reference modification `(start:len)` | `substr` | `substring` | Slicing `[s:e]` |

**See:** [OCCURS](../data-division/occurs.md), [Reference Modification](../language/reference-modification.md), [SEARCH](../procedure-division/data-movement/search.md), [ALLOCATE/FREE](../procedure-division/memory/allocate-free.md)

---

## Control Flow

| Concept | COBOL | C / C++ | Java | Python | Rust |
|---------|-------|---------|------|--------|------|
| If / else | `IF ... ELSE ... END-IF` | `if/else` | `if/else` | `if/elif/else` | `if/else` |
| Switch / case | `EVALUATE` | `switch` | `switch` | `match` (3.10+) | `match` |
| For loop | `PERFORM VARYING` | `for` | `for` | `for` | `for` |
| While loop | `PERFORM UNTIL` | `while` | `while` | `while` | `while` |
| Do-while | `PERFORM WITH TEST AFTER` | `do...while` | `do...while` | — | `loop` |
| Infinite loop | `PERFORM UNTIL EXIT` | `while(1)` | `while(true)` | `while True` | `loop` |
| Break | `EXIT PERFORM` | `break` | `break` | `break` | `break` |
| Continue | `EXIT PERFORM CYCLE` | `continue` | `continue` | `continue` | `continue` |
| Goto | `GO TO` | `goto` | — | — | — |
| Return | `GOBACK` | `return` | `return` | `return` | `return` |
| Exit program | `STOP RUN` | `exit()` | `System.exit()` | `sys.exit()` | `std::process::exit()` |
| No-op | `CONTINUE` | `;` | `;` | `pass` | — |

**See:** [IF](../procedure-division/control-flow/if.md), [EVALUATE](../procedure-division/control-flow/evaluate.md), [PERFORM](../procedure-division/control-flow/perform.md), [GOBACK](../procedure-division/control-flow/goback.md), [EXIT](../procedure-division/control-flow/exit.md), [STOP](../procedure-division/control-flow/stop.md)

---

## Functions and Subroutines

| Concept | COBOL | C / C++ | Java | Python |
|---------|-------|---------|------|--------|
| Function call (internal) | `PERFORM paragraph` | Function call | Method call | Function call |
| Function call (external) | `CALL "program"` | `dlsym` / link | Method call | `import` + call |
| Function with return | User-defined function (`FUNCTION-ID`) | `int func()` | `int method()` | `def func():` |
| Subroutine (no return) | `CALL` subprogram | `void func()` | `void method()` | `def func():` |
| Inline procedure | Inline `PERFORM ... END-PERFORM` | Lambda / block | Lambda | Lambda |
| Callback | `PROGRAM-POINTER` | Function pointer | Interface | Callable |
| Entry point | `ENTRY` statement | — | — | — |

**See:** [CALL](../procedure-division/program-linkage/call.md), [PERFORM](../procedure-division/control-flow/perform.md), [User-Defined Functions](../intrinsic-functions/user-defined.md), [ENTRY](../procedure-division/program-linkage/entry.md)

---

## I/O and File Handling

| Concept | COBOL | C / C++ | Java | Python |
|---------|-------|---------|------|--------|
| Open file | `OPEN` | `fopen` | `new FileInputStream` | `open()` |
| Read record | `READ` | `fread` / `fgets` | `read()` | `readline()` |
| Write record | `WRITE` | `fwrite` / `fprintf` | `write()` | `write()` |
| Close file | `CLOSE` | `fclose` | `close()` | `close()` |
| Update record | `REWRITE` | `fseek` + `fwrite` | RandomAccessFile | — |
| Delete record | `DELETE` | — | — | — |
| Seek / position | `START` | `fseek` | `seek()` | `seek()` |
| Console output | `DISPLAY` | `printf` | `System.out.println` | `print()` |
| Console input | `ACCEPT` | `scanf` | `Scanner` | `input()` |
| File status | `FILE STATUS` | `errno` | IOException | Exception |

**See:** [OPEN](../procedure-division/io/open.md), [READ](../procedure-division/io/read.md), [WRITE](../procedure-division/io/write.md), [File Status Codes](file-status-codes.md)

---

## Error Handling

| Concept | COBOL | C / C++ | Java | Python |
|---------|-------|---------|------|--------|
| Try / catch | `USE AFTER ERROR` (declaratives) | `try/catch` (C++) | `try/catch` | `try/except` |
| Overflow check | `ON SIZE ERROR` | — | ArithmeticException | OverflowError |
| End of file | `AT END` | `feof` | `EOFException` | `StopIteration` |
| Key not found | `INVALID KEY` | — | `NoSuchElementException` | `KeyError` |
| File error codes | `FILE STATUS` | `errno` | IOException | OSError |
| Exception object | `EXCEPTION-STATUS` function | `exception` | Exception object | Exception object |

**See:** [DECLARATIVES](../procedure-division/control-flow/declaratives.md), [File Status Codes](file-status-codes.md), [Exception Functions](../intrinsic-functions/exception-handling.md)

---

## String Operations

| Concept | COBOL | C / C++ | Java | Python |
|---------|-------|---------|------|--------|
| Concatenation | `STRING` verb | `strcat` | `+` / `concat` | `+` / `join` |
| Split | `UNSTRING` verb | `strtok` | `split()` | `split()` |
| Find / replace | `INSPECT` | `strstr` / `sed` | `replace()` | `replace()` |
| Substring | Reference modification `(s:l)` | `substr` | `substring()` | `[s:e]` slicing |
| Uppercase | `FUNCTION UPPER-CASE` | `toupper` | `toUpperCase()` | `upper()` |
| Lowercase | `FUNCTION LOWER-CASE` | `tolower` | `toLowerCase()` | `lower()` |
| Trim | `FUNCTION TRIM` | — | `trim()` | `strip()` |
| Length | `FUNCTION LENGTH` | `strlen` | `length()` | `len()` |
| Reverse | `FUNCTION REVERSE` | — | `reverse()` | `[::-1]` |

**See:** [STRING](../procedure-division/string-handling/string.md), [UNSTRING](../procedure-division/string-handling/unstring.md), [INSPECT](../procedure-division/string-handling/inspect.md), [String Functions](../intrinsic-functions/string.md)

---

## Sorting and Searching

| Concept | COBOL | C / C++ | Java | Python |
|---------|-------|---------|------|--------|
| Sort | `SORT` verb | `qsort` | `Collections.sort` | `sorted()` |
| Merge | `MERGE` verb | — | — | `heapq.merge` |
| Linear search | `SEARCH` | Linear scan | `indexOf` | `in` operator |
| Binary search | `SEARCH ALL` | `bsearch` | `binarySearch` | `bisect` |

**See:** [SORT](../procedure-division/sort-merge/sort.md), [SEARCH](../procedure-division/data-movement/search.md)

---

## Memory Management

| Concept | COBOL | C / C++ | Java | Python |
|---------|-------|---------|------|--------|
| Stack allocation | LOCAL-STORAGE | Local variables | Local variables | Local variables |
| Heap allocation | `ALLOCATE` | `malloc` / `new` | `new` | Object creation |
| Free memory | `FREE` | `free` / `delete` | Garbage collected | Garbage collected |
| Pointer | `USAGE POINTER` | `*` pointer | Reference | — |
| Address-of | `ADDRESS OF` | `&` operator | — | `id()` |
| Null check | `IF ptr = NULL` | `if (ptr == NULL)` | `if (obj == null)` | `if obj is None` |

**See:** [ALLOCATE and FREE](../procedure-division/memory/allocate-free.md), [Storage Sections](../data-division/storage-sections.md)

---

## Program Structure

| Concept | COBOL | C / C++ | Java | Python |
|---------|-------|---------|------|--------|
| Source file | COBOL program | `.c` / `.cpp` file | `.java` file | `.py` file |
| Include / import | `COPY` statement | `#include` | `import` | `import` |
| Macro / template | `COPY ... REPLACING` | `#define` | Generics | — |
| Namespace | Division / Section | `namespace` | `package` | Module |
| Main function | `PROCEDURE DIVISION` | `main()` | `public static void main` | `if __name__` |
| Comment | `*` (col 7) or `*>` | `//` or `/* */` | `//` or `/* */` | `#` |

**See:** [Source Format](../language/source-format.md), [COPY](../copy-replace/copy.md)
