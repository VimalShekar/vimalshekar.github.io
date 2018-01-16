---
layout: post
title: Capturing the Standard Output of a commandline tool in PowerShell
date: 2017-09-10
category: ScriptSamples
---

There are many ways to launch Command line tools in PowerShell. The & operator, using built in Cmdlets like Invoke-Command, Start-Process are a few options. One draw back of this approach is that it is difficult to capture the text returned by the command line tool to Standard output console. 

Using the ">" operator to pipe the output to a text is one option. But what if I told you there is a better way ?

Try this sample:
https://github.com/VimalShekar/PowerShell/blob/master/InvokeCommandLine.ps1

{% github_sample_ref /VimalShekar/PowerShell/blob/master/InvokeCommandLine.ps1 %}

In this sample, I use System.Diagnostics.Process to launch the process and then use the ReadToEnd() method of the Standard output stream to retrieve any messages printed by the process. This retains all formatting and is returned as an object.