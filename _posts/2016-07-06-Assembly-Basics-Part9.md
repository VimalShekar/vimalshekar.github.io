---
layout: post
category: Reverse-Engg
title: Assembly Language Basics - Part 9
date: 2016-07-06
---

# Stacks
Stacks are an important concept in programming. A Stack is a piece of memory mainly used for supporting procedure
calls and returns, and storing variables.

We know that a processor is capable of executing functions by jumping to a block of code at a given location, completing that function and then returning back for the original function. How does the processor know where to come back? The answer for that is stacks!

The process has a register called the stack pointer which points to a memory location. Whenever the processor is calling a function - it saves or pushes the address of the next instruction in the current function - to the stack. This way, when the called function returns, it can retrieve or pop that address from the stack and continue execution. Stacks operate in a LIFO model (Last in first out) - meaning whatever was pushed last into the stack is the one that will be popped out first. 

In Windows, every thread has 2 stacks associated with them – a User mode stack and a Kernel mode stack. In Windows - Stacks are also used to create allocations for local variables, to store exception handling information (in 32 bit) and to store return addresses and function parameters when one function/procedure calls another.

The Stack grown from higher address to lower address. Each thread has a stack that is delimited by a Stack Base and Stack limit. When the thread begins execution at the Stack base and grows towards the Stack limit value. The processor raises an exception if there is an overflow beyond the Stack limit value.

Important properties to remember about stacks:
* Stack grows from higher address to lower addresses
* Each stack element is pointer-sized – i.e. on 32Bit platform, stack element is 32Bit and 64Bit on 64Bit platform
* The ESP/RSP register is the stack pointer. On a running machine, this register always points to the last value that was pushed into the stack (aka - Top of the Stack).
* When items are inserted or pushed into the stack, the stack pointer decrements itself, when items are removed or popped from the stack - the stack pointer increments itself.
* The Instruction pointer (EIP or RIP register) always holds the address of the next instruction to be executed. During a call instruction, the value in Instruction pointer is pushed into the stack, and Instruction pointer is updated with the address of the call target.  During the Return instruction, the previously pushed Instruction pointer is popped from the stack and copied into the IP register. This is how the processor is able to call the function and return back to the location that follows the call instruction.



Walking raw stack requires one to refer to assembly of the code that has built the stack. Debuggers in the Debugging tools for windows, do this in the background to present a readable view. windbg has the `k` command that lets you view the stack. There are several variations of this command, type `.hh k` to see the help for this command.

Here's a sample output of the kb command. If you notice, the debugger shows the stack in inverted form, with the last called function is at the top. In this case `ntdll!RtlUserThreadStart()` called into `KERNEL32!BaseThreadInitThunk()` which called `notepad!__mainCRTStartup()` which called `notepad!WinMain()`.

    0:000> kn
    # Child-SP          RetAddr           Call Site
    00 000000c2`3167f7b8 00007ff6`3f049593 notepad!WinMain
    01 000000c2`3167f7c0 00007ffe`139c1fe4 notepad!__mainCRTStartup+0x19f
    02 000000c2`3167f880 00007ffe`13e8efb1 KERNEL32!BaseThreadInitThunk+0x14
    03 000000c2`3167f8b0 00000000`00000000 ntdll!RtlUserThreadStart+0x21

    0:000> kb
    # RetAddr           : Args to Child                                                           : Call Site
    00 00007ff6`3f049593 : 000001db`9a3a3b35 00000000`00000000 00000000`00000000 00000000`00000000 : notepad!WinMain
    01 00007ffe`139c1fe4 : 00000000`00000000 00000000`00000000 00000000`00000000 00000000`00000000 : notepad!__mainCRTStartup+0x19f
    02 00007ffe`13e8efb1 : 00000000`00000000 00000000`00000000 00000000`00000000 00000000`00000000 : KERNEL32!BaseThreadInitThunk+0x14
    03 00000000`00000000 : 00000000`00000000 00000000`00000000 00000000`00000000 00000000`00000000 : ntdll!RtlUserThreadStart+0x21


We'll discuss more about variables pushed into the stack in the calling convention post.