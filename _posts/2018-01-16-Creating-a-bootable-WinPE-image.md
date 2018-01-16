---
layout: post
title: Creating a bootable WinPE iso/image for offline Windows hacking
date: 2018-01-18
category: Walkthroughs
---

Recently one of my virtual machines wasn't booting up after I installed an unsigned driver. To get the machine to boot up - I could have used Last Known Good configuration (LKG), 
but that wouldn't be fun, right?. So I decided to make myself a WinPE image that I could use to boot up and modify the registry. 


** What is WinPE? **
WinPE - stands for Windows Pre-installation Environment. WinPE is a light weight bootable ISO which can be customized to do tasks and service an offline operating system. If you boot to a WinPE, you basically have full access to the file system and can modify files and registry without a problem (assuming Bitlocker was not enabled, offcourse).

The Power of WinPE is under-explained in my post for obvious reasons (I'm on the good guy's side, remember). But I'll tell you this - If you want to offline hack a Windows installation - a winPE iso file loaded with common crack tools and powershell scripts is an ideal weapon. I often prefer this over linux live CDs, mainly because the file system is readily accessible and flexibility of scripting with PowerShell on WinPE cannot be matched.


Here are the steps I followed to make my WinPE image.
* Download the latest version of ADK from Microsoft's website. I installed ADK for Windows 10 from [here](https://docs.microsoft.com/en-us/windows-hardware/get-started/adk-install). You don't have to install all the features in the ADK - just install the deployment tools and pre-installation environment.

* Once it is installed, click start and search for the Deployment tools command prompt. Alternatively, you could Navigate to the folder where ADK is installed (usually `C:\ProgramFiles(x86)\Windows Kits\10` ) and then locate the "Windows Preinstallation Environment" sub folder. 

* Use this command to copy the PE source files.
	```	
		Copype.cmd <architecture> <winpe_source folder>

		Ex: copype x86 C:\PEScratch\winpe_x86      -- note that the command creates the folder "winpe_x86"
	```
	
* Delete "bootfix.bin" in the <winpe_source>\media\boot". This will disable the prompt "press any key to boot from DVD".

* Mount the wim file and copy any tools that you would want to use from your WinPE. To mount the wim file, use this command:
	```
		Ex: Dism /Mount-Image /ImageFile:"<winpe_source>\media\sources\boot.wim" /Index:1 /MountDir:"C:\WinPE_amd64\mount"
	```	

* WinPE now supports PowerShell. You have to add the below packages to get PowerShell within WinPE. *In the below commands - make sure adk folder and mount folder are replaced with correct paths, and <arch> is replaced with correct architecture (x86 or amd64). In my case
	<mountfolder> was `C:\WinPE_amd64\mount` *
	
	```
	Dism /Add-Package /Image:"<mountfolder>" /PackagePath:"<ADKfolder>\Windows Preinstallation Environment\<arch>\WinPE_OCs\WinPE-WMI.cab"
	Dism /Add-Package /Image:"<mountfolder>" /PackagePath:"<ADKfolder>\Windows Preinstallation Environment\<arch>\WinPE_OCs\en-us\WinPE-WMI_en-us.cab"
	Dism /Add-Package /Image:"<mountfolder>" /PackagePath:"<ADKfolder>\Windows Preinstallation Environment\<arch>\WinPE_OCs\WinPE-NetFX.cab"
	Dism /Add-Package /Image:"<mountfolder>" /PackagePath:"<ADKfolder>\Windows Preinstallation Environment\<arch>\WinPE_OCs\en-us\WinPE-NetFX_en-us.cab"
	Dism /Add-Package /Image:"<mountfolder>" /PackagePath:"<ADKfolder>\Windows Preinstallation Environment\<arch>\WinPE_OCs\WinPE-Scripting.cab"
	Dism /Add-Package /Image:"<mountfolder>" /PackagePath:"<ADKfolder>\Windows Preinstallation Environment\<arch>\WinPE_OCs\en-us\WinPE-Scripting_en-us.cab"
	Dism /Add-Package /Image:"<mountfolder>" /PackagePath:"<ADKfolder>\Windows Preinstallation Environment\<arch>\WinPE_OCs\WinPE-PowerShell.cab"
	Dism /Add-Package /Image:"<mountfolder>" /PackagePath:"<ADKfolder>\Windows Preinstallation Environment\<arch>\WinPE_OCs\en-us\WinPE-PowerShell_en-us.cab"
	Dism /Add-Package /Image:"<mountfolder>" /PackagePath:"<ADKfolder>\Windows Preinstallation Environment\<arch>\WinPE_OCs\WinPE-StorageWMI.cab"
	Dism /Add-Package /Image:"<mountfolder>" /PackagePath:"<ADKfolder>\Windows Preinstallation Environment\<arch>\WinPE_OCs\en-us\WinPE-StorageWMI_en-us.cab"
	Dism /Add-Package /Image:"<mountfolder>" /PackagePath:"<ADKfolder>\Windows Preinstallation Environment\<arch>\WinPE_OCs\WinPE-DismCmdlets.cab"
	Dism /Add-Package /Image:"<mountfolder>" /PackagePath:"<ADKfolder>\Windows Preinstallation Environment\<arch>\WinPE_OCs\en-us\WinPE-DismCmdlets_en-us.cab"
	```

* Editing startup script if you want to launch your own scripts or executables. The startup script (startnet.cmd) is located here `<mountfolder>\windows\system32\startnet.cmd`. You may have to take ownership of the file and assign write permissions to yourself. Refer: https://msdn.microsoft.com/windows/hardware/commercialize/manufacture/desktop/wpeinit-and-startnetcmd-using-winpe-startup-scripts
	
	-  I usually just launch powershell.exe in my startnet.cmd. To do this, I just add powershell.exe at the end of the file.
	

* Now that all changes are done, unmount the WinPE image.
	```
	Dism /Unmount-Image /MountDir:C:\test\offline /Commit
	```

* Create bootable media, such as a USB flash drive using MakeWinPEMedia.cmd
	``` 
	MakeWinPEMedia.cmd ISO C:\PEScratch\winpe_x86 C:\winPE.iso 		
	```


Hope this helps!
	
	
	
