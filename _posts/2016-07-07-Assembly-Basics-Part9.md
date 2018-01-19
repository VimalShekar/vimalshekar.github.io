---
layout: post
category: Reverse-Engg
title: Assembly Language Basics - Part 9
date: 2016-07-07
---

# Calling Conventions 
Software is developed as a collaborative effort, there has to be some set of standards and protocols defined between the API provider and the API consumer. Calling standards or conventions are the protocol established between caller function and the callee function. Calling standards cover aspects like
* Parameter passing
* Return values
* Register usage

In the first few posts, we already discussed register usages. For example, x86 calling standard states
> - ESP register should be used as a stack pointer
> - EBP should be used as a frame pointer
> - EAX should store the return value of a function
> - EAX, ECX, EDX – are volatile registers, other general purpose registers are non-volatile

In case of x86-64 - the ABI suggests :
> - The first four parameters are passed in RCX, RDX, R8 and R9. RAX is used to store the return values. 
> - The registers RAX, RCX, RDX, R8, R9, R10, R11 are considered volatile and must be considered destroyed on function calls.
> - The registers RBX, RBP, RDI, RSI, RSP, R12, R13, R14, and R15 are considered nonvolatile and must be saved and restored by a function that uses them.


Underlying hardware architecture specifies certain level of calling standards; however the operating system can deviate from the architecturally defined calling standards. Windows defines several calling conventions. The most common calling conventions are
* C call
* Std call
* This call
* Fast call
* CLR call

In this post, we'll discuss C call and STD call. For now we are dealing with a x86 bit machine. 

# C call (or __cdecl according to MSDN)
Default calling convention for C and C++ applications. Caller pushes arguments, executes call instruction, once the function has returned - caller cleans the stack.
> * Argument passing from right to left, arguments are pushed on the stack. Registers are not used.
> * Caller is responsible for cleaning up the stack for the parameters passed to the called function. Due to this images are comparatively larger.
> * Functions symbolized with “_” prefix when you look at them symbol table.

{% highlight cpp %}

// -- This function uses C call  (You can use __cdecl to tell the Microsoft compiler that this is a C call)
int __cdecl CDeclCall(int A, int B, char X, char Y, StructureX *Z)
{
        if(Z == NULL) {
        return 0;
        }
                
        Z->fNumber = A / B;
        Z->test = (X > Y) ? X : Y;
        return 1;
}

//-- Sample assembly from a caller function preparing to call 
x86!main+0x5f [34]:
   34 00f7118f 8d45e4          lea     eax,[ebp-1Ch]
   34 00f71192 50              push    eax	            <-- variable A pushed into the stack
   34 00f71193 0fb64dff        movzx   ecx,byte ptr [ebp-1]
   34 00f71197 51              push    ecx	            <-- variable B pushed into the stack
   34 00f71198 0fb655fe        movzx   edx,byte ptr [ebp-2]
   34 00f7119c 52              push    edx	            <-- variable X pushed into the stack
   34 00f7119d 8b45f8          mov     eax,dword ptr [ebp-8]
   34 00f711a0 50              push    eax	            <-- variable Y pushed into the stack
   34 00f711a1 8b4df4          mov     ecx,dword ptr [ebp-0Ch]
   34 00f711a4 51              push    ecx	            <--structure ptr variable Z pushed into the stack
   34 00f711a5 e85bfeffff      call    x86!ILT+0(_CDeclCall) (00f71005)  <-- call to the function>
   34 00f711aa 83c414          add     esp,14h              <--Stack Cleanup is being performed by caller in C call
   34 00f711ad 85c0            test    eax,eax
   34 00f711af 741f            je      x86!main+0xa0 (00f711d0)

//-- disassembly of the function
0:000:x86> uf x86!CDeclCall
x86!CDeclCall [66]:
   66 00f71080 55              push    ebp
   66 00f71081 8bec            mov     ebp,esp
   66 00f71083 51              push    ecx
   67 00f71084 837d1800        cmp     dword ptr [ebp+18h],0
   67 00f71088 7504            jne     x86!CDeclCall+0xe (00f7108e)

x86!CDeclCall+0xa [68]:
   68 00f7108a 33c0            xor     eax,eax <--Return value
   68 00f7108c eb3c            jmp     x86!CDeclCall+0x4a (00f710ca)

x86!CDeclCall+0xe [71]:
   71 00f7108e 8b4508          mov     eax,dword ptr [ebp+8]
   71 00f71091 99              cdq
   71 00f71092 f77d0c          idiv    eax,dword ptr [ebp+0Ch]
   71 00f71095 f30f2ac0        cvtsi2ss xmm0,eax
   71 00f71099 8b4518          mov     eax,dword ptr [ebp+18h]
   71 00f7109c f30f1100        movss   dword ptr [eax],xmm0
   72 00f710a0 0fbe4d10        movsx   ecx,byte ptr [ebp+10h]
   72 00f710a4 0fbe5514        movsx   edx,byte ptr [ebp+14h]
   72 00f710a8 3bca            cmp     ecx,edx
   72 00f710aa 7e09            jle     x86!CDeclCall+0x35 (00f710b5)

x86!CDeclCall+0x2c [72]:
   72 00f710ac 0fbe4510        movsx   eax,byte ptr [ebp+10h]
   72 00f710b0 8945fc          mov     dword ptr [ebp-4],eax
   72 00f710b3 eb07            jmp     x86!CDeclCall+0x3c (00f710bc)

x86!CDeclCall+0x35 [72]:
   72 00f710b5 0fbe4d14        movsx   ecx,byte ptr [ebp+14h]
   72 00f710b9 894dfc          mov     dword ptr [ebp-4],ecx

x86!CDeclCall+0x3c [72]:
   72 00f710bc 8b5518          mov     edx,dword ptr [ebp+18h]
   72 00f710bf 8a45fc          mov     al,byte ptr [ebp-4]
   72 00f710c2 884204          mov     byte ptr [edx+4],al
   73 00f710c5 b801000000      mov     eax,1 <--Return value

//Function epilog
x86!CDeclCall+0x4a [74]:
   74 00f710ca 8be5            mov     esp,ebp
   74 00f710cc 5d              pop     ebp
   74 00f710cd c3              ret

//-- Snapshot of the stack when inside the function
0:000:x86> dps 010afe20 <---- Current child EBP
010afe20  010afe60 <--previous ESP
010afe24  00f711aa x86!main+0x7a <-- Return Address of Caller function
010afe28  00000014 <--Param 1 A
010afe2c  0000000f <--Param 2 B
010afe30  00000047 <--Param 3 C
010afe34  00000042 <--Param 4 D
010afe38  010afe44 <--Param 5 StructX
010afe3c  00f75c77 
010afe40  010afe98
010afe44  00f72d30 
010afe48  29e141c5 
010afe4c  3f800000 


{% endhighlight %}



# STD call 
This is the convention used by all win32 API. Caller pushes arguments into the stack, executes call instruction, the callee cleans up the stack by executing RET N - where n is the number of bytes allocated by caller.
> * Argument passing from right to left, arguemnts are pushed on the stack. Registers are not used.
> * Called routine responsible for cleaning up the stack even for the parameters. Caller continues without any further actions.
> * Functions symbolized with “_” prefix and @<number of bytes for arguments> after the function name.

{% highlight cpp %}

//-- If you use the __stdcall prefix, Microsoft compiler treats it as a stdcall

int __stdcall StandardCall(int A, int B, char X, char Y, StructureX *Z)
{
	if(Z == NULL) {
	return 0;
	}
		
	Z->fNumber = A / B;
	Z->test = (X > Y) ? X : Y;
	return 1;
}

//-- here's the Caller function, pushing arguments before calling StandardCall(int A, int B, char X, char Y, StructureX *Z)

   29 00f71151 8d45ec          lea     eax,[ebp-14h]
   29 00f71154 50              push    eax 	            <--Variable A
   29 00f71155 0fb64dff        movzx   ecx,byte ptr [ebp-1]
   29 00f71159 51              push    ecx	            <--Variable B
   29 00f7115a 0fb655fe        movzx   edx,byte ptr [ebp-2]
   29 00f7115e 52              push    edx	            <--Variable X
   29 00f7115f 8b45f8          mov     eax,dword ptr [ebp-8]
   29 00f71162 50              push    eax 	            <--Variable Y
   29 00f71163 8b4df4          mov     ecx,dword ptr [ebp-0Ch]
   29 00f71166 51              push    ecx	            <--structure * Z
   29 00f71167 e8a3feffff      call    x86!ILT+10(_StandardCall (00f7100f)  <--- actual call
   29 00f7116c 85c0            test    eax,eax
   29 00f7116e 741f            je      x86!main+0x5f (00f7118f)

//-- Disassembly of the called function:
0:000:x86> uf x86!StandardCall
//Function Prolog
   55 00f71030 55              push    ebp
   55 00f71031 8bec            mov     ebp,esp
   55 00f71033 51              push    ecx
   56 00f71034 837d1800        cmp     dword ptr [ebp+18h],0
   56 00f71038 7504            jne     x86!StandardCall+0xe (00f7103e)

x86!StandardCall+0xa [57]:
   57 00f7103a 33c0            xor     eax,eax <--Return value 0
   57 00f7103c eb3c            jmp     x86!StandardCall+0x4a (00f7107a)

x86!StandardCall+0xe [60]:
   60 00f7103e 8b4508          mov     eax,dword ptr [ebp+8]
   60 00f71041 99              cdq
   60 00f71042 f77d0c          idiv    eax,dword ptr [ebp+0Ch]
   60 00f71045 f30f2ac0        cvtsi2ss xmm0,eax
   60 00f71049 8b4518          mov     eax,dword ptr [ebp+18h]
   60 00f7104c f30f1100        movss   dword ptr [eax],xmm0
   61 00f71050 0fbe4d10        movsx   ecx,byte ptr [ebp+10h]
   61 00f71054 0fbe5514        movsx   edx,byte ptr [ebp+14h]
   61 00f71058 3bca            cmp     ecx,edx
   61 00f7105a 7e09            jle     x86!StandardCall+0x35 (00f71065)

x86!StandardCall+0x2c [61]:
   61 00f7105c 0fbe4510        movsx   eax,byte ptr [ebp+10h]
   61 00f71060 8945fc          mov     dword ptr [ebp-4],eax
   61 00f71063 eb07            jmp     x86!StandardCall+0x3c (00f7106c)

x86!StandardCall+0x35 [61]:
   61 00f71065 0fbe4d14        movsx   ecx,byte ptr [ebp+14h]
   61 00f71069 894dfc          mov     dword ptr [ebp-4],ecx

x86!StandardCall+0x3c [61]:
   61 00f7106c 8b5518          mov     edx,dword ptr [ebp+18h]
   61 00f7106f 8a45fc          mov     al,byte ptr [ebp-4]
   61 00f71072 884204          mov     byte ptr [edx+4],al
   62 00f71075 b801000000      mov     eax,1 <-- Return value

//Function epilog
x86!StandardCall+0x4a [63]:
   63 00f7107a 8be5            mov     esp,ebp
   63 00f7107c 5d              pop     ebp
   63 00f7107d c21400          ret     14h     <--Stack Cleanup performed by Callee in StdCall

//-- Snapshot of the stack when inside the function
0:000:x86> dps 010afe20  <----current ChildEBP
010afe20  010afe60                  <--previous BP
010afe24  00f7116c x86!main+0x3c    <-- Return Address for stdcall()
010afe28  00000014 <--Param 1 A
010afe2c  0000000f <--Param 2 B
010afe30  00000047 <--Param 3 X
010afe34  00000042 <--Param 4 Y
010afe38  010afe4c <--Param 5 structX
010afe3c  00f75c77 
010afe40  010afe98
010afe44  00f72d30 

{% endhighlight %}

Thats it for now, I'll cover Fast call in the next post.