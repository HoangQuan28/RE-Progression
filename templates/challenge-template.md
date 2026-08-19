\# <Challenge Name> – <Site>



\- Link: <URL>

\- Date: <YYYY-MM-DD>

\- Level: <very easy / easy / 1.0 / etc.>

\- Target: <PE x86 / ELF x64 / .NET / etc.>

\- Tools: <Ghidra / x64dbg / radare2 / strings / etc.>



\## Goal



<What I’m trying to do: find password, patch binary, understand check, etc.>



\## My flow (raw)



\- First impression:

&#x20; - <what I noticed first: strings, obvious checks, etc.>

\- What I tried:

&#x20; - <attempt 1: what I did, what happened>

&#x20; - <attempt 2: dead end, wrong assumption, etc.>

\- Where I got stuck:

&#x20; - <specific function / check / idea that confused me>

\- Breakthrough:

&#x20; - <what finally worked: which function, which condition, which patch>



\## What I learned



\- <pattern / technique / tool trick>

\- <thing I’ll do differently next time>

\- <new concept: e.g., “first time seeing XOR decode loop”>



\## Technical details (optional)



\- Key addresses / function names:

&#x20; - `main` at `0x...`

&#x20; - check function at `0x...`

\- Important logic (pseudo‑code / decompiled snippet):

&#x20; ```c

&#x20; // example

&#x20; if (check\_password(input) == 0) {

&#x20;     puts("Correct");

&#x20; }

&#x20; ```

\- Patch / solution (if allowed):

&#x20; - Offset / address: `0x...`

&#x20; - Original bytes: `...`

&#x20; - Patched bytes: `...`

&#x20; - Or: keygen idea / valid example input.



\## Updates



\- <YYYY-MM-DD>: <added better explanation / fixed mistake / new insight>

