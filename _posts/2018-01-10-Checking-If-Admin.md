---
layout: post
title: Checking for Admin Privilege in C++, C# and PowerShell
date: 2018-01-16
category: CodeSamples 
---

Since Windows Vista, UAC has been a key feature in mitigating some of the elevation of privilege risks. Under UAC, accounts in the local Administrators group have two access tokens, one with standard user privileges and one with administrator privileges. All processes (including the Windows explorer - *explorer.exe*) are launched under the standard token which limits the rights and privileges that process has. If the user desires more privileges, he can choose to run the process using "run as Administrator". With this optin - the process now has all privileges and rights of an administrator. 

Because of UAC access token filtering, a script or executable is normally run under the standard user token, unless it is run "as an Administrator" in elevated privilege mode. As a developer/hacker, it is important to understand what mode you are running under. 


Here's a C++ snippet to check for admin rights:

{% highlight cpp %}

BOOL IsProcessElevated()
{
	BOOL fIsElevated = FALSE;
	HANDLE hToken = NULL;
	TOKEN_ELEVATION elevation;
	DWORD dwSize;

	if (!OpenProcessToken(GetCurrentProcess(), TOKEN_QUERY, &hToken))
	{
		printf("\n Failed to get Process Token :%d.",GetLastError());
		goto Cleanup;  // if Failed, we treat as False
	}


	if (!GetTokenInformation(hToken, TokenElevation, &elevation, sizeof(elevation), &dwSize))
	{	
		printf("\nFailed to get Token Information :%d.", GetLastError());
		goto Cleanup;// if Failed, we treat as False
	}

	fIsElevated = elevation.TokenIsElevated;

Cleanup:
	if (hToken)
	{
		CloseHandle(hToken);
		hToken = NULL;
	}
	return fIsElevated; 
}

{% endhighlight %}


If your code is PowerShell you can use this snippet:

{% highlight powershell %}
function IsProcessElevated {
   If (-NOT ([Security.Principal.WindowsPrincipal] [Security.Principal.WindowsIdentity]::GetCurrent()).IsInRole([Security.Principal.WindowsBuiltInRole] "Administrator"))
    {
       return $false
    }
    return $true
}
{% endhighlight %}



If your code is in C#, here's the snippet:
{% highlight cpp %}

using System.Security.Principal;

public static bool IsProcessElevated()
{
    WindowsIdentity identity = WindowsIdentity.GetCurrent();
    WindowsPrincipal principal = new WindowsPrincipal(identity);
    return principal.IsInRole(WindowsBuiltInRole.Administrator);
}

{% endhighlight %}