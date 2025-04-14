---
layout: post
title: Enabling DebugPrints
date: 2019-10-10
category: CodeSamples ScriptSamples
---

Instrumenting your driver with DbgPrint:

Here are easy ways to add logging into your driver:

**Enable output of DbgPrint/KdPrint messages**

- Open (or add, if it's not already there) the key "HKLM\SYSTEM\CCS\Control\Session Manager\Debug Print Filter". 
- Under this key, create a  value with the name "DEFAULT"  
- Set the value of this key equal to the DWORD value 8 to enable xxx_INFO_LEVEL output as well as xxx_ERROR_LEVEL output.  Or try setting the mask to 0xF so you get all output.  
- You must reboot for these changes to take effect.  
> Note: Please create the following key and value.  Note the name is not (default) - you actually have to create one called DEFAULT.<br>
    HKLM\SYSTEM\CCS\Control\Session Manager\Debug Print Filter
    Value Type: REG_DWORD
    Value Name: DEFAULT 
    Value Data: 8
<br>

**Viewing it in a Debugger**

If you haven't set the registry but want to enable this in the debugger while live kernel debugging - you have to change the component filter mask for DPFLTR. 

Starting with Windows Vista you need to set the mask value for the DWORD at Kd_DEFAULT_MASK 
(In windbg "ed Kd_DEFAULT_MASK").

You can specify 8 to enable DPFLTR_INFO_LEVEL output in addition to DPFLTR_ERROR_LEVEL output, or 0xF to get all levels of output. You should now be able to see debug messages in the debugger.


**Viewing it in a DebugView** <br>
Get DebugView
http://technet.microsoft.com/en-us/sysinternals/bb896647.aspx

 
Run it as an Administrator and enable Capture Kernel:
 
1) Then use File->Log To File... to log it as follows.
> ![stages]({{ "/images/debugview-1.png" | absolute_url }})

2) Then the next time this happens, exit DbgView and upload the log file from that day.


