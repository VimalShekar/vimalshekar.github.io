---
layout: post
category: Reverse-Engg
title: Kernel debugging a hyper-V VM
date: 2016-09-06
---

Now that you have the debugger installed – lets dig in and attach it to a virtual machine. In this post – I’m using a Windows 8 VM running on Hyper-V. Since it’s a generation 2 VM – secure boot is enabled by default. We have to turn that off so that the machine doesn’t complain when we make bcd changs and connect the debugger.


The easiest way to do this is through powershell:
>* Launch powershell as an admin.
>* If the VM’s State is not Off, then turn off the VM either through GUI or by using : `Stop-VM <VMName> -TurnOff`
>* Run: `Get-VM <VMName> | format-Table Name, State, Generation`

>* If the VM is a generation 2 VM, then check if Secure boot is turned off.
`Get-VMFirmware <VMName> | ft VMName, SecureBoot`

>* Turn off secure boot if it is enabled using:
`Set-VMFirmware <VMName> -EnableSecureBoot Off`

>* Set the Comport of the VM to a named pipe. You can then connect to this named pipe from windbg.
`Set-VMComport -VMName <VmName> -Number 1 -Path \\.\pipe\<PipeName>`

>* To verify: `Get-VMComport -VMName <VMName>`
>* Start the Virtual Machine: `Start-VM <VMName>`

# Enabling Debug mode within the VM
Once logged in to the VM, we need to ensure that debugging mode is ON and configure the COM port speed. Log into the VM, and then use one of these methods:
>* Go to Start > Run > msconfig.exe.
>* In the boot tab, click Advanced options.
>* Check the box for Debug and ensure that the selected transport is COM 1, and baud rate is set to 11500. Click OK and reboot when prompted.

Or launch an administrative command prompt on the VM and use the command line:
`bcdedit /debug on`
`bcdedit /dbgsettings serial debugport:1 baudrate:115200`


# Attaching the debugger:
>* On the host machine, launch WinDbg as an administrator and then go to File > Kernel Debug > Com.
>* Check the boxes for ‘Pipe’ and ‘Reconnect’ ; Also give the name of the named pipe mentioned earlier in the Set-VMComport command.

>* At this point, the debugger should show the following line:

        Opened \\.\pipe\WS2012Dbg
        Waiting to reconnect…

>* If it doesn’t work immediately, reboot the VM one more time. At this point, the debugger should show that the connection was established.

        Connected to Windows 8 9600 x64 target at (Fri Aug 21 11:53:37.853 2015 (UTC + 5:30)), ptr64 TRUE
        Kernel Debugger connection established.

        Press Ctrl+Break to break into the VM. At this point the debugger should show a message similar to the one shown below.
        Break instruction exception – code 80000003 (first chance)

        *******************************************************************************
        * *
        * You are seeing this message because you pressed either *
        * CTRL+C (if you run console kernel debugger) or, *
        * CTRL+BREAK (if you run GUI kernel debugger), *
        * on your debugger machine’s keyboard. *
        * *
        * THIS IS NOT A BUG OR A SYSTEM CRASH *
        * *
        * If you did not intend to break into the debugger, press the “g” key, then *
        * press the “Enter” key now. This message might immediately reappear. If it *
        * does, press “g” and “Enter” again. *
        * *
        *******************************************************************************
        nt!DbgBreakPointWithStatus:
        fffff802`69766c90 cc int 3



If you have gotten this far - Hurray – we’ve made our first break through!