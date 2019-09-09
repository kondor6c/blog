---
title: Hurricane Matthew Weekend
date: 2016-10-12
category: devops
tags:
- linux
slug: hurricane_matthew_weekend
description: Work that I did over when Hurricane Matthew gently blew in
---

As rain falls from Hurricane Matthew, I can't split wood or  I have started to work on some items, and have recorded various thoughts.

I started to look at the failure of the btrfs volume, this truly appears that the failure was not hardware related; instead it is just corrupt checksums. I ran two SMART tests on all of my hard drives, none of them (I paid particular attention to the drives mentioned that were corrupt) had failed any tests. While SMART is far from perfect, it is the best kind of test that we really have at our disposal and I appreciate having it and smartmontools available. I do not believe I can, in good conscience recommend btrfs, even though I have **love** for open source technology. 

Quassel IRC is a nice due to the fact that I can detach graphical clients and preview links, I wish there was a something similar for BitTorrent. I have liked Qbittorrent, I can easily download magnet links and jump the the completed file to listen to it immediately. But with rtorrent you can keep it centralized. I think I am leaning in that direction since there appear to be organization methods with rtorrent so you can have for example "bitlove podcasts", "audiobooks", "fan fiction", "latest version" and maybe "archive" for images such as Damn Vulnerable Linux, which is hard to obtain any more. I'm having a difficult time having files just scattered.

Fedora seemed to have had a failure with their mirrors, strangely I tried to do a dnf|yum update, and it failed due to a 503; I had never seen that before. On IRC user "misc" reported at 13:16:08, that it was a failed firewall upgrade. Additionally user smooge said that 90% of Fedora's network is the same provider. I thought about it, I don't think more than one provider would have prevented this unless it was a failure by the provider itself. If this was a failed upgrade done by fedora staff/volunteers then it still might have failed if a provider was being used.

Writing some of the scripts I plan on using, I was using KDE's Kate editor. I can't say I like the new look and feel compared to KDE version 4. I use the vi mode, when you do a search and replace it puts you in to the vi mode, which genuinely I am moderately used to using. I know I need to get better at it, but sed works so well.
I will need to start tagging blog entries, this is the longest I have blogged and information management like this is posing some new challenges. All in all, I like writing, since I have moved I don't have many //friends// that share an interest in open source technology or what not, so I think this is an outlet to release that, so to speak.
