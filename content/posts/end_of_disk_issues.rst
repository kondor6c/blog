---
title: An End of Disk Issues?
date: 2018-08-02
tags:
  - linux
  - zfs
  - hardware
slug: end_of_disk_issues
description: A hopeful end to disk issues on my workstation and ending with a brief mention of topics to come
---

I have a workstation that I have built, it contains a multi-disk Large Form Factor ZFS array. There have been multiple posts in the past about some of my disk related problems that I've handled, you should be able to find them with a 'zfs' or 'disk' tag. Well I had several issues, that I hope should be calming down; going back to January, where I decided to move from 4x3TB RAID-Z (software RAID 5) to 4x4TB 7200RPM RAID 10 setup. The transition was prompted by this event and I really have written this up to share my experience with others. I spoke casually about this with Jim Salter, Michael Hrivnak, Wes Widner, and several others, but I brought it up mostly because of Jim's project `Sanoid <https://github.com/jimsalterjrs/sanoid>` and he's presented on ZFS before. I had one disk start clicking when I got back from running, I could see in ``dmesg`` that this disk was not responding and was trying to be reset; important thing to point out, ZFS had brought the disk OFFLINE. I had a spare 4TB 'eco' drive that really doesn't deliver the performance that is needed and would hinder the array, but I wanted to get the disk replaced so that I could get another one ordered. Well I hooked the disk up, and started the zfs replace process. I woke up and found that another disk, in this RAID-Z, had reported as failing (FAULTED), all during the replace process. I knew this was very bad, two disks failing, and you can't stop a resilver, but you can stop a scrub, by using ``zpool scrub -s ${pool}``, what I should have done is brought the disk "online" but what I did, could have been worse... I rebooted my workstation! During the resilver/replace process, not my finest hour.
This array doesn't hold too much, but it does hold "warm" backups (staged to go on to different disk stored elsewhere), KVM disk images (virtual machines), and other various data, usually high write IO that I don't want to wear on my SSD/NVMe drive, I could stand to lose this array, but I don't want to because it would be painful. When the machine came back, the pool was imported and the resilver restarted, this time with the clicking disk "ONLINE" however the other faulted disk, was now "UNAVAILABLE", so I have a replace going of an online disk that is in a poor state, but ZFS had recovered from this, and the replace completed successfully. I then transferred the data elsewhere, rebuilt the pool as a RAID-10.
The pool is now with all 7200 RPM disks in a RAID-10 setup. ZFS looks at this as one mirror (RAID 1) with a striped disk set (RAID 0), this might be confusing, I will perhaps try to do a video to cover this more. Here is some output of my now healthy disk status:

.. code-block:: bash

   NAME                        STATE     READ WRITE CKSUM
   extra                       ONLINE       0     0     0
     mirror-0                  ONLINE       0     0     0
       wwn-0x5000cca23dcdcfc7  ONLINE       0     0     0
       wwn-0x5000c500675738ba  ONLINE       0     0     0
     mirror-1                  ONLINE       0     0     0
       wwn-0x5000cca22bc39ed8  ONLINE       0     0     0
       wwn-0x50000397fc580978  ONLINE       0     0     0

The issue that I think I had encountered for a while, was that I plugged three 4TB 7200RPM disks using the same SATA power plug coming from the power supply. I had thought I was up against a bad controller/port/cable and I plugged a red SATA cable in to signify that "red is dead", but the issue continued when I plugged it into another SATA port. I realized that when I put the hard drive in a top unit or in a USB3 SATA drive, that the disk was fine it didn't even spin cycle. I believe that this new disk array was consuming more power enough that the disk would power on, but upon doing load wouldn't service the ATA commands quick enough, and linux would then reset it. Similar to this

.. code-block:: bash

   [Wed Jun  6 12:51:54 2018] ata1: link is slow to respond, please be patient (ready=0)
   [Wed Jun  6 12:52:23 2018] ata1: COMRESET failed (errno=-16)
   [Wed Jun  6 12:52:23 2018] ata1: hard resetting link
   [Wed Jun  6 12:52:28 2018] ata1: COMRESET failed (errno=-16)
   [Wed Jun  6 12:52:28 2018] ata1: reset failed, giving up

Giving this "bad disk" a dedicated power plug resolved these issues. I had a similar issue when adding five 1U HP DL160's to a Cabinet, Friday night I hooked them all up, and we started the HDFS rebalance, but upon getting load on those machines, we got a phone call from Sungard saying that we're exceeding the power limit. Load can influence things, in many ways, I knew this; I guess this one just took me longer to realize.

In other news, I have created a video on my camera showing the process of using a Yubikey and ``pass`` to get versioned passwords, but I realized that it was very ad-hoc and I deleted the video. I spent a fair amount of time editing in KDEnlive and audacity, but I just was not pleased with the end product so I will be redoing it and uploading it shortly. I also plan on writing up my home router creation and performance testing of MySQL as I hinted at last blog post. I did some performance testing of docker storage drivers and needless to say we will be avoiding btrfs, I didn't test with ``nodatacow`` as a mount option, but really XFS and overlay2 didn't have that option and performed very well. This test wasn't exhaustive, but enough to give an idea and fulfill proper investigation/baselining to change the storage drivers that my employer is using, without getting into too many details, a contractor configured us to use one that should not be used in production.
