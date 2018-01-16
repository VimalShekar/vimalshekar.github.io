---
layout: post
title: Capturing the Standard Output of a commandline tool in PowerShell
date: 2017-09-10
category: ScriptSamples
---

There are many ways to launch Command line tools in PowerShell. The & operator, using built in Cmdlets like Invoke-Command, Start-Process are a few options. One draw back of this approach is that it is difficult to capture the text returned by the command line tool to Standard output console. 

Using the ">" operator to pipe the output to a text is one option. But what if I told you there is a better way ?


In this sample, I use System.Diagnostics.Process to launch the process and then use the ReadToEnd() method of the Standard output stream to retrieve any messages printed by the process. This retains all formatting and is returned as an object.

{% highlight powershell %}
<#
.Synopsis
    Executes a given process/command line and returns the exit code or Std output
    
.DESCRIPTION
    Executes a given process/command line and returns the exit code or Std output.
    This function uses System.Diagnostics.Process to launch the process. The plus point of using 
    this class is that the stdout can be captured into a variable and can be printed to a file
    without any loss in formatting.
    
    Author : vimalsh@live.com
    Latest version at : https://github.com/VimalShekar/PowerShell/blob/master/InvokeCommandLine.ps1

.EXAMPLE
    $output = Invoke-CommandLine -CommandLine "ipconfig" -Arguments "/all" -ReturnStdOut

#>

Function Invoke-CommandLine 
{
    param(  
        # Executable name or path
        [string]$CommandLine, 

        # Switches or Arguments to be passed
        [string] $Arguments, 

        # If this switch is given, function returns the standard output of the process, else it returns exitcode
        [switch] $ReturnStdOut,

        # Wait time.
        # -1 indicates wait till exit, 0 indicates no wait and any +ve int will be used as wait time in seconds
        [int] $WaitTime = 0 # enter -1, 0 or + integer        
    )
    
    $psExitCode = -1
    $OutVar = ""
    
    Write-Host "Invoke-CommandLine: $CommandLine $Arguments" | Out-Null    
    try 
    {        
        $ps = new-object System.Diagnostics.Process
        $ps.StartInfo.Filename = $CommandLine
        $ps.StartInfo.Arguments = $Arguments
        $ps.StartInfo.RedirectStandardOutput = $True
        $ps.StartInfo.UseShellExecute = $false
        $ps.StartInfo.WindowStyle = [System.Diagnostics.ProcessWindowStyle]::Hidden
        $ps.Start() | Out-Null

        if($WaitTime -ne 0)
        {
            $ps.WaitForExit($WaitTime)
        }

        [string] $OutVar = $ps.StandardOutput.ReadToEnd();
        $psExitCode = $ps.ExitCode

    }
    catch {
        Write-Host "Invoke-CommandLine: Exception when running process. Details: $($_.Exception.Message)"  | Out-Null
    }

    if($ReturnStdOut)
    {
        return $OutVar
    }
    else {
        return $psExitCode
    }
}

{% endhighlight %}