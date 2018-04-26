---
layout: post
title: Deception Technology? What is that?
date: 2018-04-11
category: ScriptSamples
---

## Deception Technology:
Kind of sounds cool, but what is it really?

Ever heard of honeypots and honeynets? Well - Deception technology is kind of the same thing - It's old wine in a new bottle. But wait --  this bottle is more sparkly, comes with crystal glasses, beautiful barmaids and big signboards that you can see from miles away -that say "Free Wine!".... 

## What makes it so popular?
Conventional security software (existing AVs, firewalls, HIDs and NIDs) uses signature based and behavior based detection techniques. Some even claim to use advanced machine learning techniques like binary heuristics, branch prediction, user behavior analysis, packet analysis and so on. The problem with all of these products is that they have a pre-constructed perception of good and bad. In other words - they are just running a classification algorithm - that classifies a behavior as genuine user activity or malicious activity based on examples, patterns and data that the system has already seen.

Deception Technology has no perceptions of good or bad. It just creates a web of deceptions and simply waits for literally anyone to access this web. Genuine users wouldn't fall into this web because they *know* the infrastructure - what servers and services his team is allowed to access. But an attacker or anyone else who probes for goodies in the network - will eventually fall into the web. 

Lets see an example: 
Assume your SQL admin frequently logs in to your network from home via a VPN and performs activities such as backing up databases, file on file servers, cleaning out logs, resetting user passwords, creating, deleting, adding members to some SQL groups etc. Assume an attacker gains access to his laptop. He now knows the SQL admin's password, has a secure VPN connection to the company network, and has fairly decent privileges too. What he does next - is upto his imagination and skill. He can searching for file servers to copy files to/from, he mines for credentials from applications like chrome, mozilla, IE, edge, putty, filezilla, Credential manager etc on the laptop, he can even access SQL servers, websites, reporting servers, CRM, incident management, backup servers, etc based on data in SQL management studio or browser history. Would a conventional security software treat it as malicious activity ? probably not!


## Enter deception technology!:
With deception technology deployed on all endpoints and on your network - you can be sure that a good percentage of servernames and credentials that the attacker has gathered lead to a deceptive target. Applications like chrome, mozilla, IE, edge, putty, filezilla, Credential manager etc on the laptop have been carefully patches with credentials that lead to deceptive servers. You can be sure that for every real credential he finds, there would be at least 2 fake ones to confuse him. All activities, such as ping sweeps, port scans, database access, file copy, smb, http requests, RDP, etc work - just as they would on real servers. The attacker literally has nothing to differentiate here. When an attacker uses a deceptive target - he has been *detected*! :p) 

As for the attacker - he sees a real AD with real users and real machines, a real DNS server with real IPs, real web webservers with real content, real file servers with real files, real databases, real application servers and so on. As he probes, his activity is tracked and reports are generated in real time as to what he is interested in. The web deepens with more engagement, pulling him in deeper. New servers are spinned up, and new user credentials are cached for services that he is likely to use. This keeps him engaged for a while (lets say days!)... Enough time for the security team to get involved, right?

Advanced deceptive solutions allow you to configure workflows whenever an attack has been detected. These workflows integrate with other security products such as endpoint and network firewalls, and can isolate the endpoint from where the attack originated from so that further damage can be averted. 

# The players!
There are lots of players in the Deception space - Attivo Networks, Illusive Networks, Smoke screen, Cymmetria, TrapX, Acalvio etc. are just a few names that have been going around. If you are planning on deploying deception in your environment - I would highly recommend Attivo and Illusive. These two are definitely the top ones, and the other players are not up to the mark and the features they offer are too shallow. If you need any help for deploying your deception solution - feel free to contact me. Attivo was my previous employer - so I can authoritatively speak about some of the features offered by them and make fairly accurate guesses about the features offered by some of their competitors. (No trade secrets or internals, sorry - that's against my work ethics)

Hope this helps! 

