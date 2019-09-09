---
title: RAID5 btrfs total failure and subsequent abandonment
date: 2016-03-30
category: devops
tags:
- linux
- btrfs
slug: btrfs_total_failure
summary: btrfs RAID 5 fails entirely and takes my data with it

---

****************************************************
RAID5 btrfs total failure and subsequent abandonment
****************************************************

My btrfs volume failed last night. I tried to rebalance the array to a "JBOD" (just a bunch of disks), it seems that it failed during that rebalance. To really drill down into the failure, it seems based on lines of the device info (bdev...) both those devices have errors. I suspect that these one batch of these read errors were directly due to the failure last time, (this one has rd 76 corrupt 16, other one had corrupt 8 and read 76). Two "failed" devices, during the course of a rebalance, where I was trying to change the structure of the file system. I think I screwed myself over. I would have thought, based on my last rebalance that it would have refused to run, due to fact that there were bad checksums/errors on the device. I have remembered that btrfs took a very long time to get a check/repair ability (fsck), and that was one reason why ext4 was used in `Fedora 24 <https://www.reddit.com/r/linux/comments/3awppp/btrfs_as_default_filesystem_for_fedora_23/>`, although OpenSUSE has made it the default.

My next filesystem for my movie backups, Virtual Machine images, and other files, will probably be ZFS. I remember being told about ZFS back in 2006. I remember just being awe of that filesystem, mostly the fact that it had RAIDz, compression, checksumming, I didn't know a lot about the details of it but it sounded like a comprehensive filesystem. I was wanting to try it but never had the ability to use it, and then I tried to purchase spare Solaris machines on eBay. I was told never to even look at the source code of ZFS or OpenSolaris due to CDDL (I'm not a lawyer, I know the man's intent was good, but it might have been an exaggeration). But this isn't stopping me from trying to run it on Linux now, one difference that I'm not looking forward to is that there is no defragmentation support (neither offline or online, just doesn't exist), and there is no ability to change the file system type like you can with btrfs (even though that is the part that seemed to fail on me). This file system is going to be a secondary file system on my machine, so it being a tainted 3rd party kernel module is not a concern to me. Also another point is that you need ECC memory, which I have on this machine. Considering all that I should be good with ZFS. I will try to post my experiences with it.

Here is the actions I took and most of the output, if anyone comes to this post from a search engine, my volume failed I did not recover the data:
.. code-block:: console

  [root@horse ~]# btrfs balance start -dconvert=single /mnt/extra/
  ERROR: error during balancing '/mnt/extra/': Read-only file system
  There may be more info in syslog - try dmesg | tail
  [root@horse ~]#  
  [root@horse ~]# dmesg | tail
  [ 3710.919178]  [<ffffffff8f1f4d3d>] ? __vma_link_rb+0xfd/0x110
  [ 3710.919182]  [<ffffffff8f25b4b2>] do_vfs_ioctl+0xa2/0x5d0
  [ 3710.919186]  [<ffffffff8f062776>] ? __do_page_fault+0x206/0x4d0
  [ 3710.919190]  [<ffffffff8f25ba59>] SyS_ioctl+0x79/0x90
  [ 3710.919196]  [<ffffffff8f7ec572>] entry_SYSCALL_64_fastpath+0x1a/0xa4
  [ 3710.919228] ---[ end trace 58ee1a8a51b783ac ]---
  [ 3710.919232] BTRFS: error (device sdf) in btrfs_run_delayed_refs:2963: errno=-5 IO failure
  [ 3710.919235] BTRFS info (device sdf): forced readonly
  [ 3734.949833] BTRFS info (device sdg1): disk space caching is enabled
  [ 3734.949839] BTRFS info (device sdg1): has skinny extents
  [   71.460425] BTRFS info (device sdb): disk space caching is enabled
  [   71.460433] BTRFS info (device sdb): has skinny extents
  [   72.576302] BTRFS info (device sdb): bdev /dev/sdd errs: wr 0, rd 76, flush 0, corrupt 16, gen 0
  [   72.576311] BTRFS info (device sdb): bdev /dev/sdf errs: wr 510, rd 158743, flush 170, corrupt 0, gen 0
  [   78.476016] BTRFS error (device sdb): parent transid verify failed on 12670714019840 wanted 95664 found 90828
  [   78.488997] BTRFS error (device sdb): parent transid verify failed on 12670714019840 wanted 95664 found 90828
  [   78.489007] BTRFS error (device sdb): failed to read block groups: -5
  [   78.507884] BTRFS: open_ctree failed
  [root@horse ~]# mount -o degraded /dev/sdf /mnt/extra/  
  mount: wrong fs type, bad option, bad superblock on /dev/sdf,
        missing codepage or helper program, or other error
        In some cases useful info is found in syslog - try
        dmesg | tail or so.
  [root@horse ~]#
  [root@horse ~]# btrfsck --repair --init-csum-tree /dev/sdb
  enabling repair mode
  Creating a new CRC tree
  parent transid verify failed on 12670714019840 wanted 95664 found 90828
  parent transid verify failed on 12670714019840 wanted 95664 found 90828
  parent transid verify failed on 12670714019840 wanted 95664 found 90828
  Ignoring transid failure
  leaf parent key incorrect 12670714019840
  Checking filesystem on /dev/sdb
  UUID: 8289c25f-a8d1-44aa-8c18-6215831d2cae
  Reinit crc root
  parent transid verify failed on 12670694375424 wanted 95645 found 90828
  parent transid verify failed on 12670694375424 wanted 95645 found 90828
  parent transid verify failed on 12670694375424 wanted 95645 found 90828
  Ignoring transid failure
  parent transid verify failed on 12670714019840 wanted 95664 found 90828
  Ignoring transid failure
  parent transid verify failed on 12670714019840 wanted 95664 found 90828
  Ignoring transid failure
  parent transid verify failed on 12670714019840 wanted 95664 found 90828
  Ignoring transid failure
  parent transid verify failed on 12670714019840 wanted 95664 found 90828
  Ignoring transid failure
  parent transid verify failed on 12670714019840 wanted 95664 found 90828
  Ignoring transid failure
  parent transid verify failed on 12670714019840 wanted 95664 found 90828
  Ignoring transid failure
  parent transid verify failed on 12670714019840 wanted 95664 found 90828
  Ignoring transid failure
  parent transid verify failed on 12670714019840 wanted 95664 found 90828
  Ignoring transid failure
  parent transid verify failed on 12670714019840 wanted 95664 found 90828
  Ignoring transid failure
  parent transid verify failed on 12670714019840 wanted 95664 found 90828
  Ignoring transid failure
  parent transid verify failed on 12670714019840 wanted 95664 found 90828
  Ignoring transid failure
  parent transid verify failed on 12670714019840 wanted 95664 found 90828
  Ignoring transid failure
  parent transid verify failed on 12670713872384 wanted 95664 found 90828
  parent transid verify failed on 12670713872384 wanted 95664 found 90828
  bytenr mismatch, want=12670713872384, have=12670713741312
  parent transid verify failed on 12670714019840 wanted 95664 found 90828
  Ignoring transid failure
  parent transid verify failed on 12670714019840 wanted 95664 found 90828
  Ignoring transid failure
  parent transid verify failed on 12670713872384 wanted 95664 found 90828
  parent transid verify failed on 12670713872384 wanted 95664 found 90828
  bytenr mismatch, want=12670713872384, have=12670713741312
  Unable to find block group for 0
  extent-tree.c:289: find_search_start: Assertion `1` failed.
  btrfs check(+0x4e29a)[0x556fccd8d29a]
  btrfs check(btrfs_reserve_extent+0xabe)[0x556fccd9215e]
  btrfs check(btrfs_alloc_free_block+0x5f)[0x556fccd9221f]
  btrfs check(+0x46584)[0x556fccd85584]
  btrfs check(btrfs_search_slot+0x2a2)[0x556fccd863c2]
  btrfs check(btrfs_csum_file_block+0x47f)[0x556fccd9786f]
  btrfs check(+0xf67e)[0x556fccd4e67e]
  btrfs check(cmd_check+0x27be)[0x556fccd72d4e]
  btrfs check(main+0x7d)[0x556fccd4eddd]
  /lib64/libc.so.6(__libc_start_main+0xf1)[0x7fb048cd4731]
  btrfs check(_start+0x29)[0x556fccd4eee9]
  [root@horse ~]#

..


