---
title: white_spaces_can_be_dangerous 
date: 2015-12-19
tags:
  - linux
slug: white_spaces_can_be_dangerous
description: brief look at go and spacing
---

=============================
White spaces can be dangerous
=============================

When I was some go I noticed that it refused to compile due to a white space:

.. code-block:: bash
   ./go-test-router.go:17: invalid identifier character U+2502
   ./go-test-router.go:17: syntax error: unexpected name, expecting }

I have noticed this first with Microsoft Windows formatting of white spaces and new lines. This is also true Word quotes. This was due to the fact that we had to put documentation on sharepoint. Something I still don't agree with to this day.
