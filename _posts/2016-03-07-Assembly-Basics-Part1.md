---
layout: post
category: Reverse-Engg
title: Assembly Language Basics - Part 1
date: 2016-03-07
---


In this post, we'll look at the x86 or IA32 register organization. The x86 family begins with the Intel 8086 processor, which was the first 16 bit processor. 
It had 8 general purpose registers named - AX, BX, CX, DX, SP, BP, SI, DI. As time progressed, the processor evolved and newer 32 bit generations emerged. These also had the same 
8 general purpose registers but an E was added to show that the registers had been extended from 16  bits to 32 bits. So the 32 bit registers were labelled EAX, EBX, ECX, EDX, ESI, EDI, ESP, EBP. 


Technology evolved, soon 64 bit processors were introduced. There were two types of 64 bit processors. Intel announced its Itanium 64 series which weren't backward compatible with the x86 family, these were unconventional 64 bit processors with branch prediction and pipelining features. AMD announced their AMD 64 series, these were backward compatible with the IA family. This 64 bit processor allowed you to run binaries  that were originally compiled for the 32 processor, and you did not have to change anything to make it work. Obviously, this made the amd64 processor more popular than the Itanium 64 series. The general purpose registers remained the same, but now had an R prefix to denote that these are 64 bit registers (RAX, RBX, RCX, RDX, RSP, RBP, RSI, RDI). Additionally, 64 bit processors also have 8 additional registers R8, R9, R10, R11, R12, R13, R14 and R15.

Even though the registers sizes changed - a 64 bit register can still be used as a 32 bit or 16 bit register. So when you see the register's name in an instruction - you should be able to identify whether it is an 8 bit, 16 bit, 32 bit or 64 bit register. The table below shows the registers and their names when they are being used to hold 8 bit, 16 bit, 32 bit and 64 bit operands.


|	bits | 32 bit processors	|  64 bit processors	|
|---------| --------------|----------------------|
|Byte registers	(ie - when addressing 8 bit operands) | AL, BL, CL, DL, AH, BH, CH, DH |	AL, BL, CL, DL, DIL, SIL, BPL, SPL, R8L to R15L
|Word registers	(ie - when addressing 16 bit operands) | AX, BX, CX, DX, SI, DI, SP, BP	| AX, BX, CX, DX, SI, DI, SP, BP, R8W to R15W
|Double Word registers (ie - when addressing 32 bit operands) | EAX, EBX, ECX, EDX, ESP, EBP, ESI, EDI |	EAX, EBX, ECX, EDX, ESP, EBP, ESI, EDI, R8D to R15D
|Quad word register (ie when addressing as 64 bit registers) | NA | RAX, RBX, RCX, RDX, RSI, RDI, RSP, RBP, R8 to R15


** Quick FAQs: **

* Are they really general purpose or do they have any special usages or meaning ?
> Although they are called general purpose registers, from the programming perspective, some of them do have special meanings. 

* Then why not call them Special purpose registers ?
> Well, the name comes as part of convention. Since these registers can be used to store data, address or even for indirection - they are general purpose registers. Certain other registers ( the instruction pointer register - IP for instance) cannot be used to store arbitrary data. So only such registers fall under special purpose registers.

* Who decides the special meanings/usages of the general purpose registers?
> It's up to the compiler manufacturer to decide - what each registers will be used for. Microsoft has documented their compiler's behavior on MSDN, if you're reversing code that was compiled on a Microsoft compiler, then it is useful to remember the below table by-heart.
 
* Are the register usages for the 64 bit register, the same as their 32 bit counter parts? 
> Not exactly, since the 64 bit processor has many more registers than the 32 bit processors, compilers use these registers to store parameters during function calls. Hence 64 bit processors are slightly different in their register usage. 


# Register Usages:

*32 bit register usage*

|Register | Usage|
|---------|---------|
|EAX |	Accumulator for operands and return value register|
|EBX | 	Pointer to data or general purpose|
|ECX |	Counter for string and loop operations; holds first parameter in fast call|
|EDX | 	Pointer to IO or general purpose; holds second parameter in fast call|
|ESI |	general purpose; pointer to data in the segment pointed to by the DS register; source pointer for string operations|
|EDI |	general purpose; Pointer to data (or destination) in the segment pointed to by the ES register; destination pointer for string operations|
|ESP |	Stack pointer (in the SS segment)|
|EBP |	Pointer to stack base/frame pointer|


*64 bit register usage*

|Register | Usage|
|---------|---------|
|RAX |	Accumulator; Return value register (volatile) |
|RBX |	general purpose; pointer to data; must be preserved by callee (non volatile) |
|RCX |	First integer argument (volatile) |
|RDX | 	Second integer argument (volatile) |
|R8 |	Third integer argument (volatile) |
|R9 |	Fourth integer argument (volatile) |
|R10:R11 |	general purpose; (volatile) preserved by caller; used in syscall/sysret instructions |
|R12:R15 |	general purpose; (non-volatile) must be preserved by callee |
|RDI | 	general purpose; must be preserved by callee(non-volatile) |
|RSI | 	general purpose; must be preserved by callee(non-volatile) |
|RBP | 	general purpose; May be used as a frame pointer; must be preserved by callee(non volatile) |
|RSP |	Stack pointer(non-volatile) |

You will also see the words Volatile and Non-volatile. If a register is marked Volatile, then it can be used freely between function calls. The calling function knows/expects that the value of these registers would have changed in the called function. If a register is marked as non volatile - then every function has to save the values of the register before it can make use of it. So the calling function expects that the caller maintains the values in these registers as it is between function calls. We'll discuss more on this in future posts. Bye for now!
