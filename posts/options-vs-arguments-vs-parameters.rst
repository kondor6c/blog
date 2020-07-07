---
title: options-vs-arguments-vs-parameters 
date: 2016-09-26
category: devops
tags:
  - linux
slug: options-vs-arguments-vs-parameters
description: trivial discussion about semantics of positioning vocabulary
---

=================================
options, arguments, or parameters
=================================

Where I am currently employed there has been, in my opinion, an over application of the term "parameter". Or at least from where I have been in the past. I have always thought that for example:

.. code-block:: bash

   example -x test.txt

-x would be a switch, and the option after -x, test.txt, would be argument. But it seems here or recently we apply the term parameter to everything. Which I guess I'm not opposed to, I'm just observing. Is this trivial, yes, very trivial. But I'm not really trying to advocate change or anything like that. Let's look at the man page for sort:

.. code-block:: bash

   SORT(1)                                                               User Commands                                                               SORT(1)
   
   NAME
          sort - sort lines of text files
   
   SYNOPSIS
          sort [OPTION]... [FILE]...
          sort [OPTION]... --files0-from=F
   
   DESCRIPTION
          Write sorted concatenation of all FILE(s) to standard output.
   
          With no FILE, or when FILE is -, read standard input.
   
          Mandatory arguments to long options are mandatory for short options too.  Ordering options:
   
          -b, --ignore-leading-blanks
                 ignore leading blanks
   
          -d, --dictionary-order
                 consider only blanks and alphanumeric characters
   
          -f, --ignore-case
                 fold lower case to upper case characters
   
          -g, --general-numeric-sort
                 compare according to general numerical value
   
          -i, --ignore-nonprinting
                 consider only printable characters
   
          -M, --month-sort
                 compare (unknown) < 'JAN' < ... < 'DEC'
   
          -h, --human-numeric-sort
                 compare human readable numbers (e.g., 2K 1G)
   
          -n, --numeric-sort
                 compare according to string numerical value

The man page says sort, but we can't seem to find anything about "switch" or "parameter".
If we look at, for example, grep (perhaps one the most commonly used tools in my toolkit), swtich appears, look at the `source <http://git.savannah.gnu.org/cgit/grep.git/tree/doc/grep.in.1#n294>`.
I could find parameter but it was only in regards to awk, it was not in grep, sed, ping, awk, time, top. I did find it in `curl <https://github.com/curl/curl/blame/master/docs/curl.1#L773>`!
Here is a good a good page, describing that there is a difference in computer science in the terms: `<http://stackoverflow.com/questions/156767/whats-the-difference-between-an-argument-and-a-parameter>`
If we look at `wikipedia <https://en.wikipedia.org/wiki/Command-line_interface#Arguments>`, it seems to use all of the terms. Perhaps this really just highlights the triviality of all this :-) Why did I search for this? Curious I suppose. Have a good day!

