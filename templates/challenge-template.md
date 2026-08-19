# Challenge Name – Site



- Link: URL

- Date: DD-MM-YYYY

- Level: <very easy / easy / 1.0 / etc.>

- Target: <PE x86 / ELF x64 / .NET / etc.>

- Tools: <Ghidra / x64dbg / radare2 / strings / etc.>



## Goal



<What I’m trying to do: find password, patch binary, understand check, etc.>



## My flow (raw)



- First impression:

 <what I noticed first: strings, obvious checks, etc.>

- What I tried:

 <what I did, what happened>

- Where I got stuck:

 <specific function / check / idea that confused me>

- Breakthrough:

 <what finally worked: which function, which condition, which patch>



## What I learned



- <pattern / technique / tool trick>

- <thing I’ll do differently next time>

- <new concept: e.g., “first time seeing XOR decode loop”>



## Technical details (optional)



- Key addresses / function names:

 - `main` at `0x...`

 - check function at `0x...`

- Important logic (pseudo‑code / decompiled snippet):

 ```c

 // example

 if (check\_password(input) == 0) {

     puts("Correct");

 }

 ```

- Patch / solution (if allowed):

 Offset / address: `0x...`

 Original bytes: `...`

 Patched bytes: `...`

 Or: keygen idea / valid example input.



## Updates



- <YYYY-MM-DD>: <added better explanation / fixed mistake / new insight>

