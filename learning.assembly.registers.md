---
id: kevzvdlb6a80mr0lq2rki5c
title: Registers
desc: ''
updated: 1772941138525
created: 1772941127350
---

## Registers

### General Purpose
most commonly used registers for general computations, data manipulation, and memory addressing.

64 | 32 | 16 | Upper 8 | Lower 8 | Type
--- | --- | --- | --- | --- | --- |
RAX | EAX | AX | AH | AL | Accumulator (Arithmetic, Return Values)
RBX | EBX | BX | BH | BL | Base (Memory Addressing)
RCX | ECX | CX | CH | CL | Counter (Loops, Shifts)
RDX | EDX | DX | DH | DL | Data (I/O, Multiplication/Division)
RSI | ESI | SI ||| Source Index (String Operations)
RDI | EDI | DI ||| Destination Index (String Operations)
RBP | EBP | BP ||| Base Pointer (Stack Frames)
RSP | ESP | SP ||| Stack Pointer (Stack Management)
R8 | R8D | R8W || R8B | General Purpose
R9 | R9D | R9W || R9B | General Purpose
R10 | R10D | R10W || R10B | General Purpose
R11 | R11D | R11W || R11B | General Purpose
R12 | R12D | R12W || R12B | General Purpose
R13 | R13D | R13W || R13B | General Purpose
R14 | R14D | R14W || R14B | General Purpose
R15 | R15D | R15W || R15B | General Purpose


##### Why is the upper or lower 8 sometimes empty?
The upper 8-bit registers (like AH, BH, CH, DH) only exist for the original four general-purpose registers (AX, BX, CX, DX). The other registers (SI, DI, BP, SP, and R8-R15) were introduced later and do not have corresponding upper 8-bit parts. They have full 64-bit, 32-bit, and 16-bit versions, and the new registers have lower 8-bit versions as well.

This is simply a consequence of the historical evolution of the x86 architecture. It's a bit of an inconsistency, but it's something you just have to remember when working with assembly language.

The CPU's circuitry is designed to directly address and manipulate these sub-parts of the register. It doesn't fetch the entire 16-bit value of AX when you only need AH or AL. It simply accesses the relevant 8 bits.

It's unlikely to need access to sub elements of a register. If this kind of optimization for size is ever needed, look into partial register writes.


### Segment Registers

Register | 16-bit Real Mode | 32-bit Protected Mode | 64-bit Mode
--- | --- | --- | --
CS | Code Segment: Used to calculate the physical address of instructions (CS * 16 + IP). | Code Segment: Acts as a selector to index into a descriptor table (GDT or LDT). The descriptor contains the actual base address, limit, and attributes. | Code Segment: Primarily used for code segment attributes (like privilege level). The base address is usually 0, contributing to a flat memory model. Still a 16-bit selector.
DS | Data Segment: Used to calculate the physical address of data (DS * 16 + offset). | Data Segment: Acts as a selector to index into a descriptor table. The descriptor contains the actual base address, limit, and attributes. | Data Segment: Generally unused. The base address is effectively 0, contributing to a flat memory model.
SS | Stack Segment: Used to calculate the physical address of the stack (SS * 16 + SP). | Stack Segment: Acts as a selector to index into a descriptor table. The descriptor contains the actual base address, limit, and attributes. | Stack Segment: Still used for stack operations, but the base address is effectively 0 in most cases, contributing to a flat memory model.
ES | Extra Segment: Used for extra data segments (e.g., in string operations). Used to calculate the physical address of data (ES * 16 + offset). | Extra Segment: Acts as a selector to index into a descriptor table. The descriptor contains the actual base address, limit, and attributes. | Extra Segment: Generally unused. The base address is effectively 0.
FS | (Not present in early x86) | Available. Acts as a selector to index into a descriptor table. | Used for thread-local storage (TLS) or other system-specific data. The base address is usually not 0, and is managed by the OS.
GS | (Not present in early x86) | Available. Acts as a selector to index into a descriptor table. | Used for thread-local storage (TLS) or other system-specific data. The base address is usually not 0, and is managed by the OS.
Memory Model | Segmented memory model: Physical addresses are calculated using segment registers and offsets. Limited to 1MB of addressable memory without memory management techniques. | Protected memory model: Segment registers act as selectors, indexing into descriptor tables. Provides memory protection and allows for addressing up to 4GB (in 32-bit mode). | Flat memory model: Segmentation is largely disabled. All memory is accessible through a single linear address space (64-bit addresses). FS and GS are exceptions and do not have a base of 0.

FS and GS are notable exceptions in 64-bit mode, as they are used for thread-local storage and thus do not have a base of 0.
 
- 16-bit Real Mode: In the original 16-bit x86 architecture (real mode), segment registers (CS, DS, SS, ES) were crucial for memory addressing. They were used to create a 20-bit physical address by shifting the segment register value left by 4 bits and adding it to an offset. This allowed the CPU to address 1MB of memory.

- 32-bit Protected Mode: In 32-bit protected mode, the segment registers became selectors. They no longer directly contributed to the physical address calculation. Instead, they indexed into descriptor tables (Global Descriptor Table (GDT) or Local Descriptor Table (LDT)). The descriptor table entries contained the actual base address, limit, and access rights of the segment. This allowed for memory protection and addressing up to 4GB of memory.

- 64-bit Mode: In 64-bit mode, the segmentation model is largely simplified. The base addresses of the data segments (DS, ES, SS) are effectively forced to 0. This creates a flat memory model, where all memory is accessible through a single linear address space.

While the segmentation mechanism is simplified, the segment registers themselves are still 16-bit registers. However, when used in 64-bit mode, they are treated differently:
When you load a selector into a segment register in 64-bit mode (usinginstructions like mov ds, ax), the processor implicitly zero-extendsthe 16-bit selector to 64 bits before using it to index a descriptortable (if necessary). However, direct access to the segment base is notpossible in the same way as in 16-bit or 32-bit modes.

### Flags Register (RFLAGS/EFLAGS)

| Flag | Bit | Name | Description | Affected By | Used By|
----|----- | --- | --- | --- | ---
| CF | 0 | Carry Flag | Set if an arithmetic operation generates a carry-out of the most significant bit (for unsigned arithmetic) or a borrow (for subtraction).| Arithmetic instructions (ADD, SUB, MUL, DIV, etc.), shift and rotate instructions.| Conditional jumps (JC, JNC), multi-precision arithmetic.
| PF | 2 | Parity Flag| Set if the low-order byte of the result of an operation contains an even number of 1 bits (even parity). | Arithmetic and logical instructions.| Rarely used in modern programming. |
| AF | 4 | Auxiliary Carry Flag | Set if there is a carry from bit 3 to bit 4 (or a borrow from bit 4) during an arithmetic operation. Used for Binary Coded Decimal (BCD) arithmetic.| Arithmetic instructions.| Used in BCD arithmetic (less common now).|
| ZF | 6 | Zero Flag| Set if the result of an operation is zero.| Arithmetic, logical, comparison instructions. | Conditional jumps (JE, JZ, JNE, JNZ). |
| SF | 7 | Sign Flag| Set if the most significant bit of the result of an operation is 1 (indicating a negative result in signed arithmetic).| Arithmetic instructions.| Conditional jumps (JS, JNS). |
| OF | 11| Overflow Flag| Set if a signed arithmetic operation results in an overflow (i.e., the result is too large to be represented in the destination operand). | Signed arithmetic instructions.| Conditional jumps (JO, JNO). |
| TF | 8 | Trap Flag| When set, the processor generates a single-step interrupt after each instruction is executed. Used for debugging.| Set and cleared by software (debuggers). | Debugging (single-stepping). |
| IF | 9 | Interrupt Enable Flag | Controls whether the processor responds to maskable hardware interrupts. | Set and cleared by software (CLI, STI instructions). Affected by interrupt handling.| Interrupt handling.|
| DF | 10| Direction Flag | Controls the direction of string operations (using SI and DI registers). If set, string operations decrement SI and DI (processing strings from high to low addresses). If cleared, they increment SI and DI (processing strings from low to high addresses).| STD (set direction flag), CLD (clear direction flag) instructions. | String operations (MOVS, CMPS, SCAS, LODS, STOS). |
| IOPL | 12-13| I/O Privilege Level| Indicates the I/O privilege level of the currently running code. Used for controlling access to I/O ports in protected mode.| Set by privileged software (operating system).| I/O port access control. |
| NT | 14| Nested Task Flag | Controls nested task execution in protected mode (used for multitasking). | Used in task switching.| Task management (multitasking).|
| RF | 16| Resume Flag| Used in debugging and interrupt handling.| Used by debuggers and interrupt handlers. | Debugging and interrupt handling. |
| VM | 17| Virtual-8086 Mode Flag | When set, the processor operates in virtual-8086 mode, which emulates the 8086 real mode environment within protected mode.| Set by privileged software (operating system).| Running 16-bit real mode programs in a protected environment.|
| AC | 18| Alignment Check Flag | Enables alignment checking of memory accesses. If set, an exception is generated if a memory access is not properly aligned (e.g., accessing a 4-byte value at an odd address). | Set by software.| Detecting misaligned memory accesses.|
| VIF| 19| Virtual Interrupt Flag | A virtual image of the IF flag. Used in virtual-8086 mode. | Used in virtual-8086 mode.| Virtual-8086 mode interrupt handling. |
| VIP| 20| Virtual Interrupt Pending Flag | Indicates that a virtual interrupt is pending. Used in virtual-8086 mode. | Used in virtual-8086 mode.| Virtual-8086 mode interrupt handling. |
| ID | 21| ID Flag| Indicates support for the CPUID instruction.| Set by the processor at startup.| Detecting CPU features.

If you're working with the common arithmetic and control flags (CF, PF, AF, ZF, SF, OF, TF, IF, DF), you can generally think of EFLAGS and RFLAGS as being equivalent. The important thing to remember is that RFLAGS is the 64-bit extension, and it contains some additional flags in its upper 32 bits.  
When you're writing code that needs to work in both 32-bit and 64-bit modes, you generally refer to the lower 32 bits, and the same flag names and bit positions apply.