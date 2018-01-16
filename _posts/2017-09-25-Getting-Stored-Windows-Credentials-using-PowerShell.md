---
layout: post
title: Getting User Names and Passwords from Windows Credential Manager using PowerShell - Part 2 - Windows Credentials
date: 2017-09-25
category: ScriptSamples
---

In Part 1 of this series, we found a way to dump the Web Credentials portion of Credential Manager. In this part, we will dump the Windows Credentials portion of Credential Manager. To do this, we make use of a Win32 API CredEnumerate() exposed in Advapi32.dll.

I have written a C# wrapper class that imports the API. After we call the function, we get back an array of CREDENTIAL structures.

{% highlight cpp %}
    [DllImport("Advapi32.dll", SetLastError = true, EntryPoint = "CredEnumerate")]
    public static extern bool CredEnumerate([In] string Filter, [In] int Flags, out int Count, out IntPtr CredentialPtr);    
    ...
    CredEnumerate(Filter, Flags, out count, out pCredentials)  
{% endhighlight %}


Next, we loop through this array to retrieve the credentials. Each CREDENTIAL structure has a UserName, TargetName and CredentialBlob. For different type of credentials, the target name contains certain patterns.

TERMSRV\                    --> RDP saved credential
https://, http://, ftp:     --> Web
domain:target=              --> SMB credential
microsoftoffice             --> Outlook
and so on...

We can classify the credential based on pattern match and print it to the screen
Here’s the full sample:
<script src="https://gist.github.com/VimalShekar/d6a7080679a33e1ac71507a54b49dc18.js"></script>
