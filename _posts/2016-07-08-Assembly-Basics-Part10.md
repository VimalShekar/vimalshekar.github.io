---
layout: post
category: Reverse-Engg
title: Assembly Language Basics - Part 10
date: 2016-07-08
---

In the previous post, we looked at C call and Std call. Now we will discuss Fast call for both x86 and x64 architecture types.

# Fast call  
This is the default calling convention in x64 machines. It is sometimes used in x86 machines as well.
> * Calling convention uses registers to store arguments. In x86, the first two arguments are put into ECX and EDX and the rest are pushed into the stack from right to left. In x64 - the first 4 parameters are put in RCX, RDX, R8 and R9 and the remaining arguments are pushed into the stach from right to left.
> * Called routine responsible for cleaning up the stack, typically executing RET N or sub RSP, ## 
> * Functions decorated with @ prefix followed by number of bytes in parameters suffixed with @.

{% highlight cpp %}

//-- If you add the __fastcall prefix, Microsoft compiler treats it as fastcall.

int __fastcall FstCall(int A, int B, char X, char Y, StructureX* Z)
{
	if(Z == NULL) {
	return 0;
	}
		
	Z->fNumber = A / B;
	Z->test = (X > Y)? X : Y;
	
	return 1;
}

//-- snip from the caller function preparing for the call
//-- two arguments are passed in registers.
x86!main+0xa0 [39]:
   39 00f711d0 8d45dc          lea     eax,[ebp-24h] 	
   39 00f711d3 50              push    eax	                  <--Variable A
   39 00f711d4 0fb64dff        movzx   ecx,byte ptr [ebp-1]
   39 00f711d8 51              push    ecx	                  <--Variable B
   39 00f711d9 0fb655fe        movzx   edx,byte ptr [ebp-2]
   39 00f711dd 52              push    edx	                  <--Variable X
   39 00f711de 8b55f8          mov     edx,dword ptr [ebp-8]	<--Variable Y passed in register
   39 00f711e1 8b4df4          mov     ecx,dword ptr [ebp-0Ch]	<--structure ptr Z in register
   39 00f711e4 e82bfeffff      call    x86!ILT+15(FstCall (00f71014)
   39 00f711e9 85c0            test    eax,eax
   39 00f711eb 741f            je      x86!main+0xdc (00f7120c)

//-- disassembly of the fast call function
0:000:x86> uf x86!FstCall
x86!FstCall [78]:
   78 00f710d0 55              push    ebp
   78 00f710d1 8bec            mov     ebp,esp
   78 00f710d3 83ec0c          sub     esp,0Ch
   78 00f710d6 8955f4          mov     dword ptr [ebp-0Ch],edx
   78 00f710d9 894df8          mov     dword ptr [ebp-8],ecx
   79 00f710dc 837d1000        cmp     dword ptr [ebp+10h],0
   79 00f710e0 7504            jne     x86!FstCall+0x16 (00f710e6)

x86!FstCall+0x12 [80]:
   80 00f710e2 33c0            xor     eax,eax <-Return value
   80 00f710e4 eb3c            jmp     x86!FstCall+0x52 (00f71122)

x86!FstCall+0x16 [83]:
   83 00f710e6 8b45f8          mov     eax,dword ptr [ebp-8]
   83 00f710e9 99              cdq
   83 00f710ea f77df4          idiv    eax,dword ptr [ebp-0Ch]
   83 00f710ed f30f2ac0        cvtsi2ss xmm0,eax
   83 00f710f1 8b4510          mov     eax,dword ptr [ebp+10h]
   83 00f710f4 f30f1100        movss   dword ptr [eax],xmm0
   84 00f710f8 0fbe4d08        movsx   ecx,byte ptr [ebp+8]
   84 00f710fc 0fbe550c        movsx   edx,byte ptr [ebp+0Ch]
   84 00f71100 3bca            cmp     ecx,edx
   84 00f71102 7e09            jle     x86!FstCall+0x3d (00f7110d)

x86!FstCall+0x34 [84]:
   84 00f71104 0fbe4508        movsx   eax,byte ptr [ebp+8]
   84 00f71108 8945fc          mov     dword ptr [ebp-4],eax
   84 00f7110b eb07            jmp     x86!FstCall+0x44 (00f71114)

x86!FstCall+0x3d [84]:
   84 00f7110d 0fbe4d0c        movsx   ecx,byte ptr [ebp+0Ch]
   84 00f71111 894dfc          mov     dword ptr [ebp-4],ecx

x86!FstCall+0x44 [84]:
   84 00f71114 8b5510          mov     edx,dword ptr [ebp+10h]
   84 00f71117 8a45fc          mov     al,byte ptr [ebp-4]
   84 00f7111a 884204          mov     byte ptr [edx+4],al
   86 00f7111d b801000000      mov     eax,1 <--Return value

x86!FstCall+0x52 [87]:
   87 00f71122 8be5            mov     esp,ebp
   87 00f71124 5d              pop     ebp
   87 00f71125 c20c00          ret     0Ch   <--Stack Cleanup by the callee in FastCall

//-- Snapshot of the stack when inside the function
0:000:x86> dps 010afe28
010afe28  00f711cd 
010afe2c  00f711e9 x86!main+0xb9  <--Return Address
010afe30  00000047 <--Param 3 D
010afe34  00000042 <--Param 4 C
010afe38  010afe3c <--Param 5 StructX
010afe3c  00f75c77 
010afe40  010afe98 

{% endhighlight %}

X64 ABI defines support for only fase calling convention. The first four parameters in RCX, RDX, R8 and R9. Further parameters are passed on the stack. The x64 Application Binary Interface also forces compiler writers to create 1:1 stack backing for each argument in a function. The x64 architecture provides for 16 general-purpose registers as well as 16 XMM registers available for floating-point use. The following are some of the rules in x64 calling convention:

> * X64 only supports fast call convention. The first four parameters are passed in RCX, RDX, R8 and R9. If arguments are float/double – they are passed in XMM0L, XMM1L, XMM2L, XMM3L. Aggregates (structures/classes) if > 64 bits is passed as
a pointer.
> * A scalar return value that can fit into 64 bits is returned through RAX. Non-scalar types including floats, doubles are returned in XMM0
> * Caller allocates space on the stack for parameters to the callee. The x64 spec also states that the caller should allocate backing space(parameter homing space) for parameters passed through registers, the callee expects that. The actual registers
may or may not be stored in the homing area – that depends on the caller and the callee’s prolog.
> * A function's prolog is responsible for allocating stack space for local variables, saved registers, stack parameters, and register parameters. It has to pre-book space for parameters that this function may send, when it calls other functions –
called the stack params area. This is required for the debugger to rebuild the stack in the absence of frame pointer.
> * The registers RAX, RCX, RDX, R8, R9, R10, R11 are considered volatile and must be considered destroyed on function calls (unless otherwise safety-provable by analysis such as whole program optimization).
> * The registers RBX, RBP, RDI, RSI, RSP, R12, R13, R14, and R15 are considered nonvolatile and must be saved and restored by a function that uses them.

Volatile registers are scratch registers that the caller assumes - to be destroyed across a call. The registers RAX, RCX, RDX, R8, R9, R10, R11 are considered volatile by the caller and may be considered destroyed on function calls (unless otherwise safety-provable by analysis such as whole program optimization).
	
Nonvolatile registers are those that the callee must maintain - so that the callers values are alive across a function call. Callee can save Non volatile registers if they are used within the function. 

Here's a table summarizing their usage:

|Register	| Status |	Use |
|-----|-----|-----|
|RAX |	Volatile | 	Return value register  |
|RCX |	Volatile | 	First integer argument  |
|RDX |	Volatile | 	Second integer argument  |
|R8 |	Volatile | 	Third integer argument  |
|R9 |	Volatile | 	Fourth integer argument  |
|R10:R11 |	Volatile | 	Must be preserved as needed by caller; used in syscall/sysret instructions  |
|R12:R15 |	Nonvolatile | 	Must be preserved by callee  |
|RDI |	Nonvolatile | 	Must be preserved by callee  |
|RSI |	Nonvolatile | 	Must be preserved by callee  |
|RBX| 	Nonvolatile | 	Must be preserved by callee  |
|RBP| 	Nonvolatile | 	May be used as a frame pointer; must be preserved by callee  |
|RSP |	Nonvolatile | 	Stack pointer  |
|XMM0 |	Volatile | 	First FP argument  |
|XMM1 |	Volatile | 	Second FP argument  |
|XMM2 |	Volatile | 	Third FP argument  |
|XMM3 |	Volatile | 	Fourth FP argument  |
|XMM4:XMM5 |	Volatile | 	Must be preserved as needed by caller  |
|XMM6:XMM15 |	Nonvolatile | 	Must be preserved as needed by callee.  |
	
	
All memory addresses > RSP is volatile and callees should not write here. Function prolog allocates space on stack for local variables, saved registers, and stack based parameters and register parameter's backing store. Number of space allocated = 4 or the maximum space required by any function calls made within this function. For c++, this pointer is always passed through RCX.


{% highlight cpp %}

//-- here is a sample function call compiled for x64 
int RegularCall(int A, int B, char X, char Y, StructureX *Z)
{
	if(Z == NULL) {
	return 0;
	}
		
	Z->fNumber = A / B;
	Z->test = (X > Y) ? X : Y;
	return 1;
}

//-- snip from the caller:

   28 00007ff6`1c8a11d3 488d442440      lea     rax,[rsp+40h]
   28 00007ff6`1c8a11d8 4889442420      mov     qword ptr [rsp+20h],rax  	<-- Param5 in Stack
   28 00007ff6`1c8a11dd 440fb64c2430    movzx   r9d,byte ptr [rsp+30h]  	<-- Param4 in R9
   28 00007ff6`1c8a11e3 440fb6442431    movzx   r8d,byte ptr [rsp+31h] 	      <-- Param3 in R8
   28 00007ff6`1c8a11e9 8b542434        mov     edx,dword ptr [rsp+34h] 	<-- Param2 in RDX
   28 00007ff6`1c8a11ed 8b4c2438        mov     ecx,dword ptr [rsp+38h] 	<-- Param1 in RCX
   28 00007ff6`1c8a11f1 e81efeffff      call    x64!ILT+15(RegularCall) (00007ff6`1c8a1014)  
   28 00007ff6`1c8a11f6 85c0            test    eax,eax
   28 00007ff6`1c8a11f8 7422            je      x64!main+0x6c (00007ff6`1c8a121c)

//-- disassembly of the called function.
0:000> uf x64!RegularCall
//prolog 
x64!RegularCall [54]:
   54 00007ff6`1c8a1030 44884c2420      mov     byte ptr [rsp+20h],r9b<--backing
   54 00007ff6`1c8a1035 4488442418      mov     byte ptr [rsp+18h],r8b<--backing
   54 00007ff6`1c8a103a 89542410        mov     dword ptr [rsp+10h],edx<--backing
   54 00007ff6`1c8a103e 894c2408        mov     dword ptr [rsp+8],ecx<--backing
   54 00007ff6`1c8a1042 4883ec18        sub     rsp,18h <--allocation for local vars
   55 00007ff6`1c8a1046 48837c244000    cmp     qword ptr [rsp+40h],0
   55 00007ff6`1c8a104c 7504            jne     x64!RegularCall+0x22 (00007ff6`1c8a1052)

x64!RegularCall+0x1e [56]:
   56 00007ff6`1c8a104e 33c0            xor     eax,eax
   56 00007ff6`1c8a1050 eb47            jmp     x64!RegularCall+0x69 (00007ff6`1c8a1099)

x64!RegularCall+0x22 [59]:
   59 00007ff6`1c8a1052 8b442420        mov     eax,dword ptr [rsp+20h]
   59 00007ff6`1c8a1056 99              cdq
   59 00007ff6`1c8a1057 f77c2428        idiv    eax,dword ptr [rsp+28h]
   59 00007ff6`1c8a105b f30f2ac0        cvtsi2ss xmm0,eax
   59 00007ff6`1c8a105f 488b442440      mov     rax,qword ptr [rsp+40h]
   59 00007ff6`1c8a1064 f30f1100        movss   dword ptr [rax],xmm0
   60 00007ff6`1c8a1068 0fbe442430      movsx   eax,byte ptr [rsp+30h]
   60 00007ff6`1c8a106d 0fbe4c2438      movsx   ecx,byte ptr [rsp+38h]
   60 00007ff6`1c8a1072 3bc1            cmp     eax,ecx
   60 00007ff6`1c8a1074 7e0a            jle     x64!RegularCall+0x50 (00007ff6`1c8a1080)

x64!RegularCall+0x46 [60]:
   60 00007ff6`1c8a1076 0fbe442430      movsx   eax,byte ptr [rsp+30h]
   60 00007ff6`1c8a107b 890424          mov     dword ptr [rsp],eax
   60 00007ff6`1c8a107e eb08            jmp     x64!RegularCall+0x58 (00007ff6`1c8a1088)

x64!RegularCall+0x50 [60]:
   60 00007ff6`1c8a1080 0fbe442438      movsx   eax,byte ptr [rsp+38h]
   60 00007ff6`1c8a1085 890424          mov     dword ptr [rsp],eax

x64!RegularCall+0x58 [60]:
   60 00007ff6`1c8a1088 488b442440      mov     rax,qword ptr [rsp+40h]
   60 00007ff6`1c8a108d 0fb60c24        movzx   ecx,byte ptr [rsp]
   60 00007ff6`1c8a1091 884804          mov     byte ptr [rax+4],cl
   61 00007ff6`1c8a1094 b801000000      mov     eax,1

//epilog area - no fixed allocation in this function.
x64!RegularCall+0x69 [62]:
   62 00007ff6`1c8a1099 4883c418        add     rsp,18h     <--cleanup performed by the callee
   62 00007ff6`1c8a109d c3              ret


//-- Snapshot of the stack when inside the function
0:000> dps 00000082`7b6ffe30 
00000082`7b6ffe30  00007ff6`1c8c4458 x64!pinit
00000082`7b6ffe38  00000000`00000000
00000082`7b6ffe40  00000000`00000001
00000082`7b6ffe48  00007ff6`1c8a11f6 x64!main+0x46<- Return Address for current call 
00000082`7b6ffe50  00007ff6`00000014 <--Param1(backed by callee into the backing area, this wasn't pushed by caller)
00000082`7b6ffe58  00000000`0000000f <--Param2(backed by callee into the backing area, this wasn't pushed by caller)
00000082`7b6ffe60  00000000`00000047 <--Param3(backed by callee into the backing area, this wasn't pushed by caller)
00000082`7b6ffe68  00000000`00000042 <--param4(backed by callee into the backing area, this wasn't pushed by caller)
00000082`7b6ffe70  00000082`7b6ffe90 <--Param5 &structS passed to function
00000082`7b6ffe78  00007ff6`1c8a5335 
00000082`7b6ffe80  0000000f`00004742 

{% endhighlight %}


# This call  ( __thiscall )
This is the default calling convention for C++ member functions. Class member functions needs a mechanism to know the “this” pointer at any point in time. This convention makes sure that when a member function is called, this pointer be implicitly passed per the programmer.
> * The this pointer is passed in ECX register. (Variation to this is COMCALL, in which the “this” pointer is passed on the stack). Remaining arguments are passed on the stack.
> * Called routine cleans the stack. 

We'll do the stack walk of this in a later post.

Bye for now.
