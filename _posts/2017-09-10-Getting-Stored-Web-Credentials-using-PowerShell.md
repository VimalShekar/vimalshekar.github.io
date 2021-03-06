---
layout: post
title: Getting User Names and Passwords from Windows Credential Manager using PowerShell - Part 1 - Web Credentials
date: 2017-09-10
category: ScriptSamples
---

Interested in dumping all the passwords stored in the Web Credentials portion of the Windows Credential Manager? 
Well, a .NET class exposes methods to do this and you can easily invoke these from PowerShell. 
Windows.Security.Credentials.PasswordVault documented here : https://docs.microsoft.com/en-us/uwp/api/windows.security.credentials.passwordvault
 

The documentation says – “contents of the locker are specific to the app or service. Apps and services don’t have access to credentials associated with other apps or services”. This is only true when you are running within a WinRT container. If you call this class outside of a WinRT container – it gives you access to all credentials in the locker.


Here’s my sample:
<script src="https://gist.github.com/VimalShekar/7ebf2e8787a1ccab7379902ad7b76fbb.js"></script>

Obviously this only gathers credentials of the currently logged in user. If you want to retrieve passwords for another user – you will have to first create a logon session for that account, then impersonate that context and run this function again. I have already posted an impersonation sample earlier, you could easily modify that to achieve your goal.
Obviously I’m not going to walk you through that, I’m with the good guys 🙂