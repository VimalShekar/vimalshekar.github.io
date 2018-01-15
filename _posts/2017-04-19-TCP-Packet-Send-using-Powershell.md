---
layout: post
title: Send and Recieve TCP packets in PowerShell.
date: 2017-04-19
category: ScriptSamples
---

I was recently trying some buffer overflow attacks against vulnserver – just sharpening my skills. I had a simple TCP send and receive winsock program writtem in C, but I wanted some more flexibility. Online – most of the available code uses Perl or Python and I didn’t want to trouble myself to download the interpreters, copy and install them on my lab VMs. So – what’s built-in ? PowerShell, yeah! 

I wrote this function and it worked pretty well. Decided to share it out. Add one point to the powershell community!……..

<script src="https://gist.github.com/VimalShekar/35265d71d7bbce5dff4212db886990d4.js"></script>