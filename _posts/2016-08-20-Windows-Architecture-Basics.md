---
layout: post
category: Reverse-Engg
title: Windows Architecture basics
date: 2016-08-20
---

As I had mentioned earlier - I'll start a parallel post of articles related to Windows internals. Today, lets discuss the Windows architecture.

The Intel IA32 series of processors have 4 privilege levels : Ring 0, 1, 2 and 3. Ring 0 has the higest privilege and Ring 3 has the lowest privilege. Most operating systems (including Windows) that runs on this platform only make use of these two rings. Windows defines two modes of operation based on privilege – User mode and Kernel mode. 

> User mode is where applications and services run, this layer has lower privilege (ring 3) and can only use a subset of the instructions and memory available on the system.

> Kernel mode is where the kernel, and system drivers run. This layer has the higest privilege and the full instruction set and memory access.

The main components of the user mode and kernel mode are shown in the block below:
> ![Win-Arch]({{ "/images/win-arch.jpg" | absolute_url }})

# Kernel mode components

* Kernel:
> Implemented in the file ntoskrnl.exe and related files depending on CPU architecture and machine type. Provides the Core Mechanisms to be used by executive and other low level components. Mechanisms include thread scheduling, interrupt and exception handling, multi-processor synchronization, creation of kernel objects.

* Executive:
> Implemented in the same binary as the kernel. Contains several logical components that implement various policies for resource management and access by utilizing the core mechanisms exposed by the kernel. Some of the logical components are: Object manager, memory manager, cache manager, IO manager, Process and thread manager, Pnp manager,
Security reference monitor etc.

* Device drivers:
> Windows kernel is a hybrid kernel because it has user mode subsystems and services unlike other monolithic kernels. It does carry properties of monolithic kernel (such as allowing dynamic load/unload of modules). Device drivers are such modules that are created using WDK, and loaded as and when needed post initialization of the kernel. Drivers extend the functionality of the OS and provide more modular approach to handle different classes of devices.

* Hardware Abstraction layer or HAL:
> Hardware Abstraction layer (hal.dll) - enabled portability and extensibility across hardware platforms, by encapsulating the differences from the rest of the OS. HAL gives a uniform view of the h/w to the kernel and to device drivers.

* Windowing and Graphics sub-system:
> This is the subsystem that’s responsible for the visual effects of any Windows based application. This includes several APIs for creation and management of Windows and messaging mechanisms between Windows.

# User mode components
* User applications:
> Applications designed for users to help them with their tasks. They use APIs exposed by user32, kernel32, kernelbase, gdi32 and ntdll. Ex: Calc, Word, Photoshop etc.

* Windows Service processes:
> These are processes that run independently of any interactive user - typically under LocalSystem, LocalService or NetworkService. They run even if users are not logged into the system and are either hosted in a svchost.exe process or have their own executables. SCM starts/stops these as required. They use the same set of system call APIs that applications use to access services from the kernel.

* System support processes:
> These are processes that are critical and are required for the system to run. Termination of these
processes will cause failure/instability. Some of them are:

Lsass.exe - local System Authentication Subsystem
Services.exe - Service control manager
Winlogon.exe - Windows logon process
Csrss.exe - Client Server runtime Subsystem (Windows Subsystem)
Smss.exe - Session manager Subsystem

* Environment Subsystems:
> These are subsystems that enable operating environment for applications. Originally, NT designers wanted to make 16 bit MSDOS, win3.1 and POSIX applications work on NT. To achieve this they introduced subsystems that emulate these operating environments. Currently active ones are:
Windows subsystem (provided by csrss.exe/win32k)
Posix subsystem (psxss.exe is available as optional feature)
NTVDM - NT virtual DOS machine - 16 bit apps on 32 bit windows (not supported anymore
on x64 OS), OS/2 subsystem (not supported anymore)
WOW subsystem - 32 bit application on 64 bit windows    


* Subsystem libraries:
> These are dlls that an application links in order to be able to run on Windows. This includes run time APIs and those that transition to the kernel via System calls 


That completes today's post, hang around for more.