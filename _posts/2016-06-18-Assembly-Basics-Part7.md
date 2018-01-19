---
layout: post
category: Reverse-Engg
title: Assembly Language Basics - Part 7
date: 2016-06-18
---

# The NT header:

In the previous post we dumped the DOS header of a binary, got its `e_lfanew` member and calculated the address NT header using the expression - `The Pointer of NT Header = the Start Address of DOSHeader + DosHeader->e_lfanew;`
Note that `e_lfanew` is a Relative Virtual Address or RVA. Anytime you have an RVA, you add that address to the Start address of the image (ie address of DOS header), to get to the actual address. In our case, DOS header starts at `00007ff63f030000`, and e_lfnew was +0n248, so that means the NT header is at `00007ff63f0300f8`. 

    0:000> ?00007ff6`3f030000+0n248
    Evaluate expression: 140695595843832 = 00007ff6`3f0300f8  <-- offset of the NT header

Now we go back to winnt.h to get the definition for the NT header. Below is a snippet from the file:  

{% highlight cpp %}
#ifdef _WIN64
typedef IMAGE_OPTIONAL_HEADER64             IMAGE_OPTIONAL_HEADER;
typedef PIMAGE_OPTIONAL_HEADER64            PIMAGE_OPTIONAL_HEADER;
#define IMAGE_NT_OPTIONAL_HDR_MAGIC         IMAGE_NT_OPTIONAL_HDR64_MAGIC
#else
typedef IMAGE_OPTIONAL_HEADER32             IMAGE_OPTIONAL_HEADER;
typedef PIMAGE_OPTIONAL_HEADER32            PIMAGE_OPTIONAL_HEADER;
#define IMAGE_NT_OPTIONAL_HDR_MAGIC         IMAGE_NT_OPTIONAL_HDR32_MAGIC
#endif

typedef struct _IMAGE_NT_HEADERS64 { //<-- there is also a 32 bit version of this header, but I have chosen to dump 64 bit
    DWORD Signature;  //<--- This file signature is usually "PE"
    IMAGE_FILE_HEADER FileHeader; //<-- embedded file header
    IMAGE_OPTIONAL_HEADER64 OptionalHeader;  //<-- embedded optional header. There is also a 32 bit version of this struct
} IMAGE_NT_HEADERS64, *PIMAGE_NT_HEADERS64;

//-- This is the file header structure
typedef struct _IMAGE_FILE_HEADER {
    WORD    Machine;            //<--
    WORD    NumberOfSections;
    DWORD   TimeDateStamp;
    DWORD   PointerToSymbolTable;
    DWORD   NumberOfSymbols;
    WORD    SizeOfOptionalHeader;
    WORD    Characteristics;
} IMAGE_FILE_HEADER, *PIMAGE_FILE_HEADER;

//-- This is the optional header structure
typedef struct _IMAGE_OPTIONAL_HEADER64 {
    WORD        Magic;
    BYTE        MajorLinkerVersion;
    BYTE        MinorLinkerVersion;
    DWORD       SizeOfCode;
    DWORD       SizeOfInitializedData;
    DWORD       SizeOfUninitializedData;
    DWORD       AddressOfEntryPoint;
    DWORD       BaseOfCode;
    ULONGLONG   ImageBase;
    DWORD       SectionAlignment;
    DWORD       FileAlignment;
    WORD        MajorOperatingSystemVersion;
    WORD        MinorOperatingSystemVersion;
    WORD        MajorImageVersion;
    WORD        MinorImageVersion;
    WORD        MajorSubsystemVersion;
    WORD        MinorSubsystemVersion;
    DWORD       Win32VersionValue;
    DWORD       SizeOfImage;
    DWORD       SizeOfHeaders;
    DWORD       CheckSum;
    WORD        Subsystem;
    WORD        DllCharacteristics;
    ULONGLONG   SizeOfStackReserve;
    ULONGLONG   SizeOfStackCommit;
    ULONGLONG   SizeOfHeapReserve;
    ULONGLONG   SizeOfHeapCommit;
    DWORD       LoaderFlags;
    DWORD       NumberOfRvaAndSizes;
    IMAGE_DATA_DIRECTORY DataDirectory[IMAGE_NUMBEROF_DIRECTORY_ENTRIES];
} IMAGE_OPTIONAL_HEADER64, *PIMAGE_OPTIONAL_HEADER64;

{% endhighlight %}

As you can see, the NT header is composed of a File header and an Optional header. Depending on the bitness of the executable - there is a 32 bit and 64 bit version of the structure. I have chosen to dump the 64 bit one here. Once you know the structure name and address, you can dump it using `dt` or `dx` command at the debugger command window. To get help on the command syntax, type `.hh <command or search string>` at the debugger prompt.

{% highlight cpp%}
0:000> dt ntdll!_IMAGE_NT_HEADERS64 00007ff6`3f0300f8
   +0x000 Signature        : 0x4550
   +0x004 FileHeader       : _IMAGE_FILE_HEADER
   +0x018 OptionalHeader   : _IMAGE_OPTIONAL_HEADER64

The new version of windbg adds hyper links for resolvable structures like the File header and optional header from the above command. If you click the link - it expands the structure.

0:000> dx -r1 (*((ntdll!_IMAGE_FILE_HEADER *)0x7ff63f0300fc))
(*((ntdll!_IMAGE_FILE_HEADER *)0x7ff63f0300fc))                 [Type: _IMAGE_FILE_HEADER]
    [+0x000] Machine          : 0x8664 [Type: unsigned short]  <-- machine type is x86-64
    [+0x002] NumberOfSections : 0x6 [Type: unsigned short]  <--- There are 6 sections in this binary
    [+0x00c] NumberOfSymbols  : 0x0 [Type: unsigned long]
    [+0x010] SizeOfOptionalHeader : 0xf0 [Type: unsigned short]  <-- size of the optional header is 0x70
    [+0x012] Characteristics  : 0x22 [Type: unsigned short]

0:000> dx -r1 (*((ntdll!_IMAGE_OPTIONAL_HEADER64 *)0x7ff63f030110))
(*((ntdll!_IMAGE_OPTIONAL_HEADER64 *)0x7ff63f030110))                 [Type: _IMAGE_OPTIONAL_HEADER64]
    [+0x000] Magic            : 0x20b [Type: unsigned short]
    [+0x002] MajorLinkerVersion : 0xe [Type: unsigned char]   <-- version number details of linker
    [+0x003] MinorLinkerVersion : 0xa [Type: unsigned char]
    [+0x004] SizeOfCode       : 0x19000 [Type: unsigned long]   <-- size of .text section
    [+0x008] SizeOfInitializedData : 0x25200 [Type: unsigned long] 
    [+0x00c] SizeOfUninitializedData : 0x0 [Type: unsigned long]
    [+0x010] AddressOfEntryPoint : 0x193e0 [Type: unsigned long]  <-- address of the entry point or first function
    [+0x014] BaseOfCode       : 0x1000 [Type: unsigned long]
    [+0x018] ImageBase        : 0x7ff63f030000 [Type: unsigned __int64]  <--  base address, same as load address 
    [+0x020] SectionAlignment : 0x1000 [Type: unsigned long] <--  sections are aligned to 0x1000 (ie 4k page boundaries)
    [+0x024] FileAlignment    : 0x200 [Type: unsigned long]
    [+0x028] MajorOperatingSystemVersion : 0xa [Type: unsigned short] <-- version number details of OS
    [+0x02a] MinorOperatingSystemVersion : 0x0 [Type: unsigned short]
    [+0x02c] MajorImageVersion : 0xa [Type: unsigned short] <-- version number details of this image
    [+0x02e] MinorImageVersion : 0x0 [Type: unsigned short]
    [+0x030] MajorSubsystemVersion : 0xa [Type: unsigned short]
    [+0x032] MinorSubsystemVersion : 0x0 [Type: unsigned short]
    [+0x034] Win32VersionValue : 0x0 [Type: unsigned long]
    [+0x038] SizeOfImage      : 0x41000 [Type: unsigned long]  <-- image size
    [+0x03c] SizeOfHeaders    : 0x400 [Type: unsigned long]
    [+0x040] CheckSum         : 0x4ac7b [Type: unsigned long]  <-- image checksum
    [+0x044] Subsystem        : 0x2 [Type: unsigned short]
    [+0x046] DllCharacteristics : 0xc160 [Type: unsigned short] <-- You can find more on the characteristics here: http://msdn.microsoft.com/en-us/library/ms680313(VS.85).aspx .
    [+0x048] SizeOfStackReserve : 0x80000 [Type: unsigned __int64] <-- reserved stack space (not all of it is allocated yet)
    [+0x050] SizeOfStackCommit : 0x11000 [Type: unsigned __int64] <-- committed stack space (what has been allocate to the process so far)
    [+0x058] SizeOfHeapReserve : 0x100000 [Type: unsigned __int64] <-- reserved heap space (not all of it is allocated yet)
    [+0x060] SizeOfHeapCommit : 0x1000 [Type: unsigned __int64]<-- committed heap space (what has been allocate to the process so far)
    [+0x068] LoaderFlags      : 0x0 [Type: unsigned long]
    [+0x06c] NumberOfRvaAndSizes : 0x10 [Type: unsigned long]
    [+0x070] DataDirectory    [Type: _IMAGE_DATA_DIRECTORY [16]]  <-- array of directories

{% endhighlight %}


I have added comments to the important members of the structure. If you compare this with the output from the `!dh <base address>` command, you will see that most of the details are the same. `!dh` actually parsed these headers and listed the details for you.

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

well, that's it for now - hang around for more in the next post!