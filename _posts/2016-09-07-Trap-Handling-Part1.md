---
layout: post
category: Reverse-Engg
title: Trap Handling - Part 1
date: 2016-09-07
---

# How the kernel handles a trap!
The main difference between interrupts and exceptions is that an interrupt is an asynchronous event (one that can occur at any time) that is unrelated to what the processor is executing. Interrupts are generated primarily by I/O devices, processor clocks, or timers, and they can be enabled (turned on) or disabled (turned off). An exception, in contrast, is asynchronous condition that usually results from the execution of a particular instruction. (Aborts, such as machine checks, is one type of processor exception that’s typically not associated with instruction execution). Generally – running the program a second time with the same data under the same conditions can reproduce exceptions. The Windows kernel also regards system service calls as exceptions.



The processor plays a major role in handling interrupts and exceptions (together, we call them traps). On occurrence of a trap, the processor some machine state on the stack of the thread that was interrupted. If the thread was executing in user mode, then the mode is switched to kernel mode before processor saves the state. Windows provides trap handlers for various types of interrupts and exceptions it handles, third party drivers can also register their own ISRs to handle device interrupts.
 
The first task of the trap handler is to save the context of the currently executing thread. This means – saving the CPU registers to some place so that execution can continue from the same point, once the trap has been handled. These trap handlers save the state into what is called – “a trap frame”. This is a data structure stored on the stack, which consists of a subset of the context of thread. This context can be used to revert back the interrupted thread at a later point in time and continue execution as if it was never interrupted.

Once the trap frame is saved – the processor checks the Interrupt descriptor table to find the handler which can handle this trap. Lets deviate a little and take a look at the IDT in a debugger. I’ve attached my debugger to a hyper-V VM which has two processors. We discussed how to do this in an earlier post.


`!cpuinfo` will list basic information about the processors on the box:

    1: kd> !cpuinfo
    CP F/M/S Manufacturer MHz PRCB Signature MSR 8B Signature Features
    0 6,15,1 GenuineIntel 2201 0000070d00000000 20193ffe
    1 6,15,1 GenuineIntel 2200 0000070d00000000 20193ffe

    Cached Update Signature 0000070d00000000
    Initial Update Signature 0000070d00000000


You can use ~#s to switch to the context to the processor number ‘#’. You can use !idt to dump the Interrupt descriptor table:

    1: kd> ~0s; !idt
    Dumping IDT: fffff80003189070
    642a57a00000000:        fffff80001e9fc80 nt!KiDivideErrorFault
    642a57a00000001:        fffff80001e9fd40 nt!KiDebugTrapOrFault
    642a57a00000002:        fffff80001e9fe80 nt!KiNmiInterrupt        Stack = 0xFFFFF8000319B000
    642a57a00000003:        fffff80001ea01c0 nt!KiBreakpointTrap
    642a57a00000004:        fffff80001ea0280 nt!KiOverflowTrap
    642a57a00000005:        fffff80001ea0340 nt!KiBoundFault
    642a57a00000006:        fffff80001ea0400 nt!KiInvalidOpcodeFault
    642a57a00000007:        fffff80001ea05c0 nt!KiNpxNotAvailableFault
    642a57a00000008:        fffff80001ea0680 nt!KiDoubleFaultAbort        Stack = 0xFFFFF80003199000
    642a57a00000009:        fffff80001ea0740 nt!KiNpxSegmentOverrunAbort
    642a57a0000000a:        fffff80001ea0800 nt!KiInvalidTssFault
    642a57a0000000b:        fffff80001ea08c0 nt!KiSegmentNotPresentFault
    642a57a0000000c:        fffff80001ea09c0 nt!KiStackFault
    642a57a0000000d:        fffff80001ea0ac0 nt!KiGeneralProtectionFault
    642a57a0000000e:        fffff80001ea0bc0 nt!KiPageFault
    642a57a00000010:        fffff80001ea0f00 nt!KiFloatingErrorFault
    642a57a00000011:        fffff80001ea1040 nt!KiAlignmentFault
    642a57a00000012:        fffff80001ea1100 nt!KiMcheckAbort        Stack = 0xFFFFF8000319D000
    642a57a00000013:        fffff80001ea1440 nt!KiXmmException
    642a57a0000001f:        fffff80001ec7210 nt!KiApcInterrupt
    642a57a0000002c:        fffff80001ea15c0 nt!KiRaiseAssertion
    642a57a0000002d:        fffff80001ea1680 nt!KiDebugServiceTrap
    642a57a0000002f:        fffff80001eeaa30 nt!KiDpcInterrupt
    642a57a00000037:        fffff80001e32090 hal!HalpApicSpuriousService (KINTERRUPT fffff80001e32000)
    642a57a0000003f:        fffff80001e32130 hal!HalpApicSpuriousService (KINTERRUPT fffff80001e320a0)
    642a57a00000051:        fffffa80070a42d0 serial!SerialCIsrSw (KINTERRUPT fffffa80070a4240)
    642a57a00000052:        fffffa8007013810 pci!ExpressRootPortMessageRoutine (KINTERRUPT fffffa8007013780)
    642a57a00000053:        fffffa80070132d0 pci!ExpressRootPortMessageRoutine (KINTERRUPT fffffa8007013240)
    642a57a00000054:        fffffa80070a4d50 pci!ExpressRootPortMessageRoutine (KINTERRUPT fffffa80070a4cc0)
    642a57a00000055:        fffffa80070a4810 pci!ExpressRootPortMessageRoutine (KINTERRUPT fffffa80070a4780)
    642a57a00000056:        fffffa8007437d50 NDIS!ndisMiniportMessageIsr (KINTERRUPT fffffa8007437cc0)
    642a57a00000060:        fffffa8007013bd0 pci!ExpressRootPortMessageRoutine (KINTERRUPT fffffa8007013b40)
    642a57a00000062:        fffffa80070138d0 pci!ExpressRootPortMessageRoutine (KINTERRUPT fffffa8007013840)
    642a57a00000063:        fffffa8007013390 pci!ExpressRootPortMessageRoutine (KINTERRUPT fffffa8007013300)
    642a57a00000064:        fffffa80070a4e10 pci!ExpressRootPortMessageRoutine (KINTERRUPT fffffa80070a4d80)
    642a57a00000065:        fffffa80070a48d0 pci!ExpressRootPortMessageRoutine (KINTERRUPT fffffa80070a4840)
    642a57a00000066:        fffffa8007437e10 NDIS!ndisMiniportMessageIsr (KINTERRUPT fffffa8007437d80)
    642a57a00000067:        fffffa80070a4150 vmci!DllInitialize+0x5e4 (KINTERRUPT fffffa80070a40c0)
    VIDEOPRT!pVideoPortInterrupt (KINTERRUPT fffffa80070a4000)
    642a57a00000070:        fffffa8007013c90 pci!ExpressRootPortMessageRoutine (KINTERRUPT fffffa8007013c00)
    642a57a00000071:        fffffa80070a4390 i8042prt!I8042MouseInterruptService (KINTERRUPT fffffa80070a4300)
    642a57a00000072:        fffffa8007013990 pci!ExpressRootPortMessageRoutine (KINTERRUPT fffffa8007013900)
    642a57a00000073:        fffffa8007013450 pci!ExpressRootPortMessageRoutine (KINTERRUPT fffffa80070133c0)
    642a57a00000074:        fffffa80070a4ed0 pci!ExpressRootPortMessageRoutine (KINTERRUPT fffffa80070a4e40)
    642a57a00000075:        fffffa80070a4990 pci!ExpressRootPortMessageRoutine (KINTERRUPT fffffa80070a4900)
    642a57a00000076:        fffffa8007437ed0 NDIS!ndisMiniportMessageIsr (KINTERRUPT fffffa8007437e40)
    642a57a00000077:        fffffa8007437990 NDIS!ndisMiniportMessageIsr (KINTERRUPT fffffa8007437900)
    642a57a00000080:        fffffa8007013d50 pci!ExpressRootPortMessageRoutine (KINTERRUPT fffffa8007013cc0)
    642a57a00000081:        fffffa80070a4450 i8042prt!I8042KeyboardInterruptService (KINTERRUPT fffffa80070a43c0)
    642a57a00000082:        fffffa8007013a50 pci!ExpressRootPortMessageRoutine (KINTERRUPT fffffa80070139c0)
    642a57a00000083:        fffffa8007013510 pci!ExpressRootPortMessageRoutine (KINTERRUPT fffffa8007013480)
    642a57a00000084:        fffffa80070a4f90 pci!ExpressRootPortMessageRoutine (KINTERRUPT fffffa80070a4f00)
    642a57a00000085:        fffffa80070a4a50 pci!ExpressRootPortMessageRoutine (KINTERRUPT fffffa80070a49c0)
    642a57a00000086:        fffffa8007437f90 NDIS!ndisMiniportMessageIsr (KINTERRUPT fffffa8007437f00)
    642a57a00000087:        fffffa8007437a50 NDIS!ndisMiniportMessageIsr (KINTERRUPT fffffa80074379c0)
    642a57a00000090:        fffffa8007013e10 pci!ExpressRootPortMessageRoutine (KINTERRUPT fffffa8007013d80)
    642a57a00000092:        fffffa8007013b10 pci!ExpressRootPortMessageRoutine (KINTERRUPT fffffa8007013a80)
    642a57a00000093:        fffffa80070135d0 pci!ExpressRootPortMessageRoutine (KINTERRUPT fffffa8007013540)
    642a57a00000094:        fffffa8007013090 pci!ExpressRootPortMessageRoutine (KINTERRUPT fffffa8007013000)
    642a57a00000095:        fffffa80070a4b10 pci!ExpressRootPortMessageRoutine (KINTERRUPT fffffa80070a4a80)
    642a57a00000096:        fffffa80070a4510 storport!RaidpAdapterMSIInterruptRoutine (KINTERRUPT fffffa80070a4480)
    642a57a00000097:        fffffa8007437b10 NDIS!ndisMiniportMessageIsr (KINTERRUPT fffffa8007437a80)
    642a57a000000a0:        fffffa8007013ed0 pci!ExpressRootPortMessageRoutine (KINTERRUPT fffffa8007013e40)
    642a57a000000a3:        fffffa8007013690 pci!ExpressRootPortMessageRoutine (KINTERRUPT fffffa8007013600)
    642a57a000000a4:        fffffa8007013150 pci!ExpressRootPortMessageRoutine (KINTERRUPT fffffa80070130c0)
    642a57a000000a5:        fffffa80070a4bd0 pci!ExpressRootPortMessageRoutine (KINTERRUPT fffffa80070a4b40)
    642a57a000000a6:        fffffa80070a45d0 ataport!IdePortInterrupt (KINTERRUPT fffffa80070a4540)
    642a57a000000a7:        fffffa8007437bd0 NDIS!ndisMiniportMessageIsr (KINTERRUPT fffffa8007437b40)
    642a57a000000b0:        fffffa80070a4750 pci!ExpressRootPortMessageRoutine (KINTERRUPT fffffa80070a46c0)
    642a57a000000b1:        fffffa8007013f90 acpi!ACPIInterruptServiceRoutine (KINTERRUPT fffffa8007013f00)
    642a57a000000b2:        fffffa80070a4210 serial!SerialCIsrSw (KINTERRUPT fffffa80070a4180)
    642a57a000000b3:        fffffa8007013750 pci!ExpressRootPortMessageRoutine (KINTERRUPT fffffa80070136c0)
    642a57a000000b4:        fffffa8007013210 pci!ExpressRootPortMessageRoutine (KINTERRUPT fffffa8007013180)
    642a57a000000b5:        fffffa80070a4c90 pci!ExpressRootPortMessageRoutine (KINTERRUPT fffffa80070a4c00)
    642a57a000000b6:        fffffa80070a4690 ataport!IdePortInterrupt (KINTERRUPT fffffa80070a4600)
    642a57a000000b7:        fffffa8007437c90 NDIS!ndisMiniportMessageIsr (KINTERRUPT fffffa8007437c00)
    642a57a000000c1:        fffff80001e32310 hal!HalpBroadcastCallService (KINTERRUPT fffff80001e32280)
    642a57a000000d1:        fffff80001e323b0 hal!HalpRtcClockInterrupt (KINTERRUPT fffff80001e32320)
    642a57a000000df:        fffff80001e32270 hal!HalpApicRebootService (KINTERRUPT fffff80001e321e0)
    642a57a000000e1:        fffff80001ea6fa0 nt!KiIpiInterrupt
    642a57a000000e3:        fffff80001e321d0 hal!HalpLocalApicErrorService (KINTERRUPT fffff80001e32140)
    642a57a000000fd:        fffff80001e32450 hal!HalpProfileInterrupt (KINTERRUPT fffff80001e323c0)
    642a57a000000fe:        fffff80001e324f0 hal!HalpPerfInterrupt (KINTERRUPT fffff80001e32460)
    642a57a000000ff:        0000000000000000


* Can the IDT entry for a particular interrupt point to two different interrupt objects?
> Yes!, each processor has its own IDT, so that different processors can run different IDT if appropriate. For example in a multi-processor machine all processors receive clock interrupts – only one updates the system clock, all others just use this for thread quantum measurements and scheduling. Similarly some devices may require that only a particular processor handle the interrupt.


* What does an entry in the IDT denote?
> Each entry in the IDT is actually an INTERRUPT object. The INTERRUPT object is a datastructure that contain information about how to handle an interrupts – including it IRQL, ISR address and locks. Lets take the IDE port interrupt as an example.


Looking at the output from `!idt`, we see this is the interrupt object

    642a57a000000b6:        fffffa80070a4690 ataport!IdePortInterrupt (KINTERRUPT fffffa80070a4600)

    1: kd> dt nt!_KINTERRUPT fffffa80070a4600
    +0x000 Type : 0n22
    +0x002 Size : 0n160
    +0x008 InterruptListEntry : _LIST_ENTRY [ 0x00000000`00000000 – 0x00000000`00000000 ]
    +0x018 ServiceRoutine : 0xfffffa60`00caf270 unsigned char ataport!IdePortInterrupt+0
    +0x020 MessageServiceRoutine : (null)
    +0x028 MessageIndex : 0
    +0x030 ServiceContext : 0xfffffa80`06eb7050 Void
    +0x038 SpinLock : 0
    +0x040 TickCount : 0
    +0x048 ActualLock : 0xfffffa80`070283c0 -> 0
    +0x050 DispatchAddress : 0xfffff800`01e9aa30 void nt!KiInterruptDispatch+0
    +0x058 Vector : 0xb6
    +0x05c Irql : 0xb ”
    +0x05d SynchronizeIrql : 0xb ”
    +0x05e FloatingSave : 0 ”
    +0x05f Connected : 0x1 ”
    +0x060 Number : 0 ”
    +0x061 ShareVector : 0 ”
    +0x064 Mode : 1 ( Latched )
    +0x068 Polarity : 0 ( InterruptPolarityUnknown )
    +0x06c ServiceCount : 0
    +0x070 DispatchCount : 0
    +0x078 Rsvd1 : 0
    +0x080 TrapFrame : (null)
    +0x088 Reserved : (null)
    +0x090 DispatchCode : [4] 0x8d485550

* Is it possible that the same ISR may be registered for different interrupt vector numbers?

> Yes, In some cases, you may that an ISR is listed multiple times in the IDT. This is because in some systems there may be multiple physical devices that have the same interrupt handling routine. For example multiple NIC cards will have the same ISR : NDIS!ndisMiniportMessageIsr. So you will see this listed multiple times in the IDT. But when you dump out the individual Interrupt objects, you will see that they see more than one entry in idt that is executing

    Line 35: 642a57a00000056:        fffffa8007437d50 NDIS!ndisMiniportMessageIsr (KINTERRUPT fffffa8007437cc0)
    Line 41: 642a57a00000066:        fffffa8007437e10 NDIS!ndisMiniportMessageIsr (KINTERRUPT fffffa8007437d80)
    Line 50: 642a57a00000076:        fffffa8007437ed0 NDIS!ndisMiniportMessageIsr (KINTERRUPT fffffa8007437e40)

    .. snip ..

    1: kd> dt nt!_KINTERRUPT fffffa8007437cc0
    +0x000 Type : 0n22
    +0x002 Size : 0n160
    +0x008 InterruptListEntry : _LIST_ENTRY [ 0x00000000`00000000 – 0x00000000`00000000 ]
    +0x018 ServiceRoutine : 0xfffff800`01e74a40 unsigned char nt!KiInterruptMessageDispatch+0
    +0x020 MessageServiceRoutine : 0xfffffa60`0097a3d0 unsigned char NDIS!ndisMiniportMessageIsr+0
    +0x028 MessageIndex : 4
    +0x030 ServiceContext : 0xfffffa80`0741e000 Void
    +0x038 SpinLock : 0
    +0x040 TickCount : 0
    +0x048 ActualLock : 0xfffffa80`0741d6e0 -> 0
    +0x050 DispatchAddress : 0xfffff800`01e9aa30 void nt!KiInterruptDispatch+0
    +0x058 Vector : 0x56
    +0x05c Irql : 0x5 ”
    +0x05d SynchronizeIrql : 0x5 ”
    +0x05e FloatingSave : 0 ”
    +0x05f Connected : 0x1 ”
    +0x060 Number : 0 ”
    +0x061 ShareVector : 0x1 ”
    +0x064 Mode : 1 ( Latched )
    +0x068 Polarity : 0 ( InterruptPolarityUnknown )
    +0x06c ServiceCount : 0
    +0x070 DispatchCount : 0
    +0x078 Rsvd1 : 0
    +0x080 TrapFrame : (null)
    +0x088 Reserved : (null)
    +0x090 DispatchCode : [4] 0x8d485550

    1: kd> dt nt!_KINTERRUPT fffffa8007437d80
    +0x000 Type : 0n22
    +0x002 Size : 0n160
    +0x008 InterruptListEntry : _LIST_ENTRY [ 0x00000000`00000000 – 0x00000000`00000000 ]
    +0x018 ServiceRoutine : 0xfffff800`01e74a40 unsigned char nt!KiInterruptMessageDispatch+0
    +0x020 MessageServiceRoutine : 0xfffffa60`0097a3d0 unsigned char NDIS!ndisMiniportMessageIsr+0
    +0x028 MessageIndex : 3
    +0x030 ServiceContext : 0xfffffa80`0741e000 Void
    +0x038 SpinLock : 0
    +0x040 TickCount : 0
    +0x048 ActualLock : 0xfffffa80`0741d7f0 -> 0
    +0x050 DispatchAddress : 0xfffff800`01e9aa30 void nt!KiInterruptDispatch+0
    +0x058 Vector : 0x66
    +0x05c Irql : 0x6 ”
    +0x05d SynchronizeIrql : 0x6 ”
    +0x05e FloatingSave : 0 ”
    +0x05f Connected : 0x1 ”
    +0x060 Number : 0 ”
    +0x061 ShareVector : 0x1 ”
    +0x064 Mode : 1 ( Latched )
    +0x068 Polarity : 0 ( InterruptPolarityUnknown )
    +0x06c ServiceCount : 0
    +0x070 DispatchCount : 0
    +0x078 Rsvd1 : 0
    +0x080 TrapFrame : (null)
    +0x088 Reserved : (null)
    +0x090 DispatchCode : [4] 0x8d485550

    1: kd> dt nt!_KINTERRUPT fffffa8007437e40
    +0x000 Type : 0n22
    +0x002 Size : 0n160
    +0x008 InterruptListEntry : _LIST_ENTRY [ 0x00000000`00000000 – 0x00000000`00000000 ]
    +0x018 ServiceRoutine : 0xfffff800`01e74a40 unsigned char nt!KiInterruptMessageDispatch+0
    +0x020 MessageServiceRoutine : 0xfffffa60`0097a3d0 unsigned char NDIS!ndisMiniportMessageIsr+0
    +0x028 MessageIndex : 2
    +0x030 ServiceContext : 0xfffffa80`0741e000 Void
    +0x038 SpinLock : 0
    +0x040 TickCount : 0
    +0x048 ActualLock : 0xfffffa80`0741d900 -> 0
    +0x050 DispatchAddress : 0xfffff800`01e9aa30 void nt!KiInterruptDispatch+0
    +0x058 Vector : 0x76
    +0x05c Irql : 0x7 ”
    +0x05d SynchronizeIrql : 0x7 ”
    +0x05e FloatingSave : 0 ”
    +0x05f Connected : 0x1 ”
    +0x060 Number : 0 ”
    +0x061 ShareVector : 0x1 ”
    +0x064 Mode : 1 ( Latched )
    +0x068 Polarity : 0 ( InterruptPolarityUnknown )
    +0x06c ServiceCount : 0
    +0x070 DispatchCount : 0
    +0x078 Rsvd1 : 0
    +0x080 TrapFrame : 0xfffffa60`09f23c20 _KTRAP_FRAME
    +0x088 Reserved : (null)
    +0x090 DispatchCode : [4] 0x8d485550


Lets stop here for now, more in the next post. Bye!