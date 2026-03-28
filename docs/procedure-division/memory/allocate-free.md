# ALLOCATE, FREE, and Pointer Types

The `ALLOCATE` and `FREE` statements provide dynamic memory management in COBOL, allowing programs to allocate and release heap storage at runtime. These statements work with **pointer** data items and the `BASED` clause to create dynamically sized data structures.

- **Standard:** COBOL 2002, COBOL 2014

!!! note "Compiler Support"
    ALLOCATE and FREE require a COBOL 2002-compliant compiler. IBM Enterprise
    COBOL 6+, Micro Focus Visual COBOL, and GnuCOBOL 3.x support these
    statements with varying degrees of conformance.

---

## Pointer Data Items

Before using dynamic memory, you need to understand pointer types. A pointer is a data item that holds the address of another data item in memory.

### Declaring Pointers

A pointer is declared with `USAGE POINTER`:

```cobol
       WORKING-STORAGE SECTION.
       01  WS-PTR           USAGE POINTER.
       01  WS-PTR-2         USAGE POINTER VALUE NULL.
```

- A pointer occupies 4 bytes on 32-bit systems and 8 bytes on 64-bit systems.
- The initial value of a pointer is undefined unless explicitly set. Use `VALUE NULL` to initialize.
- `NULL` is a figurative constant representing a pointer that does not reference any data item.

### Pointer Operations

Pointers support a limited set of operations:

| Operation | Syntax | Description |
|-----------|--------|-------------|
| Set to address | `SET ptr TO ADDRESS OF item` | Points to an existing data item |
| Set to null | `SET ptr TO NULL` | Clears the pointer |
| Set from pointer | `SET ptr-1 TO ptr-2` | Copies a pointer value |
| Set address | `SET ADDRESS OF based-item TO ptr` | Associates a based item with a pointer |
| Compare | `IF ptr = NULL` | Tests pointer equality |
| Compare | `IF ptr-1 = ptr-2` | Tests if two pointers are equal |

```cobol
       WORKING-STORAGE SECTION.
       01  WS-PTR           USAGE POINTER.
       01  WS-RECORD.
           05  WS-NAME      PIC X(20).
           05  WS-AMOUNT    PIC 9(7)V99.

       PROCEDURE DIVISION.
           SET WS-PTR TO ADDRESS OF WS-RECORD
           IF WS-PTR NOT = NULL
               DISPLAY "Pointer is set"
           END-IF
           SET WS-PTR TO NULL
           STOP RUN.
```

### The BASED Clause

The `BASED` clause declares a data item whose storage is not allocated in WORKING-STORAGE but is instead associated with a pointer at runtime. A based item serves as a template that describes the layout of dynamically allocated memory.

```cobol
       WORKING-STORAGE SECTION.
       01  WS-PTR           USAGE POINTER.

       LINKAGE SECTION.
       01  LS-RECORD BASED.
           05  LS-NAME      PIC X(20).
           05  LS-AMOUNT    PIC 9(7)V99.
```

!!! note "Placement"
    Based items are typically declared in the `LINKAGE SECTION` because they
    describe memory that is not part of the program's static storage. Some
    compilers also allow based items in WORKING-STORAGE.

To use a based item, associate it with a pointer:

```cobol
       SET ADDRESS OF LS-RECORD TO WS-PTR
       MOVE "John Smith" TO LS-NAME
       MOVE 1500.00 TO LS-AMOUNT
```

---

## ALLOCATE

The `ALLOCATE` statement obtains dynamic storage from the heap.

### Syntax

```cobol
ALLOCATE { data-name | expression CHARACTERS }
    [ RETURNING pointer-name ]
    [ INITIALIZED ]
```

There are two forms:

**Form 1 -- Allocate by data item:**

```cobol
ALLOCATE data-name [ RETURNING pointer-name ] [ INITIALIZED ]
```

Allocates storage large enough to hold `data-name` and implicitly sets `ADDRESS OF data-name` to the allocated storage. If `RETURNING` is specified, the pointer is also stored in `pointer-name`.

**Form 2 -- Allocate by size:**

```cobol
ALLOCATE expression CHARACTERS
    RETURNING pointer-name
    [ INITIALIZED ]
```

Allocates the specified number of bytes. The `RETURNING` phrase is required because there is no data item to associate with the storage.

### Rules

- If allocation fails (e.g., insufficient memory), the pointer is set to `NULL`.
- `INITIALIZED` fills the allocated storage with the default initial values: spaces for alphanumeric items, zeros for numeric items.
- Without `INITIALIZED`, the content of allocated storage is undefined.
- Allocated storage persists until explicitly released with `FREE` or the program terminates.

### Examples

#### Allocating a Based Data Item

```cobol
       WORKING-STORAGE SECTION.
       01  WS-PTR           USAGE POINTER.

       LINKAGE SECTION.
       01  LS-CUSTOMER BASED.
           05  CUST-ID      PIC X(10).
           05  CUST-NAME    PIC X(30).
           05  CUST-BAL     PIC S9(9)V99.

       PROCEDURE DIVISION.
           ALLOCATE LS-CUSTOMER RETURNING WS-PTR
               INITIALIZED
           IF WS-PTR = NULL
               DISPLAY "Allocation failed"
               STOP RUN
           END-IF

           MOVE "C-10042"    TO CUST-ID
           MOVE "Jane Smith" TO CUST-NAME
           MOVE 8500.00      TO CUST-BAL

           DISPLAY CUST-ID " " CUST-NAME " " CUST-BAL

           FREE LS-CUSTOMER
           STOP RUN.
```

#### Allocating by Size

```cobol
       WORKING-STORAGE SECTION.
       01  WS-PTR           USAGE POINTER.
       01  WS-BUFFER-SIZE   PIC 9(5) VALUE 4096.

       LINKAGE SECTION.
       01  LS-BUFFER        PIC X(4096) BASED.

       PROCEDURE DIVISION.
           ALLOCATE WS-BUFFER-SIZE CHARACTERS
               RETURNING WS-PTR
               INITIALIZED
           IF WS-PTR = NULL
               DISPLAY "Allocation failed"
               STOP RUN
           END-IF

           SET ADDRESS OF LS-BUFFER TO WS-PTR
           MOVE "Header data" TO LS-BUFFER

           FREE LS-BUFFER
           STOP RUN.
```

---

## FREE

The `FREE` statement releases dynamically allocated storage back to the system.

### Syntax

```cobol
FREE { data-name | pointer-name } ...
```

### Rules

- After `FREE`, the pointer is set to `NULL`.
- Freeing a `NULL` pointer has no effect (it is not an error).
- Freeing storage that was not obtained by `ALLOCATE` results in undefined behavior.
- After freeing, any reference to the freed based item is undefined behavior.
- Multiple items can be freed in a single statement.

### Examples

```cobol
      *> Free a based data item
       FREE LS-CUSTOMER

      *> Free via pointer
       FREE WS-PTR

      *> Free multiple items
       FREE LS-RECORD-1 LS-RECORD-2 LS-RECORD-3
```

---

## Dynamic Data Structures

ALLOCATE and pointers enable dynamic data structures that are not possible with static COBOL tables.

### Dynamic Array (Growable Table)

```cobol
       WORKING-STORAGE SECTION.
       01  WS-MAX-ENTRIES   PIC 9(5) VALUE 0.
       01  WS-COUNT         PIC 9(5) VALUE 0.
       01  WS-IDX           PIC 9(5).
       01  WS-ENTRY-SIZE    PIC 9(5).
       01  WS-PTR           USAGE POINTER VALUE NULL.
       01  WS-NEW-PTR       USAGE POINTER VALUE NULL.

       LINKAGE SECTION.
       01  LS-ENTRY BASED.
           05  ENTRY-KEY    PIC X(10).
           05  ENTRY-VALUE  PIC X(50).

       PROCEDURE DIVISION.
       MAIN-PARA.
      *>   Allocate initial entry
           ALLOCATE LS-ENTRY RETURNING WS-PTR
               INITIALIZED
           IF WS-PTR = NULL
               DISPLAY "Allocation failed"
               STOP RUN
           END-IF

           MOVE "KEY-001" TO ENTRY-KEY
           MOVE "First entry" TO ENTRY-VALUE
           DISPLAY ENTRY-KEY " = " ENTRY-VALUE

      *>   Clean up
           FREE LS-ENTRY
           STOP RUN.
```

### Linked List

Pointers enable linked data structures where each node points to the next:

```cobol
       WORKING-STORAGE SECTION.
       01  WS-HEAD          USAGE POINTER VALUE NULL.
       01  WS-CURRENT       USAGE POINTER VALUE NULL.
       01  WS-PREV          USAGE POINTER VALUE NULL.

       LINKAGE SECTION.
       01  LS-NODE BASED.
           05  NODE-DATA    PIC X(50).
           05  NODE-NEXT    USAGE POINTER.

       PROCEDURE DIVISION.
       MAIN-PARA.
      *>   Create first node
           ALLOCATE LS-NODE RETURNING WS-HEAD
               INITIALIZED
           MOVE "First item" TO NODE-DATA
           SET NODE-NEXT TO NULL

      *>   Save pointer to first node
           SET WS-PREV TO WS-HEAD

      *>   Create second node
           ALLOCATE LS-NODE RETURNING WS-CURRENT
               INITIALIZED
           MOVE "Second item" TO NODE-DATA
           SET NODE-NEXT TO NULL

      *>   Link first node to second
           SET ADDRESS OF LS-NODE TO WS-PREV
           SET NODE-NEXT TO WS-CURRENT

      *>   Traverse and display the list
           SET WS-CURRENT TO WS-HEAD
           PERFORM UNTIL WS-CURRENT = NULL
               SET ADDRESS OF LS-NODE TO WS-CURRENT
               DISPLAY NODE-DATA
               SET WS-CURRENT TO NODE-NEXT
           END-PERFORM

      *>   Free all nodes
           SET WS-CURRENT TO WS-HEAD
           PERFORM UNTIL WS-CURRENT = NULL
               SET ADDRESS OF LS-NODE TO WS-CURRENT
               SET WS-PREV TO NODE-NEXT
               FREE LS-NODE
               SET WS-CURRENT TO WS-PREV
           END-PERFORM
           STOP RUN.
```

---

## Pointer Arithmetic

Standard COBOL does not define pointer arithmetic. However, some compilers provide extensions for it.

### IBM Enterprise COBOL

IBM allows pointer arithmetic using the `SET` statement:

```cobol
       SET WS-PTR UP BY 100
       SET WS-PTR DOWN BY 50
```

### Micro Focus / GnuCOBOL

Some compilers allow pointer arithmetic in `COMPUTE` or through vendor-specific extensions:

```cobol
      *> GnuCOBOL extension
       ADD 100 TO WS-PTR
```

!!! warning "Portability"
    Pointer arithmetic is not standard COBOL. Code that uses it is
    compiler-specific and may not port between implementations.

---

## Common Patterns

### Allocate, Use, Free

The fundamental pattern for dynamic memory:

```cobol
      *>   1. Allocate
           ALLOCATE LS-WORK-AREA RETURNING WS-PTR
               INITIALIZED
           IF WS-PTR = NULL
               PERFORM ALLOCATION-ERROR
               GOBACK
           END-IF

      *>   2. Use
           PERFORM PROCESS-DATA

      *>   3. Free
           FREE LS-WORK-AREA
```

### Null-Check Before Use

Always verify allocation succeeded before accessing the storage:

```cobol
           ALLOCATE LS-RECORD RETURNING WS-PTR
           IF WS-PTR = NULL
               DISPLAY "Out of memory"
               MOVE 12 TO RETURN-CODE
               GOBACK
           END-IF
           SET ADDRESS OF LS-RECORD TO WS-PTR
```

### Safe Free

Freeing sets the pointer to NULL, so repeated FREE calls are safe:

```cobol
      *>   This is safe -- freeing NULL is a no-op
           FREE WS-PTR
           FREE WS-PTR
```

---

## ALLOCATE vs Static Tables

| Aspect | Static (OCCURS) | Dynamic (ALLOCATE) |
|--------|----------------|-------------------|
| Size | Fixed at compile time | Determined at runtime |
| Storage | WORKING-STORAGE | Heap |
| Performance | Faster access | Allocation overhead |
| Memory | Always allocated | Allocated on demand |
| Cleanup | Automatic | Must call FREE |
| Standard | COBOL-68 | COBOL 2002 |
| Use case | Known, fixed-size collections | Variable-size or large data |

For most COBOL programs, static tables with `OCCURS` are sufficient and preferred. Use ALLOCATE when:

- The data size is not known until runtime
- The data is too large to allocate statically
- You need linked structures (lists, trees)
- Memory should be allocated only when needed

---

## See Also

- [OCCURS Clause](../../data-division/occurs.md) -- static table definitions
- [REDEFINES Clause](../../data-division/redefines.md) -- storage overlay
- [USAGE Clause](../../data-division/usage.md) -- data storage formats
- [Program Linkage (CALL)](../program-linkage/call.md) -- passing pointers between programs
