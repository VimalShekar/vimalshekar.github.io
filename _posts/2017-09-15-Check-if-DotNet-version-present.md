---
layout: post
title: Checking if .NET framework is present in PowerShell
date: 2017-09-15
category: ScriptSamples
---

Many a times, we have to check the version of Powershell to understand whether or not an assembly/namespace is available on the machine. Microsoft documentation recommends checking the **HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\NET Framework Setup\NDP** registry key.

Here’s a function I wrote to find out if the version of .NET on the machine is greater than 2.0:
<script src="https://gist.github.com/VimalShekar/8cfabf6361c6e313c5c829eb36c32035.js"></script>
 
You can modify the version string to check the desired version. For example, I changed the string to look for 3.5 or later:
<script src="https://gist.github.com/VimalShekar/099b7f08d656fe075edf6a24fed1c578.js"></script>

For version 4, the string becomes "v4". You should also check the release version under **HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\NET Framework Setup\NDP\v4\Full -- Release key**


Refer: https://docs.microsoft.com/en-us/dotnet/framework/migration-guide/how-to-determine-which-versions-are-installed

Hope this helps
