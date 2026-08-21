## Challenge Information

- **Name:** DPRK Loyalty Evaluation  
- **Author:** 23x41  
- **Source:** crackmes.one
- **Link:** https://crackmes.one/crackme/6a5995410b25d281a656896f
- **Date solved:** 19/08/2026 
- **Difficulty:** 1.2  

### Target Details

- **File name:** juche_loyalty_test  
- **Format:** ELF>  
- **Architecture:** x64 
- **Platform:** Linux
- **Notable properties:** not stripped, dynamically linked, anti-debug(ptrace)  

### Tools Used

- **Static analysis:** Ghidra
- **Dynamic analysis:** GDB 
- **Utilities:** strings, file, readelf, xxd, ltrace, strace
- **Other:**  VM 

---

## Objective

Goal is to answer 2 question correctly to pass
Alternative path: Enter RE-EDUCATION MODE and exploit format string vulnerability.

---

## First Look

Before opening the disassembler, I ran:
 - file to identify the format of the file
 - strings to see hardcoded strings for clues
 - strace, ltrace blocked by anti debugger ptrace

```bash
file juche_loyalty_test 
strings -n 4 juche_loyalty_test | less
strings juche_loyalty_test | egrep -i "key|password|answer|solution|flag"
```

- **File type:** juche_loyalty_test: ELF 64-bit LSB executable, x86-64, version 1 (SYSV), dynamically linked, interpreter /lib64/ld-linux-x86-64.so.2, BuildID[sha1]=64727242c7789c5856649372537898748fd5bce2, for GNU/Linux 3.2.0, with debug_info, not stripped  
- **Architecture:** x64  
- **Stripped:** no
- **Other observations:** none  

### Interesting Strings
Questions:
  "Question 1: How many holes-in-one has the Supreme Leader officially achieved?"
  "Question 2: At which sacred mountain did the Supreme Leader receive the Juche mandate?"

If answer correct:
  "[GLORIOUS VICTORY]
  The Supreme Leader has accepted your loyalty.
  Your Songbun is now the highest possible.
  You may now access the inner party terminal."

If answer incorrect:
  "[RE-EDUCATION MODE ACTIVATED]
  Write your self-criticism. Be thorough, comrade.
  Self-criticism:"
  "WRONG! Your ideological deviation has been noted."
  "INCORRECT! You have failed the test of loyalty."

Anti-Debugger:
  "[WARNING] Debugger detected. The Ministry of State Security has been notified."
  "ptrace"

Hardcoded flag:
  "FLAG{0x8A7_JUCHE_FORMAT_STRING_MASTERY}"
### Initial Hypothesis

So this mean the file use ptrace to for anti-debug, and it workflow can be simplified:
  Question 1
    ->Correct: Continue to question 2
    ->Incorrect: Continue to RE-EDUCATION MODE
  
  Question 2
    ->Correct: Win message -> the hardcoded flag
    ->Incorrect: Continue to RE-EDUCATION MODE
  
  RE-EDUCATION MODE:
    ->Program log out
    ->Exploit the format string vulnerability of the printf() function

---

## Static Analysis

### Entry Point and Main Flow

Because it is NOT STRIPPED, Ghidra can identified the main function so I can start there rightaway.

- **Entry point:** 0x00401000 
- **Main function:** 0x0040141a  
- **Key functions identified:**
  - take_loyalty_test() – asking question and check the answer (0x004012e4)

Describe the high-level flow in your own words:

1. <Step 1: initialization
2. <Step 2: validation through take_loyalty_test() function  
3. <Step 3: success -> grant_party_membership()
            failure -> self_criticism_mode()
**Note:** The flag is hardcoded in the `grant_party_membership()` function at `.rodata:0x404000`.
Simply finding it through static analysis (Ghidra/strings) is sufficient to solve the challenge.

### Validation Logic

- **Function:** take_loyalty_test()  
- **Inputs:** use cin
- **Outputs:** success ->  grant_party_membership() / fail -> self_criticism_mode()

The answer for question 1 is hardcode as "38\0"
The answer for question 2 is hardcode as "Mount Paektu\0"
However for question 2, typical cin function stop reading after space. Hence making normally input will never work.


Example structure :

```c
void take_loyalty_test(void)

{
  int check;
  char answer [64];
  char correct_q2 [13];
  char correct_q1 [3];
  
  correct_q1[0] = '3';
  correct_q1[1] = '8';
  correct_q1[2] = '\0';
  strncpy(correct_q2,"Mount Paektu",13);
  cout<<"\nQuestion 1: How many holes-in-one has the Supreme Leader officially achieved?\n";
  cout<<"Your answer: ";
  cin>>answer;
  check = strcmp(answer,correct_q1);
  if (check == 0) {
    cout<<"\nQuestion 2: At which sacred mountain did the Supreme Leader receive the Juche mandate?\n";
    cout<<"Your answer: ";
    cin>>answer;
    check = strcmp(answer,correct_q2);
    if (check == 0) {
      grant_party_membership();
    }
    else {
      cout<<"\nINCORRECT! You have failed the test of loyalty.\n";
      self_criticism_mode();
    }
  }
  else {
    cout<<"\nWRONG! Your ideological deviation has been noted.\n";
    self_criticism_mode();
  }
  return;
}
```
### Key Addresses and Strings

- **Main function:** 0x0040141a  
- **Check/validation function:** 0x004012e4
- **Important strings:**
  - `"<success>"` at grant_party_membership() at 0x004011a6 
  - `"<failure>"` at self_criticism_mode() at 0x00401239

- The self_criticism_mode() function have critical spot to exploit:
```code
char criticism [128];
printf(criticism);
```
=> Can apply format string exploitation with stack memory leak

In  Re-education mode:
I tried normal input such as
```text
AAAA
```
The program printf out AAAA
Then I tried to do
```text
AAAAAAAA %p %p %p %p %p %p %p %p %p %p %p %p %p
```
The output is:
```text
AAAAAAAA 0xa 0x417760 0x30 (nil) (nil) 0x4141414141414141 0x2520702520702520 0x2070252070252070 0x7025207025207025 0x2520702520702520 0x70252070252070 0x7fffffffdcf0 0x404080
```
I can see there is
```text
0x4141414141414141 (AAAAAAAA)
```
To confirmed of stack where the printf input belong I do
```text
AAAAAAAA %6$p (6 is counted base on last output)
```
Output:
```text
AAAAAAAA 0x4141414141414141
```

I confirmed the vulnerability:
- Input appears at position 6: `AAAAAAAA %6$p → 0x4141414141414141`
- Leaked over 200 stack values using `%N$p`
- **Finding:** No direct pointer to the found on the stack

This mean I can do format string vuln exploitation and stack buffer overflow for this to make the program print the flag itself. However my current skill (19/08/2026) not sufficient for this. 

---

## Solution
  FLAG{0x8A7_JUCHE_FORMAT_STRING_MASTERY}

### Alternative Solution: GDB Bypass

1. Run the binary in GDB:
   ```bash
   gdb -q ./juche_loyalty_test
   break take_loyalty_test
   run
   ```

2. Answer Question 1 correctly: `38`

3. For Question 2, enter any text (it will fail due to whitespace truncation)

4. Before the strcmp result is checked, patch the return value:
   ```gdb
   break strcmp
   continue
   # After breakpoint hits
   finish  # Run until strcmp returns
   set $eax = 0  # Force success (0 = equal)
   continue
   ```

5. The program will execute `grant_party_membership()` and print the flag.

### Core Idea

The program will ask two question, the second one are impossible to answer correctly
-> The actual solution is through the RE-EDUCATION program which use printf() of the user input lead to format string vulnerability. Exploit this can leak the flag out.

## What I Learned

- **Format string vulnerabilities:** How `printf(user_input)` allows memory leaks
- **Stack memory layout:** How to identify where input appears on the stack
- **Static analysis power:** Flags are often visible in Ghidra/strings without exploitation
- **When to stop:** Knowing when static analysis is enough vs. when to exploit

---

## Lessons for Next Time

1. **Always check static analysis first** - Run `strings` and search Ghidra before exploiting
2. **Format string exploitation is advanced** - Requires understanding memory layout deeply
3. **Not every vulnerability needs exploitation** - Sometimes the flag is just visible
4. **Document failed attempts** - They show thought process and learning
  
### Commands Used

```bash
file juche_loyalty_test
strings -n 4 juche_loyalty_test | egrep -i 'key|password|flag|answer'
ghidra
gdb -q ./[filename]
break [function_name/address]
run
si
fin

```

### Scripts or Verification Code

not figured out yet

### Additional Notes

Learn about Binary Exploitation and reattempt

---

## Updates (Optional)

