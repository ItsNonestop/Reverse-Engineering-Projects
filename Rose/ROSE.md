# [Rose](https://crackmes.one/crackme/675410a360fa67152406b51a)
## Description:
A crackme with multiple "stages" starting at beginner level, and progressively gets harder.
x64dbg is recommended as i only tested it with that.
Patching IS allowed and required for some stages (even some password/key stages).
If the text is annoying/takes too long, patch out the print function.

Make sure to not change save.txt or delete it.
If you think it's too easy, only use static analysis/patching to crack.

---

## Tools:
- Ghidra

---

#### Creating Project
<img src="https://github.com/user-attachments/assets/80c82d8c-945d-4c77-9a82-569e50b31420" width="700"/>

#### Importing Rose.exe
<img src="https://github.com/user-attachments/assets/71a313a3-2169-4ec0-8116-e639d11460a1" width="700"/>

#### Import Results Summary
<img src="https://github.com/user-attachments/assets/57c6a3df-7137-4f06-958d-ad459e600bf7" width="700"/>

#### Analysis
<img src="https://github.com/user-attachments/assets/990b877a-53a1-4b90-9ec9-9324019f12ca" width="700"/>

---

### Starting Static Analysis
Started by unauthorised static analysis

#### Rose.exe
<img src="https://github.com/user-attachments/assets/36803b13-7c7e-40cb-a4cf-bc746ae78d34" width="700"/>

#### Findings
- Strings: Found multiple references to "Bouncer", "Patcher" and "Patching"
  - Patch the loop so it stops looping
  - Changed the sleep when printing a certain charcter is 0x10ms

- Imports: `IsDebuggerPresent` `VirtualProtect` `SetWindowsHookExA` suggests anti debugging and possbile hooking
<img src="https://github.com/user-attachments/assets/3070fc15-9c42-4226-9141-ff959620a2fb" width="700"/>

<img src="https://github.com/user-attachments/assets/5ddc8ebd-332f-43dd-bd49-1b2562fea7b9" width="700"/>

<img src="https://github.com/user-attachments/assets/89ba6012-f2c6-4a43-9c16-a4d46399d38c" width="700"/>

---

### Control Flow Analysis
The main entry point seems to be at function `FUN_14000ece0`
<img src="https://github.com/user-attachments/assets/876ee67a-c60b-493d-85a3-70a5ce56e61e" width="700"/>

- Main Function: `FUN_14000ece0`
- Print Function: `FUN_140001bd0` (assumingly by usage of strings)
<img src="https://github.com/user-attachments/assets/30ad532a-8848-4d55-8738-839e53b169b5" width="700"/>

---

### Stage 1: The Loop
`FUN_14000ece0` has a `while` loop that seemingly is designed to stall code execution
- Ghidra Address: `14000eea0`
- Logic: The code checks for a value and then sleeps for 100ms in a loop if the check fails:
`while (check_flag != 5) {
    Sleep(100);
}
`
- Solution: The check `CMP ..., 0x5` followed by `JNZ` (Jump if Not Zero) causes an infinate loop
- Patch: NOP out conditional jump (JNZ) instruction to bypass the check and fall through

---

## Stage 2: The Slow Print
When the program is ran it prints the text very slowly, pretty inconvient, the hint mentions a `Sleep` duration.
- Function `FUN_140001bd0` (Print Function)
- Logic: Iterates through string if the character is `'|'` (Pipe), it sleeps for `0x12C` (300ms)
`
if (char == '|') {
    Sleep(0x12c);
}
`
- Soltion: The instructions demand changing the sleep time to `0x10` (16ms)
- Patch: Locate the `MOV ECX, 0x12C` instruction in `FUN_140001bd0` and change `0x12C` to `0x10`

---
