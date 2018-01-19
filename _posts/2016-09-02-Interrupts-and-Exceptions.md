---
layout: post
category: Reverse-Engg
title: Interrupts and Exceptions
date: 2016-09-02
---

Interrupts and Exceptions are 2 conditions that can divert the processor from executing code that is outside the normal flow of control. The processor plays an important part in generation and handling of interrupts and exceptions. In general we use the term ‘Trap handling’ to refer to the processor’s mechanism of handling an interrupt or exception and switching the context to a fixed location in the OS.

# Interrupts
An interrupt is a hardware signal available on most processors. It is meant to be used by peripherals to get the attention of the processor. When an interrupt occurs, the processor queries for the interrupting device and executes an interrupt service routine or ISR to service the interrupting device. Most of this happens transparently and is controlled by the hardware (processor, and the programmable interrupt controllers) and not by the OS.

Interrupts are classified broadly into hardware and software interrupts.
> * Hardware-generated interrupts - typically originate from I/O devices that must notify the processor when they need
service. Interrupt-driven devices allow the operating system to get the maximum use out of the processor by overlapping central processing with I/O operations.

For Example: A thread starts an I/O transfer to or from a device and then can execute other useful work while the device completes the transfer. When the device is finished, it interrupts the processor for service. Pointing devices, printers, keyboards, disk drives, and network cards are generally interrupt driven.

> * Software interrupt on the other hand is achieved when a driver/application executes an explicit `INT #` instruction. System software can also generate interrupts. For example, the kernel can issue a software interrupt to initiate thread dispatching and to asynchronously break into the execution of a thread.

The kernel can also disable interrupts so that the processor isn’t interrupted, but it does so only infrequently—at critical moments while it’s programming an interrupt controller or dispatching an exception.


# Exceptions
Exceptions are the events triggered by the processors as a response to code execution. In contrast to interrupts which are asynchronous, exceptions are synchronous. 

Examples of exceptions are “Divide by Zero”, “Invalid memory access”, Page Fault, Alignment Faults etc.

Exception classification by processor
> * Trap – The Exception is caused by the instruction/operands. If the operation is reattempted, trap will still occur.
Example – divide-by-zero, invalid memory access, debugger breakpoint

> * Fault – The exception was caused by some condition that can be remediated. If reattempted, fault may not occur.
Example – page fault, alignment fault

> * Aborts – are ambiguous


Exception Classification based on handling:
> * First chance exception
>> An exception is said to be a first chance exception when the debugger is notified of the exception for the first time. If the debugger chooses to pass the exception back to the application – the application may or may not handle this exception. If the exception is handled within the application, then this exception will not cause the
debugger to get notified again.

> * Second chance exception
>> If the debugger chooses to pass the first chance exception to the application, and the application does not handle it, then the debugger will be notified again. This is called second chance exception. If a second chance exception does occur, it means that the application has not handled the first chance exception.


# Interrupts priorities:
On the motherboard, there are devices known as Interrupt controllers interfaces with peripheral devices on one end and CPU on the other end. They implement one level of interrupt prioritization and control, by multiplexing several external interrupt lines into a single line to the processor. The processor also has a structure called interrupt dispatch table, which contains addresses of routines that can handle interrupts when they occur.

When an interrupt is raised, the processor queries the interrupt controller to get the interrupt request number (IRQ), it then uses this IRQ as an index into the interrupt dispatch table (IDT) to locate the corresponding interrupt service/dispatch routine. 

There are different variants of interrupt controllers – most x86 systems depend on either i8259 PIC or a variant of i82429 Advanced PIC. PICs are older and are only used on single processor machines. APICs on the other hand can be cascaded to support up to 256 interrupt request lines. APIC cascaded architecture typically consists of a local APIC that receives interrupts from one or more IO APICs.

Regardless of type of controller 0 the controller imposes priorities to the devices through interrupts request numbers or IRQs. In case of PICs, the IRQ is the line number where the device is attached. In case of APICs, this can be customized by software. These hardware imposed priorities impact generation and handling of the interrupt.

For Example:
- If 2 devices generate interrupt at the same time, the APIC records both interrupts and serializes them so that the one which is at higher priority gets serviced first.
- If during servicing of one interrupt, a higher priority interrupt gets signaled – then the interrupt controller will force the CPU to switch to the new interrupt and come back later and continue execution of the original interrupt.
- If during the servicing of one interrupt, the OS masks interrupts at that level and below – then the interrupt controller will hold any new interrupts until the OS unmasks the interrupts.


Although interrupt controllers perform prioritization of interrupts, windows implements its own prioritization called Interrupt request levels or IRQLs. It defines a standard set of IRQLs for software interrupts and then the HAL maps the hardware interrupt numbers to the remaining IRQLs. This mapping ensures that Interrupts are serviced in priority order and that higher priority interrupt pre-empts lower priority interrupts.

We'll discuss interrupt handling next.