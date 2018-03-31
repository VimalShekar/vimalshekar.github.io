---
layout: post
title: Console Tricks with Powershell
date: 2017-12-30
category: CodeSamples
---

I was recently writing a script that performs tasks that take time. I wanted a way to show the status to the user on a single line and keep updating the line. There is the Write-Progress cmdlet, but I didn't want to use that. Playing with the Console has been relatively difficult in PowerShell, but its easier in .NET. 

Turns out things you can use the System.Console class to achieve all kinds of interesting console behaviors. Here's my script that re-writes the same line with different texts:


{% highlight powershell %}

# Save this as a .ps1 file, and execute it using powershell

#
# Function to implement Same line printing
#
function Update-ConsoleLine ( 
    # Message to be printed
    [Parameter(Mandatory = $True, Position = 0)] 
    [string] $Message,

    # Cursor position where message is to be printed
    [int] $Leftpos = -1,
    [int] $Toppos = -1,

    # Foreground and Background colors for the message
    [System.ConsoleColor] $ForegroundColor = [System.Console]::ForegroundColor,
    [System.ConsoleColor] $BackgroundColor = [System.Console]::BackgroundColor,
    
    # Clear whatever is typed on this line currently
    [switch] $ClearLine,

    # After printing the message, return the cursor back to its initial position.
    [switch] $StayOnSameLine
) 
{
    # Save the current positions. If StayOnSameLine switch is supplied, we should go back to these.
    $CurrCursorLeft = [System.Console]::get_CursorLeft()
    $CurrCursorTop = [System.Console]::get_CursorTop()
    $CurrForegroundColor = [System.Console]::ForegroundColor
    $CurrBackgroundColor = [System.Console]::BackgroundColor

    
    # Get the passed values of foreground and backgroun colors, and left and top cursor positions
    $NewForegroundColor = $ForegroundColor
    $NewBackgroundColor = $BackgroundColor

    if ($Leftpos -ge 0) {
        $NewCursorLeft = $Leftpos
    } else {
        $NewCursorLeft = $CurrCursorLeft
    }

    if ($Toppos -ge 0) {
        $NewCursorTop = $Toppos
    } else {
        $NewCursorTop = $CurrCursorTop
    }

    # if clearline switch is present, clear the current line on the console by writing " "
    if ( $ClearLine ) {                        
        $clearmsg = " " * ([System.Console]::WindowWidth - 1)  
        [System.Console]::SetCursorPosition(0, $NewCursorTop)
        [System.Console]::Write($clearmsg)            
    }

    # Update the console with the message.
    [System.Console]::ForegroundColor = $NewForegroundColor
    [System.Console]::BackgroundColor = $NewBackgroundColor    
    [System.Console]::SetCursorPosition($NewCursorLeft, $NewCursorTop)
    if ( $StayOnSameLine ) { 
        # Dont print newline at the end, set cursor back to original position
        [System.Console]::Write($Message)
        [System.Console]::SetCursorPosition($CurrCursorLeft, $CurrCursorTop)
    } else {
        [System.Console]::WriteLine($Message)
    }    

    # Set foreground and backgroun colors back to original values.
    [System.Console]::ForegroundColor = $CurrForegroundColor
    [System.Console]::BackgroundColor = $CurrBackgroundColor

}

# Examples:
Update-ConsoleLine "Beginning process..." -ForegroundColor "White" -StayOnSameLine -ClearLine
Sleep(1)
Update-ConsoleLine "Progress: [..........]" -ForegroundColor "DarkGray" -StayOnSameLine -ClearLine
Sleep(2)
Update-ConsoleLine "Progress: [ooo.......]" -ForegroundColor "DarkGray" -StayOnSameLine -ClearLine
Sleep(2)
Update-ConsoleLine "Progress: [ooooooo...]" -ForegroundColor "Cyan" -StayOnSameLine -ClearLine
Sleep(2)
Update-ConsoleLine "Progress: [oooooooooo]" -ForegroundColor "Green" -StayOnSameLine -ClearLine
Sleep(1)
Update-ConsoleLine "Process Completed." -ForegroundColor "DarkGreen"
Update-ConsoleLine "Begining Next process..." -ForegroundColor "DarkGreen"

{% endhighlight %}

Note that this code does not work when you copy paste this on the console and execute it line by line. It also does not work within PowerShell ISE, or VSCode. You have to run it as a script to see it in action.

Hope this helps!
