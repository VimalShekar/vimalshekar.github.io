---
layout: post
category: Reverse-Engg
title: Exploring the Windows Executive - Part 1 - Object Manager
date: 2016-10-16
---

The Windows object manager is responsible for creating, protecting and managing objects. Internally – it has 3 types:

> * kernel objects : primitive set of objects implemented in the kernel.
> * executive objects : implemented in components of the kernel.
> * GDI/User objects : belong within Windows subsystem and are not used by kernel.

Other than this, there is something called “Type Objects”. To save memory, object manager saves static content for objects of the same type under a Typeobject. Any object created of a specific type will inherit the type object’s properties from the TypeObject.

Whenever user mode code wants to access an object, it open a handle to the object. Whenever kernel mode code opens the object, it gains a reference to the object. Thus the object manager also keeps track of the handle and reference counts of the object. Every time a process opens a handle to the object, the handle count in the object’s header is incremented by 1.There is also a reference count that gets incremented when handle count gets incremented, and whenever the kernel assigns a pointer to the object.

The object handle is an index into the process specific Handle Table. The index starts at 0, and increments in multiples of 4. Handle table has 3 levels -> Lowest level is created when process gets created. Other levels as needed. Lowest level -> 4k Page – with 511 handle entries and a pointer to mid-level table. Mid level has 4096 pointers to sub handle tables – each of which is can have 512 entries.

You can get to the kernel handle table by looking at the address pointed to by ObpKernelHandleTable. Addresses have the MSB bit as 1 – that is how OM knows this is a kernel handle.

{% highlight asm %}
0: kd> dt ObpKernelHandletable
nt!ObpKernelHandleTable
0xfffff8a0`000018e0 
   +0x000 TableCode        : 0xfffff8a0`00c96001
   +0x008 QuotaProcess     : (null) 
   +0x010 UniqueProcessId  : 0x00000000`00000004 Void
   +0x018 HandleLock       : _EX_PUSH_LOCK
   +0x020 HandleTableList  : _LIST_ENTRY [ 0xfffff8a0`001c8440 - 0xfffff800`028197f0 ]
   +0x030 HandleContentionEvent : _EX_PUSH_LOCK
   +0x038 DebugInfo        : (null) 
   +0x040 ExtraInfoPages   : 0n0
   +0x044 Flags            : 0
   +0x044 StrictFIFO       : 0y0
   +0x048 FirstFreeHandle  : 0x6fc
   +0x050 LastFreeHandleEntry : 0xfffff8a0`00c97ff0 _HANDLE_TABLE_ENTRY
   +0x058 HandleCount      : 0x1c8
   +0x05c NextHandleNeedingPool : 0x800
   +0x060 HandleCountHighWatermark : 0x1cc
{% endhighlight %}

The !object extension displays information about a system object.
```
    Syntax:
    !object Address
    !object 0 Name
    !object Path

```
Where:
* Address - Specifies the hexadecimal address of the system object for which to display information.
* Name - If the first argument is zero, the second argument is interpreted as the name of a class of system objects for which to display all instances.
* Path - If the first argument begins with a backslash (\), !object interprets it as an object path name. When this option is used, the display will be arranged according to the directory structure used by the Object Manager.

{% highlight asm %}
0: kd> !object \
Object: 86a05750  Type: (85261eb0) Directory
    ObjectHeader: 86a05738 (new version)
    HandleCount: 0  PointerCount: 40
    Directory Object: 00000000  Name: \
 
    Hash Address  Type          Name
    ---- -------  ----          ----
     00  86a09468 Directory     ArcName
         8a76a538 Device        Ntfs
     01  8d5f6dc0 ALPC Port     SeLsaCommandPort
         8d464500 Event         UniqueInteractiveSessionIdEvent
     03  86a116d0 Key           \REGISTRY
     04  852c2840 ALPC Port     PowerPort
     05  8d7e30c8 ALPC Port     ThemeApiPort
         8d60d5d0 Event         DSYSDBG.Debug.Trace.Memory.270
     09  86a57030 Directory     (*** Name not accessible ***)
     10  86a08db0 SymbolicLink  DosDevices
     12  9a537c58 ALPC Port     UxSmsApiPort
     13  8d4e4780 ALPC Port     SeRmCommandPort
     14  9a584a10 Event         LanmanServerAnnounceEvent
         93352248 SymbolicLink  Dfs
         86a4b030 Directory     UMDFCommunicationPorts
     16  86a5a908 Directory     Driver
     18  8a76aa68 Device        clfs
     19  86a09390 Directory     Device
     20  9335d710 Directory     Windows
         8d47a448 Event         CsrSbSyncEvent
     21  9335e530 Directory     Sessions
         8d667790 Event         SAM_SERVICE_STARTED
     22  9335d528 Directory     RPC Control
         8d5042b8 ALPC Port     SmApiPort
     23  9d1961f0 Directory     BaseNamedObjects
         86a055d0 Directory     KernelObjects
         852c2430 ALPC Port     PowerMonitorPort
     24  86a05128 Directory     GLOBAL??
         86a4bdd8 Directory     FileSystem
     25  9a8418e8 Section       LsaPerformance
     26  8d6a69a0 ALPC Port     SmSsWinStationApiPort
         86a05468 Directory     ObjectTypes
     27  86a097e8 Directory     Security
     31  86baa420 SymbolicLink  SystemRoot
     32  86a08c10 Directory     Callback
     33  8d758958 Event         UniqueSessionIdEvent
         8d73dda8 Event         EFSInitEvent
     35  933a9e38 Directory     KnownDlls
{% endhighlight %}

The \Driver directory contains all driver objects
The \Device directory contains all the device objects
The \ObjectTypes directory contains Type objects for the various object types available.

{% highlight asm %}
0: kd> !object \ObjectTypes
Object: 86a05468  Type: (85261eb0) Directory
    ObjectHeader: 86a05450 (new version)
    HandleCount: 0  PointerCount: 44
    Directory Object: 86a05750  Name: ObjectTypes
 
    Hash Address  Type          Name
    ---- -------  ----          ----
     00  852cc508 Type          TpWorkerFactory
         85261eb0 Type          Directory
     01  852c8440 Type          Mutant
         85261948 Type          Thread
     03  8a763a60 Type          FilterCommunicationPort
     04  852cd580 Type          TmTx
     05  852cc378 Type          Controller
     06  852e81e8 Type          EtwRegistration
     07  852cc828 Type          Profile
         852cb418 Type          Event
         85261f78 Type          Type
     09  852d4130 Type          Section
         852cb350 Type          EventPair
         85261de8 Type          SymbolicLink
     10  852cc5d0 Type          Desktop
         85261880 Type          UserApcReserve
     11  852e98b0 Type          EtwConsumer
         852cc8f0 Type          Timer
     12  852cdde8 Type          File
         852cc698 Type          WindowStation
     14  8a7adbd0 Type          PcwObject
     15  852cd3f0 Type          TmEn
     16  852cdf78 Type          Driver
     18  852e8540 Type          WmiGuid
         852cc760 Type          KeyedEvent
     19  852cd040 Type          Device
         85261c18 Type          Token
     20  852c2a30 Type          ALPC Port
         8525d958 Type          DebugObject
     21  852cdeb0 Type          IoCompletion
     22  85261a10 Type          Process
     23  852cd4b8 Type          TmRm
     24  852cc440 Type          Adapter
     26  852c2338 Type          PowerRequest
         852c0bd8 Type          Key
     28  85261ad8 Type          Job
     30  852d4068 Type          Session
         852cd648 Type          TmTm
     31  852617b8 Type          IoCompletionReserve
     32  852c8378 Type          Callback
     33  8a763b28 Type          FilterConnectionPort
     34  852cc9b8 Type          Semaphore
{% endhighlight %}

The \BaseNamedObjects is the global namespace (as opposed to session namespace)

The \Sessions directory contains per session namespaces. When an application creates an object, it typically goes into it respective session unless the application has specifically asked OS to place objects in the Global namespace. Separate session namespaces allow multiple clients to run the same application without interfering with each other. By Default – for a process that’s created in a session, the system uses the session’s namespace unless the process wants to use the global namespace and prepends Global\ prefix to the object name.

{% highlight asm %}
0: kd> !object \Sessions
Object: 9335e530  Type: (85261eb0) Directory
    ObjectHeader: 9335e518 (new version)
    HandleCount: 1  PointerCount: 5
    Directory Object: 86a05750  Name: Sessions
 
    Hash Address  Type          Name
    ---- -------  ----          ----
     03  9d1c1a88 Directory     BNOLINKS
     11  9d1c16d8 Directory     0
     12  9d1cfee8 Directory     1

{% endhighlight %}


The \Global?? – is the namespace where named symbolic links are created for use by all applications. The C: can be found here.

{% highlight asm %}
0: kd> !object \global??
Object: 86a05128  Type: (85261eb0) Directory
    ObjectHeader: 86a05110 (new version)
    HandleCount: 2  PointerCount: 137
    Directory Object: 86a05750  Name: GLOBAL??
 
    Hash Address  Type          Name
    ---- -------  ----          ----
     00  93385720 SymbolicLink  D:
         86bd0ee8 SymbolicLink  NDIS
         93385d88 SymbolicLink  Root#MS_NDISWANIPV6#0000#{cac88484-7515-4c03-82e6-71a87abac361}
         9d17ce38 SymbolicLink  Root#*ISATAP#0000#{ad498944-762f-11d0-8dcb-00c04fc3358c}
         93269120 SymbolicLink  DISPLAY1
     01  86a56938 SymbolicLink  STORAGE#Volume#{f7d72220-50a7-11e5-9939-806e6f6e6963}#0000000000100000#{53f5630d-b6bf-11d0-94f2-00a0c91efb8b}
<snip>
         9335d9b0 SymbolicLink  ACPI#PNP0501#2#{4d36e978-e325-11ce-bfc1-08002be10318}
     32  a93662a0 SymbolicLink  Myfault
         93269d20 SymbolicLink  Volume{f7d72227-50a7-11e5-9939-806e6f6e6963}
         86b34e70 SymbolicLink  FltMgr
         86be4b78 SymbolicLink  FtControl
     33  86bda030 SymbolicLink  C:
         86ae5830 SymbolicLink  MAILSLOT
         933a9668 SymbolicLink  {DB2B4279-B5CF-4626-9DBA-32D0ECE44C87}
<snip>
 
0: kd> !object 86bda030    << !object \Global??\C: will yield the same output
Object: 86bda030  Type: (85261de8) SymbolicLink
    ObjectHeader: 86bda018 (new version)
    HandleCount: 0  PointerCount: 1
    Directory Object: 86a05128  Name: C:
    Target String is '\Device\HarddiskVolume2'
    Drive Letter Index is 3 (C:)
{% endhighlight %}

The C: is a symbolic link for target ‘\Device\hardDiskVolume2’. Lets explore this device object

{% highlight asm %}
0: kd> !object \Device\HarddiskVolume2
Object: 8a7fa540  Type: (852cd040) Device  <<-- The Type of the object gives more details about how this object is handled.
    ObjectHeader: 8a7fa528 (new version)
    HandleCount: 0  PointerCount: 9
    Directory Object: 86a09390  Name: HarddiskVolume2
 
0: kd> dt nt!*OBJECT*TYPE
          ntkrpamp!_OBJECT_TYPE
 
0: kd> dt nt!_OBJECT_TYPE 852cd040 -r1
   +0x000 TypeList         : _LIST_ENTRY [ 0x852cd040 - 0x852cd040 ]
   +0x008 Name             : _UNICODE_STRING "Device"
   +0x010 DefaultObject    : 0x82781be0 Void
   +0x014 Index            : 0x19 ''
   +0x018 TotalNumberOfObjects : 0x11b
   +0x01c TotalNumberOfHandles : 0
   +0x020 HighWaterNumberOfObjects : 0x11c
   +0x024 HighWaterNumberOfHandles : 1
   +0x028 TypeInfo         : _OBJECT_TYPE_INITIALIZER
          +0x000 Length           : 0x50
          +0x002 ObjectTypeFlags  : 0x5 ''
          +0x002 CaseInsensitive  : 0y1
          +0x002 UnnamedObjectsOnly : 0y0
          +0x002 UseDefaultObject : 0y1
          +0x002 SecurityRequired : 0y0
          +0x002 MaintainHandleCount : 0y0
          +0x002 MaintainTypeList : 0y0
          +0x002 SupportsObjectCallbacks : 0y0
          +0x004 ObjectTypeCode   : 0
          +0x008 InvalidAttributes : 0x100
          +0x00c GenericMapping   : _GENERIC_MAPPING
          +0x01c ValidAccessMask  : 0x1f01ff
          +0x020 RetainAccess     : 0
          +0x024 PoolType         : 0 ( NonPagedPool )
          +0x028 DefaultPagedPoolCharge : 0
          +0x02c DefaultNonPagedPoolCharge : 0xe8
          +0x030 DumpProcedure    : (null) 
          +0x034 OpenProcedure    : (null) 
          +0x038 CloseProcedure   : (null) 
          +0x03c DeleteProcedure  : 0x827fa2b4        void  nt!IopDeleteDevice+0
          +0x040 ParseProcedure   : 0x828843d2        long  nt!IopParseDevice+0
          +0x044 SecurityProcedure : 0x828a319d        long  nt!IopGetSetSecurityObject+0
          +0x048 QueryNameProcedure : (null) 
          +0x04c OkayToCloseProcedure : (null) 
   +0x078 TypeLock         : _EX_PUSH_LOCK
      +0x000 Locked           : 0y0
      +0x000 Waiting          : 0y0
      +0x000 Waking           : 0y0
      +0x000 MultipleShared   : 0y0
      +0x000 Shared           : 0y0000000000000000000000000000 (0)
      +0x000 Value            : 0
      +0x000 Ptr              : (null) 
   +0x07c Key              : 0x69766544
   +0x080 CallbackList     : _LIST_ENTRY [ 0x852cd0c0 - 0x852cd0c0 ]
      +0x000 Flink            : 0x852cd0c0 _LIST_ENTRY [ 0x852cd0c0 - 0x852cd0c0 ]
      +0x004 Blink            : 0x852cd0c0 _LIST_ENTRY [ 0x852cd0c0 - 0x852cd0c0 ]
{% endhighlight %}

Now that we know the object type details, lets dump the object’s header(objectheader) and body(deviceobject)

{% highlight asm %}
0: kd> !object 86bda030   
Object: 86bda030  Type: (85261de8) SymbolicLink  <-- object
    ObjectHeader: 86bda018 (new version)  <<--header
    HandleCount: 0  PointerCount: 1
    Directory Object: 86a05128  Name: C:
    Target String is '\Device\HarddiskVolume2'
    Drive Letter Index is 3 (C:)
 
0: kd> dt nt!*Object*Header
          ntkrpamp!_OBJECT_HEADER
 
0: kd> ??sizeof( nt!_OBJECT_HEADER )
unsigned int 0x20
 
0: kd> dt nt!_OBJECT_HEADER 86bda018 
   +0x000 PointerCount     : 0n1
   +0x004 HandleCount      : 0n0
   +0x004 NextToFree       : (null) 
   +0x008 Lock             : _EX_PUSH_LOCK
   +0x00c TypeIndex        : 0x4 ''
   +0x00d TraceFlags       : 0 ''
   +0x00e InfoMask         : 0x2 ''
   +0x00f Flags            : 0x12 ''
   +0x010 ObjectCreateInfo : 0x00000001 _OBJECT_CREATE_INFORMATION
   +0x010 QuotaBlockCharged : 0x00000001 Void
   +0x014 SecurityDescriptor : 0x86a09136 Void
   +0x018 Body             : _QUAD  <-- body is at 18th offset (86bda018+18)
 
0: kd> ? 86bda018+18
Evaluate expression: -2034393040 = 86bda030
 
0: kd> dt nt!*Device*object
          ntkrpamp!_DEVICE_OBJECT
 
0: kd> dt nt!_DEVICE_OBJECT 8a7fa540  
   +0x000 Type             : 0n3
   +0x002 Size             : 0x1a0
   +0x004 ReferenceCount   : 0n1915
   +0x008 DriverObject     : 0x853ff098 _DRIVER_OBJECT
   +0x00c NextDevice       : 0x8a7fa778 _DEVICE_OBJECT
   +0x010 AttachedDevice   : 0x8a7fc668 _DEVICE_OBJECT
   +0x014 CurrentIrp       : (null) 
   +0x018 Timer            : (null) 
   +0x01c Flags            : 0x1150
   +0x020 Characteristics  : 0
   +0x024 Vpb              : 0x8a7fa4b8 _VPB
   +0x028 DeviceExtension  : 0x8a7fa5f8 Void
   +0x02c DeviceType       : 7
   +0x030 StackSize        : 8 ''
   +0x034 Queue            : <unnamed-tag>
   +0x05c AlignmentRequirement : 1
   +0x060 DeviceQueue      : _KDEVICE_QUEUE
   +0x074 Dpc              : _KDPC
   +0x094 ActiveThreadCount : 0
   +0x098 SecurityDescriptor : 0x86bab630 Void
   +0x09c DeviceLock       : _KEVENT
   +0x0ac SectorSize       : 0x200
   +0x0ae Spare1           : 1
   +0x0b0 DeviceObjectExtension : 0x8a7fa6e0 _DEVOBJ_EXTENSION
   +0x0b4 Reserved         : (null) 
 
0: kd> !devobj 8a7fa540  
Device object (8a7fa540) is for:
 HarddiskVolume2 \Driver\volmgr DriverObject 853ff098
Current Irp 00000000 RefCount 1915 Type 00000007 Flags 00001150
Vpb 8a7fa4b8 Dacl 86bab644 DevExt 8a7fa5f8 DevObjExt 8a7fa6e0 Dope 8a7fa470 DevNode 8a7fa100 
ExtensionFlags (0xc0000800)  DOE_DEFAULT_SD_PRESENT, DOE_BOTTOM_OF_FDO_STACK, 
                             DOE_DESIGNATED_FDO
Characteristics (0000000000)  
AttachedDevice (Upper) 8a7fc668 \Driver\fvevol
Device queue is not busy.

{% endhighlight %}

