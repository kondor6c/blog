---
title: standardization 
date: 2016-08-29
tags:
  - linux
slug: standardization
description: small thoughts on standardization and internal rules/policies
---

===============
Standardization
===============

I am sure many of us have seen the xkcd comic `Standrads<https://xkcd.com/927/>`, if not check it out, its kinda funny. I have been thinking about the debate and later burden that we seem to impose on ourselves through well intended standardization. This came around when there was a discussion at my full time employer about standardizing a spaces around the text and brackets
``only_if {node.fqdn.upcase =~ 'PROD'}``
compared to 
``only_if { node.fqdn.upcase =~ 'PROD' }``

Yes, I believe that the former (spaces) is easier to read. I objected to making a rule about it and burying it somewhere in a wiki that we maintain. I feared it would be a rule that would later become something that would be held against a new team member or contributor; "oh you didn't adhere to our 8 pages of rules". I felt it increased the barrier to entry and wasted time, not only in what we would spend maintaining the document, discussing it, possibly revising it, and the very time we spent debating the merits of having a rule. I thought it was very trivial that if we felt it was that bad to read the author should be able to do as he/she pleases. Rules and regulations are rigid in nature and tend to get buried, lost and forgotten; they grow old and start to become a burden on the organization!

I had thought about some of these rules that Unix must have adopted in a `blog post<https://utcc.utoronto.ca/~cks/space/blog/unix/MoreAndUnixFossilization>` about the utility ``more`` and ``less``. Whereas they don't mention any kind of discussion I can only think about the hours lost making the specification over something rather trivial such as a screen pager. Maybe I'm over-trivializing it? I'm not sure, but certainly in a modern perspective it feels like a minute detail. I worry that any rule I would put in place might become something like saying 'more' can only do these things. Even while the tool set is rapidly advancing.

Having said all that, I am a huge fan of RedHat Enterprise Linux and CentOS, which to many would complain that its PHP is X years old (to which I will typically counter that their OS X BASH version is close to 10 years old, and doesn't support named indexes/hashes). I feel with RHEL they quickly change packages and more are available through the community with `EPEL<https://fedoraproject.org/wiki/EPEL>`. They don't want to introduce a change that might change the behavior of something that is compiled on it. The operating system is a very stable piece of software. Linus even has a rule that the kernel must never `break userspace<http://yarchive.net/comp/linux/gcc_vs_kernel_stability.html>`. I don't know if CentOS/RHEL has a large rule set, it might break my heart if they did, but I would understand why it would be large (and hopefully lively document).

Therefore in conclusion, I still hold my opinion on what rules and regulations are in software. They are global variables. But they have their use, I think they should be used with care like a house plant, too many of them and they become unmanageable, a burden themselves and hard to work around. With care, they become a lively addition to any organization.


