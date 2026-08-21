# Complete Reverse Engineering Checklist

> Build repeatable habits, reduce guesswork, and become faster at analyzing binaries.
>
> **Primary focus:** ELF first  
> **Expansion:** PE, Mach-O, .NET, Android, firmware, and other executable formats  
> **Main tools:** Ghidra, GDB, CLI utilities, and radare2/rizin when useful  
> **Current level:** Beginner crackmes and easy reverse-engineering challenges

---

## 1. Environment Setup

### 1.1 Analysis Environment

- [ ] Use a dedicated analysis directory.
- [ ] Keep original samples separate from modified files.
- [ ] Use a disposable VM for unknown or suspicious binaries.
- [ ] Take a VM snapshot before execution.
- [ ] Disconnect networking when analyzing suspicious software.
- [ ] Do not run unknown binaries as root or administrator.
- [ ] Record the operating system, architecture, and tool versions.

### 1.2 Platform Preparation

- [ ] Confirm that the target can run in your environment.
- [ ] Install required compatibility libraries.
- [ ] For 32-bit Linux binaries, install multilib support if required.
- [ ] For Windows PE files on Linux, use a separate Wine prefix.
- [ ] For Mach-O files, use macOS or an appropriate analysis environment.
- [ ] For Android files, prepare an emulator or test device.

Example Wine prefix:

```bash
WINEPREFIX="$HOME/.wine-re" wineboot
```

---

## 2. Essential Tools

### 2.1 Primary Static Analysis Tool

Choose one main tool and learn it well:

- [ ] **Ghidra** — free, cross-platform, good decompiler.
- [ ] **IDA Free/Pro** — widely used in professional RE.
- [ ] **Binary Ninja** — modern interface and intermediate language.
- [ ] **Cutter/radare2** — useful graphical and command-line workflow.

Recommended starting point:

```text
Primary: Ghidra
Secondary: radare2 or Cutter
```

### 2.2 Debuggers

Use the debugger appropriate for the target:

- [ ] **GDB** — Linux ELF.
- [ ] **x64dbg** — Windows PE.
- [ ] **WinDbg** — Windows debugging.
- [ ] **LLDB** — macOS and some cross-platform targets.
- [ ] **radare2/rizin** — cross-platform debugging.

For GDB, choose one extension:

- [ ] pwndbg
- [ ] GEF
- [ ] peda

Do not install all three at once while learning.

### 2.3 Command-Line Utilities

Universal:

```text
file
strings
xxd
hexdump
```

ELF/Linux:

```text
readelf
objdump
nm
ldd
checksec
strace
ltrace
```

PE/Windows:

```text
objdump
strings
pefile
PE-bear
Detect It Easy
```

Other useful tools:

```text
binwalk
7z
unzip
python3
```

---

## 3. Identify the Target

Before opening the file in a disassembler, determine what it is.

### 3.1 Basic Identification

- [ ] Check the file type.
- [ ] Identify the architecture.
- [ ] Identify the operating system or runtime.
- [ ] Determine whether the file is executable, a library, firmware, or an archive.
- [ ] Check whether symbols are present.
- [ ] Check whether the file appears packed or obfuscated.
- [ ] Calculate a hash if you need to track file versions.

```bash
file ./binary
sha256sum ./binary
```

### 3.2 Common File Types

| Format | Typical clues | Main tools |
|---|---|---|
| ELF | `ELF` header, Linux/Unix | Ghidra, GDB, readelf, objdump |
| PE | `MZ` / `PE` header, Windows | Ghidra, IDA, x64dbg, PE-bear |
| Mach-O | Mach-O magic values, macOS/iOS | Ghidra, IDA, Hopper, LLDB |
| .NET assembly | CLR metadata | ILSpy, dnSpy forks, dotPeek |
| Java archive | `.jar`, `.class` files | JADX, CFR, FernFlower, javap |
| Android APK | ZIP archive containing DEX | JADX, apktool |
| Python bytecode | `.pyc`, `__pycache__` | pycdc, uncompyle6 |
| Firmware | Raw image, filesystem, multiple blobs | binwalk, Ghidra, strings |

---

## 4. Initial Reconnaissance

### 4.1 Universal Checks

- [ ] Run `file`.
- [ ] Run `strings`.
- [ ] Search strings for useful clues.
- [ ] Inspect the raw bytes.
- [ ] Note suspicious sections or embedded files.
- [ ] Look for prompts, success messages, error messages, URLs, filenames, and algorithm names.

```bash
file ./binary
strings -n 4 ./binary | less
strings -n 4 ./binary | grep -iE \
'key|password|serial|flag|secret|correct|success|wrong|fail|error'
xxd -l 256 ./binary
```

### 4.2 ELF Checks

```bash
readelf -h ./binary
readelf -S ./binary
readelf -l ./binary
readelf -s ./binary
readelf -d ./binary
objdump -d -M intel ./binary | less
nm ./binary 2>/dev/null
ldd ./binary
checksec --file=./binary
```

Check:

- [ ] 32-bit or 64-bit.
- [ ] Endianness.
- [ ] Entry point.
- [ ] PIE.
- [ ] NX.
- [ ] RELRO.
- [ ] Stack canary.
- [ ] Imported libraries.
- [ ] Stripped or unstripped symbols.

### 4.3 PE Checks

Check:

- [ ] PE32 or PE32+.
- [ ] x86 or x64.
- [ ] Entry point.
- [ ] Image base.
- [ ] Sections.
- [ ] Imported DLLs and functions.
- [ ] CLR/.NET metadata.
- [ ] Packing or suspicious section names.

Use:

- [ ] Ghidra.
- [ ] IDA.
- [ ] PE-bear.
- [ ] Detect It Easy.
- [ ] `objdump` or Python `pefile`.

### 4.4 Managed-Code Checks

If the target is .NET, Java, Kotlin, or Android:

- [ ] Check whether high-level source can be recovered directly.
- [ ] Inspect namespaces, packages, classes, and methods.
- [ ] Search for strings and entry points.
- [ ] Check whether obfuscation is present.
- [ ] Use bytecode or IL when the decompiler output is misleading.

---

## 5. Safe Execution

Only execute the binary in an appropriate environment.

### 5.1 Before Execution

- [ ] Make a copy of the original.
- [ ] Confirm the environment is isolated.
- [ ] Decide whether networking is required.
- [ ] Record the expected command-line arguments.
- [ ] Prepare a harmless test input.
- [ ] Avoid using real credentials or personal files.

### 5.2 Observe Normal Behavior

Record:

- [ ] Prompts.
- [ ] Expected input format.
- [ ] Success messages.
- [ ] Failure messages.
- [ ] Number of attempts allowed.
- [ ] Files created or modified.
- [ ] Network activity.
- [ ] Process behavior.

For ELF:

```bash
./binary
strace -f ./binary
ltrace ./binary
```

For PE:

```bash
WINEPREFIX="$HOME/.wine-re" wine ./binary.exe
```

Use a VM and host-based monitoring if the binary is not clearly a practice challenge.

---

## 6. Static Analysis Workflow

### 6.1 Load the File

- [ ] Import the binary into Ghidra or your chosen tool.
- [ ] Select the correct language and architecture.
- [ ] Run auto-analysis.
- [ ] Save a project copy.
- [ ] Record the image base and entry point.

### 6.2 Find the Entry Point

Look for:

| Target | Possible entry functions |
|---|---|
| ELF | `_start`, `main` |
| PE | entry point, `main`, `WinMain`, `DllMain` |
| Mach-O | `_main`, entry point |
| .NET | managed entry method |
| Java | `main` |
| Android | activities, services, exported components |

- [ ] Start from the entry point if `main` is not visible.
- [ ] Follow calls until reaching application-specific code.
- [ ] Ignore compiler/runtime initialization until necessary.

### 6.3 Map Important Functions

- [ ] Count or list functions.
- [ ] Identify imported functions.
- [ ] Identify input functions.
- [ ] Identify comparison functions.
- [ ] Identify output functions.
- [ ] Identify transformation or decryption functions.
- [ ] Rename unclear functions after understanding their purpose.
- [ ] Rename important variables.
- [ ] Add comments to important instructions and branches.

Possible names:

```text
read_input
check_length
validate_serial
transform_data
decode_string
print_success
print_failure
```

### 6.4 Follow Strings and Cross-References

- [ ] Find success messages.
- [ ] Find failure messages.
- [ ] Find input prompts.
- [ ] Follow references to each string.
- [ ] Identify the function containing the comparison or branch.
- [ ] Trace backward from the success branch.

Typical strategy:

```text
Success string
    ↓
Cross-reference
    ↓
Success branch
    ↓
Comparison
    ↓
Input transformation
    ↓
Required value
```

### 6.5 Understand Control Flow

- [ ] Trace the normal execution path.
- [ ] Identify conditional branches.
- [ ] Identify loops.
- [ ] Identify early exits.
- [ ] Identify length checks.
- [ ] Identify success and failure paths.
- [ ] Note which values control each branch.
- [ ] Check whether the decompiler correctly represents the logic.

---

## 7. Common Validation Patterns

Look for these patterns in beginner and intermediate crackmes.

### 7.1 Direct Comparison

```c
strcmp(input, "known_value") == 0
```

### 7.2 Character-by-Character Comparison

```c
for (int i = 0; i < length; i++) {
    if (input[i] != expected[i]) {
        return false;
    }
}
```

### 7.3 XOR Transformation

```c
if ((secret[i] ^ i) != input[i]) {
    return false;
}
```

or:

```c
input[i] = input[i] ^ 0x42;
```

### 7.4 Arithmetic Transformation

```c
expected[i] = input[i] + 3;
expected[i] = input[i] - i;
expected[i] = input[i] * 2;
```

### 7.5 Checksum or Character Sum

```c
sum += input[i];

if (sum == expected_sum) {
    success();
}
```

### 7.6 Partial Feedback

Different messages may reveal how many characters are correct.

- [ ] Check whether the comparison stops at the first mismatch.
- [ ] Check whether the loop counter is later used to select a message.
- [ ] Determine whether the input length must equal the expected length.

---

## 8. Dynamic Analysis Workflow

Use dynamic analysis to verify your static hypothesis.

### 8.1 Breakpoint Strategy

Set breakpoints at:

- [ ] `main` or the application entry point.
- [ ] Input functions.
- [ ] The validation function.
- [ ] The comparison instruction.
- [ ] The instruction after a transformation loop.
- [ ] Success and failure branches.
- [ ] Output functions.
- [ ] Function returns.

### 8.2 GDB Essentials

```bash
gdb -q ./binary
```

```gdb
break main
run
info breakpoints
continue
step
next
finish
backtrace
info registers
x/20i $rip
x/20gx $rsp
x/s $rdi
print $rax
```

Useful architecture reminder for Linux x86-64:

```text
1st argument: RDI
2nd argument: RSI
3rd argument: RDX
4th argument: RCX
return value: RAX
instruction pointer: RIP
stack pointer: RSP
```

For 32-bit binaries, argument and return-value locations may differ.

### 8.3 Radare2 Essentials

```bash
r2 -d -AA ./binary
```

```text
afl              # List functions
pdf @ main       # Disassemble main
s address        # Seek to an address
axt @ address    # Find references
db main          # Set breakpoint
dc               # Continue
ds               # Step one instruction
dr               # Show registers
px 64 @ address  # Print bytes
psz @ address    # Print zero-terminated string
```

### 8.4 Inspect Program State

At a useful breakpoint:

- [ ] Inspect registers.
- [ ] Follow pointer-valued registers.
- [ ] Inspect the stack.
- [ ] Inspect input buffers.
- [ ] Inspect global data.
- [ ] Inspect heap allocations.
- [ ] Compare memory before and after a transformation.
- [ ] Confirm the value used by the final comparison.

---

## 9. Decompiler Sanity Checks

Decompiler output is a hypothesis, not the original source code.

When output looks strange:

- [ ] Compare it with the assembly.
- [ ] Check variable types.
- [ ] Check pointer arithmetic.
- [ ] Check array indexing.
- [ ] Check signed versus unsigned values.
- [ ] Check loop boundaries.
- [ ] Check calling conventions.
- [ ] Check whether a value is initialized before use.
- [ ] Check whether the condition is logically possible.
- [ ] Test the suspected logic with a small program.

Common warning signs:

- [ ] Many unexplained casts.
- [ ] Impossible pointer offsets.
- [ ] Variables with incorrect types.
- [ ] Missing parameters.
- [ ] Huge expressions representing simple operations.
- [ ] Unreachable-looking branches.
- [ ] Incorrectly inferred loop conditions.

Recommended process:

1. Select the suspicious function.
2. Read the relevant assembly.
3. Identify inputs, outputs, and side effects.
4. Rewrite the logic as simple pseudo-code.
5. Test the pseudo-code.
6. Update the function and variable names in the disassembler.

---

## 10. Memory and Data Hunting

Inspect data at high-value locations:

- [ ] After a decoding or transformation loop.
- [ ] Before a success message.
- [ ] Before a comparison.
- [ ] At function return.
- [ ] In registers holding pointers.
- [ ] On the stack after input is read.
- [ ] In `.data` and `.rodata`.
- [ ] In `.bss` after initialization.
- [ ] In heap allocations returned by `malloc` or similar functions.
- [ ] In buffers passed to `printf`, `puts`, `write`, or equivalent APIs.

Useful GDB examples:

```gdb
x/s $rax
x/s $rdi
x/32bx $rsi
x/32gx $rsp
x/20i $rip
```

---

## 11. Format-Specific Extensions

### 11.1 ELF

- [ ] Inspect headers with `readelf`.
- [ ] Inspect symbols with `nm`.
- [ ] Inspect imports with `ldd` and dynamic sections.
- [ ] Check protections with `checksec`.
- [ ] Trace system calls with `strace`.
- [ ] Trace library calls with `ltrace`.
- [ ] Debug with GDB.

### 11.2 PE

- [ ] Inspect sections and imports.
- [ ] Check for .NET/CLR metadata.
- [ ] Check for packing.
- [ ] Use x64dbg for runtime analysis.
- [ ] Inspect Windows API calls.
- [ ] Use Wine only when appropriate and isolated.

### 11.3 Mach-O

- [ ] Identify CPU architecture.
- [ ] Inspect load commands.
- [ ] Check symbols and Objective-C/Swift metadata.
- [ ] Use LLDB for debugging.
- [ ] Use Ghidra, IDA, or Hopper for static analysis.

### 11.4 .NET

- [ ] Open with ILSpy, dnSpy fork, or dotPeek.
- [ ] Browse namespaces, classes, and methods.
- [ ] Search for strings and validation routines.
- [ ] Check for obfuscation.
- [ ] Inspect IL if C# decompilation is misleading.

### 11.5 Java and Android

- [ ] Open `.jar` or `.apk` with JADX.
- [ ] Inspect `AndroidManifest.xml`.
- [ ] Identify activities, services, and receivers.
- [ ] Search for validation and crypto code.
- [ ] Use apktool for resources and Smali.
- [ ] Analyze native `.so` libraries separately with Ghidra or IDA.

### 11.6 Firmware and Embedded Files

- [ ] Identify the processor architecture.
- [ ] Search for embedded filesystems.
- [ ] Check strings and compression.
- [ ] Use binwalk where appropriate.
- [ ] Load extracted components into the correct Ghidra architecture.
- [ ] Identify memory maps and peripheral addresses.

---

## 12. Verification Labs

### Lab 1: Compiler Pattern Recognition

Create and compile a small program:

```c
#include <stdio.h>

int main(void) {
    int a = 5;
    int b = 10;
    int c = a + b;

    printf("Result: %d\n", c);
    return 0;
}
```

Compile:

```bash
gcc -o lab1 lab1.c
```

Practice:

- [ ] Locate `main`.
- [ ] Identify constants.
- [ ] Identify where variables are stored.
- [ ] Match the addition to its assembly instructions.
- [ ] Observe the function call to `printf`.

### Lab 2: XOR Loop

```c
#include <stdio.h>

int main(void) {
    char data[] = "abcdefgh";

    for (int i = 0; i < 8; i++) {
        data[i] ^= 0x42;
    }

    printf("%s\n", data);
    return 0;
}
```

Practice:

- [ ] Find the loop.
- [ ] Identify the loop counter.
- [ ] Identify the XOR constant.
- [ ] Inspect `data` before and after the loop.
- [ ] Reconstruct the loop as pseudo-code.

### Lab 3: Function Call Tracing

```c
#include <stdio.h>

void transform(char *data) {
    for (int i = 0; data[i] != '\0'; i++) {
        data[i] += 1;
    }
}

int main(void) {
    char message[] = "gdkknvnqkc";

    transform(message);
    printf("%s\n", message);

    return 0;
}
```

Practice:

- [ ] Locate `transform`.
- [ ] Set a breakpoint before the call.
- [ ] Inspect the argument.
- [ ] Step through the loop.
- [ ] Inspect the data after the function returns.

---

## 13. Useful Automation

### 13.1 Basic Reconnaissance Script

```bash
#!/usr/bin/env bash

set -u

BINARY="${1:-}"

if [ -z "$BINARY" ]; then
    echo "Usage: $0 <binary>"
    exit 1
fi

echo "=== File information ==="
file "$BINARY"

echo
echo "=== SHA-256 ==="
sha256sum "$BINARY"

echo
echo "=== Interesting strings ==="
strings -n 4 "$BINARY" |
    grep -iE 'flag|key|password|serial|secret|success|correct|wrong|fail' ||
    true

echo
echo "=== ELF security information ==="
if command -v checksec >/dev/null 2>&1; then
    checksec --file="$BINARY"
else
    echo "checksec is not installed"
fi
```

Run it:

```bash
chmod +x quick-recon.sh
./quick-recon.sh ./binary
```

Use scripts to automate repetitive inspection, not to replace understanding.

---

## 14. Speed Habits

### Ghidra

- [ ] `G` — Go to address or symbol.
- [ ] `N` — Rename symbol.
- [ ] `;` — Add comment.
- [ ] `X` — Show cross-references.
- [ ] `L` — Set a label.
- [ ] Use graph view to understand branches.
- [ ] Keep decompiler and disassembly visible together.

### GDB

- [ ] Use command history.
- [ ] Use tab completion.
- [ ] Use `Ctrl+R` to search command history.
- [ ] Create aliases for commands you use repeatedly.
- [ ] Save useful debugger commands in a file.

### General

- [ ] Start with strings and program output.
- [ ] Trace backward from success and failure paths.
- [ ] Rename functions as soon as their purpose becomes clear.
- [ ] Separate confirmed facts from guesses.
- [ ] Verify every important hypothesis.
- [ ] Use small test programs to understand unfamiliar compiler patterns.

---

## 15. When You Are Stuck

1. [ ] Re-read the challenge description.
2. [ ] Run the program again and record its behavior.
3. [ ] Search all strings again.
4. [ ] Find the success and failure messages.
5. [ ] Follow their cross-references.
6. [ ] Focus on one function at a time.
7. [ ] Identify the comparison controlling the result.
8. [ ] Check input length and loop boundaries.
9. [ ] Compare decompiler output with assembly.
10. [ ] Use a debugger to confirm your hypothesis.
11. [ ] Reproduce the suspected algorithm in a small script or program.
12. [ ] If using AI or external help, ask about one specific instruction, loop, or expression.
13. [ ] Return to the binary and verify the explanation yourself.

Avoid immediately searching for a complete solution. First try to identify the exact point you do not understand.

---

## 16. Completion Checklist

Before considering an analysis complete:

- [ ] I identified the file format and architecture.
- [ ] I identified the entry point or application entry function.
- [ ] I found the main input path.
- [ ] I found the success and failure paths.
- [ ] I identified the important validation function.
- [ ] I understand the key loop or transformation.
- [ ] I verified the logic dynamically or with an independent test.
- [ ] I can explain the result in simple pseudo-code.
- [ ] I recorded the important addresses, strings, and assumptions.
- [ ] I can reproduce the solution without relying entirely on the decompiler.

---

## 17. Progress Metrics

You are improving when:

- [ ] You can identify the file format quickly.
- [ ] You can locate the application logic without spending too long in runtime startup code.
- [ ] You recognize common comparison, XOR, checksum, and loop patterns.
- [ ] Assembly begins to look structured rather than random.
- [ ] You cross-check suspicious decompiler output automatically.
- [ ] You use dynamic analysis to confirm hypotheses instead of guessing.
- [ ] You can explain a function without copying its decompiler output.
- [ ] You need less external help for basic transformations.
- [ ] You can move from easy to harder challenges without skipping fundamentals.

---

> Reverse engineering is pattern recognition, careful observation, verification, and patience.
>
> Start with facts. Form a hypothesis. Test it. Update your model. Repeat.