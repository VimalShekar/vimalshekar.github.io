---
layout: post
category: Reverse-Engg
title: Assembly Language Basics - Part 2
date: 2016-03-10
---

In the last post, I tried to describe the general purpose registers and their usage in the previous post. This time, lets looks at a special purpose register called the Program status word or FLAGS register. 

The Flags register is a Special purpose register in which every bit has a special meaning. Basically it is a group of control and status flags that may change to indicate the result of arithmetic or logical operations. Here are the bits of the 32 bit flag register:

|Flag bit | Description|
|---------|------------|
|CF (bit 0) |	Carry flag. This flag gets set if an arithmetic operation generates a carry or a borrow out of the most significant bit of the result; It is cleared otherwise. This flag also indicates an overflow condition for unsigned-integer arithmetic. It is also used in multiple-precision arithmetic. |
|PF (bit 2) |	Parity flag. This flag gets set if the least-significant byte of the result of an arithmetic or logical  instruction contains an even number of bits which are set. |
|AF (bit 4) |	Auxiliary Carry flag. This flag gets set if an arithmetic operation generates a carry or a borrow out of bit 3 of the result. This flag is used in binary-coded decimal (BCD) arithmetic. |
|ZF (bit 6) |	Zero flag — Set if the result is zero; cleared otherwise. |
|SF (bit 7) |	Sign flag — Set equal to the most-significant bit of the result, which is the sign bit of a	signed integer. (0 indicates a positive value and 1 indicates a negative value.) |
|DF (Bit10) |	Direction flag — controls string instructions (MOVS, CMPS, SCAS,LODS, and STOS) |
|OF (bit 11)| 	Overflow flag. This flag gets set if the integer result is too large a positive number or too small a negative number (excluding the sign-bit) to fit in the destination operand. |
|TF (bit 8) |	Trap Flag — Set to enable single-step mode for debugging; clear to disable single-step mode.
|IF (bit 9) |	Interrupt enable flag — Controls the response of the processor to maskable interrupt requests. Set to respond to maskable interrupts; cleared to inhibit maskable |interrupts.
|IOPL (bits 12	I/O privilege level field and 13) |	This flag indicates the I/O privilege level of the currently running program or task. The current privilege level (CPL) of the currently running program or task must be less |than or equal to the I/O privilege level to access the I/O address space. The POPF and IRET instructions can modify this field only when operating at a CPL of 0. |


In addition to arithmetic and logic instructions - Some of the flags in the EFLAGS register can be modified directly, using special-purpose instructions. Generally you first save the contents of the flag register by moving it to another register before modifying individual bits. The following instructions can be used to move groups of flags to and from the procedure stack or the EAX register:
LAHF, SAHF, PUSHF, PUSHFD, POPF, and POPFD.

After the contents of the EFLAGS register have been transferred to the procedure stack or EAX register, the flags can be examined and modified using the processor’s bit manipulation instructions (BT, BTS, BTR, and BTC). There are no instructions that allow the whole register to be examined or modified directly. 