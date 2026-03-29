# Asynchronous I/O Verbs (Removed)

The COBOL-65 and 1968 CODASYL Journal of Development defined an asynchronous I/O subsystem for overlapping computation with input/output operations on direct-access storage devices. This subsystem comprised four verbs — `HOLD`, `SEEK`, `PROCESS`, and `SUSPEND` — along with supporting Data Division constructs.

- **Introduced:** COBOL-65 / 1968 CODASYL Journal of Development
- **Removed:** Before COBOL-74
- **Status:** Removed (never included in any ANSI standard)

!!! warning "Removed"
    The entire asynchronous I/O subsystem was removed before COBOL-74. These
    verbs never appeared in any ANSI or ISO COBOL standard. Modern COBOL
    relies on the operating system and runtime for I/O optimization.

---

## HOLD

Obtained exclusive access to a record on a direct-access file, preventing other programs or processes from modifying it while the current program worked with it.

```cobol
HOLD file-name RECORD
```

- The HOLD verb locked a record for exclusive access.
- The record remained held until explicitly released or until the file was closed.
- This provided a basic record-locking mechanism for concurrent access scenarios.

---

## SEEK

Pre-positioned a direct-access storage device for a subsequent READ or WRITE, allowing the mechanical seek operation to overlap with computation.

```cobol
SEEK file-name RECORD
```

- SEEK initiated the physical positioning of the read/write head on a disk or drum.
- The program could continue executing other statements while the device positioned itself.
- The subsequent READ or WRITE would complete faster because the device was already positioned.
- This was an optimization hint — on sequential devices or in environments without overlapped I/O, SEEK had no effect.

---

## PROCESS

Initiated asynchronous, out-of-line processing using a Saved Area. This allowed a section of the program to execute independently while the main program continued.

```cobol
PROCESS section-name
```

- The named section executed asynchronously using its own Saved Area (SA) for data.
- The Saved Area was declared in the Data Division with the SA level indicator.
- PROCESS was analogous to spawning a lightweight concurrent task.

---

## SUSPEND

Yielded control during asynchronous processing, allowing other PROCESS invocations or the main program to continue.

```cobol
SUSPEND
```

- Used within a PROCESSed section to temporarily yield the processor.
- Enabled cooperative multitasking between the main program and asynchronous sections.

---

## Saved Area (SA Level Indicator)

The Saved Area was a Data Division construct that provided a buffer and working area for asynchronous record processing. It was identified by the `SA` level indicator in the File Section, analogous to `FD` for file descriptions.

```cobol
FILE SECTION.
FD  MASTER-FILE ...
SA  MASTER-SAVE-AREA ...
    01  SA-RECORD.
        05  SA-KEY       PIC X(10).
        05  SA-DATA      PIC X(200).
```

---

## The USE FOR RANDOM PROCESSING Declarative

A `USE` declarative specifically for asynchronous/random processing supplemented the HOLD/SEEK/PROCESS/SUSPEND verbs:

```cobol
DECLARATIVES.
RANDOM-PROC SECTION.
    USE FOR RANDOM PROCESSING ON MASTER-FILE.
    ...
END DECLARATIVES.
```

---

## Why It Was Removed

1. **Hardware-dependent** — the async I/O model was tightly coupled to the characteristics of specific direct-access storage devices (drums, early disk drives).
2. **Complexity** — the interaction between PROCESS, SUSPEND, Saved Areas, and the main program created difficult concurrency semantics.
3. **Operating system evolution** — as operating systems matured, they took over I/O scheduling, buffering, and device optimization, making application-level I/O hints unnecessary.
4. **Limited implementation** — few compilers fully implemented the async subsystem.
5. **Portability** — the features were inherently non-portable across different hardware configurations.

---

## Modern Alternatives

Modern COBOL programs rely on:

- **Operating system I/O scheduling** for disk optimization
- **Database systems** (via embedded SQL or EXEC CICS) for record locking and concurrent access
- **CICS** or **IMS** transaction managers for multi-user access control
- **File status codes** and `USE AFTER ERROR` for I/O error handling

---

## See Also

- [OPEN](../io/open.md) -- file opening
- [READ](../io/read.md) -- record input
- [WRITE](../io/write.md) -- record output
- [Embedded SQL](../../extensions/embedded-sql.md) -- database access
