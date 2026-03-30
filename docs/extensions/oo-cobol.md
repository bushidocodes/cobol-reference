# Object-Oriented COBOL

COBOL 2002 introduced object-oriented programming features including classes, methods, interfaces, inheritance, and object references. These features allow COBOL programs to define and use objects similarly to Java or C++.

- **Standard:** COBOL 2002, COBOL 2014, COBOL 2023
- **Category:** Language Extension (part of the standard but not universally implemented)

!!! note "Compiler Support"
    OO COBOL support varies significantly. IBM Enterprise COBOL supports a
    subset. Micro Focus Visual COBOL has the most complete OO implementation.
    GnuCOBOL has limited OO support. Many production COBOL shops do not
    use OO features.

---

## CLASS-ID

Defines a class (analogous to a Java class or C++ class).

```cobol
IDENTIFICATION DIVISION.
CLASS-ID. class-name
    [ INHERITS FROM class-name-2 [ class-name-3 ] ... ]
    .
```

A class contains a FACTORY paragraph (for class-level/static behavior) and an OBJECT paragraph (for instance behavior).

```cobol
IDENTIFICATION DIVISION.
CLASS-ID. Customer INHERITS FROM Base.

ENVIRONMENT DIVISION.
CONFIGURATION SECTION.
REPOSITORY.
    CLASS Base AS "Base"
    CLASS Customer AS "Customer".

FACTORY.
    *> Class methods and data
END FACTORY.

OBJECT.
    DATA DIVISION.
    WORKING-STORAGE SECTION.
    01  CUST-NAME      PIC X(30).
    01  CUST-BALANCE   PIC S9(9)V99.

    PROCEDURE DIVISION.

    IDENTIFICATION DIVISION.
    METHOD-ID. GET-NAME.
    DATA DIVISION.
    LINKAGE SECTION.
    01  LS-NAME        PIC X(30).
    PROCEDURE DIVISION RETURNING LS-NAME.
        MOVE CUST-NAME TO LS-NAME.
    END METHOD GET-NAME.

    IDENTIFICATION DIVISION.
    METHOD-ID. SET-NAME.
    DATA DIVISION.
    LINKAGE SECTION.
    01  LS-NAME        PIC X(30).
    PROCEDURE DIVISION USING LS-NAME.
        MOVE LS-NAME TO CUST-NAME.
    END METHOD SET-NAME.

END OBJECT.

END CLASS Customer.
```

---

## METHOD-ID

Defines a method within a class.

```cobol
IDENTIFICATION DIVISION.
METHOD-ID. method-name.

DATA DIVISION.
[ WORKING-STORAGE SECTION. ... ]
[ LINKAGE SECTION. ... ]

PROCEDURE DIVISION [ USING parameters ] [ RETURNING result ].
    [method body]
END METHOD method-name.
```

---

## INTERFACE-ID

Defines an interface that classes can implement.

```cobol
IDENTIFICATION DIVISION.
INTERFACE-ID. Printable.

PROCEDURE DIVISION.

IDENTIFICATION DIVISION.
METHOD-ID. PRINT-DETAILS.
END METHOD PRINT-DETAILS.

END INTERFACE Printable.
```

---

## INVOKE

Calls a method on an object.

```cobol
INVOKE object-reference method-name
    [ USING parameters ]
    [ RETURNING result ]
```

### Examples

```cobol
*> Create an object
INVOKE Customer "NEW" RETURNING WS-CUST-REF

*> Call instance methods
INVOKE WS-CUST-REF "SET-NAME" USING "John Smith"
INVOKE WS-CUST-REF "GET-NAME" RETURNING WS-NAME

*> Call a class (factory) method
INVOKE Customer "GET-COUNT" RETURNING WS-COUNT
```

---

## Object References

Objects are accessed through reference variables declared with USAGE OBJECT REFERENCE.

```cobol
01  WS-CUST-REF   USAGE OBJECT REFERENCE Customer.
01  WS-ANY-REF    USAGE OBJECT REFERENCE.
```

- A typed reference (`OBJECT REFERENCE Customer`) can only reference objects of that class or its subclasses.
- An untyped reference (`OBJECT REFERENCE` without a class name) can reference any object.
- `NULL` indicates no object is referenced.
- `SELF` refers to the current object within a method.
- `SUPER` invokes the parent class's implementation of a method.

---

## Inheritance

```cobol
CLASS-ID. PremiumCustomer INHERITS FROM Customer.
```

- A subclass inherits all data and methods from its parent class.
- Methods can be overridden using the `OVERRIDE` phrase on METHOD-ID.
- COBOL does not support multiple inheritance of classes but allows implementation of multiple interfaces.

---

## REPOSITORY Paragraph

Classes, interfaces, and their external names must be declared in the REPOSITORY paragraph.

```cobol
ENVIRONMENT DIVISION.
CONFIGURATION SECTION.
REPOSITORY.
    CLASS Customer AS "Customer"
    CLASS Base AS "java.lang.Object"
    INTERFACE Printable AS "Printable".
```

---

## See Also

- [Identification Division](../identification-division/index.md) -- CLASS-ID, METHOD-ID
- [CALL](../procedure-division/program-linkage/call.md) -- procedural inter-program communication
- [User-Defined Functions](../intrinsic-functions/user-defined.md) -- FUNCTION-ID
- [TYPEDEF](../data-division/typedef.md) -- user-defined types
