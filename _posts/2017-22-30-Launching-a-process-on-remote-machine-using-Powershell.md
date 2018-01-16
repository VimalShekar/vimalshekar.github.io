---
layout: post
title: Launching a process on remote machine using WMI with PowerShell
date: 2017-12-30
category: CodeSamples
---

I yesterday's post - I have provided C++ code that could connect to a remote machine and launch a process. This time, I will repeat this using PowerSHell.

Here's a PowerShell function I wrote that uses Invoke-WmiMethod cmdlet. You could also achieve the same using Invoke-CimInstance to create a Win32_Process instance and then call its create method.


{% highlight powershell %}

#
# Run Remote Process using WMI
#
function Run-RemoteProcess
{
	param(
		[string] $Cmdline,
		[string] $Arguments,
		[string] $RemoteComputerName,
		[System.Management.Automation.PSCredential] $Credential
	)
	try{
		$cmd1 = $Cmdline + " " + $Arguments
        Write-Host "Run-RemoteProcess: Executing $cmd1 on $RemoteComputerName"
		
		$localmachine = $env:computername.ToLower() 
		$RemoteComputerNameLower = $RemoteComputerName.ToLower()
		if($RemoteComputerNameLower.contains($localmachine))
		{	#dont use -credentials if running locally
			$proc = Invoke-WmiMethod -Class Win32_Process -Name Create -ArgumentList $cmd1 -ComputerName $RemoteComputerNameLower
		}
		else {
			$proc = Invoke-WmiMethod -Class Win32_Process -Name Create -ArgumentList $cmd1 -ComputerName $RemoteComputerNameLower -Credential $Credential 
		}
	}
	catch{
		Write-Host "Run-RemoteProcess: Exception"
	}
}


# Sample Usage:
# $secpass = ConvertTo-SecureString "Test@Pass1" -AsPlainText -Force
# $DomainCreds = New-Object System.Management.Automation.PSCredential ("testdom.com\testuser1", $secpass)  
# Run-RemoteProcess -Cmdline "ipconfig.exe" -Arguments "/all" -Credential $DomainCreds -RemoteComputerName "testcomp1.testdom.com"

{% endhighlight %}


To launch the payload, in the last post my approach was to map a network share, and then launch the payload from that share. This involves running the following commands in sequence:
```
    net use \\<servername>\<sharename> /user:<username> <password>
    \\<servername>\<sharename>\<payload.exe> <switches>
    net use \\<servername>\<sharename> /delete
```


I can achieve the same in powershell by passing appropriate arguments to my Run-RemoteProcess powershell function. 

{% highlight powershell %}

#Sample Usage:
$secpass = ConvertTo-SecureString "Test@Pass1" -AsPlainText -Force
$DomainCreds = New-Object System.Management.Automation.PSCredential ("testdom.com\testuser1", $secpass)  

Run-RemoteProcess -Cmdline "net.exe" -Arguments "use \\<servername>\<sharename> /user:<username> <password>" -Credential $DomainCreds -RemoteComputerName "testcomp1.testdom.com"
Run-RemoteProcess -Cmdline "\\<servername>\<sharename>\<payload.exe>" -Arguments "<switches>" -Credential $DomainCreds -RemoteComputerName "testcomp1.testdom.com"
Run-RemoteProcess -Cmdline "net.exe" -Arguments "use \\<servername>\<sharename> /delete" -Credential $DomainCreds -RemoteComputerName "testcomp1.testdom.com"

{% endhighlight %}
