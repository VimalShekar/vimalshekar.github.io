---
layout: post
title: Activating the debugger as soon as the desired process launches its first thread
date: 2017-02-15
category: Walkthroughs
---

Generally when you are kernel debugging a machine, you may be in an arbitrary process's context. It is difficult to using the stepping commands and wait till the process of your interest starts. If you want to attach to a process or activate the debugger only when the process starts, there's an easier way. Watch for PspInsertProcess, and set a breakpoint on this function.

This function will get called after initialization of EPROCESS datastructure. This is basically inserting the process to the Global Kernel Process list. The first parameter will give you the EPROCESS pointer and the second last parameter will give you the Cmdline for the process launch.

    {% highlight cpp %}

    1: kd> dt nt!PspInsertProcess
    PspInsertProcess  long (
        _EPROCESS*, 
        _EPROCESS*, 
        unsigned long, 
        unsigned long, 
        void*, 
        unsigned long, 
        _UNICODE_STRING*, 
        _PSP_OBJECT_CREATION_STATE*)
        
        
    1: kd> bp nt!PspInsertProcess

    {% endhighlight %}

Now, the debugger will get activated for every new process that I launched on the debuggee. To set your break point on a specific process name, you can create a small script with the following lines:

Create a file called Debugger.txt which contains:

    {% highlight bash %}
    r $t0 = (rcx+448)
    as /ma ${/v:ImageName} @$t0 
    .if ($spat(@"${ImageName}", "*msiexec.exe*")) {kn} .else { g }
    {% endhighlight %}

Now, Set your breakpoint such that this script gets exected at each breakpoint. Use this syntax:
```
    bp  nt!NtCreateUserProcess "$$<C:\\Temp\\Debugger.txt"
```

Essentially - we've set a conditional breakpoint which runs the lines in the Debugger.txt script. This script checks the first parameter of the PspInsertProcess  function (which is in RCX) and then gets the  448th offset. The first parameter of this function is an EPROCESS object and the 448th offset of an EPROCESS contain the image name of the process. We do a pattern match to see if this image name matches our desired process name (in this case miexec.exe). If it matches, then the debugger breaks, else it continues execution.

Once the debugger breaks, you can get to the first thread by using this:

{% highlight cpp %}
    .process /r /p /i <Eprocess address>

    1: kd> bp /p <EProcessAddr> nt!PspInsertThread

    1: kd> g
    Breakpoint 0 hit
    nt!PspInsertThread:
    810e7d9e 68b0000000      push    0B0h

    1: kd> k
    # ChildEBP RetAddr  
    00 9f3ef728 8110882d nt!PspInsertThread 
    01 9f3ef8e4 810dc961 nt!PspCreateThread+0x213 
    02 9f3efd20 80f6bde7 nt!NtCreateThreadEx+0x187 
    03 9f3efd20 774ccb70 nt!KiSystemServicePostCall 
    04 0302f5c4 774cb97a ntdll!KiFastSystemCallRet 
    05 0302f5c8 74ea7d20 ntdll!NtCreateThreadEx+0xa 
    06 0302f7fc 751245e7 KERNELBASE!CreateRemoteThreadEx+0x1f0 
    07 0302f824 00bd4f83 KERNEL32!CreateThreadStub+0x27 
    08 0302f858 00bd5925 0xbd4f83
    09 0302fb28 00be382f 0xbd5925
    0a 0302fb6c 75124198 0xbe382f
    0b 0302fb80 774b601b KERNEL32!BaseThreadInitThunk+0x24
    0c 0302fbc8 774b5fe9 ntdll!__RtlUserThreadStart+0x2b
    0d 0302fbd8 00000000 ntdll!_RtlUserThreadStart+0x1b

{% endhighlight %}

And that’s the first thread in the process. 