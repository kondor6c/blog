title: btrfs Scrub Fail
date: 2016-05-05
category: devops
tags:
- linux
- btrfs
slug: btrfs_srub_fail
description: I experience unrecoverable error on btrfs RAID 5, ignoring previous claims of instability

---

btrfs scrub fail
================
I wrote this earlier on a internal corporate blog (Atlassian's Confluence), but I don't think many read it, it doesn't give out any corporate information, but since I wrote it on "company time" I figured I should put it there. But with the late night phone calls and the subsequent difficult time in falling asleep I think a few minutes here and there all balances out. That's another topic for another time. Here is a repost of the blog. I'm really only putting it here for the sake of context and centralizing my posts. This was written on May 11th 2016.
btrfs
-----
I have been trying/using btrfs. I have really wanted to give it a chance even and I thought a perfect place for that would be at home. So a little bit of background my home machine is a Xeon E5-2620 v3 and it's got 32GB of ECC RAM, I have 4x3TB disks that are in a btrfs RAID 5 setup, it mostly holds torrents (all legal of course), virtual machine disks, and backups (old army documents, pictures). Things had been going rough when I first tried using btrfs (on my root volume), but that turned out to be a bad SSD, nothing to do with the RAID 5. So I kept ext4 on my SSD (rootfs) and I continue to use btrfs on the RAID. Just last week I had a disk fail due to bad sectors, I had time to replace it but after the replace operation.
```bash
[root@horse ~]# btrfs replace start /dev/sdf /dev/sde /mnt/extra
```
I continue to get unrepairable errors.... btrfsck doesn't address it, btrfs scrub on finds the errors. The volume mounts, and unmounts, it just seems that I should be able to address these without going into a recovery, I think it happened during the course of the replace operation. I was able to move the files to a different volume, but it doesn't seem to make a difference. I found the files by looking at the inode that the checksum failed with. As I continue to address these errors I'll keep you posted on the status of my btrfs experience. I was hesitating using ZFS due to the licensing issues. I have long been a fan of ZFS, I think I might try it now; after all I have ECC RAM which is the recommendation. Another thought is to just delete the disk, then re-add it, since it seems to be just one that has the errors.

```bash
[root@horse ~]# btrfs scrub status /mnt/extra
scrub status for 8289c25f-a8d1-44aa-8c18-6215831d2cae
        scrub started at Thu May  5 12:17:58 2016 and finished after 40:13:09
        total bytes scrubbed: 4.52TiB with 12 errors
        error details: read=4 csum=8
        corrected errors: 0, uncorrectable errors: 12, unverified errors: 0
[root@horse ~]# btrfsck --repair /dev/sda
enabling repair mode
Checking filesystem on /dev/sda
UUID: 8289c25f-a8d1-44aa-8c18-6215831d2cae
checking extents
checking free space cache
checking fs roots
checking root refs
found 4961408222736 bytes used err is 0
total csum bytes: 4837631228
total tree bytes: 6034817024
total fs tree bytes: 754761728
total extent tree bytes: 84541440
btree space waste bytes: 420880645
file data blocks allocated: 42712031051776
 referenced 4948738207744
[root@horse ~]# mount /mnt/extra/
[root@horse ~]# dmesg |grep btrfs
[703225.538312] btrfs[10452]: segfault at 30 ip 00005647596ed880 sp 00007fff57c1bb40 error 4 in btrfs[564759698000+92000]
[703232.764835] btrfs[10521]: segfault at 30 ip 00005597082f2880 sp 00007fff2371b050 error 4 in btrfs[55970829d000+92000]
[root@horse ~]# #echo "segfaults with ECC?"
[root@horse ~]# dmesg |tail -n3
[843497.711057] BTRFS info (device sda): relocating block group 12359603060736 flags 129
[843517.301202] BTRFS info (device sda): found 366 extents
[843519.453459] BTRFS info (device sda): found 366 extents
[843519.913111] BTRFS info (device sda): relocating block group 12356381835264 flags 129
[843539.377181] BTRFS info (device sda): found 366 extents
[843540.827521] BTRFS info (device sda): found 366 extents
[843720.099900] BTRFS info (device sda): disk space caching is enabled
[843720.099907] BTRFS: has skinny extents
[843720.995888] BTRFS info (device sda): bdev /dev/sde errs: wr 0, rd 76, flush 0, corrupt 8, gen 0
[843721.573752] BTRFS: checking UUID tree
[root@horse ~]# rpm -qa |grep btrfs-pr
btrfs-progs-4.4.1-1.fc23.x86_64
[root@horse ~]# uname -a
Linux horse 4.4.8-300.fc23.x86_64 #1 SMP Wed Apr 20 16:59:27 UTC 2016 x86_64 x86_64 x86_64 GNU/Linux
```
