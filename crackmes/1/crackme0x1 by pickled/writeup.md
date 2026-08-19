# crackme0x1 by picklede – crackmes.one

- **Link:** https://crackmes.one/crackme/5ab77f5433c5d40ad448c16d  
- **Date:** 2026-08-19  
- **Level:** 1.0  
- **Target:** PE x86  
- **Tools:** Ghidra, `strings`, Wine, VMware Workstation Pro  

## Goal

Find the activation key.

## My flow (raw)

### First impression

- Ran `file crackme0x1.exe` → `PE32 executable, Intel i386, 4 sections`.  
- Ran `strings crackme0x1.exe` and found messages like:
  - `"Enter activation key:"`
  - `"Congratulations you have found the correct key!"`
  - `"Hmm it's a start... try again.."`
  - `"Ooo.. getting warmer.. try again"`
  - `"Keep trying... nearly there.."`
  - `"Fuck...so close...try again"`
  - `"Who hard can it be?"`
  - `"Not even remotely close, n00b :)"`

**Guess:**  
The program asks for an activation key.  
- Correct key → `"Congratulations you have found the correct key!"`  
- Completely wrong → `"Not even remotely close, n00b :)"`  
- Partially correct → one of the “warmer” messages depending on how many characters are correct.

### What I tried

- Tried running directly on Linux:  
  `./crackme0x1` → failed with “exec format error”.  
  ⇒ Installed Wine to run the Windows binary.

- Ran the binary with Wine to understand normal flow:
  - Prompts: `"Enter activation key:"`
  - Wrong key → `"Not even remotely close, n00b :)"`

- Used Ghidra for static analysis:
  - Entry point immediately calls:
    - `security_init_cookie()`
    - `tmainCRTStartup()`
  - In the function list, I identified:
    - A function I renamed `main()` (see Technical details, call by tmainCRTStartup)
    - A function I renamed `check()` (see Technical details, call by main)

### Where I got stuck + Breakthrough

**Wine / 32‑bit issue**

- Problem: The binary is 32‑bit, but default `wine` on my system was 64‑bit only and couldn’t run 32‑bit programs.  
- Fix:
  ```bash
  sudo dpkg --add-architecture i386
  sudo apt update
  sudo apt install wine32:i386
  ```

**Dynamic analysis**

- Couldn’t get `wine --gdb` working properly.  
- Decided to rely on **full static analysis in Ghidra**.

**Understanding the obfuscated comparison**

- Confusing expression in decompiler:
  ```c
  (int)pcVar6[0x403024] ^ (uint)pcVar6 == (int)pcVar6[0x403728]
  ```
  Both `pcVar6` and `pcVar1` are `char*`.

- Reasoning:
  - `pcVar6` was initially `nullptr`, then set to the start of a string.  
  - `pcVar6[0x403024]` effectively points to the beginning of that string.  
  - `pcVar1` runs in a loop until `*pcVar1 == '\0'`, so `pcVar1 - start_of_string` is essentially `strlen`.

- There’s a lot of decompiler noise and obfuscation, so I focused on the core check:
  ```text
  ((int)pcVar6[0x403024] ^ (uint)pcVar6) == (int)pcVar6[0x403728]
  ```
  Which simplifies to:
  ```text
  (key_string[i] ^ i) == input[i]
  ```
  So the correct input must be:
  ```text
  input[i] = key_string[i] ^ i
  ```

## What I learned

- First time encountering **heavily obfuscated code** with junk instructions and anti‑bruteforce behavior.  
- Second time seeing an **XOR‑based key string**.  
- New things I learned:
  - Ghidra string labels like `s_[string]_[address]`.  
  - Basic Wine / Wine32 setup for 32‑bit Windows binaries.  
  - When only doing static analysis, **trust disassembly more than decompiled C** when obfuscation is present.

## Technical details

### Key addresses / function names

- `entry` at `0x0040182d`  
- `main` at `0x00401000`  
- `check` at `0x00401050`

### Important logic (pseudo‑code)

**Main loop (simplified):**

```c
int main(void) {
    do {
        // ask for input, call check()
    } while (!check());
}
```

**Check function (reconstructed):**

```c
bool check(void) {
    char* input = user_input;
    char* key_string = "phahh`b";

    int len_key = strlen(key_string);
    int len_input = strlen(input);

    if (len_key < len_input) {
        cout << "Not even remotely close, n00b :)\n";
        return false;
    }

    int i = 0;
    while (i < len_input && (key_string[i] ^ i) == input[i]) {
        i++;
    }

    if (i == len_key) {
        cout << "Congratulations you have found the correct key!\n";
        return true;
    }

    switch (i) {
        case 0:
            cout << "Not even remotely close, n00b :)\n";
            break;
        case 1:
            cout << "Hmm it's a start... try again..\n";
            break;
        case 2:
            cout << "Ooo.. getting warmer.. try again\n";
            break;
        case 3:
            cout << "Keep trying... nearly there..\n";
            break;
        case 4:
            cout << "Fuck...so close...try again\n";
            break;
        case 5:
            cout << "Who hard can it be?\n"; // author typo: "Who" instead of "How"
            break;
        default:
            cout << "Not even remotely close, n00b :)\n";
            break;
    }

    return false;
}
```

### Solution

Core comparison:

```text
(key_string[i] ^ i) == input[i]
```

Therefore the correct activation key is:

```text
input[i] = key_string[i] ^ i
```

With `key_string = "phahh`b"`, compute each character XOR its index to get the final key.

I didn’t write a script; I asked an AI assistant to apply this XOR for each character of `key_string = "phahh`b"` and give me the resulting activation key.
