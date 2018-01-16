---
layout: post
category: Reverse-Engg
title: Debugged a hung net.exe process with public symbols
date: 2017-05-11
---

I have a batch script to parse a csv file and create users on the local machine using net.exe. I typically schedule the script to run under system account once a day using the task scheduler. Recently I made some changes to my user and password name generation script and started seeing the script misbehave. The script completes, but no users were created. When I looked at the task manager I see several instances of net.exe processes with command line argument showing details of each of the 5 users I was trying to create.

As usual – my curiosity aroused, and I took a dump of the process and decided to examine it. Here’s my debug notes, in case you find it useful to solve trifling problems with windbg.

I took a dump of the process and examined its threads:

{% highlight asm %}
Original process:
0:000> |
.  0          id: a4c   examine               name: C:\Windows\System32\net.exe
 
0:000> ~*k
 
.  0  Id: a4c.7f4 Suspend: 0 Teb: 000007ff`fffde000 Unfrozen
# Child-SP          RetAddr           Call Site
00 00000000`0010f8b8 000007fe`fd6e10ac ntdll!ZwWaitForSingleObject+0xa
01 00000000`0010f8c0 00000000`ff8a20cf KERNELBASE!WaitForSingleObjectEx+0x79
02 00000000`0010f960 00000000`ff8a2424 net!call_net1+0x163  <-------- creates a net1 process with the following
03 00000000`0010fa50 00000000`ff8a61bc net!xxaction+0x188
04 00000000`0010fa90 00000000`ff8a6275 net!xx_parser+0x1ac
05 00000000`0010fae0 00000000`ff8a1d93 net!xx_parser+0x265
06 00000000`0010fb30 00000000`ff8a8939 net!main+0x33b
07 00000000`0010fba0 00000000`7749652d net!DelayLoadFailureHook+0x19b
08 00000000`0010fbe0 00000000`776cc521 kernel32!BaseThreadInitThunk+0xd
09 00000000`0010fc10 00000000`00000000 ntdll!RtlUserThreadStart+0x1d
 
0:000> !peb
PEB at 000007fffffdb000
    InheritedAddressSpace:    No
    ReadImageFileExecOptions: No
    BeingDebugged:            No
<snip>
    ImageFile:    'C:\Windows\system32\net.exe'
    CommandLine:  '"C:\Windows\system32\net.exe" user f201 pristineWoods22 /add /active:yes '
    DllPath:      'C:\Windows\system32;;C:\Windows\system32;C:\Windows\system;C:\Windows;.;C:\Windows\system32;C:\Windows;C:\Windows\System32\Wbem;C:\Windows\System32\WindowsPowerShell\v1.0\;'
 
0:000> ub 00000000`ff8a2424
net!xxaction+0x174:
00000000`ff8a2410 142b            adc     al,2Bh
00000000`ff8a2412 cb              retf
00000000`ff8a2413 740a            je      net!xxaction+0x183 (00000000`ff8a241f)
00000000`ff8a2415 3bcb            cmp     ecx,ebx
00000000`ff8a2417 7544            jne     net!xxaction+0x1c1 (00000000`ff8a245d)
00000000`ff8a2419 e84efbffff      call    net!call_net1 (00000000`ff8a1f6c)
00000000`ff8a241e cc              int     3
00000000`ff8a241f e848fbffff      call    net!call_net1 (00000000`ff8a1f6c) >> expanding this call
 
0:000> uF 00000000`ff8a1f6c
net!call_net1:
00000000`ff8a1f6c 488bc4          mov     rax,rsp
00000000`ff8a1f6f 48895818        mov     qword ptr [rax+18h],rbx
00000000`ff8a1f73 57              push    rdi
00000000`ff8a1f74 4881ece0000000  sub     rsp,0E0h
00000000`ff8a1f7b 83600800        and     dword ptr [rax+8],0
00000000`ff8a1f7f bf68000000      mov     edi,68h
00000000`ff8a1f84 488d4888        lea     rcx,[rax-78h]
00000000`ff8a1f88 4c8bc7          mov     r8,rdi
00000000`ff8a1f8b 33d2            xor     edx,edx
00000000`ff8a1f8d e802720000      call    net!memset (00000000`ff8a9194)
00000000`ff8a1f92 897c2470        mov     dword ptr [rsp+70h],edi
00000000`ff8a1f96 ff15acf1ffff    call    qword ptr [net!_imp_GetCommandLineW (00000000`ff8a1148)]   gets the net.exe process’s command line
00000000`ff8a1f9c 488bc8          mov     rcx,rax
00000000`ff8a1f9f ba20000000      mov     edx,20h
00000000`ff8a1fa4 488bf8          mov     rdi,rax
00000000`ff8a1fa7 ff15abf3ffff    call    qword ptr [net!_imp_wcschr (00000000`ff8a1358)]
00000000`ff8a1fad 4883c9ff        or      rcx,0FFFFFFFFFFFFFFFFh
00000000`ff8a1fb1 488d9424f8000000 lea     rdx,[rsp+0F8h]
00000000`ff8a1fb9 488bd8          mov     rbx,rax
00000000`ff8a1fbc 33c0            xor     eax,eax
00000000`ff8a1fbe 66f2af          repne scas word ptr [rdi]
00000000`ff8a1fc1 48f7d1          not     rcx
00000000`ff8a1fc4 488db909010000  lea     rdi,[rcx+109h]
00000000`ff8a1fcb 8d0c3f          lea     ecx,[rdi+rdi]
00000000`ff8a1fce e8716f0000      call    net!NetApiBufferAllocate (00000000`ff8a8f44)
00000000`ff8a1fd3 85c0            test    eax,eax
00000000`ff8a1fd5 7414            je      net!call_net1+0x7f (00000000`ff8a1feb)  Branch
 
 
0:000> uF /c net!call_net1
net!call_net1 (00000000`ff8a1f6c)
  net!call_net1+0x21 (00000000`ff8a1f8d):
    call to net!memset (00000000`ff8a9194)
  net!call_net1+0x2a (00000000`ff8a1f96):
    call to kernel32!GetCommandLineWStub (00000000`7749c480)
  net!call_net1+0x3b (00000000`ff8a1fa7):
    call to msvcrt!wcschr (000007fe`fdfc193c)
  net!call_net1+0x62 (00000000`ff8a1fce):
    call to net!NetApiBufferAllocate (00000000`ff8a8f44)
  net!call_net1+0x6f (00000000`ff8a1fdb):
    call to net!ErrorPrint (00000000`ff8a5818)
  net!call_net1+0x79 (00000000`ff8a1fe5):
    call to net!NetcmdExit (00000000`ff8a5934)
  net!call_net1+0x8c (00000000`ff8a1ff8):
    call to kernel32!GetSystemDirectoryWStub (00000000`77497120)
  net!call_net1+0x98 (00000000`ff8a2004):
    call to kernel32!GetLastErrorStub (00000000`774a2dd0)
  net!call_net1+0xa2 (00000000`ff8a200e):
    call to net!ErrorPrint (00000000`ff8a5818)
  net!call_net1+0xac (00000000`ff8a2018):
    call to net!NetcmdExit (00000000`ff8a5934)
  net!call_net1+0xc9 (00000000`ff8a2035):
    call to net!wcscpy_s (00000000`ff8a8aee)
  net!call_net1+0xe1 (00000000`ff8a204d):
    call to net!wcscat_s (00000000`ff8a8ae2)
  net!call_net1+0x126 (00000000`ff8a2092):
    call to kernel32!CreateProcessW (00000000`774a1bb0)  ----- this is where net1.exe is spawned!
  net!call_net1+0x130 (00000000`ff8a209c):
    call to kernel32!GetLastErrorStub (00000000`774a2dd0)
  net!call_net1+0x13a (00000000`ff8a20a6):
    call to net!ErrorPrint (00000000`ff8a5818)
  net!call_net1+0x144 (00000000`ff8a20b0):
    call to net!NetcmdExit (00000000`ff8a5934)
  net!call_net1+0x14f (00000000`ff8a20bb):
    call to kernel32!CloseHandleImplementation (00000000`774a2f80)
  net!call_net1+0x15d (00000000`ff8a20c9):
    call to kernel32!WaitForSingleObject (00000000`774a2b20)  - this is where we wait on the handle of the newly created process.  This is where we are in frame 2
  net!call_net1+0x170 (00000000`ff8a20dc):
    call to kernel32!GetExitCodeProcessImplementation (00000000`774912b0)
  net!call_net1+0x17b (00000000`ff8a20e7):
    call to kernel32!CloseHandleImplementation (00000000`774a2f80)
  net!call_net1+0x188 (00000000`ff8a20f4):
    call to net!MyExit (00000000`ff8a1df4)
  net!xxcondition+0x4d (00000000`ff8a214d):
    call to net!IsDeviceName (00000000`ff8a6778)
  net!xxcondition+0x59 (00000000`ff8a2159):
    call to net!IsWildCard (00000000`ff8a6910)
  net!xxcondition+0xa5 (00000000`ff8a21a5):
    call to net!ValidateSwitches (00000000`ff8a6964)
  net!xxcondition+0xb2 (00000000`ff8a21b2):
    call to net!IsDeviceName (00000000`ff8a6778)
  net!xxcondition+0xf6 (00000000`ff8a21f6):
    call to net!IsWildCard (00000000`ff8a6910)
  net!xxcondition+0x100 (00000000`ff8a2200):
    call to net!IsDeviceName (00000000`ff8a6778)
  net!xxcondition+0x112 (00000000`ff8a2212):
    call to net!IsWildCard (00000000`ff8a6910)
  net!xxcondition+0x142 (00000000`ff8a2242):
    call to net!NetpwNameValidate (00000000`ff8a8f68)
  net!xxcondition+0x158 (00000000`ff8a2258):
   call to net!IsQualifiedUsername (00000000`ff8a67e0)
  net!xxcondition+0x16f (00000000`ff8a226f):
    call to net!NetpwNameValidate (00000000`ff8a8f68)
  net!xxcondition+0x180 (00000000`ff8a2280):
    call to net!IsResource (00000000`ff8a66cc)

{% endhighlight %}
So, net.exe seems to be doing its job, it actually spawns a child process. next I dumped the child – net1.exe process 
(this is for the other user — so the command line shows f205 and corresponding password)


{% highlight asm %}
0:000> |
.  0          id: 68c   examine               name: C:\Windows\System32\net1.exe
 
0:000> !peb
PEB at 000007fffffdf000
<snip>
CommandLine:  'C:\Windows\system32\net1 user f205 pikachuVsRaichu&&Ash22 /add /active:yes '
<snip>
 
0:000> ~*k
.  0  Id: 68c.8d8 Suspend: 0 Teb: 000007ff`fffdd000 Unfrozen
# Child-SP          RetAddr           Call Site
00 00000000`001ff228 00000000`774a2f58 ntdll!NtRequestWaitReplyPort+0xa
01 00000000`001ff230 00000000`774d5321 kernel32!ConsoleClientCallServer+0x54
02 00000000`001ff260 00000000`774ea65c kernel32!ReadConsoleInternal+0x1f1
03 00000000`001ff3b0 00000000`ff2a3d0e kernel32!ReadConsoleW+0xbc
04 00000000`001ff490 00000000`ff2a7025 net1!GetString+0x56
05 00000000`001ff510 00000000`ff2a373f net1!LUI_YorNIns+0x221
06 00000000`001ff870 00000000`ff29ec52 net1!YorN+0x27
07 00000000`001ff8a0 00000000`ff297ccc net1!user_add+0x1ce
08 00000000`001ffb80 00000000`ff2a5fec net1!xxaction+0x860
09 00000000`001ffbb0 00000000`ff2a60a5 net1!xx_parser+0x1ac
0a 00000000`001ffc00 00000000`ff295c43 net1!xx_parser+0x265
0b 00000000`001ffc50 00000000`ff2a9aa1 net1!main+0x33b
0c 00000000`001ffcc0 00000000`7749652d net1!LsaFreeMemory+0x199
0d 00000000`001ffd00 00000000`776cc521 kernel32!BaseThreadInitThunk+0xd
0e 00000000`001ffd30 00000000`00000000 ntdll!RtlUserThreadStart+0x1d
 
0:000> .frame /r 7
07 00000000`001ff8a0 00000000`ff297ccc net1!user_add+0x1ce
rax=00000000001ff3f0 rbx=0000000000000002 rcx=0000000000000003
rdx=00000000001ff6a0 rsi=00000000ff2920b2 rdi=000000000039e434
rip=00000000ff29ec52 rsp=00000000001ff8a0 rbp=0000000000000001
r8=0000000000000001  r9=00000000001ff518 r10=0000000000000f54
r11=00000000001ff3f0 r12=00000000ff2bdee0 r13=000000000039e3fc
r14=000000000039e406 r15=0000000000000000
iopl=0         nv up ei pl zr na po nc
cs=0033  ss=002b  ds=002b  es=002b  fs=0053  gs=002b             efl=00000246
net1!user_add+0x1ce:
00000000`ff29ec52 85c0            test    eax,eax
 
0:000> ub 00000000`ff297ccc
net1!xxaction+0x845:
00000000`ff297cb1 33c9            xor     ecx,ecx
00000000`ff297cb3 e8ecc30000      call    net1!help_help (00000000`ff2a40a4)
00000000`ff297cb8 cc              int     3
00000000`ff297cb9 488b5210        mov     rdx,qword ptr [rdx+10h]
00000000`ff297cbd eb85            jmp     net1!xxaction+0x7d8 (00000000`ff297c44)
00000000`ff297cbf 488b5210        mov     rdx,qword ptr [rdx+10h]   parameter 2
00000000`ff297cc3 488b4f08        mov     rcx,qword ptr [rdi+8]   parameter 1
00000000`ff297cc7 e8b86d0000      call    net1!user_add (00000000`ff29ea84)
 
0:000> u net1!user_add 
net1!user_add:
00000000`ff29ea84 48895c2418      mov     qword ptr [rsp+18h],rbx
00000000`ff29ea89 48896c2420      mov     qword ptr [rsp+20h],rbp
00000000`ff29ea8e 56              push    rsi
00000000`ff29ea8f 57              push    rdi   backed up in the stack
00000000`ff29ea90 4154            push    r12
00000000`ff29ea92 4155            push    r13
00000000`ff29ea94 4156            push    r14
00000000`ff29ea96 4881ecb0020000  sub     rsp,2B0h
<snip>
0:000> ub ff29ec52 
net1!user_add+0x1b3:
00000000`ff29ec37 66f2af          repne scas word ptr [rdi]
00000000`ff29ec3a 48f7d1          not     rcx
00000000`ff29ec3d 482bcd          sub     rcx,rbp
00000000`ff29ec40 4883f90e        cmp     rcx,0Eh
00000000`ff29ec44 7618            jbe     net1!user_add+0x1da (00000000`ff29ec5e)
00000000`ff29ec46 8bd5            mov     edx,ebp
00000000`ff29ec48 b9a0140000      mov     ecx,14A0h  ------- we’re prompting for a Y or N question from the user because of this error code
00000000`ff29ec4d e8c64a0000      call    net1!YorN (00000000`ff2a3718)
 
0:000> !error 14A0h
Error code: (Win32) 0x14a0 (5280) - The password entered is longer than 14 characters.  Computers  with Windows prior to Windows 2000 will not be able to use  this account. Do you want to continue this operation? %1:
 
0:000> kn
# Child-SP          RetAddr           Call Site
00 00000000`001ff228 00000000`774a2f58 ntdll!NtRequestWaitReplyPort+0xa
01 00000000`001ff230 00000000`774d5321 kernel32!ConsoleClientCallServer+0x54
02 00000000`001ff260 00000000`774ea65c kernel32!ReadConsoleInternal+0x1f1 <----this prompts the question and is waiting for an input from the user
03 00000000`001ff3b0 00000000`ff2a3d0e kernel32!ReadConsoleW+0xbc
04 00000000`001ff490 00000000`ff2a7025 net1!GetString+0x56
05 00000000`001ff510 00000000`ff2a373f net1!LUI_YorNIns+0x221
06 00000000`001ff870 00000000`ff29ec52 net1!YorN+0x27    
07 00000000`001ff8a0 00000000`ff297ccc net1!user_add+0x1ce
08 00000000`001ffb80 00000000`ff2a5fec net1!xxaction+0x860
{% endhighlight %}

Hmmm… So, when the passwords are > 14 characters – net1.exe shows a prompt and asks for a confirmation. Too bad my script was being launched  by the task scheduler under the LOCAL SYSTEM account. Since that session was non-interactive, nobody sees or responds to the prompt. That’s  why net1.exe appears to be hung!

Mitigation: Well, I can live with local accounts whose passwords are less than 14 characters. This is a test system anyway!