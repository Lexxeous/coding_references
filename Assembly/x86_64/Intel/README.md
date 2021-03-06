# <img src="../../../.pics/Lexxeous/lexx_headshot_clear.png" width="90px"/> Lexxeous's Intel x86_64 README: <img src="../../../.pics/Assembly/x86_64/Intel/intel_nasm_logo.png" width="130"/>

> The **Intel** syntax was originally used in the documentation of the Intel processor and is the dialect primarily used in Windows operating systems.

### External References:

1. [Unofficial Instruction Documentation](https://www.felixcloutier.com/x86/index.html)
2. [System Calls Table](https://blog.rchapman.org/posts/Linux_System_Call_Table_for_x86_64/)
3. [System Calls Guide](https://blog.packagecloud.io/eng/2016/04/05/the-definitive-guide-to-linux-system-calls/#syscallsysret)

### General:

> The flag `-lm` refers to the `C` math library because the math functions are not part of the `C` standard library.

```bash
$ nasm -f elf64 <filename>.asm # compile assembly into 64-bit binary/object file
$ gcc -static -o <filename> <filename>.o -lm # link the binary/object file into an executable
$ ./<filename> <args> # run the executable with a set of arguments
```

> The default syntax for the `GCC` compiler is **AT&T**, but **Intel** syntax is also supported using:

```bash
$ gcc -masm=Intel -o <exec_name> <prog_name>.c
```

**Intel** chips are little-endian.

Using `global <func_name>` exports the contents of the file/function to be used in other compiled/linked programs.

`ld` is the default assembly linker if you are not using the `C` runtime. You can then replace the global `main` with `_start` for **Linux** or `start` for **Windows**. So compiling and linking a pure assembly program without the `C` runtime would look somthing like this: `nasm -f elf64 <file_name>.asm && ld -o <file_name> <file_name>.o`.

Add instructions manipulate the carry flag, but increment instructions do NOT manipulate the carry flag!!

### Instructions:

> The source for an instruction is right hand side, while the destination is the left hand side: `<instruction> <lhs>, <rhs>` --> `<instruction> <destination>, <source>`

`test <reg_name>, <reg_name>` is a bitwise `AND` operation that will only return `0` if `<reg_name>` already contains `0`.

`xchg <reg_name> <reg_name>` is a `nop`

`ret` pops a value off the top of the stack (a call's return address) and sets the instruction pointer (`rip`) to this value. It transfers control to the return address located on the stack. **DO NOT USE** `ret` in the middle of a program, especially if you have not cleaned up the stack!

`leave` puts the base pointer (`ebp`/`rbp`) back on the stack (`esp`/`rsp`).

`lea` stands for "Load Effective Address". This command is capable of doing quick addressing calculations and assigning the value to a register like: `lea <reg_name>, [<base> + <idx>*<scale> + displacement]`. Although assembly's bracket notation `[]` typically implies the dereference of a pointer (grabbing the data at a location, rather than the address), `lea` is an exception. 

`syscall` takes the value in `rax` to determine what system call to use. It reaches down into the kernel of the operating system (ring 0) to use a system function. It also clobbers `rcx` by saving the current value of the instruction pointer (`rip`) into it.

`hlt` is used to terminate an assembly program when it is not linked with the `C` runtime.

#### Move Instructions:

`mov` is effectively a copy operation like: `mov LHS, RHS`. The RHS gets copied into the LHS, while the RHS retains its original value.

`movsd` stands for "Move Static Double" and is used for moving values that are larger than 64 bits (80 bits, 128 bits (`xmm<n>`), etc.) into 64-bit registers.

`movzx` is a zero extended move. The destination is always a register and the source can be a register or memory. This instruction always zeros out the upper N bits when using the standard `mov` instruction will not (for 16-bit and 8-bit moves).

`movsx` is a sign extended move. The destination is always a register and the source can be a register or memory. Example: moving the byte `0xef` (-17) to a double word register will be extended to `0xffffffef`, which is still -17, in decimal.

`movsxd` is a special instruction that will move a double word to a quadword with sign extension.

#### String Instructions:

The string instructions will move, compare, scan, load, store, output, or input a byte (`b`), word (`word`), double-word (`d`), or a quad-word (`q`) from a source (SRC) to a destination (DEST), while automatically incrementing/decrementing `rsi` and/or `rdi`.

| Instruction(s)          | SRC –> DEST             | INC/DEC        | Changes `ZF`   | Name (Action)     |
|-------------------------|:-----------------------:|----------------|:--------------:|-------------------|
| movsb/movsw/movsd/movsq | [ds:rsi] –> [es:rdi]    | ± (rsi && rdi) | N              | Move (Copy)       |
| cmpsb/cmpsw/cmpsd/cmpsq | [ds:rsi] –> [es:rdi]    | ± (rsi && rdi) | Y              | Compare (Compare) |
| scasb/scasw/scasd/scasq | [es:rdi] –> rax         | ± rdi          | Y              | Scan (Compare)    |
| lodsb/lodsw/lodsd/lodsq | [ds:rsi] –> rax         | ± rsi          | N              | Load (Move)       |
| stosb/stosw/stosd/stosq | rax –> [es:rdi]         | ± rdi          | N              | Store (Move)      |
| outsb/outsw/outsd/outsq |                         |                | N              |                   |
| insb/insw/insd/insq     |                         |                | N              |                   |

> **IMPORTANT**: THE STRING INSTRUCTIONS ALWAYS INCREMENT/DECREMENT THE VALUE IN `rcx`.

The increment and decrement value (1, 2, 4, 8) of `rsi` and/or `rdi` is based on the instruction width (`b`, `w`, `d`, & `q` respectively). The direction of the increment and decrement is based on the direction flag (`DI`).

If `DI` is clear (0), then the string instructions will increment (clear `DI` with the `cld` instruction).
If `DI` is set (1), then the string instructions will decrement (set `DI` with the `std` instruction).

> In most cases, you can assume that the direction flag will be cleared, but you should always check, just in case, before using it, to avoid bugs. These string instructions are used fairly often in real executables.

##### String Instruction Prefixes:

These repeat instructions work similarly to the loop instructions but are primarily used to repeat the action of a string instruction.

  * `rep` (terminates when `rcx` is 0)
  * `repe` (terminates when `rcx` is 0 or when `ZF` is 0)
  * `repne` (terminates when `rcx` is 0 or when `ZF` is 1)
  * `repz` (terminates when `rcx` is 0 or when `ZF` is 0)
  * `repnz` (terminates when `rcx` is 0 or when `ZF` is 1)

#### Loop Instructions:

> **WARNING**: Some jump instructions that decrement register values will decrement on the last iteration before falling through. This may result in a value being different than you expect (off by one).

  * `loop <label>` (automatically decrements the value in register `rcx`; does not set any flags)
  * `loope <label>` (loop terminates when the zero flag (`ZF`) is 0)
  * `loopne <label>` (loop terminates when the zero flag (`ZF`) is not 0)
  * `loopz <label>` (loop terminates when `rcx` is 0 ; automatically decrements the value in register `rcx`)
  * `loopnz <label>` (loop terminates when `rcx` is not 0 ; automatically decrements the value in register `rcx`)

#### Jump Instructions:

> These 3 jump instructions do not check the zero flag (`ZF`), but instead check the contents of the counter register `rcx`, `ecx`, and `cx` respectively and compares it against 0. These instructions are found very rarely in real executables.

  * `jrcxz <label>` (jumps if `rcx` is 0)
  * `jecxz <label>` (jumps if `ecx` is 0)
  * `jcxz <label>` (jumps if `cx` is 0)

### External Functions:

`strtod` uses `rdi` and `rsi` as default arguments, and the result is stored in `xmm0`.

`sqrt` uses what is in register `xmm0` and stores the result in `xmm0`. You must store the original value in a secondary register (or on the stack) if you don't want to lose/overwrite it.

`printf`'s first argument by default is `rdi`. It also needs to know how many formatted items (`%f`, `%d`, `%i`, `%c`, etc...) to print, which is retrieved from `eax`. `printf` also clobbers `eax`. `printf` uses the `xmm<n>` registers as the data to print in order from `xmm0` to `xmm15`.

`puts`, by default, will write what is in `rdi`.

### Sections:

Using `section .text` denotes a section that is meant as executable code. It gets accumulated into the **code segment (CS)** of the resulting executable.

Using `section .data` denotes a section that is meant as non-executable code, but rather data that is to be used for convenience within the program. It gets accumuleted into the **data segment (DS)** of the resulting executable.

Using `section .bcc` denotes a section that is meant for modifiable data. The **Block Started by Symbol (BSS)** area is of fixed size capable of storing modifiable variables. This section is adjacent to the data segment (DS).

You can also apply properties to sections to alter the way that they behave:
  * `nowrite` appended to a `.data` section will not allow the section of memory to be written to (read-only)
  * `align=<num>` appended to a `.data` section will force the section to be aligned on a `<num>`-byte boundary

You can also use the `$` character to refer to the current memory address inline. This allows the use of small computations within a `.data` section to calculate, for example, the exact length of a string that has been statically defined. Here, the local target `.len` is being used on each of the message targets (`msg1` & `msg2`) and being set equal (`equ`) to the difference between the current memory address and the start of the immediate parent message (`$-msg<n>`).

```asm
	section .data nowrite align=16
msg1: db "string one",10
.len: equ $-msg1 ; will set msg1.len = 11
msg2: db "string2",10
.len: equ $-msg2 ; will set msg2.len = 8
```

### Registers:

> The parameter passing sequence for user-level applications when using assembly is: (`rdi`, `rsi`, `rdx`, `rcx`, `r8`, & `r9`). This means that for functions that follow the calling conventions, the first `p` parameters will be passed to these registers, in order, where `p ≤ 6`.

#### x86 16-Bit Registers:

| Name                | Abbrv. |  Aliasing  |
|:--------------------|:------:|:-----------|
| Accumulator         |   AX   | AX = AH.AL |
| Base                |   BX   | BX = BH.BL |
| Counter             |   CX   | CX = CH.CL |
| Data                |   DX   | DX = DH.DL |
| Base Pointer        |   BP   |            |
| Source Index        |   SI   |            |
| Destination Index   |   DI   |            |
| Stack Pointer       |   SP   |            |
| Instruction Pointer |   IP   |            |

#### x86 32-Bit Registers (Extended):

| Name                | Abbrv. |  Aliasing          |
|:--------------------|:------:|:-------------------|
| Accumulator         |  EAX   | EAX = (16 bits).AX |
| Base                |  EBX   | EBX = (16 bits).BX |
| Counter             |  ECX   | ECX = (16 bits).CX |
| Data                |  EDX   | EDX = (16 bits).DX |
| Base Pointer        |  EBP   |                    |
| Source Index        |  ESI   |                    |
| Destination Index   |  EDI   |                    |
| Stack Pointer       |  ESP   |                    |
| Instruction Pointer |  EIP   |                    |

#### x86_64 64-Bit Registers:

> **GPR** stands for "General Purpose Register".

| Name                           | Abbrv. (64 bits) | Aliasing (Low 32 bits) | Aliasing (Low 16 bits) | Aliasing (Low 8 bits)  |
|:-------------------------------|:----------------:|:-----------------------|:-----------------------|:-----------------------|
| Accumulator                    |  RAX (R0)        | RAX = (32 bits).EAX    | RAX = (48 bits).AX     | RAX = (56 bits).AL     |
| Base                           |  RBX (R1)        | RBX = (32 bits).EBX    | RBX = (48 bits).BX     | RBX = (56 bits).BL     |
| Counter                        |  RCX (R2)        | RCX = (32 bits).ECX    | RCX = (48 bits).CX     | RCX = (56 bits).CL     |
| Data                           |  RDX (R3)        | RDX = (32 bits).EDX    | RDX = (48 bits).DX     | RDX = (56 bits).DL     |
| Base Pointer                   |  RBP (R4)        | RBP = (32 bits).EBP    | RBP = (48 bits).BP     | RBP = (56 bits).BPL    |
| Source Index                   |  RSI (R5)        | RSI = (32 bits).ESI    | RSI = (48 bits).SI     | RSI = (56 bits).SIL    |
| Destination Index              |  RDI (R6)        | RDI = (32 bits).EDI    | RDI = (48 bits).DI     | RDI = (56 bits).DIL    |
| Stack Pointer                  |  RSP (R7)        | RSP = (32 bits).ESP    | RSP = (48 bits).SP     | RSP = (56 bits).SPL    |
| GPR                            |  R8              | R8 = (32 bits).R8D     | R8 = (48 bits).R8W     | R8 = (56 bits).R8B     |
| GPR                            |  R9              | R9 = (32 bits).R9D     | R9 = (48 bits).R9W     | R9 = (56 bits).R9B     |
| GPR                            |  R10             | R10 = (32 bits).R10D   | R10 = (48 bits).R10W   | R10 = (56 bits).R10B   |
| GPR                            |  R11             | R11 = (32 bits).R11D   | R11 = (48 bits).R11W   | R11 = (56 bits).R11B   |
| GPR                            |  R12             | R12 = (32 bits).R12D   | R12 = (48 bits).R12W   | R12 = (56 bits).R12B   |
| GPR                            |  R13             | R13 = (32 bits).R13D   | R13 = (48 bits).R13W   | R13 = (56 bits).R13B   |
| GPR                            |  R14             | R14 = (32 bits).R14D   | R14 = (48 bits).R14W   | R14 = (56 bits).R14B   |
| GPR                            |  R15             | R15 = (32 bits).R15D   | R15 = (48 bits).R15W   | R15 = (56 bits).R15B   |
| Instruction Pointer            |  RIP             |                        |                        |                        |

> **Setting the lower 32 bits will clear the upper 32 bits. Setting the low 16 or 8 bits will NOT clear the remaining 48 or 56 upper bits. There are also no aliases that are functionally similar to the `AH`, `BH`, `CH`, & `DH` registers to manipulate the high 8 bits of a 16-bit alias for any other registers.**

<img src="../../../.pics/Assembly/x86_64/Intel/registers.png" width="600px"/>

#### x86_64 Segment Registers:

| Name            | Abbrv. |   Version  |
|:----------------|:------:|:-----------|
| Code Segment    |   CS   |   80286+   |
| Data Segment    |   DS   |   80286+   |
| Stack Segment   |   SS   |   80286+   |
| Extra Segment   |   ES   |   80286+   |
| General Segment |   FS   |   80386+   |
| General Segment |   GS   |   80386+   |

> There is only a subset of registers that are preserved through external function calls (`rbp`, `rbx`, `r12`, `r13`, `r14`, & `r15`). **All other registers value are not guarunteed to stay the same through calls to external functions. All other register values can potentially be clobbered.** To avoid data corruption, you must learn what registers are clobbered for certain external function calls.

`eax` (`rax`) is the default register that will be the return value for a function call.

`edi` (`rdi`), by default, contains the number of arguments passed into the function (`argc`).

`esi` (`rsi`), by default, contains a memory address pointer that points to the beginning of the argument array (`argv[]`). Effectively, is an array of `char*` datatypes, meaning that it is of type `char**`.

> **DO NOT** use `esp` as the stack pointer when doing 64-bit address computations! The register `esp` is a lingering by-product of working with 32-bit computers. If you use `esp`, it will point to an incorrect address because the upper 32 bits are not being taken into consideration.

### Addressing:

<img src="../../../.pics/Assembly/x86_64/Intel/addressing.png" width="450px"/>

> An address in memory can be accessed using the following format: `[<base> + <index>*<scale> + displacement]`. Where `<base>` & `<index>` are registers, `<displacement>` is an integer, and `<scale>` is 1, 2, 4, or 8; `<scale>` needs to match the size of the data types your are skipping over in terms of bytes (4 for 32-bit unsigned integers, 1 for characters, etc).

> **GPR `<base>` and `<index>` sizes MUST be the same (16-bit register each, 32-bit register each, or 64-bit register each)!**

### Flags:

#### x86 16-Bit FLAGS:

| Bit#  |  Mask  | Abbrv. | Description                                  | Category |  =1                   |    =0                   |
|:-----:|:------:|:------:|:---------------------------------------------|:--------:|:----------------------|:------------------------|
|   0   | 0x0001 |   CF   | Carry                                        |  Status  | Carry (CY)            | No Carry (NC)           |
|   1   | 0x0002 |        | Reserved, always 1 in EFLAGS                 |          |                       |                         |
|   2   | 0x0004 |   PF   | Parity                                       |  Status  | Parity Even (PE)      | Parity Odd (PO)         |
|   3   | 0x0008 |        | Reserved                                     |          |                       |                         |
|   4   | 0x0010 |   AF   | Adjust                                       |  Status  | Auxiliary Carry (AC)  | No Auxiliary Carry (NA) |
|   5   | 0x0020 |        | Reserved                                     |          |                       |                         |
|   6   | 0x0040 |   ZF   | Zero                                         |  Status  | Zero (ZR)             | Not Zero (NZ)           |
|   7   | 0x0080 |   SF   | Sign                                         |  Status  | Negative (NG)         | Positive (PL)           |
|   8   | 0x0100 |   TF   | Trap                                         |  Control |                       |                         |
|   9   | 0x0200 |   IF   | Interrupt Enable                             |  Control | Enable Interrupt (EI) | Disable Interrupt (DI)  |
|   10  | 0x0400 |   DF   | Direction                                    |  Control | Down/Decrement (DN)   | Up/Increment (UP)       |
|   11  | 0x0800 |   OF   | Overflow                                     |  Status  |                       |                         |
| 12~13 | 0x3000 |  IOPL  | I/O privilege level (80286+), always 1 prior |  System  |                       |                         |
|   14  | 0x4000 |   NT   | Nested Task (80286+), always 1 prior         |  System  |                       |                         |
|   15  | 0x8000 |        | Reserved (1 on 8086 & 80186), (0 on 80286+)  |          |                       |                         |

`pushf` pushes the 16 bits of FLAGS to the top of the stack. </br>
`popf` loads 16 bits from the top of the stack into FLAGS. </br>

`lahf` moves bits 0 through 7 of FLAGS into AH. </br>
`sahf` moves AH into bits 0 through 7 of FLAGS. </br>

##### The Trap Flag (TF):

The trap flag is important for debuggers and malware creators alike. When the trap flag is set, there is a `SIGTRAP` interrupt signal that fires after every executed instruction. This functionality is used by debuggers like `GDB` to step through code, one line at a time. Therefore, `if(TF == 1)`, then the program is most likely being debugged.

Malware creators attempt to take advantage of this fact by confuscating thier own code, reading the status of the trap flag in many unique ways. If the malware successfully detects the use of the trap flag, it can jump to the part of the code that does good and useful things. Otherwise, if the malware is being run outside of a debugger, it can detect that the trap flag is clear and may run some harmful code, like mass file encryption for ransomware. The goal of the malware creator is to trick the user/developer into thinking that there is nothing wrong with the code when they debug it, leaving them vulnerable to the true intentions of the malware creator when the program is run outside of a debugger. The following is an example of how a malware creator may inject a block of code that reads the status of the trap flag in a confusing way:

<img src="../../../.pics/Assembly/x86_64/Intel/trap_flag_example.png" width="900px"/>

#### x86 32-Bit EFLAGS:

| Bit#  |  Mask       | Abbrv. | Description                          | Category |
|:-----:|:-----------:|:------:|:-------------------------------------|:--------:|
|   16  | 0x0001_0000 |   RF   | Resume (80386+)                      |  System  |
|   17  | 0x0002_0000 |   VM   | Virtual 8086 Mode (80386+)           |  System  |
|   18  | 0x0004_0000 |   AC   | Alignment Check (486SX+)             |  System  |
|   19  | 0x0008_0000 |  VIF   | Virtual Interrupt (Pentium+)         |  System  |
|   20  | 0x0010_0000 |  VIP   | Virtual Interrupt Pending (Pentium+) |  System  |	
|   21  | 0x0020_0000 |   ID   | Can use CPUID (Pentium+)             |  System  |
| 22~31 | 0xFFC0_0000 |        | Reserved                             |  System  |

`pushfd` pushes the 32 bits of EFLAGS to the top of the stack. </br>
`popfd` loads 32 bits from the top of the stack into EFLAGS. </br>

#### x86_64 64-Bit RFLAGS:

| Bit#  |  Mask                 | Abbrv. | Description                | Category |
|:-----:|:---------------------:|:------:|:---------------------------|:--------:|
| 32~63 | 0xFFFF_FFFF_0000_0000 |        | Reserved                   |          |

`pushfq` pushes the 64 bits of RFLAGS to the top of the stack. </br>
`popfq` loads 64 bits from the top of the stack into RFLAGS. </br>

### Mathmatical Instructions:

#### Addition:

#### Subtraction:

#### Multiplication:

#### Division:

  * `div` (unsigned division)
  * `idiv` (signed division)

<img src="../../../.pics/Assembly/x86_64/Intel/div_regs.png" width="700"/>

The dividend is the concatenation of two 64-bit registers `rdx` and `rax` (making a 128-bit value equal to `2^64•rdx + rax`), while the divisor is `rbx`.

> If the quotient is too large for the designated register (`rax`), then there is a divide error (`#DE`), which Linux will sometimes interpret as a *floating point exception (core dumped)*, even though there is no floating point arithmatic occuring.


### Command Line Arguments: `argv[ ]` & `argc`

<img src="../../../.pics/Assembly/x86_64/Intel/argv_argc.png" width="650"/>

> **The last column is slightly wrong, it should be `[[rsi+8*1]+n]` to access the first string argument, which in this case is "Hello world"**

### Stack Management:

In general, the boiler plate instructions for conserving the state of the stack, before and after the actions of a program are:

```asm
push rbp
mov rbp, rsp
;
; body
;
mov rsp, rbp ; this line and
pop rbp ; this line can be replaced with "leave"
ret
```

#### Reserving Specific Amounts of Space on the Stack

For the majority of assembly programs, a call to a function will push the return address (8 bytes) onto the stack and push `rbp` (8 bytes) onto the stack. If this is the case, then the stack will be aligned on a 16-byte boundary automatically. However, there will be problems if the stack is not aligned on a 16-byte boundary when a call to another function occurs that uses an `xmm<n>` register (it will cause a segmentation fault).

Fortunately, if a function call does not use an `xmm<n>` register, then you needn't worry about aligning the stack.

There are two main ways to properly allocate space on the stack (after knowing the minimum amount of space needed): <br>

1. Say you only need 24 bytes of space on the stack for a program (for storing three 8-byte register values). The value 24 is not a multiple of 16 and will not align the stack, in most cases. The easiest solution is to round up to the nearest multiple of 16 (32, in this case), and subtract that value from the stack pointer (`rsp`), like: `sub rsp, 32`.
2. A nearly identical solution that will reveal the intentional amount of space that is desired on the stack is to first move the stack pointer with: `sub rsp, 24`, then to zero-out the last nybble of the stack memory. Aligning the stack on a 16-byte boundary means that all the addresses that are aligned will end with the hexidecimal digit `0x0`. The way to achieve this, regardless of the stack pointer's location, is to `AND` the current value of the stack pointer with -16, like: `and rsp, -16`. Because 16 in binary is `0000_..._0001_0000`, the 2's complement representation (-16) is `1111_..._1111_0000`. Effectively, this instruction will leave the stack unchanged except for the last nybble (`AND`-ed with zeros), which will be cleared; this forces the stack pointer address to end with `0x0`.

> For these two options, the flags are also set a bit differently. The `AND` instruction leaves the `AF` flag undefined, clears the `CF` and `OF` flags, sets `PF` if the low byte of the result has odd parity, sets `SF` if the result is negative (viewed in two's complement), and sets `ZF` if the result is zero.

> The `sub` instruction evaluates the result for both signed and unsigned integer operands and sets the `OF` and `CF` flags to indicate any overflow in the signed or unsigned result, respectively. The `AF` flag is set based on whether the first nybble overflows. The `SF` flag indicates the sign of a signed result.

The image below provides an example of stack management with a program that seeks to store two 8-byte arguments and the register `rsi` on the stack. Implicitly, the return address for this call has been placed on top of the stack first. The base pointer register `rbp` is pushed, aligning the stack. When 16 is subtracted from the stack pointer (`rsp`), the stack remains aligned with space for the two arguments. When `rsi` is pushed to the stack, it becomes unaligned. Fortunatly, `atol` does not use any `xmm<n>` registers, so calling the function is inconsequential to the state of the stack (with regards to alignement). Once the program is almost finished, and has no more need for the stack, the stack pointer (`rsp`) is returned to the location of the base pointer (`rbp`) with: `mov rbp, rsp`. From this point, all there is left to do is pop the base pointer and return (`ret`); returning will pop the return address and place it in the instruction pointer register (`rip`).

<img src="../../../.pics/Assembly/x86_64/Intel/stack_vis.png" width="750"/>

> There is a security issue related to moving the stack pointer to the base pointer location, once the stack is no longer of use. If the stack values are not cleared, the stack becomes "dirty". These raw data values are left unattended and can cause data leakage. For most programs this won't be an issue, but for any important values that are stored on the stack, malicious users can look through the stack after a process, and steal valuable information. One common way to clear the stack is to implement a subroutine that will increment the address of the stack pointer (`rsp`) until it is equal to the address of the base pointer (`rbp`), clearing each byte along the way.

### Inline Assembly:

> The syntax for inline assembly mostly follows the same rules as the parent syntax (i.e., **Intel** or **AT&T**), and the compiler used, so check for compatibility before barging in. The following documentation will be for **Intel Assembly** and the `GCC` compiler.

Inline assembly code can be used with `GCC` and other compilers, as long as the language and the hardware system supports it. In some cases, programmers use inline assembly blocks to prohibit the compilers from making automatic optimizations to thier code (which could otherwise break some code that they wrote intentionally, e.g., for loop delay function with no actionable instructions nested in the middle).

> Keywords like `asm`, `typeof`, `volatile`, & `inline` may not available in programs compiled with `-ansi` or `-std` (although `inline` can be used in a program compiled with `-std=c99` or a later standard).
The way to solve these problems is to put a double underscore (`__`) at the beginning and end of each problematical keyword. For example, use `__asm__` instead of `asm`, and `__inline__` instead of inline.

Creating an inline assembly code block in `C` is fairly simple, but gets more complex when dealing with details:

```c
__asm__ (
  "assembly code" // all assembly code lines are strings and will end with a new line character ('\n')
  "comes first" // all sections are separated by a colon character (':')
  : outputs // the "outputs" and "inputs" sections connect values from a C program to registers
  : inputs
  : clobbers // the "clobbers" section identifies the registers that will be changed (clobbered)
)
```

> ***The syntax for connecting the outputs and inputs between the `C` program and the assembly can be very cryptic; you will need to follow constraints mentioned [here](https://gcc.gnu.org/onlinedocs/gcc/Using-Assembly-Language-with-C.html).***

As an example, below is a `C` code sample that includes inline assembly.

```c
// gcc -c -m32 -masm=intel main.c && gcc -o main main.o -m32 -masm=intel

#include <stdio.h>

int main(void) {
	int value = 17;
	int incr = 12;
	printf("value=%d and incr=%d\n", value, incr);

	__asm__(
		"mov eax, %[value]\n"
		"add eax, %[incr]\n"
		"mov %[value], eax\n" // instructions
		: [value] "=r"(value) // outputs <value> in eax
		: "0"(value), [incr] "r"(incr) // inputs <value> into eax, and <incr> into any general purpose register
		: "eax", "cc" // clobbers eax and condition FLAGS
	);

	printf("value=%d and incr=%d\n", value, incr);
	return 0;
}
```

### Using and Starting the `C` Runtime:

In `C` the `libc_start_main` function is very important for starting the `C` runtime. Because of its specific order of parameters and its use in conjuction with assembly, there is a fairly strict order of assembly instructions that will successfully start this function with the correct values. Below is the function definition for `__libc_start_main`, the order of parameters (based on the assembly calling convention), and a mapping to the list of instructions (and memory addresses) associated with the assembly necesary to properly use this function.

```c
// Function definition:             // ASM Convention:   // ASM Instruction (Address) Map:
int __libc_start_main (             // call              // 11 (0x67f8)
  int* (main)(int, char**, char**), // rdi               // 10 (0x67f1)
  int argc,                         // rsi               // 03 (0x67d9)
  char** ubp_av,                    // rdx               // 04 (0x67da)
  void (*init)(void),               // rcx               // 09 (0x67ea)
  void (*fini)(void),               // r8                // 08 (0x67e3)
  void (*rtld_fini)(void),          // r9                // 02 (0x67d6)
  void (*stack_end)                 // stack             // 07 (0x67e2)
);
```

> The instructions shown are from the disassembly of the `ls` instruction for **Unix**.

```txt
Initial Stack Contents:
    <–––– rbp
argc
argv[]
envp (for <ubp_av>)
    <–––– rsp
```

```asm
00 (0x67d0): endbr64                ; branch protection, nop
01 (0x67d0): xor ebp, ebp           ; clear ebp
02 (0x67d0): mov r9, rdx            ; addr of destructor function call for dynamic linker _dl_fini
03 (0x67d0): pop rsi                ; pops argc of the top of the stack into rsi
04 (0x67d0): mov rdx, rsp           ; moves rsp addr to rdx (for <ubp_av>)
05 (0x67d0): and rsp, -16           ; aligns stack on 16-byte boundary (-16 = 0xfffffffffffffff0)
06 (0x67d0): push rax               ; couldve pushed rsp twice
07 (0x67d0): push rsp               ; aligns stack on 16-byte boundary (for <*stack_end>)
08 (0x67d0): lea r8, [rip+0x10d66]  ; (for <*fini>)
09 (0x67d0): lea rcx, [rip+0x10cef]            ; (for <*rtld_fini>)
10 (0x67d0): lea rdi, [rip+0xffffffffffffe5f8] ; (for int* (main))
11 (0x67d0): call QWORD PTR [rip+0x1c7d2]      ; (for call to __libc_start_main)
12 (0x67d0): hlt                               ; (_start needs sys_exit)
```
> In general, these 12 lines of code can be copied almost exactly for any assembly program that wants to make a direct call to the `C` runtime. The main differences will only be in the immediate values given to the addressing computations on lines 8 through 11.

### Macros and Constant Definitions:

Constants can be defined in an assembly file by using the keyword `equ`. Constants will typically just at the top of the assembly file. Some example are as follows:
```asm
SYS_WRITE equ 1
RET_ERR equ 1
RET_SUCCESS equ 0
```

You can define single line macros with: `%define <macro_name>([arg]) <instruction>([arg])`. Optionally, variables can be passed in as arguments to the macro by surrounding the parameter(s) with parenthesis `([arg])`. Some examples are as follows:
```asm
%define stack0 QWORD [rbp - 8] ; will return the top 64-bit value on the stack
%define stack1 QWORD [rbp - 16] ; will return the second to top 64-bit value on the stack
%define argv(idx) QWORD [rsi+8*(idx)] ; will return the idx-th command line argument
```

You can also define multi-line macros by follwing a similar convention:
```asm
%macro <macro_name> <num_args>
	; macro body
%endmacro
```

Example (where `%00` refers to the immediately previous label, if there is one):
```asm
%macro err 2
	; Declare an error message and return an errno.
	%00: db %1
	.len equ $-%00
	.errno equ %2
%endmacro
```

Example Usage:
```asm
BADARG: err {'ERROR: Expect exactly two arguments.',10}, 1
```
Here, `%00` is `BADARG:`, `%1` is a newline terminated string, & `%2` is the `errno` equal to 1.

> The curly brackets `{}` protect any internal commas from separating argument values. Without them, the above example would read 3 input arguments instead of one, and throw an error for improper use of the `err` macro.

### Instruction Groups:

Instruction groups collect like instructions together based on the actions that they perform. An instruction can be part of more than one group at time. The most common groups are 1 ~ 7.

| Group Name              | Group ID | Description                                                |
|-------------------------|----------|----------------------------------------------------------- |
| X86_GRP_JUMP            | 1        | Jump target is a leader. End block.                        |
| X86_GRP_CALL            | 2        | Call target and next instruction are leaders. End block.   |
| X86_GRP_RET             | 3        | End block.                                                 |
| X86_GRP_INT             | 4        | Instruction after interrupt is a leader.                   |
| X86_GRP_IRET            | 5        | End block.                                                 |
| X86_GRP_PRIVILEGE       | 6        | Ignore.                                                    |
| X86_GRP_BRANCH_RELATIVE | 7        | Branch target and next instruction are leaders. End block. |
| X86_GRP_VM              | 128      | All virtualization instructions (VT-x + AMD-V).            |
| . . .                   | . . .    | . . .                                                      |
| X86_GRP_ENDING          | 170      |                                                            |

> For more information and/or to see the group definitions, visit the [Capstone GitHub Repository](https://github.com/aquynh/capstone/blob/master/include/capstone/x86.h#L1901).

### Jump Tables:

A common structure that can implemented in `C` is a *switch* statement, and it uses the following structure:

```c
switch(n) {
	case 0: f(); // execute function f() if n = 0
	case 1: g(); // execute function g() if n = 1
	case 2: h(); // execute function h() if n = 2
	...
	case N: N(); // execute function N() if n = N 
	default: j(); // execute function j() if n != [0..N]
}
```

In assembly, this structure can be implemented fairly easily by building a *jump table* with a list of targets like:

```asm
section .data
jumptab: dq target_f, target_g, target_h, ... , target_N

section .text
	; ...
	cmp rax, N ; rax should contain the input 'n'
	ja .def ; jump if above to the default target
	jmp [jumptab+rax*8] ; else jump to the jump table (list of targets, indexed by rax)
.def
	jmp target_j

target_f:
	; ...
	jmp switch_done

target_g:
	; ...
	jmp switch_done

target_h:
	; ...
	jmp switch_done

  ...

target_N:
	; ...
	jmp switch_done

target_j:
	; ...
	jmp switch_done

switch_done:
	; ...
```

### Using `objdump` and `readelf` (`gobjdump` for macOS):

Both `objdump` and `readelf` use the same general format in the terminal:

```bash
$ objdump [flags] <file_name>
$ readelf [flags] <file_name>
```

Useful `objdump` flags (use `man objdump` for more information):

  * `-d` or `--disassemble` (disassembles only sections that are expected to contain instructions)
  * `-F` or `--file-offsets` (shows the file offset values when disassembling sections)
  * `-f` or `--file-headers` (shows summary info from each parent header)
  * `-j` (shows only a user specified list of sections during disassebmly)
  * `-M` or `--disassembler-options` (allows disassembler configuration for syntax, architecture, etc.)
  * `-s` or `--full-contents` (shows full contents of any section(s) requested)
  * `-t` or `--syms` (shows the symbol table)

Useful `readelf` flags (use `man readelf` for more information):

  * `-f` or `--file-headers` (shows summary info from each parent header)
  * `-s` or `--full-contents` (shows full contents of any section(s) requested)
  * `-S` or `--source` (intermix source code with disassembly)
  * `-t` or `--syms` (shows the symbol table)

> If you want to use a tool that is similar to `readelf` for **macOS**, use `brew install binutils` and follow the resulting instructions about adding the tools directory to your PATH and setting environment variables. Then you can use `gobjdump` like:

```bash
$ temp_path=`which <exec_name>`
$ gobjdump [flags] $temp_path
```

For `<exec_name>` = `make` and `[flags]` = `-s`:

```bash
$ temp_path=`which make`
$ gobjdump -s $temp_path
```
```txt
/usr/bin/make:     file format mach-o-x86-64

Contents of section .text:
 100000f77 554889e5 8d47ff48 8d560848 8d3d2900  UH...G.H.V.H.=).
 100000f87 000031c9 89c6e800 000000             ..1........
Contents of section __TEXT.__stubs:
 100000f92 ff257800 0000                        .%x...
Contents of section __TEXT.__stub_helper:
 100000f98 4c8d1d69 00000041 53ff2559 00000090  L..i...AS.%Y....
 100000fa8 68000000 00e9e6ff ffff               h.........
Contents of section .cstring:
 100000fb2 6d616b65 00                          make.
```

### `PIC`, `PIE`, `ASLR`, `PLT`, `GOT`, and RIP-Relative Addressing:

#### Acronyms:

  * `PIC` (Position Independent Code)
  * `PIE` (Position Independent Executable)
  * `PLT` (Program Linkage Table)
  * `GOT` (Global Offset Table)
  * `ASLR` (Address Space Layout Randomization)
  * `WRT` (With Respect To)

#### Background:

The motivations for `PIC`, `PIE`, `ASLR`, `PLT`, `GOT`, and RIP-Relative Addressing are based on the responsibilties of the linker and loader. After code compilation, the program's object code is linked into a binary file (executable) and loaded into memory. It is the linker's job to figure out where to place the program's data (and external references to other libraries).

#### The `-no-pie` Flag:

Ideally, we want executable code to run the same no matter where it is placed in memory. Using the `-no-pie` flag during linking tells the linker that the executable code should not be position independent and that the location of all the program data will not be randomized. By default, the linker uses a defence mechanism called `ASLR` that will randomize the location of a program's data in memory every time it is ran. This tactic is a good, constant defence against malware; it is much more difficult for malware to attack certain applications if the data is being randomly placed in memory at the start of every process run.

#### The `-static` Flag:

The use of the static flag makes the difference between loading and running external library calls dynamically, during runtime, and including the entirety of an external library statically, during compilation and linking, for any particular executable. Ideally, we want shared libraries loaded in memory once so that all executables can access them, rather than unecessarily including entire libraries of binary data with the executable binary. The latter includes functions that will never be used and takes up exessive amounts of space in memory.

As an example, imagine an assembly program that has already been compiiled with `NASM` into an object file. While using `gcc` to compile and link the object file into an executable, with the `C` runtime and default libraries, we have an option to include the `-static` flag:

```bash
gcc -o copy copy.o ; ls -l copy # 16,640 bytes
gcc -o copy copy.o -static ; ls -l copy # 863,064 bytes
```

This example uses `copy.asm` found in the `copy/` directory. It is clear that using the `-static` flag drastically increases the size of executables. In this case, the executable is almost 52 times as large.

#### The `__GLOBAL_OFFSET_TABLE__` Section:

The `GOT` is the primary 32-bit solution to referencing external resources from a program. The `.got` section is included as part of the executable information that stores addresses for data and routines that were going to be used for any particular program, rather than having to know thier locations and reference them directly.

During assembly, the `.got` section is "fixed up" with all of the absolute addresses to external resrouces needed for the program at hand. It is possible to locate the `GOT` using `objdump -s -j .got <path_to_exe>`. Although it is possible to locate the `.got` section for a particular executable and use it, this is not the primary solution for the 64-bit realm.

#### RIP-Relative Addressing:

For the 64-bit world, at load time, addresses that exist in assembly code are "converted" by the linker and replaced relative to the instruction pointer (`rip`). As a result, this creates position/location independent code (`PIC`).

> Unfortunately, this solution cannot handle function calls unless the files are linked with the static flag. The `PLT` solves this problem but its more complex to use.

You can see the conversion in practice if you dump the files during compilation and linking. Take an instruction that uses `lea` and a data resource called `str` that points to an address containing the bytes of a string:

```asm
lea rsi, [rel str]     ; | Original
lea rsi, [rip+0x0]     ; | After Assembler
lea rsi, [rip+0xbc465] ; V After Linking
```

With this 64-bit solution, there are also 3 useful keywords that are available:

  * `rel` (will assign relative addressing to a particular assembly pointer)
  * `abs` (will assign absolute addressing to a particular assembly pointer)
  * `default <rel/abs>` (will set the default addressing scheme for all assembly pointers in a file)

> The default keyword is similar to the concept of writing `using namespace std;` in a `C/C++` program. For assembly, using the `default` keyword will implicitly include `rel` or `abs` for all address pointers in an assembly file.

> If you are jumping to an address in the same section of an executable, you do not need to use either of the addressing keywords. The linker will already know the locations of addresses/resources in the currently loaded section. You will only need relative or absolute addressing for accessing things outside of the currently loaded section and/or outside the currently loaded program.

#### The `PLT` Section:

> The following description of the `PLT` is also documented as an answer to the *Stack Overflow* question: ["*What does @plt mean here?*"](https://stackoverflow.com/questions/5469274/what-does-plt-mean-here?answertab=votes#tab-top)

If you see something like `<func_name>@plt` during disassembly, it represents a way to get code fixups (adjusting addresses based on where code sits in virtual memory, which may be different across different processes) without having to maintain a separate copy of the code for each process. The `PLT` is the **program linkage table**, one of the structures which makes dynamic loading and linking easier to use.

The `<func_name>@plt` is actually a small stub which (eventually) calls the real `<func_name>` function, modifying things on the way to make subsequent calls faster.

The real `<func_name>` function may be mapped into any location in a given process (virtual address space) as may the code that is trying to call it.

So, in order to allow proper code sharing of calling code (left side below) and called code (right side below), you don't want to apply any fixups to the calling code directly since that will restrict where it can be located in other processes.

So the `PLT` is a smaller process-specific area at a reliably-calculated-at-runtime address that isn't shared between processes, so any given process is free to change it however it wants to, without adverse effects.

See the following diagram which shows both your code and the library code mapped to different virtual addresses in two different processes, proc_A and proc_B:

```txt
Address: 0x1234          0x9000      0x8888
        +-------------+ +---------+ +---------+
        |             | | Private | |         |
proc_A  |             | | PLT/GOT | |         |
        | Shared      | +---------+ | Shared  |
========| application |=============| library |====>
        | code        | +---------+ | code    |
        |             | | Private | |         |
proc_B  |             | | PLT/GOT | |         |
        +-------------+ +---------+ +---------+
Address: 0x2020          0x9000      0x6666
```

The original way in which code was shared meant it they had to be loaded at the same memory location in each virtual address space of every process that used it. Either that or it couldn't be shared, since the act of fixing up the single shared copy for one process would totally stuff up other processes where it was mapped to a different location.

By using position independent code, along with the `PLT` and a global offset table (`GOT`), the first call to a function `<func_name>@plt` (in the `PLT`) is a multi-stage operation, in which the following actions take place:

1. You call `<func_name>@plt` in the `PLT`.
2. It calls the `GOT` version (via a pointer) which initially points back to some set-up code in the `PLT`.
3. This set-up code loads the relevant shared library if not yet done, then modifies the `GOT` pointer so that subsequent calls directly to the real `<func_name>` rather than the `PLT` set-up code.
4. It then calls the loaded `<func_name>` code at the correct address for this process.

On subsequent calls, because the `GOT` pointer has been modified, the multi-stage approach is simplified:

1. You call `<func_name>@plt` in the `PLT`.
2. It calls the `GOT` version (via pointer), which now points to the real `<func_name>`.


### Liveness and Trace Tables:

A variable is *live* at a particular point in a program if its value at that point will be used in the future. Otherwise the variable is *dead*.

  * This means you have to know the future uses of the variable. Structuring pays off here.
  * This analysis is used for register allocation when compiling programs. Lots of variables, but only a few registers... multiple variables can use the same register if at most one variable is live at any given time.

Also, another thing to notice is that variables `a` & `b` are not used on the RHS of any one line of code at the same time (lines 5 & 7). Given this fact, we can use one register to hold the changing values of `a` & `b` through out the entire program.

<img src="../../../.pics/Assembly/x86_64/Intel/liveness_and_trace_tables.png" width="800"/>

Another way to think about allocation of registers is to create sets of live edge pairs for each of the variables based on where they are used in the code. So for this example, the live sets for `a`, `b`, & `c` would be the following:

> The variable `c` is not necessarily used in every line of code, but some register must hold onto its value for the entire program because its value must be returned at the very end. This implies that `c` will always be live.

```txt
a: {3->5, 7->8, 8->5} OR {3->5, 7->5}
b: {5->6, 6->7} OR {5->7}
c: Live for the whole program because of the return statement.
```

The sets of live edge pairs for variables `a` & `b` are *disjoint*, which means that they are never used at the same time and can take advantage of using the same register (and even one same variable), instead of two. This further solidifies the above point about `a` & `b` not being used on the RHS of any line of code at the same time.

> It can be useful to create a trace table that lines up directly with each row of code that potentially changes the value of a particular variable. Looking at the LHS and RHS of the associated lines of code will determine whether a particular variable is currently being modified or not.

### Program Slicing:

Program slicing is a method of simplifying a program to make analysis simpler by focusing on a particular aspect of program semantics called the slicing criterion.

Given a location `l` and a variable `v`, a slice is constructed with respect to (`l`, `v`) by deleting all statements irrelevant to the value of `v` at `l`. 

There are two directions for slicing:

  * **Backward** slicing consists of statements that have an *effect on* the slicing criterion.
  * **Forward** slicing consists of statements that are *affected by* the slicing criterion.

Within these two directions of slicing, there are 3 types of slicing:

  * **Static** slicing generalizes the problem across all possible inputs. This type of slicing simply removes any line(s) of code that are not relevant to the variable in question.
  * **Dynamic** slicing assumes that you know what the value(s) of the input(s) are going to be at runtime. In this way, you can slice/remove lines of code that will not run for the given input. In many cases, this type of slicing results in a smaller remaining set of code to analyze.
  * **Constrained** slicing instantiates rules/constraints on the possible inputs and attempts to remove sections of code that are not useful or needed, effectively optimizing the program and isolating the variable in question.

#### Slicing Assembly:

Given a set of assembly code, we can reduce the program down to its necessary parts after we have decided which variable we want to analyze (`v`) and to what line of code (`l`).

Assume that we want to know what the value of `v = rax` is at line `l = 7` for the following assembly code. We create a table of reads, writes, and dependencies for each line, start from the bottom, and replace any dependency that what written, with what was simultaneously read, per line of code, as follows:

| Line # | Code               | Read     | Written  | Depends       | Relevant |
|:------:|:-------------------|:---------|:---------|:--------------|:--------:|
| 0      | inc rax            | rax      | rax      | rsp, rax      | Y        |
| 1      | lea rcx, [rax * 8] | rax      | rcx      | rsp, rax      | N        |
| 2      | push rcx           | rsp, rcx | rsp, M   | rsp, rcx, rax | Y        |
| 3      | push rax           | rsp, rax | rsp, M   | rsp, rax      | Y        |
| 4      | mov rdi, 21        | –––      | rdi      | rsp, M        | N        |
| 5      | call _optc         | rax, rdi | rax, rcx | rsp, M        | N        |
| 6      | pop rcx            | rsp, M   | rsp, rcx | rsp, M        | Y        |
| 7      | pop rax            | rsp, M   | rsp, rax | rsp, M        | Y        |
|        |                    |          |          | rax           |          |

After the dependencies column is populated from the reads and writes columns, look for disjoint pairs between the dependencies and writes to determine the relevancy of each line of code (if the writes and dependencies for any row are disjoint, then the row is not needed and can be removed).

