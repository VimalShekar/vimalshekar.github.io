---
layout: post
title: Checking if a given username and password is valid in PowerShell
date: 2017-09-10
category: ScriptSamples
---
Validating a given username and password is a common task in many scripts. Online, there are several examples that make use of ValidateCredentials method of System.DirectoryServices.AccountManagement.PrincipalContext class. You could create an instance of this class in PowerShell and then call the method.

Here’s my version that uses PrincipalContext class:
<script src="https://gist.github.com/VimalShekar/fd1429a154f8b6843cfe6439cabfc541.js"></script>

The only problem with this is that this class is only available from .NET version 3.5. If you want this functionality on machines where .NET 3.5 is not installed, there’s another way to do it by using Win32 API.

Here’s another version of my script that checks if the given username and password is valid by using LogonUser Win32 API. It basically imports the API and then calls it.
<script src="https://gist.github.com/VimalShekar/94e5cf1a64c56f0913595bfb910a7f93.js"></script>