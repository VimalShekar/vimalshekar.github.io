---
layout: post
category: Reverse-Engg
title: Assembly Language Basics - Part 4
date: 2016-04-10
---

The PE format!
For a binary to run in an operating environment, it is essential that the binary adhere to the standard that operating system defines. 
The Portable Executable (PE) file format is used by Windows executables, object code, and DLLs. This format has been derived from the COFF
(Common Object File Format) image format - Refer to “Microsoft PE and COFF Specification” on MSDN for complete specification:
https://msdn.microsoft.com/en-us/windows/hardware/gg463119.aspx

In practice - the PE file format is a data structure that contains the information necessary for the Windows OS loader to load and manage 
the executable. PE files begin with a header that includes information about the code, the type of application, required library functions, 
and space requirements, and in my opinion - this should be your starting point for analyzing the malware.


Before we dig into the format, lets take a look at some important terms:
The actual creation of an executable image is a multi-step process. Several development kits exists to create the image, Microsoft has also provided 
tools in the SDK/WDK and an integrated development environment called Visual Studio to guide you through this process. The overall work flow looks like this:
![stages]({{ "/images/imagecreation.jpg" | absolute_url }})

> Assemblers convert assembly language to machine code in the form of object files. These are machine dependent and generally not portable code. Assemblers operate on one source module at a time.

> Compilers convert code written in high level languages to machine code in the form of object files. Compilers operate on one source module at a time. C, C++, C# are some examples of high level languages for which Microsoft has released compilers. You can download Visual Studio Community edition and it choose the options you need, there are lots of walkthroughs for this online.

> Librarian in contrast to assembler or compiler just assists in bundling various object files into a single library file.

> Linker generates the final binary by combining all the object files, resolving all the dependencies. Linker is the tool capable of generating symbol files to assist in debugging.

> Debugging is a methodical process of identifying a problem in a piece of software or hardware.

> Debugger is a program that allows observation of another program either in an invasive or a noninvasive fashion. Two typical features that debuggers offer would be:
    * Ability to set up breakpoints
    * Ability to step or trace through code

Debuggers are classified into:
* low-level debugger – the one that assist in debugging but with the help of
disassembly
* symbolic debugger – assists debugging with help of symbols like variable, functions
names
* source-level debugger – assists debugging with relevant source code

Now let's get a debugger installed. I dunno about you, but my career didn’t start out 
with programming. I did my engineering in Electronics and had some background with Microprocessors and micro controller programming. In my current 
role as an EE, we deal with analysing process, kernel and complete dumps all the time. The first debugger introduced to me was windbg, and so even 
before I started writing production quality code, I had windbg on my machine. 

If you’re a Windows guy like me, then you can get the Software development kit (which includes a compiler) and the Debugging tools for 
Windows together as part of the Windows SDK. At the time of writing, the SDK for 8.1 was the latest – you can download it from the below 
link: https://www.microsoft.com/click/services/Redirect2.ashx?CR_EAC=300135395

It’s better to install the SDK and debugging tools together, so you have a complete package for your learning and don’t have to come back 
to install it again. The Performance toolkit is a great tool for troubleshooting leaks, and high CPU issues – I highly recommend installing 
it. Application verifier is optional.

![sdksetup]({{ "/images/SettingUpSDK.png" | absolute_url }})

Once you have the debugging tools downloaded, also set up your Symbol Path. It’s a good idea to do this using environment variables so that 
your other tools such as procmon, wpa and message analyzer can also pick up symbols using the same path.

# Configuring the _NT_SYMBOL_PATH environment variable:

> * Start > run > sysdm.cpl
> * Go to the Advanced Tab > Environment Variables.
> * Under user variables, click New.
> * Give the variable name as : _NT_SYMBOL_PATH
> * Give the variable value as :  `srv*C:\PublicSymbols*http://msdl.microsoft.com/downloads/symbols` and then click OK to save.
> * Make sure that C:\PublicSymbols folder exists on the machine, otherwise create a folder at that path.

Now that you have a debugger installed you are ready to load dumps and executable and attach to processes etc. To read the PE format, you can just
load an executable file (say notepad.exe) into the debugger as if it was a memory dump. I'll walk through that in the next blog as we dig into the PE headers.


