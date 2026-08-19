# crackme0x1 by pickled

- **Source:** [crackmes.one](https://crackmes.one/crackme/5ab77f5433c5d40ad448c16d)
- **Date:** 2026-08-19
- **Difficulty:** 1.0
- **Target:** PE32 / x86
- **Tools:** Ghidra, `file`, `strings`, Wine, VMware Workstation Pro

## Goal

Find the activation key.

## First look

I started by identifying the file:

```bash
file crackme0x1.exe
```

The result showed that it was a 32-bit Windows executable:

```text
PE32 executable, Intel i386, 4 sections
```

Since the file is a Windows executable, I could not run it directly on Linux:

```bash
./crackme0x1.exe
```

This resulted in an `exec format error`.

I then used `strings` to look for useful messages:

```bash
strings crackme0x1.exe
```

Some interesting strings were:

```text
Enter activation key:
Congratulations you have found the correct key!
Hmm it's a start... try again..
Ooo.. getting warmer.. try again
Keep trying... nearly there..
Fuck...so close...try again
Who hard can it be?
Not even remotely close, n00b :)
```

These messages suggested that the program gives feedback based on how many characters of the activation key are correct. My initial guess was that the program probably compares the input character by character.

## Running the program

I installed Wine so that I could run the executable:

```bash
sudo dpkg --add-architecture i386
sudo apt update
sudo apt install wine32:i386
```

After setting up Wine, I ran the program and confirmed its basic behavior:

```text
Enter activation key:
```

Entering an incorrect value produced:

```text
Not even remotely close, n00b :)
```

The feedback messages made me think that the validation function probably stops at the first incorrect character and uses the position of that character to select a response.

## Finding the validation function

I imported the executable into Ghidra and ran the normal analysis.

The entry point first went through runtime initialization:

- `security_init_cookie()`
- `tmainCRTStartup()`

At first, these functions were distracting because they were not part of the crackme’s actual logic. I followed the calls until I reached the application-specific code.

I renamed the relevant functions:

- `0x00401000` → `main`
- `0x00401050` → `check`

The simplified program flow looked like this:

```text
tmainCRTStartup
        ↓
      main
        ↓
      check
        ↓
 success or feedback message
```

The `main` function repeatedly asks for an activation key and calls `check()`.

## Getting stuck in the decompiler

The decompiler output was difficult to read. One expression looked like this:

```c
(int)pcVar6[0x403024] ^ (uint)pcVar6 == (int)pcVar6[0x403728]
```

At first, I was unsure what this expression meant. The variables were both represented as `char *`, and the pointer arithmetic made the output look much more complicated than it probably was.

I also saw a loop that walked through a string until it reached a null byte. This was effectively calculating the string length:

```c
while (*pcVar1 != '\0') {
    pcVar1++;
}
```

So:

```text
pcVar1 - start_of_string
```

was effectively:

```text
strlen(string)
```

The decompiler output contained a lot of noise, so I stopped trying to understand every generated variable and focused on the comparison that controls the result.

## Understanding the key check

After following the relevant variables and comparing the decompiler output with the disassembly, I simplified the important condition to:

```text
(key_string[i] ^ i) == input[i]
```

The hardcoded string was:

```text
phahh`b
```

This means the program does not compare the input directly with the hardcoded string. Instead, it XORs each character with its index:

```text
input[i] = key_string[i] ^ i
```

The validation loop can be represented like this:

```c
bool check(void) {
    char *input = user_input;
    char *key_string = "phahh`b";

    int len_key = strlen(key_string);
    int len_input = strlen(input);

    if (len_key < len_input) {
        puts("Not even remotely close, n00b :)");
        return false;
    }

    int i = 0;

    while (i < len_input &&
           (key_string[i] ^ i) == input[i]) {
        i++;
    }

    if (i == len_key) {
        puts("Congratulations you have found the correct key!");
        return true;
    }

    switch (i) {
        case 0:
            puts("Not even remotely close, n00b :)");
            break;
        case 1:
            puts("Hmm it's a start... try again..");
            break;
        case 2:
            puts("Ooo.. getting warmer.. try again");
            break;
        case 3:
            puts("Keep trying... nearly there..");
            break;
        case 4:
            puts("Fuck...so close...try again");
            break;
        case 5:
            puts("Who hard can it be?");
            break;
        default:
            puts("Not even remotely close, n00b :)");
            break;
    }

    return false;
}
```

The `switch` explains the feedback messages: `i` represents how many characters matched before the first mismatch.

## Solution

The required key is generated with:

```text
input[i] = key_string[i] ^ i
```

Using:

```text
key_string = "phahh`b"
```

the key can be calculated character by character.

I did not write a script for this step. I used an AI assistant to apply the XOR operation to each character after I had already derived the algorithm from the binary.

The important part was not the calculation itself, but understanding the validation logic:

```text
(key_string[i] ^ i) == input[i]
```

## What I learned

- This was my first encounter with heavily confusing decompiler output containing junk code and unnecessary variables.
- The decompiled expression looked intimidating, but the actual validation logic was simple once I focused on the comparison.
- The “warmer” messages revealed that the check works character by character.
- I learned to follow cross-references from success and failure strings.
- I learned that Ghidra’s decompiler output should be treated as an interpretation, not the original source code.
- When the decompiler becomes confusing, I should compare it with the assembly and isolate the smallest important condition.
- I learned how to install Wine32 support for running 32-bit Windows programs on Linux.

## Important addresses

```text
Entry point: 0x0040182d
main:        0x00401000
check:       0x00401050
```

## Takeaway

This crackme looked more complicated than it really was because of the decompiler output. The useful path was:

```text
Find success string
        ↓
Follow its cross-reference
        ↓
Locate the validation function
        ↓
Ignore decompiler noise
        ↓
Simplify the comparison
        ↓
Derive the XOR formula
```

The main lesson was to focus on the condition that decides success rather than trying to understand every line of noisy decompiled code.
