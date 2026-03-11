# holylib

Part of the **holymoly** project.

---

## Overview

`holylib` provides a small set of utility functions that wrap standard C input/output with safer, more predictable behavior. It addresses common pitfalls of raw `scanf` usage — buffer overflows, leftover newlines, and silent type mismatches — through a unified input handler and helper functions.

---

## Files

| File | Description |
|---|---|
| `holylib.h` | Header declarations |
| `holylib.c` | Implementation (contains the functions below) |

---

## Functions

### `void clean()`

Flushes the stdin buffer by consuming all remaining characters up to and including the next newline or EOF.

**Use case:** Called internally after a buffer overflow is detected (i.e., input exceeded the buffer size), preventing leftover characters from corrupting subsequent reads.

```c
void clean();
```

---

### `void pause()`

Displays a "Press ENTER to continue" prompt and waits for the user to press Enter.

**Use case:** Used after error messages to let the user read the output before the screen updates.

```c
void pause();
```

**Output:**
```
> > > Press ENTER to continue < < <
```

---

### `int holyscanf(void *output, char type, int size)`

A safe, type-aware replacement for `scanf`. Reads a line from stdin, strips the trailing newline, and parses it into the requested type.

```c
int holyscanf(void *output, char type, int size);
```

#### Parameters

| Parameter | Type | Description |
|---|---|---|
| `output` | `void *` | Pointer to the variable that will receive the parsed value |
| `type` | `char` | Type specifier: `'s'` for string, `'d'` for int, `'f'` for float |
| `size` | `int` | Max buffer size — **only relevant for `'s'`** (prevents overflow); pass `0` for numeric types |

#### Return Value

| Value | Meaning |
|---|---|
| `1` | Input was successfully read and parsed |
| `0` | Parsing failed (wrong type) or read error |

#### Type Specifiers

| Specifier | Target Type | Notes |
|---|---|---|
| `'s'` | `char[]` | Safe string copy via `strncpy`; always null-terminates |
| `'d'` | `int` | Validates via `sscanf`; prints error and calls `pause()` on failure |
| `'f'` | `float` | Validates via `sscanf`; prints error and calls `pause()` on failure |

#### Usage Examples

```c
// Read a string
char name[64];
holyscanf(name, 's', sizeof(name));

// Read an integer
int age;
holyscanf(&age, 'd', 0);

// Read a float
float price;
holyscanf(&price, 'f', 0);
```

---

## Safety Behaviors

- **Buffer overflow protection:** If the input fills the internal 1024-byte buffer without hitting a newline, `clean()` is called to discard the overflow.
- **Null termination:** String outputs are always null-terminated, even if the input reaches the size limit.
- **Type validation:** For numeric types, failed parses produce an error message rather than silently leaving the output in an undefined state.
- **No dangling newlines:** Input is stripped of the trailing `\n` before processing.

---

## Internal Buffer

`holyscanf` uses a fixed internal buffer of **1024 bytes**. Inputs longer than 1023 characters will be truncated and the excess flushed via `clean()`.

---

## Dependencies

- `<stdio.h>` — `fgets`, `printf`, `getchar`, `sscanf`
- `<string.h>` — `strchr`, `strcspn`, `strlen`, `strncpy`
- `holylib.h` — project header

---

## Project Structure

```
holymoly/
├── holylib/
│   ├── include/
│   │   └── holylib.h
│   ├── src/
│   │   └── holylib.c
│   └── README.md       ← you are here
└── ...
```

---

## Notes

- `holyscanf` is designed for **interactive CLI programs**. It is not suitable for non-interactive or piped input where `pause()` prompts would be disruptive.
- The `size` parameter is ignored for `'d'` and `'f'` types — passing `0` is the convention.

- Error messages print directly to `stdout`, not `stderr`. If your project requires separation of output streams, this would need modification.

