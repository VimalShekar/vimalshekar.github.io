---
layout: post
category: WindowsInternals
title: The Windows Boot Process Part 1
date: 2016-01-22
---

Recently I was going through my notes from when I was a support engineer for Windows and found some nice goodies to share - much of it may be legacy at this point though, but still worth recording - so here goes. 

Let's look at the boot process of a legacy OS (Windows XP) running on a traditional BIOS based computer system.

1. **Power-On and POST:**
When you switch on a BIOS based computer, the power supply sends a "Power Good" signal to the CPU, and it begins executing a program called POST (Power-On Self Test). This is stored in EEPROM/firmware stored on the motherboard. POST conducts a series of self-tests to verify that essential hardware components (RAM, CPU, video adapter, etc.) are working properly. If an error is detected—for example, via beep codes—the boot process will halt until the issue is resolved. 
<br><br> Note: At this stage, no operating system files are involved yet; this is purely hardware initialization.


2. **BIOS & MBR:**
The BIOS checks CMOS settings (which store boot order preferences) to determine which device (hard disk, floppy, CD/DVD, etc.) to boot from. Next it loads the boot sectors(usually the first sector) of that boot device into memory and passes control to it. The very first sector of the boot device typically contains the Master Boot Record (MBR). This 512-byte sector (with roughly 446 bytes dedicated to boot loader code) contains the partition table and minimal code for starting the boot process. <br> <br>The MBR code identifies the active partition from the partition table (the one marked “bootable”) and hands over control to that partition’s boot sector.

3. **Partition Boot Sector:**
The MBR contains a table of partitions along with the start, size and flags of each partition on that disk. The first sector of the partition generally is called the Partition boot sector, and it contains code to read files of that partition. Assuming that we are on an NTFS formatted partition - this is also sometimes called the NTFS boot sector. The boot sector contains a small piece of code that is responsible for finding the OS loader file (in XP's case - NTLDR). This code is highly size-constrained as it has to fit within the 512 bytes and is designed solely to load NTLDR.

4. **NTLDR, Boot Configuration parsing and Hardware Detection:**
The boot sector code loads NTLDR (also called the NT Loader) typically is at the root directory of the active, or “system,” partition (typically C:\NTLDR) into memory. Optionally, if the system requires it (for example, if the boot drive is attached via SCSI), NTLDR will also load ntbootdd.sys (a disk driver) to ensure that the nt loader can use it to access the boot media.<br><br>After being loaded, NTLDR reads the boot.ini file (found in the root of the system partition). This plain-text file lists the available OS entries (using ARC paths) and any boot-time options such as safe mode, PAE, 3BG etc. If you have a dual-boot setup, this file determines what options are presented to the user.
<br><br>
Immediately after, NTLDR runs NTDETECT.COM. This small executable gathers basic information about the hardware (processor type, RAM, etc.) on the system. The collected hardware information is later incorporated into the Windows registry (under keys such as HKEY_LOCAL_MACHINE\HARDWARE).

5. **Kernel Loading: NTOSKRNL.EXE and HAL.DLL**
With hardware details in hand, NTLDR loads the kernel, NTOSKRNL.EXE (the Windows NT Operating System kernel). This file is typically located in \WINDOWS\system32 (or \WINNT\system32 depending on your installation). At the same time, the Hardware Abstraction Layer, HAL.DLL, is loaded. The HAL sits between NTOSKRNL.EXE and the physical hardware to provide a level of abstraction that lets Windows work with different hardware configurations without needing custom code for every possible device.

6. **Transitioning to Kernel Mode:** 
Kernel Initialization: The kernel switches the CPU from the “flat” memory mode ([Real Addressing mode](https://en.wikipedia.org/wiki/Real_mode)) set up by NTLDR to its fully operational mode([Protected Virtual Address Mode](https://en.wikipedia.org/wiki/Protected_mode)). At this point, it starts initializing critical components, including memory management, process scheduling, and interrupt handling. Next, configuration information stored in the registry hives is loaded from hive files stored in \WINDOWS\system32\config. The key registry hives include:<br>
SYSTEM: Contains settings related to system hardware and driver configuration.<br>
SOFTWARE: Holds information about installed software and system configuration.<br>
SECURITY, SAM, and DEFAULT: Used for security settings, account management, and default configurations.
<br>
Following the registry hives, other drivers and system services are loaded and executed based on their Load/Start order mentioned in the SYSTEM\\Services registry key. Finally control is transferred to the Session manager. Session Manager (SMSS.EXE) performs further initialization, such as creating system sessions and starting up essential subsystems.

7. **User Logon and Desktop Environment:**
SMSS.EXE launches WINLOGON.EXE, which is responsible for managing interactive logons. WINLOGON displays the welcome/login screen and enforces security by ensuring that the secure attention sequence (Ctrl+Alt+Del) is used.<br>
The user enters credentials, which WINLOGON validates with the Local Security Authority(LSA) or Network Logon Service(netlogon).

8. Starting the Windows Shell: Once the logon process is complete, WINLOGON starts EXPLORER.EXE, the graphical shell that provides the desktop, Start menu, taskbar, and other user interface components. At this point, the system is fully booted and ready for interactive use.


Here's a quick flowchart to keep in mind:


            +-----------------------------+
            |      Power On / POST        |
            |  (BIOS initializes hardware,|
            |   performs self-tests)      |
            +-------------+---------------+
                          │
                          ▼
            +-------------+---------------+
            | BIOS Boot Device Selection  |
            |   (Reads CMOS boot order)   |
            +-------------+---------------+
                          │
                          ▼
            +-------------+----------------+
            |        MBR (Sector 0)        |
            | (Selects the active partition|
            |  and loads boot code)        |
            +-------------+----------------+
                          │
                          ▼
            +-------------+----------------+
            | Boot Sector of Active Part.  |
            | (Contains minimal code to    |
            |  locate & load NTLDR)        |
            +-------------+----------------+
                          │
                          ▼
            +-------------+---------------+
            |          NTLDR              |
            | (Main boot loader for XP)   |
            +------+------+---------------+
                   │             │
                   │             │
                   ▼             ▼
         +---------+-----+  +-----------+---------+
         |   boot.ini    |  |   NTDETECT.COM      |
         | (OS options & |  | (Detects basic      |
         |  boot config) |  |  hardware info)     |
         +---------+-----+  +-----------+---------+
                   │
                   ▼
         +---------+---------------------+
         |   NTOSKRNL.EXE & HAL.DLL      |
         | (Loads the Windows kernel &   |
         |   abstracts hardware)         |
         +---------+---------------------+
                   │
                   ▼
         +---------+---------------------+
         |        SMSS.EXE               |
         | (Session Manager; creates     |
         |  system sessions & initializes|
         |  environment)                 |
         +---------+---------------------+
                   │
                   ▼
         +---------+---------------------+
         |       WINLOGON.EXE            |
         | (Manages logon, secures login |
         |  process & enforces SAS)      |
         +---------+---------------------+
                   │
                   ▼
         +---------+----------------------+
         |       EXPLORER.EXE             |
         | (Launches the Windows shell,   |
         |  providing the desktop, taskbar|
         |  and user interface)           |
         +--------------------------------+

In the next post, I'll discuss some troubleshooting aspects and how things changed with Windows Vista - which sets the standard for modern era operating systems like win7, win8/8.1, win10 and win11. 

Until next time 👋

