---
layout: post
title: Enabling or Disabling a given Event Viewer Channel using Powershell
date: 2017-11-21
category: ScriptSamples
---

Windows provides many hidden gems to troubleshoot issues. Among them are the advanced ETW channels exposed in the Windows Event viewer. Recently I was troubleshooting a printer issue for a customer and the admin a customer wanted a way to track information about print jobs, when are they spooled, rendered and completed. I did some testing and found that Windows already logs all that information, all he needed to do was to enable PrintService Admin logging and then create a custom view for it. 


For this particular issue - Information about job spooling time, rendering time, job print complete time is logged into event IDs  800, 805, 307 in the PrintService Operational logs. You will need to enable the logging for this to work. To enable this logging:
1. Go to: Event Viewer > Applications and Service Logs > Microsoft > Windows > Print Service > Operational
2. Right click and choose "Enable log". If the logging is already enabled, then you may see "Disable Log" in place of "Enable Log".


If you don't want to do it using the UI - you can always use wevtutil command to achieve the same thing. 
    `wevtutil el`                     -- lists all the event channels
    `wevtutil sl <logname> /e:true`   -- enables the event channel
    `wevtutil sl <logname> /e:false`  -- disables the event channel
    
Another option is to use the System.Diagnostics.Eventing.Reader.EventLogConfiguration class in a PowerShell script. Here's my sample script: 
<script src="https://gist.github.com/VimalShekar/0513fe1170be0b818bbf6d63713b7198.js"></script>