---
title: powershell_linux_debut 
date: 2016-08-19
category: devops
tags:
- linux
slug: powershell_linux_debut
description: Powershell was announced for Linux, I take a quick look
---

Powershell Linux debut
======================

I began to experiment with Powershell. I feel that Microsoft has to follow through on showing the community that it meant what it said, that Microsoft Loves Linux. Perhaps I can lump myself into that group too, because I remember the October Document, and the utter frustration in trying to get Linux to work in a mostly Windows environment. I think Microsoft senses that the game is changing, and is changing with it. Linus Torvalds had said that "If Microsoft ever does applications for Linux it means I've won." and now that they are changing their business practices to bring software to Linux, is totally incredible. Its good as a company they are becoming a software company that makes software for products in demand.

While I am writing this I am trying to use Powershell to some degree, my family members and friends/co-workers have told me to look into Powershell (I think mostly to assist with scripts ;-) but still). My initial observations is that everything is well laid out, with thought, Get-*, Invoke-*, Set-* (you get the idea). However as a shell, something you interact with frequently just to get a look at things it makes features such as tab completion very hard since they're all the same structure. I see both sides, it being good for a consistent theme, but it gets extreme. Really what it feels like is more of an object oriented language shell. Yeah, kinda awkward. I would like to revisit this awkward-ness and either support it with more evidence or 

```
PS /root> type ./.ssh/
type : Access to the path '/root/.ssh/' is denied.
At line:1 char:1
+ type ./.ssh/
+ ~~~~~~~~~~~~~~~~~~~~~
+ CategoryInfo          : PermissionDenied: (/root/.ssh/:String) [Get-Content], UnauthorizedAccessException
PS /root> cat .ssh 
/bin/cat: .ssh: Is a directory
PS /root> 
```
hmmm seems like type should handle a directory? ls -l and find show me its a directory, but type is link a symlink to Get-Content. I could see this causing problems, especially since type is a builtin of BASH. I don't know off the top of my head why Get-Content would be effectively symlinked.

```

PS /root> Get-Process -Name rtorrent
Get-Process : Cannot find a process with the name "rtorrent". Verify the process name and call the cmdlet again.                                                        At line:1 char:1                                                                                                                                                        
+ Get-Process -Name rtorrent
+ ~~~~~~~~~~~~~~~~~~~~~~~~~~
    + CategoryInfo          : ObjectNotFound: (rtorrent:String) [Get-Process], ProcessCommandException
    + FullyQualifiedErrorId : NoProcessFoundForGivenName,Microsoft.PowerShell.Commands.GetProcessCommand
 
PS /root> ps aux |grep rtorrent
ec2-user 26164  0.0  1.0 658724 10488 pts/1    Sl+  Aug03  21:36 rtorrent
PS /root> 
```
It worked on systemd. 

Granted this is an alpha version, I found it hard to get "help" on this item using --help, -help, /help did not... help! Later learned that there is a facility entirely to this Get-Help (aka man!). Also consider that powershell will act as coreutils + more, as in it won't have bc, wget/curl, vi, tar, or others. So powershell needs to include many these functions with itself; Invoke-RestMethod, this will come in handy later! We can set alias' but I thought some of the alias' I would be overwriting Linux utilities and then I would be in a world of pain (Yes I could specify the full path, but still not fun).

In conclusion this is a good thing, I don't want my negativity to make it sound like anything bad. But I could never imagine using this as any kind of a default shell any time soon or ever. I haven't actually got to learn that much about it, I've done more typing than learning.
