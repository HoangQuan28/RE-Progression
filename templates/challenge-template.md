# Writeup Template 

> Use this as a flexible guide, not a rigid form.  
> Focus on your actual thought process: what noticed, tried, misunderstood, and eventually figured out.  
> It is fine if early writeups are messy; improve them over time.

---

## Challenge Information

- **Name:** <challenge name>  
- **Author:** <author name>  
- **Source:** <crackmes.one / Root‑Me / other>  
- **Link:** <URL>  
- **Date solved:** YYYY-MM-DD  
- **Difficulty:** <e.g., 1.0 / 1.5 / 2.0 or “very easy / easy / medium”>  

### Target Details

- **File name:** <e.g., crackme0x1.exe>  
- **Format:** <ELF / PE / Mach‑O / .NET / Java / Android / other>  
- **Architecture:** <x86 / x64 / ARM / other>  
- **Platform:** <Linux / Windows / macOS / cross‑platform>  
- **Notable properties:** <stripped / packed / obfuscated / anti‑debug / none of the above>  

### Tools Used

- **Static analysis:** <Ghidra / IDA / Binary Ninja / radare2 / other>  
- **Dynamic analysis:** <GDB / x64dbg / WinDbg / radare2 / none>  
- **Utilities:** <strings / file / readelf / objdump / xxd / other>  
- **Other:** <Wine / VM / scripts / AI assistance>  

---

## Objective

<State the goal in 1–2 sentences, in your own words.>

Examples:

- “Find a valid name/serial pair.”  
- “Make the program print the success message.”  
- “Understand and reproduce the key generation algorithm.”

---

## First Look

Describe what you did before opening the disassembler.

- How did you identify the file?  
- What did `file` and `strings` show?  
- What did running the program (in a safe environment) tell you?

Include only the commands and outputs that mattered to you.

```bash
file <filename>
strings -n 4 <filename> | less
strings -n 4 <filename> | grep -iE 'key|password|correct|wrong|fail|success'
```

- **File type:** <output of `file`>  
- **Architecture:** <x86/x64/etc.>  
- **Stripped:** <yes/no/unsure>  
- **Other observations:** <sections, imports, obvious packing, etc.>  

### Interesting Strings

List the strings that shaped your hypothesis:

- `"<Enter password/serial/key>"` – <why it matters>  
- `"<Success message>"` – <why it matters>  
- `"<Failure message>"` – <why it matters>  
- Other notable strings:
  - `"<string>"` – <why it matters>  

### Initial Hypothesis

Write 2–5 sentences about what you thought was happening before deep analysis.

Examples:

- “The program reads a serial, compares it against a transformed version of a hardcoded string, and prints success if they match.”  
- “There is a length check, then a character-by-character comparison with some transformation (likely XOR or simple arithmetic).”  
- “The multiple failure messages suggest partial feedback based on how many characters are correct.”

---

## Static Analysis

### Entry Point and Main Flow

Describe how you found the application logic:

- Where did you start in the disassembler?  
- Which functions were runtime noise (CRT, loaders, etc.)?  
- Which functions did you rename, and why?

- **Entry point:** <address or description>  
- **Main function:** <name or address>  
- **Key functions identified:**
  - `<function name or address>` – <purpose>  
  - `<function name or address>` – <purpose>  

Describe the high-level flow in your own words:

1. <Step 1: input, initialization, etc.>  
2. <Step 2: transformation or validation>  
3. <Step 3: success/failure output>  

### Validation Logic

Describe the core validation routine as you understood it:

- **Function:** <name or address>  
- **Inputs:** <where input comes from>  
- **Outputs:** <success/failure, messages, return value>  

Explain the important comparison or transformation:

- <Describe the condition that decides success/failure.>  
- <Show simplified pseudo-code or a short decompiled snippet if helpful.>  

Example structure (adapt to your challenge):

```c
// Pseudo-code example (adapt to your challenge)
bool check_serial(const char *input) {
    const char *secret = "<hardcoded string>";
    int len = strlen(secret);

    if (strlen(input) != len) {
        return false;
    }

    for (int i = 0; i < len; i++) {
        if ((secret[i] ^ i) != (unsigned char)input[i]) {
            return false;
        }
    }

    return true;
}
```

If the decompiler output was confusing, describe:

- What looked wrong or noisy.  
- How you simplified it (e.g., focusing on one comparison, checking assembly, etc.).  

### Key Addresses and Strings

- **Main function:** <address>  
- **Check/validation function:** <address>  
- **Important strings:**
  - `"<success>"` at <address / function>  
  - `"<failure>"` at <address / function>  
- **Other important locations:** <addresses and brief description>  

---

## Dynamic Analysis (if used)

If you used a debugger, describe only what mattered. If not, say so briefly.

### Debugger Setup

- **Debugger:** <GDB / x64dbg / radare2 / other>  
- **Target:** <ELF / PE / other>  
- **Environment:** <native Linux / Wine / VM>  

### Breakpoints and Observations

Describe only the most important breakpoints and what they taught you:

- **Breakpoint 1:** <location, e.g., `main` or `check` function>  
  - **Observation:** <what you saw: registers, memory, input buffer, etc.>  

- **Breakpoint 2:** <e.g., just before the success/failure branch>  
  - **Observation:** <values used in comparison, transformed data, etc.>  

Example (GDB-style):

```gdb
break *0x401234
run
info registers
x/s $rdi
x/20bx $rsi
```

Explain what these values told you, in your own words.

---

## Solution

### Core Idea

Explain the key insight in plain language:

- <What transformation is applied to the input or secret?>  
- <What condition must be satisfied for success?>  

Example:

- “The correct serial is obtained by XORing each character of a hardcoded string with its index.”  
- “The program expects `input[i] = (secret[i] + i) & 0xFF`.”  

### Formula / Algorithm

Write the final formula or algorithm clearly:

```text
input[i] = f(secret, i)
```

or

```c
// Example pattern (adapt to your challenge)
for (int i = 0; i < len; i++) {
    required[i] = secret[i] ^ i;
}
```

### Example Valid Input

Provide at least one concrete example:

- **Name (if applicable):** `<example name>`  
- **Serial / Key:** `<example serial>`  

Briefly describe how you derived it:

- <“I computed `secret[i] ^ i` for each character.”>  
- <“I wrote a small script to generate the serial from the formula.”>  

If you used AI or a script, mention it honestly:

- “I used a short Python script to apply the formula.”  
- “I used an AI assistant to compute the final key from the derived formula.”

---

## What I Learned

Write this as a short reflection, not a checklist.

Include:

- New patterns you saw (e.g., XOR-with-index, checksum, partial feedback).  
- Tool skills you improved (e.g., better Ghidra navigation, GDB commands).  
- Concepts that became clearer (e.g., calling conventions, stack layout, PE structure).  
- Mistakes or dead ends, and what you will do differently next time.

Example bullets:

- “First time seeing partial-feedback messages based on how many characters are correct.”  
- “Learned to trust disassembly more than decompiler output when obfuscation is present.”  
- “Practiced following cross-references from success strings to find the validation function faster.”  
- “Spent too long on runtime initialization code; next time I will ignore CRT noise earlier.”

---

## Appendix (Optional)

### Commands Used

Include only the commands that were actually useful:

```bash
file <filename>
strings -n 4 <filename> | grep -iE '...'
readelf -h <filename>
objdump -d -M intel <filename> | less
```

### Scripts or Verification Code

If you wrote a script or small program to verify or generate the key, include it here (or link to it).

```python
# Example Python snippet (adapt or remove)
secret = "hello"
key = "".join(chr(ord(c) ^ i) for i, c in enumerate(secret))
print(key)
```

### Additional Notes

Any extra observations, alternative approaches, or ideas for future improvements.

---

## Updates (Optional)

