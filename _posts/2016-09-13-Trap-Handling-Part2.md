---
layout: post
category: Reverse-Engg
title: Trap Handling - Part 2
date: 2016-09-13
---

# How the kernel handles a trap!
Lets continue our investigation on how interrupts are handled:

When an interrupt arrives, the processor may be executing any random thread that may be unrelated to the interrupt. It is important to save the current state of execution before jumping to service the interrupt. Microsoft’s MSDN documentation about writing ISRs doesnt mention much about saving context. This is because the Windows kernel itself takes care of doing it.

To make sure the current state of execution is saved, trap handlers are inserted by the OS into each ISR and entry in the IDT. This trap handling or Dispatch code saves the context of the thread onto its kernel mode stack – this is called a TRAP frame. 


    //-- You can take a look at the structure by using this command:
    1: kd> dt nt!_KTRAP_FRAME
    +0x000 P1Home : Uint8B
    +0x008 P2Home : Uint8B
    +0x010 P3Home : Uint8B
    +0x018 P4Home : Uint8B
    +0x020 P5 : Uint8B
    +0x028 PreviousMode : Char
    +0x029 PreviousIrql : UChar
    +0x02a FaultIndicator : UChar
    +0x02b ExceptionActive : UChar
    +0x02c MxCsr : Uint4B
    ..snip..

Once context is saved, it performs some maintenence work before transferring control to an external routine (the ISR) to handle the interrupt. Device drivers supply ISRs to service device interrupts, and the kernel provides interrupt-handling routines for other types of interrupts.

Again, not all registers are saved by the OS, the processor saves some of the registers for us by default. Only the remaining registers need to be saved.

* What the processor saves for us.
a. The processor saves the current state of the EFLAGS, CS, and EIP registers on the current stack
b. If an exception causes an error code to be saved, it is pushed on the current stack after the EIP value.

If privilege level of currently executing code is lesser than privilege level required to run interrupt, then we switch to the kernel mode stack of the process. Windows fill tss appropriately when dispatching a thread. Processor loads the segment selector and stack pointer for the new stack


Let’s examine this in the disassembly:

— Pickup an interrupt object from the IDT and dump it:

    kd> dt nt!_KINTERRUPT fffff80002c24000
    +0x000 Type : 0n22
    +0x002 Size : 0n160
    +0x008 InterruptListEntry : _LIST_ENTRY [ 0x00000000`00000000 – 0x00000000`00000000 ]
    +0x018 ServiceRoutine : 0xfffff800`02bf52bc unsigned char hal!HalpApicSpuriousService+0
    +0x020 MessageServiceRoutine : (null)
    +0x028 MessageIndex : 0
    +0x030 ServiceContext : (null)
    +0x038 SpinLock : 0
    +0x040 TickCount : 0
    +0x048 ActualLock : 0xffffffff`fffffffe -> ??
    +0x050 DispatchAddress : 0xfffff800`02686310 void nt!KiInterruptDispatch+0
    +0x058 Vector : 0x37
    +0x05c Irql : 0x3 ”
    +0x05d SynchronizeIrql : 0x3 ”
    +0x05e FloatingSave : 0 ”
    +0x05f Connected : 0 ”
    +0x060 Number : 0
    +0x064 ShareVector : 0 ”
    +0x065 Pad : [3] “”
    +0x068 Mode : 1 ( Latched )
    +0x06c Polarity : 0 ( InterruptPolarityUnknown )
    +0x070 ServiceCount : 0
    +0x074 DispatchCount : 0
    +0x078 Rsvd1 : 0
    +0x080 TrapFrame : (null)
    +0x088 Reserved : (null)
    +0x090 DispatchCode : [4] 0xd905ff48

— Notice that irrespective of what interrupt it is, the Dispatch address always goes to one of these 3 functions ?

    kd> x nt!Ki*Interrupt*Dispatch
    fffff800`02686310 nt!KiInterruptDispatch (<no parameter info>)
    fffff800`0264fd70 nt!KiInterruptMessageDispatch (<no parameter info>)

    kd> x nt!Ki*chained*Dispatch
    fffff800`02685f30 nt!KiChainedDispatch (<no parameter info>)

— Lets set a breakpoint at KiInterruptDispatch and examine what’s on the stack:

    kd> bp nt!KiInterruptDispatch

    kd> g
    Breakpoint 0 hit

    kd> kn
    # Child-SP RetAddr Call Site
    00 fffff880`02f82448 fffff800`02685ede nt!KiInterruptDispatch
    01 fffff880`02f82480 fffff880`0105fe95 nt!KeSynchronizeExecution+0x4e
    02 fffff880`02f824c0 fffff880`0105fb6e ataport!IdeStartIoCallBack+0x2d5
    03 fffff880`02f82630 fffff880`0105e803 ataport!IdeDispatchChannelRequest+0x10a
    04 fffff880`02f82660 fffff880`0105e668 ataport!IdeStartChannelRequest+0x113
    05 fffff880`02f826e0 fffff880`0105f9fa ataport!IdeStartDeviceRequest+0x2c4
    06 fffff880`02f827a0 fffff880`0105b4ee ataport!IdePortPdoDispatch+0xb6
    07 fffff880`02f827d0 fffff880`00f907a7 ataport!IdePortDispatch+0x16
    08 fffff880`02f82800 fffff880`00f98789 ACPI!ACPIDispatchForwardIrp+0x37
    09 fffff880`02f82830 fffff880`00f90a3f ACPI!ACPIIrpDispatchDeviceControl+0x75
    0a fffff880`02f82860 fffff880`00ee00c2 ACPI!ACPIDispatchIrp+0x12b
    0b fffff880`02f828e0 fffff880`00ef559f Wdf01000!FxIoTarget::SubmitSync+0x24a
    0c fffff880`02f82990 fffff880`01620012 Wdf01000!imp_WdfRequestSend+0x24b
    0d fffff880`02f829e0 fffff880`01616dab cdrom!RequestSendMcnRequest+0x5a
    0e fffff880`02f82a30 fffff880`00f126fb cdrom!RequestProcessSerializedIoctl+0x583
    0f fffff880`02f82b10 fffff800`02980f33 Wdf01000!FxWorkItem::WorkItemThunk+0x113
    10 fffff880`02f82b40 fffff800`02694a21 nt!IopProcessWorkItem+0x23
    11 fffff880`02f82b70 fffff800`02927cce nt!ExpWorkerThread+0x111
    12 fffff880`02f82c00 fffff800`0267bfe6 nt!PspSystemThreadStartup+0x5a
    13 fffff880`02f82c40 00000000`00000000 nt!KiStartSystemThread+0x16

— See that the stack shows fffff800`02685ede as return address of nt!KiInterruptDispatch. But does nt!KeSynchronizeExecution actually call KiInterruptDispatch ???

    kd> ub fffff800`02685ede
    nt!KeSynchronizeExecution+0x2f:
    fffff800`02685ebf 488b542448 mov rdx,qword ptr [rsp+48h]
    fffff800`02685ec4 488b4c2450 mov rcx,qword ptr [rsp+50h]
    fffff800`02685ec9 ffd2 call rdx
    fffff800`02685ecb 488bce mov rcx,rsi
    fffff800`02685ece 408af0 mov sil,al
    fffff800`02685ed1 e85af20000 call nt!KeReleaseSpinLockFromDpcLevel (fffff800`02695130)
    fffff800`02685ed6 8b4c2420 mov ecx,dword ptr [rsp+20h]
    fffff800`02685eda 440f22c1 mov cr8,rcx

    kd> u
    nt!KeSynchronizeExecution+0x4e:
    fffff800`02685ede 408ac6 mov al,sil <–------ See, there are no calls to KiInterruptDispatch() in this function
    fffff800`02685ee1 488b742430 mov rsi,qword ptr [rsp+30h]
    fffff800`02685ee6 4883c438 add rsp,38h
    fffff800`02685eea c3 ret
    fffff800`02685eeb cc int 3
    fffff800`02685eec cc int 3
    fffff800`02685eed cc int 3
    fffff800`02685eee cc int 3

We know that the processor saves some stuff for us. Lets look for these by dumping the raw stack using dps at the stack pointer address.
a. The processor saves the current state of the EFLAGS, CS, and EIP registers on the current stack
b. If an exception causes an error code to be saved, it is pushed on the current stack after the EIP value.

    kd> .frame /r 0

    00 fffff880`02f82448 fffff800`02685ede nt!KiInterruptDispatch
    rax=fffff88002f82301 rbx=0000000000000004 rcx=0000000000000002
    rdx=0000000000000170 rsi=fffffa83029ebf01 rdi=fffffa8302c86010
    rip=fffff80002686310 rsp=fffff88002f82448 rbp=fffffa8302094cc0
    r8=0000000000000000 r9=fffffa83029d46a8 r10=fffffa8303442380
    r11=fffff88002f823b0 r12=fffffa8302c817d0 r13=fffffa8302c863f8
    r14=fffff8800106a1a0 r15=fffffa83029d31a0
    iopl=0 nv up di pl zr na po nc
    cs=0010 ss=0018 ds=002b es=002b fs=0053 gs=002b efl=00000046

    kd> dps fffff880`02f82448
    fffff880`02f82458 fffff800`02685ede nt!KeSynchronizeExecution+0x4e <–IP
    fffff880`02f82460 00000000`00000010 <–cs
    fffff880`02f82468 00000000`00000246 <–EFlags
    fffff880`02f82470 fffff880`02f82480 <–
    fffff880`02f82478 00000000`00000018
    fffff880`02f82480 00000000`00000004
    fffff880`02f82488 fffff880`01055c74 ataport!IdeLogCrbActive+0xe0
    fffff880`02f82490 fffffa83`029ebf00
    fffff880`02f82498 00000000`00009b23
    fffff880`02f824a0 fffffa83`00000002
    fffff880`02f824a8 fffffa83`02c86010


— Now, lets dis-assemble and examine KiInterruptDispatch

    kd> uF nt!KiInterruptDispatch
    nt!KiInterruptDispatch:
    fffff800`02686310 56 push rsi
    fffff800`02686311 4881ec50010000 sub rsp,150h <<– allocating space for a TRAP frame
    fffff800`02686318 488bf5 mov rsi,rbp

— Here the code performs context save by saving registers

    fffff800`0268631b 488dac2480000000 lea rbp,[rsp+80h]
    fffff800`02686323 c645ab00 mov byte ptr [rbp-55h],0
    fffff800`02686327 488945b0 mov qword ptr [rbp-50h],rax
    fffff800`0268632b 48894db8 mov qword ptr [rbp-48h],rcx
    fffff800`0268632f 488955c0 mov qword ptr [rbp-40h],rdx
    fffff800`02686333 4c8945c8 mov qword ptr [rbp-38h],r8
    fffff800`02686337 4c894dd0 mov qword ptr [rbp-30h],r9
    fffff800`0268633b 4c8955d8 mov qword ptr [rbp-28h],r10
    fffff800`0268633f 4c895de0 mov qword ptr [rbp-20h],r11
    fffff800`02686343 f685f000000001 test byte ptr [rbp+0F0h],1
    fffff800`0268634a 7421 je nt!KiInterruptDispatch+0x5d (fffff800`0268636d) Branch

    nt!KiInterruptDispatch+0x3c:
    fffff800`0268634c 0f01f8 swapgs
    fffff800`0268634f 654c8b142588010000 mov r10,qword ptr gs:[188h]
    fffff800`02686358 41f6420303 test byte ptr [r10+3],3
    fffff800`0268635d 66c785800000000000 mov word ptr [rbp+80h],0
    fffff800`02686366 7405 je nt!KiInterruptDispatch+0x5d (fffff800`0268636d) Branch

    nt!KiInterruptDispatch+0x58:
    fffff800`02686368 e883470000 call nt!KiSaveDebugRegisterState (fffff800`0268aaf0) <– saves debug registers

    nt!KiInterruptDispatch+0x5d:
    fffff800`0268636d fc cld <- direction flag cleared
    fffff800`0268636e 0fae5dac stmxcsr dword ptr [rbp-54h]
    fffff800`02686372 650fae142580010000 ldmxcsr dword ptr gs:[180h]
    fffff800`0268637b 0f2945f0 movaps xmmword ptr [rbp-10h],xmm0
    fffff800`0268637f 0f294d00 movaps xmmword ptr [rbp],xmm1
    fffff800`02686383 0f295510 movaps xmmword ptr [rbp+10h],xmm2
    fffff800`02686387 0f295d20 movaps xmmword ptr [rbp+20h],xmm3
    fffff800`0268638b 0f296530 movaps xmmword ptr [rbp+30h],xmm4
    fffff800`0268638f 0f296d40 movaps xmmword ptr [rbp+40h],xmm5
    fffff800`02686393 488b0546502300 mov rax,qword ptr [nt!KiInterlockedPopEntrySListResumeEntryPoint (fffff800`028bb3e0)]
    fffff800`0268639a 483b85e8000000 cmp rax,qword ptr [rbp+0E8h]
    fffff800`026863a1 7319 jae nt!KiInterruptDispatch+0xac (fffff800`026863bc) Branch >>

    .. snip .. 

    nt!KiInterruptDispatch+0x159:
    fffff800`02686469 488b4e48 mov rcx,qword ptr [rsi+48h]
    fffff800`0268646d e83ea1ffff call nt!KeAcquireSpinLockAtDpcLevel (fffff800`026805b0) <– interrupt Spinlock
    fffff800`02686472 488bce mov rcx,rsi
    fffff800`02686475 488b5630 mov rdx,qword ptr [rsi+30h]
    fffff800`02686479 ff5118 call qword ptr [rcx+18h] >>> actual ISR is called

    kd> dt nt!_KINTERRUPT fffff80002c24000 ServiceRoutine
    +0x018 ServiceRoutine : 0xfffff800`02bf52bc unsigned char hal!HalpApicSpuriousService+0
    fffff800`0268647c 884588 mov byte ptr [rbp-78h],al
    fffff800`0268647f 488b4e48 mov rcx,qword ptr [rsi+48h]
    fffff800`02686483 e8a8ec0000 call nt!KeReleaseSpinLockFromDpcLevel (fffff800`02695130) <–--- release interrupt spinlock
    fffff800`02686488 f685f3000000ff test byte ptr [rbp+0F3h],0FFh
    fffff800`0268648f 741a je nt!KiInterruptDispatch+0x19b (fffff800`026864ab) Branch

    nt!KiInterruptDispatch+0x181:
    fffff800`02686491 4c8b4d50 mov r9,qword ptr [rbp+50h]
    fffff800`02686495 4c8b85e0000000 mov r8,qword ptr [rbp+0E0h]
    fffff800`0268649c 0fb65588 movzx edx,byte ptr [rbp-78h]
    fffff800`026864a0 8a7658 mov dh,byte ptr [rsi+58h]
    fffff800`026864a3 488bce mov rcx,rsi
    fffff800`026864a6 e8b5070900 call nt!PerfInfoLogInterrupt (fffff800`02716c60) <--- saves info for performance counter
    
    nt!KiInterruptDispatch+0x19b:
    fffff800`026864ab fa cli
    fffff800`026864ac 51 push rcx
    fffff800`026864ad 488b0d5c1c1300 mov rcx,qword ptr [nt!_imp_HalPerformEndOfInterrupt (fffff800`027b8110)]
    fffff800`026864b4 ff11 call qword ptr [rcx]
    fffff800`026864b6 59 pop rcx
    fffff800`026864b7 65488b0c2520000000 mov rcx,qword ptr gs:[20h]
    fffff800`026864c0 80792001 cmp byte ptr [rcx+20h],1
    fffff800`026864c4 7772 ja nt!KiInterruptDispatch+0x228 (fffff800`02686538) Branch

    nt!KiInterruptDispatch+0x1b6:
    fffff800`026864c6 0f31 rdtsc
    fffff800`026864c8 48c1e220 shl rdx,20h
    fffff800`026864cc 480bc2 or rax,rdx
    fffff800`026864cf 482b8140470000 sub rax,qword ptr [rcx+4740h]
    fffff800`026864d6 48018178470000 add qword ptr [rcx+4778h],rax
    fffff800`026864dd 48018140470000 add qword ptr [rcx+4740h],rax
    fffff800`026864e4 488b4108 mov rax,qword ptr [rcx+8]
    fffff800`026864e8 f6400204 test byte ptr [rax+2],4
    fffff800`026864ec 7414 je nt!KiInterruptDispatch+0x1f2 (fffff800`02686502) Branch
    
    nt!KiInterruptDispatch+0x1de:
    fffff800`026864ee 0f94c2 sete dl
    fffff800`026864f1 488bc8 mov rcx,rax
    fffff800`026864f4 e8f7ab0800 call nt!KiBeginCounterAccumulation (fffff800`027110f0) <---— probably to accumulate clock ticks
    fffff800`026864f9 65488b0c2520000000 mov rcx,qword ptr gs:[20h]
    
    nt!KiInterruptDispatch+0x1f2:
    fffff800`02686502 8a5106 mov dl,byte ptr [rcx+6]
    fffff800`02686505 80610600 and byte ptr [rcx+6],0
    fffff800`02686509 80790700 cmp byte ptr [rcx+7],0
    fffff800`0268650d 7529 jne nt!KiInterruptDispatch+0x228 (fffff800`02686538) Branch
    
    nt!KiInterruptDispatch+0x1ff:
    fffff800`0268650f 84d2 test dl,dl
    fffff800`02686511 7425 je nt!KiInterruptDispatch+0x228 (fffff800`02686538) Branch
    
    nt!KiInterruptDispatch+0x203:
    fffff800`02686513 807da902 cmp byte ptr [rbp-57h],2
    fffff800`02686517 730b jae nt!KiInterruptDispatch+0x214 (fffff800`02686524) Branch
    
    nt!KiInterruptDispatch+0x209:
    fffff800`02686519 80612000 and byte ptr [rcx+20h],0
    fffff800`0268651d e81eff0400 call nt!KiDpcInterruptBypass (fffff800`026d6440)
    fffff800`02686522 eb17 jmp nt!KiInterruptDispatch+0x22b (fffff800`0268653b) Branch
    
    nt!KiInterruptDispatch+0x214:
    fffff800`02686524 b902000000 mov ecx,2
    fffff800`02686529 ff15e91b1300 call qword ptr [nt!_imp_HalRequestSoftwareInterrupt (fffff800`027b8118)] <– DPC interrupt
    fffff800`0268652f 65488b0c2520000000 mov rcx,qword ptr gs:[20h]
    
    nt!KiInterruptDispatch+0x228:
    fffff800`02686538 fe4920 dec byte ptr [rcx+20h]
    
    nt!KiInterruptDispatch+0x22b:
    fffff800`0268653b 0fb64da9 movzx ecx,byte ptr [rbp-57h]
    fffff800`0268653f 440f22c1 mov cr8,rcx
    fffff800`02686543 488bb5d0000000 mov rsi,qword ptr [rbp+0D0h]
    fffff800`0268654a f685f000000001 test byte ptr [rbp+0F0h],1
    fffff800`02686551 0f84bb000000 je nt!KiInterruptDispatch+0x302 (fffff800`02686612) Branch
    
    nt!KiInterruptDispatch+0x247:
    fffff800`02686557 65488b0c2588010000 mov rcx,qword ptr gs:[188h]
    fffff800`02686560 80797a00 cmp byte ptr [rcx+7Ah],0
    fffff800`02686564 7419 je nt!KiInterruptDispatch+0x26f (fffff800`0268657f) Branch
    
    nt!KiInterruptDispatch+0x256:
    fffff800`02686566 b901000000 mov ecx,1
    fffff800`0268656b 440f22c1 mov cr8,rcx
    fffff800`0268656f fb sti
    fffff800`02686570 e85b7bffff call nt!KiInitiateUserApc (fffff800`0267e0d0) <<– if there were any APCs
    fffff800`02686575 fa cli
    fffff800`02686576 b900000000 mov ecx,0
    fffff800`0268657b 440f22c1 mov cr8,rcx
    
    nt!KiInterruptDispatch+0x26f:
    fffff800`0268657f 65488b0c2588010000 mov rcx,qword ptr gs:[188h]
    fffff800`02686588 f70100000240 test dword ptr [rcx],40020000h
    fffff800`0268658e 7425 je nt!KiInterruptDispatch+0x2a5 (fffff800`026865b5) Branch
    
    nt!KiInterruptDispatch+0x280:
    fffff800`02686590 f6410202 test byte ptr [rcx+2],2
    fffff800`02686594 740e je nt!KiInterruptDispatch+0x294 (fffff800`026865a4) Branch
    
    nt!KiInterruptDispatch+0x286:
    fffff800`02686596 e885d30900 call nt!KiCopyCounters (fffff800`02723920)
    fffff800`0268659b 65488b0c2588010000 mov rcx,qword ptr gs:[188h]
    
    nt!KiInterruptDispatch+0x294:
    fffff800`026865a4 f6410340 test byte ptr [rcx+3],40h
    fffff800`026865a8 740b je nt!KiInterruptDispatch+0x2a5 (fffff800`026865b5) Branch

    nt!KiInterruptDispatch+0x29a:
    fffff800`026865aa 488d6580 lea rsp,[rbp-80h]
    fffff800`026865ae b101 mov cl,1
    fffff800`026865b0 e80b3f0000 call nt!KiUmsExit (fffff800`0268a4c0)
    
    nt!KiInterruptDispatch+0x2a5:
    fffff800`026865b5 0fae55ac ldmxcsr dword ptr [rbp-54h]
    fffff800`026865b9 6683bd8000000000 cmp word ptr [rbp+80h],0
    fffff800`026865c1 7405 je nt!KiInterruptDispatch+0x2b8 (fffff800`026865c8) Branch

— Restoring registers.

    nt!KiInterruptDispatch+0x2b3:
    fffff800`026865c3 e8b8440000 call nt!KiRestoreDebugRegisterState (fffff800`0268aa80)

    nt!KiInterruptDispatch+0x2b8:
    fffff800`026865c8 0f2845f0 movaps xmm0,xmmword ptr [rbp-10h]
    fffff800`026865cc 0f284d00 movaps xmm1,xmmword ptr [rbp]
    fffff800`026865d0 0f285510 movaps xmm2,xmmword ptr [rbp+10h]
    fffff800`026865d4 0f285d20 movaps xmm3,xmmword ptr [rbp+20h]
    fffff800`026865d8 0f286530 movaps xmm4,xmmword ptr [rbp+30h]
    fffff800`026865dc 0f286d40 movaps xmm5,xmmword ptr [rbp+40h]
    fffff800`026865e0 4c8b5de0 mov r11,qword ptr [rbp-20h]
    fffff800`026865e4 4c8b55d8 mov r10,qword ptr [rbp-28h]
    fffff800`026865e8 4c8b4dd0 mov r9,qword ptr [rbp-30h]
    fffff800`026865ec 4c8b45c8 mov r8,qword ptr [rbp-38h]
    fffff800`026865f0 488b55c0 mov rdx,qword ptr [rbp-40h]
    fffff800`026865f4 488b4db8 mov rcx,qword ptr [rbp-48h]
    fffff800`026865f8 488b45b0 mov rax,qword ptr [rbp-50h]
    fffff800`026865fc 488be5 mov rsp,rbp
    fffff800`026865ff 488badd8000000 mov rbp,qword ptr [rbp+0D8h]
    fffff800`02686606 4881c4e8000000 add rsp,0E8h
    fffff800`0268660d 0f01f8 swapgs
    fffff800`02686610 48cf iretq  <-- IRET returns.

- By now the interrupt returns from its execution.

