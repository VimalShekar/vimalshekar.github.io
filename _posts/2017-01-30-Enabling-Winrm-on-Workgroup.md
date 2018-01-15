---
layout: post
title: Enabling WinRM on a workgroup machine
date: 2017-01-30
category: ScriptSamples
---

I’ve been trying to get WinRM enabled on my test machine that is not domain joined. If you run winrm quick config on newer operating systems, and if the network location on any of your NIC cards are set to Public, winrm quickconfig fails and throws and error. To get it to work, you have to change the Network location to something other than Public. I had to do this repeatedly, because I keep reverting my machine to its initial snapshot. So I scripted it out.

In my case, these are test machines I use for my hacking lab and I dont connect them to any external network. So I was ok to do this. I advice caution and review before using this script. Don’t open security holes on your machine! 

Sample script:
<script src="https://gist.github.com/VimalShekar/810448f1887c750869ff693a43637967.js"></script>