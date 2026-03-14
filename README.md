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

## Installation & Usage

`holylib` is not a compiled library — you just copy the two files into your project and compile them alongside your own code. No installers, no package managers.

### Step 1 — Copy the files

Download or clone the repo, then copy these two files into your project folder:

```
holylib.h
holylib.c
```

### Step 2 — Include the header

In your `.c` file:

```c
#include "holylib.h"
```

### Step 3 — Compile together

You must compile `holylib.c` together with your own source file. See the IDE-specific instructions below.

---

### Visual Studio Code

#### Windows (MinGW)

Make sure you have MinGW installed and `gcc` is available in your PATH. Then in the terminal:

```bash
gcc main.c holylib.c -o main.exe
./main.exe
```

To run this automatically via VSCode's build task, open `.vscode/tasks.json` and set the `args` to include both files:

```json
"args": [
    "-g",
    "main.c",
    "holylib.c",
    "-o",
    "${fileDirname}\\main.exe"
]
```

#### Linux / macOS (GCC)

GCC is usually pre-installed. If not:
- **Linux (Debian/Ubuntu):** `sudo apt install gcc`
- **macOS:** `xcode-select --install`

Then in the terminal:

```bash
gcc main.c holylib.c -o main
./main
```

Same `tasks.json` approach applies, just change the output to `main` instead of `main.exe` and use forward slashes.

---

### Dev-C++ (Windows only)

Dev-C++ requires you to manually add `holylib.c` to the project — just including the header is not enough.

#### New Project

1. Open Dev-C++: **File → New → Project → Console Application → C**
2. Save the project in the same folder as `holylib.h` and `holylib.c`
3. Add `holylib.c`: **Project → Add to Project** → select `holylib.c`
4. Press **F11** to compile and run

#### Existing Project

1. Copy `holylib.h` and `holylib.c` into your project folder
2. In Dev-C++: **Project → Add to Project** → select `holylib.c`
3. Press **F11** to compile and run

> <!!!> If you only add `holylib.h` without adding `holylib.c` to the project, Dev-C++ will throw undefined reference errors at link time. Both files must be in the project <!!!>

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

### `int holysscanf(const char *type, int size, ...)`

A safe multi-value input function. Reads a full line from stdin, then parses it against a format string — similar to `sscanf` but with input safety and result validation.

```c
int holysscanf(const char *type, int size, ...);
```

#### Parameters

| Parameter | Type | Description |
|---|---|---|
| `type` | `const char *` | `sscanf`-style format string (e.g. `"%d %d %f"`) |
| `size` | `int` | Number of values expected to be parsed |
| `...` | variadic | Pointers to the variables that will receive the parsed values |

#### Return Value

| Value | Meaning |
|---|---|
| `1` | All expected values were successfully parsed |
| `0` | Read failed, or number of parsed values did not match `size` |

#### Usage Examples

```c
int a, b, c;
printf("Enter three integers: ");
holysscanf("%d %d %d", 3, &a, &b, &c);

float x, y;
printf("Enter two floats: ");
holysscanf("%f %f", 2, &x, &y);

int n;
float f;
printf("Enter an int and a float: ");
holysscanf("%d %f", 2, &n, &f);
```

#### When to use `holysscanf` vs `holyscanf`

| Situation | Use |
|---|---|
| One value per prompt | `holyscanf` |
| Multiple values on one line | `holysscanf` |

Prefer `holyscanf` for single values — it provides more specific error messages ("invalid input for an int") compared to `holysscanf`'s generic format mismatch error.

---

## Safety Behaviors

- **Buffer overflow protection:** If the input fills the internal 1024-byte buffer without hitting a newline, `clean()` is called to discard the overflow.
- **Null termination:** String outputs are always null-terminated, even if the input reaches the size limit.
- **Type validation:** For numeric types, failed parses produce an error message rather than silently leaving the output in an undefined state.
- **No dangling newlines:** Input is stripped of the trailing `\n` before processing.
- **Result validation:** `holysscanf` checks that the number of successfully parsed values matches `size`, rejecting partial parses.

---

## Internal Buffer

Both `holyscanf` and `holysscanf` use a fixed internal buffer of **1024 bytes**. Inputs longer than 1023 characters will be truncated and the excess flushed via `clean()`.

---

## Dependencies

- `<stdio.h>` — `fgets`, `printf`, `getchar`, `sscanf`, `vsscanf`
- `<string.h>` — `strchr`, `strcspn`, `strlen`, `strncpy`
- `<stdarg.h>` — `va_list`, `va_start`, `va_end`
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
- The `size` parameter in `holyscanf` is ignored for `'d'` and `'f'` types — passing `0` is the convention.
- `holysscanf` may partially write to output variables before detecting a format mismatch. This is consistent with standard `sscanf` behavior — reinitialize variables before retrying on failure.
- Error messages print directly to `stdout`, not `stderr`. If your project requires separation of output streams, this would need modification.
---

## Contributing

Feel free to fork this repository and submit pull requests for improvements, bug fixes, or additional features. Contributions are welcome!

- Fork the repo
- Create a new branch for your changes
- Submit a pull request with a clear description

For questions or suggestions, open an issue in the repository.
