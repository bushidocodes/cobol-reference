# Glossary

Key COBOL terms and their definitions.

---

**Alphanumeric item** — A data item whose PICTURE contains only `X` characters. Can hold letters, digits, and special characters.

**BASED item** — A data item in the LINKAGE SECTION whose storage is provided by ALLOCATE or SET ADDRESS OF, not by a calling program.

**Compilation unit** — A single source program that is compiled as a unit. May contain nested programs.

**Condition name** — A name (level-88 entry) associated with specific values of a conditional variable. Used in IF and EVALUATE statements as boolean-like tests.

**Conditional variable** — A data item that has one or more level-88 condition names defined on it.

**Copybook** — A file containing COBOL source text intended for inclusion via the COPY statement. Analogous to header files in C.

**Data item** — A unit of data defined in the Data Division. Can be elementary (no subordinates) or group (has subordinates).

**Declaratives** — A set of USE procedures at the beginning of the Procedure Division that are invoked automatically in response to specific events (file errors, report events).

**Division** — One of the four major structural units of a COBOL program: Identification, Environment, Data, and Procedure.

**Elementary item** — A data item with no subordinate items. Has a PICTURE clause that defines its size and type.

**Entry** — A data description in the Data Division, consisting of a level number, a data name, and clauses.

**FD (File Description)** — An entry in the FILE SECTION that describes the physical characteristics of a file.

**Figurative constant** — A reserved word representing a specific value: ZERO, SPACE, HIGH-VALUE, LOW-VALUE, QUOTE, ALL literal, NULL.

**FILLER** — An unnamed data item used to reserve space in a record without providing a reference name.

**Group item** — A data item with one or more subordinate items. Has no PICTURE clause; its size is the sum of its subordinates.

**Imperative statement** — A statement that specifies an unconditional action (e.g., MOVE, ADD). Contrast with conditional statement.

**Index** — A compiler-managed data item associated with a table (OCCURS INDEXED BY) that holds a displacement value. Manipulated via SET.

**Level number** — A number (01-49, 66, 77, 88) indicating the hierarchy of data items in a record structure.

**Literal** — A constant value written directly in the source code. Numeric (e.g., `42`, `-3.14`) or nonnumeric (e.g., `"HELLO"`).

**Main program** — The first program in a run unit. Entered from the operating system. Terminated by STOP RUN or GOBACK.

**Noise word** — An optional word in a COBOL statement that improves readability but has no effect on execution (e.g., IS, THEN, THAN).

**Paragraph** — A named unit within the Procedure Division, consisting of a paragraph-name followed by sentences. The smallest unit that can be PERFORMed or referenced by GO TO.

**Procedure** — A paragraph or section in the Procedure Division. Referenced by PERFORM and GO TO.

**Qualification** — Using IN or OF to distinguish between data items with the same name in different records or groups.

**Record** — A group of related data items treated as a unit for I/O purposes. Defined at level 01 in the Data Division.

**Reference modification** — Substring access using the notation `identifier(start:length)`.

**Reserved word** — A word with special meaning in COBOL that cannot be used as a user-defined name.

**Run unit** — The set of all programs (main and called subprograms) that are loaded and executing together. Terminated by STOP RUN.

**SD (Sort Description)** — An entry in the FILE SECTION that describes a sort or merge work file.

**Scope terminator** — A keyword that explicitly ends a statement scope: END-IF, END-PERFORM, END-READ, etc. Introduced in COBOL-85.

**Section** — A named group of paragraphs in the Procedure Division. Used with SORT INPUT/OUTPUT PROCEDURE and DECLARATIVES.

**Sentence** — One or more statements terminated by a period. In structured code, sentences typically contain one statement.

**Source program** — A COBOL program in its original source form, before compilation.

**Special register** — A compiler-generated data item (RETURN-CODE, SORT-RETURN, TALLY, LINAGE-COUNTER, etc.) available without explicit declaration.

**Statement** — The basic unit of executable code in the Procedure Division, consisting of a verb and its operands.

**Subprogram** — A program invoked by CALL from another program. Returns control via GOBACK or EXIT PROGRAM.

**Subscript** — An integer value used to reference a specific occurrence of a table item (defined with OCCURS).

---

## See Also

- [Terminology Mapping](terminology.md) -- cross-language concept mapping
- [Reserved Words](reserved-words.md) -- complete reserved word list
- [Special Registers](../language/special-registers.md)
- [Figurative Constants](../language/figurative-constants.md)
