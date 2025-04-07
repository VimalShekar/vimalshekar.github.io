---
layout: post
category: WindowsInternals
title: The Windows Boot Process Part 1
date: 2016-01-22
---

The Windows XP boot process is a layered “chain of hand-offs” that begins at the hardware level and works its way up through several specialized files like we discussed in the previous blog. You can look at the most common error messages to determine which phase it fails and then figure out the steps taken to troubleshoot will be different.

1. **Failures during POST (Power-On Self Test):**<br>
BIOS based machines make a "Beep" sound at the end of the Power-On Self Test. All Failures before the beep are hardware related, you might have to troubleshoot the hardware (re-seat the memory/CPU, check the powersupply voltages, temperature, fan control etc).


2)  **Failures during initial disk access:**<br>
If boot fails at this point, it could be due to several reasons such as - corrupt MBR, corrupt PBS, hardware issues causing problems accessing the disk.


       > "Operating System not found"<br>

       Bios looks for MBR’s 55AAh and starts the Boot code in the MBR. This error implies that the 55AAh signature is not present in the MBR. Typically you can fix this by loading the disk on another machine and using a hex editor tool like winhex or dskprobe from the windows 2000 resource toolkit. 	
       
       <br>

       > "Invalid Parition Table"
                     
       MBR code looks for active partition, only one partition is expected to be set to active. Look for "80" here:
       <image>
       Invalid Partition Table error typically means there are multiple active partitions or no active partitions. You can solve it by marking the right partition as active (has 0x80) and ensuring the other partitions have 0x00

       
              
       <br>

       > Error Loading Operating System
       
       Invalid boot sector. MBR code loads boot code from the first sector of the partition and looks for 55AAh at the end of the partition boot sector.
       
       <br>

       > "Missing Operating System"

       This means the 55AAh signature is missing from the partition boot sector. Use a hexeditor to fix it            
       <br>

       > NTLDR is missing

       Corrupt or missing NTLDR file. Boot by copying 	
       NTLDR, NTDETECT.COM, and BOOT.INI to a floppy drive  - typically you should be able to boot by inserting a floppy and choosing it as the boot device.


                     
       <br>

       > “Disk error” or “A disk read error occurred”	
       
       NTLDR is located on a bad sector.	Chkdsk /f /r
       or replace the NTLDR file
                     
       <br>

       > Black screen with blinking cursor
       
       Corrupt MBR	
       Windows NT: use FDISK /MBR from the recovery console
              -or-	
       Windows 2000/XP/.NET: FIXMBR from the recovery console
       
       <br>
       <br>
3. **Failures during the Boot Loader Phase:**
<br>These are failures occurring between the boot.ini until Windows splash screen NT screen. All file-system activity during this phase uses BIOS Int13, Extended Int13 or NTBOOTDD.SYS (the storage driver). If boot fails at this point:
- Verify that Extended Int13 is enabled in the system’s BIOS or SCSI BIOS 
- Verify that we are using the correct SCSI drivers (NTBOOTDD.SYS)
- Specify the /SOS switch in the BOOT.INI file and monitor the output to the screen. /SOS switch output can help identify common errors.<br>
Here are some common errors seen in this phase:
       
       > ntoskrnl.exe missing or corrupt
              
       Typically means the ARC path in BOOT.INI file is incorrect. Inspect and correct the ARC path.

       <br>

       > Boot hangs after listing hal

       Possibly the incorrect Hal was loaded/installed. Log in to recovery console – replace with correct hal

       <br>

       > Boot hangs after listing specific drivers 	

       Missing\corrupt drivers. Log in to recovery console – replace\disable the last driver which was loaded

       <br>

       > boot hangs with blank /SOS output

       Typically means that SYSTEM hive is corrupt or missing.  Check SYSTEM32\Config to see if hives are present. Try restoring using hives from the SYSTEM32\Config\Repair\Regback folder
       
       <br>
       <br> 

4. **Failures after the Kernel Phase:**
If the Windows splash screen is displayed, it indicates that the kernel was successfully loaded. However, if the boot fails at this point, the most common errors manifest as Blue Screen of Death (BSOD) or STOP errors. To troubleshoot, you can create a boot log (`c:\winnt\ntbtlog.txt`) by adding the `/bootlog` switch to the ARC path in the `BOOT.INI` file.

#### Common STOP Errors and Troubleshooting Steps

1. **STOP Errors Referencing NTOSKRNL (e.g., STOP 0xA, STOP 0x1E):**
<br>
       - Restore the System hive from the Emergency Repair Disk (ERD) or the SYSTEM32\Config\Repair\Regback folder.<br>
       - Replace the System hive from an offline backup.<br>
       - If there was a recent driver installation/change - Disable the problematic driver by changing the Start state for that driver in the registry or Replace the corrupted driver file.
```
  Note: Service/driver start values can be seen under HKLM\System\Current Control Set\Services\<service or driver name>

       Valid Services start options are: 
         Boot        0x0
         System      0x1
         Automatic   0X2
         Manual      0x3
         Disabled    0x4
       
        Valid Services start options are: 
         Automatic   0X2
         Manual      0x3
         Disabled    0x4
```                     

2. **NTOSKRNL Loads SMSS:**
       - **STOP 0x0000006F (SMSS.EXE Missing or Corrupt):**
       <br>
         - For FAT file systems: Copy the missing `SMSS.EXE` file to the `\system32` folder.

3. **SMSS Loads Hives for HKLM\SAM, HKLM\Security, and HKLM\Software Keys:**
<br>
       - **STOP 0xC0000218 (Missing or Corrupt Registry Hive):**
       <br>
         - Check for missing registry hives. Restore the hive from the ERD or the Repair folder. If unavailable, restore the hive from an offline backup.
         <br>
       - **Error: "Windows NT could not start because the following file is missing or corrupt: `<hive name>`":**
         - Restore the missing hive from an offline backup.

4. **SMSS Creates/Activates the Pagefile:**
<br>
       - **STOP Errors Referencing FASTFAT or NTFS:**
       <br>
         - These errors often indicate hard drive corruption. To resolve:
               - Delete the page file and reboot.
               - Run `chkdsk /f /r` to repair the disk.
               - Address any low drive space conditions.
       <br>
       - **The Computer Stops Responding or Reboots Unexpectedly:**
         - Hard drive corruption or low drive space may be the cause. Follow the same steps as above:
               - Delete the page file and reboot.
               - Run `chkdsk /f /r`.

<br>

        
  
 
5. Logon Phase (from logon until the desktop finishes loading):

       If boot fails at this point - it could be issues with login, LSASS.EXE errors, MSGINA.DLL errors, or some problems with a service that has 0x2 or 0x3 Start values. The first thing to try would be to boot into Safemode.
       
       If Safe Mode works - problem will be with a service, device driver or a program launched during startup. 
       From Safemode, Check:
       • Event Viewer - Check for applicable messages regarding the previous boot attempt
       • Disable Services - disable device drivers and/or stop third party services - Clean boot.
       • Startup Group/Run Key rename HKLM\SOFTWARE\MICROSOFT\WINDOWS\CURRENTVERSION\RUN to \-run Remove any programs listed in the Startup folder
       • MSCONFIG - Windows XP version of MSCONFIG.EXE to disable third-party services and software that loads during startup 
       
          
 
           

In the next post, we'll list out the boot process of modern era operating systems - such as Windows Vista - which sets the standard for modern era operating systems like win7, win8/8.1, win10 and win11. 

Until next time 👋

