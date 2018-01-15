---
layout: post
title: Unlink GPO and delete them using PowerShell
date: 2017-02-09
category: ScriptSamples
---

I ran into an unusual scenario where I wanted to unlink a GPO from all OUs that it was linked to, and then delete the GPO. I had to write it in such a way that it worked on all platforms (Windows 2008 through 2016) and it had minimum dependencies (read as - it shouldn't depend upon the Active directory module).

Eventually I found gpmc had COM classes that could do the job. After a lot of testing, I wrote a PowerShell wrapper that could get my job done.  It takes the name of the domain and name of the GPO object as input. It checks to see if the GPO is linked anywhere. If it isn't then we proceed with deleting the GPO.

Here's my sample:
<script src="https://gist.github.com/VimalShekar/adb77ebaf6c5ca3fd25b0ec656465a81.js"></script>