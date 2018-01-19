---
layout: post
category: Reverse-Engg
title: Assembly Language Basics - Part 6
date: 2016-05-22
---

Go ahead and launch notepad under the debugger just like you did in the last post. Assuming you have already set the _NT_SYMBOL_PATH environment variable - go ahead and run `.reload -f`. This will load the symbols for all modules (We'll discuss symbols later). Next, run `lmvm notepad`, it should give you an output similar to the one below. The address given under *"start"* is the address where this module was loaded - in my case `00007ff63f030000`. 

    0:000> lmvm notepad
    Browse full module list
    start             end                 module name
    00007ff6`3f030000 00007ff6`3f071000   notepad    (deferred)             
        Image path: notepad.exe
        Image name: notepad.exe
        Browse all global symbols  functions  data
        Timestamp:        A0C4CEAB
        CheckSum:         0004AC7B
        ImageSize:        00041000
        File version:     10.0.16299.15
        Product version:  10.0.16299.15
        File flags:       0 (Mask 3F)
        File OS:          40004 NT Win32
        File type:        1.0 App
        File date:        00000000.00000000
        Translations:     0409.04b0
        CompanyName:      Microsoft Corporation
        ProductName:      Microsoft® Windows® Operating System
        InternalName:     Notepad
        OriginalFilename: NOTEPAD.EXE
        ProductVersion:   10.0.16299.15
        FileVersion:      10.0.16299.15 (WinBuild.160101.0800)
        FileDescription:  Notepad
        LegalCopyright:   © Microsoft Corporation. All rights reserved.



You can use the `!dh <start address>` command to dump the PE header. !dh is a command that is part of a built extension in windbg, it dumps the header of a file given its address. A lot of this will not make sense till we manually run through the structures, you can ignore this till then.

{% highlight asm %}
0:000> !dh 00007ff6`3f030000

File Type: EXECUTABLE IMAGE
FILE HEADER VALUES
    8664 machine (X64)
       6 number of sections
A0C4CEAB time date stamp Mon Jun 21 21:48:43 2055

       0 file pointer to symbol table
       0 number of symbols
      F0 size of optional header
      22 characteristics
            Executable
            App can handle >2gb addresses

OPTIONAL HEADER VALUES
     20B magic #
   14.10 linker version
   19000 size of code
   25200 size of initialized data
       0 size of uninitialized data
   193E0 address of entry point
    1000 base of code
         ----- new -----
00007ff63f030000 image base
    1000 section alignment
     200 file alignment
       2 subsystem (Windows GUI)
   10.00 operating system version
   10.00 image version
   10.00 subsystem version
   41000 size of image
     400 size of headers
   4AC7B checksum
0000000000080000 size of stack reserve
0000000000011000 size of stack commit
0000000000100000 size of heap reserve
0000000000001000 size of heap commit
    C160  DLL characteristics
            High entropy VA supported
            Dynamic base
            NX compatible
            Guard
            Terminal server aware
       0 [       0] address [size] of Export Directory
   1F4D8 [     244] address [size] of Import Directory
   26000 [   19CE0] address [size] of Resource Directory
   25000 [     8DC] address [size] of Exception Directory
       0 [       0] address [size] of Security Directory
   40000 [     21C] address [size] of Base Relocation Directory
   1E370 [      54] address [size] of Debug Directory
       0 [       0] address [size] of Description Directory
       0 [       0] address [size] of Special Directory
       0 [       0] address [size] of Thread Storage Directory
   1A550 [     100] address [size] of Load Configuration Directory
       0 [       0] address [size] of Bound Import Directory
   1A650 [     978] address [size] of Import Address Table Directory
       0 [       0] address [size] of Delay Import Directory
       0 [       0] address [size] of COR20 Header Directory
       0 [       0] address [size] of Reserved Directory


SECTION HEADER #1
   .text name
   18FCE virtual size
    1000 virtual address
   19000 size of raw data
     400 file pointer to raw data
       0 file pointer to relocation table
       0 file pointer to line numbers
       0 number of relocations
       0 number of line numbers
60000020 flags
         Code
         (no align specified)
         Execute Read

SECTION HEADER #2
  .rdata name
    7602 virtual size
   1A000 virtual address
    7800 size of raw data
   19400 file pointer to raw data
       0 file pointer to relocation table
       0 file pointer to line numbers
       0 number of relocations
       0 number of line numbers
40000040 flags
         Initialized Data
         (no align specified)
         Read Only


Debug Directories(3)
	Type       Size     Address  Pointer
	cv           24       1e7cc    1dbcc	Format: RSDS, guid, 1, notepad.pdb
	(    13)     2c0       1e7f0    1dbf0
	(    16)       0           0        0

SECTION HEADER #3
   .data name
    2D14 virtual size
   22000 virtual address
     C00 size of raw data
   20C00 file pointer to raw data
       0 file pointer to relocation table
       0 file pointer to line numbers
       0 number of relocations
       0 number of line numbers
C0000040 flags
         Initialized Data
         (no align specified)
         Read Write

SECTION HEADER #4
  .pdata name
     8DC virtual size
   25000 virtual address
     A00 size of raw data
   21800 file pointer to raw data
       0 file pointer to relocation table
       0 file pointer to line numbers
       0 number of relocations
       0 number of line numbers
40000040 flags
         Initialized Data
         (no align specified)
         Read Only

SECTION HEADER #5
   .rsrc name
   19CE0 virtual size
   26000 virtual address
   19E00 size of raw data
   22200 file pointer to raw data
       0 file pointer to relocation table
       0 file pointer to line numbers
       0 number of relocations
       0 number of line numbers
40000040 flags
         Initialized Data
         (no align specified)
         Read Only

SECTION HEADER #6
  .reloc name
     21C virtual size
   40000 virtual address
     400 size of raw data
   3C000 file pointer to raw data
       0 file pointer to relocation table
       0 file pointer to line numbers
       0 number of relocations
       0 number of line numbers
42000040 flags
         Initialized Data
         Discardable
         (no align specified)
         Read Only

{% endhighlight %}

The output may seem long and strange, but its easy once you walk through the internal structures.

# PE structures:

The PE file format is made up of IMAGE_DOS_HEADER - aka DOS header, IMAGE_NT_HEADERS - aka NT header (which consists of IMAGE_FILE_HEADER, IMAGE_OPTIONAL_HEADER), IMAGE_SECTION_HEADERS aka Section headers, and Sections. You can get the details on the structures of PE file format on Winnt.h which will be available once you install the SDK (usually at this location `"C:\Program Files (x86)\Windows Kits\8.1\Include\um\winnt.h"`. 

PE has been around since the DOS days, which is why the DOS header appears at the start of the file. Here's a snippet from the winnt.h file showing the IMAGE_DOS_HEADER structure and its member. The comments beside each member tells you the fields.

{% highlight cpp %}

typedef struct _IMAGE_DOS_HEADER {      // DOS .EXE header
    WORD   e_magic;                     // Magic number
    WORD   e_cblp;                      // Bytes on last page of file
    WORD   e_cp;                        // Pages in file
    WORD   e_crlc;                      // Relocations
    WORD   e_cparhdr;                   // Size of header in paragraphs
    WORD   e_minalloc;                  // Minimum extra paragraphs needed
    WORD   e_maxalloc;                  // Maximum extra paragraphs needed
    WORD   e_ss;                        // Initial (relative) SS value
    WORD   e_sp;                        // Initial SP value
    WORD   e_csum;                      // Checksum
    WORD   e_ip;                        // Initial IP value
    WORD   e_cs;                        // Initial (relative) CS value
    WORD   e_lfarlc;                    // File address of relocation table
    WORD   e_ovno;                      // Overlay number
    WORD   e_res[4];                    // Reserved words
    WORD   e_oemid;                     // OEM identifier (for e_oeminfo)
    WORD   e_oeminfo;                   // OEM information; e_oemid specific
    WORD   e_res2[10];                  // Reserved words
    LONG   e_lfanew;                    // File address of new exe header
  } IMAGE_DOS_HEADER, *PIMAGE_DOS_HEADER;

{% endhighlight %}

On a debugger, you can use the dump type or dt command to dump a data type. `dt <symbol>` should just dump the data structure and all its fields of the type name or symbol. If you provide an address along with the symbol name, then it also dumps the contents of this address and arranges the values against each member of the type - An example for `dt ntdll!_IMAGE_DOS_HEADER <start address>` is listed below. You can also dump the raw bytes and character repersentation using `dc <address>` as shown below.

{% highlight cpp %}

    0:000> dt ntdll!_IMAGE_DOS_HEADER 00007ff6`3f030000
    +0x000 e_magic          : 0x5a4d  <--magic number MZ
    +0x002 e_cblp           : 0x90 
    +0x004 e_cp             : 3  <-- number of pages in the file.
    +0x006 e_crlc           : 0
    +0x008 e_cparhdr        : 4
    +0x00a e_minalloc       : 0
    +0x00c e_maxalloc       : 0xffff
    +0x00e e_ss             : 0
    +0x010 e_sp             : 0xb8
    +0x012 e_csum           : 0
    +0x014 e_ip             : 0
    +0x016 e_cs             : 0
    +0x018 e_lfarlc         : 0x40
    +0x01a e_ovno           : 0
    +0x01c e_res            : [4] 0
    +0x024 e_oemid          : 0
    +0x026 e_oeminfo        : 0
    +0x028 e_res2           : [10] 0
    +0x03c e_lfanew         : 0n248  <-- relative value to be added to the start address to get to the next structure

//-- This command is similar to the sizeof() operator in C, it shows the size of memory of the datatype.
0:000> ??sizeof( ntdll!_IMAGE_DOS_HEADER )
unsigned int64 0x40

0:000> ?0007ff6`3f030000 + 0x40
Evaluate expression: 140695595843648 = 00007ff6`3f030040

//-- Since the PE header is 0x40 bytes in size, it extends from 0007ff6`3f030000 to 00007ff6`3f030040

0:000> dc 00007ff6`3f030000 
00007ff6`3f030000  00905a4d 00000003 00000004 0000ffff  MZ..............  <-- first two bytes are MZ
00007ff6`3f030010  000000b8 00000000 00000040 00000000  ........@.......
00007ff6`3f030020  00000000 00000000 00000000 00000000  ................
00007ff6`3f030030  00000000 00000000 00000000 000000f8  ................  <-- PE header ends
00007ff6`3f030040  0eba1f0e cd09b400 4c01b821 685421cd  ........!..L.!Th
00007ff6`3f030050  70207369 72676f72 63206d61 6f6e6e61  is program canno
00007ff6`3f030060  65622074 6e757220 206e6920 20534f44  t be run in DOS 
00007ff6`3f030070  65646f6d 0a0d0d2e 00000024 00000000  mode....$.......

{% endhighlight %}

Now, lets take a moment to see what we have just done. 
* We first used `lmvm` command to get the module's load address. This is where the module starts in memory.
* Once we know the start address we used an internal command `!dh` to parse the header and dumps out useful information.
* Since our aim is to learn about the actual data structure, we referred to winnt.h to understand the structure members. 
* We then dumped the start address again by typecasting it as ntdll!_IMAGE_DOS_HEADER in the ` dt ntdll!_IMAGE_DOS_HEADER <start address>` command. This gave us the members of the structure along with their values.
* Once we read about the members, we also calculated the size of the strucute using `??sizeof(<type>)` and dumped the raw hexadecimal view of bytes from the start address using `dc`. 
* The size of the Header is 0x40 bytes, which means that the header extends from 0007ff63f030000 to 00007ff63f030040. 
* The first member of the structure is `e_magic`, this number will be hexadecimal representation of characters "MZ".  We could see that in the raw view (see `dc` command output above). 
* Immediately after the header is a tiny program that prints out “This program cannot be run in MS-DOS mode”, if this executable is run in DOS.
* The most important member of the DOS header is `e_lfanew`, this member in the MS-DOS stub header is a relative offset (or RVA) to the actual NT header. To get a pointer to the NT header in memory, we just add that field’s value to the image base as follows:

    The Pointer of NT Header = the Start Address of DOSHeader + DosHeader->e_lfanew;
    
    0:000> ?00007ff6`3f030000+0n248
    Evaluate expression: 140695595843832 = 00007ff6`3f0300f8  <-- offset of the NT header

Note that when windbg prints number, it adds a prefix. If numbers are prefixed with 0x - those are hexadecimal representation, of a number is prefixed with 0n, that is the decimal representation and if a number is prefixed with 0y, that is the binary representation.


We now know how to calculate the offset of the NT header. We will dig into the NT header in the next post, you can also read about the format from Microsoft's documentation here: http://msdn.microsoft.com/en-us/library/ms809762.aspx.