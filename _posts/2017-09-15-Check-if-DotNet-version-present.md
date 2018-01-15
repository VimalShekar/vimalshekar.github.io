---
layout: post
title: Checking if .NET framework is present in PowerShell
date: 2017-09-15
category: Personal
---

Many a times, we have to check the version of Powershell to understand whether or not an assembly/namespace is available on the machine. Microsoft documentation recommends checking the 

Here’s a function I wrote to find out if the version of .NET on the machine is greater than 2.0:
<script src="https://gist.github.com/VimalShekar/8cfabf6361c6e313c5c829eb36c32035.js"></script></p>

You can modify the version string to check the desired version. For example, I changed the string to look for 3.5 or later:
<script src="https://gist.github.com/VimalShekar/099b7f08d656fe075edf6a24fed1c578.js"></script></p>

Hope this helps
